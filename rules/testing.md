# テスト

## 前提条件

本ルールは以下のツールがプロジェクトに導入されていることを前提とする:

- Vitest（ユニット/統合テスト）
- React Testing Library（コンポーネントテスト）
- Playwright（E2Eテスト）

導入コマンド例:

```bash
npm install -D vitest @testing-library/react @testing-library/user-event @playwright/test
```

## TDDワークフロー

テストを先に書くことで仕様を明確化し、リグレッションを防止する。
実装コードより先にテストを書く（Red-Green-Refactor サイクル）。

1. **Red**: 失敗するテストを書く（期待する振る舞いを定義）
2. **Green**: テストが通る最小限の実装を書く
3. **Refactor**: テストが通ったままコードを改善する

## テスト実装の原則

- 機能の仕様を正しく理解してからテストを書くこと
- 不明な点があれば仮の実装を進めず、ユーザーに必ず確認すること
- テストが通ることよりも、正しい仕様を検証することを優先する

## テスト種別と対象

| 種別 | ツール | 対象 | テスト観点 |
| ---- | ------ | ---- | ---------- |
| ユニットテスト | Vitest | ユーティリティ関数、カスタムフック、バリデーションスキーマ | 入出力の正確性、境界値、エラーケース |
| 統合テスト | Vitest + React Testing Library | コンポーネント、フック連携 | ユーザー操作に対する画面の振る舞い |
| E2Eテスト | Playwright | 重要なユーザーフロー | タスクの追加/編集/削除/完了の一連操作 |

## テスト対象の優先順位

テストを書く価値が高い順:

1. **必須**: ビジネスロジック（ユーティリティ関数、データ変換、計算処理）
2. **必須**: カスタムフック（状態管理、副作用を伴うロジック）
3. **必須**: バリデーションスキーマ（正常値、境界値、異常値）
4. **必須**: ユーザー操作を伴うコンポーネント（フォーム入力、ボタン操作、モーダル等）
5. **必須**: E2Eで主要ユーザーフロー
6. **不要**: propsを受け取って表示するだけの純粋表示コンポーネント（E2Eで間接的にカバー）
7. **不要**: Tailwindクラスを当てるだけの薄いUIラッパー

上記1～5はすべて必須。リソースが限られる場合は番号順に優先する。

判断基準: そのコンポーネントに**条件分岐やロジックがあるか**。なければテスト不要。

## テストコードの品質基準

- テストは必ず実装の機能を検証すること
- 意味のないアサーションを書かない（例: `expect(true).toBe(true)`）
- 各テストケースは具体的な入力と期待される出力を検証する
- テスト名は「何を・どういう条件で・何が期待されるか」が分かるように書く

## ハードコーディングの禁止

- テストを通すだけのハードコードは絶対禁止
- 本番コードに `if (process.env.NODE_ENV === 'test')` のようなテスト専用の条件分岐を入れない
- テスト用の特別な値（マジックナンバー）を本番コードに埋め込まない
- 環境変数や設定ファイルを使用して、テスト環境と本番環境を適切に分離する

## カバレッジ要件

- 最低 **80%** の行カバレッジ (line coverage) を維持する
- カバレッジ計測コマンド: `npx vitest --coverage`
- カバレッジプロバイダ: `v8`（`@vitest/coverage-v8` を使用）
- カバレッジレポート形式: `text`（ターミナル表示）+ `html`（詳細確認用）
- カバレッジ設定は `vitest.config.ts` の `coverage` セクションで管理する
- ロジックを持たない純粋表示コンポーネントはカバレッジ対象から除外してよい

## テスト実行コマンド

| コマンド | 用途 |
| -------- | ---- |
| `npx vitest` | ユニット/統合テストをウォッチモードで実行 |
| `npx vitest run` | ユニット/統合テストを一括実行（CI向け） |
| `npx vitest --coverage` | カバレッジ計測付きで実行 |
| `npx playwright test` | E2Eテストを実行 |
| `npx playwright test --ui` | E2Eテストをインタラクティブモードで実行 |

## ファイル配置

推奨: 対象ファイルと同階層に配置する。既存プロジェクトで `__tests__/` ディレクトリの慣習がある場合はそちらに従ってよい。

```text
src/
  components/
    todo-list.tsx
    todo-list.test.tsx          # 推奨: 対象ファイルと同階層
  hooks/
    use-todos.ts
    use-todos.test.ts
  lib/
    storage.ts
    storage.test.ts
e2e/
  todo-crud.spec.ts             # E2Eテスト
  dashboard.spec.ts
```

命名規則:

- ユニット/統合テスト: `<対象ファイル名>.test.ts(x)`
- E2Eテスト: `<機能名>.spec.ts`

## テスト実装パターン

### ユニットテスト（ユーティリティ関数）

```typescript
import { describe, it, expect } from 'vitest'
import { filterTodosByStatus } from './todo-utils'

describe('filterTodosByStatus', () => {
  const todos: ReadonlyArray<Todo> = [
    { id: '1', title: 'Task A', status: 'completed' },
    { id: '2', title: 'Task B', status: 'pending' },
  ]

  it('指定したステータスのタスクのみ返す', () => {
    const result = filterTodosByStatus(todos, 'pending')
    expect(result).toHaveLength(1)
    expect(result[0].id).toBe('2')
  })

  it('該当なしの場合は空配列を返す', () => {
    const result = filterTodosByStatus(todos, 'in_progress')
    expect(result).toHaveLength(0)
  })
})
```

### 統合テスト（コンポーネント）

```typescript
import { describe, it, expect } from 'vitest'
import { render, screen } from '@testing-library/react'
import userEvent from '@testing-library/user-event'
import { TodoList } from './todo-list'

describe('TodoList', () => {
  it('タスクを追加できる', async () => {
    const user = userEvent.setup()
    render(<TodoList />)

    await user.type(screen.getByRole('textbox'), '新しいタスク')
    await user.click(screen.getByRole('button', { name: '追加' }))

    expect(screen.getByText('新しいタスク')).toBeInTheDocument()
  })
})
```

### 非同期処理のエラーテスト

```typescript
import { describe, it, expect } from 'vitest'

it('無効な入力でエラーをスローする', async () => {
  await expect(asyncOperation(invalidInput)).rejects.toThrow('期待するエラーメッセージ')
})
```

### E2Eテスト

Playwrightの設定で `baseURL` を指定する（`playwright.config.ts` で `use: { baseURL: 'http://localhost:3000' }`）。
`page.goto('/')` は `baseURL` からの相対パスとして解決される。

> Agent Browserを用いたセマンティックテストや、不安定なテストの隔離・アーティファクト管理など高度なE2E運用については `agents/e2e-runner.md` を参照すること。本ルールはPlaywright標準の方針を定め、`agents/e2e-runner.md` はその拡張として機能する。

```typescript
import { test, expect } from '@playwright/test'

test('タスクの追加から完了までの一連操作', async ({ page }) => {
  await page.goto('/')

  // タスクを追加
  await page.getByRole('textbox').fill('E2Eテスト用タスク')
  await page.getByRole('button', { name: '追加' }).click()

  // タスクが表示されることを確認
  await expect(page.getByText('E2Eテスト用タスク')).toBeVisible()

  // タスクを完了にする
  await page.getByRole('checkbox').click()
  await expect(page.getByText('E2Eテスト用タスク')).toHaveClass(/completed/)
})
```

## モック方針

### 基本原則

- モックは必要最小限に留め、実際の動作に近い形でテストをすること
- モックが多いほどテストの信頼性は下がることを意識する
- 外部I/O（API、DB、ファイルシステム）のみモックし、ビジネスロジックはモックしない

### テスト分離の原則

- 各テストは他のテストに依存しない（実行順序に依存しない）
- `beforeEach` で状態を初期化し、`afterEach` で副作用をクリーンアップする
- テスト間でグローバル状態を共有しない

### localStorage のモック

本プロジェクトはlocalStorageでデータ永続化するため、テストでは以下のようにモックする:

```typescript
import { beforeEach, vi } from 'vitest'

const localStorageMock = (() => {
  // モックの内部状態管理のため、イミュータブルパターンの例外としてletを使用
  let store: Record<string, string> = {}
  return {
    getItem: vi.fn((key: string) => store[key] ?? null),
    setItem: vi.fn((key: string, value: string) => {
      store = { ...store, [key]: value }
    }),
    removeItem: vi.fn((key: string) => {
      const { [key]: _, ...rest } = store
      store = rest
    }),
    clear: vi.fn(() => {
      store = {}
    }),
  }
})()

beforeEach(() => {
  Object.defineProperty(window, 'localStorage', { value: localStorageMock })
  localStorageMock.clear()
})
```

### 外部依存のモック原則

- localStorage: 常にモックする（テスト間の独立性を確保）
- 日付/時刻: `vi.useFakeTimers()` でモックする
- ランダム値: `vi.spyOn(Math, 'random')` で固定する
- 外部APIを将来追加した場合: MSW (Mock Service Worker) を使用する

## 検証レポート

実装完了時に以下の検証を実行し、結果をユーザーに報告する。

### 検証ツールの前提

以下のツールがプロジェクトに導入されていること:

- ESLint（静的解析）
- Prettier（コードフォーマット）

設定ファイルが未整備の場合は、検証項目4・5をスキップしてよい。

### 実行する検証

| # | 検証項目 | コマンド | 合格基準 |
| - | -------- | -------- | -------- |
| 1 | ユニット/統合テスト | `npx vitest run` | 全テスト合格 |
| 2 | カバレッジ | `npx vitest --coverage` | 行カバレッジ 80% 以上 |
| 3 | E2Eテスト | `npx playwright test` | 全テスト合格 |
| 4 | 静的解析 | `npx eslint src/` | 警告・エラーなし |
| 5 | フォーマット | `npx prettier --check src/` | 差分なし |
| 6 | セキュリティ | `npm audit` | 重大な脆弱性なし |

### レポートフォーマット

検証完了後、以下の形式でユーザーに報告する:

```markdown
## 検証結果

| 検証項目 | 結果 | 詳細 |
| -------- | ---- | ---- |
| テスト   | OK / NG | xx passed, xx failed |
| カバレッジ | OK / NG | xx.x% (目標: 80%) |
| E2E      | OK / NG | xx passed, xx failed |
| ESLint   | OK / NG | xx warnings, xx errors |
| Prettier | OK / NG | xx files differ |
| npm audit | OK / NG | xx vulnerabilities |

総合判定: OK / NG
```

### 検証ルール

- 検証で問題が見つかった場合は、修正してから再検証する
- すべての検証が合格するまで「完了」としない
- 検証結果は省略せず、必ずユーザーに報告する

## 関連ルール

> 注: 以下のルールファイルが存在しない場合は無視して、本ファイルおよびルート `CLAUDE.md` の記述に従うこと。

- コーディング規約: `coding-style.md`
- セキュリティ: `security.md`
- エージェント運用: `agents.md`
