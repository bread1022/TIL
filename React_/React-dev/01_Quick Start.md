# 1. Quick Start

### Creating and nesting components
> 컴포넌트 생성 및 중첩하기

#### 컴포넌트
- 리액트앱은 컴포넌트로 만들어진다.  
- 컴포넌트는 로직을 가진 UI 일부로 마크업을 반환하는 JavaScript 함수이다.
- 컴포넌트명은 항상 대문자로 시작해야되고 HTML은 소문자로 시작해야한다.

### Writing markup with JSX
> JSX로 마크업 작성하기

- JSX는 단일태그도 닫아야한다.
- 컴포넌트는 여러개의 JSX태그를 반환할 수 없기때문에 `<div>...</div>` 또는 `<>...</>` 같이 최상단에 하나의 공유 부모로 감싸야한다.
- key값이 필요하면서 div가 아닌 빈 래퍼가 필요할땐 `<Fragment key={key}></Fragment>` 를 사용하면된다.

### Adding styles
> 스타일 추가하기

- CSS 클래스를 지정하기 위해선 `className`을 사용해야한다
  ```jsx
  <img className="avatar">
  ```
  ```css
  .avatar{
    border: 1px solid #000;
  }
  ```


### Displaying data
> 데이터 표시하기

- JSX에서 중괄호 `{}`를 사용하면 JavaScript 문법을 사용할 수 있다.
  - JavaScript 변수를 삽입하고, 그 값을 읽을 수 있으며
  - 표현식을 넣을 수도 있다.
  - 
    ```jsx
    const user = {
    name: 'Hedy Lamarr',
    imageUrl: 'https://i.imgur.com/yXOvdOSs.jpg',
    imageSize: 90,
    };

    export default function Profile() {
    return (
      <>
        <h1>{user.name}</h1>
        <img
          className="avatar"
          src={user.imageUrl}
          alt={'Photo of ' + user.name}
          style={{
            width: user.imageSize,
            height: user.imageSize
          }}
        />
      </>
    );
    }
    ```


### Conditional rendering
> 조건부 렌더링

- if문과 같이 조건에 따라 분기처리를 위한 간단한 방법
  **1. 조건부 `? 연산자`**
  ```jsx
  <div>
    {isLoggedIn ? (
      <AdminPanel />
    ) : (
      <LoginForm />
    )}
  </div>
  ```
  **2. `논리 && 구문`**
  ```jsx
  <div>
    {isLoggedIn && <AdminPanel />}
  </div>
  ```
- 논리 && 구문에서 주의해야할 점은 앞의 조건 값이 falsy 값일 땐 해당 객체를 반환하기때문에 뒤의 조건에 해당하는 JSX 컴포넌트를 렌더링하지 않지만, 앞의 조건이 값이 0일 땐 0반환되어 렌더링되기때문에 && 좌변에 숫자를 넣으면 안된다.
- 
    ```jsx
    export default function App() {
    return (
      <>
        {user.map(
          (userInfo) =>
            userInfo.id && <Profile user={userInfo} key={userInfo.id} />
        )}
      </>
    );
    }
    ```

### Rendering lists
> 목록 렌더링

- 컴포넌트 목록을 렌더링하기 위해서는 map()메서드를 사용해야한다.
- 이때 key 속성이 필요하고, key 값으론 해당 항목을 고유하게 식별할 수 있는 문자열이나 숫자를 전달해야한다. 나중에 리액트에서 항목을 삽입∙삭제∙재정렬을 할 때 키를 사용하기 때문에 고유한 값이 여야한다.


### Responding to events
> 이벤트에 응답하기

- 컴포넌트 내부에 이벤트 핸들러 함수를 선언하여 이벤트를 응답할 수 있다. 이때 이벤트 핸들러 함수는 즉시실행하지 않고 전달만 해야한다. 리액트는 유저가 이벤트를 실행할 때 해당 이벤트 핸들러를 호출하기 때문이다.
  ```jsx
  onClick={handleClick}
  ```
- 즉시실행함수를 전달하면 실행한 결과가 이벤트에 할당되기 때문이다
  ```jsx
  onClick={handleClick()} // 실행한 결과가 onClick에 전달되는 것임
  ```


### Updating the screen
> 화면 업데이트하기

- useState를 사용하여 state 변수를 사용한다.
- useState를 사용하면 현재 `state` 와 이 상태를 변화시킬 수 있는 함수 `setState` 를 함께 호출할 수 있다.
  ```js
  const [state, setState] = useState()
  ```
- 동일한 컴포넌트여도 여러번 렌더링하면 각각 고유한 state를 가진다.
- 
    ```jsx
    import { useState } from 'react';

    export default function MyApp() {
      return (
        <div>
          <h1>Counters that update separately</h1>
          <MyButton /> // 각각 고유한 상태를 가짐
          <MyButton /> // 다른 버튼에 영향을 주지 않음
        </div>
      );
    }

    function MyButton() {
      const [count, setCount] = useState(0);

      function handleClick() {
        setCount(count + 1);
      }

      return (
        <button onClick={handleClick}>
          Clicked {count} times
        </button>
      );
    }
    ```


### Using Hooks
> 훅 사용하기

- `use`로 시작하는 함수를 `훅(Hook)`이라 한다.
- 훅은 컴포넌트의 **최상위레벨**에서만 호출할 수 있다. (일반함수보다 제한적)
  - useEffect, 등 다른 함수 내부에서 사용할 수 없다.
- 조건문이나 반복문에서 훅을 사용하고 싶다면 새로운 컴포넌트를 생성하고 그 컴포넌트에 작성해야한다.


### Sharing data between components
> 컴포넌트 간 데이터 공유하기

- 데이터를 공유하고 함께 업데이트를 해야하는 경우에는 개별 컴포넌트에서 가장 가까운 컴포넌트로 state를 위쪽으로 이동시켜야한다.
  <img src="https://react-ko.dev/_next/image?url=%2Fimages%2Fdocs%2Fdiagrams%2Fsharing_data_parent.png&w=640&q=75" width=250px> <img src="https://react-ko.dev/_next/image?url=%2Fimages%2Fdocs%2Fdiagrams%2Fsharing_data_parent_clicked.png&w=640&q=75" width=250px>
  - `count` state를 자식에게 모두 전달하여 `MyApp`을 클릭하는 순간 `MyButton`의 `count`값은 동시에 업데이트가 된다.
  - 
    ```jsx
    export default function MyApp() {
      const [count, setCount] = useState(0);

      function handleClick() {
        setCount(count + 1);
      }

      return (
        <div>
          <h1>Counters that update separately</h1>
          <MyButton count={count} onClick={handleClick} />
          <MyButton count={count} onClick={handleClick} />
        </div>
      );
    }

    function MyButton({ count, onClick }) {
      return (
        <button onClick={onClick}>
          Clicked {count} times
        </button>
      );
    }
    ```