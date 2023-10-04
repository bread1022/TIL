# useEffect

> 컴포넌트를 외부 시스템과 동기화할 수 있는 React Hook

```js
useEffect(setup, dependencies);
```

## Parameter

- `setup function`: effect 로직이 포함된 함수로 외부 시스템과 동기화하는 로직을 포함한다.
  - React 컴포넌트가 DOM에 추가되면 setup 함수가 실행되고,
  - 의존성이 변경되어 리렌더링할 때마다 클린업 함수가 있으면 이전 값으로 클린업 함수를 실행하고 새로운 값으로 setup 함수를 실행한다.
  - 컴포넌트가 DOM에서 제거되면, React는 클린업 함수를 실행한다.
- `dependencies?`: setup 코드 내에 참조된 모든 반응형 값들의 목록
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
- 등등..

<br>

## External system 연결하는 방법

> 외부 시스템: 네트워크, 브라우저 API, 라이브러리 등등 React로 제어되지 않는 코드 조각을 의미

- `useEffect`에 setup 함수와 의존성 배열을 전달한다.
  - setup 함수는 외부 시스템과 연결을 설정하고, 클린업 함수를 반환한다.
  - 의존성목록에는 setup 함수에서 참조하는 모든 반응형 값들을 포함해야한다.
- React에서는 필요할 때마다 setup, clean up 함수를 호출한다.
  1. 컴포넌트가 마운트될 때마다 setup 함수가 실행된다.
  2. 의존성배열 내 요소가 변경되어 리렌더링될 때마다 이전 state로 clean up 함수를 실행하고 그 다음 새 state로 setup 함수를 실행한다.
  3. 컴포넌트가 언마운트될 때 clean up 함수를 실행한다.

#### 채팅연결

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

#### 전역 브라우저 이벤트 수신

```js
import { useEffect, useState } from 'react';

export default function App() {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  useEffect(() => {
    const handleMove = (e) => {
      setPosition({ x: e.clientX, y: e.clientY });
    };
    window.addEventListener('pointermove', handleMove);
    return () => {
      window.removeEventListener('pointermove', handleMove);
    };
  }, []);
}
```

#### 애니메이션 트리거하기

```js
import { useEffect, useState, useRef } from 'react';
import { FadeIn } from './animation.js';

function WelcomeMessage() {
  const messageRef = useRef(null);

  useEffect(() => {
    const animation = new FadeIn(messageRef.current);
    animation.start(1000);
    return () => animation.stop();
  }, []);

  return <h1 ref={ref}>Welcome!</h1>;
}

// FadeIn (animation.js)
export class FadeIn {
  constructor(node) {
    this.node = node;
  }
  start(duration) {
    this.duration = duration;
    if (this.duration === 0) {
      // Jump to end immediately
      this.onProgress(1);
    } else {
      this.onProgress(0);
      // Start animating
      this.startTime = performance.now();
      this.frameId = requestAnimationFrame(() => this.onFrame());
    }
  }
  onFrame() {
    const timePassed = performance.now() - this.startTime;
    const progress = Math.min(timePassed / this.duration, 1);
    this.onProgress(progress);
    if (progress < 1) {
      // We still have more frames to paint
      this.frameId = requestAnimationFrame(() => this.onFrame());
    }
  }
  onProgress(progress) {
    this.node.style.opacity = progress;
  }
  stop() {
    cancelAnimationFrame(this.frameId);
    this.startTime = null;
    this.frameId = null;
    this.duration = 0;
  }
}
```

## Custom Hook으로 Effect 감싸는 방법

#### windown event listener 로직을 Custom Hook으로 추출

```js
function useWindowListener(eventType, listener) {
  useEffect(() => {
    window.addEventListener(eventType, listener);
    return () => {
      window.removeEventListener(eventType, listener);
    };
  }, [eventType, listener]);
}

// 사용시
const handleMove = (e) => setPosition({ x: e.clientX, y: e.clientY });
useWindowListener('pointermove', handleMove);
```

#### target element의 IntersectionObserver 로직을 Custom Hook으로 추출

```js
function useIntersectionObserver(ref) {
  const [isIntersecting, setIsIntersecting] = useState(false);

  useEffect(() => {
    const div = ref.current;
    const observer = new IntersectionObserver((entries) => {
      const entry = entries[0];
      setIsIntersecting(entry.isIntersecting);
    });
    observer.observe(div, {
      threshold: 1.0,
    });
    return () => {
      observer.disconnect();
    };
  }, [ref]);

  return isIntersecting;
}
```

## Data fetching 에 Effect를 사용하는 방법

```js
let ignore = false;
const [data, setData] = useState(null);

useEffect(() => {
  if (ignore) return;

  fetch(url).then((result) => {
    setData(result);
  });

  return () => {
    ignore = true;
  };
}, []);
```

- ignore flag를 사용하여 Strict mode에 중복 요청을 하지 않도록 클린업함수를 꼭 작성해야한다.
- 이렇게 하면 네트워크 요청과 응답의 순서가 다르게 도착해도 조건경합이 발생하지 않는다.
- Effect에서 데이터를 패칭하게 되면 네트워크 워터폴을 만들기가 쉽고 데이터를 미리 로드하거나 캐싱할 수 없기때문에 컴포넌트가 마운트되고 언마운트될 때 데이터 패칭을 해야한다.
- 프레임워크를 사용하는 경우, 빌트인 데이터 패칭 메커니즘을 사용하는 것이 좋고
- 또는 client-side 캐시를 사용하기 위해서 React Query나 SWR 를 사용하는 것도 좋다!

## 🌟 불필요한 객체와 함수 의존성 제거하는 방법

- 렌더링 중에 생성된 객체를 의존성 배열에 넣지 않는다.  
  -> 대신에 Effect 내에서 객체를 생성해서 사용하기!!
- 함수는 렌더링될 때마다 다른 값으로 인지하기 때문에 렌더링 중에 생성된 함수를 의존성 배열에 넣지 않는다.  
  -> 대신에 Effect 내에서 선언하여 바로 사용하는 것이 좋음!!!

  ```js
  function ChatRoom({ roomId }) {
    const [message, setMessage] = useState('');

    // 렌더링 중에 함수를 생성하고
    function createOptions() {
      return {
        serverUrl: serverUrl,
        roomId: roomId,
      };
    }

    useEffect(() => {
      const options = createOptions();
      // ❌ 의존성 배열에 함수를 넣으면 모든 리렌더링 후에 또 다시 실행하게 되는 문제 발생
      const connection = createConnection();
      connection.connect();
      return () => connection.disconnect();
    }, [createOptions]);
  }
  ```

  ```js
  const serverUrl = 'https://localhost:1234';

  function ChatRoom({ roomId }) {
    const [message, setMessage] = useState('');

    useEffect(() => {
      // ✅ Effect 내부에서 함수를 정의하고, 의존성에서 제거하기 !!! 😱
      function createOptions() {
        return {
          serverUrl: serverUrl,
          roomId: roomId,
        };
      }

      const options = createOptions();
      const connection = createConnection(options);
      connection.connect();
      return () => connection.disconnect();
    }, [roomId]);

    return (
      //...
    );
  }
  ```

<!-- ## Effect에서 최신 props 및 state 읽는 방법: `useEffectEvent`
https://react-ko.dev/reference/react/useEffect#reading-the-latest-props-and-state-from-an-effect
 -->

<br>

### Effect가 무한 실행될 때

1. Effect에서 state를 업데이트하는 경우
2. state가 업데이트되면서 리렌더링되고, 또 Effect 의존성이 변경될 때

Effect에 꼭 state를 설정해야하는지 다시 한번 확인해보고,  
외부 시스템과 동기화하는 로직이 아니라면 Effect를 제거하고 로직을 단순화하는 것이 좋다.  
또한 렌더링에 사용되지 않는 데이터를 저장할 땐 ref를 사용하는 것이 좋다.

### 브라우저 화면 리페인팅 도중 깜빡임이 발생할 때: `useLayoutEffect`

- 브라우저 리페인팅을 차단하고 Effect를 실행하려면 `useLayoutEffect`를 사용할 수 있다.
- 이때 Blocking 과정이 필요하므로 성능에 영향을 줄 수 있음을 주의해야한다.
