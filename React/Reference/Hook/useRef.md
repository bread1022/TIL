# useRef
> 렌더링에 필요하지 않은 값을 참조할 수 있는 React Hook

```js
const ref = useRef(initialValue)
```

```js
import { useRef } from 'react'

function MyComponent() {
  const inputRef = useRef(null)
  return <input ref={inputRef} />
}
```

## Parameters

- initialValue: ref객체 current 프로퍼티의 초기값
  - 어떤 타입의 값이든 지정가능하다
  - 초기 렌더링에만 필요한 값이다

## Returns

- `ref` 객체를 반환한다.
  - `current` : 초기값은 initialValue로 지정되며 다른값으로 변경할 수 있다.
    - ref객체를 JSX 노드의 ref속성으로 전달하면 React에서는 `current` 프로퍼티를 해당 노드로 설정한다.

## 주의사항

- `ref.current` 프로퍼티는 변이가 가능하지만 렌더링에 사용되는 객체를 포함하는 경우 `ref`를 변이하면 안된다.
- `ref.current`는 JS 객체이기때문에 변이 되어도 React에서 변이를 감지하지 못해 리렌더링을 트리거하지 않는다.
- 초기화할 때를 제외하곤 렌더링 중에 `ref.current`를 읽거나 쓰지 않는 것이 좋다. (컴포넌트 동작을 예측할 수 없기 때문이다)
- Strict Mode에서 컴포넌트가 두번 호출되기때문에 `ref`객체도 두번 생성되고 하나는 버려지는데 컴포넌트가 순수하다면 로직에 영향을 주지 않는다.

<br>

## ref로 값 참조

```js
import { useRef } from 'react'

function Stopwatch() {
  const intervalRef = useRef(0);

  const handleStart = () => {
    // 🌟 내부의 값을 업데이트할땐 current 프로퍼티를 수동으로 변경해야됨
    intervalRef.current = setInterval(() => {
      // ...
    }, 1000);
  }

  const handleStop = () => {
    const intervalId = intervalRef.current;
    clearInterval(intervalId);
  }
}
```

- `useRef`는 초기값으로 설정된 단일 current 프로퍼티를 가진 ref객체를 반환한다.
- ref는 컴포넌트의 시각적 출력에 영향을 미치지 않는 정보를 저장하는데 적합하다.
  - 변이되어도 리렌더링을 트리거하지 않으므로 화면에 표시되는 정보를 저장하는데는 적합하지 않다.
- ref가 일반변수와 다른점은 리렌더링사이에 정보를 저장할 수 있다. (일반변수는 렌더링될 때마다 재설정됨)
- state변수와 다른점은 변경되어도 리렌더링을 트리거하지 않는다.
- 외부변수와 다른점은 컴포넌트내부에 로컬로 저장되는 정보이다. (정보가 공유되지 않음)


#### ref를 사용해도 되는 경우

```js
function Counter() {
  let ref = useRef(0);

  const handleClick = () => {
    // 🌟 이벤트핸들러에서만 ref를 읽고 사용하는 건 괜찮다!
    ref.current = ref.current + 1;
  }

  return <button onClick={handleClick}>버튼클릭</button>
}
```


#### 렌더링 중 ref를 사용해도 되는 경우

```js
function MyComponent() {
  const myRef = useRef(0);

  const handleClick = () => { // ✅ 이벤트핸들러에서는 읽어도 됨
    doSomthing(myRef.current);
  };

  useEffect(() => { // ✅ useEffect에서는 읽어도 됨
    myRef.current = 123;
  });
}
```
```js
function MyComponent() {
  const myRef = useRef(0);

  myRef.current = 123; // ❌ 렌더링중에 ref를 작성하면 안됨

  return <button onClick={handleClick}>{myRef.current}</button>
}
```

## ref로 DOM 조작

```js
import { useRef } from 'react';

const MyComponent = () => {
  const inputRef = useRef(null); // 🌟 DOM 조작시 보통 초기값으로 null을 지정

  // JSX에 속성으로 ref 객체를 전달
  return <input ref={inputRef} />;
}
```
- ref객체를 JSX 노드의 ref속성으로 전달하면 React는 ref객체의 current 프로퍼티를 해당 노드로 설정한다.
  - DOM 노드 <input> 접근해 focus()와 같은 메서드를 호출할 수 있다.
  - 노드가 화면에서 제거되면 current 프로퍼티는 다시 null로 설정된다.

## ref 재생성 방지하는 방법

```js
function Video() {
  // 생성자함수에 생성된 결과는 초기 렌더링에 사용되지만,
  // 호출 자체는 이후 모든 렌더링에 계속 이루어짐
  const playerRef = useRef(new VideoPlayer()); // ❌ 생성자함수호출은 객체를 계속해서 생성하게됨
}
```

```js
function Video() {
  const playerRef = useRef(null);
  if (playerRef.current === null) {
    // 1️⃣ 초기에만 생성할 수 있게끔 조건을 걸어야함
    // 결과가 항상 동일하고 초기화에만 실행되는 조건이므로 렌더링중 ref.current를 읽어도됨
    playerRef.current = new VideoPlayer();
  }
}
```
```js
function Video() {
  const playerRef = useRef(null);

  // 2️⃣ 이벤트핸들러에서 getPlayer() 함수를 호출하여 playerRef.current를 읽는 방법도 있음
  function getPlayer() {
    if (playerRef.current !== null) {
      return playerRef.current;
    }
    const player = new VideoPlayer();
    playerRef.current = player;
    return player;
  }
  // ...
}
```

## 커스텀 컴포넌트에 대한 ref를 얻는 방법 : `forwardRef`

```js
import { useRef } from 'react';

function MyComponent() {
  const inputRef = useRef(null);

  return <MyInput ref={inputRef} />;
}
```

```js
// ❌ 커스텀 컴포넌트에 대한 ref 접근할 수 없음
export default function MyInput({ value, onChange }) {
  return (
    <input
      value={value}
      onChange={onChange}
    />
  );
}
```
- 컴포넌트 내부 DOM 노드에 대한 ref를 얻을 수 없기때문에 커스텀 컴포넌트에 ref를 얻고싶다면 `forwardRef`를 사용해야한다.


```js
import { forwardRef } from 'react';

// ✅ forwardRef를 사용해야 부모컴포넌트가 ref를 가져올 수 있음
export const MyInput = forwardRef(({ value, onChange }, ref) => {
  return (
    <input
      value={value}
      onChange={onChange}
      ref={ref}
    />
  );
});
```