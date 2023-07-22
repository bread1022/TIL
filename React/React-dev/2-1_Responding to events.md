# Responding to events


## Adding event handlers
> 이벤트 핸들러 추가하기

이벤트 핸들러를 추가하는 방법


(1) 먼저 함수를 정의한다.  
(2) 적절한 JSX태그에 prop으로 전달한다.


```jsx
export default function Button() {
  function handleClick() { // (1) 이벤트 핸들러 함수를 정의
    alert('You clicked me!');
  }

  return (
    // (2) prop으로 이벤트 핸들러를 전달
    <button onClick={handleClick}>
      Click me
    </button>
  );
}
```

#### event handler
- 일반적으로 컴포넌트 안에 정의한다.
- 이벤트 핸들러 함수 네이밍은 `handle`뒤에 이벤트 이름이 오도록한다.
- 이벤트 핸들러에 전달되는 함수는 호출되는 게 아니라 전달되어야 한다.
  - 호출되는 게 아니라 전달되어야한다.
  - 리액트가 이벤트 핸들러를 기억하고 있다가 사용자 행동에 의해 이벤트가 발생할 때 마다 해당 함수를 호출하도록 지시한다.
  - 만약 전달되지않고 바로 호출되면 렌더링될 때마다 즉시 함수가 실행된다.
  - 이벤트 핸들러를 인라인으로 정의할 땐 익명함수로 감싸야한다.
- 함수가 짧은 경우 JSX 인라인으로 이벤트 핸들러를 정의할 수도 있고 화살표함수를 사용할 수도 있다.
    ```jsx
    <button onClick={function handleClick() {
      alert('You clicked me!');
    }}>
    ```
    ```jsx
    <button onClick={() => {
      alert('You clicked me!');
    }}>
    ```


## Passing event handlers as props
> 이벤트 핸들러를 props로 전달하기

- 이벤트 핸들러는 컴포넌트 내부에서 선언되기 때문에 컴포넌트의 prop에 접근이 가능하다.
- 부모컴포넌트에서 자식컴포넌트의 이벤트핸들러를 지정하고 싶을 땐 prop으로 핸들러를 전달하여 이벤트 발생되면 전달된 함수를 호출하도록 리액트에 지시하는 원리로 동작한다.
    ```jsx
    function Button({ onClick, children }) {
      return (
        <button onClick={onClick}>
          {children}
        </button>
      );
    }
    ```
  ```jsx
  function PlayButton({ movieName }) {
    function handlePlayClick() {
      alert(`Playing ${movieName}!`);
    }
    return (
      <Button onClick={handlePlayClick} />
    );
  }

  function UploadButton() {
    return (
      <Button onClick={() => alert('Uploading!')} />
    );
  }

  export default function Toolbar() {
    return (
      <div>
        <PlayButton movieName="Kiki's Delivery Service" />
        <UploadButton />
      </div>
    );
  }
  ```


## Naming event handler props
> 이벤트 핸들러 props 이름 정하기

- 기본 제공 컴포넌트는 브라우저 이벤트 이름만 지원한다.
- 자체 컴포넌트를 생성할 땐 이벤트 핸들러의 prop의 이름을 원하는 방식으로 지정이 가능하다.
- 이벤트 핸들러의 props는 `on`으로 시작하고 그 뒤에 대문자가 와야한다.


## Event propagation
> 이벤트 전파

- 이벤트 핸들러는 컴포넌트에 있는 모든 하위 컴포넌트의 이벤트도 포착한다.
- `onScroll`을 제외한 모든 이벤트는 리액트에서 전파된다. (`onScroll`은 첨부한 JSX태그에만 작동)


## Stopping propagation
> 전파 중지하기

- 이벤트 핸들러는 `event 객체`를 유일한 인수를 받기때문에 이 객체를 이용하여 이벤트에 대한 정보를 알 수 있다.
- 이벤트가 상위 컴포넌트에 도달하지 못하도록 하려면 하위 컴포넌트에 `event.stopPropagation()`을 호출해야한다.
    ```jsx
    function Button({ onClick, children }) {
      return (
        <button onClick={e => {
          e.stopPropagation();  // 상위 컴포넌트에 전파 중지
          onClick();
        }}>
          {children}
        </button>
      );
    }

    export default function Toolbar() {
      return (
        <div className="Toolbar" onClick={() => {
          alert('You clicked on the toolbar!');
        }}>
          <Button onClick={() => alert('Playing!')}>
            Play Movie
          </Button>
          <Button onClick={() => alert('Uploading!')}>
            Upload Image
          </Button>
        </div>
      );
    }
    ```
  - 버튼을 클릭하면 이벤트가 버블링되지 않도록 `e.stopPropagation()`을 호출한다.
  - `Toolbar`컴포넌트에서 전달된 props인 `onClick`함수를 호출한다.
  - `Toolbar`컴포넌트에 정의된 함수는 버튼 자체의 경고를 표시하고 전파가 중지되었으므로 `div`의 `onClick`핸들러는 실행되지 않는다.


### event Capture

- 이벤트 전파 중지 외에 하위 요소의 모든 이벤트를 포착해야하는 경우도 있다.
- 전파로직에 관계없이 모든 클릭을 분석도구에 기록하고자할 땐 이벤트 이름 끝에 `Capture`를 추가하여 수행한다.
    ```jsx
    <div onClickCapture={() => { /* this runs first | 먼저 실행됨 */ }}>
      <button onClick={e => e.stopPropagation()} />
      <button onClick={e => e.stopPropagation()} />
    </div>
    ```
  - **이벤트 전파 3단계**
  1. 아래로 이동하면서 모든 `onClickCapture`핸들러를 호출한다.
  2. 클릭한 요소의 `onClick`핸들러를 실행한다.
  3. 상위로 이동하면서 모든 `onClick`핸들러를 호출한다.



## Passing handlers as alternative to propagation
> 전파의 대안으로 핸들러 전달하기

- 부모에서 prop으로 전달받은 이벤트 핸들러를 호출하기 전에 코드를 더 추가하여 전파를 중지할 수 있다.
- 자식컴포넌트가 이벤트를 처리하는 동시에 부모 컴포넌트가 추가 동작을 지정할 수 있게 한다.
    ```jsx
    function Button({ onClick, children }) {
      return (
        <button onClick={e => {
          e.stopPropagation();
          // 코드를 더 추가할 수 있다.
          onClick();
        }}>
          {children}
        </button>
      );
    }
    ```


## Preventing default behavior
> 기본 동작 방지

- 일부 브라우저 이벤트에 연결된 기본 동작을 방지하기 위해서는 `event.preventDefault()`를 호출한다.
- 예를들어, `<form>` submit 이벤트는 내부 버튼 클릭시 페이지 reload가 발생한다.  
이러한 불필요한 새로고침을 막기위해 `e.preventDefault()` 를 사용한다.


## Can event handlers have side effects?
> 이벤트 핸들러에 부작용이 생길 수 있나요?

- 이벤트 핸들러는 렌더링함수와 달리 순수할 필요가 없으므로 응답으로 데이터를 변경하기 좋은 곳이다. 일부 데이터를 변경하여 저장하기 위해선 리액트에서 컴포넌트의 메모리인 `state`를 사용해 데이터 변경 저장 작업을 수행한다.