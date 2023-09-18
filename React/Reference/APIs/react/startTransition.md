# startTransition
> UI 생성을 차단하지 않고 state를 업데이트할 수 있다.

```js
startTransition(scope)
```

<!-- startTransition 함수를 사용하면 state 업데이트를 트랜지션으로 표시할 수 있다 ? 모르겠따 동영상 보고 다시 정리할 것.-->


## Parameters

- scope : 하나이상의 set함수(설정자함수)를 호출하여 일부 state를 업데이트 하는 함수
  - 매개변수 없이 scope를 즉시 호출하고 scope 함수 호출 중에 동기적으로 예약된 모든 state 업데이트를 트랜지션으로 표시한다.
  - 전달된 함수는 동기적으로 동작해야한다.
  - React에서 전달된 함수를 즉시 실행하여 실행하는 동안 발생하는 모든 state 업데이트를 트랜지션으로 표시하고 비동기로 작업하려고 하면 트랜지션으로 표시되지 않는다.


## 주의사항

- 해당 state의 set 함수에 접근할 수 있는 경우에만 트랜지션으로 update 로직을 감쌀 수 있다.
- 'pending' 상태를 표시하기위해서는 `useTransition` 이 필요하다.
- 일부 prop이나 custom hook 에 대한 응답으로 트랜지션을 시작하려면 `useDefferedValue`를 사용해야한다.
- 트랜지션으로 표시된 state 업데이트는 다른 state 업데이트에 의해 중단된다.
- 트랜지션 업데이트는 text input을 제어하는데 사용할 수 없다.
- useTransition과 유사하므로 useTransition을 사용할 수 없을 땐 startTransition을 사용할 수 있다.
  - startTransition은 데이터 라이브러리같은 외부 컴포넌트에서 사용할 수 있다.


## state 업데이트를 논블로킹 트랜지션으로 표시


```js
import { startTransition } from 'react';

const TabContainer = () => {
  const [tab, setTab] = useState('home');

  const selectTab = (tab) => {
    startTransition(() => {
      setTab(tab);
    });
  };
}
```
- 트랜지션을 이용할 경우 느린 기기에서도 사용자 인터페이스 업데이트를 반응성 있게 유지할 수 있다는 장점을 가진다.
- 트랜지션을 사용하면 리렌더링중에도 UI의 반응성이 유지된다.