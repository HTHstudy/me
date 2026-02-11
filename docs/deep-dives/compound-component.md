---  
layout: default  
title: 합성 컴포넌트  
parent: Deep Dives  
nav_order: 2  
permalink: /docs/deep-dives/compound-component  
---

# 합성 컴포넌트

> "혼자서는 동작하지 않는 컴포넌트를  
> 왜 따로 만들까"

---

## 합성 컴포넌트란 무엇인가

여러 컴포넌트가 하나의 기능을 이루되,  
각자 독립된 역할을 가지는 패턴이다.

부모가 상태와 로직을 소유하고,  
자식이 구조와 렌더링을 담당한다.  
부모와 자식 사이에는 **암묵적 계약**이 존재한다.

핵심은 하나로 만들 수도 있는 것을  
**의도적으로 나눈** 패턴이라는 점이다.

단순히 컴포넌트를 나열하는 것과는 다르다.  
합성 컴포넌트의 자식들은  
부모의 내부 상태를 공유한다.

```tsx
// 단순 나열 — 각 컴포넌트가 독립적
<Header />
<Content />
<Footer />

// 합성 컴포넌트 — 부모의 상태를 자식이 공유
<Select>
  <SelectTrigger />
  <SelectContent>
    <SelectItem value="a">A</SelectItem>
    <SelectItem value="b">B</SelectItem>
  </SelectContent>
</Select>
```

위 `Select`의 자식들은  
`Select` 바깥에서는 의미를 가지지 못한다.  
`SelectItem`이 클릭되면 `Select`의 상태가 바뀌고,  
`SelectTrigger`가 그 상태를 표시한다.

이 컴포넌트들은 서로를 전제한다.  
그것이 **암묵적 계약**이다.

---

## 왜 존재하는가 — 합성 컴포넌트가 해결하는 문제

Select 컴포넌트를 하나로 만든다고 가정해보자.

```tsx
interface SelectProps {
  value: string;
  onChange: (value: string) => void;
  items: Array<{ value: string; label: string }>;
  placeholder?: string;
  disabled?: boolean;
  renderItem?: (item: Item) => ReactNode;
  searchable?: boolean;
  onSearch?: (query: string) => void;
  groupBy?: (item: Item) => string;
  maxHeight?: number;
  loading?: boolean;
  emptyMessage?: string;
}

const Select = ({
  value,
  onChange,
  items,
  placeholder,
  disabled,
  renderItem,
  searchable,
  onSearch,
  groupBy,
  maxHeight,
  loading,
  emptyMessage,
}: SelectProps) => {
  const [isOpen, setIsOpen] = useState(false);
  const [searchQuery, setSearchQuery] = useState('');

  return (
    <div>
      <button onClick={() => setIsOpen(!isOpen)} disabled={disabled}>
        {value
          ? items.find(item => item.value === value)?.label
          : placeholder}
      </button>
      {isOpen && (
        <div style={{ maxHeight }}>
          {searchable && (
            <input
              value={searchQuery}
              onChange={e => {
                setSearchQuery(e.target.value);
                onSearch?.(e.target.value);
              }}
            />
          )}
          {loading && <div>로딩 중...</div>}
          {items.length === 0 && <div>{emptyMessage}</div>}
          {items.map(item => (
            <div key={item.value} onClick={() => onChange(item.value)}>
              {renderItem ? renderItem(item) : item.label}
            </div>
          ))}
        </div>
      )}
    </div>
  );
};
```

처음에는 `items`, `value`, `onChange`면 충분했다.  
그런데 요구사항이 추가된다.

- 검색이 가능해야 한다 → `searchable`, `onSearch`  
- 아이템을 그룹으로 묶어야 한다 → `groupBy`  
- 아이템 렌더링을 커스텀해야 한다 → `renderItem`  
- 로딩 상태를 보여줘야 한다 → `loading`, `emptyMessage`

props가 비대해진다.  
하지만 더 본질적인 문제는  
**구조를 바꿀 수 없다**는 점이다.

검색 입력을 목록 위가 아니라 트리거 옆에 놓고 싶다면?  
아이템 사이에 구분선을 넣고 싶다면?  
이 모든 변형에 대해 props를 추가해야 한다.

합성 컴포넌트로 해결하면 이렇게 된다.

```tsx
const Select = ({ children }: { children: ReactNode }) => {
  const [isOpen, setIsOpen] = useState(false);
  const [value, setValue] = useState<string | null>(null);

  return (
    <SelectContext.Provider value={{ isOpen, setIsOpen, value, setValue }}>
      <div className="select">{children}</div>
    </SelectContext.Provider>
  );
};

const SelectTrigger = ({ placeholder }: { placeholder?: string }) => {
  const { isOpen, setIsOpen, value } = useSelectContext();

  return (
    <button onClick={() => setIsOpen(!isOpen)}>
      {value ?? placeholder}
    </button>
  );
};

const SelectContent = ({ children }: { children: ReactNode }) => {
  const { isOpen } = useSelectContext();

  if (!isOpen) return null;

  return <div className="select-content">{children}</div>;
};

const SelectItem = ({ value, children }: { value: string; children: ReactNode }) => {
  const { setValue, setIsOpen } = useSelectContext();

  return (
    <div onClick={() => { setValue(value); setIsOpen(false); }}>
      {children}
    </div>
  );
};
```

사용하는 쪽에서 구조를 결정한다.

```tsx
// 기본 사용
<Select>
  <SelectTrigger placeholder="선택하세요" />
  <SelectContent>
    <SelectItem value="a">옵션 A</SelectItem>
    <SelectItem value="b">옵션 B</SelectItem>
  </SelectContent>
</Select>

// 검색 추가 — 컴포넌트 내부를 수정하지 않아도 된다
<Select>
  <SelectTrigger placeholder="선택하세요" />
  <SelectContent>
    <SearchInput />
    <SelectItem value="a">옵션 A</SelectItem>
    <SelectItem value="b">옵션 B</SelectItem>
  </SelectContent>
</Select>

// 그룹핑 — 역시 구조만 바꾸면 된다
<Select>
  <SelectTrigger placeholder="선택하세요" />
  <SelectContent>
    <SelectGroup label="과일">
      <SelectItem value="apple">사과</SelectItem>
      <SelectItem value="banana">바나나</SelectItem>
    </SelectGroup>
    <SelectGroup label="채소">
      <SelectItem value="carrot">당근</SelectItem>
    </SelectGroup>
  </SelectContent>
</Select>
```

단일 컴포넌트에서는 props로 표현해야 했던 것들이  
합성 컴포넌트에서는 **JSX 구조**로 표현된다.

새로운 요구사항이 추가되어도  
기존 컴포넌트의 인터페이스를 바꿀 필요가 없다.  
구조를 조합하는 방식으로 대응한다.

---

## 사용 사례

### Tabs — 가장 전형적인 합성 컴포넌트

Tabs는 합성 컴포넌트의 특성을 가장 잘 보여주는 사례다.

```tsx
const TabsContext = createContext<{
  activeTab: string;
  setActiveTab: (tab: string) => void;
} | null>(null);

const Tabs = ({ defaultTab, children }: { defaultTab: string; children: ReactNode }) => {
  const [activeTab, setActiveTab] = useState(defaultTab);

  return (
    <TabsContext.Provider value={{ activeTab, setActiveTab }}>
      <div className="tabs">{children}</div>
    </TabsContext.Provider>
  );
};

const TabList = ({ children }: { children: ReactNode }) => {
  return <div role="tablist">{children}</div>;
};

const Tab = ({ value, children }: { value: string; children: ReactNode }) => {
  const { activeTab, setActiveTab } = useTabsContext();

  return (
    <button
      role="tab"
      aria-selected={activeTab === value}
      onClick={() => setActiveTab(value)}
    >
      {children}
    </button>
  );
};

const TabPanel = ({ value, children }: { value: string; children: ReactNode }) => {
  const { activeTab } = useTabsContext();

  if (activeTab !== value) return null;

  return <div role="tabpanel">{children}</div>;
};
```

```tsx
<Tabs defaultTab="overview">
  <TabList>
    <Tab value="overview">개요</Tab>
    <Tab value="specs">사양</Tab>
    <Tab value="reviews">리뷰</Tab>
  </TabList>
  <TabPanel value="overview">개요 내용...</TabPanel>
  <TabPanel value="specs">사양 내용...</TabPanel>
  <TabPanel value="reviews">리뷰 내용...</TabPanel>
</Tabs>
```

여기서 주목할 점은 역할의 분리다.

- `Tabs` — 상태를 소유한다 (어떤 탭이 활성화되어 있는지)  
- `TabList` — 탭 버튼들의 레이아웃을 담당한다  
- `Tab` — 하나의 탭 버튼을 렌더링하고, 클릭 시 상태를 바꾼다  
- `TabPanel` — 활성 탭에 해당하는 내용을 렌더링한다

각 컴포넌트가 자신의 역할만 담당한다.  
이것을 하나의 컴포넌트로 만들었다면  
`tabs`, `renderTab`, `renderPanel`, `defaultTab`, `onChange`...  
props 기반의 설계가 되었을 것이다.

합성 컴포넌트에서는  
TabList와 TabPanel 사이에 다른 요소를 넣을 수도 있고,  
Tab의 렌더링을 자유롭게 커스텀할 수도 있다.  
**구조의 유연성**이 props 기반에서는 얻기 어려운 장점이다.

### Context 없이도 가능한 합성 컴포넌트

합성 컴포넌트라고 하면  
Context를 떠올리기 쉽다.

하지만 합성 컴포넌트의 본질은  
**"부모가 상태를 소유하고 자식이 그것을 공유한다"**이지,  
"Context를 쓴다"가 아니다.

간단한 경우에는 `children`과 `cloneElement`로도 구현할 수 있다.

```tsx
const RadioGroup = ({ value, onChange, children }: RadioGroupProps) => {
  return (
    <div role="radiogroup">
      {Children.map(children, child => {
        if (isValidElement(child)) {
          return cloneElement(child, {
            checked: child.props.value === value,
            onChange: () => onChange(child.props.value),
          });
        }
        return child;
      })}
    </div>
  );
};

const RadioItem = ({ value, checked, onChange, children }: RadioItemProps) => {
  return (
    <label>
      <input
        type="radio"
        value={value}
        checked={checked}
        onChange={onChange}
      />
      {children}
    </label>
  );
};
```

```tsx
<RadioGroup value={selected} onChange={setSelected}>
  <RadioItem value="small">소</RadioItem>
  <RadioItem value="medium">중</RadioItem>
  <RadioItem value="large">대</RadioItem>
</RadioGroup>
```

`RadioGroup`이 `cloneElement`로  
각 `RadioItem`에 `checked`와 `onChange`를 주입한다.  
Context 없이도 부모와 자식 사이의 암묵적 계약이 성립한다.

이 방식은 제약이 있다.  
직접 자식만 대상으로 동작하기 때문에  
중간에 다른 요소가 끼어들면 작동하지 않는다.

```tsx
// 이 구조에서 cloneElement는 div를 대상으로 동작한다
<RadioGroup value={selected} onChange={setSelected}>
  <div className="option-wrapper">
    <RadioItem value="small">소</RadioItem>
  </div>
</RadioGroup>
```

트리가 깊어지거나 구조가 유연해야 한다면  
Context 기반이 자연스럽다.  
하지만 직접 자식만 다루는 간단한 경우에는  
`cloneElement`로도 충분하다.

합성 컴포넌트의 핵심은 **구현 방식**이 아니라  
**부모가 상태를 소유하고 자식이 역할을 나누는 구조**다.  
Context는 그 구조를 실현하는 방법 중 하나일 뿐이다.

---

## 정리하며

합성 컴포넌트는  
하나의 기능을 여러 컴포넌트로 나누되,  
**암묵적 계약**으로 연결하는 패턴이다.

부모가 상태와 로직을 소유하고,  
자식이 구조와 렌더링을 담당한다.  
사용하는 쪽이 JSX 구조로 조합 방식을 결정할 수 있어서,  
props 기반의 단일 컴포넌트보다 구조의 유연성이 높다.

이 유연성에는 비용이 따른다.  
합성 컴포넌트를 사용하는 쪽은  
각 자식의 역할과 조합 규칙을 알아야 한다.  
암묵적 계약은 props처럼 타입으로 드러나지 않을 수 있다.

합성 컴포넌트가 내부 상태를 공유하는 방식으로  
Context가 자주 사용된다.  
[React Context의 역할](/me/docs/deep-dives/role-of-context)에서  
Context가 스코프를 가진 전달 도구로 쓰이는 사례를 다루었다.  
합성 컴포넌트는 그 대표적인 활용이다.

"어떤 컴포넌트를 합성 컴포넌트로 만들 것인가"는  
"이 컴포넌트를 나눠야 하는가"라는 더 넓은 결정의 일부다.  
[이 컴포넌트는 나눠야 하는가](/me/docs/decisions-over-patterns/should-component-be-split)에서  
나누는 것의 기준과 비용을 다룬다.
