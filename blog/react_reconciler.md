React의 선언적 프로그래밍 모델은 개발자가 UI의 최종 상태를 기술하면 React가 나머지 복잡한 과정을 처리하는 강력한 패러다임입니다.
이 패러다임의 핵심에는 조정 이라는 숨겨ㄹ진 엔진이 있음
조정 알고리즘은 UI를 업데이트 해야할 떄, 이전 엘리머트 트리와 새로운 엘리먼트 트리를 비교하여 최소한의 DOM조작으로 원하는 상태를 구현하는 프로세스임
이 내부 메커니즘을 이해하는 것은 단순히 성능 문제를 해결하는 것을 ㄴ머어, 예측 간으하고 유지보수하기 쉬운 애플리케이션을 설계하는ㄷ ㅔ 필수적인 역량임

이 글의 목표는 조정 프로세스를 깊이 있게 탐구하고, 이것이 성능 ㅅ최적화 및 아키텍처 설계와 어떻게 직접적으로 연결되는지 분석하는것
이를 통해 개발자는 React.memo와 같은 최적화 도구에 의존하기 전에 근볹거으로 더 효율적이고 견고한 애플리케이션을 구축하는 설계 원칙을 습득할 수 있음

본 문서는 컴포넌트의 '식별성'이라는 기본 원리에서 출바해, 조정 알고리즘의 세가지 핵시 규칙을 살펴보고,
key 속성의 전략적인 활용법을 분석할 것
마지막으로 이런 지식을 바타응로 효과적인 아키텍처 패턴과 클링 아키텍처 원칙을 적용해 최적의 성능을 이끌어내는 방법을 제히하며 마무리할 것.

## 2. 컴포넌트 식별성과 상태 유지의 원리

React가 UI를 업데이트할 떄, 어떤 컴포넌트를 유지하고 어던 컴포너트를 새로 만들지 결정하는 기준은 바로 '식별성'임.
React가 컴포너트의 식별성을 어떻게 인식하느지 이해하는것은 상태 유실이나 붏필요한 리렌더링 과 같은 성능 문제의 근본 원인을 진단하는 척설음임.
이는 상태를 React를 통해 선언적으로 관리할지, DOM 자체에 맡길지를 결정하며, 이 결정은 성능과 복잡성에 대한 중대한 트레이드오프를 수반하기 때문임

```jsx
const UserInfoForm = () => {
	const [isEditing, setIsEditing] = useState(false);
	return (
		<div className='form-container'>
			<button onClick={() => setIsEditing(!isEditing)}>{isEditing ? 'Cancel' : 'Edit'}</button>
			{isEditing ? (
				<input
					type='text'
					placeholder='Enter your name'
					className='edit-input'
				/>
			) : (
				<input
					type='text'
					placeholder='Enter your name'
					disabled
					className='view-input'
				/>
			)}
		</div>
	);
};
```

- 이 컴포넌트에서 흥미로운 점은, 사용자가 Edit모드에서 input 필드에 텍스트를 입력한 후 Cancel버튼을 눌러도, 다시 Edit버튼을 눌렀을때 이전에 입력한 텍스트가 그대로 남아있다는 것임
- 두 Input엘리머튼느 서로 다른 props를 가지고 있음에도 불구하고 상태가 유지됨
- 이 현상은 조정 알고리즘의 관점을 명확히 설명함
- React는 렌더링 트리의 동일한 위치에 동일한 타입의 엘리먼트가 렌더링 되면 두 엘리먼트를 동ㅇ리한 것으로 간주함
- 이 경우, isEditing 상태가 변경되어도 해당 위치에는 여전히 input 타입의 엘리먼트가 존재하므로 React는 기존 DOM 엘리멈ㄴ트를 파괴하는 돼ㅔ신 단순히 props만 업덴이트함
- 따라서 DOM 엘리먼트 자체와 그 내부 상태가 보존되는것

```jsx
{
	isEditing ? (
		<input
			type='text'
			placeholder='Enter your name'
			className='edit-input'
		/>
	) : (
		<div className='view-only-display'>Name will appear here</div>
	);
}
```

- 이 경우, Edit모드를 토글할때 마다 엘리먼트 타입이 input에서 div로 변경됨
- React는 타입이 달렺ㅆ기에, 기존 input컴포너트를 완전히 마우트 해제하고 새로운 div을 마운트함
- 이 과전에서 input의 모든 내부 상태를 유실됨

- 이 두 예제는 조정 알고리즘의 핵심 원칙을 보여줌
- 즉, "엘리먼트 타입이 실별성을 결정하는 가장 중요한 요소"라ㅣ는 점

## 3. React의 조정 알고리즘 작동 방식

많은 개발자가 Rewact가 가상 DOM을 사용한다고 알고 있지만, 내부 동작을 더 정확하게 이해하기 위해서는 엘리먼트 트리라는 모델로 생각하느것이 유용함
엘리먼트 트리는 화면에 표시되어야 할 UI에 대한 가변운 자바스크립트 객체 기술서임
상태 변경으로 인해 리렌덜이이 발생하면, React는 새로운 엘리ㅁ너트 트리를 생성하고 이전 틀리와 비교하여 실제 DOM에 적용하 ㄹ최소한의 변경사항을 계산함
이 비교 과정이 바로 조정이며 다음 3가지 규칙을 따름

### 규칙 1. 엘리먼트 타입이 식별성을 결정한다.

- React는 트리를 비교할 떄 가장 먼저 엘리먼트의 type을 확인함
- 만약 루트 엘리먼트 타입이 다르다면, React는 이전 트리를 완전히 버리고 새로운 트리를 처음부터 구축

```jsx
// 첫 번째 렌더링
<div>
  <Counter />
</div>

// 두 번째 렌더링
<span>
  <Counter />
</span>
```

- 위 예제에서 루트 엘리먼트가 변경되었음
- React는 타입이 달라졌다고 판단하여ㅛ, 기존의 div 와 그 하위의 모든 컴포넌트
- 즉 Counter를 파괴함
- 이 과정ㅇ세ㅓ Counter가 가지고 있던 모든 상태는 사라짐
- 그 후, 새로운 span과 새로운 Countyer인스턴스가 생성하여 마운트됨

- 이 규칙은 컴포넌트 정의 위치의 중요성을 서렴ㅇ하는 핵심 그건이기도함
- 만약 부모 컴포넌트의 렌더함수 내부에 사직 커모펀트를 정의하면,
- 부모가 리렌더링 될때마다 매번 새로운 함수가 생성됨
- REact는 이를 타입 벽경으로 가주하여 규칙 1에 따라 해당 하위 트리를 불피룡하게 전부 마운트 해제하고 다시 마운트함
- 이는 아키텍트가 반드시 피해햐할 고전적인 ㅅ성능 안티패턴임

### 규칙 2. 트리의 위치가 식별성에 영향을 미친다.

- 엘리먼트의 타입이 동이하더라도 트리의 어떠 ㄴ위치에 렌더링되는지가 식별성에 영향을 미침
- React는 각 위치를 기준으로 이전트리와 새트리를 비교함

```jsx
<>{showDetails ? <UserProfile userId={123} /> : <LoginPrompt />}</>
```

- 이 조건부 렌더링 예제에서
- 프래그먼트의 첫 번쨰 자식 위치는 하나의 슬록으로 취급됨
- showDetails가 true에서 false로 바뀌면
- React는 이 슬롯에 있던 컴포넌트가 UserProfile에서 LoginPrompt로 변경되었음을 감지함
- 비록 두 컴포넌트가 다른 역할읗ㄹ하지만,
- REact의 관점에서는 같은 위치의 엘리머트 타입이 변경된것
- 따라서 규칙 1에 따락 기존 UserProfile컴포넌트는 마운트 해제되고, 새로운 LoginPropkmt가 마운트됨

### 규칙 3. key 속성을 위치 기반 비교를 재정의 한다.

- key속성은 React에게 컴포넌트의 식별성을 명시적을 ㅗ알려주는 강력한 도구임
- key를 사용하면 위치에 의존하는 기본 비교 동작을 재정의하여, React가 여러 렌더링에 걸쳐 특정 엘리먼트를 추적할 수 있도록 도와줌

```jsx
// 초기 상태
const users = [
	{ id: 1, name: 'Alice' },
	{ id: 2, name: 'Bob' },
];

// users 배열이 재정렬된 후
const users = [
	{ id: 2, name: 'Bob' },
	{ id: 1, name: 'Alice' },
];

// 렌더링 코드
users.map((user) => <li key={user.id}>{user.name}</li>);
```

- key가 없다면, REact는 첫 번쨰 li의 내용이 Alice에서 bob으로
- 두 번째 li의 내용이 boib에서 alice로 변경되었다고 판단하여 각 DOM노드의 내요ㅕㅇ을 수정할 것
- 하지만 key를 사용화면 React는 key1인 li와 key=2인 li를 식별하고, 단순히 이 두 DOM노드의 순서를 바꾸는 훨씬 효율적인 작업을 수행할것

## 4. key 속성의 전략적 사용

- key 속성은 단순히 리스트 렌더링시 발생하는 경고를 없애기 위한 장치가 아님
- 이는 REact조정 알고리즘에 직접적인 영향을 미쳐 컴포넌트의 생명주기와 상태를 정밀하게 제어하는 고급 전략 도구임

### 리스트렌더링에서의 핵심 역할

- key의 일반적인 사용 사례는 동적 리스틀 ㅔㄴ더링임'
- key가 없는 리스트에 새로운 아ㅓ이템이 수가되거나 삭제되면, REact는 위치기반으로만 엘리먼트를 비교함
- 예를 들어 리스트의 맨 앞에 아이템을 추가하면, React는 모든 후속 아이템들의 props가 변경되었다고 판ㅇ다해 비효율적인 전체 리렌더링을 수행할 수 있음

```jsx
<ul>
	{items.map((item) => (
		<li key={item.id}>{item.text}</li>
	))}
</ul>
```

- key로 각 아이템의 고유한 식별자를 부여하면
- React는 아이템의 순서가 바뀌거나 새로운 아이템이 추가되어도 각 아이템을 개별적으로 추젃할 ㅜㅅ 있음
- 이를 통해 DOM조작을 최소화하여 꼭 필요하 아이템만 추가, 삭제 또는 재정렬하는 효율적인 업데이트 가능

### 리스트 외부에서의 key 활요

- 다음 UserForm 예제를 보자
- 이 폼은 props로 바은 userId에 따라 다른 사용자 정보를 표시하며, 비제어 input을 사용함

```jsx
const UserForm = ({ userId }) => {
	return (
		<form>
			<input
				key={userId} // userId가 변경되면 input이 완전히 새로 생성됨
				name='username'
				defaultValue='' // 비제어 컴포넌트
			/>
		</form>
	);
};
```

- input이 비제어 컴포넌트 이므로, 사용자가 입력한 텍스트와 같은 상태는 REact상태가 아닌 실제 DOM노드에 직접 저장됨
- 만약 key가 없다면, userID가 A에서 B로 변경될 때
- React는 동일한 위치의 동일한 input으로 인식하고 DOM노드를 재사용할것임
- 결과적으로 사용자 A를 위해 입력했던 내용이 사용자 B의 폼에 남게됨

- 하지만 key={userId}를 추가하면, userId가 변경될떄마나 REact는 keyt가 달라져싿고 판다하여 기존 input DOM노드와 그오ㅘ 연결된 모든 상태를 폐기하고, 깨씃한 새 노드르 생성함
- 이 key 기반 초기화는 useEffect를 통해 수동으로 상태를 정ㄹ히난 로직을 구현하느것보다 훨씬 선언적이고 강력한 기법임

### 동적 요소와 정적 요소의 혼합 렌더링

- 동적 리스트 뒤에 정적 엘리먼특 오는 경우, 리스트의 길ㅇ기ㅏ 변하면 정적 엘리먼트가 불피룡하게 리마운트 될것이라는 우ㅠ료ㅕ가 있을 수 있음

```jsx
<>
	{items.map((item) => (
		<ListItem key={item.id} />
	))}
	<StaticElement /> {/* items 배열이 변경되어도 리마운트되지 않음 */}
</>
```

- React는 이렇나 시나리오는 지능적으로 처리함
- 내부넞긍로 items.map의 결과인 동적 배열 전체를 단일 자식 단위로 간주함
- 따라서 StaticElement는 항상 두 번째 자식 위치를 안정적으로 유지하며
- items 리스트의 길이나 순서가 변경되어도 식별성이 흔들리지 ㅇ낳아 리마운트 되지 않음
- 그러나 key와 컴포너트 타입이 함꼐 작동한다는점을 명심해야함
- key는 식별성을 제어하지만, 타입이 다른 컴포넌트간의 상태ㄹ를 보존할 수는 없음
- 예를 들어, key=content를 할당하더라도, PrfileTab key="content"과 SettingTab key="contemt" 사이를 전화하는것은 컴포넌트타입이 다르기 떄문에 항상 리마운트를 유발함
- 이 경우 상태 끄,ㄹ어올리긱를 통해 부모 컴포너트에서 상태를 관ㄹ히나늑서이 더 적절한 해결책임
- 결론적으로 key의 올바른 이해와 전략은 붎칠요한 DOM조작을 방지하고 예측 가능한 상태 관리를 가능하게 하는 핵심 기술임

## 5. 성능 최적화를 위한 아키텍처 패턴

- React의 성능 최적화는 React.memo와 같은 API를 사용한 개별 컴폰전트의 미세 조정을 넘어섬
- 진정한 쵲거화는 애플리케이션 전반적인 데이터흐름과 컴포넌트 구졸르 설계하는 아키텍처 수준에히ㅓ 리워짐
- 조정 알고리즘의 원릴르 이해하면 자연스럽게 선응이 뛰어나 아키텣거 패턴을 도추랗 ㄹ수 이ㅏㅆ음

### 1. 상태 지역화

- 상태 지역화는 상태를 사용하는 커뫂너트롸 최대한 가깝게 위치시키는 패터임
- 상태를 피룡이상 상위 컴포넌트에 두면, 상태가 변경될 떄마낟 관련업슨 수많으 자식 컴포너트가불피룡학 리렌덜이되는 문제가 잇음

```jsx
// 나쁜 예: 필터 텍스트가 변경될 때마다 App 전체가 리렌더링됨
const App = () => {
	const [filterText, setFilterText] = useState('');
	const filteredUsers = users.filter(/* ... */);
	return (
		<>
			<SearchBox
				filterText={filterText}
				onChange={setFilterText}
			/>
			<UserList users={filteredUsers} />
			<ExpensiveComponent /> {/* filterText와 관련 없음에도 리렌더링됨 */}
		</>
	);
};
```

```jsx
// 좋은 예: 상태를 UserSection으로 지역화
const UserSection = () => {
	const [filterText, setFilterText] = useState('');
	const filteredUsers = users.filter(/* ... */);
	return (
		<>
			<SearchBox
				filterText={filterText}
				onChange={setFilterText}
			/>
			<UserList users={filteredUsers} />
		</>
	);
};

const App = () => {
	return (
		<>
			<UserSection />
			<ExpensiveComponent />
		</>
	);
};
```

- UserSection 이라는 새로운 커모펀ㄴ트람 ㄴ들어 필터 관련 상태와 로직을 안으로 옮김
- 이제 필터 텍스트가 변ㄴ경되어도 리레런딜ㅇ 범위를 UserSection내부이며 ExpensiveComponent는 더이상 영향을 받지 ㅇ낳음
- 이 패턴은 성능을 직접적으로 샹상시킬 분 아니라, 더 중요하게는 명확하 ㄴ컴포넌트 경계를 강제함으로 아키텍처의 무결성을 높임

### 2. 관심사 분리를 통한 컴포넌트 설계

- 성능 문제는 종종 컴포넌트 설계 문제임
- 하나의 컴포너특 너무 많은 역할과 상태를 책임지면, ㅅ
- 사소한 변경 하나가 과련 없느 부분까지 연쇄적으로 리렌더링을 유발함
- React.memo를 적용하기 전에 이 커컴포넌트가 여러 관심사를 혼합하여 다루고 있는가 를 먼저 자문해야함

```jsx
// 나쁜 예: 여러 관심사가 혼합되어 있음
const ProductPage = ({ productId }) => {
	const [selectedSize, setSelectedSize] = useState('medium');
	const [quantity, setQuantity] = useState(1);
	const [shipping, setShipping] = useState('express');
	const [reviews, setReviews] = useState([]);
	// ...
	return (
		<div>
			<ProductInfo /* size, quantity props */ />
			<ShippingOptions /* shipping props */ />
			<Reviews reviews={reviews} />
		</div>
	);
};
```

- 이 구조에서는 사용자가 상푸 ㅁ사이즈를 변경할 떄마나 ProductPage전체가 리렌더링 되어
- 사이트오 ㅏ아무 상관없은 Revies섹션까지 불필요핟ㄱ 다시 그림

```jsx
// 좋은 예: ProductConfig와 ReviewsSection으로 관심사 분리
const ProductPage = ({ productId }) => {
  return (
    <div>
      <ProductConfig productId={productId} />
      <ReviewsSection productId={productId} />
    </div>
  );
};

const ProductConfig = ({ productId }) => {
  const [selectedSize, setSelectedSize] = useState("medium");
  // ... quantity, shipping 상태 관리
  return (/* ... */);
};

const ReviewsSection = ({ productId }) => {
  const [reviews, setReviews] = useState([]);
  // ... reviews 데이터 fetching
  return <Reviews reviews={reviews} />;
};
```

## 6. 조정과 클린 아키텍처

- React의 종정 메커니즘에 대한 깊은 이해는 단순히 성능 최적화 기술으 넘어섬
- 이는 견고하고 유지보수가 간으한 애플리케이션 구추갛기 우힌 클린 아키텍처원칙과 연결됨
- 조정 아록리즘의 삭종 방식에 부함하도곩 컴포넌트를 설계하는것은 자연스럽게 좋은 소프트웨어 공학 실저임

- 당니 책임 원칙:
  - 이 원칙은 하나의 커모펀트가 변경되어야 할 이유는 단 하나여야한다고 말함
  - 5장에 살펴본 ProductPage 리팩토링 사롂 이를 보여줌
  - 제품 설정과 리뷰ㅜ라느 2가지 변경 이유를 가졌더 ㄴ컴포넌트를 ProductCofig와 ReviewSection으로 분리함으로서 각 커포넌트는 단 하나의 고립된 벽ㄴ경 이유를 갖게 됨
- 의존성 역전 원칙:
  - 컴포넌트는 구체적인 구현이 아닌 추상화에 의존해야함
  - 5장의 UserSection패턴에서 이 원칙 확인 가은
  - UserSection을 생성ㅎㅁ으로써 App컴포너트는 더이상 필터 로직의 구체적인 구현에 의존하지 ㅇ낳고
  - UserSection이라는 추상화에 의존하게 됨
  - 이런 의존서 역전은 리렌덜이 경계를 격리시키는 효괒거인 방법임
- 인터페이스 분리 원칙:
  - 컴포넌트느 자신이 사용하지 않은 props에 의존해서는 안됨
  - 즉 컴포넌트는 최소한의 명확한 인터페이스props를 가져앟ㅁ
  - 거대한 props 객체를 통쨰로 넘기지 말고 명시적으로 전다랗면 관련없느 덷이터의 변경으로 인해 발새아흔 ㄴ불피룡하 리렌덜를 방지할 수 있음

## 7. 결론: 조정 알고리즘에 부합하는 설계의 가치

- 본 백서응ㄹ 토앻 우리는 React의 조정 알고리즘이 단순한 내부 구현 세부사항이 아니라
- 애플리케이션으 선능과 구조를 결저아흔 핵시 ㅁ원리임을 알게됨
- 조정 아로리즘의 작종 방식을 이해하는 것은, 성능이 뛰언 아키텍처를 설꼐하나 ㄴ길을 여렁줌

- 컴포넌트 정의를 부모 컴포넌트 외부에 두십시오. 렌더링 시마다 새로운 컴포넌트 타입이 생성되어 불필요한 리마운트를 유발하는 것을 방지합니다.
- 리렌더링 경계를 격리하기 위해 상태를 최대한 아래로 내리십시오(상태 지역화). 상태 변경의 영향 범위를 최소화하여 불필요한 업데이트를 줄입니다.
- 동일한 위치에서는 컴포넌트 타입을 일관되게 유지하십시오. 조건부 렌더링 시 타입이 변경되면 컴포넌트가 파괴되고 상태가 유실된다는 점을 기억하고, 이를 피하는 설계를 고려하십시오.
- 리스트뿐만 아니라 컴포넌트 식별성을 명시적으로 제어해야 할 때 key를 전략적으로 사용하십시오. 특히 비제어 컴포넌트의 상태를 초기화하는 강력한 도구로 활용할 수 있습니다.
