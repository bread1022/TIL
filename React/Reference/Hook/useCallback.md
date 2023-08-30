# useCallback
> 최상위 컴포넌트에서 useCallback을 호출하면,
>
> 리렌더링 사이에 함수 정의를 캐시할 수 있게한다.

```js
const cachedFn = useCallback(fn, dependencies)
```

### Parameters

- `fn`: 캐시할 함수
  - 초기 렌더링하는 동안 함수를 반환하고 (호출X) 다음 렌더링에서 마지막 렌더링 이후 의존성 배열에 변경이 없다면 동일한 함수를 다시 반환한다.
  - 현재 렌더링 중 전달한 함수를 제공하고 나중에 재사용할 수 있도록 저장한다.
  <!-- - React는 함수를 호출하지않고 함수를 반환하므로 호출 시기와 여부를 결정할 수 있다.-->
- `dependencies`: fn 코드 내에 참조된 모든 반응형 값의 배열 (함수 내 사용되는 컴포넌트 내부 모든 값을 포함하는 의존성배열)


### Return

- `useCallback`의 매개변수로 전달한 **`fn`함수를 초기렌더링에 반환**한다.
- 렌더링 중 마지막 렌더링에서 이미 저장된 `fn`함수를 반환하거나 렌더링 중 전달했던 `fn`함수를 반환한다.
- *초기렌더링에는 매개변수로 전달했던 함수를 반환하고 그 다음 렌더링부터는 React는 이전 렌더링에서 전달된 의존성과 비교하여 의존성 중 변경된 것이 없다면 `useCallback`은 이전 렌더링에서 반환한 함수를 반환하고 변경된 것이 있다면 이번 렌더링에서 전달한 함수를 반환한다.*
- **`useCallback`은 의존성이 변경되기 전까지는 리렌더링에 대해 함수를 캐시하는 것이다.**


### 주의사항

- `useCallback`은 Hook이기 때문에 컴포넌트의 최상위 레벨이나 커스텀훅에서만 호출할 수 있고, 조건문이나 반복문 안에서는 호출할 수 없다.
- React는 특별한 이유가 없는 한 캐시된 함수를 버리지 않는다.
  - 캐시된 함수를 버리는 경우
    - 개발단계에서 컴포넌트 파일을 수정할 때
    - 초기 마운트 중에 컴포넌트가 일시중단될 때


<br>

### 컴포넌트 리렌더링 건너뛰는 방법

- 렌더링 성능을 최적화할 때 자식 컴포넌트에 전달하는 함수를 캐시해야할 때가 있다. 리렌더링 사이에 함수를 캐시하기 위해선 함수 정의를 `useCallback`으로 감싸야한다.

```js
function ProductPage({ productId, referrer, theme }) {
  const handleSubmit = useCallback(() => {
    // ...
  }, [productId, referrer]);

  return (
    <div className={theme}>
      <ShippingForm onSubmit={handleSubmit} />
    </div>
  );
}
```
- 컴포넌트가 리렌더링될 때 모든 자식 컴포넌트들도 재귀적으로 리렌더링된다.  
  리렌더링할 때 많은 계산이 필요하지 않을 땐 괜찮지만  
  리렌더링하는데 많은 시간이 소요되는데 props가 지난 렌더링과 동일한 경우 `memo`로 감싸서 자식 컴포넌트의 리렌더링을 건너뛸 수 있다.

    ```js
    import { memo } from 'react';

    const ShippingForm = memo(function ShippingForm({ onSubmit }) {
      // ...
    });
    ```
  - Memo로 컴포넌트를 감싼경우 모든 props가 마지막 렌더링과 동일한 경우 리렌더링을 건너뛴다.
  - *이때 props로 전달한 함수를 useCallback으로 감싸지않으면 리렌더링마다 항상 다른 함수를 생성하기때문에 props는 계속 변경되어 memo로 감싼 의미가 없어진다. 리렌더링사이에 함수를 캐싱하도록 지시하기위해 `useCallback`을 사용해야하고, 의존성이 변경되지않는 한 리렌더링 사이에 동일한 함수가 되도록 할 수 있다.*


### `useCallback`과 `useMemo`

- `useMemo`와 함께 `useCallback`은 자식 컴포넌트를 최적화할 때 props를 캐시하는 데 자주 사용된다.
    ```js
    import { useMemo, useState } from 'react';

    function ProductPage({ productId, referrer }) {

      const product = useData('/product/' + productId);

      const requirements = useMemo(() => {
        // 함수를 호출하고 그 결과를 캐시
        return computeRequirements(product);
      }, [product]);

      const handleSubmit = useCallback((orderDetails) => {
        // 함수 자체를 캐시
        post('/product/' + productId + '/buy', {
          referrer,
          orderDetails,
        });
      }, [productId, referrer]);

      return (
        <div className={theme}>
          <ShippingForm requirements={requirements} onSubmit={handleSubmit} />
        </div>
      );
    }
    ```
- 차이점
  - `useMemo`: **함수를 호출하고 그 결과를 캐시**
    - 의존성이 변경되지않는 한 리렌더링사이에 동일한 결과를 반환
    - 필요시 렌더링 중 전달된 함수를 호출하고 결과를 계산
  - `useCallback`: **함수 자체를 캐시**
    - 제공한 함수를 호출하진않고 함수를 캐시하여 의존성이 변경되지않는 한 리렌더링사이에 동일한 함수를 반환
    - 이를 사용하면 props으로 전달한 함수가 변경되지 않기때문에 자식컴포넌트를 리렌더링하지않고 함수를 전달할 수 있다.

<!-- useMemo 읽고 내용 추가 -->


#### `useCallback`을 사용해서 함수를 캐싱하는게 유용한 경우
- memo로 감싼 자식컴포넌트에 props로 전달하는 함수를 캐싱할 때 메모화를 사용하면 의존성이 변경될 때만 리렌더링할 수 있다.
- props으로 함수를 전달하게되면 컴포넌트 내부 hook에서 의존성으로 사용될 수 있다. (`useCallback`으로 감싼 함수가 props로 전달된 함수에 의존하거나 `useEffect`에서 이 함수에 의존할 수 있음)


#### 메모화를 사용하지않고 효율적으로 컴포넌트를 생성하는 방법
1. 다른 컴포넌트를 감싸는 경우, JSX에서 children을 사용하여 자식 컴포넌트를 렌더링한다.
    ```js
    function Card({ children }) {
      return (
        <div className="card">
          {children}
        </div>
      );
    }
    ```
    - Wrapper 컴포넌트가 state를 업데이트해도 자식컴포넌트의 리렌더링이 필요하지 않다.
2. 필요이상으로 state를 끌어올리지 말고 일시적인 state는 로컬로 관리하는 것이 좋다.  
   트리상단에 state를 모두 끌어올렸다간 하위 컴포넌트가 불필요하게 리렌더링될 수 있다.
3. 렌더링 로직을 순수하게 유지하면 컴포넌트가 불필요하게 리렌더링되는 문제를 해결할 수 있다.
4. useEffect 내부에서 state를 업데이트하는 불필요한 로직은 최대한 지양해야한다.  
   대부분의 성능 문제는 Effect의 업데이트 체인에 의해 컴포넌트가 반복적으로 렌더링하는 경우가 많기때문이다.
5. useEffect의 의존성에서 불필요한 값을 제거한다.


*위의 원칙을 따랐는데도 불구하고 성능이 저하된다면 React DevTools Profiler를 사용하여 성능 문제를 진단하고 메모화가 필요한 컴포넌트를 찾아 최적화를 적용해볼 수 있다.*



### 🌟 메모된 callback에서 state를 업데이트하는 방법

```js
function TodoList() {
  const [todos, setTodos] = useState([]);

  const handleAddTodo = useCallback((text) => {
    const newTodo = { id: nextId++, text };
    setTodos([...todos, newTodo]);
  }, [todos]); // 🔺 todos의 변경에 따라 함수를 새로 생성함
}
```
```js
function TodoList() {
  const [todos, setTodos] = useState([]);

  const handleAddTodo = useCallback((text) => {
    const newTodo = { id: nextId++, text };
    setTodos(todos => [...todos, newTodo]);
  }, []); // ✅ Updater 함수를 사용하여 state를 업데이트 -> 의존성에서 제거해도됨
}
```
- 메모화된 함수는 최대한 적은 의존성을 가지는 게 좋다.
- 이전 state를 사용해서 다음 state를 계산해야되는경우에는 `updater`함수를 사용하여 의존성을 제거하도록 한다!



### Effect가 너무 자주 발생하지 않도록 하는 방법

- 가장 좋은 건 의존성배열에 함수를 넣지 않는 것이고, 의존성배열에 함수를 넣어야하는 경우에는 `useCallback`을 사용하여 함수를 캐싱하는 것이 좋다.
  ```js
  function ChatRoom({ roomId }) {
    const [message, setMessage] = useState('');

    function createOptions() {
      return {
        serverUrl: 'https://localhost:1234',
        roomId: roomId
      };
    }

    useEffect(() => {
      const options = createOptions();
      const connection = createConnection();
      connection.connect();
      return () => connection.disconnect();
    }, [createOptions]);
    // ❌ 렌더링시마다 함수가 변경되기 때문에 Effect가 계속 발생 => useCallback 사용해보기
  }
  ```
      렌더링마다 함수가 새로 생성되기때문에 Effect가 계속 발생한다.
  ```js
  function ChatRoom({ roomId }) {
    const [message, setMessage] = useState('');

    // 🔺 Effect에서 호출해야되는 함수를 useCallback으로 감싸서 캐싱
    const createOptions = useCallback(() => {
      return {
        serverUrl: 'https://localhost:1234',
        roomId: roomId
      };
    }, [roomId]); // 🔺 roomId 변경시에만 새로 생성

    useEffect(() => {
      const options = createOptions();
      const connection = createConnection();
      connection.connect();
      return () => connection.disconnect();
    }, [createOptions]); // 🔺 createOptions 변경시에만 변경됨
  }
  ```
      의존성이 동일한 경우 리렌더링 사이 useCallback으로 감싼 함수는 동일하게 적용된다.
  ```js
  function ChatRoom({ roomId }) {
    const [message, setMessage] = useState('');

    useEffect(() => {
      function createOptions() { // ✅ useCallback이나 함수에 대한 의존성이 필요하지 않음!
        return {
          serverUrl: 'https://localhost:1234',
          roomId: roomId
        };
      }

      const options = createOptions();
      const connection = createConnection();
      connection.connect();
      return () => connection.disconnect();
    }, [roomId]); // ✅ roomId 변경시에만 변경됨
  }
  ```
      가장 베스트는 함수의존성을 없애고 Effect내부로 이동시켜 최소한의 의존성만 남기도록한다.


### 🌟 custom hook 최적화하는 방법

```js
function useRouter() {
  const { dispatch } = useContext(RouterStateContext);

  const navigate = useCallback((url) => {
    dispatch({ type: 'navigate', url });
  }, [dispatch]);

  const goBack = useCallback(() => {
    dispatch({ type: 'back' });
  }, [dispatch]);

  return {
    navigate,
    goBack,
  };
}
```
- 커스텀훅을 작성한 경우 내부 함수들을 모두 useCallback으로 감싸면 좋다..!
- hook을 사용할 때마다 코드를 최적화할 수 있기때문이다.

<!-- 정말로 ?? ??????? -->


#### 리렌더링마다 `useCallback`에서 새로운 함수를 반환하는 경우?

```js
function ProductPage({ productId, referre }) {

  const handleSubmit = useCallback(() => {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }, []); // ❌ 의존성이 없기때문에 항상 새로운 함수를 반환

  return (
    <div className={theme}>
      <ShippingForm onSubmit={handleSubmit} />
    </div>
  );
}
```
- 의존성 배열을 제대로 지정했는지 확인하고, 알맞은 값을 지정한다. (의존성이 없다면 `useCallback`을 사용하는 의미가 없다)

  ```js
  function ProductPage({ productId, referrer }) {

    const handleSubmit = useCallback(() => {
      post('/product/' + productId + '/buy', {
        referrer,
        orderDetails,
      });
    }, [referrer, referrer]); // ✅ 의존성에서 하나 이상 변경이 있을 때만 새로운 함수를 반환

    return (
      <div className={theme}>
        <ShippingForm onSubmit={handleSubmit} />
      </div>
    );
  }
  ```
  - `useCallback`의존성배열을 추가하였는데도 새 함수를 반환하여 리렌더링이 발생된다면 수동으로 의존성에 대한 콘솔을 찍어서 디버깅할 수도 있다.
    ```js
    console.log([referrer, referrer])
    // 의존성 배열을 전역변수로 저장한다음 두배열의 각 의존성이 동일한지 확인
    Object.is(temp1[0], temp2[0]);
    Object.is(temp1[1], temp2[1]);
    ```


#### 반복문안에서 `useCallback`을 호출하려는데 허용이 안되는 경우? 최상위레벨에서 호출해야한다.

```js
function ReportList({ items }) {
  return (
    <article>
      {items.map(item => {
        const handleClick = useCallback(() => { // ❌ 반복문안에서 호출하면 안됨
          sendReport(item)
        }, [item]);

        return (
          <figure key={item.id}>
            <Chart onClick={handleClick} />
          </figure>
        );
      })}
    </article>
  );
}
```

```js
function ReportList({ items }) {
  return (
    <article>
      {items.map(item =>
        <Report key={item.id} item={item} /> // 1️⃣ 항목 컴포넌트를 추출하는 방법
      )}
    </article>
  );
}

function Report({ item }) {
  // 1️⃣ 최상위 레벨에서 useCallback을 호출
  const handleClick = useCallback(() => {
    sendReport(item)
  }, [item]);

  return (
    <figure>
      <Chart onClick={handleClick} />
    </figure>
  );
}
```

```js
function ReportList({ items }) {
  // ...
}

// 2️⃣ 컴포넌트 자체를 memo 하여 props인 item이 변경되지 않는한 리렌더링을 방지하는 방법
// props가 변경되지않으면 Report도 리렌더링되지않고, 당연하게 자식 컴포넌트들도 리렌더링되지않는다.
const Report = memo(function Report({ item }) {
  function handleClick() {
    sendReport(item);
  }

  return (
    <figure>
      <Chart onClick={handleClick} />
    </figure>
  );
});
```