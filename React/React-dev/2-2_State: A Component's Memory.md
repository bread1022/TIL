# State: A Component's Memory

## When a regular variable isn’t enough
> 일반 변수로 충분하지 않은 경우

- 리액트는 컴포넌트를 렌더링할 때 지역변수의 변경을 고려하지 않아 변경 사항이 있어도 리렌더링되지 않는다.
- 컴포넌트를 새 데이터로 업데이트하기 위해서는 렌더링 동안 데이터를 유지하며 변경된 새 데이터로 컴포넌트를 리렌더링하도록 리액트를 트리거한다.
- 렌더링 사이에 데이터를 유지하기 위해 state 변수와 변수를 업데이트하고 리액트 컴포넌트가 리렌더링하도록 트리거하는 state 업데이트 함수를 사용한다.

```jsx
export default function Gallery() {
  let index = 0;
  function handleClick() {
    index = index + 1;
  }
  let sculpture = sculptureList[index];
  return (
    <>
      <button onClick={handleClick}>
        Next
      </button>
      <h3>
        ({index + 1} of {sculptureList.length})
      </h3>
      <img
        src={sculpture.url}
        alt={sculpture.alt}
      />
    </>
  );
}
```
    지역변수는 값이 변경되어도 리액트의 리렌더링을 트리거하지 않는다.

```jsx
import { useState } from 'react';
import { sculptureList } from './data.js';

export default function Gallery() {
  const [index, setIndex] = useState(0);
  function handleClick() {
    setIndex(index + 1);
  }
  let sculpture = sculptureList[index];
  return (
    <>
      <button onClick={handleClick}>
        Next
      </button>
      <h3>
        ({index + 1} of {sculptureList.length})
      </h3>
      <img
        src={sculpture.url}
        alt={sculpture.alt}
      />
    </>
  );
}
```
    index의 값이 변할 때마다 리액트의 리렌더링을 트리거하기 위해 useState 훅을 사용한다.


## Adding a state variable
> state 변수 추가하기

- `let index = 0` 일반 지역변수를 `const [index, setIndex] = useState(0)` state 변수로 변경한다.
- 리액트의 `useState` 훅은 state 변수와 state 업데이트 함수를 반환한다.
  - `배열 구조 분해`를 사용하여 state 변수와 state 업데이트 함수를 변수에 할당하여 사용한다.

<br>

## `useState` Hook
> 리액트 컴포넌트에서 상태를 기억하고 업데이트하는 데 사용하는 훅

- **`useState` 매개변수**
  - state 변수의 초기값
- **`useState` 반환값**
  - 컴포넌트가 렌더링될 때마다 `useState`는 두개의 값을 포함하는 배열을 반환한다.
  - `state variable = state` : 렌더링 사이에 데이터를 유지하기 위한 변수, 저장한 현재 값을 갖는다.
  - `state setter function = setState` : state 변수를 업데이트하고 리액트 컴포넌트가 리렌더링하도록 트리거하는 함수
- 동작 방식
  1. 컴포넌트의 첫 렌더링 때 초기값으로 전달한 인자를 `state값`으로 기억한다.
  2. 유저의 행동에 의해 `state값`이 변경되면 `setState`함수를 호출하여 `state값`을 업데이트하고, state가 변하면 다음 렌더링을 트리거한다.
  3. 컴포넌트가 두번째로 렌더링될 때, 리액트는 useState 훅이 변경된 `state값`을 기억하고 있기 때문에 [`변경된 state`, `setState`]를 반환한다.


### `Hook` 이란 ?
- 리액트에서 `use`로 시작하는 함수를 훅이라고 부른다.
- 리액트가 렌더링 중일 때만 사용할 수 있는 함수이다.
- 훅은 **컴포넌트의 최상위 레벨**(최상위 컴포넌트X)또는 **커스텀훅**에서만 호출이 가능하다.
- 조건문, 반복문 또는 중첩 함수 내부에서는 호출할 수 없다.

## Giving a component multiple state variables
> 컴포넌트에 여러 state 변수 지정하기

- `useState` 훅을 여러번 호출하여 하나의 컴포넌트에 여러 state 변수를 지정할 수 있다.
- 서로 연관이 없는 경우에는 여러개의 `state` 변수로 분리하여 갖는 것이 좋다.
- But, `state` 변수를 자주 함께 변경하는 경우에는 두 변수를 객체로 합쳐서 하나의 `state`로 관리하는 것이 더 편리할 때도 있다.
- [state 구조화 원칙](https://react-ko.dev/learn/choosing-the-state-structure)
- 동일한 컴포넌트를 여러개 호출하여도 각각의 컴포넌트는 독립적인 `state`를 갖기때문에 서로 영향을 주지 않는다.
- `state`는 선언하는 컴포넌트 외에는 공개되지 않는 정보로 다른 컴포넌트에 영향을 주지 않고 `state`를 변경할 수 있다.
- 여러 컴포넌트에서 하나의 `state`를 동기화하기 위해서는 가장 가까운 공유 부모 컴포넌트에 `state`를 추가하는 것이다.

