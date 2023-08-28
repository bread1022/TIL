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
- dependencies: fn 코드 내에 참조된 모든 반응형 값의 배열 (함수 내 사용되는 컴포넌트 내부 모든 값을 포함하는 의존성배열)


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
- 컴포넌트가 리렌더링될 때 모든 자식 컴포넌트들도 재귀적으로 리렌더링된다. 리렌더링할 때 많은 계산이 필요하지 않을 땐 괜찮지만  
  리렌더링하는데 많은 시간이 소요되는데 props가 지난 렌더링과 동일한 경우 `memo`로 감싸서 자식 컴포넌트의 리렌더링을 건너뛸 수 있다.

    ```js
    import { memo } from 'react';

    const ShippingForm = memo(function ShippingForm({ onSubmit }) {
      // ...
    });
    ```
  - Memo로 컴포넌트를 감싼경우 모든 props가 마지막 렌더링과 동일한 경우 리렌더링을 건너뛴다.
  - **이때 props로 전달한 함수를 useCallback으로 감싸지않으면 리렌더링마다 항상 다른 함수를 생성하기때문에 props는 계속 변경되어 memo로 감싼 의미가 없어진다. 리렌더링사이에 함수를 캐싱하도록 지시하기위해 `useCallback`을 사용해야하고, 의존성이 변경되지않는 한 리렌더링 사이에 동일한 함수가 되도록 할 수 있다.**



### useCallback과 useMemo의 차이




### useCallback을 사용해야하는 곳




### 메모된 callback에서 state를 업데이트하는 방법




### Effect가 너무 자주 발생하지 않도록 하는 방법





#### useCallback은 컴포넌트가 리렌더링될때마다 다른 함수를 반환한다.
