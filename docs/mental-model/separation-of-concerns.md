---
layout: default
title: 관심사 분리
parent: Mental Model
nav_order: 2
permalink: /docs/mental-model/separation-of-concerns
---

# 관심사 분리: 경계를 어떻게 나눌 것인가

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

## 관심사의 종류

프론트엔드에서 자주 마주치는 관심사들:

### 1. 데이터 관심사

```tsx
// 데이터 가져오기
export const useUser = () => {
  const [user, setUser] = useState(null);
  useEffect(() => {
    fetchUser().then(setUser);
  }, []);
  return user;
};
```

변경 이유:
- API 엔드포인트가 바뀐다
- 데이터 구조가 바뀐다
- 캐싱 전략이 바뀐다

### 2. 비즈니스 로직 관심사

```tsx
// 권한 확인
export const usePermission = (user) => {
  return user?.role === 'admin' || user?.isOwner;
};
```

변경 이유:
- 권한 규칙이 바뀐다
- 비즈니스 요구사항이 바뀐다

### 3. UI 관심사

```tsx
// 화면 표시
export const UserProfile = ({ user }) => {
  return (
    <div>
      <h1>{user.name}</h1>
    </div>
  );
};
```

변경 이유:
- 디자인이 바뀐다
- 레이아웃이 바뀐다
- 스타일이 바뀐다

### 4. 상태 관리 관심사

```tsx
// 편집 상태
export const useEditMode = () => {
  const [isEditing, setIsEditing] = useState(false);
  const [editedValue, setEditedValue] = useState('');
  
  return {
    isEditing,
    editedValue,
    startEdit: () => setIsEditing(true),
    cancelEdit: () => setIsEditing(false),
    updateValue: setEditedValue,
  };
};
```

변경 이유:
- 편집 흐름이 바뀐다
- 상태 관리 방식이 바뀐다

---

## 좋은 분리 vs 나쁜 분리

모든 분리가 좋은 것은 아니다.

### 나쁜 분리: 억지로 나눔

```tsx
// 나쁨: 의미 없는 분리
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

이것은 과도한 분리다.  
이름과 이메일은 **함께 변경**된다.  
나눌 이유가 없다.

### 좋은 분리: 책임에 따라 나눔

```tsx
// 좋음: 의미 있는 분리
export const UserData = ({ user }) => {
  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
};

export const UserActions = ({ onEdit, onDelete }) => {
  return (
    <div>
      <button onClick={onEdit}>편집</button>
      <button onClick={onDelete}>삭제</button>
    </div>
  );
};

export const UserProfile = ({ user, onEdit, onDelete }) => {
  return (
    <div>
      <UserData user={user} />
      <UserActions onEdit={onEdit} onDelete={onDelete} />
    </div>
  );
};
```

데이터 표시와 액션은  
서로 다른 이유로 변경된다.  
나눌 가치가 있다.

---

## 분리의 신호

언제 나눠야 하는지 알려주는 신호들:

### 1. 컴포넌트가 너무 길다

```tsx
// 200줄 이상의 컴포넌트
export const UserProfile = () => {
  // ... 100줄
  // ... 더 많은 코드
};
```

→ 여러 책임이 섞여 있을 가능성

### 2. useEffect가 3개 이상

```tsx
export const UserProfile = () => {
  useEffect(() => { /* 사용자 데이터 */ }, []);
  useEffect(() => { /* 권한 확인 */ }, []);
  useEffect(() => { /* 알림 구독 */ }, []);
  // ...
};
```

→ 여러 데이터 소스를 다루고 있음

### 3. useState가 5개 이상

```tsx
export const UserProfile = () => {
  const [user, setUser] = useState(null);
  const [isEditing, setIsEditing] = useState(false);
  const [editedName, setEditedName] = useState('');
  const [hasPermission, setHasPermission] = useState(false);
  const [isLoading, setIsLoading] = useState(false);
  // ...
};
```

→ 여러 상태를 관리하고 있음

### 4. "그리고"가 들어가는 설명

> "이 컴포넌트는 사용자 데이터를 가져오고, **그리고** 편집 기능을 제공하고, **그리고** 권한을 확인한다."

"그리고"가 들어가면  
여러 책임을 가지고 있다는 신호다.

---

## 분리의 판단 기준

관심사를 나눌지 말지 판단하는 기준:

| 질문 | Yes | No |
|-----|-----|-----|
| **독립적으로 변경**되는가? | 나눈다 | 함께 둔다 |
| **재사용**될 가능성이 있는가? | 나눈다 | 함께 둔다 |
| **테스트**를 독립적으로 해야 하는가? | 나눈다 | 함께 둔다 |
| 변경 **이유가 다른가**? | 나눈다 | 함께 둔다 |
| **이해하기** 쉬워지는가? | 나눈다 | 함께 둔다 |

"나눈다"가 3개 이상이면  
분리할 가치가 있다.

---

## 나누는 방법

관심사를 나누는 구체적인 방법들:

### 1. Custom Hook으로 분리

```tsx
// 데이터 로직 분리
export const useUser = (userId) => {
  const [user, setUser] = useState(null);
  const [isLoading, setIsLoading] = useState(false);
  
  useEffect(() => {
    setIsLoading(true);
    fetchUser(userId)
      .then(setUser)
      .finally(() => setIsLoading(false));
  }, [userId]);
  
  return { user, isLoading };
};

// 사용
export const UserProfile = ({ userId }) => {
  const { user, isLoading } = useUser(userId);
  
  if (isLoading) return <Loading />;
  return <div>{user.name}</div>;
};
```

### 2. 컴포넌트로 분리

```tsx
// UI 로직 분리
export const UserAvatar = ({ user }) => {
  return (
    <div className="avatar">
      <img src={user.avatar} alt={user.name} />
    </div>
  );
};

export const UserInfo = ({ user }) => {
  return (
    <div className="info">
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
};

// 조합
export const UserProfile = ({ user }) => {
  return (
    <div>
      <UserAvatar user={user} />
      <UserInfo user={user} />
    </div>
  );
};
```

### 3. 유틸 함수로 분리

```tsx
// 비즈니스 로직 분리
export const canEditUser = (currentUser, targetUser) => {
  return currentUser.role === 'admin' || currentUser.id === targetUser.id;
};

export const formatUserName = (user) => {
  return `${user.firstName} ${user.lastName}`;
};

// 사용
export const UserProfile = ({ user, currentUser }) => {
  const canEdit = canEditUser(currentUser, user);
  const displayName = formatUserName(user);
  
  return (
    <div>
      <h1>{displayName}</h1>
      {canEdit && <button>편집</button>}
    </div>
  );
};
```

---

## 나누는 것의 비용

분리에도 비용이 있다.

### 비용 1: 파일이 늘어난다

```
// 분리 전: 1개 파일
UserProfile.tsx

// 분리 후: 4개 파일
UserProfile.tsx
useUser.ts
UserAvatar.tsx
UserInfo.tsx
```

### 비용 2: 추적이 어려워진다

```tsx
// 분리 전: 한 곳에서 모든 것
export const UserProfile = () => {
  // 여기서 모든 것 확인 가능
};

// 분리 후: 여러 곳 확인 필요
export const UserProfile = () => {
  const { user } = useUser();  // useUser.ts 확인 필요
  return <UserInfo user={user} />;  // UserInfo.tsx 확인 필요
};
```

### 비용 3: 과도한 추상화

```tsx
// 나쁨: 과도한 분리
export const useFetchData = () => { /* ... */ };
export const useLoadingState = () => { /* ... */ };
export const useErrorState = () => { /* ... */ };
export const useDataProcessing = () => { /* ... */ };
export const useDataValidation = () => { /* ... */ };
// ...
```

모든 것을 나누면  
오히려 복잡해진다.

---

## 적절한 균형

관심사 분리의 목표는  
**모든 것을 나누는 것이 아니라**,  
**적절한 균형을 찾는 것**이다.

### 나누지 않아도 되는 경우

- 항상 함께 변경되는 것
- 서로 강하게 의존하는 것
- 재사용 가능성이 없는 것
- 분리해도 이해하기 쉬워지지 않는 것

### 반드시 나눠야 하는 경우

- 독립적으로 변경되는 것
- 재사용될 가능성이 있는 것
- 독립적으로 테스트해야 하는 것
- 변경 이유가 명확히 다른 것

---

## 예시: 실전 적용

### Before: 섞여 있음

```tsx
export const UserProfile = ({ userId }) => {
  const [user, setUser] = useState(null);
  const [isLoading, setIsLoading] = useState(false);
  const [isEditing, setIsEditing] = useState(false);
  const [editedName, setEditedName] = useState('');
  
  useEffect(() => {
    setIsLoading(true);
    fetch(`/api/users/${userId}`)
      .then(res => res.json())
      .then(setUser)
      .finally(() => setIsLoading(false));
  }, [userId]);
  
  const handleEdit = () => {
    setIsEditing(true);
    setEditedName(user.name);
  };
  
  const handleSave = () => {
    fetch(`/api/users/${userId}`, {
      method: 'PUT',
      body: JSON.stringify({ name: editedName })
    }).then(() => {
      setUser({ ...user, name: editedName });
      setIsEditing(false);
    });
  };
  
  if (isLoading) return <div>로딩중...</div>;
  
  return (
    <div>
      {isEditing ? (
        <div>
          <input 
            value={editedName} 
            onChange={(e) => setEditedName(e.target.value)} 
          />
          <button onClick={handleSave}>저장</button>
          <button onClick={() => setIsEditing(false)}>취소</button>
        </div>
      ) : (
        <div>
          <h1>{user.name}</h1>
          <button onClick={handleEdit}>편집</button>
        </div>
      )}
    </div>
  );
};
```

### After: 관심사 분리

```tsx
// 1. 데이터 관심사
export const useUser = (userId) => {
  const [user, setUser] = useState(null);
  const [isLoading, setIsLoading] = useState(false);
  
  useEffect(() => {
    setIsLoading(true);
    fetch(`/api/users/${userId}`)
      .then(res => res.json())
      .then(setUser)
      .finally(() => setIsLoading(false));
  }, [userId]);
  
  const updateUser = (updates) => {
    return fetch(`/api/users/${userId}`, {
      method: 'PUT',
      body: JSON.stringify(updates)
    }).then(() => {
      setUser({ ...user, ...updates });
    });
  };
  
  return { user, isLoading, updateUser };
};

// 2. 편집 상태 관심사
export const useEditMode = (initialValue) => {
  const [isEditing, setIsEditing] = useState(false);
  const [editedValue, setEditedValue] = useState(initialValue);
  
  return {
    isEditing,
    editedValue,
    startEdit: () => {
      setIsEditing(true);
      setEditedValue(initialValue);
    },
    updateValue: setEditedValue,
    cancelEdit: () => setIsEditing(false),
    save: (onSave) => {
      onSave(editedValue);
      setIsEditing(false);
    },
  };
};

// 3. UI 관심사
export const UserDisplay = ({ user, onEdit }) => {
  return (
    <div>
      <h1>{user.name}</h1>
      <button onClick={onEdit}>편집</button>
    </div>
  );
};

export const UserEdit = ({ value, onChange, onSave, onCancel }) => {
  return (
    <div>
      <input value={value} onChange={(e) => onChange(e.target.value)} />
      <button onClick={onSave}>저장</button>
      <button onClick={onCancel}>취소</button>
    </div>
  );
};

// 4. 조합
export const UserProfile = ({ userId }) => {
  const { user, isLoading, updateUser } = useUser(userId);
  const editMode = useEditMode(user?.name);
  
  if (isLoading) return <div>로딩중...</div>;
  
  return (
    <div>
      {editMode.isEditing ? (
        <UserEdit
          value={editMode.editedValue}
          onChange={editMode.updateValue}
          onSave={() => editMode.save((name) => updateUser({ name }))}
          onCancel={editMode.cancelEdit}
        />
      ) : (
        <UserDisplay user={user} onEdit={editMode.startEdit} />
      )}
    </div>
  );
};
```

---

## 정리하며 — 관심사 분리를 다시 정의하면

관심사 분리는  
파일을 나누는 기술이 아니다.

관심사 분리는  
**변경의 이유를 명확히 하는 것**이다.

> 각 모듈/컴포넌트가  
> 하나의 이유로만 변경되도록 만드는 것.

### 관심사 분리를 하는 이유

- 변경이 한 곳에서만 일어나게 (변경의 국소성)
- 코드를 이해하기 쉽게
- 테스트를 독립적으로
- 재사용 가능하게

### 관심사를 나누는 기준

- 변경의 이유가 다른가?
- 독립적으로 변경되는가?
- 재사용될 가능성이 있는가?

### 나누는 것의 균형

모든 것을 나누는 것이 답이 아니다.  
적절한 균형이 중요하다.

- 너무 적게 나누면 → 복잡해진다
- 너무 많이 나누면 → 추적하기 어렵다

중요한 것은  
**변경의 이유를 명확히 하는 것**이다.

관심사 분리는  
복잡도를 다루기 위한  
가장 기본적이고 강력한 원칙이다.
