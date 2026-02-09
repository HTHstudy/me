---
layout: default
title: 이 컴포넌트는 제어되어야 하는가
parent: Decisions over Patterns
nav_order: 4
permalink: /docs/decisions-over-patterns/should-component-be-controlled
---

# 이 컴포넌트는 제어되어야 하는가

> "이 컴포넌트의 상태는 누가 소유하는가  
> 그 경계는 무엇을 기준으로 정해지는가"

## 들어가며 — 나눈 컴포넌트를 어떻게 연결할 것인가

컴포넌트를 나눴다.  
각 조각은 자신의 책임 안에서 완결되어 있다.

그런데 새로운 질문이 생긴다.

```tsx
// 이전 문서에서 분리한 구조
const ProductPage = ({ productId }) => {
  return (
    <div>
      <ProductDetail productId={productId} />
      <PurchaseOptions variants={product.variants} onAddToCart={handleAddToCart} />
      <ProductReviews productId={productId} />
    </div>
  );
};
```

PurchaseOptions는 사용자가 옵션을 선택하고 수량을 정하는 컴포넌트다.  
이 컴포넌트가 선택된 옵션을 자기 안에서 관리하면 되는가?  
아니면 ProductPage가 알아야 하는가?

만약 ProductDetail에도 선택된 옵션을 표시해야 한다면?  
PurchaseOptions 안에 있는 상태를 밖에서 어떻게 알 수 있는가?

```tsx
// PurchaseOptions가 자체적으로 상태를 관리한다면
const PurchaseOptions = ({ variants, onAddToCart }) => {
  const [selectedVariant, setSelectedVariant] = useState(null);
  const [quantity, setQuantity] = useState(1);
  // ...
};

// ProductDetail은 선택된 옵션을 알 수 없다
const ProductDetail = ({ product }) => {
  // selectedVariant가 여기 없다
  // 어떤 옵션이 선택되었는지 표시할 수 없다
};
```

이 상황에서 자연스럽게  
이런 질문이 떠오른다.

**"이 컴포넌트는 제어되어야 하는가?"**

이 문서는  
컴포넌트를 제어할지 말지 결정할 때  
어떤 기준으로 판단하는지를 다룬다.

---

## 제어와 비제어

본격적인 선택지를 보기 전에  
"제어"가 무엇을 의미하는지 짚어본다.

```tsx
// 비제어: 컴포넌트가 자신의 상태를 소유한다
const Dropdown = ({ items, onSelect }) => {
  const [isOpen, setIsOpen] = useState(false);
  const [selected, setSelected] = useState(null);

  const handleSelect = (item) => {
    setSelected(item);
    setIsOpen(false);
    onSelect?.(item);  // 부모에게 알려주기만 한다
  };

  return (/* ... */);
};

// 사용하는 쪽
<Dropdown items={options} onSelect={handleOptionChange} />
```

```tsx
// 제어: 부모가 상태를 소유하고, 컴포넌트는 전달받는다
const Dropdown = ({ items, isOpen, selected, onOpenChange, onSelect }) => {
  return (/* ... */);
};

// 사용하는 쪽
const [isOpen, setIsOpen] = useState(false);
const [selected, setSelected] = useState(null);

<Dropdown
  isOpen={isOpen}
  selected={selected}
  onOpenChange={setIsOpen}
  onSelect={setSelected}
  items={options}
/>
```

비제어 컴포넌트는 자기 안에서 동작이 완결된다.  
제어 컴포넌트는 부모가 상태를 소유하고, 컴포넌트는 그것을 반영한다.

이 차이는 단순히 "props가 많은가 적은가"가 아니다.  
**상태의 소유권이 어디에 있는가**의 문제다.

---

## 선택지들

### 비제어 — 컴포넌트가 자신의 상태를 관리한다

```tsx
const DatePicker = ({ onSelect }) => {
  const [isOpen, setIsOpen] = useState(false);
  const [selectedDate, setSelectedDate] = useState(null);
  const [viewMonth, setViewMonth] = useState(new Date());

  const handleDateClick = (date) => {
    setSelectedDate(date);
    setIsOpen(false);
    onSelect?.(date);
  };

  return (
    <div>
      <button onClick={() => setIsOpen(!isOpen)}>
        {selectedDate ? format(selectedDate) : '날짜 선택'}
      </button>
      {isOpen && (
        <Calendar
          month={viewMonth}
          onMonthChange={setViewMonth}
          onDateClick={handleDateClick}
          selected={selectedDate}
        />
      )}
    </div>
  );
};

// 사용하는 쪽
const OrderForm = () => {
  const handleDateSelect = (date) => {
    console.log('선택된 날짜:', date);
  };

  return <DatePicker onSelect={handleDateSelect} />;
};
```

사용하는 쪽의 코드가 단순하다.  
DatePicker를 놓고 결과만 받으면 된다.

열림/닫힘, 월 이동, 선택 상태 —  
이런 내부 동작은 DatePicker가 알아서 처리한다.

**언제?**

- 컴포넌트의 동작이 자체적으로 완결될 때
- 부모가 중간 상태(열림/닫힘, 월 이동 등)를 알 필요가 없을 때
- 사용하는 쪽을 단순하게 유지하고 싶을 때

**왜?**

- 사용하는 쪽의 코드가 간결하다
- 컴포넌트 내부 동작이 캡슐화된다
- 상태 관리 로직이 한곳에 모여 있어 이해하기 쉽다

**비용**

- 부모가 컴포넌트의 상태를 알거나 바꿀 수 없다
- 외부에서 "3월로 이동해라", "이 날짜를 선택해라" 같은 제어가 어렵다
- 여러 컴포넌트 간에 상태를 동기화하기 어렵다

### 제어 — 부모가 상태를 소유한다

```tsx
const DatePicker = ({
  isOpen,
  selectedDate,
  viewMonth,
  onOpenChange,
  onDateSelect,
  onMonthChange,
}) => {
  return (
    <div>
      <button onClick={() => onOpenChange(!isOpen)}>
        {selectedDate ? format(selectedDate) : '날짜 선택'}
      </button>
      {isOpen && (
        <Calendar
          month={viewMonth}
          onMonthChange={onMonthChange}
          onDateClick={onDateSelect}
          selected={selectedDate}
        />
      )}
    </div>
  );
};

// 사용하는 쪽
const BookingForm = () => {
  const [isOpen, setIsOpen] = useState(false);
  const [selectedDate, setSelectedDate] = useState(null);
  const [viewMonth, setViewMonth] = useState(new Date());

  // 부모가 동작을 제어할 수 있다
  const handleDateSelect = (date) => {
    if (isWeekend(date)) return;  // 주말은 선택 불가
    setSelectedDate(date);
    setIsOpen(false);
  };

  return (
    <div>
      <DatePicker
        isOpen={isOpen}
        selectedDate={selectedDate}
        viewMonth={viewMonth}
        onOpenChange={setIsOpen}
        onDateSelect={handleDateSelect}
        onMonthChange={setViewMonth}
      />
      {selectedDate && <p>선택: {format(selectedDate)}</p>}
    </div>
  );
};
```

사용하는 쪽이 모든 동작을 결정한다.  
주말 선택을 막거나, 특정 날짜로 이동하거나,  
다른 컴포넌트와 선택 상태를 공유할 수 있다.

**언제?**

- 부모가 컴포넌트의 동작을 제어해야 할 때
- 여러 컴포넌트가 같은 상태를 공유할 때
- 선택에 대한 유효성 검사나 조건이 있을 때

**왜?**

- 동작을 사용하는 쪽에서 완전히 제어할 수 있다
- 상태가 부모에 있으므로 다른 컴포넌트와 공유가 쉽다
- 예측 가능하다 — 전달한 값이 그대로 반영된다

**비용**

- 사용하는 쪽의 코드가 길어진다
- 상태와 핸들러를 모두 직접 관리해야 한다
- 단순한 사용에도 많은 설정이 필요하다

### 혼합 — 일부만 제어한다

```tsx
const DatePicker = ({ selectedDate, onDateSelect, onOpenChange }) => {
  // 열림/닫힘은 내부에서 관리
  const [isOpen, setIsOpen] = useState(false);
  // 월 이동도 내부에서 관리
  const [viewMonth, setViewMonth] = useState(new Date());

  // 선택된 날짜는 부모가 소유
  const handleDateClick = (date) => {
    onDateSelect(date);
    setIsOpen(false);
    onOpenChange?.(false);
  };

  const handleToggle = () => {
    setIsOpen(!isOpen);
    onOpenChange?.(!isOpen);
  };

  return (
    <div>
      <button onClick={handleToggle}>
        {selectedDate ? format(selectedDate) : '날짜 선택'}
      </button>
      {isOpen && (
        <Calendar
          month={viewMonth}
          onMonthChange={setViewMonth}
          onDateClick={handleDateClick}
          selected={selectedDate}
        />
      )}
    </div>
  );
};

// 사용하는 쪽
const BookingForm = () => {
  const [selectedDate, setSelectedDate] = useState(null);

  return (
    <div>
      <DatePicker
        selectedDate={selectedDate}
        onDateSelect={setSelectedDate}
      />
      {selectedDate && <p>선택: {format(selectedDate)}</p>}
    </div>
  );
};
```

핵심 상태(선택된 날짜)는 부모가 소유하고,  
부수적인 상태(열림/닫힘, 월 이동)는 컴포넌트가 관리한다.

**언제?**

- 핵심 값은 부모가 알아야 하지만, 모든 상태를 제어할 필요는 없을 때
- UI 동작(열림/닫힘, 스크롤 위치 등)은 컴포넌트 내부 관심사일 때
- 단순한 사용과 유연한 제어를 동시에 지원하고 싶을 때

**왜?**

- 핵심 상태는 부모가 제어하면서도 사용이 간결하다
- UI 동작은 컴포넌트가 알아서 처리한다
- 대부분의 사용처에서 적은 props로 충분하다

**비용**

- 어떤 상태가 제어되고 어떤 상태가 내부인지 명확해야 한다
- API 설계가 복잡해질 수 있다
- 제어 범위를 나중에 바꾸면 인터페이스가 변경된다

---

## 판단의 질문들

컴포넌트를 제어할지 결정할 때  
던져볼 수 있는 질문들이 있다.

### 이 상태를 부모가 알아야 하는가?

```tsx
// 부모가 알아야 하는 상태
const ProductPage = () => {
  // selectedVariant를 ProductDetail에도, PurchaseSection에도 전달해야 한다
  // → 부모가 알아야 한다 → 제어
  const [selectedVariant, setSelectedVariant] = useState(null);

  return (
    <>
      <ProductDetail
        product={product}
        highlightedVariant={selectedVariant}
      />
      <VariantSelector
        variants={product.variants}
        selected={selectedVariant}
        onSelect={setSelectedVariant}
      />
    </>
  );
};
```

```tsx
// 부모가 알 필요 없는 상태
const SearchResults = () => {
  return (
    <div>
      {/* Tooltip의 열림/닫힘을 부모가 알 필요 없다 */}
      <ResultItem result={result}>
        <Tooltip content="상세 정보">
          <InfoIcon />
        </Tooltip>
      </ResultItem>
    </div>
  );
};
```

Tooltip이 열려 있는지 닫혀 있는지는  
SearchResults가 알 필요가 없다.  
Tooltip은 비제어로 충분하다.

하지만 selectedVariant는  
여러 컴포넌트가 함께 사용한다.  
부모가 소유하는 것이 자연스럽다.

### 이 상태가 형제 컴포넌트에 영향을 주는가?

```tsx
// 형제에게 영향을 주는 상태
const FilterableList = () => {
  // filter가 바뀌면 ResultList도 바뀌어야 한다
  // → 형제 간 공유 → 부모가 소유 → FilterPanel은 제어
  const [filter, setFilter] = useState({ category: 'all', sort: 'recent' });

  return (
    <div>
      <FilterPanel filter={filter} onChange={setFilter} />
      <ResultList filter={filter} />
    </div>
  );
};
```

```tsx
// 형제에게 영향을 주지 않는 상태
const Dashboard = () => {
  return (
    <div>
      {/* 각 차트의 확대/축소 상태는 서로 영향을 주지 않는다 */}
      <SalesChart data={salesData} />
      <TrafficChart data={trafficData} />
      <ConversionChart data={conversionData} />
    </div>
  );
};
```

SalesChart가 확대되어도  
TrafficChart에는 영향이 없다.  
각 차트는 자신의 확대 상태를 스스로 관리하면 된다.

하지만 FilterPanel의 필터가 바뀌면  
ResultList도 바뀌어야 한다.  
이 상태는 부모에 있는 것이 자연스럽고,  
FilterPanel은 제어되는 것이 자연스럽다.

### 사용하는 쪽에서 동작을 바꿔야 하는가?

```tsx
// 동작을 바꿔야 하는 경우
const BookingForm = () => {
  const [date, setDate] = useState(null);

  const handleDateSelect = (newDate) => {
    // 과거 날짜는 선택 불가
    if (isBefore(newDate, new Date())) return;
    // 이미 예약된 날짜는 선택 불가
    if (bookedDates.includes(newDate)) return;
    setDate(newDate);
  };

  // DatePicker가 제어되어야 이런 로직이 가능하다
  return <DatePicker selected={date} onSelect={handleDateSelect} />;
};
```

```tsx
// 동작을 바꿀 필요 없는 경우
const ProfilePage = () => {
  return (
    <div>
      {/* 아코디언의 열림/닫힘에 특별한 조건이 없다 */}
      <Accordion title="기본 정보">
        <BasicInfo user={user} />
      </Accordion>
      <Accordion title="활동 내역">
        <ActivityLog userId={user.id} />
      </Accordion>
    </div>
  );
};
```

아코디언을 열고 닫는 데  
특별한 조건이 없다면  
비제어로 충분하다.

하지만 DatePicker에서 특정 날짜를 거부해야 한다면  
선택 동작을 부모가 제어하는 것이 자연스럽다.

### 이 컴포넌트의 사용처마다 동작이 다른가?

```tsx
// 사용처마다 동작이 다른 경우

// A 페이지: 단일 선택
<TagSelector selected={selectedTag} onSelect={setSelectedTag} />

// B 페이지: 다중 선택
<TagSelector selected={selectedTags} onSelect={setSelectedTags} />

// C 페이지: 선택 + 새 태그 생성
<TagSelector
  selected={selectedTags}
  onSelect={setSelectedTags}
  onCreateTag={handleCreate}
/>
```

사용처마다 동작이 달라야 한다면  
컴포넌트는 동작을 결정하지 않고  
사용하는 쪽에 맡기는 것이 자연스럽다.

```tsx
// 사용처에 관계없이 동작이 같은 경우
// 어디에서 쓰든 같은 방식으로 동작한다
<PasswordStrengthMeter password={password} />
<LoadingSpinner />
<Avatar src={user.avatarUrl} name={user.name} />
```

모든 곳에서 같은 방식으로 동작하는 컴포넌트는  
내부에서 동작을 결정해도 문제가 없다.

---

## 결정의 흐름

몇 가지 상황에서  
어떤 질문을 던지게 되는지 살펴본다.

### 모달 컴포넌트

```tsx
// 상황: 삭제 확인 모달
const UserManagement = () => {
  const [showDeleteModal, setShowDeleteModal] = useState(false);
  const [targetUser, setTargetUser] = useState(null);

  const handleDeleteClick = (user) => {
    setTargetUser(user);
    setShowDeleteModal(true);
  };

  const handleConfirm = async () => {
    await deleteUser(targetUser.id);
    setShowDeleteModal(false);
    setTargetUser(null);
  };

  return (
    <div>
      <UserList users={users} onDeleteClick={handleDeleteClick} />
      <DeleteConfirmModal
        isOpen={showDeleteModal}
        userName={targetUser?.name}
        onConfirm={handleConfirm}
        onClose={() => setShowDeleteModal(false)}
      />
    </div>
  );
};
```

**판단 과정**

1. 부모가 이 상태를 알아야 하는가?  
   → 부모가 "삭제 버튼을 눌렀을 때" 모달을 열어야 한다.  
   → 삭제 완료 후 모달을 닫아야 한다.  
   **부모가 열림/닫힘을 알아야 한다.**

2. 동작을 바꿔야 하는가?  
   → API 호출이 성공해야 닫히는 등,  
   닫히는 조건을 부모가 결정한다.  
   **동작을 제어해야 한다.**

**결론: 제어 컴포넌트**

모달은 대부분 제어된다.  
열리는 시점과 닫히는 시점을  
부모가 결정해야 하는 경우가 많기 때문이다.

### 아코디언 컴포넌트

```tsx
// 상황: FAQ 페이지의 아코디언
const FAQPage = () => {
  return (
    <div>
      <Accordion title="배송은 얼마나 걸리나요?">
        <p>보통 2-3일 소요됩니다.</p>
      </Accordion>
      <Accordion title="반품은 어떻게 하나요?">
        <p>마이페이지에서 신청 가능합니다.</p>
      </Accordion>
      <Accordion title="결제 수단은 무엇이 있나요?">
        <p>신용카드, 계좌이체, 간편결제를 지원합니다.</p>
      </Accordion>
    </div>
  );
};
```

**판단 과정**

1. 부모가 열림/닫힘 상태를 알아야 하는가?  
   → FAQ 페이지는 어떤 항목이 열려 있는지 알 필요 없다.  
   **알 필요 없다.**

2. 형제에게 영향을 주는가?  
   → 하나를 열어도 다른 것에 영향이 없다.  
   **영향 없다.**

**결론: 비제어 컴포넌트**

```tsx
const Accordion = ({ title, children }) => {
  const [isOpen, setIsOpen] = useState(false);

  return (
    <div>
      <button onClick={() => setIsOpen(!isOpen)}>{title}</button>
      {isOpen && <div>{children}</div>}
    </div>
  );
};
```

**만약 "하나만 열리게" 해야 한다면?**

```tsx
// 하나만 열리게 하려면 부모가 상태를 소유해야 한다
const FAQPage = () => {
  const [openIndex, setOpenIndex] = useState(null);

  return (
    <div>
      {faqs.map((faq, index) => (
        <Accordion
          key={index}
          title={faq.question}
          isOpen={openIndex === index}
          onToggle={() => setOpenIndex(openIndex === index ? null : index)}
        >
          <p>{faq.answer}</p>
        </Accordion>
      ))}
    </div>
  );
};
```

같은 컴포넌트가  
요구사항에 따라 제어될 수도, 비제어일 수도 있다.

"하나만 열리게"라는 요구사항이 생기면  
형제 간 상태 조율이 필요해지고,  
자연스럽게 제어 컴포넌트로 전환된다.

### 검색 입력 컴포넌트

```tsx
// 상황: 검색 기능이 있는 페이지
const SearchPage = () => {
  // 검색어를 URL과 동기화해야 한다
  // 검색어가 바뀌면 결과 목록도 바뀌어야 한다
};
```

**판단 과정**

1. 부모가 이 상태를 알아야 하는가?  
   → 검색어가 바뀌면 결과를 다시 가져와야 한다.  
   → URL에도 반영해야 한다.  
   **부모가 알아야 한다.**

2. 형제에 영향을 주는가?  
   → 검색어 → 결과 목록, URL 모두 영향.  
   **영향을 준다.**

**결론: 제어 컴포넌트**

```tsx
const SearchPage = () => {
  const [query, setQuery] = useSearchParams('q');
  const results = useSearchResults(query);

  return (
    <div>
      <SearchInput value={query} onChange={setQuery} />
      <ResultList results={results} />
    </div>
  );
};
```

검색어는 URL, 결과 목록, 검색 입력 세 곳에서 사용된다.  
이 상태를 SearchInput이 소유하면  
나머지 두 곳이 접근할 수 없다.

### 탭 컴포넌트

```tsx
// 상황 A: 단순한 탭
const SettingsPage = () => {
  return (
    <Tabs>
      <Tab label="일반"><GeneralSettings /></Tab>
      <Tab label="알림"><NotificationSettings /></Tab>
    </Tabs>
  );
};
```

**판단: 비제어로 충분**

어떤 탭이 선택되어 있는지  
SettingsPage가 알 필요가 없다.

```tsx
// 상황 B: URL과 동기화되는 탭
const DashboardPage = () => {
  const [activeTab, setActiveTab] = useSearchParams('tab');

  return (
    <Tabs activeTab={activeTab} onTabChange={setActiveTab}>
      <Tab id="overview" label="개요"><Overview /></Tab>
      <Tab id="analytics" label="분석"><Analytics /></Tab>
    </Tabs>
  );
};
```

**판단: 제어 필요**

탭이 URL과 동기화되어야 하므로  
부모가 상태를 소유하는 것이 자연스럽다.

같은 Tabs 컴포넌트가  
사용처에 따라 제어될 수도, 비제어일 수도 있다.

---

## 제어의 비용

### 제어는 유연하지만 무겁다

```tsx
// 비제어: 세 줄로 충분
<Dropdown items={options} onSelect={handleSelect} />

// 제어: 상태, 핸들러, props가 모두 필요
const [isOpen, setIsOpen] = useState(false);
const [selected, setSelected] = useState(null);

<Dropdown
  isOpen={isOpen}
  selected={selected}
  onOpenChange={setIsOpen}
  onSelect={setSelected}
  items={options}
/>
```

제어가 항상 좋은 것은 아니다.  
유연함에는 비용이 따른다.

필요하지 않은 제어는  
사용하는 쪽의 코드를 복잡하게 만들 뿐이다.

### 나중에 제어가 필요해지면?

비제어로 시작했는데  
나중에 부모가 상태를 알아야 하는 상황이 생길 수 있다.

이때 선택지가 두 가지 있다.

**1. 제어 컴포넌트로 전환한다**

기존 비제어 사용처를 모두 변경한다.  
단순하지만, 사용처가 많으면 비용이 크다.

**2. 비제어 기본값 + 선택적 제어를 지원한다**

```tsx
const Accordion = ({
  isOpen: controlledIsOpen,
  onToggle,
  defaultOpen = false,
  title,
  children,
}) => {
  const [internalIsOpen, setInternalIsOpen] = useState(defaultOpen);

  const isControlled = controlledIsOpen !== undefined;
  const isOpen = isControlled ? controlledIsOpen : internalIsOpen;

  const handleToggle = () => {
    if (isControlled) {
      onToggle?.();
    } else {
      setInternalIsOpen(prev => !prev);
    }
  };

  return (
    <div>
      <button onClick={handleToggle}>{title}</button>
      {isOpen && <div>{children}</div>}
    </div>
  );
};
```

기존 비제어 사용처를 깨뜨리지 않으면서  
제어가 필요한 곳에서는 제어할 수 있다.

하지만 이 패턴에도 비용이 있다.  
컴포넌트 내부가 복잡해지고,  
"이 컴포넌트가 지금 제어되고 있는가?"를  
런타임에 판단해야 한다.

어느 쪽이 적합한지는  
사용처의 수와 변경 비용에 따라 다르다.

---

## 정리하며 — 제어는 소유권의 문제다

컴포넌트를 제어할지 결정하는 것은  
"상태의 소유권을 누가 가질 것인가"를 정하는 것이다.

### 선택지별 특성

| 선택지 | 언제 | 비용 |
|--------|------|------|
| 비제어 | 동작이 자체 완결될 때 | 외부에서 제어 불가 |
| 제어 | 부모가 상태를 알거나 제어해야 할 때 | 사용하는 쪽이 복잡해짐 |
| 혼합 | 핵심 값만 제어, 나머지는 내부 | API 경계 설계가 필요함 |

### 판단의 출발점이 될 수 있는 규칙들

절대적인 규칙은 아니지만,  
반복적으로 유효한 패턴들이 있다.

**1. 부모가 알 필요 없는 상태는 컴포넌트가 소유한다**

Tooltip의 열림/닫힘, 내부 애니메이션 상태,  
스크롤 위치 같은 것은  
대부분 컴포넌트 내부 관심사다.

**2. 형제 간에 공유되는 상태는 부모가 소유한다**

한 컴포넌트의 상태가  
형제 컴포넌트에 영향을 줘야 한다면  
그 상태는 부모에 있는 것이 자연스럽다.  
자연스럽게 제어 컴포넌트가 된다.

**3. 동작에 조건이 필요하면 제어를 고려한다**

"이 날짜는 선택 불가", "이미 열린 탭은 닫을 수 없음" 같은  
조건이 있다면  
사용하는 쪽이 동작을 제어하는 것이 자연스럽다.

**4. 확신이 없으면 비제어로 시작할 수 있다**

비제어에서 제어로 전환하는 것은  
설계 변경이지만 불가능하지는 않다.  
처음부터 모든 것을 제어하면  
사용하는 쪽이 불필요하게 복잡해진다.

> 컴포넌트를 제어한다는 것은  
> 유연함을 얻는 것이지만  
> 동시에 책임을 가져오는 것이다.  
> 그 책임이 사용하는 쪽에 있어야 할 때  
> 제어는 의미가 있다.
