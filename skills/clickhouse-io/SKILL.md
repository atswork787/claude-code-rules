---
name: clickhouse-io
description: 高パフォーマンスな分析ワークロードのためのClickHouseデータベースパターン、クエリ最適化、分析、データエンジニアリングベストプラクティス。
---

# ClickHouse分析パターン

高パフォーマンスな分析とデータエンジニアリングのためのClickHouse固有のパターン。

## 概要

ClickHouseは、オンライン分析処理（OLAP）のための列指向データベース管理システム（DBMS）です。大規模データセットでの高速な分析クエリ用に最適化されています。

**主な機能:**
- 列指向ストレージ
- データ圧縮
- 並列クエリ実行
- 分散クエリ
- リアルタイム分析

## テーブル設計パターン

### MergeTreeエンジン（最も一般的）

```sql
CREATE TABLE markets_analytics (
    date Date,
    market_id String,
    market_name String,
    volume UInt64,
    trades UInt32,
    unique_traders UInt32,
    avg_trade_size Float64,
    created_at DateTime
) ENGINE = MergeTree()
PARTITION BY toYYYYMM(date)
ORDER BY (date, market_id)
SETTINGS index_granularity = 8192;
```

### ReplacingMergeTree（重複排除）

```sql
-- 重複がある可能性のあるデータ用（例: 複数ソースから）
CREATE TABLE user_events (
    event_id String,
    user_id String,
    event_type String,
    timestamp DateTime,
    properties String
) ENGINE = ReplacingMergeTree()
PARTITION BY toYYYYMM(timestamp)
ORDER BY (user_id, event_id, timestamp)
PRIMARY KEY (user_id, event_id);
```

## クエリ最適化パターン

### 効率的なフィルタリング

```sql
-- ✅ 良い: インデックス列を最初に使用
SELECT *
FROM markets_analytics
WHERE date >= '2025-01-01'
  AND market_id = 'market-123'
  AND volume > 1000
ORDER BY date DESC
LIMIT 100;

-- ❌ 悪い: インデックスされていない列で最初にフィルター
SELECT *
FROM markets_analytics
WHERE volume > 1000
  AND market_name LIKE '%election%'
  AND date >= '2025-01-01';
```

### 集約

```sql
-- ✅ 良い: ClickHouse固有の集約関数を使用
SELECT
    toStartOfDay(created_at) AS day,
    market_id,
    sum(volume) AS total_volume,
    count() AS total_trades,
    uniq(trader_id) AS unique_traders,
    avg(trade_size) AS avg_size
FROM trades
WHERE created_at >= today() - INTERVAL 7 DAY
GROUP BY day, market_id
ORDER BY day DESC, total_volume DESC;

-- ✅ パーセンタイルにはquantileを使用（percentileより効率的）
SELECT
    quantile(0.50)(trade_size) AS median,
    quantile(0.95)(trade_size) AS p95,
    quantile(0.99)(trade_size) AS p99
FROM trades
WHERE created_at >= now() - INTERVAL 1 HOUR;
```

## データ挿入パターン

### バルク挿入（推奨）

```typescript
import { ClickHouse } from 'clickhouse'

const clickhouse = new ClickHouse({
  url: process.env.CLICKHOUSE_URL,
  port: 8123,
  basicAuth: {
    username: process.env.CLICKHOUSE_USER,
    password: process.env.CLICKHOUSE_PASSWORD
  }
})

// ✅ バッチ挿入（効率的）
async function bulkInsertTrades(trades: Trade[]) {
  const values = trades.map(trade => `(
    '${trade.id}',
    '${trade.market_id}',
    '${trade.user_id}',
    ${trade.amount},
    '${trade.timestamp.toISOString()}'
  )`).join(',')

  await clickhouse.query(`
    INSERT INTO trades (id, market_id, user_id, amount, timestamp)
    VALUES ${values}
  `).toPromise()
}

// ❌ 個別挿入（遅い）
async function insertTrade(trade: Trade) {
  // ループでこれをしないでください！
  await clickhouse.query(`
    INSERT INTO trades VALUES ('${trade.id}', ...)
  `).toPromise()
}
```

## マテリアライズドビュー

### リアルタイム集約

```sql
-- 時間別統計のマテリアライズドビューを作成
CREATE MATERIALIZED VIEW market_stats_hourly_mv
TO market_stats_hourly
AS SELECT
    toStartOfHour(timestamp) AS hour,
    market_id,
    sumState(amount) AS total_volume,
    countState() AS total_trades,
    uniqState(user_id) AS unique_users
FROM trades
GROUP BY hour, market_id;

-- マテリアライズドビューをクエリ
SELECT
    hour,
    market_id,
    sumMerge(total_volume) AS volume,
    countMerge(total_trades) AS trades,
    uniqMerge(unique_users) AS users
FROM market_stats_hourly
WHERE hour >= now() - INTERVAL 24 HOUR
GROUP BY hour, market_id;
```

## 一般的な分析クエリ

### 時系列分析

```sql
-- 日次アクティブユーザー
SELECT
    toDate(timestamp) AS date,
    uniq(user_id) AS daily_active_users
FROM events
WHERE timestamp >= today() - INTERVAL 30 DAY
GROUP BY date
ORDER BY date;

-- リテンション分析
SELECT
    signup_date,
    countIf(days_since_signup = 0) AS day_0,
    countIf(days_since_signup = 1) AS day_1,
    countIf(days_since_signup = 7) AS day_7,
    countIf(days_since_signup = 30) AS day_30
FROM (
    SELECT
        user_id,
        min(toDate(timestamp)) AS signup_date,
        toDate(timestamp) AS activity_date,
        dateDiff('day', signup_date, activity_date) AS days_since_signup
    FROM events
    GROUP BY user_id, activity_date
)
GROUP BY signup_date
ORDER BY signup_date DESC;
```

## ベストプラクティス

### 1. パーティショニング戦略
- 時間でパーティション（通常は月または日）
- 多すぎるパーティションを避ける（パフォーマンス影響）
- パーティションキーにDATE型を使用

### 2. 順序キー
- 最も頻繁にフィルターされる列を最初に配置
- カーディナリティを考慮（高カーディナリティを最初に）
- 順序は圧縮に影響

### 3. データ型
- 最小の適切な型を使用（UInt32 vs UInt64）
- 繰り返し文字列にLowCardinalityを使用
- カテゴリデータにEnumを使用

### 4. 避けるべきこと
- SELECT *（列を指定）
- FINAL（クエリ前にデータをマージ）
- 多すぎるJOIN（分析用に非正規化）
- 小さな頻繁な挿入（代わりにバッチ）

### 5. 監視
- クエリパフォーマンスを追跡
- ディスク使用量を監視
- マージ操作をチェック
- 遅いクエリログをレビュー

**覚えておく**: ClickHouseは分析ワークロードに優れています。クエリパターン用にテーブルを設計し、挿入をバッチ化し、リアルタイム集約にマテリアライズドビューを活用してください。
