# Lifecycle of Reactive Effects


## The lifecycle of an Effect
> Effect의 생명주기

#### 컴포넌트의 생명주기
- 화면에 추가될 때 **mounts**
- 새로운 props나 state를 받으면 **updates**
- 화면에서 제거될 때 **unmounts**

React에서 컴포넌트가 마운트될 때 동기화가 시작되고 마운트해제될 때 동기화가 중지될 것이라고 생각할 수 있지만  
때때로 컴포넌트가 마운트된 상태에 동기화가 여러번 발생할 수 있다.


## Why synchronization may need to happen more than once


```js
function ChatRoom({ roomId /* "general" */ }) {
  // ...
  useEffect(() => {  // 2️⃣ Effect를 실행하여 동기화 시작
    const connection = createConnection(serverUrl, roomId);
    // 3️⃣ 클린업 함수가 호출되기 전까지 동기화를 시작 (연결이 끊어질 때까지)
    connection.connect();
    return () => {
      connection.disconnect();  // 4️⃣ (다른 컴포넌트에서) 클린업 함수가 호출되면 동기화 중지
    };
  }, [roomId]);
  return <h1>Welcome to the {roomId} room!</h1>; // 1️⃣ UI를 렌더링
}
```
- prop이 바뀔경우
  - 이전 Effect의 동기화를 중지하고 새로운 Effect를 시작한다. (***이전 동기화 중지 후 재동기화 시작***)
  - 컴포넌트가 다른 prop으로 리렌더링될 때마다 클린업함수를 호출하여 Effect 동기화를 중지하고, 새로운 Prop으로 재동기화한다.
  - 컴포넌트가 마운트 해제되면 React는 마지막으로 Effect의 동기화를 중지한다.



## Thinking from the Effect’s perspective
> Effect의 관점에서 생각하기

**컴포넌트 관점**에서는 첫 prop으로 컴포넌트가 마운트되고, (Effect 동기화 시작)  
두번째 prop으로 컴포넌트가 업데이트되고, (첫번째 상태의 Effect클린업 함수를 호출하여 동기화를 중지한 뒤, 두번째 상태의 Effect로 재동기화)  
세번째 prop으로 컴포넌트가 업데이트되고, (이전의 Effect를 중지하고 새로운 Effect로 재동기화)  
다른 페이지로 이동하면 컴포넌트의 마운트가 해제되는 것이다. (마지막 Effect를 중지)



**Effect 관점**에서는 첫번째 prop으로 동기화를 시작하여 중지될 때까지 동기화를 유지하고,  
두번째 prop으로 동기화를 시작하여 중지될 때까지 동기화를 유지하고,
세번째 prop으로 동기화를 시작하여 중지될 때까지 동기화를 유지하는 것이다.



## How React verifies that your Effect can re-synchronize
> React가 Effect의 재동기화 가능 여부를 확인하는 방법


- 개발모드에서 React는 항상 컴포넌트를 마운트하고 마운트해제했다가 다시 마운트하고  
  **즉시 동기화를 수행하여 Effect가 다시 동기화될 수 있는지 확인**한다.
  React는 개발중에 Effect를 한번 더 시작하고 중지하여 클린업함수를 잘 구현하였는지 확인한다.

<!-- 다시 -->


## How React knows that it needs to re-synchronize the Effect
> React가 Effect의 재동기화 필요성을 인식하는 방법

- 이때 재동기화가 필요한지 여부를 결정하는 것은 Effect의 의존성이다.
- 시간에 따라 변경될 수 있는 변수를 의존성으로 지정하면,  
  React는 Effect가 재동기화되어야 하는지 여부를 결정할 수 있다. (변경될 때마다 동기화되게끔함)
- 초기렌더링에 전달된 의존성이 이후 렌더링 때 바뀌면 React는 `Object.is`로 비교했을 때 다른 값이라고 판단하기 때문에 React는 Effect를 재동기화한다.
- 의존성에 지정할 필요가 없는 변수 : **해당 변수에 의해 리렌더링이 발생하지 않고, 항상 동일할 경우**! 의존성 지정을 하지 않는다!!!
- 의존성에 지정해야하는 변수 : 컴포넌트 내부에서 선언된 Props, state 등 렌더링 중에 계산되고 React **데이터 흐름에 참여(반응형)하는 변수**는 렌더링 때 달라질 수 있기때문에 의존성에 지정해야한다.


## Each Effect represents a separate synchronization process
> 각 Effect는 별도의 독립적인 동기화 프로세스를 나타낸다.

```js
// ❌
function ChatRoom({ roomId }) {
  useEffect(() => {
    logVisit(roomId); // 1️⃣ 룸아이디 저장
    const connection = createConnection(serverUrl, roomId); // 2️⃣ 채팅 연결
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId]);
  // ...
}
```
- Effect 내부 코드는 동시에 실행되기때문에 관련 없는 로직은 분리하는 것이 좋다.
  ```js
  // ✅ 각 Effect는 별도의 동기화 과정이어야함!!
  function ChatRoom({ roomId }) {
    useEffect(() => {
      logVisit(roomId);
    }, [roomId]);

    useEffect(() => {
      const connection = createConnection(serverUrl, roomId);
      // ...
    }, [roomId]);
    // ...
  }
  ```
  - 서로 다른 것을 동기화하므로 분리하는 것이 합리적이다.  
  (코드의 가독성보다 프로세스가 동일한지, 분리되어 있는지가 중요하다!)


## What an Effect with empty dependencies means
> 빈 의존성을 가지고 있는 Effect의 의미

- 컴포넌트 관점에서, 빈 의존성 배열은 컴포넌트가 마운트될 때만 Effect를 실행하고,  
  컴포넌트가 마운트 해제될 때만 클린업 함수를 호출한다.
- Effect 관점에서, 동기화를 시작하고 중지하는 작업에 대한 기준을 의존성에 추가하는 것이다.


## All variables declared in the component body are reactive
> 컴포넌트 본문에서 선언된 모든 변수는 반응형입니다


- 반응형 값은 Props와 state, 그리고 컴포넌트가 렌더링되고 그로부터 계산된 값들도 반응형 값이기때문에 컴포넌트 내부의 모든 값은 반응형이다. 모든 반응형 값은 리렌더링할 때 변경될 수 있으므로 반응형 값을 Effect 의존성으로 포함시켜야한다.
- 하지만 변이가능한 전역변수는 반응형이 아니기때문에 의존성이 될 수 없다.
- 전역변수는 React 렌더링 데이터 흐름 외부에서 언제든지 바뀔 수 있고 이 값을 변경해도 컴포넌트가 리렌더링되지 않는다. 따라서 의존성에 지정하더라도 React는 이 값이 변경될 때 Effect를 다시 동기화해야하는지 알 수 없고 렌더링 중 변이가능한 값을 읽는 것은 순수하지 않기 때문에 의존성에 지정하지 않아야한다.
- 또한 객체와 함수도 의존성으로 사용하지 않는다. 왜냐하면 렌더링 중 객체와 함수를 생성한 후 Effect에서 읽으면 렌더링할 때마다 객체와 함수가 달라지고 매번 Effect를 재동기화해야하기때문이다.
- (이때 모든 반응형 값을 의존성으로 지정했는지 검토하기 위한 방법으로 [eslint](https://www.npmjs.com/package/eslint-config-react-app)를 설치하여 의존성 요소 지정을 체크할 수 있다.)



## What to do when you don’t want to re-synchronize
> 재동기화를 원치 않는 경우 어떻게 해야할까

- 어떠한 값이 반응형이 아닌, 렌더링 결과로 변경될 수 없는 항상 같은 값이라면  
  컴포넌트 밖으로 변수 위치를 이동시켜 의존성에 지정하지 않아도된다. (린터 버그가 일어나지 않음~!)
- Effect는 반응형 코드블록이다. Effect 내부의 값이 변경되면 Effect는 재동기화된다.
- 이벤트핸들러처럼 상호작용에 의해 한번만 실행되지 않고 Effect는 동기화가 필요할 때마다 실행되는 것이다.  
  따라서 Effect는 독립적으로 동기화되어야하며 여러개의 Effect를 사용하여 동기화를 분할하는 것이 좋다.
