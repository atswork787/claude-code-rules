# Evalコマンド

評価駆動開発ワークフローを管理します。

## 使用法

`/eval [define|check|report|list] [feature-name]`

## 評価を定義

`/eval define feature-name`

新しい評価定義を作成:

1. テンプレートで`.claude/evals/feature-name.md`を作成:

```markdown
## EVAL: feature-name
作成日: $(date)

### 機能評価
- [ ] [機能1の説明]
- [ ] [機能2の説明]

### リグレッション評価
- [ ] [既存の動作1がまだ動作する]
- [ ] [既存の動作2がまだ動作する]

### 成功基準
- 機能評価でpass@3 > 90%
- リグレッション評価でpass^3 = 100%
```

2. ユーザーに特定の基準を記入するよう促す

## 評価をチェック

`/eval check feature-name`

機能の評価を実行:

1. `.claude/evals/feature-name.md`から評価定義を読み取る
2. 各機能評価について:
   - 基準を検証しようと試みる
   - PASS/FAILを記録
   - `.claude/evals/feature-name.log`に試行をログ
3. 各リグレッション評価について:
   - 関連するテストを実行
   - ベースラインと比較
   - PASS/FAILを記録
4. 現在のステータスを報告:

```
評価チェック: feature-name
========================
機能: X/Y合格
リグレッション: X/Y合格
ステータス: 進行中 / 準備完了
```

## 評価を報告

`/eval report feature-name`

包括的な評価レポートを生成:

```
評価レポート: feature-name
=========================
生成日: $(date)

機能評価
----------------
[eval-1]: PASS (pass@1)
[eval-2]: PASS (pass@2) - 再試行が必要だった
[eval-3]: FAIL - 注記を参照

リグレッション評価
----------------
[test-1]: PASS
[test-2]: PASS
[test-3]: PASS

メトリクス
-------
機能pass@1: 67%
機能pass@3: 100%
リグレッションpass^3: 100%

注記
-----
[問題、エッジケース、または観察事項]

推奨
--------------
[SHIP / 要作業 / ブロック]
```

## 評価をリスト

`/eval list`

すべての評価定義を表示:

```
評価定義
================
feature-auth      [3/5合格] 進行中
feature-search    [5/5合格] 準備完了
feature-export    [0/4合格] 未開始
```

## 引数

$ARGUMENTS:
- `define <n>` - 新しい評価定義を作成
- `check <n>` - 評価を実行してチェック
- `report <n>` - 完全なレポートを生成
- `list` - すべての評価を表示
- `clean` - 古い評価ログを削除（最後の10回の実行を保持）
