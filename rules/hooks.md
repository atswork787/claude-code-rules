# フックシステム

## フックタイプ

- **PreToolUse**: ツール実行前（検証、パラメータ変更）
- **PostToolUse**: ツール実行後（自動フォーマット、チェック）
- **Stop**: セッション終了時（最終検証）

## 現在のフック（~/.claude/settings.jsonに）

### PreToolUse
- **tmuxリマインダー**: 長時間実行コマンド（npm、pnpm、yarn、cargoなど）にtmuxを提案
- **git pushレビュー**: プッシュ前にZedでレビューを開く
- **docブロッカー**: 不必要な.md/.txtファイルの作成をブロック

### PostToolUse
- **PR作成**: PR URLとGitHub Actionsステータスをログ
- **Prettier**: 編集後にJS/TSファイルを自動フォーマット
- **TypeScriptチェック**: .ts/.tsxファイル編集後にtscを実行
- **console.log警告**: 編集されたファイルのconsole.logについて警告

### Stop
- **console.log監査**: セッション終了前にすべての変更されたファイルのconsole.logをチェック

## 自動承認権限

注意して使用:
- 信頼できる、明確に定義された計画に対して有効化
- 探索的作業では無効化
- 決してdangerously-skip-permissionsフラグを使用しない
- 代わりに`~/.claude.json`で`allowedTools`を設定

## TodoWriteベストプラクティス

TodoWriteツールを使用して:
- 複数ステップタスクの進捗を追跡
- 指示の理解を検証
- リアルタイムの操縦を可能にする
- 詳細な実装ステップを表示

Todoリストが明らかにするもの:
- 順序が間違っているステップ
- 欠落しているアイテム
- 余分な不要なアイテム
- 間違った粒度
- 誤解された要件
