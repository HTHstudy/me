---
layout: default
title: 추상화
parent: Mental Model
nav_order: 3
permalink: /docs/mental-model/abstraction
---

# 추상화: 무엇을 하나로 묶을 것인가

## 들어가며 — 중복이 보이면 자연스럽게

컴포넌트를 나누고 나면  
자연스럽게 이런 상황을 마주하게 된다.

```tsx
// 저장 버튼
export const SaveButton = () => {
  return (
    <button className="btn btn-primary">
      저장
    </button>
  );
};

// 제출 버튼
export const SubmitButton = () => {
  return (
    <button className="btn btn-primary">
      제출
    </button>
  );
};

// 완료 버튼
export const CompleteButton = () => {
  return (
    <button className="btn btn-primary">
      완료
    </button>
  );
};
```

구조가 거의 같다.  
`btn btn-primary` 스타일이 반복된다.

이때 자연스럽게 이런 생각이 든다.

> "중복이 많네. 이거 하나로 묶으면 되지 않을까?"

이것은 좋은 본능이다.  
중복을 줄이고,  
일관성을 유지하며,  
한 곳만 수정하면 모든 곳에 적용된다.

그래서 우리는 추상화를 시작한다.

```tsx
export const PrimaryButton = ({ children }) => {
  return (
    <button className="btn btn-primary">
      {children}
    </button>
  );
};

// 사용
<PrimaryButton>저장</PrimaryButton>
<PrimaryButton>제출</PrimaryButton>
<PrimaryButton>완료</PrimaryButton>
```

중복이 사라졌다.  
코드가 깔끔해졌다.

이 문서는  
이 자연스러운 선택이  
**무엇을 의미하는지**,  
그리고 **무엇을 묶어야 하는지**를  
다시 생각해보기 위해 쓰였다.

---

## 추상화의 본질: 여러 구현을 하나로

추상화를 한다는 것은  
여러 개의 구체적인 것 위에  
하나의 이름과 인터페이스를 올리는 것이다.

```tsx
// 구체적인 것들
SaveButton, SubmitButton, CompleteButton

// 하나의 추상
PrimaryButton
```

이 순간 우리는 하나의 선언을 한다.

> "이 셋은 같은 것이다."

더 정확히는:

> "이 셋은 **같은 방식으로 다룰 수 있는** 것이다."

추상화는 차이를 없애는 것이 아니다.  
차이를 **매개변수로 표현**하는 것이다.

```tsx
// 차이: 텍스트 내용
<PrimaryButton>저장</PrimaryButton>
<PrimaryButton>제출</PrimaryButton>
```

이제 SaveButton, SubmitButton, CompleteButton은  
"PrimaryButton의 인스턴스"가 되었다.

이것이 추상화의 본질이다.

> 추상화는  
> 여러 구현을 하나의 개념으로 묶고,  
> 그 차이를 매개변수로 표현하는 것이다.

---

## 추상화를 하는 이유

추상화를 하는 이유는 명확하다.

### 중복 제거

같은 코드를 여러 번 쓰지 않아도 된다.

```tsx
// 추상화 전: 3번 반복
<button className="btn btn-primary">저장</button>
<button className="btn btn-primary">제출</button>
<button className="btn btn-primary">완료</button>

// 추상화 후: 1번만
<PrimaryButton>저장</PrimaryButton>
<PrimaryButton>제출</PrimaryButton>
<PrimaryButton>완료</PrimaryButton>
```

### 일관성 유지

스타일이 바뀌어도 한 곳만 수정하면 된다.

```tsx
// PrimaryButton 컴포넌트만 수정
export const PrimaryButton = ({ children }) => {
  return (
    <button className="btn btn-primary-v2">  {/* 여기만 바꾸면 */}
      {children}
    </button>
  );
};

// 모든 버튼에 자동 적용
```

### 변경의 용이성

새로운 기능을 추가할 때도  
한 곳에만 추가하면 된다.

```tsx
// PrimaryButton에 클릭 효과 추가
export const PrimaryButton = ({ children, onClick }) => {
  return (
    <button 
      className="btn btn-primary"
      onClick={onClick}
      onMouseDown={(e) => e.currentTarget.classList.add('active')}  {/* 모든 버튼에 추가됨 */}
    >
      {children}
    </button>
  );
};
```

이것이 추상화의 가치다.

> 추상화는 중복을 줄이고,  
> 일관성을 유지하며,  
> 변경을 쉽게 만든다.

---

## 추상화는 미래를 고정하는 선택이다

추상화를 하는 순간,  
우리는 하나의 가정을 한다.

> "이것들은 **앞으로도** 같은 방식으로 변할 것이다."

이것은 단순한 관찰이 아니라  
**미래에 대한 예측**이다.

```tsx
// 추상화 = 이런 선언
export const PrimaryButton = ({ children }) => {
  // "모든 Primary 버튼은 같은 스타일을 가진다"
  // "이 버튼들은 앞으로도 함께 변할 것이다"
  return (
    <button className="btn btn-primary">
      {children}
    </button>
  );
};
```

이 가정이 맞을 때,  
추상화는 구조를 단순하게 만든다.

하지만 이 가정이 틀리는 순간,  
추상화는 변화를 막는 장벽이 된다.

```tsx
// 새 요구사항: "저장 버튼만 크게 만들어주세요"
// 새 요구사항: "제출 버튼에만 아이콘을 넣어주세요"
// 새 요구사항: "완료 버튼은 초록색이어야 해요"

// 추상화가 이를 막는다
```

추상화는  
현재를 정리하는 것처럼 보이지만,  
실제로는 **미래를 고정**하는 선택이다.

---

## 중복과 추상화는 다르다

추상화를 할 때  
가장 흔한 근거는 "중복"이다.

> "코드가 중복되니까 추상화하자."

하지만 중복과 추상화는  
서로 다른 문제다.

### 중복은 현재의 관찰이다

```tsx
// 지금 이 코드들이 비슷하다
<div className="btn">클릭</div>
<div className="btn">저장</div>
<div className="btn">취소</div>
```

중복은 **지금 이 순간**의 상태를 말한다.  
"현재 이 코드들이 비슷하다"는 관찰이다.

### 추상화는 미래의 가정이다

```tsx
// 이것들을 하나로 묶는다
<Button>클릭</Button>
<Button>저장</Button>
<Button>취소</Button>
```

추상화를 하는 순간,  
우리는 이렇게 가정한다.

> "이것들은 **앞으로도** 같은 방식으로 변할 것이다."

### 중복의 종류

| 구분 | 중복 | 추상화 |
|-----|------|--------|
| 시점 | 현재 | 미래 |
| 의미 | 지금 비슷하다 | 앞으로도 같이 변한다 |
| 판단 | 관찰 | 가정 |

**중복을 제거한다고 해서  
항상 추상화가 필요한 것은 아니다.**

지금 비슷해 보이는 것이  
앞으로도 같을 것이라는 보장은 없다.

---

## 우연한 중복 vs 본질적 중복

모든 중복이 같은 것은 아니다.

### 우연한 중복

```tsx
// 사용자 프로필 페이지
export const UserProfile = () => {
  const [data, setData] = useState(null);
  
  useEffect(() => {
    fetch('/api/user').then(res => res.json()).then(setData);
  }, []);
  
  if (!data) return <Loading />;
  return <div>{data.name}</div>;
};

// 상품 상세 페이지
export const ProductDetail = () => {
  const [data, setData] = useState(null);
  
  useEffect(() => {
    fetch('/api/product').then(res => res.json()).then(setData);
  }, []);
  
  if (!data) return <Loading />;
  return <div>{data.title}</div>;
};
```

이 둘은 코드가 거의 같다.  
하지만:

- 하나는 **사용자** 데이터
- 하나는 **상품** 데이터
- API 엔드포인트가 다름
- 표시하는 정보가 다름
- 변경 이유가 다름

이것은 **우연히 비슷한** 것이지,  
본질적으로 같은 것이 아니다.

### 본질적 중복

```tsx
// Primary 버튼들
<button className="btn btn-primary">저장</button>
<button className="btn btn-primary">제출</button>
<button className="btn btn-primary">완료</button>
```

이것들은:
- 같은 **역할**을 한다 (버튼)
- 같은 **스타일**을 가진다
- 같은 **규칙**을 따른다
- **함께 변경**되어야 한다

이것은 **본질적으로 같은** 것이다.

### 판단 기준

| 질문 | 우연한 중복 | 본질적 중복 |
|-----|-----------|-----------|
| 같은 역할을 하는가? | ❌ | ✅ |
| 함께 변경되는가? | ❌ | ✅ |
| 같은 규칙을 따르는가? | ❌ | ✅ |
| 같은 의미를 가지는가? | ❌ | ✅ |

우연한 중복은 추상화하지 않는다.  
본질적 중복만 추상화한다.

---

## 무엇을 추상화할 것인가

추상화를 결정하기 전에  
던져야 할 질문들이 있다.

### 추상화 판단 기준

| 질문 | Yes | No |
|-----|-----|-----|
| **같은 역할**을 한다고 말할 수 있는가? | 추상화 가능 | 우연한 중복 가능성 |
| **함께 변경**될 것 같은가? | 추상화 고려 | 독립적으로 유지 |
| 사용하는 **데이터와 규칙**이 같은가? | 추상화 가능 | 별도 관리 필요 |
| **한 문장**으로 설명할 수 있는가? | 명확한 추상화 | 아직 이르다 |
| 지금 추상화하지 않으면 **문제**가 생기는가? | 고려 가치 있음 | 미뤄도 됨 |

"No"가 많을수록  
추상화는 시기상조다.

### 예시: 버튼

```tsx
// 질문: 이것들을 추상화해야 하나?
<button onClick={handleSave}>저장</button>
<button onClick={handleCancel}>취소</button>
<button onClick={handleSubmit}>제출</button>
```

**질문 1: 같은 역할인가?**  
→ Yes. 모두 "버튼"이다.

**질문 2: 함께 변경되는가?**  
→ Yes. 버튼 스타일이 바뀌면 모두 바뀌어야 한다.

**질문 3: 한 문장으로 설명되는가?**  
→ Yes. "클릭 가능한 버튼"

**결론:** 추상화할 가치가 있다.

### 예시: 데이터 페칭

```tsx
// 질문: 이것들을 추상화해야 하나?
useEffect(() => { fetch('/api/user').then(setUser); }, []);
useEffect(() => { fetch('/api/product').then(setProduct); }, []);
```

**질문 1: 같은 역할인가?**  
→ No. 하나는 사용자, 하나는 상품 데이터

**질문 2: 함께 변경되는가?**  
→ No. 독립적으로 변경될 가능성이 높음

**질문 3: 한 문장으로 설명되는가?**  
→ "데이터 페칭"? 너무 모호함

**결론:** 우연한 중복. 추상화하지 않는다.

---

## 좋은 추상화의 특징

좋은 추상화는 명확한 특징을 가진다.

### 1. 이름만 봐도 역할이 명확하다

```tsx
// 좋음
<Button>클릭</Button>
<Card>내용</Card>
<Modal>팝업</Modal>

// 나쁨
<Component type="button">클릭</Component>
<Container variant="card">내용</Container>
<Wrapper mode="modal">팝업</Wrapper>
```

### 2. 최소한의 props만 가진다

```tsx
// 좋음
<Button variant="primary" onClick={handleClick}>
  저장
</Button>

// 나쁨
<Button 
  type="button"
  variant="primary"
  size="medium"
  shape="rounded"
  elevation="2"
  ripple={true}
  disabled={false}
  loading={false}
  onClick={handleClick}
>
  저장
</Button>
```

### 3. 책임이 하나다

```tsx
// 좋음: 레이아웃만
export const PageLayout = ({ children }) => {
  return <div className="page">{children}</div>;
};

// 나쁨: 레이아웃 + 데이터 + 로직
export const Page = ({ type, userId, onSave }) => {
  const data = useFetch(`/api/${type}/${userId}`);
  // ... 복잡한 로직
};
```

### 4. 사용처를 예측할 수 있다

```tsx
// 좋음: 명확한 사용처
<PrimaryButton onClick={handleSave}>저장</PrimaryButton>

// 나쁨: 불명확
<Button 
  as="a"
  href="/home"
  variant="primary"
  size="large"
/>
```

---

## 추상화는 발견하는 것이다

좋은 추상화는  
처음부터 설계하는 것이 아니라  
사용 패턴에서 **발견**하는 것이다.

### 1단계: 중복 없이 작성

```tsx
// 저장 버튼
export const SaveButton = () => {
  return (
    <button className="btn btn-primary">
      저장
    </button>
  );
};
```

아직 다른 버튼이 없다면  
추상화하지 않는다.

### 2단계: 2-3번 반복되면 패턴 관찰

```tsx
// 저장 버튼
<button className="btn btn-primary">저장</button>

// 제출 버튼
<button className="btn btn-primary">제출</button>

// 완료 버튼
<button className="btn btn-primary">완료</button>
```

비슷한 패턴이 보인다.  
하지만 아직 추상화하지 않는다.

**차이를 관찰한다:**
- 무엇이 같은가? (btn, btn-primary 클래스)
- 무엇이 다른가? (버튼 텍스트)
- 왜 다른가? (각 버튼의 역할)

### 3단계: 차이의 본질을 이해한 후 추상화

관찰 결과:
- 스타일은 **항상 같다**
- 텍스트는 **각 버튼마다 다르다**
- 이 스타일은 **앞으로도 유지될 것 같다**

이제 추상화한다.

```tsx
export const PrimaryButton = ({ children }) => {
  return (
    <button className="btn btn-primary">
      {children}
    </button>
  );
};
```

최소한의 props만으로  
차이를 표현할 수 있다.

### 4단계: 새 요구사항이 생기면 재평가

```tsx
// 새 요구사항: "버튼에 클릭 핸들러가 필요해요"
```

선택지:
1. props 추가: `onClick?: () => void`
2. 추상화 재검토: 모든 버튼이 onClick이 필요한가?

**판단:**
- 모든 버튼이 클릭 가능해야 한다면 → props 추가
- 일부만 클릭 가능하다면 → 별도 컴포넌트

추상화는 고정된 것이 아니라  
요구사항에 따라 **진화**한다.

---

## Rule of Three: 세 번째에 추상화하라

추상화를 서두르지 않기 위한 간단한 규칙이 있다.

```
1회: 작성
2회: 복사 (중복 발생)
3회: 추상화 (패턴 확인)
```

### 왜 세 번째인가?

**한 번:** 패턴이 아니다. 그냥 하나의 구현이다.

**두 번:** 우연일 수 있다. 아직 패턴이라 보기 어렵다.

**세 번:** 이제 패턴이 드러난다.

```tsx
// 1회
<button className="btn btn-primary">저장</button>

// 2회: 복사
<button className="btn btn-primary">제출</button>

// 3회: 패턴 확인 → 이제 추상화
export const PrimaryButton = ({ children }) => {
  return <button className="btn btn-primary">{children}</button>;
};
```

세 번째에 추상화하면:
- 패턴이 충분히 드러났다
- 차이가 무엇인지 명확하다
- 우연한 중복인지 본질적 중복인지 판단 가능

---

## 중복을 두려워하지 마라

추상화를 서두르는 이유 중 하나는  
중복에 대한 두려움이다.

하지만 중복이 항상 나쁜 것은 아니다.

### 잘못된 추상화보다 중복이 낫다

```tsx
// 중복: 독립적
<button className="btn-save">저장</button>
<button className="btn-save">저장</button>

// 잘못된 추상화: 결합됨
<Button 
  type="save"
  specialCaseForPageA={true}
  customStyleForModal={true}
/>
```

중복은:
- 명확하다
- 독립적이다
- 되돌리기 쉽다

잘못된 추상화는:
- 모호하다
- 결합되어 있다
- 되돌리기 어렵다

### 중복의 비용 vs 추상화의 비용

| 구분 | 중복 | 잘못된 추상화 |
|-----|------|-------------|
| 명확성 | 높음 | 낮음 |
| 결합도 | 낮음 | 높음 |
| 변경 용이성 | 쉬움 | 어려움 |
| 되돌리기 | 쉬움 | 어려움 |

의심스러울 때는  
추상화보다 중복을 택하라.

중복은 나중에도 제거할 수 있지만,  
잘못된 추상화는 되돌리기 훨씬 어렵다.

---

## 추상화와 관심사 분리

추상화를 할 때  
관심사 분리를 잊으면 안 된다.

### 추상화는 경계를 흐릴 수 있다

```tsx
// 나쁨: 레이아웃 + 데이터 + 로직
export const Page = ({ type }) => {
  const data = useFetch(`/api/${type}`);
  
  if (type === 'user') {
    // 사용자 로직
  }
  
  if (type === 'product') {
    // 상품 로직
  }
  
  return <div>{/* ... */}</div>;
};
```

이것은 추상화가 아니라  
**모든 것을 한곳에 모은 것**이다.

### 좋은 추상화는 경계를 지킨다

```tsx
// 좋음: 레이아웃만
export const PageLayout = ({ children }) => {
  return <div className="page">{children}</div>;
};

// 사용
export const UserPage = () => {
  const user = useUser();  // 사용자 로직
  return (
    <PageLayout>
      <UserProfile user={user} />
    </PageLayout>
  );
};

export const ProductPage = () => {
  const product = useProduct();  // 상품 로직
  return (
    <PageLayout>
      <ProductDetail product={product} />
    </PageLayout>
  );
};
```

레이아웃은 레이아웃만,  
데이터는 각 페이지가 책임진다.

---

## 정리하며 — 추상화는 선택이다

추상화는  
중복을 보면 자동으로 하는 것이 아니라,  
의식적으로 선택해야 하는 것이다.

> 추상화는  
> 여러 구현을 하나로 묶고,  
> "앞으로도 같을 것"이라고 가정하는 선택이다.

### 추상화하기 전에

- 이것들은 **정말 같은** 것인가?
- **우연히** 비슷한 것은 아닌가?
- **함께 변경**될 것인가?
- **한 문장**으로 설명되는가?

### 좋은 추상화는

- 이름만 봐도 역할이 명확하고
- 최소한의 props만 가지며
- 책임이 하나이고
- 사용처를 예측할 수 있다

### 추상화는

- 설계하는 것이 아니라 **발견**하는 것
- 처음부터 만드는 것이 아니라 **진화**하는 것
- 중복을 두려워해서 하는 것이 아니라 **가치가 있을 때** 하는 것

중복을 줄이는 것도 중요하지만,  
더 중요한 것은  
**무엇을 묶고 무엇을 나눌 것인지**를  
명확히 판단하는 것이다.

추상화는  
코드를 줄이는 기술이 아니라,  
**경계를 정의하는 선택**이다.
