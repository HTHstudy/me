---
layout: default
title: 이 로직은 훅이어야 하는가
parent: Decisions over Patterns
nav_order: 5
permalink: /docs/decisions-over-patterns/should-logic-be-hook
---

# 이 로직은 훅이어야 하는가

> "이 로직은 컴포넌트 안에 있어야 하는가, 밖으로 나와야 하는가  
> 나온다면 훅이어야 하는가, 함수면 충분한가"

## 들어가며 — 반복되는 로직

상태를 정리하고,  
컴포넌트를 나누고,  
제어 방식도 결정했다.

그런데 코드를 보면  
같은 패턴이 반복되고 있다.

```tsx
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

  if (isLoading) return <Loading />;
  if (error) return <Error message={error.message} />;

  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
};

const ProductDetail = ({ productId }) => {
  const [product, setProduct] = useState(null);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    setIsLoading(true);
    fetchProduct(productId)
      .then(setProduct)
      .catch(setError)
      .finally(() => setIsLoading(false));
  }, [productId]);

  if (isLoading) return <Loading />;
  if (error) return <Error message={error.message} />;

  return (
    <div>
      <h1>{product.name}</h1>
      <p>{product.price}원</p>
    </div>
  );
};
```

상태 선언, useEffect, 로딩/에러 처리 —  
구조가 거의 같다.

이 상황에서 자연스럽게  
이런 질문이 떠오른다.

**"이 로직은 훅으로 만들어야 하는가?"**

하지만 반복된다고 해서  
항상 훅으로 만들어야 하는 것은 아니다.  
훅이 아닌 다른 선택지도 있다.

이 문서는  
로직을 추출할지, 추출한다면 어떤 형태로 할지  
결정할 때 어떤 기준으로 판단하는지를 다룬다.

---

## 선택지들

### 그대로 둔다

```tsx
const ShippingForm = () => {
  const [address, setAddress] = useState('');
  const [city, setCity] = useState('');
  const [zipCode, setZipCode] = useState('');

  const isValid = address.length > 0 && city.length > 0 && /^\d{5}$/.test(zipCode);

  const handleSubmit = () => {
    if (!isValid) return;
    submitShipping({ address, city, zipCode });
  };

  return (
    <form onSubmit={handleSubmit}>
      <input value={address} onChange={e => setAddress(e.target.value)} />
      <input value={city} onChange={e => setCity(e.target.value)} />
      <input value={zipCode} onChange={e => setZipCode(e.target.value)} />
      <button disabled={!isValid}>제출</button>
    </form>
  );
};
```

유효성 검사 로직이 있지만,  
이 컴포넌트에서만 사용되고,  
로직이 간단하다.

이것을 useShippingForm 같은 훅으로 빼면  
코드는 분리되지만,  
이 컴포넌트를 이해하려면 두 곳을 봐야 한다.

**언제?**

- 로직이 한곳에서만 사용될 때
- 로직이 간단하여 분리해도 이점이 크지 않을 때
- 상태와 UI가 밀접하게 연결되어 있을 때

**왜?**

- 한곳에서 모든 흐름을 볼 수 있다
- 추상화 없이 직접 읽히는 코드가 된다
- 분리에 따르는 간접 참조가 없다

**비용**

- 컴포넌트가 길어질 수 있다
- 같은 패턴이 다른 곳에서도 필요해지면 중복이 생긴다

### 일반 함수로 추출한다

```tsx
// utils/format.ts
const formatPrice = (price: number): string => {
  return price.toLocaleString('ko-KR') + '원';
};

const calculateDiscount = (price: number, discountRate: number): number => {
  return Math.round(price * (1 - discountRate));
};

// utils/validation.ts
const validateZipCode = (zipCode: string): boolean => {
  return /^\d{5}$/.test(zipCode);
};
```

```tsx
// 컴포넌트에서 사용
const ProductCard = ({ product }) => {
  const finalPrice = calculateDiscount(product.price, product.discountRate);

  return (
    <div>
      <h3>{product.name}</h3>
      <p>{formatPrice(finalPrice)}</p>
    </div>
  );
};
```

React의 기능(상태, 이펙트, 컨텍스트 등)이 필요 없는 로직이다.  
입력을 받아 결과를 돌려주기만 한다.

이런 로직을 훅으로 만들면 불필요한 제약이 생긴다.  
훅은 컴포넌트 최상위에서만 호출할 수 있고,  
조건부 호출이 불가능하다.  
일반 함수에는 이런 제약이 없다.

**언제?**

- React의 상태, 이펙트, 컨텍스트를 사용하지 않을 때
- 순수한 계산, 변환, 포맷팅 로직일 때
- 컴포넌트 밖에서도 사용될 수 있을 때

**왜?**

- 어디서든 자유롭게 호출할 수 있다
- 테스트가 간단하다 — 입력과 출력만 확인하면 된다
- React에 대한 의존이 없다

**비용**

- React 기능이 필요해지면 훅으로 전환해야 한다

### 커스텀 훅으로 추출한다

```tsx
// hooks/useFetch.ts
const useFetch = <T>(url: string) => {
  const [data, setData] = useState<T | null>(null);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    setIsLoading(true);
    fetch(url)
      .then(res => res.json())
      .then(setData)
      .catch(setError)
      .finally(() => setIsLoading(false));
  }, [url]);

  return { data, isLoading, error };
};
```

```tsx
// 사용하는 쪽
const UserProfile = ({ userId }) => {
  const { data: user, isLoading, error } = useFetch(`/api/users/${userId}`);

  if (isLoading) return <Loading />;
  if (error) return <Error message={error.message} />;

  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
};

const ProductDetail = ({ productId }) => {
  const { data: product, isLoading, error } = useFetch(`/api/products/${productId}`);

  if (isLoading) return <Loading />;
  if (error) return <Error message={error.message} />;

  return (
    <div>
      <h1>{product.name}</h1>
      <p>{product.price}원</p>
    </div>
  );
};
```

상태 관리와 데이터 페칭 패턴이 훅 안에 캡슐화된다.  
사용하는 쪽은 데이터, 로딩, 에러만 받으면 된다.

**언제?**

- React의 상태, 이펙트 등을 사용하는 로직일 때
- 같은 패턴이 여러 컴포넌트에서 반복될 때
- 로직을 분리하면 컴포넌트의 의도가 더 명확해질 때

**왜?**

- 반복되는 상태 관리 패턴을 한곳에서 관리한다
- 컴포넌트는 "무엇을 보여줄 것인가"에 집중할 수 있다
- 훅의 이름이 로직의 의도를 설명한다

**비용**

- 간접 참조가 생긴다 — 동작을 이해하려면 훅 코드를 봐야 한다
- 훅의 규칙을 따라야 한다 (조건부 호출 불가, 최상위에서만 호출)
- 너무 범용적으로 만들면 사용이 복잡해진다

### 훅을 조합한다

```tsx
// 기본 훅들
const useFetch = <T>(url: string) => {
  // ... 위와 같음
};

const usePagination = (initialPage = 1) => {
  const [page, setPage] = useState(initialPage);
  const next = () => setPage(p => p + 1);
  const prev = () => setPage(p => Math.max(1, p - 1));
  return { page, next, prev, setPage };
};

// 조합된 훅
const usePaginatedFetch = <T>(baseUrl: string) => {
  const { page, next, prev } = usePagination();
  const { data, isLoading, error } = useFetch<T>(`${baseUrl}?page=${page}`);

  return { data, isLoading, error, page, next, prev };
};
```

```tsx
// 사용하는 쪽
const ProductList = () => {
  const { data: products, isLoading, page, next, prev } =
    usePaginatedFetch('/api/products');

  return (
    <div>
      {isLoading ? <Loading /> : (
        <ul>
          {products.map(p => <li key={p.id}>{p.name}</li>)}
        </ul>
      )}
      <button onClick={prev}>이전</button>
      <span>{page}</span>
      <button onClick={next}>다음</button>
    </div>
  );
};
```

작은 훅들을 조합해서 더 구체적인 동작을 만든다.  
각 훅은 하나의 관심사를 다루고,  
조합된 훅은 이들을 연결한다.

**언제?**

- 이미 존재하는 훅들을 조합해서 더 구체적인 동작을 만들 수 있을 때
- 여러 훅의 상호작용이 반복되는 패턴일 때

**왜?**

- 각 훅이 하나의 관심사만 다룬다
- 조합된 훅을 통해 복잡한 동작을 간결하게 표현한다
- 기본 훅들은 다른 조합에서도 재사용할 수 있다

**비용**

- 추상화 레이어가 깊어진다
- 기본 훅의 인터페이스가 바뀌면 조합된 훅도 영향을 받는다
- 조합이 복잡해지면 디버깅이 어려워진다

---

## 판단의 질문들

로직을 추출할지 결정할 때  
던져볼 수 있는 질문들이 있다.

### 이 로직에 React의 기능이 필요한가?

```tsx
// React 기능이 필요 없는 로직
const formatDate = (date: Date): string => {
  return new Intl.DateTimeFormat('ko-KR').format(date);
};

const validateEmail = (email: string): boolean => {
  return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
};

// 이것은 훅이 아니라 함수다
```

```tsx
// React 기능이 필요한 로직
const useWindowSize = () => {
  const [size, setSize] = useState({
    width: window.innerWidth,
    height: window.innerHeight,
  });

  useEffect(() => {
    const handleResize = () => {
      setSize({ width: window.innerWidth, height: window.innerHeight });
    };
    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);

  return size;
};

// 상태와 이펙트가 필요하다 → 훅이 자연스럽다
```

useState, useEffect, useContext, useRef 등  
React의 기능을 사용하지 않는 로직은  
훅이 아니라 일반 함수로 충분하다.

### 이 로직이 여러 곳에서 반복되는가?

```tsx
// 한곳에서만 사용되는 로직
const UserSettings = () => {
  const [theme, setTheme] = useState(
    () => localStorage.getItem('theme') || 'light'
  );

  useEffect(() => {
    localStorage.setItem('theme', theme);
    document.body.dataset.theme = theme;
  }, [theme]);

  // 이 컴포넌트에서만 테마를 관리한다
  // 굳이 useTheme 훅으로 빼야 하는가?
};
```

```tsx
// 여러 곳에서 반복되는 로직
const UserProfile = () => {
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState(null);
  // ... 같은 페칭 패턴
};

const ProductList = () => {
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState(null);
  // ... 같은 페칭 패턴
};

const OrderHistory = () => {
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState(null);
  // ... 같은 페칭 패턴
};

// 같은 패턴이 세 번 반복된다
// 훅으로 만들 이유가 있다
```

한곳에서만 사용되는 로직을 훅으로 빼는 것은  
코드를 이동시키는 것일 뿐이다.  
반복이 있을 때 훅의 가치가 생긴다.

단, 반복이 없더라도  
컴포넌트가 너무 복잡해서 관심사를 분리하고 싶을 때  
훅으로 빼는 것이 도움이 될 수 있다.

### 훅의 이름이 무엇을 하는지 설명하는가?

```tsx
// 이름이 의도를 설명한다
const useWindowSize = () => { /* ... */ };
const useDebounce = (value, delay) => { /* ... */ };
const useLocalStorage = (key, initialValue) => { /* ... */ };
const useOnClickOutside = (ref, handler) => { /* ... */ };
```

```tsx
// 이름이 모호하다
const useHelper = () => { /* ... */ };
const useData = () => { /* ... */ };
const useLogic = () => { /* ... */ };
const useStuff = () => { /* ... */ };
```

훅의 이름이 무엇을 하는지 설명할 수 없다면,  
그 로직이 하나의 관심사로 묶이지 않는 것일 수 있다.

훅은 "하나의 관심사"를 캡슐화한다.  
이름이 모호하다면 여러 관심사가 섞여 있거나,  
추출할 만한 단위가 아닐 수 있다.

### 분리하면 컴포넌트가 더 읽기 쉬워지는가?

```tsx
// 분리 전: 데이터 페칭과 갱신 로직이 컴포넌트에 섞여 있다
const Dashboard = () => {
  const [data, setData] = useState(null);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    setIsLoading(true);
    fetchDashboardData()
      .then(setData)
      .catch(setError)
      .finally(() => setIsLoading(false));
  }, []);

  const [refreshInterval, setRefreshInterval] = useState(30000);

  useEffect(() => {
    const timer = setInterval(() => {
      fetchDashboardData().then(setData);
    }, refreshInterval);
    return () => clearInterval(timer);
  }, [refreshInterval]);

  if (isLoading) return <Loading />;
  if (error) return <Error message={error.message} />;

  return (
    <div>
      <RefreshControl interval={refreshInterval} onChange={setRefreshInterval} />
      <Charts data={data} />
      <Tables data={data} />
    </div>
  );
};
```

```tsx
// 분리 후: 컴포넌트의 의도가 드러난다
const useDashboardData = (refreshInterval: number) => {
  const [data, setData] = useState(null);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    setIsLoading(true);
    fetchDashboardData()
      .then(setData)
      .catch(setError)
      .finally(() => setIsLoading(false));
  }, []);

  useEffect(() => {
    const timer = setInterval(() => {
      fetchDashboardData().then(setData);
    }, refreshInterval);
    return () => clearInterval(timer);
  }, [refreshInterval]);

  return { data, isLoading, error };
};

const Dashboard = () => {
  const [refreshInterval, setRefreshInterval] = useState(30000);
  const { data, isLoading, error } = useDashboardData(refreshInterval);

  if (isLoading) return <Loading />;
  if (error) return <Error message={error.message} />;

  return (
    <div>
      <RefreshControl interval={refreshInterval} onChange={setRefreshInterval} />
      <Charts data={data} />
      <Tables data={data} />
    </div>
  );
};
```

분리 후 Dashboard 컴포넌트는  
"대시보드 데이터를 가져와서 보여준다"는 의도가 명확해졌다.

데이터 페칭의 세부 구현은 훅 안에 있고,  
컴포넌트는 결과만 사용한다.

---

## 결정의 흐름

몇 가지 상황에서  
어떤 질문을 던지게 되는지 살펴본다.

### 가격 포맷팅 로직

```tsx
// 상황: 여러 곳에서 가격을 포맷팅한다
const ProductCard = ({ product }) => {
  const formattedPrice = product.price.toLocaleString('ko-KR') + '원';
  return <p>{formattedPrice}</p>;
};

const CartItem = ({ item }) => {
  const formattedPrice = item.price.toLocaleString('ko-KR') + '원';
  return <span>{formattedPrice}</span>;
};
```

**판단 과정**

1. React 기능이 필요한가?  
   → 아니다. 순수한 문자열 변환이다.  
   **함수로 충분하다.**

**결론: 일반 함수**

```tsx
const formatPrice = (price: number): string => {
  return price.toLocaleString('ko-KR') + '원';
};

// 어디서든 사용
<p>{formatPrice(product.price)}</p>
<span>{formatPrice(item.price)}</span>
```

이것을 useFormatPrice 같은 훅으로 만들 이유가 없다.  
React 기능을 사용하지 않는 로직은  
함수가 더 자연스럽다.

### 윈도우 크기 감지

```tsx
// 상황: 화면 크기에 따라 레이아웃을 바꿔야 한다
const Dashboard = () => {
  const [windowWidth, setWindowWidth] = useState(window.innerWidth);

  useEffect(() => {
    const handleResize = () => setWindowWidth(window.innerWidth);
    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);

  return windowWidth > 768 ? <DesktopLayout /> : <MobileLayout />;
};
```

**판단 과정**

1. React 기능이 필요한가?  
   → 그렇다. 상태와 이펙트가 필요하다.  
   **훅이 자연스럽다.**

2. 여러 곳에서 반복되는가?  
   → 반응형 레이아웃이 필요한 곳이 여럿이다.  
   **반복된다.**

3. 이름이 의도를 설명하는가?  
   → useWindowSize — 무엇을 하는지 명확하다.  
   **그렇다.**

**결론: 커스텀 훅**

```tsx
const useWindowSize = () => {
  const [size, setSize] = useState({
    width: window.innerWidth,
    height: window.innerHeight,
  });

  useEffect(() => {
    const handleResize = () => {
      setSize({ width: window.innerWidth, height: window.innerHeight });
    };
    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);

  return size;
};

const Dashboard = () => {
  const { width } = useWindowSize();
  return width > 768 ? <DesktopLayout /> : <MobileLayout />;
};
```

### 로그인 폼 상태

```tsx
// 상황: 간단한 로그인 폼
const LoginForm = () => {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [error, setError] = useState(null);

  const handleSubmit = async (e) => {
    e.preventDefault();
    try {
      await login(email, password);
    } catch (err) {
      setError(err.message);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input value={email} onChange={e => setEmail(e.target.value)} />
      <input
        type="password"
        value={password}
        onChange={e => setPassword(e.target.value)}
      />
      {error && <p>{error}</p>}
      <button type="submit">로그인</button>
    </form>
  );
};
```

**판단 과정**

1. 이 로직이 여러 곳에서 반복되는가?  
   → 로그인 폼은 하나뿐이다.  
   **반복되지 않는다.**

2. 분리하면 더 읽기 쉬워지는가?  
   → 상태와 UI가 밀접하게 연결되어 있다.  
   useLoginForm으로 빼면 오히려 두 곳을 봐야 한다.  
   **아니다.**

**결론: 그대로 둔다**

```tsx
// useLoginForm으로 빼면?
const useLoginForm = () => {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [error, setError] = useState(null);

  const handleSubmit = async () => {
    try {
      await login(email, password);
    } catch (err) {
      setError(err.message);
    }
  };

  return { email, setEmail, password, setPassword, error, handleSubmit };
};

// 훅에서 반환하는 값이 너무 많다
// 컴포넌트의 거의 모든 로직이 훅으로 옮겨갔을 뿐이다
```

훅으로 빼도 되지만,  
이 경우 훅은 "로직을 정리한 것"이 아니라  
"로직을 이동한 것"에 가깝다.

### 로컬 스토리지 동기화

```tsx
// 상황: 여러 설정을 로컬 스토리지와 동기화한다
const Settings = () => {
  const [theme, setTheme] = useState(
    () => localStorage.getItem('theme') || 'light'
  );
  const [fontSize, setFontSize] = useState(
    () => Number(localStorage.getItem('fontSize')) || 16
  );
  const [language, setLanguage] = useState(
    () => localStorage.getItem('language') || 'ko'
  );

  useEffect(() => { localStorage.setItem('theme', theme); }, [theme]);
  useEffect(() => { localStorage.setItem('fontSize', String(fontSize)); }, [fontSize]);
  useEffect(() => { localStorage.setItem('language', language); }, [language]);

  // ...
};
```

**판단 과정**

1. React 기능이 필요한가?  
   → 그렇다. 상태와 이펙트가 필요하다.

2. 같은 패턴이 반복되는가?  
   → "로컬 스토리지에서 읽고, 상태로 관리하고, 변경 시 저장"이  
   세 번 반복된다.  
   **반복된다.**

3. 이름이 의도를 설명하는가?  
   → useLocalStorage — 명확하다.

**결론: 커스텀 훅**

```tsx
const useLocalStorage = <T>(key: string, initialValue: T) => {
  const [value, setValue] = useState<T>(() => {
    const stored = localStorage.getItem(key);
    return stored !== null ? JSON.parse(stored) : initialValue;
  });

  useEffect(() => {
    localStorage.setItem(key, JSON.stringify(value));
  }, [key, value]);

  return [value, setValue] as const;
};

const Settings = () => {
  const [theme, setTheme] = useLocalStorage('theme', 'light');
  const [fontSize, setFontSize] = useLocalStorage('fontSize', 16);
  const [language, setLanguage] = useLocalStorage('language', 'ko');

  // 각 설정의 동기화 로직이 사라졌다
  // 컴포넌트는 설정을 사용하는 것에 집중한다
};
```

---

## 훅 추출의 비용

### 간접 참조가 늘어난다

```tsx
// 훅이 없을 때: 동작이 바로 보인다
const Component = () => {
  const [value, setValue] = useState('');

  useEffect(() => {
    // 여기서 무슨 일이 일어나는지 바로 알 수 있다
    const subscription = subscribe(value);
    return () => subscription.unsubscribe();
  }, [value]);

  // ...
};

// 훅으로 빼면: 동작을 보려면 훅 파일로 가야 한다
const Component = () => {
  const result = useSubscription(value);
  // useSubscription이 정확히 무엇을 하는지는 여기서 알 수 없다
  // ...
};
```

좋은 훅 이름은 내부를 안 봐도 무엇을 하는지 짐작할 수 있게 한다.  
하지만 구체적인 동작을 확인하려면 결국 훅 코드를 봐야 한다.

이 간접 참조가  
여러 곳의 중복을 제거하는 것보다 가치가 있을 때  
훅 추출이 의미가 있다.

### 훅에 담기기 어려운 로직

```tsx
// 상태 업데이트 직후 DOM 조작이 필요한 경우
const AutoResizeTextarea = () => {
  const [value, setValue] = useState('');
  const textareaRef = useRef(null);

  // 값이 바뀔 때마다 높이를 조절한다
  // 이 로직은 ref와 상태가 밀접하게 연결되어 있다
  useEffect(() => {
    const textarea = textareaRef.current;
    textarea.style.height = 'auto';
    textarea.style.height = textarea.scrollHeight + 'px';
  }, [value]);

  return (
    <textarea
      ref={textareaRef}
      value={value}
      onChange={e => setValue(e.target.value)}
    />
  );
};
```

이 로직을 훅으로 빼면  
ref를 훅과 컴포넌트가 공유해야 하고,  
상태 업데이트와 DOM 조작의 관계가 분리된다.

모든 로직이 훅에 담기기 적합한 것은 아니다.  
상태와 UI가 긴밀하게 얽혀 있을 때  
분리하면 오히려 복잡해질 수 있다.

### 너무 범용적인 훅

```tsx
// 너무 범용적: 무엇을 하는 훅인가?
const useFetch = (url, options = {}) => {
  const {
    method = 'GET',
    headers = {},
    body,
    retries = 0,
    retryDelay = 1000,
    timeout = 5000,
    transform,
    cache = true,
    cacheKey,
    onSuccess,
    onError,
    enabled = true,
  } = options;

  // ... 100줄의 구현
};
```

범용적으로 만들수록  
옵션이 늘어나고, 코드가 복잡해지고,  
사용하는 쪽도 옵션을 이해해야 한다.

훅은 "이 상황에서 필요한 만큼"만 추상화하는 것이 자연스럽다.  
모든 경우를 대비한 범용 훅보다  
구체적인 목적을 가진 작은 훅이 읽기 쉽다.

---

## 정리하며 — 훅은 관심사의 단위다

로직을 추출할지 결정하는 것은  
"이 관심사를 어디에 둘 것인가"를 정하는 것이다.

### 선택지별 특성

| 선택지 | 언제 | 비용 |
|--------|------|------|
| 그대로 둔다 | 한곳에서만 사용, 간단한 로직 | 컴포넌트가 길어질 수 있음 |
| 일반 함수 | React 기능 불필요, 순수 계산 | React 기능 필요 시 전환 필요 |
| 커스텀 훅 | React 기능 사용, 반복되는 패턴 | 간접 참조 증가 |
| 훅 조합 | 기존 훅들의 조합으로 해결 가능 | 추상화 깊이 증가 |

### 판단의 출발점이 될 수 있는 규칙들

절대적인 규칙은 아니지만,  
반복적으로 유효한 패턴들이 있다.

**1. React 기능을 사용하지 않으면 함수로 충분하다**

순수한 계산, 변환, 유효성 검사는  
훅이 아니라 일반 함수가 자연스럽다.  
훅의 제약(최상위 호출, 조건부 불가)이 없어  
더 자유롭게 사용할 수 있다.

**2. 한곳에서만 사용되는 로직은 급하게 빼지 않아도 된다**

두 번째 사용처가 생겼을 때 추출해도 늦지 않다.  
미리 훅으로 빼면 필요 없는 간접 참조만 생긴다.

**3. 훅의 이름이 의도를 설명할 수 없으면 추출을 다시 생각한다**

useHelper, useLogic 같은 이름이 붙는다면  
그 로직이 하나의 관심사로 묶이지 않는 것일 수 있다.

**4. 훅은 이동이 아니라 캡슐화일 때 가치가 있다**

컴포넌트의 모든 로직을 훅으로 옮기는 것은  
캡슐화가 아니라 장소 변경이다.  
훅으로 빼고 나서 컴포넌트가 더 명확해지는지가 기준이다.

> 로직을 훅으로 빼는 것은  
> 코드를 줄이는 기술이 아니라  
> 관심사의 경계를 정하는 것이다.  
> 그 경계가 의미를 가질 때  
> 추출은 가치가 있다.
