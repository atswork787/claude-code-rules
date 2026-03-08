# Evalコマンド

評価駆動開発ワークフローを管理します。

## 使用法

`/eval [define|check|report|list|clean] [feature-name]`

## 評価を定義

`/eval define feature-name`

新しい評価定義を作成:

1. テンプレートで`.claude/evals/feature-name.md`を作成:

```markdown
## EVAL: feature-name
作成日: {YYYY-MM-DD形式の現在日付}

### 機能評価
- [ ] [機能1の説明]
- [ ] [機能2の説明]

### リグレッション評価
- [ ] [既存の動作1がまだ動作する]
- [ ] [既存の動作2がまだ動作する]

### 成功基準
- 機能評価でpass@3 > 90%
- リグレッション評価でpass@3 = 100%
```

2. ユーザーに特定の基準を記入するよう促す

## 評価をチェック

`/eval check feature-name`

**注意**: `.claude/evals/feature-name.md` が存在しない場合は、エラーメッセージを表示してユーザーに `/eval define <feature-name>` の実行を促す。

機能の評価を実行:

1. `.claude/evals/feature-name.md`から評価定義を読み取る
2. 各機能評価について:
   - 評価定義に記載されたチェック項目をコード・ファイル・テスト実行で確認する
   - PASS/FAILを記録
   - `.claude/evals/feature-name.log` に以下の形式でログを追記する:

     ```text
     [YYYY-MM-DD HH:MM] eval-1: PASS
     [YYYY-MM-DD HH:MM] eval-2: FAIL - 理由
     ```

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

**注意**: `.claude/evals/feature-name.md` が存在しない場合は、エラーメッセージを表示してユーザーに `/eval define <feature-name>` の実行を促す。

包括的な評価レポートを生成:

```
評価レポート: feature-name
=========================
生成日: {YYYY-MM-DD形式の現在日付}

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
リグレッションpass@3: 100%

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

1. `.claude/evals/` 配下の全 `.md` ファイルを走査する
2. 各ファイルのチェックボックス `[x]` / `[ ]` の数を集計して合格/全体を算出する
3. 対応する `.log` ファイルが存在する場合はステータスを「進行中」、全項目合格なら「準備完了」、ログなしは「未開始」と表示する

```
評価定義
================
feature-auth      [3/5合格] 進行中
feature-search    [5/5合格] 準備完了
feature-export    [0/4合格] 未開始
```

## 評価をクリーン

`/eval clean`

`.claude/evals/` 配下の各 `.log` ファイルについて、最新10エントリを残して古いエントリを削除する。削除前にリストを表示してユーザーに確認を求める。

## 引数

$ARGUMENTS:

- `define <feature-name>` - 新しい評価定義を作成
- `check <feature-name>` - 評価を実行してチェック
- `report <feature-name>` - 完全なレポートを生成
- `list` - すべての評価を表示
- `clean` - 各ログファイルの最新10エントリを残して古いエントリを削除
