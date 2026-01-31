# チェックポイントコマンド

ワークフローでチェックポイントを作成または検証します。

## 使用法

`/checkpoint [create|verify|list] [name]`

## チェックポイントを作成

チェックポイントを作成する際:

1. `/verify quick`を実行して現在の状態がクリーンであることを確認
2. チェックポイント名でgit stashまたはコミットを作成
3. チェックポイントを`.claude/checkpoints.log`にログ:

```bash
echo "$(date +%Y-%m-%d-%H:%M) | $CHECKPOINT_NAME | $(git rev-parse --short HEAD)" >> .claude/checkpoints.log
```

4. チェックポイント作成を報告

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

## チェックポイントをリスト

すべてのチェックポイントを以下で表示:
- 名前
- タイムスタンプ
- Git SHA
- ステータス（現在、遅れている、進んでいる）

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

## 引数

$ARGUMENTS:
- `create <n>` - 名前付きチェックポイントを作成
- `verify <n>` - 名前付きチェックポイントに対して検証
- `list` - すべてのチェックポイントを表示
- `clear` - 古いチェックポイントを削除（最後の5つを保持）
