---  
layout: default  
title: React Context의 역할  
parent: Deep Dives  
nav_order: 1  
permalink: /docs/deep-dives/role-of-context
---

# React Context의 역할

> "Context는 상태를 관리하는 도구인가,
> 값을 전달하는 도구인가"

---

## Context란 무엇인가

Context는 React의 **내장 의존성 주입(Dependency Injection) 도구**다.

부모 컴포넌트가 제공한 값을  
트리의 깊이와 상관없이  
하위 컴포넌트에서 직접 접근할 수 있게 해준다.

```tsx
const ThemeContext = createContext<'light' | 'dark'>('light');

const App = () => {
  return (
    <ThemeContext.Provider value="dark">
      <Page />
    </ThemeContext.Provider>
  );
};

const Button = () => {
  const theme = useContext(ThemeContext);

  return <button className={theme}>확인</button>;
};
```

세 가지 요소로 구성된다.

- `createContext` — 전달할 값의 통로를 만든다
- `Provider` — 값을 주입한다
- `useContext` — 주입된 값을 꺼낸다

여기서 주목할 점이 있다.  
Context 자체는 **상태를 보유하지 않는다**.

위 예시에서 `"dark"`라는 값은  
Context가 만든 것이 아니라  
Provider에 전달된 것일 뿐이다.

Context는 값을 만들거나 관리하는 도구가 아니다.  
**값을 전달하는 통로**다.

흔히 Context를 "전역 상태 관리 도구"로 부르는 경우가 있다.  
이것은 오해에 가깝다.

```tsx
const AuthContext = createContext<AuthState | null>(null);

const AuthProvider = ({ children }: { children: ReactNode }) => {
  const [user, setUser] = useState<User | null>(null);

  return (
    <AuthContext.Provider value={{ user, setUser }}>
      {children}
    </AuthContext.Provider>
  );
};
```

이 패턴을 보면  
Context가 상태를 관리하는 것처럼 보인다.

하지만 상태를 관리하는 것은 `useState`다.  
Context는 그 상태를 하위 트리에 **전달**할 뿐이다.

`useState`를 걷어내면  
Context에는 전달할 값 자체가 없다.

---

## Context가 해결하는 문제

React에서 데이터는 위에서 아래로 흐른다.  
부모가 자식에게 props로 값을 넘긴다.

이 방식은 명확하다.  
하지만 트리가 깊어지면 문제가 생긴다.

```tsx
const App = () => {
  const [user, setUser] = useState<User>(currentUser);

  return <Layout user={user} />;
};

const Layout = ({ user }: { user: User }) => {
  return (
    <div>
      <Header />
      <Sidebar user={user} />
      <Main />
    </div>
  );
};

const Sidebar = ({ user }: { user: User }) => {
  return (
    <nav>
      <Menu />
      <UserAvatar user={user} />
    </nav>
  );
};

const UserAvatar = ({ user }: { user: User }) => {
  return <img src={user.avatarUrl} alt={user.name} />;
};
```

`user`를 사용하는 컴포넌트는 `UserAvatar` 하나다.  
하지만 `Layout`과 `Sidebar`도 `user`를 props로 받아야 한다.  
자신이 쓰지 않는 데이터를 전달하기 위해서다.

이것이 **prop drilling**이다.

중간 컴포넌트들이  
자신과 무관한 데이터에 결합된다.  
`user`의 타입이 바뀌면  
`Layout`과 `Sidebar`의 인터페이스도 함께 바뀌어야 한다.

Context는 이 문제를 해결한다.

```tsx
const UserContext = createContext<User | null>(null);

const App = () => {
  const [user, setUser] = useState<User>(currentUser);

  return (
    <UserContext.Provider value={user}>
      <Layout />
    </UserContext.Provider>
  );
};

const Layout = () => {
  return (
    <div>
      <Header />
      <Sidebar />
      <Main />
    </div>
  );
};

const Sidebar = () => {
  return (
    <nav>
      <Menu />
      <UserAvatar />
    </nav>
  );
};

const UserAvatar = () => {
  const user = useContext(UserContext);

  return <img src={user?.avatarUrl} alt={user?.name} />;
};
```

`Layout`과 `Sidebar`는 더 이상 `user`를 모른다.  
`UserAvatar`만 Context에서 직접 값을 꺼낸다.

중간 컴포넌트의 불필요한 결합이 사라졌다.

다만 알아야 할 동작이 있다.  
**Provider의 value가 바뀌면  
그것을 구독하는 모든 컴포넌트가 리렌더링된다.**

이것은 Context의 설계다.  
선택적 구독(특정 값만 골라서 구독)은 지원하지 않는다.

자주 바뀌지 않는 값 — 테마, 언어, 인증 정보 — 에 대해서는  
이 비용이 무시할 수 있을 정도로 작다.

하지만 매 키 입력마다 바뀌는 값이나  
애니메이션 프레임마다 바뀌는 값을 Context에 넣으면  
구독하는 모든 컴포넌트가 매번 리렌더링된다.  
이때는 Context가 아닌 다른 도구가 적합하다.

---

## 사용 사례

### 테마 주입

가장 전형적인 사용 사례다.

```tsx
const ThemeContext = createContext<Theme>(defaultTheme);

const App = () => {
  const [theme, setTheme] = useState<Theme>(defaultTheme);

  return (
    <ThemeContext.Provider value={theme}>
      <ThemeToggle onToggle={() => setTheme(prev => toggle(prev))} />
      <Dashboard />
    </ThemeContext.Provider>
  );
};

const Card = () => {
  const theme = useContext(ThemeContext);

  return <div style={{ background: theme.surface }}>...</div>;
};
```

테마는 사용자가 명시적으로 전환할 때만 바뀐다.  
변경 빈도가 낮고, 접근이 필요한 컴포넌트는 트리 전역에 퍼져 있다.  
Context가 자연스럽게 맞는 상황이다.

### 인증 상태 전달

```tsx
const AuthContext = createContext<{ user: User | null; logout: () => void } | null>(null);

const ProtectedPage = () => {
  const auth = useContext(AuthContext);

  if (!auth?.user) return <Redirect to="/login" />;

  return <Dashboard user={auth.user} />;
};
```

인증 상태는 로그인과 로그아웃 시에만 바뀐다.  
네비게이션, 프로필, 권한 체크 등  
다양한 깊이의 컴포넌트에서 접근이 필요하다.

### 합성 컴포넌트의 내부 상태 공유

Context는 전역에서만 쓰이는 것이 아니다.  
특정 컴포넌트 내부에서 스코프를 한정해서 쓸 수도 있다.

```tsx
const AccordionContext = createContext<{
  openIndex: number | null;
  toggle: (index: number) => void;
} | null>(null);

const Accordion = ({ children }: { children: ReactNode }) => {
  const [openIndex, setOpenIndex] = useState<number | null>(null);

  const toggle = (index: number) => {
    setOpenIndex(prev => (prev === index ? null : index));
  };

  return (
    <AccordionContext.Provider value={{ openIndex, toggle }}>
      {children}
    </AccordionContext.Provider>
  );
};

const AccordionItem = ({ index, title, children }: AccordionItemProps) => {
  const context = useContext(AccordionContext);
  const isOpen = context?.openIndex === index;

  return (
    <div>
      <button onClick={() => context?.toggle(index)}>{title}</button>
      {isOpen && <div>{children}</div>}
    </div>
  );
};
```

```tsx
<Accordion>
  <AccordionItem index={0} title="섹션 1">내용 1</AccordionItem>
  <AccordionItem index={1} title="섹션 2">내용 2</AccordionItem>
</Accordion>
```

`Accordion`이 Provider가 되고  
`AccordionItem`이 Consumer가 된다.  
Provider의 범위가 이 컴포넌트 내부로 한정된다.

이 패턴이 **합성 컴포넌트(Compound Component)**다.  
Context가 전역 도구가 아니라  
**스코프를 가진 전달 도구**라는 점을 보여준다.

### Context 스코프의 중첩

Context는 Provider 단위로 스코프가 결정된다.  
같은 Context라도 Provider를 중첩하면 가장 가까운 Provider의 값을 받는다.

```tsx
<ThemeContext.Provider value="light">
  <Header />
  <ThemeContext.Provider value="dark">
    <Sidebar />
  </ThemeContext.Provider>
</ThemeContext.Provider>
```

`Header`는 `"light"`를 받고  
`Sidebar`는 `"dark"`를 받는다.

서로 다른 종류의 Context도 자유롭게 중첩된다.

```tsx
<ThemeContext.Provider value={theme}>
  <AuthContext.Provider value={auth}>
    <FormContext.Provider value={formConfig}>
      <App />
    </FormContext.Provider>
  </AuthContext.Provider>
</ThemeContext.Provider>
```

각 Context는 독립적으로 자신의 값을 전달한다.

### Context와 외부 상태 관리 도구

Context의 사용 사례를 보면  
자연스럽게 떠오르는 질문이 있다.  
"그러면 Redux나 Zustand 대신 Context를 쓰면 되는 것 아닌가?"

이 둘은 해결하는 문제가 다르다.

| 관점 | Context | 외부 상태 관리 (Zustand, Redux 등) |
|------|---------|-----------------------------------|
| 본질 | 값 전달 (의존성 주입) | 상태 관리 + 구독 |
| 리렌더링 | Provider 하위 전체 | 선택적 구독 (selector) |
| 상태 위치 | React 트리 내부 | React 트리 외부 |
| 적합한 대상 | 자주 바뀌지 않는 값 | 자주 바뀌는 공유 상태 |

Context는 값을 **전달**하는 도구이고  
외부 상태 관리 도구는 값을 **관리하고 구독**하는 도구다.

하나가 다른 하나를 대체하는 관계가 아니다.  
같은 애플리케이션에서 함께 쓰이는 경우도 흔하다.

---

## 정리하며

Context는  
상태를 관리하는 도구가 아니라  
**값을 전달하는 도구**다.

prop drilling을 해결하기 위해 존재하고,  
Provider 단위로 스코프가 결정되며,  
전역에서도, 컴포넌트 내부에서도 쓰일 수 있다.

Context의 값이 바뀌면  
모든 Consumer가 리렌더링된다.  
이것은 자주 바뀌지 않는 값에는 문제가 되지 않지만,  
자주 바뀌는 값에는 다른 도구가 적합하다는 뜻이다.

[상태를 어디에 둘 것인가](/me/docs/decisions-over-patterns/where-to-put-state)에서  
Context는 상태를 배치하는 선택지 중 하나로 등장한다.  
이 문서에서 다룬 Context의 본질을 이해하면  
그 선택이 어떤 상황에서 자연스러운지 판단하기 수월해진다.
