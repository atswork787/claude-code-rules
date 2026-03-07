# チェックポイントコマンド

ワークフローでチェックポイントを作成または検証します。

## 使用法

`/checkpoint [create|verify|list] [name]`

## チェックポイントを作成

チェックポイントを作成する際:

1. `/verify full`を実行してビルド・型・Lint・テストがすべてクリーンであることを確認
2. チェックポイント名でgit stashまたはコミットを作成（コミット可能な状態であれば `git commit`、未コミットの変更がある場合は `git stash`）
3. チェックポイントを`.claude/checkpoints.log`にログ（テスト結果とカバレッジはステップ1の`/verify full`出力から取得）:

```bash
echo "$(date +%Y-%m-%d-%H:%M) | $CHECKPOINT_NAME | $(git rev-parse --short HEAD) | tests:PASS_COUNT/TOTAL | coverage:COVERAGE%" >> .claude/checkpoints.log
```

4. チェックポイント作成を報告

**エラーケース:**

- `.claude/checkpoints.log` が存在しない場合は自動作成する
- 同名のチェックポイントが既に存在する場合は、タイムスタンプを付与してリネーム（例: `name-20240101-120000`）

## チェックポイントを検証

チェックポイントに対して検証する際:

1. ログからチェックポイントを読み取る
2. 現在の状態をチェックポイントと比較:
   - チェックポイント以降追加されたファイル
   - チェックポイント以降変更されたファイル
   - 現在とその時のテスト合格率
   - 現在とその時のカバレッジ

3. 報告:
```
チェックポイント比較: $NAME
============================
変更されたファイル: X
テスト: +Y合格 / -Z失敗
カバレッジ: +X% / -Y%
ビルド: [PASS/FAIL]
```

**エラーケース:**

- 指定した名前のチェックポイントが見つからない場合はエラーを報告して終了する

## チェックポイントをリスト

すべてのチェックポイントを以下で表示:
- 名前
- タイムスタンプ
- Git SHA
- ステータス（現在のHEADと一致するか否か）

## ワークフロー

典型的なチェックポイントフロー:

```
[開始] --> /checkpoint create "feature-start"
   |
[実装] --> /checkpoint create "core-done"
   |
[テスト] --> /checkpoint verify "core-done"
   |
[リファクタ] --> /checkpoint create "refactor-done"
   |
[PR] --> /checkpoint verify "feature-start"
```

## チェックポイントをクリア

古いチェックポイントを削除する際:

1. `.claude/checkpoints.log` からチェックポイント一覧を読み取る
2. タイムスタンプ順で最新の5つを保持し、残りを削除
3. 削除したチェックポイント数を報告

## 引数

$ARGUMENTS:

- `create <name>` - 名前付きチェックポイントを作成
- `verify <name>` - 名前付きチェックポイントに対して検証
- `list` - すべてのチェックポイントを表示
- `clear` - 古いチェックポイントを削除（最後の5つを保持）
