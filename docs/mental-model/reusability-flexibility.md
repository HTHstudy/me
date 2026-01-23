---
layout: default
title: 재사용과 유연성
parent: Mental Model
nav_order: 4
permalink: /docs/mental-model/reusability-flexibility
---

# 재사용과 유연성

> “우리는 언제 ‘재사용해야겠다’고 판단하는가  
> 재사용하려다 왜 복잡해지는가  
> 유연성이 커질수록 무엇이 흐려지는가”

## 들어가며 — 재사용 가능한 컴포넌트를 만들자

실무에서 컴포넌트를 작성하다 보면  
자연스럽게 이런 생각이 든다.

> "이 컴포넌트, 다른 곳에서도 쓸 수 있게 만들면 좋겠는데?"

이것은 좋은 의도다.  
중복을 줄이고,  
일관성을 유지하며,  
한 번 잘 만들어두면 계속 쓸 수 있다.

그래서 우리는 컴포넌트를 **재사용 가능하게** 만들기 시작한다.

```tsx
// 처음: 단순한 페이지 레이아웃
export const Page = ({ title, children }: { title: string; children: ReactNode }) => {
  return (
    <div className="container">
      <h1>{title}</h1>
      <div className="content">{children}</div>
    </div>
  );
};
```

이 페이지는 단순하고 명확하다.  
하지만 곧 다른 요구사항이 들어온다.

> "페이지마다 너비가 달라야 해요."

```tsx
export const Page = ({ 
  title,
  children,
  width = 'default'
}: { 
  title: string;
  children: ReactNode;
  width?: 'default' | 'wide' | 'narrow';
}) => {
  return (
    <div className={`container container-${width}`}>
      <h1>{title}</h1>
      <div className="content">{children}</div>
    </div>
  );
};
```

괜찮아 보인다. 그리고 또 다른 요구사항.

> "제목 크기도 조절할 수 있어야 해요."  
> "로딩 상태도 보여줘야 해요."  
> "푸터가 필요한 페이지도 있어요."  
> "배경색이 다른 페이지도 있어요."

```tsx
export const Page = ({
  title,
  children,
  width = 'default',
  titleSize = 'large',
  loading = false,
  footer,
  backgroundColor,
  padding,
}: PageProps) => {
  // ... 복잡한 로직
};
```

어느 순간부터  
이 페이지는 더 이상 단순하지 않다.

각 요구사항은 모두 합리적이었다.  
그리고 각 props를 추가하는 순간에는  
여전히 관리 가능해 보였다.

하지만 시간이 지나면서  
이 컴포넌트를 수정하는 것이  
점점 두려워지기 시작했다.

이 문서는  
**재사용 가능하게 만들려는 선의가  
왜 복잡도로 이어지는지**,  
그 관계를 다시 이해하기 위해 쓰였다.

---

## 재사용성의 전제: 차이를 흡수해야 한다

재사용 가능한 컴포넌트를 만든다는 것은  
여러 곳에서 사용될 수 있다는 의미다.

하지만 각 사용처는  
조금씩 다른 요구사항을 가진다.

- A에서는 큰 버튼이 필요하고
- B에서는 작은 버튼이 필요하며
- C에서는 로딩 표시가 필요하고
- D에서는 아이콘이 필요하다

재사용하려면  
이 **차이들을 하나의 컴포넌트가 감당**해야 한다.

이것이 재사용성의 전제다.

> 재사용 가능하다는 것은  
> 여러 사용처의 차이를  
> 하나의 인터페이스로 흡수할 수 있다는 의미다.

그리고 이 차이를 흡수하는 방법은  
대부분 **유연성을 높이는 것**이다.

---

## 유연성의 도구: Props와 옵션

차이를 흡수하는 가장 일반적인 방법은  
props를 추가하는 것이다.

```tsx
// 재사용 1회차
<Page title="홈">...</Page>

// 재사용 2회차: 차이 발생
<Page title="프로필" width="wide">...</Page>

// 재사용 3회차: 더 많은 차이
<Page title="설정" width="narrow" loading={true}>...</Page>

// 재사용 4회차: 계속 늘어나는 차이
<Page 
  title="대시보드"
  width="wide"
  titleSize="large"
  loading={isLoading}
  footer={<Footer />}
  backgroundColor="#f5f5f5"
>
  ...
</Page>
```

재사용할 때마다  
새로운 차이가 드러나고,  
그 차이를 흡수하기 위해  
props가 하나씩 추가된다.

이것이 유연성이다.

> 유연성이란  
> 하나의 컴포넌트가  
> 여러 사용처의 차이를 수용할 수 있는 능력이다.

유연성은 재사용성의 전제 조건이다.  
유연하지 않으면 재사용할 수 없다.

하지만 여기서 문제가 시작된다.

---

## 유연성의 비용: 조합 폭발

props가 하나 추가될 때마다  
가능한 조합의 수는 기하급수적으로 늘어난다.

```tsx
type PageProps = {
  title: string;
  width?: 'default' | 'wide' | 'narrow' | 'full';
  titleSize?: 'small' | 'medium' | 'large';
  loading?: boolean;
  footer?: ReactNode;
  backgroundColor?: string;
  padding?: 'none' | 'small' | 'medium' | 'large';
};
```

이 인터페이스가 만들어내는 조합:
- width: 4가지
- titleSize: 3가지
- loading: 2가지
- footer: 있음/없음 2가지
- backgroundColor: 무한대
- padding: 4가지

**총 가능한 조합: 4 × 3 × 2 × 2 × ∞ × 4 = 거의 무한대**

이 중에서  
실제로 유효한 조합은 몇 개인가?

- `width: 'full'`인데 `padding: 'large'`는 말이 되나?
- `loading: true`인데 `footer`를 보여야 하나?
- `titleSize: 'large'`인데 `width: 'narrow'`는 디자인에 존재하나?

무수히 많은 조합 중  
실제로 유효한 조합은 아마 10-20가지 정도일 것이다.

하지만 타입 시스템은  
나머지 수백 개의 불가능한 조합도 허용한다.

이것이 유연성의 첫 번째 비용이다.

> 유연성이 높아질수록,  
> 가능한 조합은 기하급수적으로 증가하며,  
> 그 중 대부분은 의미 없거나 불가능한 상태다.

---

## 복잡도의 시작: 조건부 로직

조합이 늘어나면  
내부 구현도 복잡해진다.

```tsx
export const Page = (props: PageProps) => {
  const {
    title,
    children,
    width = 'default',
    titleSize = 'large',
    loading = false,
    footer,
    backgroundColor,
    padding = 'medium',
  } = props;

  // 조건부 클래스명
  const className = classNames(
    'container',
    `container-${width}`,
    `padding-${padding}`,
    {
      'container-loading': loading,
    }
  );

  const titleClassName = classNames(
    'title',
    `title-${titleSize}`,
  );

  // 조건부 스타일
  const style = {
    ...(backgroundColor && { backgroundColor }),
  };

  // 조건부 렌더링
  const content = loading ? (
    <div className="loading-overlay">
      <Spinner />
    </div>
  ) : (
    <>
      <h1 className={titleClassName}>{title}</h1>
      <div className="content">{children}</div>
      {footer && <div className="footer">{footer}</div>}
    </>
  );

  return (
    <div className={className} style={style}>
      {content}
    </div>
  );
};
```

props가 7개인데  
조건부 로직은 10개가 넘는다.

각 props의 조합에 따라  
다른 동작을 해야 하기 때문이다.

이것이 유연성의 두 번째 비용이다.

> 유연성이 높아질수록,  
> 그 유연성을 구현하기 위한  
> 조건부 로직이 증가한다.

---

## 재사용의 역설: 한 곳을 고치면 모든 곳이 영향받는다

재사용 가능한 컴포넌트의 목표는  
"한 번 고치면 모든 곳에 적용"이다.

하지만 이것은 양날의 검이다.

```tsx
// 홈 페이지에서 사용
<Page title="홈" width="wide">
  ...
</Page>

// 설정 페이지에서 사용
<Page title="설정" width="narrow">
  ...
</Page>
```

만약 `width="wide"`의 너비를 바꾸면?  
홈 페이지도 함께 바뀐다.

이것이 재사용의 장점이다.

하지만 만약 홈 페이지만 바꾸고 싶다면?  
더 이상 재사용 가능한 Page로는 해결할 수 없다.

그래서 다시 props를 추가한다.

```tsx
<Page 
  title="홈"
  width="wide"
  customWidth="1400px"  // 예외 처리
>
  ...
</Page>
```

재사용을 위해 만든 컴포넌트가  
예외 처리를 위한 props를 계속 추가하게 된다.

이것이 재사용의 역설이다.

> 재사용성이 높을수록,  
> 한 곳의 변경이 모든 곳에 영향을 주며,  
> 예외를 처리하기 위한 props가 계속 늘어난다.

---

## 유연성의 함정: 의미가 흐려진다

props가 늘어날수록  
컴포넌트의 의미가 흐려진다.

```tsx
// 이것은 무엇인가?
<Page 
  title="페이지"
  width="default"
  titleSize="medium"
  loading={false}
  footer={null}
  backgroundColor={undefined}
  padding="medium"
>
  내용
</Page>

// 이것과 다른가?
<Page title="페이지">
  내용
</Page>
```

두 페이지는 결과적으로 같다.  
하지만 첫 번째는  
명시적으로 모든 옵션을 지정했다.

이게 더 명확한가?  
아니면 더 복잡한가?

props가 많아질수록,  
"이 컴포넌트가 무엇을 하는 것인지"가  
props의 조합에 의해 결정된다.

```tsx
// 이것은 페이지인가? 모달인가?
<Page 
  title="알림"
  width="narrow"
  floating={true}
  overlay={true}
  closeButton={true}
>
  내용
</Page>
```

이 순간부터  
Page는 더 이상 "페이지"가 아니다.

그것은 "페이지처럼 보일 수도 있고  
모달처럼 보일 수도 있는  
무언가"가 된다.

이것이 유연성의 함정이다.

> 유연성이 높아질수록,  
> 컴포넌트의 정체성은 흐려지고,  
> "무엇이든 될 수 있는 것"이 된다.

---

## 재사용과 복잡도의 관계

지금까지 살펴본 것을 정리하면  
이런 관계가 드러난다.

```
재사용하려면
  ↓
차이를 흡수해야 하고
  ↓
유연성이 필요하며
  ↓
Props/옵션이 늘어나고
  ↓
조합이 폭발하며
  ↓
조건부 로직이 증가하고
  ↓
복잡도가 증가한다
```

이 관계는 많은 팀에서 반복적으로 관찰된다.

재사용성과 복잡도는  
서로를 끌어당기는 긴장에 놓인다.

> 재사용의 범위가 넓어질수록 유연성이 요구되고,  
> 유연성이 커질수록 복잡도가 늘어나는 흐름이 보인다.

이 흐름은 특정 선택의 잘못이라기보다  
재사용이 작동하는 방식에 가깝다.

---

## 그렇다면 재사용하지 말아야 하나?

여기까지 읽으면  
이런 질문이 자연스럽다.

> "그럼 재사용 가능한 컴포넌트를 만들지 말아야 하나?"

그렇게 느껴지기도 한다.

재사용하지 않으면  
중복이 늘어나고,  
일관성이 깨지며,  
유지보수 비용이 증가한다.

문제는 재사용 자체라기보다,  
**무엇을 재사용 대상으로 삼는가**라는 경계에 가깝다.

---

## 재사용할 만한 가치가 있는가?

재사용 가능한 컴포넌트를 만들기 전에  
자주 떠오르는 질문들이 있다.

### 1. 이 차이들은 정말 같은 것의 변형인가?

```tsx
// 같은 것의 변형
<Button variant="primary">저장</Button>
<Button variant="secondary">취소</Button>
```

이 둘은 같은 "버튼"으로 보이기 쉽다.  
차이는 색상에만 있는 듯하다.

```tsx
// 정말 같은 것인가?
<Button>클릭</Button>
<Button as="a" href="/home">홈으로</Button>
```

하나는 버튼이고  
하나는 링크다.

겉모습이 비슷하더라도  
같은 것의 변형인지 다시 묻게 된다.

### 2. 변경이 항상 함께 일어나는가?

```tsx
<PrimaryButton>저장</PrimaryButton>
<PrimaryButton>제출</PrimaryButton>
<PrimaryButton>완료</PrimaryButton>
```

이 세 버튼의 스타일이  
항상 함께 바뀌어야 한다고 느껴진다면,  
하나로 묶는 선택이 자연스럽게 떠오른다.

하지만 만약  
"저장 버튼만 다르게 해주세요"라는 요구가  
자주 들어온다면?

그 경우에는  
묶어둠이 오히려 긴장을 키울 수 있다.

### 3. 예외를 위한 props가 늘어나고 있는가?

```tsx
<Button
  variant="primary"
  size="large"
  specialCaseForPageA={true}  // 🚨
  disableHoverForMobile={true}  // 🚨
  customPaddingForModal={16}   // 🚨
>
  저장
</Button>
```

이런 props가 생기기 시작하면  
재사용의 범위를 다시 보게 된다.

---

## 재사용의 범위가 흐려질 때

재사용 가능한 컴포넌트가  
버거워지는 이유는 대부분  
**너무 많은 차이를 한 번에 흡수하려 하기 때문**이다.

### 범위가 명확한 재사용

```tsx
// Design System의 기본 레이아웃
// 범위: 전체 애플리케이션
// 책임: 일관된 페이지 구조
export const PageLayout = ({
  children,
  ...props
}: PageLayoutProps) => {
  return (
    <div className="page-layout" {...props}>
      {children}
    </div>
  );
};
```

이 레이아웃은  
**구조적 일관성**에만 집중한다.

제목, 로딩, 특수한 스타일 등은  
각 도메인에서 이 레이아웃을 조합해서 만든다.

```tsx
// 특정 도메인의 Dashboard 페이지
// 범위: 대시보드 관련 기능
// 책임: 대시보드 로직과 상태
export const DashboardPage = () => {
  const { data, isLoading } = useDashboard();
  
  return (
    <PageLayout>
      <h1>대시보드</h1>
      {isLoading ? <Spinner /> : <DashboardContent data={data} />}
    </PageLayout>
  );
};
```

이제 재사용의 범위가 상대적으로 또렷해진다.

- `PageLayout`: 구조 재사용
- `DashboardPage`: 대시보드 로직 재사용

각각은 서로 다른 범위와 책임을 가진다.

---

## 유연성의 적정선

유연성은 필요하지만,  
**어디까지 유연해야 하는가**는  
사용 범위에 따라 달라지기 쉽다.

### 범위가 넓게 읽히는 경우

```tsx
// 예시: 범위가 넓게 읽히는 레이아웃
// 유연성: 낮게 유지되는 장면
// 이유: 일관성이 더 크게 작용함
type PageLayoutProps = {
  children: ReactNode;
  maxWidth?: 'default' | 'wide';
};
```

1-2개의 props만으로  
대부분의 경우가 정리되는 장면이 많다.

더 복잡한 요구사항은  
이 레이아웃을 조합하는 방식으로 풀리곤 한다.

범위가 특정 기능에 가까워질수록  
필요한 유연성이 늘어나되,  
과도한 옵션은 금방 부담으로 느껴지곤 한다.

### 범위가 좁게 읽히는 경우

```tsx
// 예시: 특정 사용자의 프로필 페이지
// 유연성: 낮게 유지되는 장면
// 이유: 재사용이 전제되지 않음
export const UserProfilePage = () => {
  const { user, isLoading, handleSave } = useUserProfile();
  
  return (
    <PageLayout>
      <h1>{user.name}의 프로필</h1>
      {isLoading ? <Spinner /> : <ProfileForm user={user} onSave={handleSave} />}
    </PageLayout>
  );
};
```

재사용을 전제로 하지 않으면  
유연성이 의미를 잃기도 한다.

---

## 중복과 재사용의 균형

재사용을 강조하다 보면  
중복을 극도로 피하려 한다.

하지만 중복이 항상 나쁜 것으로 보이지는 않는다.

### 의미 있는 중복

```tsx
// 홈 페이지
<div className="container">
  <h1>홈</h1>
  <div className="content">...</div>
</div>

// 설정 페이지
<div className="container">
  <h1>설정</h1>
  <div className="content">...</div>
</div>
```

이 두 페이지는 구조가 거의 같다.

하지만 만약:
- 홈 페이지는 자주 변경되고
- 설정 페이지는 안정적이라면?

이 둘을 하나로 합치는 순간,  
홈 페이지의 변경이 설정 페이지에 영향을 주게 된다.

이 경우 중복을 유지하는 것이  
긴장을 낮추는 선택일 수 있다.

### 중복을 두고 갈리는 지점

중복을 줄여도 괜찮아 보이는 경우:
- **항상 함께 변경**되는 장면이 많다
- **같은 의미**로 읽히는 순간이 많다
- **같은 규칙**이 반복되는 편이다

중복을 남겨두는 편이 자연스러운 경우:
- **변경 이유가 다르다**는 감각이 크다
- **우연히 비슷할 뿐** 의미가 다르게 읽힌다
- **독립적으로 진화**할 것 같은 흐름이 있다

---

## 재사용이 발견되는 흐름

재사용 가능한 컴포넌트는  
처음부터 정해지기보다,  
단계적으로 드러나는 경우가 많다.

### 1단계: 중복이 없을 때는 재사용이 떠오르지 않는다

```tsx
// 홈 페이지에만 있는 레이아웃
export const HomePage = () => {
  return (
    <div className="container">
      <h1>홈</h1>
      <div>...</div>
    </div>
  );
};
```

아직 다른 곳에서 쓸지 모른다.  
재사용을 의식하기 어렵다.

### 2단계: 2-3번 반복되면 패턴을 관찰한다

```tsx
// 홈 페이지
<div className="container">
  <h1>홈</h1>
  <div>...</div>
</div>

// 프로필 페이지
<div className="container">
  <h1>프로필</h1>
  <div>...</div>
</div>

// 설정 페이지
<div className="container">
  <h1>설정</h1>
  <div>...</div>
</div>
```

비슷한 패턴이 보이기 시작한다.

하지만 아직 추상화하지 않는다.  
**차이가 무엇인지 관찰**하는 시간이 생긴다.

### 3단계: 차이의 본질을 이해한 후 추상화

```tsx
// 관찰 결과:
// - 모두 같은 구조
// - 제목과 내용만 다름
// - 로직은 각자 다름

// 이제 추상화
export const PageLayout = ({ 
  title,
  children 
}: PageLayoutProps) => {
  return (
    <div className="container">
      <h1>{title}</h1>
      <div>{children}</div>
    </div>
  );
};
```

차이의 본질을 이해했기 때문에  
최소한의 props로 정리되는 순간이 온다.

### 4단계: 새로운 요구사항이 생기면 재평가

```tsx
// 새로운 요구사항: 페이지마다 너비가 달라요
```

이때 선택지:
1. props 추가 → 유연성 ↑, 복잡도 ↑
2. 새 컴포넌트 → 중복 ↑, 복잡도 →

**판단의 실마리:**
- 모든 페이지가 너비 조절이 필요한가?
- 일부만 필요한가?

일부만 필요하다면  
조합으로 풀리기도 한다.

```tsx
export const WidePage = ({
  title,
  children,
}: WidePageProps) => {
  return (
    <div className="container container-wide">
      <PageLayout title={title}>
        {children}
      </PageLayout>
    </div>
  );
};
```

---

## 정리하며 — 재사용성은 공짜가 아니다

재사용 가능한 컴포넌트를 만드는 것은  
좋은 의도에서 시작한다.

중복을 줄이고,  
일관성을 유지하며,  
효율을 높이려는 시도다.

하지만 이 문서에서 살펴봤듯,  
재사용성은 공짜로 느껴지지 않는다.

> 재사용하려면 유연성이 필요하고,  
> 유연성을 높이면 복잡도가 증가한다.

이것은 쉽게 사라지지 않는 관계다.

그래서 중요한 질문은  
"재사용 가능하게 만들 수 있는가"보다,  
**"재사용할 만한 가치가 있는가"**에 가까워진다.

### 재사용하기 전에 던질 질문

- 이 컴포넌트들은 정말 **같은 것**인가?
- 변경이 **항상 함께** 일어나는가?
- 예외를 위한 props가 계속 늘어나고 있지 않은가?
- 재사용으로 얻는 이득이 복잡도 증가보다 큰가?

### 재사용을 둘러싼 렌즈

- **범위가 넓어질수록**  
  모든 곳에서 같은 방식으로 쓰기 어려워진다

- **유연성이 커질수록**  
  필요 이상의 조합과 비용이 함께 커진다

- **중복이 남아 있는 편이**  
  결합보다 의미를 보존하는 경우가 있다

- **패턴이 드러난 뒤에야**  
  재사용이 또렷해지는 흐름이 있다

재사용 가능한 컴포넌트는  
처음부터 완벽하게 만들어지지 않는다.

그것은 사용 패턴이 충분히 드러난 후,  
차이의 본질을 이해했을 때,  
비로소 만들어질 수 있다.

이 문서는  
재사용을 포기하라는 이야기가 아니라,  
**재사용의 비용을 이해하고  
의식적으로 선택하는 감각**에 관한 이야기다.

복잡도는  
재사용의 실패라기보다  
자주 동반되는 대가에 가깝다.

중요한 것은  
그 대가를 지불할 가치가 있는지  
판단하는 것이다.
