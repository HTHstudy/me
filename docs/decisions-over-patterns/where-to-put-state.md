---
layout: default
title: 상태를 어디에 둘 것인가
parent: Decisions over Patterns
nav_order: 2
permalink: /docs/decisions-over-patterns/where-to-put-state
---

# 상태를 어디에 둘 것인가

> "이 상태는 여기 있어야 하는가, 저기 있어야 하는가  
> 그 선택은 무엇을 단순하게 만들고  
> 무엇을 복잡하게 만드는가"

## 들어가며 — 상태를 정의하는 순간

컴포넌트를 작성하다 보면  
자연스럽게 상태가 필요해진다.

```tsx
const [value, setValue] = useState('');
```

이 한 줄은 단순해 보인다.  
하지만 이 상태를 **어디에** 둘 것인지는  
단순하지 않은 결정이다.

```tsx
// 이 상태는 여기 있어야 하나?
const SearchInput = () => {
  const [query, setQuery] = useState('');
  
  return <input value={query} onChange={e => setQuery(e.target.value)} />;
};

// 아니면 부모에 있어야 하나?
const SearchPage = () => {
  const [query, setQuery] = useState('');
  
  return (
    <>
      <SearchInput value={query} onChange={setQuery} />
      <SearchResults query={query} />
    </>
  );
};
```

상태의 위치가 바뀌면  
컴포넌트의 역할이 바뀌고,  
데이터의 흐름이 바뀌며,  
변경의 영향 범위가 바뀐다.

이 문서는  
상태를 어디에 둘 것인지 결정할 때  
어떤 질문을 던져야 하는지,  
각 선택이 무엇을 의미하는지를  
다시 생각해보기 위해 쓰였다.

---

## 상태의 종류를 먼저 구분하기

상태를 어디에 둘지 결정하기 전에,  
**어떤 종류의 상태인지** 먼저 구분하면  
결정이 명확해지는 경우가 많다.

### UI 상태 (Ephemeral State)

```tsx
const [isOpen, setIsOpen] = useState(false);
const [inputValue, setInputValue] = useState('');
const [activeTab, setActiveTab] = useState(0);
```

일시적인 인터랙션 상태다.

- 모달 열림/닫힘
- 입력 필드 값
- 탭 선택
- 드롭다운 열림 상태

새로고침하면 사라져도 된다.  
대부분 **컴포넌트와 생명주기를 함께**한다.

### 도메인 상태 (Client State)

```tsx
const [cart, setCart] = useState([]);
const [selectedItems, setSelectedItems] = useState([]);
const [draftForm, setDraftForm] = useState({});
```

클라이언트가 소유하는 데이터다.

- 장바구니에 담긴 상품
- 선택한 항목들
- 작성 중인 폼 초안

사용자의 행동에 따라 변하고,  
여러 화면에서 공유될 수 있다.  
**클라이언트가 진실의 근원**이다.

### 서버 상태 (Server State)

```tsx
const { data: user } = useQuery({
  queryKey: ['user', userId],
  queryFn: () => fetchUser(userId),
});
```

서버가 소유하는 데이터다.

- 사용자 프로필
- 상품 목록
- 게시물 상세

클라이언트에서 변경해도  
결국 **서버와 동기화**가 필요하다.  
캐싱, 리페치, 동기화가 중요해진다.

### 구분이 주는 힌트

| 종류 | 진실의 근원 | 공유 범위 | 자연스러운 위치 |
|------|-------------|-----------|-----------------|
| UI 상태 | 컴포넌트 | 좁음 | 로컬 상태 |
| 도메인 상태 | 클라이언트 | 넓음 | 끌어올리기 / 전역 |
| 서버 상태 | 서버 | 캐시 공유 | 서버 상태 관리 |

이 구분이 절대적인 것은 아니다.  
하지만 **"이 데이터의 진실은 어디에 있는가?"**라는 질문은  
결정의 출발점이 될 수 있다.

---

## 선택지들

상태를 둘 수 있는 곳은 여러 가지다.  
각각은 서로 다른 특성을 가진다.

### 로컬 상태 (useState)

```tsx
const Counter = () => {
  const [count, setCount] = useState(0);
  
  return (
    <button onClick={() => setCount(c => c + 1)}>
      {count}
    </button>
  );
};
```

**언제?**  
- 이 컴포넌트만 이 상태를 안다
- 컴포넌트가 사라지면 상태도 사라져도 된다
- 다른 컴포넌트와 공유할 필요가 없다

**왜?**  
가장 단순하고, side-effect가 가장 작다.  
상태가 필요한 곳 근처에 두면 가독성이 높아진다.

하지만 다른 컴포넌트가 이 상태를 알아야 하는 순간,  
로컬 상태만으로는 부족해진다.

### 로컬 상태 (useReducer)

```tsx
type State = { step: number; values: Record<string, string> };
type Action =
  | { type: 'NEXT' }
  | { type: 'BACK' }
  | { type: 'CHANGE'; name: string; value: string };

function reducer(state: State, action: Action): State {
  switch (action.type) {
    case 'NEXT': return { ...state, step: state.step + 1 };
    case 'BACK': return { ...state, step: state.step - 1 };
    case 'CHANGE': return { ...state, values: { ...state.values, [action.name]: action.value } };
  }
}

const MultiStepForm = () => {
  const [state, dispatch] = useReducer(reducer, { step: 0, values: {} });
  // ...
};
```

**언제?**  
- 여러 액션이 상태를 변경한다
- 상태 전이에 규칙이 있다 (단계별 폼, 드래그앤드롭 등)
- useState가 여러 개 생기면서 서로 연관되어 있다

**왜?**  
업데이트 로직을 한 곳에 모아서  
예측 가능성과 테스트 용이성을 높인다.

### 상태 끌어올리기

```tsx
const Parent = () => {
  const [count, setCount] = useState(0);
  
  return (
    <>
      <Counter count={count} onIncrement={() => setCount(c => c + 1)} />
      <Display count={count} />
    </>
  );
};

const Counter = ({ count, onIncrement }) => {
  return <button onClick={onIncrement}>{count}</button>;
};

const Display = ({ count }) => {
  return <p>현재 값: {count}</p>;
};
```

상태를 공통 부모로 올렸다.

**언제?**  
- 두 개 이상의 컴포넌트가 같은 상태를 알아야 한다
- 그 컴포넌트들이 가까운 트리에 있다

**왜?**  
React의 단방향 데이터 흐름을 따르면서  
상태를 공유하는 가장 기본적인 방식이다.

하지만 컴포넌트 트리가 깊어지면  
props를 여러 단계 내려보내야 한다. (props drilling)

### Context

```tsx
const CountContext = createContext(null);

const App = () => {
  const [count, setCount] = useState(0);
  
  return (
    <CountContext.Provider value={{ count, setCount }}>
      <DeepNestedTree />
    </CountContext.Provider>
  );
};

const DeepChild = () => {
  const { count, setCount } = useContext(CountContext);
  
  return <button onClick={() => setCount(c => c + 1)}>{count}</button>;
};
```

상태를 Context에 넣었다.

**언제?**  
- 테마, 언어, 권한처럼 **자주 바뀌지 않는** 값을 공유할 때
- props drilling을 피하고 싶지만 전역 스토어까지는 필요 없을 때
- 특정 범위(Provider) 내에서만 공유하면 될 때

**왜?**  
깊이에 상관없이 접근 가능하고,  
props drilling이 사라진다.

하지만 Context의 값이 바뀌면  
그것을 구독하는 모든 컴포넌트가 리렌더링된다.

**side-effect가 중간 정도**다.  
자주 바뀌는 상태를 Context에 두면 성능 문제가 생긴다.

### 외부 상태 관리

```tsx
// Zustand 예시
const useStore = create((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
}));

const Counter = () => {
  const count = useStore((state) => state.count);
  const increment = useStore((state) => state.increment);
  
  return <button onClick={increment}>{count}</button>;
};
```

상태를 React 바깥에 두었다.

**언제?**  
- 여러 화면에서 공유되는 클라이언트 소유 데이터 (장바구니, 에디터 버퍼 등)
- 선택적 구독이 필요할 때 (자주 바뀌지만 일부만 구독)
- React 외부에서도 접근해야 할 때

**왜?**  
컴포넌트 트리와 독립적이고,  
선택적 구독으로 리렌더링을 최소화할 수 있다.

하지만 **side-effect가 가장 크다.**  
앱 전체에 영향을 미치므로 신중하게 관리해야 한다.  
전역 상태 변경은 어디서든 일어날 수 있고,  
추적이 어려워질 수 있다.

### 서버 상태

```tsx
// TanStack Query 예시
const UserProfile = ({ userId }) => {
  const { data: user, isLoading, error } = useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetchUser(userId),
  });
  
  if (isLoading) return <Loading />;
  if (error) return <Error />;
  
  return <div>{user.name}</div>;
};
```

상태가 서버에 있고,  
클라이언트는 그것의 캐시를 가진다.

**언제?**  
- 서버가 진실의 근원인 데이터 (목록, 상세, 프로필 등)
- 캐싱, 리페치, 동기화가 필요할 때
- 로딩/에러 상태를 자동으로 관리하고 싶을 때

**왜?**  
서버 데이터를 클라이언트 상태처럼 다루면  
동기화 문제가 생긴다.  
캐싱, 중복 요청 제거, stale-while-revalidate 등을  
직접 구현하지 않아도 된다.

서버 상태는 "클라이언트가 소유한 상태"가 아니라  
"서버 상태의 캐시"로 생각하면 결정이 명확해진다.

---

## 판단의 질문들

상태를 어디에 둘지 결정할 때  
던져볼 수 있는 질문들이 있다.

### 이 상태를 누가 알아야 하는가?

```tsx
// 이 컴포넌트만 알면 되는가?
const Dropdown = () => {
  const [isOpen, setIsOpen] = useState(false);
  // ...
};

// 부모도 알아야 하는가?
const Dropdown = ({ isOpen, onToggle }) => {
  // ...
};

// 멀리 떨어진 컴포넌트도 알아야 하는가?
const useDropdownState = create(/* ... */);
```

상태를 알아야 하는 범위가  
상태의 위치를 결정한다.

- 이 컴포넌트만 → 로컬 상태
- 부모와 형제들 → 끌어올리기
- 멀리 떨어진 곳들 → Context 또는 외부 상태

### 이 상태의 생명주기는 무엇인가?

```tsx
// 컴포넌트와 함께 사라져도 되는가?
const SearchInput = () => {
  const [query, setQuery] = useState('');
  // 다른 페이지로 갔다 오면 query는 사라진다
};

// 컴포넌트보다 오래 살아야 하는가?
const useSearchStore = create((set) => ({
  recentSearches: [],
  // 페이지를 이동해도 유지된다
}));
```

상태가 얼마나 오래 살아야 하는지에 따라  
위치가 달라진다.

- 컴포넌트와 함께 → 로컬 상태
- 컴포넌트보다 오래 → 상위 컴포넌트 또는 외부
- 세션 동안 유지 → 외부 상태 또는 sessionStorage
- 영구적 → 서버 또는 localStorage

### 이 상태가 변하면 무엇이 다시 렌더링되어야 하는가?

```tsx
// Context: 모든 구독자가 리렌더링
const ThemeContext = createContext('light');

const App = () => {
  const [theme, setTheme] = useState('light');
  // theme이 바뀌면 Provider 아래 모든 useContext(ThemeContext)가 리렌더링
  return (
    <ThemeContext.Provider value={theme}>
      {/* ... */}
    </ThemeContext.Provider>
  );
};

// 외부 상태: 선택적 구독
const useStore = create((set) => ({
  theme: 'light',
  user: null,
}));

const ThemeButton = () => {
  // theme만 구독, user가 바뀌어도 리렌더링 안 됨
  const theme = useStore((state) => state.theme);
};
```

상태 변경의 영향 범위를 고려해야 한다.

- 자주 바뀌고 많은 곳에 영향 → 선택적 구독이 가능한 방식
- 드물게 바뀌고 전체에 영향 → Context도 괜찮음

---

## 결정의 흐름

몇 가지 상황에서  
어떤 질문을 던지게 되는지 살펴본다.

### 폼 입력 상태

```tsx
const LoginForm = () => {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  
  const handleSubmit = () => {
    login(email, password);
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input value={email} onChange={e => setEmail(e.target.value)} />
      <input value={password} onChange={e => setPassword(e.target.value)} />
      <button type="submit">로그인</button>
    </form>
  );
};
```

폼 입력 상태는 보통:

- 이 폼 컴포넌트만 알면 된다
- 제출하면 사라져도 된다
- 다른 곳에서 접근할 필요가 없다

→ 로컬 상태가 자연스럽다.

하지만 만약:

- 여러 단계로 나뉜 폼이라면?
- 다른 컴포넌트가 폼 상태를 알아야 한다면?
- 페이지를 이동해도 입력값이 유지되어야 한다면?

→ 상태를 올리거나 외부로 빼는 것을 고려하게 된다.

### 모달/드롭다운 열림 상태

```tsx
// 선택 1: 로컬 상태
const Dropdown = () => {
  const [isOpen, setIsOpen] = useState(false);
  
  return (
    <div>
      <button onClick={() => setIsOpen(!isOpen)}>열기</button>
      {isOpen && <Menu />}
    </div>
  );
};

// 선택 2: 부모가 제어
const Dropdown = ({ isOpen, onToggle }) => {
  return (
    <div>
      <button onClick={onToggle}>열기</button>
      {isOpen && <Menu />}
    </div>
  );
};
```

**로컬 상태**를 선택하면:

- Dropdown이 스스로 열림/닫힘을 관리한다
- 부모는 Dropdown의 상태를 모른다
- 단순하지만, 외부에서 제어할 수 없다

**부모 제어**를 선택하면:

- 부모가 열림/닫힘을 결정한다
- 다른 동작과 연동할 수 있다 (예: 하나만 열리게)
- 하지만 Dropdown을 쓸 때마다 상태를 관리해야 한다

어떤 것이 맞다고 말하기 어렵다.  
**Dropdown이 독립적으로 동작해야 하는지**,  
**외부에서 제어해야 하는지**에 따라 달라진다.

### 서버에서 온 데이터

```tsx
// 선택 1: 로컬 상태로 관리
const UserProfile = ({ userId }) => {
  const [user, setUser] = useState(null);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    setIsLoading(true);
    fetchUser(userId)
      .then(setUser)
      .catch(setError)
      .finally(() => setIsLoading(false));
  }, [userId]);
  
  // ...
};

// 선택 2: 서버 상태 라이브러리 사용
const UserProfile = ({ userId }) => {
  const { data: user, isLoading, error } = useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetchUser(userId),
  });
  
  // ...
};
```

서버 데이터를 로컬 상태로 관리하면:

- 로딩, 에러, 캐싱을 직접 구현해야 한다
- 같은 데이터를 여러 곳에서 요청하면 중복된다
- 데이터 동기화를 직접 관리해야 한다

서버 상태 라이브러리를 사용하면:

- 캐싱, 재시도, 동기화가 처리된다
- 같은 쿼리키면 데이터가 공유된다
- 하지만 새로운 개념과 API를 학습해야 한다

서버 데이터는  
"클라이언트가 소유한 상태"가 아니라  
"서버 상태의 캐시"로 생각하면  
결정이 명확해지는 경우가 많다.

### 여러 페이지가 공유하는 상태

```tsx
// 선택 1: 최상위 컴포넌트에서 관리
const App = () => {
  const [user, setUser] = useState(null);
  
  return (
    <Routes>
      <Route path="/" element={<Home user={user} />} />
      <Route path="/profile" element={<Profile user={user} />} />
      <Route path="/settings" element={<Settings user={user} setUser={setUser} />} />
    </Routes>
  );
};

// 선택 2: Context
const UserContext = createContext(null);

const App = () => {
  const [user, setUser] = useState(null);
  
  return (
    <UserContext.Provider value={{ user, setUser }}>
      <Routes>{/* ... */}</Routes>
    </UserContext.Provider>
  );
};

// 선택 3: 외부 상태
const useUserStore = create((set) => ({
  user: null,
  setUser: (user) => set({ user }),
}));
```

여러 페이지가 공유하는 상태는  
어디에 두어도 "작동"은 한다.

하지만:

- **끌어올리기**: props를 계속 내려보내야 한다
- **Context**: Provider 구조가 필요하다
- **외부 상태**: React 바깥에 의존성이 생긴다

어떤 비용을 감수할 것인지가  
선택의 기준이 된다.

---

## 상태 위치와 복잡도

상태의 위치는  
컴포넌트의 복잡도에 직접적으로 영향을 준다.

### 상태가 너무 많은 곳에 모이면

```tsx
// 모든 상태가 App에 있다
const App = () => {
  const [user, setUser] = useState(null);
  const [theme, setTheme] = useState('light');
  const [notifications, setNotifications] = useState([]);
  const [cart, setCart] = useState([]);
  const [isMenuOpen, setIsMenuOpen] = useState(false);
  const [searchQuery, setSearchQuery] = useState('');
  // ... 더 많은 상태들
  
  return (
    // 모든 것을 props로 내려보내야 함
  );
};
```

한 곳에 상태가 모이면:

- 그 컴포넌트가 모든 것을 알아야 한다
- 변경의 이유가 여러 가지가 된다
- 어떤 상태가 어디에 쓰이는지 추적하기 어렵다

### 상태가 너무 흩어지면

```tsx
// 모든 컴포넌트가 자기 상태를 가진다
const Header = () => {
  const [user, setUser] = useState(null);
  // ...
};

const Sidebar = () => {
  const [user, setUser] = useState(null);
  // ...
};
```

같은 상태가 여러 곳에 있으면:

- 동기화 문제가 생긴다
- 어느 것이 진짜인지 모호해진다
- 불일치 상태가 가능해진다

### 적절한 위치

상태는  
**그것을 필요로 하는 컴포넌트들의 가장 가까운 공통 조상**에  
두는 것이 기본 원칙이다.

```tsx
// SearchInput과 SearchResults 둘 다 query가 필요하다
// → 둘의 공통 부모인 SearchPage에 둔다
const SearchPage = () => {
  const [query, setQuery] = useState('');
  
  return (
    <>
      <SearchInput value={query} onChange={setQuery} />
      <SearchResults query={query} />
    </>
  );
};
```

하지만 이것은 시작점일 뿐이다.

- 트리가 깊어지면 → Context나 외부 상태를 고려
- 상태가 자주 바뀌면 → 렌더링 범위를 고려
- 서버 데이터라면 → 서버 상태로 다루는 것을 고려

---

## 정리하며 — 상태의 위치는 설계의 선택이다

상태를 어디에 둘 것인지는  
단순한 기술적 결정이 아니다.

상태의 위치가 결정하는 것들:

- **책임의 경계**: 누가 이 상태를 소유하는가
- **데이터 흐름**: 상태가 어떻게 전파되는가
- **변경의 영향**: 상태가 바뀌면 무엇이 영향받는가
- **복잡도의 분포**: 어디가 복잡해지고 어디가 단순해지는가

### 결정할 때 던질 질문들

- 이 상태를 **누가** 알아야 하는가?
- 이 상태의 **생명주기**는 무엇인가?
- 이 상태가 바뀌면 **무엇이** 다시 렌더링되어야 하는가?
- 이 상태는 **클라이언트가 소유**하는가, **서버가 소유**하는가?

### 선택지별 특성

| 선택지 | 범위 | 생명주기 | 비용 |
|--------|------|----------|------|
| 로컬 상태 | 컴포넌트 내부 | 컴포넌트와 함께 | 단순, 공유 불가 |
| 끌어올리기 | 부모-자식 | 부모 컴포넌트와 함께 | props drilling |
| Context | Provider 하위 전체 | Provider와 함께 | 전체 리렌더링 |
| 외부 상태 | 전역 | 앱 생명주기 | 외부 의존성 |
| 서버 상태 | 캐시 | 서버와 동기화 | 학습 비용 |

어떤 선택이 "정답"이라고 말하기 어렵다.  
각 선택은 서로 다른 비용과 이점을 가진다.

중요한 것은  
**왜 이 위치를 선택했는지**를  
설명할 수 있는 것이다.

### Side-effect의 크기

상태의 범위가 넓어질수록  
변경의 영향도 커진다.

```
로컬 상태      →  side-effect 낮음 (컴포넌트 내부)
     ↓
Context       →  side-effect 중간 (Provider 하위)
     ↓
전역 상태      →  side-effect 높음 (앱 전체)
```

**기본은 로컬**에서 시작하고,  
**필요해지면 점진적으로 승격**하며,  
**전역화는 최후의 수단**으로 남겨두는 것이  
side-effect를 관리하는 방향이다.

### 판단의 출발점이 될 수 있는 규칙들

절대적인 규칙은 아니지만,  
반복적으로 유효한 패턴들이 있다.

**1. 서버에서 오는 데이터는 서버 상태로**

서버가 진실의 근원이라면,  
클라이언트 상태에 복제하지 않는다.  
캐싱과 동기화는 서버 상태 도구에 맡긴다.

**2. 화면 내부 로직은 로컬 상태가 기본**

한 컴포넌트나 가까운 트리에서만 필요하다면  
useState나 useReducer로 충분하다.

**3. 전역 스토어는 클라이언트 진실 + 다화면 공유일 때만**

장바구니, 에디터 버퍼처럼  
클라이언트가 소유하면서 여러 화면에서 접근해야 할 때.

**4. Context는 값 주입 용도로, 자주 바뀌는 상태는 피한다**

테마, 권한, 설정처럼 자주 바뀌지 않는 값 주입에 적합하다.  
자주 바뀌는 상태는 전역 스토어로 이전하고 선택적 구독을 사용한다.

> 상태의 위치를 결정하는 것은  
> 복잡도를 어디에 둘 것인지 결정하는 것이다.  
> 그 복잡도가 가장 자연스러운 곳에  
> 상태가 있어야 한다.
