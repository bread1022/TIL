# Queueing a Series of State Updates


## React batches state updates
> state 업데이트 일괄처리

- `state`를 설정하면 다음 렌더링이 `Queue`(대기열)에 들어간다.
- 리액트는 `state` 업데이트 전 이벤트 핸들러의 모든 코드를 실행한 이후에 `state`업데이트를 처리한다.
- `state`는 컴포넌트에서 처리하는 게 아니라 `state`를 관리하는 별도의 메모리공간에 `setState`를 **일괄 처리**(**batching**)한 후 다음 렌더링 때 업데이트된 `state`를 전달하여 렌더링한다.
- 하나의 이벤트 핸들러에서 state를 여러번 업데이트하는 과정이 있다면 `updater function`을 사용할 수 있다.


## Updating the same state multiple times before the next render
> 다음 렌더링 전에 동일한 state 변수를 여러 번 업데이트하기

- `setState`에 `state`값을 전달하지 않고, `updater function`을 전달하면 리액트는 `Queue`의 `prev state`를 기반으로 `next state`를 계산하는 함수를 전달할 수 있다. (리액트에 값을 대체하는 게 아니라 `state`값을 이용하여 무언가 계산하도록 지시하는 것)


## What happens if you update state after replacing it

```jsx
export default function Counter() {
  const [number, setNumber] = useState(0);
  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(number + 5);
        setNumber(n => n + 1);
      }}>Increase the number</button>
    </>
  )
}
```
- 위의 코드에서 이벤트 핸들러가 실행됐을 때 리액트에게 지시하는 작업 순서
  1. 리액트는 "값을 5로 바꾸기" 를 `Queue`에 추가한다.
  2. 그 다음 "n => n + 1"하는 `updater function`을 `Queue`에 추가한다.
  3. 다음 렌더링 동안 리액트는 `state Queue`를 순회하면서 최종 결과를 저장하여 `useState`에서 반환한다.


## What happens if you replace state after updating it

```jsx
export default function Counter() {
  const [number, setNumber] = useState(0);
  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(number + 5);
        setNumber(n => n + 1);
        setNumber(42);
      }}>Increase the number</button>
    </>
  )
}
```
- 위의 코드에서 이벤트 핸들러가 실행됐을 때 리액트에게 지시하는 작업 순서
  1. 리액트는 "number변수를 5로 바꾸기" 를 `Queue`에 추가한다.
  2. 그 다음 "n => n + 1"하는 `updater function`을 `Queue`에 추가한다.
  3. "42로 바꾸기"를 `Queue`에 추가한다.
  4. 다음 렌더링동안 리액트는 `Queue`를 순회하고 최종 결과를 저장하여 `useState`에서 반환한다.


- 이벤트 핸들러가 완료되면 리액트는 리렌더링을 실행한다.
- 리렌더링하는 동안 리액트는 `Queue`를 처리한다.

### updater function

- 업데이터함수의 네이밍은 `state`변수의 첫글자로 시작하거나 변수명을 반복해서 짓거나 prefix로 "prev"를 사용하는 것이 일반적이다.
- `Updater function`은 렌더링 중에 실행되는 함수이기때문에 순수해야하며 함수 내부에서 `state`를 변경하너 다른 사이드 이벡트를 실행하지 않고 결과만 반환해야한다.
