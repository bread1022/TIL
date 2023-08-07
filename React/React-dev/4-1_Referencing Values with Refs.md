# Referencing Values with Refs

컴포넌트가 데이터를 저장하면서 이 데이터의 상태에 의해 렌더링을 촉발하지 않게 하기 위해서 `ref`를 사용한다.


## Adding a ref to your component


```js
import { useRef } from 'react';

const ref = useRef(0);
```
- React에서 `useRef`Hook을 import하여 사용하며 초기값을 인자로 전달한다.
- `useRef`는 `current`라는 속성을 가진 객체를 반환한다.
  ```js
  {
    current: 0
  }
  ```
- React에서 `ref.current` 속성으로 선택한 ref의 현재값에 접근할 수 있고 값을 할당할 수도 있다. (읽기, 쓰기가 모두 가능)
- ref에는 state, string, object, function 등 모든 타입의 값을 가리킬 수 있고 current속성을 읽고 수정할 수 있는 일반 JS객체이다.
- `state`와 마찬가지로 React에 의해 리렌더링사이에 `ref`는 유지되지만  
  `ref`의 값이 변하여도 컴포넌트 리렌더링을 트리거하진않는다. (렌더링에 사용되는 값이 아님)
- 리렌더링에 사용되는 정보는 `state`로 유지하고  
  이벤트 핸들러에서만 정보를 필요로 하고 변경해도 리렌더링이 필요하지 않는 경우 `ref`를 사용한다.


### Example: building a stopwatch


```js
import { useState, useRef } from 'react';

export default function Stopwatch() {
  const [startTime, setStartTime] = useState(null); // 렌더링에 필요한 정보
  const [now, setNow] = useState(null);
  const intervalRef = useRef(null); // 이벤트핸들러에서만 값을 필요로함

  function handleStart() {
    setStartTime(Date.now());
    setNow(Date.now());

    clearInterval(intervalRef.current);
    intervalRef.current = setInterval(() => {
      setNow(Date.now());
    }, 10);
  }

  function handleStop() {
    clearInterval(intervalRef.current);
  }

  let secondsPassed = 0;
  if (startTime != null && now != null) {
    secondsPassed = (now - startTime) / 1000;
  }

  return (
    <>
      <h1>Time passed: {secondsPassed.toFixed(3)}</h1>
      <button onClick={handleStart}>
        Start
      </button>
      <button onClick={handleStop}>
        Stop
      </button>
    </>
  );
}
```


## Differences between refs and state

refs|state
---|---
`{ current : initialValue } = useRef(initialValue)`|`[state, setState] = useState(initialValue)`
변경되어도 리렌더링을 트리거하지 않음 | 변경시 리렌더링을 트리거함
렌더링과정 외부에서 current값을 수정하고 update할 수 있음 | `setState`함수를 통해서 `state`변수를 리런더링 대기열에 추가해야됨
**렌더링과정 중에는 `ref.current`값을 읽거나 쓰지 않아야함** | 렌더링에 변경되지않는 `state snapshot`이 있기때문에 언제든지 `state`를 읽을 수 있음
ref.current는 언제 변경되는 지 알 수 없어 렌더링중에 읽으면 컴포넌트의 동작을 예측하기 어려움 | _


### `useRef`의 내부동작원리

```js
function useRef(initialValue) {
  const [ref, unusedSetRef] = useState({ current: initialValue });
  return ref;
}
```
- 첫렌더링에 `useRef`는 `{ current: initialValue }`를 반환한다.
- React에 의해 저장되어 다음 렌더링 중에도 동일한 객체가 반환된다. (`setState`함수가 사용되지않음)


## When to use refs

- 외부 API, 브라우저 API 와 통신할 때(UI에 영향을 주지 않는 상황)
- timeout ID 저장할 때
- 다음 페이지에서 다룰 DOM elements를 저장하거나 조작할 때
- JSX를 계산하는데 필요하지 않은 다른 객체 저장할 때
- **컴포넌트의 일부 정보를 저장하면서 렌더링 로직에는 영향을 미치지않는 경우 ref를 사용**


## Refs and the DOM

- `ref`는 모든 값을 가리킬 수 있고 DOM 요소에도 접근할 수 있다.
- 보통 input focus를 맞출때 유용하게 사용된다.


-----

#### 과제3

```js
import { useRef } from 'react';
// let timeoutID;

function DebouncedButton({ onClick, children }) {
  const timeoutRef = useRef(null); // 고유한 ref을 가지므로 서로 충돌하지 않음

  return (
    <button onClick={() => {
      clearTimeout(timeoutID);
      timeoutID = setTimeout(() => {
        onClick();
      }, 1000);
    }}>
      {children}
    </button>
  );
}

export default function Dashboard() {
  return (
    <>
      <DebouncedButton
        onClick={() => alert('Spaceship launched!')}
      >
        Launch the spaceship
      </DebouncedButton>
      <DebouncedButton
        onClick={() => alert('Soup boiled!')}
      >
        Boil the soup
      </DebouncedButton>
      <DebouncedButton
        onClick={() => alert('Lullaby sung!')}
      >
        Sing a lullaby
      </DebouncedButton>
    </>
  )
}
```


----

#### 과제4


```js
import { useState, useRef } from 'react';

export default function Chat() {
  const [text, setText] = useState('');

  function handleSend() {
    setTimeout(() => {
      alert('Sending: ' + text);
    }, 3000);
  }

  return (
    <>
      <input
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <button
        onClick={handleSend}>
        Send
      </button>
    </>
  );
}
```

```js
import { useState, useRef } from 'react';

export default function Chat() {
  const [text, setText] = useState('');
  const textRef = useRef(text);

  function handleChange(e) {
    setText(e.target.value); // 값을 저장하고
    textRef.current = e.target.value; // 가장 최신의 값을 ref에 저장
  }

  function handleSend() {
    setTimeout(() => {
      alert('Sending: ' + textRef.current); // ref에 저장된 값을 사용
    }, 3000);
  }

  return (
    <>
      <input
        value={text}
        onChange={handleChange}
      />
      <button
        onClick={handleSend}>
        Send
      </button>
    </>
  );
}
```
- `state`변수는 렌더링용, `ref`은 타임아웃 읽기용으로 쓰인다.
- `state`는 스냅샷처럼 작동하므로 비동기 작업에서 최신 `state`를 읽을 수 없다. 이러한 경우 최신 입력 value를 `ref`에 저장하고 `ref.current`속성을 읽어서 비동기 작업 후의 최신 상태값을 읽을 수 있다!!!!