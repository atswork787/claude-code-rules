---
description: テスト駆動開発ワークフローを強制します。インターフェースをスキャフォールドし、まずテストを生成し、その後、合格するための最小限のコードを実装します。80%以上のカバレッジを確保します。
---

# TDDコマンド

このコマンドは、テスト駆動開発方法論を強制するために**tdd-guide**エージェントを呼び出します。

## このコマンドが行うこと

1. **インターフェースのスキャフォールド** - 最初に型/インターフェースを定義
2. **まずテストを生成** - 失敗するテストを書く（RED）
3. **最小限のコードを実装** - 合格するために必要最小限を書く（GREEN）
4. **リファクタリング** - テストをグリーンに保ちながらコードを改善（REFACTOR）
5. **カバレッジを検証** - 80%以上のテストカバレッジを確保

## 使用する場合

以下の場合に`/tdd`を使用:
- 新機能の実装
- 新しい関数/コンポーネントの追加
- バグ修正（まずバグを再現するテストを書く）
- 既存コードのリファクタリング
- 重要なビジネスロジックの構築

## 動作方法

tdd-guideエージェントは以下を行います:

1. **インターフェースを定義**入力/出力用
2. **失敗するテストを書く**（コードがまだ存在しないため）
3. **テストを実行**して正しい理由で失敗することを確認
4. **最小限の実装を書く**テストを合格させるために
5. **テストを実行**して合格することを確認
6. **コードをリファクタリング**テストをグリーンに保ちながら
7. **カバレッジをチェック**80%未満の場合はさらにテストを追加

## TDDサイクル

```
RED → GREEN → REFACTOR → 繰り返し

RED:      失敗するテストを書く
GREEN:    合格するための最小限のコードを書く
REFACTOR: コードを改善し、テストを合格させ続ける
繰り返し: 次の機能/シナリオ
```

## 使用例

```
ユーザー: /tdd マーケット流動性スコアを計算する関数が必要です

エージェント (tdd-guide):
# TDDセッション: マーケット流動性スコア計算機

## ステップ1: インターフェースを定義（スキャフォールド）

```typescript
// lib/liquidity.ts
export interface MarketData {
  totalVolume: number
  bidAskSpread: number
  activeTraders: number
  lastTradeTime: Date
}

export function calculateLiquidityScore(market: MarketData): number {
  // TODO: 実装
  throw new Error('未実装')
}
```

## ステップ2: 失敗するテストを書く（RED）

```typescript
// lib/liquidity.test.ts
import { calculateLiquidityScore } from './liquidity'

describe('calculateLiquidityScore', () => {
  it('流動性の高いマーケットに対して高いスコアを返すべき', () => {
    const market = {
      totalVolume: 100000,
      bidAskSpread: 0.01,
      activeTraders: 500,
      lastTradeTime: new Date()
    }

    const score = calculateLiquidityScore(market)

    expect(score).toBeGreaterThan(80)
    expect(score).toBeLessThanOrEqual(100)
  })

  it('流動性の低いマーケットに対して低いスコアを返すべき', () => {
    const market = {
      totalVolume: 100,
      bidAskSpread: 0.5,
      activeTraders: 2,
      lastTradeTime: new Date(Date.now() - 86400000) // 1日前
    }

    const score = calculateLiquidityScore(market)

    expect(score).toBeLessThan(30)
    expect(score).toBeGreaterThanOrEqual(0)
  })

  it('エッジケースを処理すべき: ゼロボリューム', () => {
    const market = {
      totalVolume: 0,
      bidAskSpread: 0,
      activeTraders: 0,
      lastTradeTime: new Date()
    }

    const score = calculateLiquidityScore(market)

    expect(score).toBe(0)
  })
})
```

## ステップ3: テストを実行 - 失敗を確認

```bash
npm test lib/liquidity.test.ts

FAIL lib/liquidity.test.ts
  ✕ 流動性の高いマーケットに対して高いスコアを返すべき (2 ms)
    Error: 未実装

1テスト失敗、0テスト合格
```

✅ 期待通りテストが失敗しました。実装の準備ができました。

## ステップ4: 最小限のコードを実装（GREEN）

```typescript
// lib/liquidity.ts
export function calculateLiquidityScore(market: MarketData): number {
  // ゼロボリュームのエッジケースを処理
  if (market.totalVolume === 0) {
    return 0
  }

  // コンポーネントスコアを計算（0-100スケール）
  const volumeScore = Math.min(market.totalVolume / 1000, 100)
  const spreadScore = Math.max(100 - (market.bidAskSpread * 1000), 0)
  const traderScore = Math.min(market.activeTraders / 10, 100)

  // 最近のアクティビティボーナス
  const hoursSinceLastTrade = (Date.now() - market.lastTradeTime.getTime()) / (1000 * 60 * 60)
  const recencyScore = Math.max(100 - (hoursSinceLastTrade * 10), 0)

  // 加重平均
  const score = (
    volumeScore * 0.4 +
    spreadScore * 0.3 +
    traderScore * 0.2 +
    recencyScore * 0.1
  )

  return Math.min(Math.max(score, 0), 100) // 0-100にクランプ
}
```

## ステップ5: テストを実行 - 合格を確認

```bash
npm test lib/liquidity.test.ts

PASS lib/liquidity.test.ts
  ✓ 流動性の高いマーケットに対して高いスコアを返すべき (3 ms)
  ✓ 流動性の低いマーケットに対して低いスコアを返すべき (2 ms)
  ✓ エッジケースを処理すべき: ゼロボリューム (1 ms)

3テスト合格
```

✅ すべてのテストが合格！

## ステップ8: カバレッジをチェック

```bash
npm test -- --coverage lib/liquidity.test.ts

ファイル         | % Stmts | % Branch | % Funcs | % Lines
---------------|---------|----------|---------|--------
liquidity.ts   |   100   |   100    |   100   |   100

カバレッジ: 100% ✅（目標: 80%）
```

✅ TDDセッション完了！
```

## TDDベストプラクティス

**すべきこと:**
- ✅ 実装前にまずテストを書く
- ✅ 実装前にテストを実行して失敗を確認
- ✅ テストを合格させるための最小限のコードを書く
- ✅ テストがグリーンになってからリファクタリング
- ✅ エッジケースとエラーシナリオを追加
- ✅ 80%以上のカバレッジを目指す（重要なコードは100%）

**してはいけないこと:**
- ❌ テスト前に実装を書く
- ❌ 各変更後のテスト実行をスキップ
- ❌ 一度に多くのコードを書きすぎる
- ❌ 失敗するテストを無視
- ❌ 実装の詳細をテスト（動作をテスト）
- ❌ すべてをモック（統合テストを優先）

## 含めるべきテストタイプ

**ユニットテスト**（関数レベル）:
- ハッピーパスシナリオ
- エッジケース（空、null、最大値）
- エラー条件
- 境界値

**統合テスト**（コンポーネントレベル）:
- APIエンドポイント
- データベース操作
- 外部サービス呼び出し
- フック付きReactコンポーネント

**E2Eテスト**（`/e2e`コマンドを使用）:
- 重要なユーザーフロー
- 複数ステップのプロセス
- フルスタック統合

## カバレッジ要件

- すべてのコードに**最低80%**
- 以下には**100%必須**:
  - 財務計算
  - 認証ロジック
  - セキュリティクリティカルなコード
  - コアビジネスロジック

## 重要な注意事項

**必須**: テストは実装前に書かなければなりません。TDDサイクルは:

1. **RED** - 失敗するテストを書く
2. **GREEN** - 合格するように実装
3. **REFACTOR** - コードを改善

REDフェーズをスキップしないでください。テスト前にコードを書かないでください。

## 他のコマンドとの統合

- まず`/plan`を使用して何を構築するかを理解
- テスト付きで実装するには`/tdd`を使用
- ビルドエラーが発生した場合は`/build-and-fix`を使用
- 実装をレビューするには`/code-review`を使用
- カバレッジを検証するには`/test-coverage`を使用

## 関連エージェント

このコマンドは以下にある`tdd-guide`エージェントを呼び出します:
`~/.claude/agents/tdd-guide.md`

そして以下の`tdd-workflow`スキルを参照できます:
`~/.claude/skills/tdd-workflow/`
