# You Might Not Need an Effect

Effect를 사용하면 컴포넌트를 네트워크, 브라우저DOM과 같은 외부시스템과 동기화할 수 있다.  
props나 state가 변경될 때 컴포넌트의 state를 업데이트하려는 경우(외부와 관련x)에는 Effect를 사용하지 않는다.
불필요한 Effect를 제거하면 컴포넌트가 더욱 단순해지고, 더욱 빠르게 렌더링된다.

## How to remove unnecessary Effects
> 불필요한 Effect를 제거하는 방법

#### Effect가 필요없는 경우

- 렌더링을 위해 데이터를 변환하는 경우
  - state 업데이트함수를 Effect 내부에 작성하는 것보다 최상위 컴포넌트에서 데이터를 변환하여 props나 state로 전달하는 것이 좋다. 그럼 자동으로 prop가 변경될 때마다 자동으로 리렌더링되기 때문이다.
- **사용자와 특정 상호작용**으로 인해 state를 업데이트하는 경우
  - 사용자의 액션에 의한 state 업데이트가 Effect 내부에 작성되면 어떤 액션에 의해 업데이트되는지 추적하기 어렵기때문에 사용자 이벤트는 이벤트핸들러에서 처리하는 것이 좋다.
- ***Effect 대신 key로 state 재설정하는 방법이나, 렌더링 중에 계산이 가능하면 state를 사용하지 않고 렌더링 중에 계산하는 방법을 사용하는 것이 가장 좋다!***


#### Effect가 필요한 경우

- 외부 시스템과 동기화해야하는 경우 (데이터 요청)
- ***컴포넌트가 사용자에게 표시된 후 실행되어야하는 코드에만 Effect***를 사용한다. (나머지는 이벤트 핸들러에!)


## Updating state based on props or state
> props 또는 state에 따라 state 업데이트하기

props나 state에서 계산이 가능한 것은 state로 관리하지 않고 렌더링 중에 계산하여 사용한다.  
그래야 계단식 업데이트를 방지할 수 있고 일부 코드를 제거하고  
서로 다른 state변수가 동기화되지 않아 발생하는 버그를 피할 수 있다.

```js
function Form() {
  const [firstName, setFirstName] = useState('Taylor');
  const [lastName, setLastName] = useState('Swift');
  const [fullName, setFullName] = useState(''); // ❌ 중복 state 및 불필요한 Effect
  useEffect(() => {
    setFullName(firstName + ' ' + lastName);
  }, [firstName, lastName]);
  // ...
}
```

```js
function Form() {
  const [firstName, setFirstName] = useState('Taylor');
  const [lastName, setLastName] = useState('Swift');

  const fullName = firstName + ' ' + lastName; // ⭕️ 렌더링 중 계산하는 방법
  // ...
}
```


## Caching expensive calculations
> 고비용 계산 캐싱하기

```js
function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState('');
  const [visibleTodos, setVisibleTodos] = useState([]); // ❌ 중복 state 및 불필요한 Effect

  useEffect(() => {
    setVisibleTodos(getFilteredTodos(todos, filter));
  }, [todos, filter]);
  // ...
}
```

```js
function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState('');
  const visibleTodos = getFilteredTodos(todos, filter); // ⭕️ 렌더링 중 계산하는 방법
  // ...
}
```

```js
import { useMemo, useState } from 'react';

function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState('');

  // ✅ todos, filter가 변경될때만 재실행됨
  const visibleTodos = useMemo(() =>
    getFilteredTodos(todos, filter); // 해당 함수와 관련없는 state가 변경되었을 때 불필요한 계산을 하지 않도록함
  , [todos, filter]);
  // ...
}
```
- ### `useMemo`
  - 초기 렌더링의 반환값을 기억해두었다가 렌더링 중에 의존성이 다른지 확인하고 동일하다면 마지막 저장 결과값을 반환하고
  - 같지않다면 내부함수를 다시 호출하고 그 결과를 저장한다.
  - `useMemo`는 업데이트할 때 불필요한 작업을 건너뛰는데에만 도움이 되기때문에 첫번째 렌더링 성능과는 상관이 없다.
  - 따라서 비용이 많이 드는 계산을 캐시할 때 사용하는 것이 좋다.
  - 브라우저에서 성능을 저하시켜서 테스트하고싶다면 [Chrome CPU Throttling](https://developer.chrome.com/blog/new-in-devtools-61/#throttling)을 참고하자.


## Resetting all state when a prop changes
> prop이 변경되면 모든 state 재설정하기


```js
export default function ProfilePage({ userId }) {
  const [comment, setComment] = useState('');

  useEffect(() => {
    setComment(''); // 초기화시켜야되는 로직을 Effect에 작성함으로써 불필요한 Effect가 발생함
  }, [userId]);  // ❌ userId가 변경될 때마다 불필요한 Effect
  // 첫번째 렌더링으로 이전값을 렌더링한 뒤, 새로운 값으로 다시 렌더링을 발생시키므로 비효율적임!!!!!!!!!
  return (
    // ...
  )
}
```

```js
export default function ProfilePage({ userId }) {
  return (
    <Profile
      userId={userId}
      key={userId} // ✅ Key를 전달함으로써 props가 변경될 때마다 다른 컴포넌트임을 명시함
    />
  );
}

// ✅ React에서 다른 Key를 전달하면 다른 컴포넌트로 인식하고 state가 자동으로 재설정됨
function Profile({ userId }) {
  const [comment, setComment] = useState('');
  // ...
}
```



## Adjusting some state when a prop changes
> props가 변경될 때 일부 state 조정하기

```js
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selection, setSelection] = useState(null);

  useEffect(() => {
    setSelection(null);
  }, [items]); // ❌ props items가 변경될 때마다 불필요한 Effect
  // ...
}
```
- 위의 코드처럼 useEffect 내부에 로직을 넣게되면, 저장되어있던 오래된 값으로 초기 렌더링을 진행한 뒤 React는 DOM을 업데이트하고 Effect를 실행한다. 그 다음 Effect내부의 로직이 실행되면서 다시 업데이트된 값으로 렌더링을 진행하게 되므로 비효율적이다.  
  그러므로 렌더링 중에 직접 state를 조정할 수 있게끔 로직을 수정하는 것이 좋다.


```js
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selectedId, setSelectedId] = useState(null);

  // ✅ (Best) props가 변경되면 저절로 렌더링되면서 값을 계산하므로 Effect가 필요없음
  const selection = items.find(item => item.id === selectedId) ?? null;
  // ...
}
```



## Sharing logic between event handlers
> 이벤트 핸들러 간 로직 공유


```js
function ProductPage({ product, addToCart }) {
  // ❌ 각각 다른 클릭이벤트가 같은 알림을 표시하기 때문에 useEffect안에 넣고 싶겠지만..
  // 새로고침 등 불필요한 Effect와 버그를 발생시킬 가능성이 높음!!
  useEffect(() => {
    if (product.isInCart) {
    showNotification(`Added ${product.name} to the shopping cart!`);
    }
  }, [product]);

  function handleBuyClick() {
    addToCart(product);
  }

  function handleCheckoutClick() {
    addToCart(product);
    navigateTo('/checkout');
  }
  // ...
}
```

```js
function ProductPage({ product, addToCart }) {
  // ✅ 이벤트 핸들러 간 공유되는 로직을 함수로 분리하여 호출
  function buyProduct() {
    addToCart(product);
    showNotification(`Added ${product.name} to the shopping cart!`);
  }

  function handleBuyClick() {
    buyProduct();
  }

  function handleCheckoutClick() {
    buyProduct();
    navigateTo('/checkout');
  }
  // ...
}
```


## Initializing the application
> 애플리케이션 초기화하기

App이 로드되고 딱 한번만 실행되어야하는 초기화 로직을 useEffect에 넣었다가  
컴포넌트가 개발모드에 다시 마운트하면서 함수가 두번 호출되는 문제가 발생할 수 있다.  
이를 방지하기 위해 컴포넌트 마운트당 한번이아니라  
앱 로드당 한.번. 실행되어야할 땐 변수로 초기화를 체크하고 초기화가 되었으면 다시 실행하지 않는 조건문을 추가해야한다!

```js
let didInit = false;

function App() {
  useEffect(() => {
    if (!didInit) {
      didInit = true; // ✅ 초기화가 되었으면 다시 실행하지 않기때문에 앱 로드당 한번만 실행됨
      loadDataFromLocalStorage();
      checkAuthToken();
    }
  }, []);
  // ...
}
```
- 앱전체 초기화 로직은 루트컴포넌트 모듈이나 App 엔트리 포인트에 유지하는 것이 좋다.  
  임의의 컴포넌트를 import할 때 속도 저하나 예상치 못한 동작이 발생할 수 있기 때문이다.




## Subscribing to an external store
> 외부 스토어 구독하기

### `useSyncExternalStore` Hook
> 외부 저장소를 구독할 때 사용

- useEffect로 데이터를 가져와서, state를 수동으로 동기화하는 것보다 오류가 적다.

```js
function subscribe(callback) {
  window.addEventListener('online', callback);
  window.addEventListener('offline', callback);
  return () => {
    window.removeEventListener('online', callback);
    window.removeEventListener('offline', callback);
  };
}

function useOnlineStatus() {

  return useSyncExternalStore(
    subscribe, // React는 동일한 함수를 전달하는 한 다시 구독하지 않음
    () => navigator.onLine, // 클라이언트 값
    () => true  // 서버에서의 값
  );
}

function ChatIndicator() {
  const isOnline = useOnlineStatus();
  // ...
}
```


## Fetching data
> 데이터 페칭하기

- Fetch 함수는 타이핑이벤트로 발생하는 것이 아니기 때문에 이벤트핸들러에 넣을 필요가 없다.
- 클린업함수를 사용하여 마지막으로 요청된 응답을 제외한 응답을 무시하도록 하는 게 좋다. (경쟁 조건 방지)