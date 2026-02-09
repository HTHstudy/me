---
layout: default
title: 이 컴포넌트는 나눠야 하는가
parent: Decisions over Patterns
nav_order: 3
permalink: /docs/decisions-over-patterns/should-component-be-split
---

# 이 컴포넌트는 나눠야 하는가

> "이 컴포넌트는 하나여야 하는가, 여럿이어야 하는가  
> 그 경계는 무엇을 기준으로 그어지는가"

## 들어가며 — 상태를 정리해도 복잡한 컴포넌트

불필요한 상태를 제거했다.  
상태의 위치도 정리했다.

그런데도 컴포넌트가 여전히 복잡하다.

```tsx
const ProductPage = () => {
  const [product, setProduct] = useState(null);
  const [isLoading, setIsLoading] = useState(true);

  const [selectedVariant, setSelectedVariant] = useState(null);
  const [quantity, setQuantity] = useState(1);

  const [reviews, setReviews] = useState([]);
  const [reviewSort, setReviewSort] = useState('recent');

  useEffect(() => {
    fetchProduct(id).then(setProduct).finally(() => setIsLoading(false));
    fetchReviews(id).then(setReviews);
  }, [id]);

  const sortedReviews = [...reviews].sort(/* reviewSort에 따라 */);
  const totalPrice = selectedVariant
    ? selectedVariant.price * quantity
    : 0;

  const handleAddToCart = () => { /* ... */ };
  const handleReviewSort = (sort) => { /* ... */ };

  if (isLoading) return <Loading />;

  return (
    <div>
      {/* 상품 정보 */}
      {/* 옵션 선택 */}
      {/* 가격 표시 */}
      {/* 장바구니 버튼 */}
      {/* 리뷰 목록 */}
      {/* 리뷰 정렬 */}
    </div>
  );
};
```

상태는 모두 필요하다.  
위치도 적절하다.  
파생 값은 계산으로 처리했다.

그런데 이 컴포넌트를 읽으려면  
여러 관심사를 한꺼번에 따라가야 한다.

- 상품 데이터를 가져오는 로직
- 옵션 선택과 수량 관리
- 리뷰 목록과 정렬
- 장바구니 추가

이 상황에서 자연스럽게  
이런 질문이 떠오른다.

**"이 컴포넌트를 나눠야 하는가?"**

이 문서는  
컴포넌트를 나눌지 말지 결정할 때  
어떤 기준으로 판단하는지를 다룬다.

---

## 선택지들

컴포넌트가 복잡해졌을 때  
선택할 수 있는 방향은 여러 가지다.

### 나누지 않는다

```tsx
const ShippingForm = () => {
  const [name, setName] = useState('');
  const [address, setAddress] = useState('');
  const [city, setCity] = useState('');
  const [zipCode, setZipCode] = useState('');
  const [phone, setPhone] = useState('');

  const isValid = name && address && city && zipCode && phone;

  const handleSubmit = () => {
    submitShipping({ name, address, city, zipCode, phone });
  };

  return (
    <form onSubmit={handleSubmit}>
      <input value={name} onChange={e => setName(e.target.value)} placeholder="이름" />
      <input value={address} onChange={e => setAddress(e.target.value)} placeholder="주소" />
      <input value={city} onChange={e => setCity(e.target.value)} placeholder="도시" />
      <input value={zipCode} onChange={e => setZipCode(e.target.value)} placeholder="우편번호" />
      <input value={phone} onChange={e => setPhone(e.target.value)} placeholder="전화번호" />
      <button disabled={!isValid}>제출</button>
    </form>
  );
};
```

상태가 다섯 개다.  
코드도 꽤 길다.

하지만 이 컴포넌트가 바뀌는 이유는 하나다.  
**배송 정보 입력 방식이 바뀔 때.**

필드를 추가하거나, 유효성 검사를 바꾸거나, 제출 로직을 수정할 때  
**모두 이 컴포넌트 안에서 일어난다.**

이런 경우 나누면 오히려 문제가 생긴다.

```tsx
// 나눴지만 의미 없는 분리
const NameField = ({ value, onChange }) => (
  <input value={value} onChange={onChange} placeholder="이름" />
);

const AddressField = ({ value, onChange }) => (
  <input value={value} onChange={onChange} placeholder="주소" />
);

// ... 나머지 필드들도 각각 컴포넌트로

const ShippingForm = () => {
  const [name, setName] = useState('');
  const [address, setAddress] = useState('');
  // ...

  return (
    <form>
      <NameField value={name} onChange={e => setName(e.target.value)} />
      <AddressField value={address} onChange={e => setAddress(e.target.value)} />
      {/* ... */}
    </form>
  );
};
```

파일은 늘었지만  
변경은 여전히 여러 곳에서 일어난다.  
나누기 전보다 오히려 추적이 어려워졌다.

**언제?**

- 변경의 이유가 하나일 때
- 상태들이 긴밀하게 연관되어 있을 때
- 나눈 조각들이 독립적으로 의미를 갖지 못할 때

**왜?**

- 하나의 흐름으로 읽을 수 있다
- 관련된 코드가 한곳에 있어서 변경이 쉽다
- 응집도가 높다

**비용**

- 코드가 길어질 수 있다
- 여러 관심사가 섞여 보일 수 있다

### 렌더링 영역을 분리한다

```tsx
const ProductInfo = ({ product, selectedVariant, onVariantSelect }) => (
  <div>
    <h1>{product.name}</h1>
    <p>{product.description}</p>
    <VariantSelector
      variants={product.variants}
      selected={selectedVariant}
      onSelect={onVariantSelect}
    />
  </div>
);

const PurchaseSection = ({ totalPrice, quantity, onQuantityChange, onAddToCart }) => (
  <div>
    <p>총액: {totalPrice}원</p>
    <input
      type="number"
      value={quantity}
      onChange={e => onQuantityChange(Number(e.target.value))}
    />
    <button onClick={onAddToCart}>장바구니 담기</button>
  </div>
);

const ReviewList = ({ reviews, sort, onSortChange }) => {
  const sortedReviews = [...reviews].sort(/* sort에 따라 */);

  return (
    <div>
      <select value={sort} onChange={e => onSortChange(e.target.value)}>
        <option value="recent">최신순</option>
        <option value="rating">평점순</option>
      </select>
      <ul>
        {sortedReviews.map(review => (
          <li key={review.id}>{review.text}</li>
        ))}
      </ul>
    </div>
  );
};

const ProductPage = () => {
  const [product, setProduct] = useState(null);
  const [selectedVariant, setSelectedVariant] = useState(null);
  const [quantity, setQuantity] = useState(1);
  const [reviews, setReviews] = useState([]);
  const [reviewSort, setReviewSort] = useState('recent');

  // 데이터 페칭 로직...

  const totalPrice = selectedVariant ? selectedVariant.price * quantity : 0;

  return (
    <div>
      <ProductInfo
        product={product}
        selectedVariant={selectedVariant}
        onVariantSelect={setSelectedVariant}
      />
      <PurchaseSection
        totalPrice={totalPrice}
        quantity={quantity}
        onQuantityChange={setQuantity}
        onAddToCart={handleAddToCart}
      />
      <ReviewList
        reviews={reviews}
        sort={reviewSort}
        onSortChange={setReviewSort}
      />
    </div>
  );
};
```

렌더링 코드를 영역별로 분리했다.  
ProductPage는 상태와 흐름을 관리하고,  
각 영역은 전달받은 데이터를 표시한다.

**언제?**

- 렌더링 코드가 길어서 전체 구조가 안 보일 때
- UI 영역이 시각적으로 독립적일 때
- 각 영역의 디자인이 독립적으로 바뀔 수 있을 때

**왜?**

- ProductPage를 읽으면 전체 구조가 한눈에 보인다
- 각 영역의 UI를 수정할 때 해당 컴포넌트만 보면 된다
- 상태 관리는 여전히 한곳에 있어서 흐름을 추적하기 쉽다

**비용**

- props 전달이 늘어난다
- 상태는 부모에 남아 있어서, 부모의 복잡도는 크게 줄지 않는다
- 렌더링 영역만 분리했을 뿐, 관심사 자체는 분리되지 않을 수 있다

### 책임 단위로 분리한다

```tsx
// 상품 정보를 가져오고 표시하는 책임
const ProductDetail = ({ productId }) => {
  const [product, setProduct] = useState(null);
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    fetchProduct(productId).then(setProduct).finally(() => setIsLoading(false));
  }, [productId]);

  if (isLoading) return <Loading />;

  return (
    <div>
      <h1>{product.name}</h1>
      <p>{product.description}</p>
    </div>
  );
};

// 구매 옵션을 관리하는 책임
const PurchaseOptions = ({ variants, onAddToCart }) => {
  const [selectedVariant, setSelectedVariant] = useState(null);
  const [quantity, setQuantity] = useState(1);

  const totalPrice = selectedVariant ? selectedVariant.price * quantity : 0;

  return (
    <div>
      <VariantSelector
        variants={variants}
        selected={selectedVariant}
        onSelect={setSelectedVariant}
      />
      <input
        type="number"
        value={quantity}
        onChange={e => setQuantity(Number(e.target.value))}
      />
      <p>총액: {totalPrice}원</p>
      <button onClick={() => onAddToCart(selectedVariant, quantity)}>
        장바구니 담기
      </button>
    </div>
  );
};

// 리뷰를 가져오고 표시하는 책임
const ProductReviews = ({ productId }) => {
  const [reviews, setReviews] = useState([]);
  const [sort, setSort] = useState('recent');

  useEffect(() => {
    fetchReviews(productId).then(setReviews);
  }, [productId]);

  const sortedReviews = [...reviews].sort(/* sort에 따라 */);

  return (
    <div>
      <select value={sort} onChange={e => setSort(e.target.value)}>
        <option value="recent">최신순</option>
        <option value="rating">평점순</option>
      </select>
      <ul>
        {sortedReviews.map(review => (
          <li key={review.id}>{review.text}</li>
        ))}
      </ul>
    </div>
  );
};

// 페이지는 조합만 한다
const ProductPage = ({ productId }) => {
  return (
    <div>
      <ProductDetail productId={productId} />
      <PurchaseOptions
        variants={/* ... */}
        onAddToCart={handleAddToCart}
      />
      <ProductReviews productId={productId} />
    </div>
  );
};
```

각 컴포넌트가 자신의 상태를 소유한다.  
ProductPage는 조합만 하고,  
각 컴포넌트는 자신의 책임 안에서 완결된다.

**언제?**

- 변경의 이유가 여러 개이고 독립적일 때
- 각 부분이 자체적으로 데이터를 관리할 수 있을 때
- 상품 정보 변경, 구매 로직 변경, 리뷰 기능 변경이 서로 영향을 주지 않을 때

**왜?**

- 각 컴포넌트가 하나의 이유로만 변경된다
- 리뷰 기능을 수정해도 상품 정보 컴포넌트는 건드리지 않는다
- 각 컴포넌트가 독립적으로 이해 가능하다

**비용**

- 컴포넌트 간에 데이터를 공유해야 할 때 설계가 필요하다
- 분리 경계를 잘못 잡으면 오히려 복잡해진다
- 전체 흐름을 보려면 여러 컴포넌트를 넘나들어야 한다

### 합성 패턴으로 재구성한다

```tsx
// 레이아웃만 제공하는 컴포넌트
const PageLayout = ({ header, main, sidebar }) => (
  <div className="page-layout">
    <header>{header}</header>
    <main>{main}</main>
    <aside>{sidebar}</aside>
  </div>
);

// 사용하는 쪽이 내용을 결정한다
const ProductPage = ({ productId }) => {
  return (
    <PageLayout
      header={<ProductDetail productId={productId} />}
      main={<PurchaseOptions productId={productId} />}
      sidebar={<ProductReviews productId={productId} />}
    />
  );
};

const ArticlePage = ({ articleId }) => {
  return (
    <PageLayout
      header={<ArticleHeader articleId={articleId} />}
      main={<ArticleContent articleId={articleId} />}
      sidebar={<RelatedArticles articleId={articleId} />}
    />
  );
};
```

PageLayout은 **구조만** 정의한다.  
무엇을 넣을지는 사용하는 쪽이 결정한다.

```tsx
// children을 사용하는 합성
const Card = ({ children }) => (
  <div className="card">{children}</div>
);

const ProductCard = ({ product }) => (
  <Card>
    <h3>{product.name}</h3>
    <p>{product.price}원</p>
  </Card>
);

const ReviewCard = ({ review }) => (
  <Card>
    <p>{review.text}</p>
    <span>{review.rating}점</span>
  </Card>
);
```

Card는 스타일을 제공하고,  
내용물은 사용하는 쪽에서 채운다.

**언제?**

- 같은 구조 안에서 내용물이 달라지는 경우
- 레이아웃이나 스타일은 공유하되 안의 내용은 다를 때
- 여러 곳에서 같은 틀을 재사용할 때

**왜?**

- 구조와 내용이 분리된다
- 사용하는 쪽이 유연하게 조합할 수 있다
- 레이아웃 변경은 틀 컴포넌트만, 내용 변경은 사용하는 쪽만 수정하면 된다

**비용**

- 컴포넌트의 역할이 "틀"로 바뀐다
- 사용하는 쪽의 책임이 커진다
- 틀이 너무 유연하면 일관성이 깨질 수 있다

---

## 판단의 질문들

컴포넌트를 나눌지 결정할 때  
던져볼 수 있는 질문들이 있다.

### 이 컴포넌트가 바뀌는 이유는 몇 가지인가?

```tsx
// 이 컴포넌트가 바뀌는 이유는?
const ProductPage = () => {
  // 1. 상품 데이터 가져오는 방식이 바뀔 때
  const [product, setProduct] = useState(null);
  useEffect(() => { fetchProduct(id).then(setProduct); }, [id]);

  // 2. 구매 로직이 바뀔 때
  const [quantity, setQuantity] = useState(1);
  const handleAddToCart = () => { /* ... */ };

  // 3. 리뷰 기능이 바뀔 때
  const [reviews, setReviews] = useState([]);
  const [reviewSort, setReviewSort] = useState('recent');

  // ...
};
```

변경의 이유가 하나라면 나누지 않는 것이 자연스럽다.  
변경의 이유가 여럿이고 독립적이라면  
분리를 고려할 수 있다.

### 나눈 조각들은 독립적으로 이해할 수 있는가?

```tsx
// 독립적으로 이해 가능
const ProductReviews = ({ productId }) => {
  // 리뷰를 가져오고, 정렬하고, 보여준다
  // 이 컴포넌트만 봐도 무엇을 하는지 알 수 있다
};

// 독립적으로 이해 불가
const ProductNameSection = ({ name }) => {
  return <h1>{name}</h1>;
};
// 이것만 봐서는 왜 존재하는지 알 수 없다
```

나눈 조각이 혼자서 의미를 가지지 못한다면  
분리가 코드를 더 읽기 어렵게 만들 수 있다.

### 나눈 후 데이터 흐름은 더 단순해지는가?

```tsx
// 나누기 전: 상태가 한곳에 있고 직접 사용
const ProductPage = () => {
  const [product, setProduct] = useState(null);
  const [selectedVariant, setSelectedVariant] = useState(null);

  // selectedVariant를 product.variants에서 선택
  // → 같은 스코프에 있어서 자연스럽다
};

// 나눈 후: 상태를 공유해야 한다
const ProductInfo = ({ product, onVariantSelect }) => { /* ... */ };
const PurchaseSection = ({ selectedVariant, price }) => { /* ... */ };

const ProductPage = () => {
  const [product, setProduct] = useState(null);
  const [selectedVariant, setSelectedVariant] = useState(null);

  // 상태는 여전히 여기에 있고
  // props로 내려보내야 한다
  return (
    <>
      <ProductInfo product={product} onVariantSelect={setSelectedVariant} />
      <PurchaseSection selectedVariant={selectedVariant} price={/* ... */} />
    </>
  );
};
```

나눈 후에 상태를 공유하기 위해  
props 전달이 복잡해진다면  
분리가 정말 도움이 되는지 다시 생각해볼 수 있다.

### 나눈 조각이 다른 곳에서도 의미가 있는가?

```tsx
// 다른 곳에서도 의미 있음
const ReviewList = ({ reviews, sort, onSortChange }) => {
  // 상품 페이지에서도, 마이 페이지에서도 쓸 수 있다
};

// 이 페이지에서만 의미 있음
const ProductPageHeader = ({ product }) => {
  // ProductPage 전용 헤더
  // 다른 곳에서 쓸 일이 없다
};
```

재사용 가능성은  
분리의 **목적이 아니라 보너스**다.

다른 곳에서 쓰이지 않더라도  
변경의 이유가 독립적이라면 분리할 수 있고,  
여러 곳에서 쓰이더라도  
그것만으로 분리의 이유가 되지는 않는다.

---

## 결정의 흐름

몇 가지 상황에서  
어떤 질문을 던지게 되는지 살펴본다.

### 큰 폼 컴포넌트

```tsx
// 상황: 필드가 많은 주문 폼
const OrderForm = () => {
  // 배송 정보
  const [name, setName] = useState('');
  const [address, setAddress] = useState('');
  const [phone, setPhone] = useState('');

  // 결제 정보
  const [cardNumber, setCardNumber] = useState('');
  const [expiry, setExpiry] = useState('');
  const [cvc, setCvc] = useState('');

  // 주문 옵션
  const [coupon, setCoupon] = useState('');
  const [usePoints, setUsePoints] = useState(false);

  // 유효성 검사, 제출 로직...
};
```

**판단 과정**

1. 변경의 이유가 몇 가지인가?  
   → 배송 정보 변경, 결제 방식 변경, 주문 옵션 변경.  
   **세 가지 이유가 있다.**

2. 나눈 조각들이 독립적으로 이해 가능한가?  
   → "배송 정보 입력", "결제 정보 입력"은 각각 의미가 있다.  
   **그렇다.**

3. 나눈 후 데이터 흐름은?  
   → 각 섹션은 자신의 필드만 관리하고,  
   제출 시점에만 데이터를 모으면 된다.  
   **단순해진다.**

**결론: 책임 단위로 분리**

```tsx
const ShippingSection = ({ values, onChange }) => (
  <fieldset>
    <legend>배송 정보</legend>
    <input value={values.name} onChange={e => onChange('name', e.target.value)} />
    <input value={values.address} onChange={e => onChange('address', e.target.value)} />
    <input value={values.phone} onChange={e => onChange('phone', e.target.value)} />
  </fieldset>
);

const PaymentSection = ({ values, onChange }) => (
  <fieldset>
    <legend>결제 정보</legend>
    <input value={values.cardNumber} onChange={e => onChange('cardNumber', e.target.value)} />
    <input value={values.expiry} onChange={e => onChange('expiry', e.target.value)} />
    <input value={values.cvc} onChange={e => onChange('cvc', e.target.value)} />
  </fieldset>
);

const OrderForm = () => {
  const [shipping, setShipping] = useState({ name: '', address: '', phone: '' });
  const [payment, setPayment] = useState({ cardNumber: '', expiry: '', cvc: '' });

  const handleShippingChange = (field, value) => {
    setShipping(prev => ({ ...prev, [field]: value }));
  };

  const handlePaymentChange = (field, value) => {
    setPayment(prev => ({ ...prev, [field]: value }));
  };

  return (
    <form>
      <ShippingSection values={shipping} onChange={handleShippingChange} />
      <PaymentSection values={payment} onChange={handlePaymentChange} />
      <button type="submit">주문하기</button>
    </form>
  );
};
```

**만약 폼이 단순하다면?**

필드가 3-4개이고 모두 같은 맥락이라면  
나누지 않는 것이 더 자연스럽다.

### 모드를 가진 컴포넌트

```tsx
// 상황: 보기/편집 모드가 있는 프로필
const UserProfile = ({ userId }) => {
  const [user, setUser] = useState(null);
  const [isEditing, setIsEditing] = useState(false);
  const [editedName, setEditedName] = useState('');
  const [editedBio, setEditedBio] = useState('');

  useEffect(() => {
    fetchUser(userId).then(setUser);
  }, [userId]);

  const handleEdit = () => {
    setIsEditing(true);
    setEditedName(user.name);
    setEditedBio(user.bio);
  };

  const handleSave = async () => {
    await updateUser(userId, { name: editedName, bio: editedBio });
    setUser({ ...user, name: editedName, bio: editedBio });
    setIsEditing(false);
  };

  if (!user) return <Loading />;

  if (isEditing) {
    return (
      <form onSubmit={handleSave}>
        <input value={editedName} onChange={e => setEditedName(e.target.value)} />
        <textarea value={editedBio} onChange={e => setEditedBio(e.target.value)} />
        <button type="submit">저장</button>
        <button type="button" onClick={() => setIsEditing(false)}>취소</button>
      </form>
    );
  }

  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.bio}</p>
      <button onClick={handleEdit}>편집</button>
    </div>
  );
};
```

**판단 과정**

1. 변경의 이유가 몇 가지인가?  
   → 보기 UI 변경, 편집 UI 변경, 저장 로직 변경.  
   **적어도 두 가지 이유가 있다.**

2. 나눈 조각들이 독립적으로 이해 가능한가?  
   → "프로필 보기"와 "프로필 편집"은 각각 의미가 있다.  
   **그렇다.**

3. 나눈 후 데이터 흐름은?  
   → 보기 컴포넌트는 user만, 편집 컴포넌트는 user와 onSave만 알면 된다.  
   **단순해진다.**

**결론: 모드별로 분리**

```tsx
const ProfileView = ({ user, onEdit }) => (
  <div>
    <h1>{user.name}</h1>
    <p>{user.bio}</p>
    <button onClick={onEdit}>편집</button>
  </div>
);

const ProfileEdit = ({ user, onSave, onCancel }) => {
  const [name, setName] = useState(user.name);
  const [bio, setBio] = useState(user.bio);

  return (
    <form onSubmit={() => onSave({ name, bio })}>
      <input value={name} onChange={e => setName(e.target.value)} />
      <textarea value={bio} onChange={e => setBio(e.target.value)} />
      <button type="submit">저장</button>
      <button type="button" onClick={onCancel}>취소</button>
    </form>
  );
};

const UserProfile = ({ userId }) => {
  const [user, setUser] = useState(null);
  const [isEditing, setIsEditing] = useState(false);

  useEffect(() => {
    fetchUser(userId).then(setUser);
  }, [userId]);

  const handleSave = async (updates) => {
    await updateUser(userId, updates);
    setUser({ ...user, ...updates });
    setIsEditing(false);
  };

  if (!user) return <Loading />;

  if (isEditing) {
    return <ProfileEdit user={user} onSave={handleSave} onCancel={() => setIsEditing(false)} />;
  }

  return <ProfileView user={user} onEdit={() => setIsEditing(true)} />;
};
```

각 모드가 자신의 관심사만 다룬다.  
보기 UI를 수정해도 편집 컴포넌트는 건드리지 않는다.

### 리스트와 아이템

```tsx
// 상황: 할일 목록
const TodoList = ({ todos, onToggle, onDelete }) => (
  <ul>
    {todos.map(todo => (
      <li key={todo.id}>
        <input
          type="checkbox"
          checked={todo.completed}
          onChange={() => onToggle(todo.id)}
        />
        <span style={{ textDecoration: todo.completed ? 'line-through' : 'none' }}>
          {todo.text}
        </span>
        <button onClick={() => onDelete(todo.id)}>삭제</button>
      </li>
    ))}
  </ul>
);
```

**판단 과정**

1. 아이템의 렌더링이 복잡해지고 있는가?  
   → 체크박스, 텍스트, 삭제 버튼, 스타일 조건부 적용.  
   **점점 복잡해지고 있다.**

2. 아이템 하나가 독립적으로 이해 가능한가?  
   → **그렇다.** "하나의 할일 항목"은 명확한 단위다.

**결론: 아이템 컴포넌트 추출**

```tsx
const TodoItem = ({ todo, onToggle, onDelete }) => (
  <li>
    <input
      type="checkbox"
      checked={todo.completed}
      onChange={() => onToggle(todo.id)}
    />
    <span style={{ textDecoration: todo.completed ? 'line-through' : 'none' }}>
      {todo.text}
    </span>
    <button onClick={() => onDelete(todo.id)}>삭제</button>
  </li>
);

const TodoList = ({ todos, onToggle, onDelete }) => (
  <ul>
    {todos.map(todo => (
      <TodoItem
        key={todo.id}
        todo={todo}
        onToggle={onToggle}
        onDelete={onDelete}
      />
    ))}
  </ul>
);
```

리스트에서 반복되는 아이템을 추출하는 것은  
가장 자연스러운 분리 중 하나다.

아이템의 렌더링이 변경되어도 리스트 구조는 바뀌지 않고,  
리스트의 정렬이나 필터링이 바뀌어도 아이템은 바뀌지 않는다.

### 데이터를 가져오고 표시하는 컴포넌트

```tsx
// 상황: 데이터를 가져와서 보여주는 컴포넌트
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
      <p>{user.bio}</p>
    </div>
  );
};
```

**판단 과정**

1. 변경의 이유가 몇 가지인가?  
   → API가 바뀌면 데이터 로직 변경, 디자인이 바뀌면 UI 변경.  
   **두 가지 이유가 있다.**

2. 어떻게 나눌 것인가?  
   → 데이터 페칭을 커스텀 훅으로 분리하면  
   컴포넌트는 표시만 담당한다.

**결론: 로직을 훅으로 분리**

```tsx
const useUser = (userId) => {
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

  return { user, isLoading, error };
};

const UserProfile = ({ userId }) => {
  const { user, isLoading, error } = useUser(userId);

  if (isLoading) return <Loading />;
  if (error) return <Error message={error.message} />;

  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
      <p>{user.bio}</p>
    </div>
  );
};
```

이 경우는 컴포넌트를 나눈 것이 아니라  
**로직을 분리**한 것이다.

컴포넌트 분리와 로직 분리는 다른 결정이다.  
로직을 훅으로 빼는 것에 대한 판단은  
"이 로직은 훅이어야 하는가"에서 더 자세히 다룬다.

---

## 나누는 것의 비용

나누면 항상 좋아지는 것은 아니다.

### 나누면 복잡도가 사라지는가?

나누는 것은  
복잡도를 **제거**하는 것이 아니라  
**재배치**하는 것이다.

```tsx
// 나누기 전: 복잡도가 한곳에 있다
const ProductPage = () => {
  // 상품 로직 + 구매 로직 + 리뷰 로직
  // 복잡하지만, 한 파일에서 모든 흐름을 볼 수 있다
};

// 나눈 후: 복잡도가 여러 곳에 분산된다
const ProductDetail = () => { /* 상품 로직 */ };
const PurchaseOptions = () => { /* 구매 로직 */ };
const ProductReviews = () => { /* 리뷰 로직 */ };
const ProductPage = () => { /* 조합 */ };
// 각각은 단순하지만, 전체를 보려면 네 파일을 넘나들어야 한다
```

분리의 가치는  
전체를 한꺼번에 볼 필요가 줄어들 때 생긴다.

리뷰 기능을 수정할 때  
ProductDetail이나 PurchaseOptions를 볼 필요가 없다면,  
분리가 도움이 된 것이다.

하지만 수정할 때마다  
여러 파일을 함께 봐야 한다면,  
분리가 오히려 비용이 된 것이다.

### 함께 변경되어야 하는데 나눴을 때

```tsx
// 나쁜 분리: 상품명과 가격을 별도 컴포넌트로
const ProductName = ({ name }) => <h1>{name}</h1>;
const ProductPrice = ({ price }) => <p>{price}원</p>;

const ProductCard = ({ product }) => (
  <div>
    <ProductName name={product.name} />
    <ProductPrice price={product.price} />
  </div>
);
```

상품명과 가격은  
레이아웃이 바뀌면 함께 바뀐다.

이것을 나누면  
변경할 때 항상 두 컴포넌트를 함께 수정해야 한다.  
분리의 의미가 없다.

### 나누기 위해 나누는 경우

```tsx
// "컴포넌트가 50줄이 넘으면 나눈다"
// 이런 규칙으로 기계적으로 분리하면

const FormTitle = () => <h2>주문서</h2>;
const FormDivider = () => <hr />;
const SubmitButtonWrapper = ({ children }) => (
  <div className="submit-area">{children}</div>
);
```

줄 수는 줄었지만  
의미 있는 분리가 아니다.

코드의 길이가 아니라  
**변경의 이유**가 분리의 기준이다.

---

## 정리하며 — 나누는 것은 경계를 긋는 것이다

컴포넌트를 나누는 것은  
코드를 짧게 만드는 기술이 아니다.

나누는 것은  
**변경의 경계를 긋는 것**이다.

### 선택지별 특성

| 선택지 | 언제 | 비용 |
|--------|------|------|
| 나누지 않는다 | 변경 이유가 하나일 때 | 코드가 길어질 수 있음 |
| 렌더링 영역 분리 | UI 구조를 명확히 할 때 | props 전달 증가 |
| 책임 단위 분리 | 변경 이유가 독립적일 때 | 데이터 흐름 설계 필요 |
| 합성 패턴 | 구조는 같고 내용이 다를 때 | 사용하는 쪽의 책임 증가 |

### 판단의 출발점이 될 수 있는 규칙들

절대적인 규칙은 아니지만,  
반복적으로 유효한 패턴들이 있다.

**1. 변경의 이유가 하나라면 나누지 않는다**

변경의 이유가 하나인 컴포넌트는  
길더라도 응집도가 높다.  
나누면 관련된 코드가 흩어진다.

**2. 나눈 조각이 독립적으로 이해 가능해야 한다**

나눈 컴포넌트가 혼자서 의미를 갖지 못하면  
분리가 아니라 조각내기에 가까워진다.

**3. 나누는 것은 복잡도를 제거하지 않는다**

복잡도는 사라지지 않고 재배치된다.  
각 조각이 독립적으로 다뤄질 수 있을 때  
재배치의 가치가 생긴다.

**4. 재사용은 분리의 보너스이지 목적이 아니다**

"나중에 다른 곳에서도 쓸 수 있으니까"는  
분리의 충분한 이유가 되지 않는다.  
분리의 기준은 변경의 독립성이다.

> 컴포넌트를 나눈다는 것은  
> 복잡도를 줄이는 것이 아니라  
> 복잡도를 다루는 단위를 정하는 것이다.  
> 그 단위가 변경의 이유와 일치할 때  
> 분리는 의미가 있다.
