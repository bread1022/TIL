# Synchronizing with Effects

`useEffect`는 렌더링 이후 코드를 실행할 수 있게 하는 hook으로 컴포넌트를 React 외부 시스템과 동기화할 때 유용하다.


## What are Effects and how are they different from events?

#### Rendering Code
> 컴포넌트의 최상위 레벨에 있는 렌더링 코드는 props와 state를 업데이트하고 JSX를 반환한다. (**순수함수**)

#### Event Handlers
> 컴포넌트 내부에 중첩된 함수로
> 사용자의 액션에 의해 발생하는 이벤트 작업을 수행한다.
> ex. input field 업데이트, HTTP POST 요청 등
> 사용자의 작업에 의해 발생하는 부수효과(side effect)가 포함되어 있다.
> 특정 상호 작용에 의해 발생하면 이벤트!~!!!!!

#### Effect란?
- Side Effect는 특정 이벤트에 의한 것이 아니라 **렌더링으로 인해 발생하는 부수효과를 표시**하는 것이다. (렌더링 자체에 의해 발생)
- **Side Effect는 화면 업데이트 후 커밋이 끝날 때! 실행**된다. (렌더링 중에 발생하는 것이 아니라 렌더링 직후 발생, 초기 렌더링 포함)
- 일반적으로 Side Effect는 외부 시스템(브라우저 API, 서드파티 위젯, 네트워크)과 동기화 하는데에 사용된다.
- 따라서 *다른 state기반으로 state를 조정하는 경우에는 Side Effect가 필요하지 않으므로 이러한 경우에는 사용을 지양해야한다.*


## How to write an Effect
> Effect 작성 방법

1. **Declare an Effect**
   - useEffect를 선언한다.
   - 모든 렌더링 이후 실행될 것.
    ```jsx
    import { useEffect } from 'react';

    function MyComponent() {
      useEffect(() => {
        // 여기의 코드는 매 렌더링 후에 실행됩니다.
      });
      return <div />;
    }
    ```
    - 컴포넌트는 렌더링될 때마다 React화면을 업데이트하고 `useEffect`내부 코드를 실행한다.
    - `useEffect`는 렌더링이 화면에 표시될 때까지 코드 실행을 지연한다.
    - 보통 DOM node를 조작해야하는 코드들을 `useEffect`콜백 내부에 작성한다.
      ```js
      import { useEffect, useRef } from 'react';

      function VideoPlayer({ src, isPlaying }) {
        const ref = useRef(null);

        useEffect(() => {
          if (isPlaying) {
            ref.current.play(); // DOM node 조작은 렌더링 이후에 실행되어야함!!
          } else {
            ref.current.pause();
          }
        }); // 의존성 배열 추가해야됨

        return <video ref={ref} src={src} loop playsInline />;
      }
      ```
2. **Specify the Effect dependencies**
   - 의존성을 명시한다.
   - Effect는 모든 렌더링이 아니라 **필요할 때만 재실행되어야 하므로 의존성을 지정**하여 제어해야한다.
   - 의존성 배열을 지정해주면 이전 렌더링과 같을 때 Effect를 실행하지 않는다.
   - 의존성 배열에 여러개의 요소를 지정하게되면 **모든 의존성이 마지막 렌더링 시점과 정확히 동일한 경우**에만 Effect를 실행하지 않는다. (건너뜀)
     - `Object.is(a, b)` 메서드로 비교
   - 의존성 배열에는 `useEffect` 내부에서 사용되는 모든 변수를 지정해야한다. (이때 Ref같이 렌더링과 상관없는, 변경되지않는 변수는 제외함)
      ```jsx
      import { useState, useRef, useEffect } from 'react';

      function VideoPlayer({ src, isPlaying }) {
        const ref = useRef(null);

        useEffect(() => {
          if (isPlaying) {
            console.log('Calling video.play()');
            ref.current.play();
          } else {
            console.log('Calling video.pause()');
            ref.current.pause();
          }
        }, [isPlaying]);
        // isPlaying이 변경될 때만 Effect가 실행됨
        return <video ref={ref} src={src} loop playsInline />;
      }
      ```
      - `useEffect`내부에서 사용되는 변수가 `isPlaying`말고 `ref`도 있지만 `ref`를 의존성배열에 추가하지않는 이유 : `ref` 객체가 렌더링할때마다 `useRef`에서 동일한 객체를 반환하기때문에 의존성 배열에 추가할 필요가 없다.  
      (변하지 않으니 `ref`로 인해서 Effect가 다시 실행되지 않기때문!)
    - 렌더링 될 때마다 실행
       ```jsx
       useEffect(() => {
         ... // 렌더링때마다 실행
       });
       ```
     - **빈 의존성 배열** : 컴포넌트가 마운트될 때만 실행 (화면에 추가되는 시점)
       ```jsx
       useEffect(() => {
         ... // 컴포넌트가 마운트될 때만 실행 (나타날 때만)
       }, []); // 내부코드가 props나 state를 사용하지 않으면 빈 배열로 지정
       ```
     - 의존성이 변경될 때 실행
       ```jsx
       useEffect(() => {
         ... // 컴포넌트 마운트될 때 + a 또는 b 또는 c가 직전 렌더링때와 달라지면 실행
       }, [a, b, c]);
       ```
3. **Add cleanup if needed**
   - 필요한 경우 클린업함수를 이용하여 Effect가 수행 중이던 작업을 중지하거나 취소하는 기능을 추가해야한다.
      ```js
      export default function ChatRoom() {
        useEffect(() => {
          const connection = createConnection();
          connection.connect();
          // 클린업함수를 반환하여
          // Effect가 다시 실행되기 전에 수행되어야할 작업을 지정함
          // 컴포넌트가 마운트 해제(제거)될 때 마지막으로 한번 더 호출하게 됨
          return () => {
            connection.disconnect();
          };
        }, []);
        // (의존성배열 []) 처음 화면 렌더링될 때만(마운트될 때) 코드를 실행하도록함
        return <h1>Welcome to the chat!</h1>;
      }
      ```
   - React는 Effect가 다시 실행되기 전에 매번 클린업 함수를 호출하고  
    컴포넌트가 제거될 때(마운트해제) 마지막으로 한번 더 클린업함수를 호출한다.
   - 개발환경(strict mode)에서 컴포넌트를 다시 마운트하는 건 코드가 깨지지 않는지 확인하기 위함이기때문에 클린업함수를 잘 구현해놓으면 사용자가 체감할 수 있는 차이가 없을 수 있다.
   - strict mode에서 두번 실행되어도 Effect가 문제없이 동작할 수 있는 방법은 클린업 함수를 잘 구현하는 것이고 설정-> 정리 -> 설정 하는 시퀀스를 사용자가 느낄 수 없게 해야한다.



#### Effect 무한 루프를 생성하는 경우

```jsx
const [count, setCount] = useState(0);
useEffect(() => {
  setCount(count + 1); // Effect내부에서 state를 업데이트하면 무한루프가 발생함
});
```
- Effect는 렌더링 이후에 실행되는 것이다. 렌더링 결과에 의해 Effect 내부 코드가 실행되므로  
  위의 코드에서 `useEffect`내부에 `state`를 업데이트하게 되면 또 렌더링을 트리고하고,   
  그럼 Effect가 실행되고 `state`를 또 업데이트하고 또 렌더링이 발생하면서 무한루프가 발생한다.
- Effect는 보통 컴포넌트 외부 시스템과 동기화할 때 사용하는 것이고  
  이렇게 `state`기반으로 일부 `state`만 조정하려는 경우엔 Effect가 필요하지않을 수 있다.



## Subscribing to events
> 이벤트 구독하기


- Effect 내부 코드에서 이벤트를 구독한 경우, 구독을 취소하는 클린업함수를 반환해야한다. (메모리 누수 방지)
- React 개발환경에서 코드에 버그가 있는지 검사하기 위해  
  모든 컴포넌트를 최초 마운트 직후에 한번씩 다시 마운트를 한다.
  (컴포넌트가 언마운트되고 다시 마운트되는 것이 아니라 컴포넌트가 마운트된 상태에서 다시 마운트된다.)
- Effect가 다시 실행되기전 클린업 함수를 호출하고 컴포넌트가 마운트 해제(제거)될 때 마지막으로 한번 더 클린업 함수를 호출하는 것이다.



## Triggering animations
> 애니메이션 촉발하기


- Effect가 애니메이션을 촉발하는 경우 클린업 함수는 애니메이션을 초기값으로 재설정해야한다.
  ```js
  useEffect(() => {
    const node = ref.current;
    node.style.opacity = 1; // 애니메이션을 적용하였으면

    // 초기화하는 클린업함수도 꼭 추가해야한다. 초기값으로 재설정!!!
    return () => {
      node.style.opacity = 0;
    }
  }, []);
  ```
  - 개발환경에서 불투명도가 1이었다가 0으로 바뀐다음, 다시 1로 설정된다. (상용환경에서 1을 한번만 설정하는 것과 같음)


## Fetching data
> 데이터 페칭하기


```jsx
useEffect(() => {
  let ignore = false;

  async function startFetching() {
    const json = await fetchTodos(userId); // 네트워크 요청을 보내고
    if (!ignore) setTodos(json);
  }
  startFetching();
  return () => {
    // 클린업함수로 패치를 중단하거나 결과를 무시하게끔함
    ignore = true;
  };
}, [userId]);
```

- 네트워크 중복 요청을 방지하기 위해 컴포넌트 간 응답을 캐시하는 해결책으로 `useSomeDataLibrary`를 사용할 수도있다.


#### `useSomeDataLibrary`

<!-- TODO: useSomeDataLibrary -->


#### What are good alternatives to data fetching in Effects?
> Effect에서의 데이터 패칭 대안은 무엇인가?

- 초기 서버 렌더링되는 HTML에는 data 없이 로딩 state만 포함되며  
  클라이언트 컴퓨터에서 모든 JS를 다운로드하고 앱을 렌더링하고 난 후 데이터를 로드한다.
- Effect에서 data fetch를 하게되면 상위컴포넌트에서 렌더링 후 fetch하고, 하위 컴포넌트를 렌더링한 다음 다시 data fetch가 진행된다. 이렇게 되면 모든 데이터를 병렬로 패치하는 것보다 훨씬 느리게 된다.
- Effect에서 직접 패치할 경우 data를 미리 로드하거나 캐시하지 않기때문에  
  컴포넌트를 마운팅하고, 마운트 해제되었다가 다시 마운트되면 또 다시 fetch하게 된다.
- 이를 대체하기 위한 오픈소스로 `React Query`, `useSWR` `React Router 6.4+` 등이 있다.



## Sending analytics
> 분석 보내기

#### `intersection observers`

<!-- TODO: intersection observers -->


## Not an Effect: Initializing the application
> Effect가 아님: 애플리케이션 초기화하기

- App컴포넌트가 실행될 때 한번만 실행되어야하는 로직은 컴포넌트 외부에 작성할 수 있다.
- 외부에 작성한 코드는 브라우저가 페이지를 로드한 후 한 번만 실행된다!!!
  ```js
  // 브라우저가 페이지 로드한 후 1번만 실행
  if (typeof window !== 'undefined') {
  // Server side rendering (next.js 서버환경은 window 객체없음)때문에 실행환경이 브라우저인지 확인
    checkAuthToken();
    loadDataFromLocalStorage();
  }

  function App() {
    ...
  }
  ```


## Putting it all together





### Each render has its own Effects




https://react-ko.dev/learn/synchronizing-with-effects