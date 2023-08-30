# useMemo
> 최상위 컴포넌트에서 useMemo를 사용하면
>
> 리렌더링 사이에 함수 호출 결과값을 캐시할 수 있게한다.


```js
const cachedValue = useMemo(calculateValue, dependencies)
```

### Parameters

- `calculateValue`: 캐시할 값을 계산하는 함수
  - 인자를 받지 않는, 어떠한 값이든 반환하는 **순수함수**이어야 한다.
  - 초기렌더링 중에 함수를 호출하여 값을 캐시한 다음,  
  이후 렌더링에서 의존성이 변경되지 않으면 이전에 캐시한 값을 반환하고  
  변경되었으면 `calculateValue`를 호출하여 새로운 값을 반환하고 이를 캐시한다.
  - 반환값을 캐싱하는 것을 `"memoization(메모화)"`라고 한다.
- `dependencies`: 캐시할 함수 내에서 참조되는 모든 반응형 값의 배열


### Return

- 초기 렌더링엔 `useMemo`의 매개변수로 전달한 **`calculateValue`의 함수를 호출하여 그 결과값을 반환**한다.
- 이후 렌더링부턴 의존성의 변경을 확인하여 변경되었을 경우 다시 내부 함수를 호출하여 새로운 값을 반환하고 그렇지 않을경우 이전에 캐시한 값을 반환한다.


### 주의사항

- `useMemo`는 Hook이기 때문에 컴포넌트의 최상위 레벨이나 커스텀훅에서만 호출할 수 있고, 조건문이나 반복문 안에서는 호출할 수 없다.
- 리액트는 캐시된 값을 유지하려고 한다.
  - 예외적인 경우
    - 초기마운트 중 컴포넌트가 일시중단되면 캐시를 폐기한다.
    - 성능최적화를 위해서는 `useMemo`가 적합하지만 그렇지않을경우에는 `변수` 또는 `ref`가 적절할 수도 있다.


<br>


### 계산비용이 많이 드는 함수 값 캐싱하는 방법

리렌더링간 계산값을 캐시하려면 컴포넌트 최상단에서 `useMemo`를 사용한다.

```js
function TodoList({ todos, tab, theme }) {
  const visibleTodos = filterTodos(todos, tab);
  // ...
}
```
- TodoList 컴포넌트의 Props가 변경될 때마다 컴포넌트는 전체 본문을 다시 실행한다. 이때 filterTodos가 일반함수이면서 고비용의 계산을 수행한다면 렌더링마다 함수를 호출하고 재계산해야된다.
- 이때 filterTodos 매개변수인 todos, tab이 렌더링 전후로 동일하다면 고비용의 계산을 건너뛰는게 효율적이기때문에 `useMemo`로 감싸서 이미 계산해놓은 값을 캐시하는게 좋다! => **`memoization(메모화)`**
  ```js
  import { useMemo } from 'react';

  function TodoList({ todos, tab, theme }) {
    const visibleTodos = useMemo(() => filterTodos(todos, tab), [todos, tab]);
    // ...
  }
  ```
  - `filterTodos`는 `todos`와 `tab`이 변경될 때만 호출된다.
  - 의존성이 변경되기 전까지는 초기 렌더링때 호출했던 계산 함수의 값을 캐시하고 있다.


### 고비용 계산 함수인지 확인하는 방법

- 수행하려는 인터렉션이 소요되는 시간을 콘솔로 찍어 확인했을 때 1ms 이상으로 확인된다면 메모해두는 것이 좋고 `useMemo`로 감싸서 시간이 감소했는지 확인해본다.  
  (이때 개발모드는 정확한 성능측정을 할 수 없기때문에 App을 빌드해서 사용자가 사용하는 것과 동일한 기기에서 테스트하여 정확한 타이밍을 측정하는 것이 좋음)
- `useMemo`는 첫 렌더링에는 함수를 무조건 호출해서 계산해야되므로 첫 렌더링에는 영향이 없고 업데이트시 불필요한 작업을 건너뛰는데 도움이 되는 hook이다.


### `useMemo`를 사용하여 최적화하는 것이 유용한 경우

1. 고비용계산이 수행되면서 의존성이 거의 변하지 않는 경우
2. `memo`로 감싼 컴포넌트에 props으로 값을 전달할 때 해당 값이 변경되지않아 리렌더링을 방지하고 싶은 경우  
   (이때 전달한 props는 또다른 `useMemo`, `useEffect` 의존성으로 사용될 수 있음)


#### 메모화를 사용하지않고 효율적으로 컴포넌트를 생성하는 방법

[useCallback에서의 효율적으로 컴포넌트 생성하는 방법과 같다 - 참고](https://github.com/bread1022/TIL/blob/master/React/Reference/Hook/useCallback.md)



### 컴포넌트 리렌더링 건너뛰는 방법

- 부모컴포넌트에서 `useMemo`를 사용하여 자식컴포넌트의 리렌더링 성능을 최적화하는 방법
  ```js
  export default function TodoList({ todos, tab, theme }) {
    const visibleTodos = filterTodos(todos, tab);
    // ❌ 고비용함수이나 반환값이 이전 렌더링과 변함이 없는 경우 => useMemo로 캐싱해보기!
    return (
      <div className={theme}>
        <List items={visibleTodos} /> // 컴포넌트를 호출할 때마다 새로운 props로 인식함
        // 자식컴포넌트에 전달하는 visibleTodos props가 변경되지 않는 값이라면
        // memo로 감싸서 리렌더링을 건너뛰게끔하는 것이 좋다!
      </div>
    );
  }
  ```
- `memo`로 컴포넌트를 감싸서 최적화하는 방법
  ```js
  import { memo } from 'react';

  // 이전 렌더링과 동일한 prop이 전달되는 경우 memo로 컴포넌트를 감싸서 최적화할 수 있다.
  const List = memo(function List({ items }) {
    // ...
  });
  ```
    - 하지만 이때 props로 전달되는 값이 객체나 배열이라면 `memo`로 감싸는 것만으로는 최적화가 되지않는다. (컴포넌트를 렌더링할때마다 다른 값을 반환한다고 리액트에서 인식하기 때문) => 이럴때 `useMemo`를 사용하는 게 유용하다!
- **`useMemo`를 사용해서 props로 전달하는 값을 캐싱하여 최적화하는 방법**
  ```js
  export default function TodoList({ todos, tab, theme }) {
    // ✅ 리렌더링사이에 반환값을 캐싱
    const visibleTodos = useMemo(() => filterTodos(todos, tab),
      [todos, tab] // 의존성이 변동되지 않는다면 캐싱해놓은 값을 반환
    );
    return (
      <div className={theme}>
      // 자식컴포넌트에 전달하는 visibleTodos props가 변경되지 않는 값이라면
      // 불필요한 리렌더링을 방지할 수 있음
        <List items={visibleTodos} />
      </div>
    );
  }
  ```
  - `useMemo`의 의존성이 변경될 때까지 리렌더링사이에는 캐싱된 값을 반환하면서 props가 동일하다고 판단하기 때문에 자식컴포넌트의 리렌더링을 방지할 수 있다.
  - 추가적인 방법으로 개별 JSX노드로 분리해서 메모화하는 방법도 있지만 조건부로 작업을 수행할 수 없기때문에 `useMemo`를 사용하는게 좋다.
    ```js
    export default function TodoList({ todos, tab, theme }) {
      const visibleTodos = useMemo(() => filterTodos(todos, tab), [todos, tab]);
      const children = useMemo(() => <List items={visibleTodos} />, [visibleTodos]);
      // JSX 반환하는 함수를 메모화히면
      // visibleTodos값이 변동되지않는 한 자식 컴포넌트를 리렌더링하지 않음
      return (
        <div className={theme}>
          {children}
        </div>
      );
    }
    ```

<!-- 하지만 React는 이전 렌더링 때와 동일한 JSX를 발견하면 컴포넌트를 리렌더링하려고 시도하지 않습니다. JSX 노드는 불변이기 때문입니다. JSX 노드 객체는 시간이 지나도 변경될 수 없으므로 React는 리렌더링을 건너뛰어도 안전하다는 것을 알고 있습니다. 다만 이것이 작동하려면 노드가 단순히 코드에서 동일하게 보이는 것이 아니라 실제로 동일한 객체여야 합니다.

이 예시에서 useMemo는 바로 이런 역할을 합니다. ?
JSX노드를 비교하는 역할? 무슨 역할? -->



<!-- memo만 사용해도 되는데 추가로 useMemo까지 사용해야되는 경우??? -->



### 다른 훅의 의존성 메모화


```js
function Dropdown({ allItems, text }) {
  const searchOptions = { matchMode: 'whole-word', text };

  const visibleItems = useMemo(() => {
    return searchItems(allItems, searchOptions);
  }, [allItems, searchOptions]); // ❌ 객체를 의존성에 넣으면 의미가 없음
}
```
  - 객체를 의존성으로 전달하기 위해 객체 자체를 메모화하는 것도 가능하다.
    ```js
    function Dropdown({ allItems, text }) {
      // ✅ 객체 자체를 메모화하고 의존성으로 전달
      const searchOptions = useMemo(() => {
        return { matchMode: 'whole-word', text };
      }, [text]);

      const visibleItems = useMemo(() => {
        return searchItems(allItems, searchOptions);
      }, [allItems, searchOptions]);
    }
    ```
  - `useMemo` 계산함수 내부에 객체를 선언하는 방법도 있다.
    ```js
    function Dropdown({ allItems, text }) {
      // ✅  Best : 계산함수 내부에 객체를 선언
      const visibleItems = useMemo(() => {
        const searchOptions = { matchMode: 'whole-word', text };
        return searchItems(allItems, searchOptions);
      }, [allItems, text]);
    }
    ```


### 🌟 함수 메모화 : `useMemo`과 `useCallback`, `useCallback`을 사용해야되는 때


```js
export default function ProductPage({ productId, referrer }) {
  function handleSubmit(orderDetails) {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails
    });
  }

  return <Form onSubmit={handleSubmit} />; // Memo로 감싸진 컴포넌트 Form에 함수를 전달할 때
}
```
- 위의 `handleSubmit` 함수는 컴포넌트가 리렌더링될 때마다 새로운 함수를 생성하기때문에 Form 컴포넌트가 `memo`로 감싸져있어도 메모화의 의미가 없다.
  - 객체 리터럴(`{}`)이 항상 다른 객체를 생성하는 것처럼 함수 선언 및 함수 표현식 등은 리렌더링마다 다른 함수를 생성하는 것과 같다.


```js
export default function Page({ productId, referrer }) {
  const handleSubmit = useMemo(() => {
    return (orderDetails) => {
      post('/product/' + productId + '/buy', {
        referrer,
        orderDetails
      });
    };
  }, [productId, referre]);

  return <Form onSubmit={handleSubmit} />;
}
```
- `useMemo`는 계산 함수가 실행되어 그 결과값이 반환되어야하므로 `useMemo`에 함수를 캐싱하기위해서는 캐싱하고싶은 함수를 반환하는 함수를 전달해야한다.
  - 함수를 메모화하는 React hook `useCallback`을 사용하면 중첩합수로 작성하지 않아도 된다는 장점이 있다.
  ```js
  export default function Page({ productId, referrer }) {
    // 🌟 함수를 캐싱할 땐 useCallback을 사용한다!
    const handleSubmit = useCallback((orderDetails) => {
      post('/product/' + productId + '/buy', {
        referrer,
        orderDetails
      });
    }, [productId, referrer]);

    return <Form onSubmit={handleSubmit} />;
  }
  ```


<br>



#### `useMemo`호출이 예상과 달리 undefined를 반환하는 경우

```js
const options = useMemo(() => {
  // ❌ options 는 undefined를 반환한다. (전달되는 값이 없음 그냥 실행되는 코드일뿐임)
  matchMode: 'whole-word',
  text: text
}, [text]);
```
- `useMemo`는 값을 반환해야하므로 `return`문을 명시적으로 작성해야한다.
  ```js
  const options = useMemo(() => {
    // ✅ 객체 리터럴을 반환하는 익명함수를 전달
    return {
      matchMode: 'whole-word',
      text: text
    };
  }, [text]);
  ```


#### 반복문에서 `useMemo`를 호출하려는데 허용이 안되는 경우? 최상위레벨에서 호출하기위해 컴포넌트를 분리해야한다.

```js
function ReportList({ items }) {
  return (
    <article>
      {items.map(item => {
        // ❌ 반복문안에서 useMemo 호출할 수 없음
        const data = useMemo(() => calculateReport(item), [item]);
        return (
          <figure key={item.id}>
          // memo로 감싸져있는 컴포넌트에 props 전달할 때
          // 부모컴포넌트가 리렌더링될때마다 차트를 계산하는 함수를 캐싱하고싶어서
          // useMemo를 반복문안에 넣어도 제대로 호출할 수 없음
            <Chart data={data} />
          </figure>
        );
      })}
    </article>
  );
}
```
- 반복문 안에서 `useMemo`를 사용하기보단 컴포넌트를 추출하여 개별 항목에 대한 데이터를 메모화하는 것이 좋다.
```js
function ReportList({ items }) {
  return (
    <article>
      {items.map(item =>
    // ✅ 메모화를 위해 개별 컴포넌트를 추출하고 그 내부에서 useMemo 호출하도록함
        <Report key={item.id} item={item} />
      )}
    </article>
  );
}

function Report({ item }) {
  // 🌟 최상위 레벨에서 useMemo 호출
  const data = useMemo(() => calculateReport(item), [item]);
  return (
    <figure>
      <Chart data={data} />
    </figure>
  );
}
```
- 또는 `useMemo`대신 `memo`로 감싸서 최적화할 수도 있다.
  - props의 값이 변하지 않으면 `memo`로 감싼 컴포넌트를 리렌더링하지 않기때문에 이것 또한 좋은 방법이다
  ```js
  function ReportList({ items }) {
    // ...
  }
  // 🌟 컴포넌트 자체를 최적화 하는 방법
  const Report = memo(function Report({ item }) {
    const data = calculateReport(item);
    return (
      <figure>
        <Chart data={data} />
      </figure>
    );
  });
  ```


----


[useMemo와 성능에 대해 참고하면 좋을만한 글](https://github.com/yeonjuan/dev-blog/blob/master/JavaScript/should-you-really-use-usememo.md)




[memo - React dev](https://react-ko.dev/reference/react/memo)