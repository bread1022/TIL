# `memo`
> 컴포넌트를 memo를 감싸면 메모화된 컴포넌트를 얻을 수 있게하는 함수
>
> 컴포넌트의 Props가 변경되지 않은 경우 리렌더링을 건너뛸 수 있다.


```js
const memoizedComponent = memo(Component, arePropsEqual?)
```
- memo로 감싸면 메모화된 컴포넌트를 얻을 수 있다.
- props가 변경되지 않는 한 부모컴포넌트가 리렌더링될 때 같이 리렌더링이 되지 않도록 하는 방법이지만 성능 최적화를 보장하진 않는다.
- `memo`는 성능 최적화를 위해서만 사용한다. (`memo`를 사용하지 않고 근본적인 문제를 해결하고 추가적으로 성능 개선을 위해 사용)
  - 컴포넌트에 전달되는 Props이 항상 다른경우에는 효과가 없으므로  
  `useCallback`, `useMemo`를 사용하여 컴포넌트에 전달되는 Props을 최적화할 때 사용한다.


### Parameters

- `Component`: 메모화할 컴포넌트
  - `memo`를 사용하면 컴포넌트를 수정하지 않고 새롭게 메모화된 컴포넌트를 반환한다.
- `arePropsEqual`: 새로운 prop과 이전 prop을 비교한 뒤 불리언값을 반환하는 함수로 일반적으로는 지정하지않는다.


### Return

- 새 리액트 컴포넌트를 반환  
  (부모가 리렌더링되어도 prop이 변경되지않으면 리렌더링하지 않음)


<br>


### prop이 변경되지 않았을 때 리렌더링 건너뛰는 방법

- 일반적으로 부모 컴포넌트가 리렌더링될 때마다 자식 컴포넌트도 함께 리렌더링되지만,  
  `memo`를 사용하면 부모 컴포넌트가 리렌더링될 때 새로운 props가 이전 props와 동일하면 리렌더링하지 않는 **메모화된 컴포넌트**를 생성할 수 있다.
- 리액트 컴포넌트는 props, state, context가 변경되지않는 한 항상 동일입력에 동일출력을 반환하는 순수함수이어야한다.
- `memo`를 사용하면 props가 변경되지않는 한 리렌더링을 건너뛸 수 있다. 하지만 `memo`를 적용하더라도 컴포넌트의 state나 context가 변경되면 리렌더링된다.


### state를 사용하여 메모화된 컴포넌트 업데이트하는 방법

- `memo`로 감싼 컴포넌트는 props에만 연관이 있고  
  내부 state나 context에는 연관이 없기때문에 해당 값이 변경되면 당연히 리렌더링된다.


### context를 사용하여 메모화된 컴포넌트 업데이트하는 방법

- `memo`는 오직 부모로부터 전달되는 props에만 연관이 있기때문에 사용중인 context가 변경되면 리렌더링된다.
- `context`에 따라 일부 컴포넌트만 리렌더링하고, 일부 컴포넌트의 리렌더링을 막고싶다면 context를 분리하여  
  외부 컴포넌트의 context에서 필요한 값을 사용하고, 메모화된 자식에게 Prop으로 전달하는 방법을 사용해야한다.


### 🌟 props를 최소화로 변경하는 방법

- `memo`를 사용하면 `Object.is` 비교를 통해 어떤 prop이든 이전과 이후 prop에 대한 얕은 비교를 한다.
- 따라서 `memo`를 사용하려면 props가 변경되는 시간을 최소화해야한다.  
  props로 객체를 전체 전달하는 대신 객체의 개별값을 사용하는 것이다.
    ```js
    function Page() {
      const [name, setName] = useState('Taylor');
      const [age, setAge] = useState(42);

      const person = useMemo(
        () => ({ name, age }),
        [name, age]
      );
      return <Profile person={person} />; // ❌ 계속해서 새로운 객체를 반환하기때문에 최적화가 무의미함
    }

    const Profile = memo(function Profile({ person }) {
      // ...
    });
    ```
    ```js
    function Page() {
      const [name, setName] = useState('Taylor');
      const [age, setAge] = useState(42);
      return <Profile name={name} age={age} />; // ✅ 개별 값에 따라 리렌더링을 건너뛸 수 있음
    }

    const Profile = memo(function Profile({ name, age }) {
      // ...
    });
    ```
- props로 함수를 전달해야될 땐, 변경되지 않도록 컴포넌트 외부에 선언하거나 `useCallback`을 사용하여 캐시하는 것이 좋다.


### 사용자 정의 비교 함수 지정하는 방법

- `memo`를 사용하여 컴포넌트를 감쌀 때 두번째 인자로 컴포넌트 비교 커스텀 함수를 매개변수로 전달할 수 있다.
- 새로운 props가 이전 props와 동일한 출력을 생성하는 경우 true를 반환하고 그렇지 않으면 false를 반환한다.
    ```js
    const Chart = memo(function Chart({ dataPoints }) {
      // ...
    }, arePropsEqual); // memo의 두번째 인자로 비교함수를 전달

    function arePropsEqual(oldProps, newProps) {
      return (
        oldProps.dataPoints.length === newProps.dataPoints.length &&
        oldProps.dataPoints.every((oldPoint, index) => {
          const newPoint = newProps.dataPoints[index];
          return oldPoint.x === newPoint.x && oldPoint.y === newPoint.y;
        })
      );
    }
    ```


#### prop이 객체, 배열, 함수일때 컴포넌트가 리렌더링되는 이유

리액트는 이전 Prop과 새로운 Prop을 얕은 비교로 동등성을 파악한다.  
부모컴포넌트가 리렌더링될 때마다 새로운 객체, 새로운 배열을 생성하면 개별 요소들이 모두 동일하더라도 리액트는 변경된 것으로 간주하기 때문에 자식 컴포넌트들도 리렌더링된다.  
이를 방지하기 위해서는 부모컴포넌트에서 [prop를 단순화하거나 메모화](#props를-최소화로-변경하는-방법)해야한다.