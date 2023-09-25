# useEffect
> 컴포넌트를 외부 시스템과 동기화할 수 있는 React Hook

```js
useEffect(setup, dependencies);
```

## Parameter

- setup function: effect 로직이 포함된 함수로 외부 시스템과 동기화하는 로직을 포함한다.
  - React 컴포넌트가 DOM에 추가되면 setup 함수가 실행되고,
  - 의존성이 변경되어 리렌더링할 때마다 클린업 함수가 있으면 이전 값으로 클린업 함수를 실행하고 새로운 값으로 setup 함수를 실행한다.
  - 컴포넌트가 DOM에서 제거되면, React는 클린업 함수를 실행한다.
- dependencies?: setup 코드 내에 참조된 모든 반응형 값들의 목록
  - 의존성을 지정하지 않으면 리렌더링할 때마다 Effect가 실행된다.
  - 의존성을 전달하지 않을 때: 모든 렌더링, 리렌더링마다 Effect를 실행한다.
  - 빈배열을 전달: 초기 렌더링 후, Effect를 한 번만 실행한다. (반응형 값을 사용하지 않는다면 1번만 실행)
  - 의존성 배열을 전달: 초기 렌더링 후, 변경된 의존성으로 리렌더링한 다음 Effect를 실행한다.


## Returns

- `undefined`를 반환한다.

## 주의사항

- 최상위 레벨, 또는 커스텀 Hook 내에서만 호출해야 한다.
- server 렌더링 중에는 실행되지 않고 client에서만 실행되는 로직이다.
- 외부 시스템과 동기화하려는 목적외에는 지양해야한다.
- 의존성배열에 객체 또는 함수가 포함된 경우 Effect가 필요이상으로 재실행될 수 있으므로 불필요한 객체, 함수는 의존성에서 제거하는 것이 좋다.
- Effect가 상호작용에 의한 것이 아니라면 React는 브라우저에서 화면을 먼저 그린다음 Effect를 실행한다.  
  Effect가 시각적인 작업을 수행하거나, 작업 지연이 눈에 띄눈 경우 `useLayoutEffect`를 사용하는 것이 좋다.
- Effect가 상호작용에 의해 발생했더라도 브라우저는 화면을 리렌더링하고 Effect 내부의 state를 업데이트한다. 이때 리페인팅을 막아야할땐 `useLayoutEffect`를 사용하는 것이 좋다.
- Strict mode 에서 React는 실제 setup 함수 전에 setup, clean up 함수를 한번 더 실행한다. clean up 함수는 setup 함수가 수행중인 작업을 중지하거나 취소하는 로직이여야한다. 그렇게 하여 사용자 경험에서의 setup 함수 1번 호출한 것과 같이 개발환경에서 setup, clean up, setup 순서로 호출되는 것을 구분할 수 없게해야한다.
- 모든 Effect는 독립적인 프로세스로 작성하고 한번에 하나의 set up, clean up 주기로 작성해야한다.

### 외부 시스템
- setInterval(), clearInterval()
- window.addEventListener(), window.removeEventListener()
- animation.start(), animation.reset()

<br>

## External system 연결하는 방법
> 외부 시스템: 네트워크, 브라우저 API, 라이브러리 등등 React로 제어되지 않는 코드 조각을 의미

```js
import { useEffect } from 'react';
import { createConnection } from './chat.js';

function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:3000');

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [serverUrl, roomId]);
}
```
- `useEffect`에 setup 함수와 의존성 배열을 전달한다.
  - setup 함수는 외부 시스템과 연결을 설정하고, 클린업 함수를 반환한다.
  - 의존성목록에는 setup 함수에서 참조하는 모든 반응형 값들을 포함해야한다.
- React에서는 필요할 때마다 setup, clean up 함수를 호출한다.
  1. 컴포넌트가 마운트될 때마다 setup 함수가 실행된다.
  2. 의존성배열 내 요소가 변경되어 리렌더링될 때마다 이전 state로 clean up 함수를 실행하고 그 다음 새 state로 setup 함수를 실행한다.
  3. 컴포넌트가 언마운트될 때 clean up 함수를 실행한다.


## Custom Hook으로 Effect 감싸는 방법





## React가 아닌 위젯 제어하는 방법





## Data fetching 에 Effect를 사용하는 방법





## 반응형 의존성을 지정하는 방법





## Effect의 의전 state를 기반으로 state업데이트 하는 방법
