---
name: frontend-patterns
description: React、Next.js、状態管理、パフォーマンス最適化、UIベストプラクティスのためのフロントエンド開発パターン。
---

# フロントエンド開発パターン

React、Next.js、パフォーマンスの高いユーザーインターフェースのための現代的なフロントエンドパターン。

## コンポーネントパターン

### 継承よりもコンポジション

```typescript
// ✅ 良い: コンポーネントコンポジション
interface CardProps {
  children: React.ReactNode
  variant?: 'default' | 'outlined'
}

export function Card({ children, variant = 'default' }: CardProps) {
  return <div className={`card card-${variant}`}>{children}</div>
}

export function CardHeader({ children }: { children: React.ReactNode }) {
  return <div className="card-header">{children}</div>
}

export function CardBody({ children }: { children: React.ReactNode }) {
  return <div className="card-body">{children}</div>
}

// 使用法
<Card>
  <CardHeader>タイトル</CardHeader>
  <CardBody>コンテンツ</CardBody>
</Card>
```

## カスタムフックパターン

### 状態管理フック

```typescript
export function useToggle(initialValue = false): [boolean, () => void] {
  const [value, setValue] = useState(initialValue)

  const toggle = useCallback(() => {
    setValue(v => !v)
  }, [])

  return [value, toggle]
}

// 使用法
const [isOpen, toggleOpen] = useToggle()
```

### デバウンスフック

```typescript
export function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState<T>(value)

  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value)
    }, delay)

    return () => clearTimeout(handler)
  }, [value, delay])

  return debouncedValue
}

// 使用法
const [searchQuery, setSearchQuery] = useState('')
const debouncedQuery = useDebounce(searchQuery, 500)

useEffect(() => {
  if (debouncedQuery) {
    performSearch(debouncedQuery)
  }
}, [debouncedQuery])
```

## パフォーマンス最適化

### メモ化

```typescript
// ✅ 高コストな計算にuseMemo
const sortedMarkets = useMemo(() => {
  return markets.sort((a, b) => b.volume - a.volume)
}, [markets])

// ✅ 子に渡す関数にuseCallback
const handleSearch = useCallback((query: string) => {
  setSearchQuery(query)
}, [])

// ✅ 純粋コンポーネントにReact.memo
export const MarketCard = React.memo<MarketCardProps>(({ market }) => {
  return (
    <div className="market-card">
      <h3>{market.name}</h3>
      <p>{market.description}</p>
    </div>
  )
})
```

### コード分割と遅延ロード

```typescript
import { lazy, Suspense } from 'react'

// ✅ 重いコンポーネントを遅延ロード
const HeavyChart = lazy(() => import('./HeavyChart'))
const ThreeJsBackground = lazy(() => import('./ThreeJsBackground'))

export function Dashboard() {
  return (
    <div>
      <Suspense fallback={<ChartSkeleton />}>
        <HeavyChart data={data} />
      </Suspense>

      <Suspense fallback={null}>
        <ThreeJsBackground />
      </Suspense>
    </div>
  )
}
```

## フォーム処理パターン

### バリデーション付き制御フォーム

```typescript
interface FormData {
  name: string
  description: string
  endDate: string
}

interface FormErrors {
  name?: string
  description?: string
  endDate?: string
}

export function CreateMarketForm() {
  const [formData, setFormData] = useState<FormData>({
    name: '',
    description: '',
    endDate: ''
  })

  const [errors, setErrors] = useState<FormErrors>({})

  const validate = (): boolean => {
    const newErrors: FormErrors = {}

    if (!formData.name.trim()) {
      newErrors.name = '名前は必須です'
    } else if (formData.name.length > 200) {
      newErrors.name = '名前は200文字以内である必要があります'
    }

    if (!formData.description.trim()) {
      newErrors.description = '説明は必須です'
    }

    if (!formData.endDate) {
      newErrors.endDate = '終了日は必須です'
    }

    setErrors(newErrors)
    return Object.keys(newErrors).length === 0
  }

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault()

    if (!validate()) return

    try {
      await createMarket(formData)
      // 成功処理
    } catch (error) {
      // エラー処理
    }
  }

  return (
    <form onSubmit={handleSubmit}>
      <input
        value={formData.name}
        onChange={e => setFormData(prev => ({ ...prev, name: e.target.value }))}
        placeholder="マーケット名"
      />
      {errors.name && <span className="error">{errors.name}</span>}

      {/* その他のフィールド */}

      <button type="submit">マーケットを作成</button>
    </form>
  )
}
```

## アクセシビリティパターン

### キーボードナビゲーション

```typescript
export function Dropdown({ options, onSelect }: DropdownProps) {
  const [isOpen, setIsOpen] = useState(false)
  const [activeIndex, setActiveIndex] = useState(0)

  const handleKeyDown = (e: React.KeyboardEvent) => {
    switch (e.key) {
      case 'ArrowDown':
        e.preventDefault()
        setActiveIndex(i => Math.min(i + 1, options.length - 1))
        break
      case 'ArrowUp':
        e.preventDefault()
        setActiveIndex(i => Math.max(i - 1, 0))
        break
      case 'Enter':
        e.preventDefault()
        onSelect(options[activeIndex])
        setIsOpen(false)
        break
      case 'Escape':
        setIsOpen(false)
        break
    }
  }

  return (
    <div
      role="combobox"
      aria-expanded={isOpen}
      aria-haspopup="listbox"
      onKeyDown={handleKeyDown}
    >
      {/* ドロップダウン実装 */}
    </div>
  )
}
```

**覚えておく**: 現代的なフロントエンドパターンは、保守可能でパフォーマンスの高いユーザーインターフェースを可能にします。プロジェクトの複雑さに合ったパターンを選択してください。
