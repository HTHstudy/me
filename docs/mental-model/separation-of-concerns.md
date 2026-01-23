---
layout: default
title: 관심사 분리
parent: Mental Model
nav_order: 2
permalink: /docs/mental-model/separation-of-concerns
---

# 관심사 분리
> “우리는 언제 ‘나눠야겠다’고 판단하는가  
> 나눴는데도 복잡해지는 이유는 무엇인가  
> 잘못 잡힌 경계는 무엇을 망가뜨리는가”




## 들어가며 — 복잡하니까 나눠야지

컴포넌트가 복잡해지면  
자연스럽게 이런 생각이 든다.

> "이거 너무 복잡한데, 나눠야 하지 않을까?"

```tsx
// 복잡한 UserProfile 컴포넌트
export const UserProfile = () => {
  // 사용자 데이터
  const [user, setUser] = useState(null);
  const [isLoading, setIsLoading] = useState(false);
  
  // 편집 상태
  const [isEditing, setIsEditing] = useState(false);
  const [editedName, setEditedName] = useState('');
  
  // 권한 확인
  const [hasPermission, setHasPermission] = useState(false);
  
  useEffect(() => {
    fetchUser().then(setUser);
    checkPermission().then(setHasPermission);
  }, []);
  
  const handleEdit = () => { /* ... */ };
  const handleSave = () => { /* ... */ };
  const handleCancel = () => { /* ... */ };
  
  // 100줄 이상의 렌더링 로직
  return (
    <div>
      {/* 복잡한 UI */}
    </div>
  );
};
```

이 컴포넌트는:
- 데이터를 가져오고
- 편집 상태를 관리하고
- 권한을 확인하고
- UI를 렌더링한다

**너무 많은 것을 하고 있다.**

이것이 컴포넌트를 나누게 되는 자연스러운 이유다.

하지만 여기서 질문이 생긴다.

> "어떻게 나눠야 할까?"

이 문서는  
컴포넌트를 나누는 기술이 아니라,  
**무엇을 기준으로 나눌 것인지**를  
다시 생각해보기 위해 쓰였다.

---

## 관심사 분리란 무엇인가

관심사 분리(Separation of Concerns)는  
**서로 다른 책임을 서로 다른 곳에 두는 것**이다.

### 간단한 예시

```tsx
// 나쁨: 모든 것이 한 곳에
export const UserProfile = () => {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    fetch('/api/user').then(res => res.json()).then(setUser);
  }, []);
  
  if (!user) return <div>로딩중...</div>;
  
  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
};
```

이 컴포넌트는:
- 데이터를 **가져오는** 책임
- 데이터를 **표시하는** 책임

두 가지를 모두 가진다.

```tsx
// 좋음: 책임을 나눔
// 데이터 가져오기
export const useUser = () => {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    fetch('/api/user').then(res => res.json()).then(setUser);
  }, []);
  
  return user;
};

// 데이터 표시하기
export const UserProfile = () => {
  const user = useUser();
  
  if (!user) return <div>로딩중...</div>;
  
  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
};
```

이제 각각은 하나의 책임만 가진다.

이것이 관심사 분리다.

> 관심사 분리는  
> 각각의 책임을 명확히 하고,  
> 서로 다른 책임을 섞지 않는 것이다.

---

## 왜 나눠야 하는가

관심사 분리를 하는 이유는  
**변경을 쉽게 만들기 위해서**다.

### 변경의 국소성

코드를 수정할 때  
이상적인 상황은 이렇다.

> "A를 바꾸려면 A만 수정하면 된다."

하지만 관심사가 섞여 있으면:

```tsx
// 나쁨: 섞여 있음
export const UserProfile = () => {
  const [user, setUser] = useState(null);
  
  // API 엔드포인트 변경이 필요하면?
  useEffect(() => {
    fetch('/api/user').then(res => res.json()).then(setUser);
  }, []);
  
  // UI 레이아웃 변경이 필요하면?
  return (
    <div className="profile">
      <h1>{user?.name}</h1>
    </div>
  );
};
```

- API 엔드포인트를 바꾸려면 → UserProfile 수정
- UI 레이아웃을 바꾸려면 → UserProfile 수정
- 모든 변경이 같은 파일로 모인다

```tsx
// 좋음: 나뉘어 있음
export const useUser = () => {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    fetch('/api/user').then(res => res.json()).then(setUser);
  }, []);
  
  return user;
};

export const UserProfile = () => {
  const user = useUser();
  
  return (
    <div className="profile">
      <h1>{user?.name}</h1>
    </div>
  );
};
```

- API 엔드포인트를 바꾸려면 → `useUser` 수정
- UI 레이아웃을 바꾸려면 → `UserProfile` 수정
- 각 변경이 자기 자리에서 일어난다

이것이 **변경의 국소성**이다.

> 관심사를 나누면,  
> 변경의 이유가 명확해지고,  
> 변경이 한 곳에서만 일어난다.

---

## 무엇을 기준으로 나누는가

그렇다면 **무엇을** 기준으로 나눠야 할까?

### 변경의 이유

가장 중요한 기준은  
**"왜 이 코드가 바뀌는가"**다.

```tsx
// 이 컴포넌트가 바뀌는 이유는?
export const UserProfile = () => {
  // 1. API 엔드포인트가 바뀔 때
  const [user, setUser] = useState(null);
  useEffect(() => {
    fetch('/api/user').then(res => res.json()).then(setUser);
  }, []);
  
  // 2. 권한 규칙이 바뀔 때
  const canEdit = user?.role === 'admin';
  
  // 3. UI 디자인이 바뀔 때
  return (
    <div>
      <h1>{user?.name}</h1>
      {canEdit && <button>편집</button>}
    </div>
  );
};
```

이 컴포넌트는 **세 가지 이유**로 바뀐다.

각 이유는 서로 다른 관심사다.

```tsx
// 1. 데이터 가져오기 (API)
export const useUser = () => {
  const [user, setUser] = useState(null);
  useEffect(() => {
    fetch('/api/user').then(res => res.json()).then(setUser);
  }, []);
  return user;
};

// 2. 권한 확인 (비즈니스 규칙)
export const useCanEditUser = (user) => {
  return user?.role === 'admin';
};

// 3. UI 표시
export const UserProfile = () => {
  const user = useUser();
  const canEdit = useCanEditUser(user);
  
  return (
    <div>
      <h1>{user?.name}</h1>
      {canEdit && <button>편집</button>}
    </div>
  );
};
```

이제 각각은 하나의 이유로만 바뀐다.

### 단일 책임 원칙

이것을 **단일 책임 원칙**(Single Responsibility Principle)이라고 한다.

> 각 모듈/컴포넌트는  
> 변경의 이유가 하나여야 한다.

---

## 관심사가 어떻게 구분되는가

프론트엔드 코드를 보면  
여러 종류의 관심사가 섞여 있다.

하지만 중요한 것은  
"이것은 데이터 관심사다"라고 분류하는 것이 아니라,  
**왜 이런 관심사들이 생기는가**다.

### 관심사는 변경의 이유로 구분된다

관심사는 내용이 아니라,  
**변경의 이유**로 구분된다.

같은 코드라도  
변경의 이유가 다르면 다른 관심사다.

```tsx
// 이 코드는 어떤 관심사인가?
export const UserProfile = () => {
  const [user, setUser] = useState(null);
  useEffect(() => {
    fetch('/api/user').then(res => res.json()).then(setUser);
  }, []);
  
  return <div>{user?.name}</div>;
};
```

이 코드는:
- API 엔드포인트가 바뀔 때 변경 → **데이터 관심사**
- 디자인이 바뀔 때 변경 → **UI 관심사**

두 가지 관심사가 섞여 있다.

### 변경의 이유가 다르면 다른 관심사다

```tsx
// 데이터 관심사
export const useUser = () => {
  const [user, setUser] = useState(null);
  useEffect(() => {
    fetchUser().then(setUser);  // API가 바뀌면 여기만 수정
  }, []);
  return user;
};

// UI 관심사
export const UserProfile = ({ user }) => {
  return (
    <div>
      <h1>{user.name}</h1>  // 디자인이 바뀌면 여기만 수정
    </div>
  );
};
```

이 둘은 변경의 이유가 다르다.

- API가 바뀌면 → `useUser`만 수정
- 디자인이 바뀌면 → `UserProfile`만 수정

**변경의 이유가 다르기 때문에**  
다른 관심사로 구분된다.

### 관심사 분류는 고정된 것이 아니다

"데이터 관심사", "UI 관심사" 같은 분류는  
편의를 위한 이름일 뿐이다.

중요한 것은 분류가 아니라,  
**변경의 이유가 다른가**다.

같은 코드라도:
- API가 바뀌는 이유로 변경되면 → 데이터 관심사
- 디자인이 바뀌는 이유로 변경되면 → UI 관심사
- 비즈니스 규칙이 바뀌는 이유로 변경되면 → 비즈니스 로직 관심사

**변경의 이유가 관심사를 구분한다.**

---

## 잘못된 분리가 무엇을 망가뜨리는가

분리를 하면  
항상 좋아지는 것은 아니다.

잘못된 분리는  
오히려 복잡도를 높인다.

### 함께 변경되어야 하는 것들을 나누면

```tsx
// 이름과 이메일을 나눔
export const UserNamePart = ({ name }) => {
  return <h1>{name}</h1>;
};

export const UserEmailPart = ({ email }) => {
  return <p>{email}</p>;
};

export const UserProfile = ({ user }) => {
  return (
    <div>
      <UserNamePart name={user.name} />
      <UserEmailPart email={user.email} />
    </div>
  );
};
```

이름과 이메일은  
**함께 변경**되어야 한다.

- 사용자 정보 레이아웃이 바뀌면 → 둘 다 수정해야 함
- 스타일이 바뀌면 → 둘 다 수정해야 함

나눴지만 여전히 함께 변경해야 한다면,  
분리의 의미가 없다.

오히려:
- 파일이 늘어난다
- 추적이 어려워진다
- 변경이 여러 곳에 흩어진다

**함께 변경되어야 하는 것들을 나누면**  
변경의 국소성이 사라진다.

### 다른 이유로 변경되는 것들을 함께 두면

```tsx
// 데이터와 UI를 함께 둠
export const UserProfile = () => {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    fetch('/api/user').then(res => res.json()).then(setUser);
  }, []);
  
  return (
    <div>
      <h1>{user?.name}</h1>
    </div>
  );
};
```

이 코드는:
- API가 바뀌면 → 전체 컴포넌트 수정
- 디자인이 바뀌면 → 전체 컴포넌트 수정

**다른 이유로 변경되는 것들을 함께 두면**  
변경이 한 곳에 모이지 않는다.

### 분리가 잘못되면

잘못된 분리는:
- 함께 변경되어야 하는 것들을 나눈다
- 또는 다른 이유로 변경되는 것들을 함께 둔다

둘 다 문제다.

- 함께 변경되어야 하는데 나눴다 → 변경이 여러 곳에 흩어짐
- 다른 이유로 변경되는데 함께 뒀다 → 변경이 한 곳에 모이지 않음

**분리의 목적은 변경의 국소성**이다.

잘못된 분리는 이 목적을 망가뜨린다.

---

## 분리의 신호가 나타나는 이유

분리의 욕구는 보통 특정 신호로 먼저 나타난다.

- 코드가 길어짐
- 상태·Effect가 늘어남
- 설명에 "그리고"가 반복됨

하지만 중요한 것은 신호 자체가 아니다.  
**신호는 기준이 아니다. 신호는 ‘변경 이유가 섞였다’는 징후일 뿐이다.**

---

## 분리의 판단 기준

관심사를 나눌지 말지 판단할 때  
던져야 할 질문들이 있다.

- 변경 **이유가 다른가?**
- **독립적으로 변경**되는가?
- **함께 변경**되어야 하는가?

이 질문들에 대한 답이  
나눌지 말지를 결정한다.

중요한 것은  
체크리스트를 채우는 것이 아니라,  
**변경의 이유를 명확히 보는 것**이다.

---


## 나누는 것의 비용

분리에도 비용이 있다.

하지만 중요한 것은  
비용을 열거하는 것이 아니라,  
**비용의 성격을 이해하는 것**이다.

분리의 비용은 보통 다음 질문으로 드러난다.

- 어디를 찾아가야 하는가?  
- 어떤 흐름을 따라가야 하는가?  
- 어떤 규칙을 지켜야 하는가?

이 질문들은 예시에 가깝다.  
맥락에 따라 다른 질문이 더 중요할 수도 있다.

예를 들어:

```tsx
// UserProfile.tsx
const user = useUser();           // useUser.ts
const canEdit = useCanEdit(user); // useCanEdit.ts
return <UserView user={user} />;  // UserView.tsx
```

이 작은 흐름 하나를 이해하려고도  
파일을 넘나들어야 한다.  
이 순간 비용이 생긴다.

어떤 비용이 더 크게 느껴지는지는  
팀, 코드베이스 크기, 도구, 테스트 전략에 따라 달라진다.

비용은 분리의 결과다.  
그래서 분리는 항상 트레이드오프다.

---

## 정리하며 — 관심사 분리를 다시 정의하면

관심사 분리는  
파일을 나누는 기술이 아니다.

관심사 분리는  
**변경의 이유를 명확히 하는 것**이다.

> 각 모듈/컴포넌트가  
> 하나의 이유로만 변경되도록 만드는 것.

이것이 관심사 분리의 본질이다.

관심사 분리는  
구조를 예쁘게 만들기 위한 선택이 아니라,  
미래의 변경을 예측하고 감당하기 위한 결정이다.

변경의 이유가 다르면 나누고,  
함께 변경되어야 하면 함께 둔다.

이것이 판단의 출발점이다.
