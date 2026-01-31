---
name: coding-standards
description: TypeScript、JavaScript、React、Node.js開発のための普遍的なコーディング標準、ベストプラクティス、パターン。
---

# コーディング標準とベストプラクティス

すべてのプロジェクトに適用可能な普遍的なコーディング標準。

## コード品質の原則

### 1. 可読性第一
- コードは書くよりも読まれる
- 明確な変数名と関数名
- コメントよりも自己文書化コードを優先
- 一貫したフォーマット

### 2. KISS（Keep It Simple, Stupid）
- 動作する最もシンプルな解決策
- 過度な設計を避ける
- 早すぎる最適化をしない
- 理解しやすい > 巧妙なコード

### 3. DRY（Don't Repeat Yourself）
- 共通ロジックを関数に抽出
- 再利用可能なコンポーネントを作成
- モジュール間でユーティリティを共有
- コピー＆ペーストプログラミングを避ける

### 4. YAGNI（You Aren't Gonna Need It）
- 必要になる前に機能を構築しない
- 投機的な汎用性を避ける
- 必要な時だけ複雑さを追加
- シンプルに始めて、必要時にリファクタリング

## TypeScript/JavaScript標準

### 変数命名

```typescript
// ✅ 良い: 説明的な名前
const marketSearchQuery = 'election'
const isUserAuthenticated = true
const totalRevenue = 1000

// ❌ 悪い: 不明確な名前
const q = 'election'
const flag = true
const x = 1000
```

### 関数命名

```typescript
// ✅ 良い: 動詞-名詞パターン
async function fetchMarketData(marketId: string) { }
function calculateSimilarity(a: number[], b: number[]) { }
function isValidEmail(email: string): boolean { }

// ❌ 悪い: 不明確または名詞のみ
async function market(id: string) { }
function similarity(a, b) { }
function email(e) { }
```

### イミュータビリティパターン（クリティカル）

```typescript
// ✅ 常にスプレッド演算子を使用
const updatedUser = {
  ...user,
  name: '新しい名前'
}

const updatedArray = [...items, newItem]

// ❌ 決して直接ミューテートしない
user.name = '新しい名前'  // 悪い
items.push(newItem)     // 悪い
```

### エラー処理

```typescript
// ✅ 良い: 包括的なエラー処理
async function fetchData(url: string) {
  try {
    const response = await fetch(url)

    if (!response.ok) {
      throw new Error(`HTTP ${response.status}: ${response.statusText}`)
    }

    return await response.json()
  } catch (error) {
    console.error('取得失敗:', error)
    throw new Error('データの取得に失敗しました')
  }
}

// ❌ 悪い: エラー処理なし
async function fetchData(url) {
  const response = await fetch(url)
  return response.json()
}
```

**覚えておく**: コード品質は交渉の余地がありません。明確で保守可能なコードは、迅速な開発と自信を持ったリファクタリングを可能にします。
