---
layout: default
title: 이 값은 상태여야 하는가
parent: Decisions over Patterns
nav_order: 1
permalink: /docs/decisions-over-patterns/should-it-be-state
---

# 이 값은 상태여야 하는가

> "이 값은 상태여야 하는가, 계산이어야 하는가  
> 그 선택은 무엇을 단순하게 만들고  
> 무엇을 동기화해야 하게 만드는가"

## 들어가며 — 상태가 필요한가

컴포넌트를 작성하다 보면  
새로운 값이 필요해진다.

```tsx
const Cart = ({ items }) => {
  // 총합이 필요하다
  // 이걸 상태로 만들어야 하나?
};
```

이 질문에 답하기 전에  
먼저 물어야 할 것이 있다.

**이 값은 다른 상태로부터 계산할 수 있는가?**

```tsx
const [items, setItems] = useState([]);

// total은 items로부터 계산할 수 있다
const total = items.reduce((sum, item) => sum + item.price, 0);
```

계산할 수 있다면,  
상태로 만들 이유가 있는지 의심한다.

```tsx
// 선택 1: 계산한다 (상태가 아니다)
const total = items.reduce((sum, item) => sum + item.price, 0);

// 선택 2: 상태로 만든다
const [total, setTotal] = useState(0);
useEffect(() => {
  setTotal(items.reduce((sum, item) => sum + item.price, 0));
}, [items]);
```

둘 다 "작동"은 한다.  
하지만 두 번째는 상태가 두 개다.  
items가 바뀔 때 total도 바꿔줘야 한다.  
동기화를 잊으면 버그가 된다.

이처럼 다른 상태로부터 계산 가능한 값을  
**파생 값(derived value)**이라고 부른다.

이 문서는  
값을 상태로 만들지 계산할지 결정할 때  
어떤 기준으로 판단하는지를 다룬다.

---

## 선택지들

"상태로 만들지 않는다"고 해도  
값을 다루는 방법은 여러 가지다.

### 렌더링 중 계산

```tsx
const Cart = ({ items }) => {
  const total = items.reduce((sum, item) => sum + item.price, 0);
  const itemCount = items.length;
  const hasItems = items.length > 0;

  return (
    <div>
      <p>상품 {itemCount}개</p>
      <p>총액: {total}원</p>
      {hasItems && <CheckoutButton />}
    </div>
  );
};
```

가장 단순한 방식이다.  
렌더링할 때마다 계산한다.

**언제?**

- 계산이 가볍다 (O(n) 이하, n이 작음)
- 대부분의 파생 값은 여기에 해당한다

**왜?**

- 상태가 하나뿐이다 (items만 관리)
- 동기화 문제가 없다
- 코드가 가장 단순하다

**비용**

- 매 렌더마다 계산된다
- 하지만 대부분의 계산은 충분히 빠르다

### 계산이 비싸면?

계산 비용이 실제로 클 때는  
`useMemo`로 메모이제이션할 수 있다.

```tsx
const sortedItems = useMemo(() => [...items].sort(complexSort), [items]);
```

하지만 대부분의 계산은 충분히 빠르다.  
**측정 없이 useMemo를 쓰지 않는다.**

### useState + useEffect (대부분 안티패턴)

```tsx
const SearchResults = ({ items, query }) => {
  const [filteredItems, setFilteredItems] = useState([]);

  useEffect(() => {
    setFilteredItems(items.filter((item) => item.name.includes(query)));
  }, [items, query]);

  return (
    <ul>
      {filteredItems.map((item) => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  );
};
```

별도 상태로 저장하고 Effect로 동기화한다.

**언제?**  
대부분의 경우, 이 방식은 **필요하지 않다.**

**왜 문제인가?**

- 상태가 두 개가 된다 (items, filteredItems)
- 동기화 로직이 필요하다
- Effect는 렌더링 후에 실행된다
- 불필요한 이중 렌더링이 발생한다

```tsx
// 렌더링 흐름
1. items가 바뀜
2. 첫 번째 렌더링 (filteredItems는 아직 이전 값)
3. Effect 실행, setFilteredItems 호출
4. 두 번째 렌더링 (filteredItems가 새 값)
```

이것은 React 공식 문서에서  
"You Might Not Need an Effect"로 다루는 대표적인 사례다.

**그럼에도 필요한 경우**

드물지만, 파생 값을 상태로 관리해야 하는 경우가 있다.

```tsx
// 비동기 계산이 필요한 경우
const SearchResults = ({ query }) => {
  const [results, setResults] = useState([]);
  const [isLoading, setIsLoading] = useState(false);

  useEffect(() => {
    setIsLoading(true);
    fetchSearchResults(query)
      .then(setResults)
      .finally(() => setIsLoading(false));
  }, [query]);

  // ...
};
```

이 경우는 "파생 상태"가 아니라  
"서버 상태"에 가깝다.  
query가 바뀌면 서버에서 새로 가져와야 한다.

### useState + 이벤트 핸들러

```tsx
const SearchPage = () => {
  const [items, setItems] = useState([]);
  const [filteredItems, setFilteredItems] = useState([]);
  const [query, setQuery] = useState("");

  const handleQueryChange = (newQuery) => {
    setQuery(newQuery);
    // 원본이 바뀌는 시점에 함께 갱신
    setFilteredItems(items.filter((item) => item.name.includes(newQuery)));
  };

  const handleItemsLoad = (newItems) => {
    setItems(newItems);
    setFilteredItems(newItems.filter((item) => item.name.includes(query)));
  };

  // ...
};
```

Effect 대신 이벤트 핸들러에서 동기화한다.

**언제?**

- 파생 값의 계산이 복잡하고
- 동기화 시점이 명확할 때
- 하지만 대부분은 여전히 렌더링 중 계산이 더 낫다

**왜?**

- Effect의 이중 렌더링을 피한다
- 동기화 시점이 명시적이다

**비용**

- 모든 변경 지점에서 동기화를 잊지 않아야 한다
- 로직이 분산된다
- 동기화를 빠뜨리면 버그가 된다

**대안: 그냥 계산하기**

위의 복잡한 코드 대신:

```tsx
const SearchPage = () => {
  const [items, setItems] = useState([]);
  const [query, setQuery] = useState("");

  // 계산하면 된다
  const filteredItems = items.filter((item) => item.name.includes(query));

  // ...
};
```

훨씬 단순하다.  
동기화 문제도 없다.

---

## 판단의 질문들

파생 값을 어떻게 다룰지 결정할 때  
던져볼 수 있는 질문들이 있다.

### 이 값은 다른 상태로부터 계산 가능한가?

```tsx
// items가 있으면 total은 계산 가능하다
const [items, setItems] = useState([]);
const total = items.reduce((sum, item) => sum + item.price, 0);

// query와 items가 있으면 filteredItems는 계산 가능하다
const filteredItems = items.filter((item) => item.name.includes(query));
```

계산 가능하다면,  
별도 상태로 만들 이유가 있는지 먼저 의심한다.

**진실의 근원(Single Source of Truth)**

```tsx
// 좋지 않음: 두 개의 진실
const [items, setItems] = useState([]);
const [filteredItems, setFilteredItems] = useState([]);
// items와 filteredItems가 불일치할 수 있다

// 좋음: 하나의 진실
const [items, setItems] = useState([]);
const filteredItems = items.filter(/* ... */);
// filteredItems는 항상 items와 일치한다
```

상태가 하나면 불일치가 불가능하다.

### 이 값을 직접 수정해야 하는가?

```tsx
// 사용자가 filteredItems를 직접 조작하는가?
// 예: 필터 결과에서 일부를 제외

const [items, setItems] = useState([]);
const [excludedIds, setExcludedIds] = useState(new Set());

// 파생은 계산, 제외는 별도 상태
const filteredItems = items.filter((item) => item.active).filter((item) => !excludedIds.has(item.id));

const handleExclude = (id) => {
  setExcludedIds((prev) => new Set([...prev, id]));
};
```

직접 수정이 필요하다면,  
그 "수정 사항"을 별도 상태로 관리하고  
최종 값은 계산으로 도출한다.

### 계산 비용이 실제로 큰가?

대부분의 계산은 충분히 빠르다.

```tsx
// 이 정도는 그냥 계산해도 된다
const hasItems = items.length > 0;
const total = items.reduce((sum, item) => sum + item.price, 0);
```

계산이 비싸서 문제가 되는 경우는 드물다.  
**체감되는 버벅임이 있을 때** 측정하고,  
그때 메모이제이션을 고려한다.

### 이 값이 렌더링 외부에서 필요한가?

```tsx
// 렌더링 안에서만 필요하다면 → 그냥 계산
const Cart = ({ items }) => {
  const total = items.reduce((sum, item) => sum + item.price, 0);
  return <p>총액: {total}</p>;
};

// 다른 곳에서도 접근해야 한다면 → 위치를 고민
const useCart = () => {
  const items = useCartItems();
  // total을 여러 곳에서 사용한다면?
  // 커스텀 훅에서 계산해서 반환
  const total = items.reduce((sum, item) => sum + item.price, 0);
  return { items, total };
};
```

---

## 결정의 흐름

몇 가지 상황에서  
어떤 질문을 던지게 되는지 살펴본다.

### 필터링된 목록

```tsx
// 상황: 검색어로 목록을 필터링
const ProductList = ({ products }) => {
  const [query, setQuery] = useState("");

  // 질문: filteredProducts를 어떻게 다룰 것인가?
};
```

**판단 과정**

1. filteredProducts는 products와 query로부터 계산 가능한가?  
   → **그렇다**

2. filteredProducts를 직접 수정해야 하는가?  
   → **아니다** (검색 결과를 임의로 바꾸지 않는다)

3. 계산 비용이 큰가?  
   → 목록이 수천 개가 아니라면 **아니다**

**결론: 렌더링 중 계산**

```tsx
const ProductList = ({ products }) => {
  const [query, setQuery] = useState("");

  const filteredProducts = products.filter((product) => product.name.toLowerCase().includes(query.toLowerCase()));

  return (
    <>
      <input value={query} onChange={(e) => setQuery(e.target.value)} />
      <ul>
        {filteredProducts.map((product) => (
          <li key={product.id}>{product.name}</li>
        ))}
      </ul>
    </>
  );
};
```

**만약 목록이 매우 크다면?**

```tsx
const ProductList = ({ products }) => {
  const [query, setQuery] = useState("");

  // useMemo로 메모이제이션
  const filteredProducts = useMemo(
    () => products.filter((product) => product.name.toLowerCase().includes(query.toLowerCase())),
    [products, query]
  );

  // ...
};
```

### 폼 유효성 검사

```tsx
// 상황: 여러 필드의 유효성을 검사
const SignupForm = () => {
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [confirmPassword, setConfirmPassword] = useState("");

  // 질문: isValid를 어떻게 다룰 것인가?
};
```

**판단 과정**

1. isValid는 다른 상태로부터 계산 가능한가?  
   → **그렇다** (email, password, confirmPassword로 계산)

2. isValid를 직접 수정해야 하는가?  
   → **아니다**

3. 계산 비용이 큰가?  
   → **아니다** (간단한 검증 로직)

**결론: 렌더링 중 계산**

```tsx
const SignupForm = () => {
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [confirmPassword, setConfirmPassword] = useState("");

  const isEmailValid = email.includes("@");
  const isPasswordValid = password.length >= 8;
  const isPasswordMatch = password === confirmPassword;
  const isValid = isEmailValid && isPasswordValid && isPasswordMatch;

  return (
    <form>
      <input value={email} onChange={(e) => setEmail(e.target.value)} />
      {!isEmailValid && email && <span>유효한 이메일을 입력하세요</span>}

      <input value={password} onChange={(e) => setPassword(e.target.value)} />
      {!isPasswordValid && password && <span>8자 이상 입력하세요</span>}

      <input value={confirmPassword} onChange={(e) => setConfirmPassword(e.target.value)} />
      {!isPasswordMatch && confirmPassword && <span>비밀번호가 일치하지 않습니다</span>}

      <button disabled={!isValid}>가입</button>
    </form>
  );
};
```

**잘못된 접근**

```tsx
// 이렇게 하지 않는다
const SignupForm = () => {
  const [email, setEmail] = useState("");
  const [isEmailValid, setIsEmailValid] = useState(false);

  useEffect(() => {
    setIsEmailValid(email.includes("@"));
  }, [email]);

  // 불필요한 상태와 Effect
};
```

### 정렬된 목록

```tsx
const DataTable = ({ data }) => {
  const [sortKey, setSortKey] = useState('name');
  const [sortOrder, setSortOrder] = useState('asc');

  // sortedData는 계산 가능하다
  const sortedData = [...data].sort((a, b) => {
    const result = a[sortKey] < b[sortKey] ? -1 : 1;
    return sortOrder === 'asc' ? result : -result;
  });

  return (/* ... */);
};
```

정렬도 계산이다.  
데이터가 많아서 느리다면 메모이제이션을 고려한다.

### 복합 파생: 필터 + 정렬 + 페이지네이션

여러 단계의 파생이 있어도 원칙은 같다.

```tsx
const ProductTable = ({ products }) => {
  const [query, setQuery] = useState('');
  const [sortKey, setSortKey] = useState('name');
  const [page, setPage] = useState(1);
  const pageSize = 20;

  // 단계별로 계산
  const filteredProducts = products.filter(p => p.name.includes(query));
  const sortedProducts = [...filteredProducts].sort((a, b) =>
    a[sortKey].localeCompare(b[sortKey])
  );
  const totalPages = Math.ceil(filteredProducts.length / pageSize);
  const pagedProducts = sortedProducts.slice(
    (page - 1) * pageSize,
    page * pageSize
  );

  return (/* ... */);
};
```

상태는 `query`, `sortKey`, `page` 세 개뿐이다.  
나머지는 모두 계산이다.

---

## 상태 정규화와 파생

때로는 데이터 구조 자체를 바꾸면  
파생 계산이 단순해지거나 사라진다.

### 배열 vs 맵

```tsx
// 배열로 저장
const [items, setItems] = useState([
  { id: 1, name: "A" },
  { id: 2, name: "B" },
]);

// id로 찾기: O(n)
const findItem = (id) => items.find((item) => item.id === id);

// 맵으로 저장
const [itemsById, setItemsById] = useState({
  1: { id: 1, name: "A" },
  2: { id: 2, name: "B" },
});

// id로 찾기: O(1)
const findItem = (id) => itemsById[id];
```

자주 id로 조회한다면  
맵 구조가 파생 계산을 없앤다.

### 선택 상태

```tsx
// 패턴 1: 선택된 항목들을 배열로
const [selectedIds, setSelectedIds] = useState([1, 3, 5]);

// "이 항목이 선택되었는가?" → O(n)
const isSelected = (id) => selectedIds.includes(id);

// 패턴 2: Set으로
const [selectedIds, setSelectedIds] = useState(new Set([1, 3, 5]));

// "이 항목이 선택되었는가?" → O(1)
const isSelected = (id) => selectedIds.has(id);
```

### 정규화된 상태

```tsx
// 비정규화: 중첩된 구조
const [posts, setPosts] = useState([
  {
    id: 1,
    title: "글 제목",
    author: { id: 1, name: "작성자" },
    comments: [{ id: 1, text: "댓글", author: { id: 2, name: "댓글 작성자" } }],
  },
]);

// 정규화: 분리된 구조
const [posts, setPosts] = useState({ 1: { id: 1, title: "글 제목", authorId: 1 } });
const [users, setUsers] = useState({ 1: { id: 1, name: "작성자" }, 2: { id: 2, name: "댓글 작성자" } });
const [comments, setComments] = useState({ 1: { id: 1, text: "댓글", authorId: 2, postId: 1 } });

// 조합이 필요할 때 파생
const getPostWithAuthor = (postId) => {
  const post = posts[postId];
  return { ...post, author: users[post.authorId] };
};
```

정규화하면:

- 데이터 중복이 없다
- 업데이트가 한 곳에서 일어난다
- 하지만 조합할 때 파생 계산이 필요하다

비정규화하면:

- 데이터가 중복된다
- 업데이트 시 여러 곳을 바꿔야 한다
- 하지만 읽을 때 계산이 필요 없다

어느 쪽이 낫다고 말하기 어렵다.  
**읽기가 많은지, 쓰기가 많은지**에 따라 달라진다.

---

## 정리하며 — 계산할 수 있으면 계산한다

파생 값을 다루는 핵심 원칙은 단순하다.

**다른 상태로부터 계산할 수 있으면,  
별도 상태로 만들지 않는다.**

### 왜 계산이 기본인가

상태가 하나면:

- 동기화 문제가 없다
- 불일치 상태가 불가능하다
- 코드가 단순하다

상태가 둘 이상이면:

- 동기화가 필요하다
- 불일치 가능성이 생긴다
- 복잡도가 증가한다

```tsx
// 상태 하나: items가 바뀌면 total은 자동으로 맞다
const [items, setItems] = useState([]);
const total = items.reduce((sum, item) => sum + item.price, 0);

// 상태 둘: items가 바뀔 때 total도 바꿔야 한다
const [items, setItems] = useState([]);
const [total, setTotal] = useState(0);
// 동기화를 잊으면 버그
```

### 선택지별 특성

| 선택지            | 언제               | 비용                |
| ----------------- | ------------------ | ------------------- |
| 렌더링 중 계산    | 대부분의 경우      | 매 렌더마다 계산    |
| useMemo           | 계산이 비쌀 때     | 의존성 관리         |
| useState + Effect | 거의 사용하지 않음 | 동기화, 이중 렌더링 |
| useState + 핸들러 | 특수한 경우        | 모든 변경점 관리    |

### 판단의 출발점이 될 수 있는 규칙들

절대적인 규칙은 아니지만,  
반복적으로 유효한 패턴들이 있다.

**1. 계산 가능하면 계산한다**

```tsx
// 기본: 그냥 계산
const total = items.reduce((sum, item) => sum + item.price, 0);
```

**2. 성능 문제가 측정되면 useMemo**

```tsx
// 실제로 느릴 때만
const sortedItems = useMemo(() => [...items].sort(complexSort), [items]);
```

**3. useState + useEffect 동기화는 의심한다**

```tsx
// 이 패턴이 보이면 "계산으로 바꿀 수 있나?" 묻는다
const [derived, setDerived] = useState(/*...*/);
useEffect(() => {
  setDerived(/* source로부터 계산 */);
}, [source]);
```

**4. 데이터 구조를 먼저 고민한다**

```tsx
// O(n) 조회가 반복되면
// 계산을 최적화하기보다 구조를 바꾼다
// 배열 대신 맵으로 저장하면 O(1) 조회
const [itemsById, setItemsById] = useState({});
```

> 상태는 가능한 적게,  
> 계산은 가능한 단순하게.  
> 복잡한 동기화보다  
> 단일 진실의 근원이 낫다.
