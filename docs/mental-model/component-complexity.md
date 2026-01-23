---
layout: default
title: 컴포넌트 복잡도
parent: Mental Model
nav_order: 1
permalink: /docs/mental-model/component-complexity
---

# 컴포넌트 복잡도
> “컴포넌트는 언제, 어떤 선택 때문에  
> 추론하기 어려운 시스템으로 변하는가”

## 들어가며 — 단순했던 컴포넌트가 복잡해지는 순간

실무에서 컴포넌트를 작성할 때,  
처음에는 대부분 단순하고 명확하게 시작한다.

```tsx
export const UserProfile = () => {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    fetchUser().then(setUser);
  }, []);
  
  if (!user) return <Loading />;
  
  return <div>{user.name}</div>;
};
```

이 정도 코드는 누가 봐도 쉽게 이해할 수 있다.  
무엇을 하는지, 어떤 상태를 가지는지, 언제 데이터를 가져오는지.

하지만 시간이 지나고  
요구사항이 하나씩 추가되면서  
이 컴포넌트는 점점 다른 모습으로 변해간다.

> "에러 처리도 해야 해요."  
> "새로고침 버튼도 필요해요."  
> "편집 모드도 추가해주세요."  
> "권한에 따라 다르게 보여야 해요."

각 요구사항은 모두 합리적이었다.  
그리고 각각을 추가하는 순간에는  
여전히 코드가 관리 가능해 보였다.

하지만 어느 순간부터  
이 컴포넌트를 수정하는 것이  
두려워지기 시작했다.

코드는 여전히 작동했지만,  
더 이상 한눈에 들어오지 않았고,  
어디를 고치면 어디가 영향을 받는지  
예측하기 어려워졌다.

이 문서는  
그 "어느 순간"에 무슨 일이 일어났는지,  
무엇이 컴포넌트를 점점 읽기 어렵게 만드는지를  
다시 들여다보기 위해 쓰였다.

---

## 복잡도의 근원: 상태

컴포넌트가 복잡해지는 가장 큰 이유는  
**상태(State)**다.

프론트엔드에서 말하는 상태란  
시간에 따라 변하는 값이며,  
컴포넌트의 렌더링 결과를 결정한다.

```tsx
UI = f(state)
```

이 공식은 단순해 보이지만,  
그 이면에는 중요한 함의가 숨어있다.

> 상태가 늘어날수록,  
> 가능한 UI의 조합도 기하급수적으로 늘어난다.

---

### 상태의 조합 폭발

상태가 하나일 때는 단순하다.

```tsx
const [isLoading, setIsLoading] = useState(false);

// 가능한 경우: 2가지
// 1. isLoading = true
// 2. isLoading = false
```

상태가 둘이 되면 조합이 늘어난다.

```tsx
const [isLoading, setIsLoading] = useState(false);
const [isError, setIsError] = useState(false);

// 가능한 경우: 4가지
// 1. loading: false, error: false
// 2. loading: false, error: true
// 3. loading: true,  error: false
// 4. loading: true,  error: true  // 이게 가능한가?
```

상태가 셋이 되면 문제가 분명해진다.

```tsx
const [isLoading, setIsLoading] = useState(false);
const [isError, setIsError] = useState(false);
const [isSuccess, setIsSuccess] = useState(false);

// 가능한 경우: 8가지
// 하지만 그 중 논리적으로 말이 되는 경우는 몇 개인가?
// loading과 success가 동시에 true?
// error와 success가 동시에 true?
```

상태가 N개의 boolean이라면,  
가능한 조합은 2^N개다.

3개면 8가지,  
4개면 16가지,  
5개면 32가지.

이 모든 조합이  
논리적으로 타당한 상태인가?  
그렇지 않다면,  
**불가능한 상태를 만들 수 있는 설계**를  
우리는 허용하고 있는 것이다.

이것이 복잡도의 첫 번째 근원이다.

> 상태의 개수가 늘어날수록,  
> 관리해야 할 경우의 수는 기하급수적으로 증가한다.

---

## 상태 간 결합

상태가 늘어나는 것만으로는  
아직 문제가 명확하지 않을 수 있다.

진짜 문제는  
상태들이 **독립적이지 않을 때** 발생한다.

---

### 독립적인 상태 vs 결합된 상태

독립적인 상태는 괜찮다.

```tsx
const [username, setUsername] = useState('');
const [theme, setTheme] = useState('light');

// username과 theme는 서로 무관
// 한쪽을 바꿔도 다른 쪽에 영향 없음
```

하지만 상태들이 서로 의존하기 시작하면  
복잡도가 급격히 증가한다.

```tsx
const [isEditing, setIsEditing] = useState(false);
const [editedValue, setEditedValue] = useState('');
const [originalValue, setOriginalValue] = useState('');

// 규칙:
// - isEditing이 true일 때만 editedValue가 의미 있음
// - isEditing이 false가 되면 editedValue를 어떻게 해야 하나?
// - originalValue는 언제 업데이트되나?
```

이런 상태들은  
각자 독립적으로 존재하는 것처럼 보이지만,  
실제로는 **하나의 논리적 상태**를 여러 조각으로 나눠둔 것이다.

문제는 이 논리적 관계가  
코드에 명시되지 않는다는 점이다.

---

### Effect로 상태를 동기화하기 시작하는 순간

상태 간 의존성이 생기면  
자연스럽게 다음 패턴이 나타난다.

```tsx
const [count, setCount] = useState(0);
const [isEven, setIsEven] = useState(true);

// count가 바뀌면 isEven도 업데이트
useEffect(() => {
  setIsEven(count % 2 === 0);
}, [count]);
```

이 코드는 작동한다.  
하지만 여기서부터 복잡도가 시작된다.

`isEven`은 진짜 상태가 아니다.  
그것은 `count`로부터 **파생된 값**이다.

상태로 만드는 순간,  
우리는 동기화의 책임을 떠안게 된다.

- count가 바뀔 때마다 isEven을 업데이트해야 한다
- 순서가 중요해진다 (count가 먼저, 그 다음 isEven)
- 버그 가능성이 생긴다 (동기화를 잊어버리면?)

파생 상태가 하나 더 늘어나면,  
Effect도 하나 더 늘어난다.

```tsx
const [count, setCount] = useState(0);
const [isEven, setIsEven] = useState(true);
const [isPositive, setIsPositive] = useState(false);

useEffect(() => {
  setIsEven(count % 2 === 0);
}, [count]);

useEffect(() => {
  setIsPositive(count > 0);
}, [count]);
```

Effect가 늘어날수록,  
실행 순서와 의존성을 추적하기 어려워진다.

이것이 복잡도의 두 번째 근원이다.

> 상태들이 서로 의존할 때,  
> 동기화의 책임이 생기고,  
> 변경의 연쇄 반응이 시작된다.

---

## 암시적 상태의 함정

앞에서 본 여러 boolean 상태의 문제를  
다시 생각해보자.

```tsx
const [isLoading, setIsLoading] = useState(false);
const [isError, setIsError] = useState(false);
const [isSuccess, setIsSuccess] = useState(false);
```

이 설계의 문제는  
**불가능한 상태를 표현할 수 있다**는 점이다.

- `isLoading: true, isSuccess: true` — 동시에?
- `isError: true, isSuccess: true` — 둘 다?
- 모두 false일 때는 무슨 상태인가? — idle?

이런 설계는 암시적으로  
"개발자가 이 규칙을 기억하고 지킬 것"이라고 가정한다.

```tsx
// 로딩 시작
setIsLoading(true);
setIsError(false);
setIsSuccess(false);

// 성공
setIsLoading(false);
setIsError(false);
setIsSuccess(true);

// 에러
setIsLoading(false);
setIsError(true);
setIsSuccess(false);
```

매번 세 개의 상태를 올바르게 설정해야 한다.  
하나라도 잊으면 버그가 된다.

---

### 명시적 상태로 불가능한 상태 제거하기

같은 로직을 명시적 상태로 표현하면 이렇게 된다.

```tsx
type Status = 'idle' | 'loading' | 'success' | 'error';
const [status, setStatus] = useState<Status>('idle');

// 로딩 시작
setStatus('loading');

// 성공
setStatus('success');

// 에러
setStatus('error');
```

이제 불가능한 상태는  
타입 시스템이 막아준다.

- `loading`과 `success`가 동시에 true인 상태? → 불가능
- `error`와 `success`가 동시에 true? → 불가능
- 세 개의 상태를 동기화할 책임? → 사라짐

상태의 개수는 3개에서 1개로 줄었고,  
가능한 조합도 8가지에서 4가지로 줄었으며,  
그 4가지는 모두 의미 있는 상태다.

이것이 명시적 상태 설계의 힘이다.

> 불가능한 상태를 만들 수 없게 설계하면,  
> 동기화의 책임이 사라지고,  
> 버그 가능성이 구조적으로 제거된다.

---

## 파생 상태의 유혹

컴포넌트가 복잡해지는 또 다른 이유는  
**계산 가능한 값을 상태로 저장**하는 순간이다.

---

### 진짜 상태 vs 파생된 값

다음 두 코드를 비교해보자.

```tsx
// 파생 상태 (문제 있음)
const [items, setItems] = useState([]);
const [itemCount, setItemCount] = useState(0);

useEffect(() => {
  setItemCount(items.length);
}, [items]);
```

```tsx
// 파생 값 (권장)
const [items, setItems] = useState([]);
const itemCount = items.length;
```

두 번째 코드가 더 단순하고 안전하다.

`itemCount`는 진짜 상태가 아니다.  
그것은 `items`로부터 **계산 가능한 값**이다.

상태로 만드는 순간:
- 동기화 책임이 생긴다
- useEffect가 추가된다
- 버그 가능성이 생긴다 (동기화를 잊으면?)
- 컴포넌트 복잡도가 증가한다

---

### 언제 상태가 필요한가

그렇다면 언제 상태가 필요한가?

상태는 다음 조건을 만족할 때만 필요하다.

- **시간에 따라 변하고**
- **렌더링에 영향을 주며**
- **다른 값으로부터 계산할 수 없을 때**

이 조건을 만족하지 않는다면,  
그것은 상태가 아니라  
상수이거나, props이거나, 파생된 값이다.

```tsx
// 상태가 필요함
const [inputValue, setInputValue] = useState('');

// 상태가 필요 없음 (계산 가능)
const isEmpty = inputValue.length === 0;
const charCount = inputValue.length;
const isValid = inputValue.length >= 3;
```

파생 가능한 값을 상태로 만들지 않는 것,  
이것이 복잡도를 줄이는 첫걸음이다.

> 계산 가능한 값은 저장하지 마라.  
> 필요할 때 계산하라.

---

## Effect의 증식

상태가 늘어나고  
상태 간 의존성이 생기면  
자연스럽게 Effect도 늘어난다.

---

### Effect가 Effect를 낳는 순간

다음과 같은 패턴을 본 적이 있을 것이다.

```tsx
const [userId, setUserId] = useState(null);
const [user, setUser] = useState(null);
const [posts, setPosts] = useState([]);

useEffect(() => {
  if (userId) {
    fetchUser(userId).then(setUser);
  }
}, [userId]);

useEffect(() => {
  if (user) {
    fetchPosts(user.id).then(setPosts);
  }
}, [user]);
```

`userId` 변경 → `user` 업데이트 → `posts` 업데이트

이 연쇄 반응은  
각 Effect가 독립적으로 보이지만,  
실제로는 강하게 결합되어 있다.

- 실행 순서가 중요해진다
- 중간에 하나가 실패하면?
- user가 null이 되면 posts는?
- 의존성 배열 관리가 복잡해진다

Effect가 다른 상태를 업데이트하고,  
그 상태가 또 다른 Effect를 트리거하는 구조는  
복잡도를 가파르게 증가시킨다.

---

### 상태가 아니라 이벤트로 생각하기

많은 경우,  
Effect의 연쇄는 필요하지 않다.

```tsx
const [userId, setUserId] = useState(null);
const [user, setUser] = useState(null);
const [posts, setPosts] = useState([]);

const loadUserAndPosts = async (id) => {
  const userData = await fetchUser(id);
  setUser(userData);
  
  const postsData = await fetchPosts(userData.id);
  setPosts(postsData);
};

// userId 변경 시 한 번만 실행
useEffect(() => {
  if (userId) {
    loadUserAndPosts(userId);
  }
}, [userId]);
```

상태 변화에 반응하는 것이 아니라,  
명확한 이벤트(userId 변경)에 반응하는 구조가  
더 이해하기 쉽다.

---

## 읽기 어려운 컴포넌트의 신호들

지금까지 살펴본 문제들은  
컴포넌트에서 다음과 같은 신호로 나타난다.

### 구조적 신호

- useState가 5개 이상
- useEffect가 3개 이상
- useEffect 의존성 배열이 길다 (4개 이상)
- 조건부 렌더링이 3단계 이상 중첩
- 함수가 100줄 이상

### 변경의 신호

- 한 곳을 수정했는데 다른 곳이 깨진다
- 새 기능 추가 시 기존 코드를 여러 곳 수정해야 한다
- "이거 건드리면 무슨 일이 생길지 모르겠다"는 생각

### 이해의 신호

- 컴포넌트를 다시 읽는데 10분 이상 걸린다
- 상태가 어떻게 변하는지 추적하기 어렵다
- "이 상태가 왜 필요한지 모르겠다"

이런 신호들이 보인다면,  
그 컴포넌트는 이미  
읽기 어려운 상태에 도달했을 가능성이 크다.

---

## 복잡도를 낮추는 원칙

복잡도를 낮추기 위한 원칙은  
복잡하지 않다.

### 1. 상태를 최소화하라

```tsx
// ❌ 나쁨: 파생 상태
const [items, setItems] = useState([]);
const [count, setCount] = useState(0);

// ✅ 좋음: 계산
const [items, setItems] = useState([]);
const count = items.length;
```

### 2. 불가능한 상태를 제거하라

```tsx
// ❌ 나쁨: 8가지 조합
const [isLoading, setIsLoading] = useState(false);
const [isError, setIsError] = useState(false);
const [isSuccess, setIsSuccess] = useState(false);

// ✅ 좋음: 4가지, 모두 유효
type Status = 'idle' | 'loading' | 'success' | 'error';
const [status, setStatus] = useState<Status>('idle');
```

### 3. 상태 간 의존성을 명시하라

```tsx
// ❌ 나쁨: 암시적 관계
const [isEditing, setIsEditing] = useState(false);
const [editValue, setEditValue] = useState('');

// ✅ 좋음: 구조로 관계 표현
const [mode, setMode] = useState<
  | { type: 'viewing' }
  | { type: 'editing'; value: string }
>({ type: 'viewing' });
```

### 4. Effect 연쇄를 끊어라

```tsx
// ❌ 나쁨: Effect가 Effect를 트리거
useEffect(() => {
  setB(computeB(a));
}, [a]);

useEffect(() => {
  setC(computeC(b));
}, [b]);

// ✅ 좋음: 이벤트에서 한 번에
const handleChange = (newA) => {
  setA(newA);
  const newB = computeB(newA);
  setB(newB);
  const newC = computeC(newB);
  setC(newC);
};
```

---

## 정리하며 — 복잡도는 상태에서 시작된다

컴포넌트가 읽기 어려워지는 이유는  
복잡한 로직이나 긴 코드가 아니다.

진짜 이유는  
**상태의 설계**에 있다.

- 상태가 많을수록 조합이 폭발한다
- 상태 간 결합이 생기면 동기화 책임이 생긴다
- 파생 상태는 Effect를 낳는다
- Effect가 늘면 실행 순서와 의존성 추적이 어려워진다
- 암시적 상태는 불가능한 상태를 허용한다

컴포넌트 복잡도는  
기술의 문제가 아니라  
**상태를 어떻게 모델링했는지**의 문제다.

> 컴포넌트가 복잡하다면,  
> 먼저 상태를 다시 보라.  
> 그것은 정말 상태여야 하는가?  
> 파생 가능하지 않은가?  
> 더 명시적으로 만들 수 없는가?

좋은 컴포넌트는  
적은 코드를 가진 컴포넌트가 아니라,  
**적은 상태를 가진 컴포넌트**다.

상태가 줄어들면,  
복잡도는 자연스럽게 따라온다.

이 문서는  
컴포넌트를 단순하게 만드는 방법이 아니라,  
**무엇이 컴포넌트를 복잡하게 만드는지**를  
다시 이해하기 위한 기록이다.
