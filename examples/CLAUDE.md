# 例 プロジェクトCLAUDE.md

これは、プロジェクトレベルのCLAUDE.mdファイルの例です。プロジェクトルートに配置してください。

## プロジェクト概要

[プロジェクトの簡単な説明 - 何をするか、技術スタック]

## 重要なルール

### 1. コード構成

- 少数の大きなファイルよりも多数の小さなファイル
- 高凝集、低結合
- 通常200-400行、ファイルあたり最大800行
- 型ではなく機能/ドメインで整理

### 2. コードスタイル

- コード、コメント、またはドキュメントに絵文字なし
- 常にイミュータビリティ - オブジェクトや配列を決してミューテートしない
- 本番コードにconsole.logなし
- try/catchで適切なエラー処理
- Zodまたは同様のもので入力検証

### 3. テスト

- TDD: まずテストを書く
- 最低80%のカバレッジ
- ユーティリティのユニットテスト
- APIの統合テスト
- 重要なフローのE2Eテスト

### 4. セキュリティ

- ハードコードされた秘密情報なし
- 機密データは環境変数
- すべてのユーザー入力を検証
- パラメータ化されたクエリのみ
- CSRF保護を有効化

## ファイル構造

```
src/
|-- app/              # Next.jsアプリルーター
|-- components/       # 再利用可能なUIコンポーネント
|-- hooks/            # カスタムReactフック
|-- lib/              # ユーティリティライブラリ
|-- types/            # TypeScript定義
```

## 主要パターン

### APIレスポンス形式

```typescript
interface ApiResponse<T> {
  success: boolean
  data?: T
  error?: string
}
```

### エラー処理

```typescript
try {
  const result = await operation()
  return { success: true, data: result }
} catch (error) {
  console.error('操作が失敗しました:', error)
  return { success: false, error: 'ユーザーフレンドリーなメッセージ' }
}
```

## 環境変数

```bash
# 必須
DATABASE_URL=
API_KEY=

# オプション
DEBUG=false
```

## 利用可能なコマンド

- `/tdd` - テスト駆動開発ワークフロー
- `/plan` - 実装計画を作成
- `/code-review` - コード品質をレビュー
- `/build-fix` - ビルドエラーを修正

## モジュール化されたルール

詳細なガイドラインは`~/.claude/rules/`にあります:

| ルールファイル | 内容 |
| ----------- | ---------- |
| security.md | セキュリティチェック、秘密情報管理 |
| coding-style.md | イミュータビリティ、ファイル構成、エラー処理 |
| testing.md | TDDワークフロー、80%カバレッジ要件 |
| git-workflow.md | コミット形式、PRワークフロー |
| agents.md | エージェントオーケストレーション、どのエージェントをいつ使用するか |
| patterns.md | APIレスポンス、リポジトリパターン |
| performance.md | モデル選択、コンテキスト管理 |
| hooks.md | フックシステム |

## 利用可能なエージェント

`~/.claude/agents/`に配置:

| エージェント | 目的 |
| ------- | --------- |
| planner | 機能実装計画 |
| architect | システム設計とアーキテクチャ |
| tdd-guide | テスト駆動開発 |
| code-reviewer | 品質/セキュリティのコードレビュー |
| security-reviewer | セキュリティ脆弱性分析 |
| build-error-resolver | ビルドエラー解決 |
| e2e-runner | Playwright E2Eテスト |
| refactor-cleaner | デッドコードクリーンアップ |
| doc-updater | ドキュメント更新 |

## 成功指標

以下の場合に成功です:

- すべてのテストが合格（80%以上のカバレッジ）
- セキュリティ脆弱性なし
- コードが読みやすく保守可能
- ユーザー要件が満たされている

## Gitワークフロー

- 慣例的なコミット: `feat:`、`fix:`、`refactor:`、`docs:`、`test:`
- mainに直接コミットしない
- PRはレビューが必要
- マージ前にすべてのテストが合格する必要がある
