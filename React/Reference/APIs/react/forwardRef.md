# forwardRef
> 부모컴포넌트에 컴포넌트 내부 ref로 DOM 노드를 전달할 수 있게하는 API

```js
const SomeComponent = forwardRef(render)
```

## Parameters

- `render`: 컴포넌트 함수로 React는 컴포넌트가 부모로부터 받은 props와 ref를 가지고 이 함수를 호출한다.

## Returns

- 반환값은 React 컴포넌트로 일반 컴포넌트 함수와 달리 prop으로 ref를 전달 받을 수 있다.

## 주의사항

- ref는 과도하게 사용하면 안된다.
- prop으로 표현할 수 없는 필수적인 동작에만 ref를 사용해야 한다.
- 예시로 Node로 스크롤하기, 초점맞추기, 애니메이션 트리거하기, 텍스트 선택하기 등에 적합하다.

## render function

- 매개변수로 props와 ref를 받는다.
  - render 함수의 매개변수로 기존 props와 ref를 각각 받는다.
  ```js
  const MyInput = forwardRef(function MyInput(props, ref) {
    return (
      //...
    );
  });
  ```
  - `props`: 부모 컴포넌트에서 전달된 props
  - `ref`: 부모 컴포넌트에서 전달한 ref 속성 (객체 또는 함수), 전달하지 않은 경우 null이다.
- 반환값으로 React 컴포넌트를 반환한다.
  - 일반 함수로 정의된 컴포넌트와 달리 ref를 전달받을 수 있다.


## 부모 컴포넌트에 자식 컴포넌트의 DOM 노드 노출하는 방법

```js
import { forwardRef } from 'react';

const FancyButton = forwardRef((props, ref) => (
  <button ref={ref} className="FancyButton">
    {props.children}
  </button>
));
```
- 기본적으로 컴포넌트 내부 DOM 노드는 비공개이지만, `forwardRef()`로 컴포넌트를 정의하면 ref를 전달받고 노출하려는 DOM 노드에 전달하면 부모컴포넌트에서 접근이 가능하다.
- 컴포넌트 내부의 DOM 노드에 대한 ref를 노출하게 되면 나중에 컴포넌트 내부를 변경하기 어려워지므로 저수준의 컴포넌트에서만 사용하는 것이 좋다.


```js
import { useRef } from 'react';
import MyVideoPlayer from './MyVideoPlayer.js';

export default function App() {
  const ref = useRef(null);
  return (
    <>
      <button onClick={() => ref.current.play()}>Play</button>
      <button onClick={() => ref.current.pause()}>Pause</button>
      <br />
      <MyVideoPlayer
        ref={ref}
        src="https://interactive-examples.mdn.mozilla.net/media/cc0-videos/flower.mp4"
        type="video/mp4"
        width="250"
      />
    </>
  );
}
```

```js
import { forwardRef } from 'react';

export const VideoPlayer = forwardRef(function VideoPlayer({ src, type, width }, ref) {
  return (
    // 🌟 video DOM 노드에 ref를 전달하여 부모컴포넌트에서 접근할 수 있게 함!
    <video width={width} ref={ref}>
      <source src={src} type={type} />
    </video>
  );
});
```

## 여러 컴포넌트를 통해 ref를 전달하는 방법

- `forwardRef`를 사용하면 부모 컴포넌트에서 전달받은 ref를 자식 컴포넌트에서 다른 컴포넌트로 전달할 수 있고 DOM 노드를 노출할 수 있게 된다.

```js
function Form() {
  const ref = useRef(null);

  function handleClick() {
    ref.current.focus();
  }

  return (
    <form>
      <FormField label="Enter your name:" ref={ref} isRequired={true} />
      <button type="button" onClick={handleClick}>
        Edit
      </button>
    </form>
  );
}
```
```js
const FormField = forwardRef(function FormField(props, ref) {
  // ...
  return (
    <>
      <MyInput ref={ref} />
      ...
    </>
  );
});

const MyInput = forwardRef((props, ref) => {
  const { label, ...otherProps } = props;
  return (
    <label>
      {label}
      <input {...otherProps} ref={ref} />
    </label>
  );
});
```


## DOM 노드 대신 명령형 핸들 노출하는 방법 : `useImperativeHandle`

```js
import { forwardRef, useRef, useImperativeHandle } from 'react';

const MyInput = forwardRef(function MyInput(props, ref) {
  const inputRef = useRef(null);

  // ✅ 부모 컴포넌트에 노출하는 정보를 제한할 수 있음
  useImperativeHandle(ref, () => {
    return {
      focus() {
        inputRef.current.focus();
      },
      scrollIntoView() {
        inputRef.current.scrollIntoView();
      },
    };
  }, []);

  return <input {...props} ref={inputRef} />;
});
```
- `useImperativeHandle`에 ref를 전달하고 노출시킬 값을 지정할 수 있다.
- 노출하는 정보를 명령형 핸들로 제한할 수 있으므로 부모 컴포넌트에서 노출된 정보를 사용할 때 불필요한 정보를 사용하지 않도록 할 수 있다.
- 대신 명령형 핸들보다 prop으로 전달했을 때 동작을 더 잘 이해할 수도 있는 경우에는 ref를 사용하지 않는 것이 좋다.
```js
<Modal isOpen={isOpen} /> // ⭕️
```
```js
const handleOpen = () => {
  ref.current.open(); // 🔺 이 방법보단 prop으로 명령형 동작을 노출하는 것이 낫다!
};

<Modal ref={ref}>
```


## forwardRef로 감싸져 있는데 null을 반환할 때?

```js
const MyInput = forwardRef(function MyInput({ label }, ref) {
  return (
    <label>
      {label}
      // ❌ 매개변수로 전달받은 ref를 DOM 노드에 전달하지 않음
      <input />
    </label>
  );
});
```

```js
const MyInput = forwardRef(function MyInput({ label, showInput }, ref) {
  return (
    <label>
      {label}
      <input ref={ref} />
      {showInput && <input ref={ref} />} // 🌟 조건부로 전달한 경우에는 ref가 null이 될 수 있으므로 주의해야함!
    </label>
  );
});
```