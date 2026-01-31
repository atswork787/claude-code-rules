# コーディングスタイル

## イミュータビリティ（クリティカル）

常に新しいオブジェクトを作成し、決してミューテートしない:

```javascript
// 間違い: ミューテーション
function updateUser(user, name) {
  user.name = name  // ミューテーション！
  return user
}

// 正しい: イミュータビリティ
function updateUser(user, name) {
  return {
    ...user,
    name
  }
}
```

## ファイル構成

多数の小さなファイル > 少数の大きなファイル:
- 高凝集、低結合
- 通常200-400行、最大800行
- 大きなコンポーネントからユーティリティを抽出
- 型ではなく機能/ドメインで整理

## エラー処理

常に包括的にエラーを処理:

```typescript
try {
  const result = await riskyOperation()
  return result
} catch (error) {
  console.error('操作が失敗しました:', error)
  throw new Error('詳細なユーザーフレンドリーなメッセージ')
}
```

## 入力検証

常にユーザー入力を検証:

```typescript
import { z } from 'zod'

const schema = z.object({
  email: z.string().email(),
  age: z.number().int().min(0).max(150)
})

const validated = schema.parse(input)
```

## コード品質チェックリスト

作業完了前に:
- [ ] コードが読みやすく適切に命名されている
- [ ] 関数が小さい（<50行）
- [ ] ファイルが焦点を絞っている（<800行）
- [ ] 深いネストがない（>4レベル）
- [ ] 適切なエラー処理
- [ ] console.logステートメントがない
- [ ] ハードコードされた値がない
- [ ] ミューテーションがない（イミュータブルパターン使用）
