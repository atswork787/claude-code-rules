# コーディングスタイル（TypeScript）

本ドキュメントは `coding-style.md` の言語非依存の原則を TypeScript に特化して補足する。

## 型安全性

理由: コンパイル時に不整合を検出し、実行時エラーを未然に防ぐ
ルール: 厳密な型付けを徹底する

- `any` 型を使用しない（`unknown` を使用し、型ガードで絞り込む）
- 戻り値の型を明示する
- 型アサーション（`as`）より型ガードを優先する
- `Readonly<T>` や `ReadonlyArray<T>` でイミュータビリティを型レベルで保証する

## 命名規則

- 変数・関数: camelCase
- クラス・型・インターフェース: PascalCase
- 定数: UPPER_SNAKE_CASE
- ファイル名: kebab-case

## import順序

```typescript
// 1. 外部ライブラリ
import { z } from 'zod'
import express from 'express'

// 2. 内部モジュール
import { UserService } from '@/services/user'

// 3. 相対パス
import { formatDate } from '../utils/date'
import { Button } from './Button'
```

## イミュータビリティ

```typescript
// 間違い: ミューテーション
function updateUser(user: User, name: string): User {
  user.name = name
  return user
}

// 正しい: スプレッド構文で新しいオブジェクトを作成
function updateUser(user: User, name: string): User {
  return {
    ...user,
    name
  }
}

// 間違い: 破壊的な配列操作
items.push(newItem)
items.splice(index, 1)

// 正しい: 非破壊的な配列操作
const added = [...items, newItem]
const removed = items.filter((_, i) => i !== index)
```

## エラー処理

```typescript
try {
  const result = await riskyOperation()
  return result
} catch (error) {
  console.error('操作が失敗しました:', error)
  throw new Error('ユーザー情報の更新に失敗しました。再度お試しください')
}
```

ログ出力の使い分け:

- `console.error` / `console.warn`: エラー処理として許可
- `console.log` / `console.debug`: デバッグ用途のため本番コードに残さない

## 入力検証（Zod）

```typescript
import { z } from 'zod'

const schema = z.object({
  email: z.string().email(),
  age: z.number().int().min(0).max(150)
})

const validated = schema.parse(input)
```
