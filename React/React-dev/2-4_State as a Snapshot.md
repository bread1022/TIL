# State as a Snapshot


## Setting state triggers renders
> state를 설정하면 렌더링이 촉발됩니다

- 인터페이스가 이벤트에 반응하려면 `state`를 업데이트해야한다. 사용자 이벤트에 반응하여 사용자 인터페이스(UI)가 직접 변경된다고 생각할 수 있다.
- UI가 이벤트에 반응하여 이벤트 핸들러가 실행되고, `setState`가 상태를 변화시켜 저장하면 새 렌더링을 Queue에 대기시킨다. 리액트는 새로운 `state`값에 따라 컴포넌트를 리렌더링한다.


## Rendering takes a snapshot in time
> 렌더링은 그 시점의 스냅샷을 찍는다

- 렌더링이란, 리액트가 컴포넌트를 호출한다는 의미이다.
- 컴포넌트 함수가 반환하는 JSX는 시간상 UI의 스냅샷과 같다.
- `prop`, `event handler`, `local variable` 등의 값은 모두 렌더링 순간의 `state`를 사용해 계산된다.


#### React가 리렌더링할 때
1. 리액트가 함수를 다시 호출한다.
2. 함수가 새 JSX 스냅샷을 반환한다.
3. 리액트가 반환한 스냅샷과 일치하도록 DOM tree를 업데이트한다.

## State over time
> 시간 경과에 따른 state

- 스냅샷은 사용자가 상호작용한 시점에 state스냅샷을 사용하기 때문에 state값은 이벤트 핸들러 코드가 비동기로 동작해도 렌더링하는 동안에는 변경되지 않는다.
- 리액트는 렌더링 이벤트 핸들러 내에서 state값을 고정으로 유지하기때문에 코드가 실행되는 동안 state의 변경에 영향을 받지 않는다. 만약 리렌더링 직전 최신 state값이 필요할 땐 `setState((prevState) => newState)` 에 값을 전달하는 게 아니라 함수를 전달한 `updater function`을 이용해야한다.