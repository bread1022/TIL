# startTransition
> UI 생성을 차단하지 않고 state를 업데이트할 수 있다.

```js
startTransition(scope)
```

#### transition 적용을 한 경우

<img src="https://github.com/bread1022/TIL/assets/107349637/839c80dc-2dfa-4ed3-b622-b147bffdcfb0" width="150"/>

- UI blocking이 발생하지 않음
- 트랜지션으로 표시되므로 느리게 리렌더링되어도 사용자 인터페이스가 반응성을 유지할 수 있다.

#### transition 적용을 하지 않은 경우

<img src="https://github.com/bread1022/TIL/assets/107349637/3c271f24-4c8c-4ac7-b040-e0663534f727" width="150"/>

- UI blocking이 발생

## Parameters

- `scope` : **하나이상의 set함수(설정자함수)를 호출**하여 일부 state를 업데이트 하는 함수
  - 매개변수 없이 scope를 즉시 호출하고 scope 함수 호출 중에 동기적으로 예약된 모든 state 업데이트를 트랜지션으로 표시한다.
  - 전달된 함수는 동기적으로 동작해야한다.
    ```js
    const onClick = () => {
                // ❌ 비동기 함수안됨!
      startTransition(async() => setA((pre) => pre + 1))
    }
    ```
  - React에서 전달된 함수를 즉시 실행하여 실행하는 동안 발생하는 모든 state 업데이트를 트랜지션으로 표시하고 비동기로 작업하려고 하면 트랜지션으로 표시되지 않는다.


## 주의사항

- 해당 state의 set 함수에 접근할 수 있는 경우에만 트랜지션으로 update 로직을 감쌀 수 있다.
- **'pending'상태를 표시하기위해서는** **`useTransition`이 필요하다.**
- 일부 prop이나 custom hook 에 대한 응답으로 트랜지션을 시작하려면 `useDefferedValue`를 사용해야한다.
- `startTransition` 내부에 업데이트 함수는 다른 state 업데이트보다 우선순위가 밀리기 때문에 다른 업데이트에 의해 중단될 수 있다.
- 트랜지션 업데이트는 text input을 제어하는데 사용할 수 없다.
- `useTransition`과 유사하므로 `useTransition`을 사용할 수 없을 땐 `startTransition`을 사용할 수 있다.
  - `startTransition`은 데이터 라이브러리같은 외부 컴포넌트에서 사용할 수 있다.


## `startTransition` 로 state 업데이트 트랜지션 표시하기

```js
import { startTransition } from 'react'; // 🌟 리액트에서 바로 불러옴

const TabContainer = () => {
  const [tab, setTab] = useState('home');

  const selectTab = (tab) => {
  // ✅ transition으로 표시하면 interaction이 느려도 반응성을 막지않음
    startTransition(() => {
      setTab(tab);
    });
  };
}
```
- 트랜지션을 이용할 경우 느린 기기에서도 사용자 인터페이스 업데이트를 반응성 있게 유지할 수 있다는 장점을 가진다.
- 트랜지션을 사용하면 리렌더링중에도 UI의 반응성이 유지된다.


### `useTransition`과 `startTransition`의 차이점

```js
import { useTransition } from 'react';

const Component = () => {
  const [isPending, startTransition] = useTransition();

  const onClick = () => startTransition();

  // 🌟 'isPending'에 따른 transition 적용이 가능하다!
  // startTransition()을 호출하면 isPending = true되고
  // transition이 끝나면 isPending = false된다.
}
```

```js
import { startTransition } from 'react';

const Component = () => {
  // 🌟 'isPending'을 사용하지 않고 transition만 적용가능
  const onClick = () => startTransition();
}
```