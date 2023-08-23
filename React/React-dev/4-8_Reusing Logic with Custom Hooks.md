# Reusing Logic with Custom Hooks

## Custom Hooks: Sharing logic between components
> 커스텀 훅: 컴포넌트간의 로직 공유

```js
import { useState, useEffect } from 'react';

export default function StatusBar() {
  const [isOnline, setIsOnline] = useState(true);
  useEffect(() => {
    function handleOnline() {
      setIsOnline(true);
    }
    function handleOffline() {
      setIsOnline(false);
    }
    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);
    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);

  // return <h1>{isOnline ? 'Online' : 'Disconnected'}</h1>;
  // ...
}
```
- 여러 컴포넌트에 동일한 로직을 사용하고 싶을 때, 커스텀 훅을 직접 만들어 코드 중복을 제거할 수 있다.
- 먼저 혹으로 사용할 함수에 `use` 접두사를 붙여 선언하고 중복된 코드를 추출한 뒤, state를 반환하여 state가 필요한 곳에서 호출하여 컴포넌트간 로직을 공유할 수 있다.
- 이렇게 되면 **컴포넌트에는 반복되는 로직이 없어지고  
  컴포넌트 내부 코드가 무엇을 하는지 명확해지는 장점**이 있다.


## Extracting your own custom Hook from a component
> 컴포넌트에서 커스텀 훅 추출하기

1. 커스텀 훅 추출
    ```js
    function useOnlineStatus() {
      const [isOnline, setIsOnline] = useState(true);

      useEffect(() => {
        function handleOnline() {
          setIsOnline(true);
        }
        function handleOffline() {
          setIsOnline(false);
        }
        window.addEventListener('online', handleOnline);
        window.addEventListener('offline', handleOffline);
        return () => {
          window.removeEventListener('online', handleOnline);
          window.removeEventListener('offline', handleOffline);
        };
      }, []);
      return isOnline;
    }
    ```
2. 커스텀 훅 사용
    ```js
    import { useOnlineStatus } from './useOnlineStatus.js';
    ```
    ```js
    function StatusBar() {
      const isOnline = useOnlineStatus(); // ✅ 커스텀훅을 호출하고 반환된 값을 변수에 할당한다!

      return <h1>{isOnline ? 'Online' : 'Disconnected'}</h1>;
    }
    ```
    ```js
    function SaveButton() {
      const isOnline = useOnlineStatus();

      function handleSaveClick() {
        console.log('Progress saved');
      }

      return (
        <button disabled={!isOnline} onClick={handleSaveClick}>
          {isOnline ? 'Save progress' : 'Reconnecting...'}
        </button>
      );
    }
    ```
  - state를 공유하는 게 아니라 상태적인 로직을 공유하는 것이다.
  - 커스텀훅  내부에 있는 state변수와 effect는 컴포넌트마다 독립적으로 동작한다.


## Custom Hooks let you share stateful logic, not state itself
> 커스텀 훅은 state 자체가 아닌 상태적인 로직(stateful logic)을 공유합니다.

```js
import { useState } from 'react';

export default function Form() {
  const [firstName, setFirstName] = useState('Mary');
  const [lastName, setLastName] = useState('Poppins');

  function handleFirstNameChange(e) {
    setFirstName(e.target.value);
  }

  function handleLastNameChange(e) {
    setLastName(e.target.value);
  }

  return (
    <>
      <label>
        First name:
        <input
          value={firstName}
          onChange={handleFirstNameChange}
        />
      </label>
      <label>
        Last name:
        <input
          value={lastName}
          onChange={handleLastNameChange}
        />
      </label>
      <p>{firstName} {lastName}</p>
    </>
  );
}
```
- 위의 로직에서 state를 생성하고, e.target.value를 저장하는 chage 이벤트 핸들러, input에 대한 value와 onChage 속성을 지정하는 JSX를 반환하는 로직이 반복될 때 input에 대한 커스텀 훅을 추출할 수 있다.
```js
function useFormInput (initialValue) {
  const [value, setValue] = useState(initialValue);

  function handleChange(e) {
    setValue(e.target.value);
  }

  const inputProps = {
    value,
    onChange: handleChange,
  };
  return inputProps;
}
```
```js
import { useFormInput } from './useFormInput.js';

export default function Form() {
  const firstNameProps = useFormInput('Mary'); // ✅ 커스텀 훅 호출
  const lastNameProps = useFormInput('Poppins');

  return (
    <>
      <label>
        First name:
        <input {...firstNameProps} />
      </label>
      <label>
        Last name:
        <input {...lastNameProps} />
      </label>
      <p>{firstNameProps.value} {lastNameProps.value}</p>
    </>
  );
}
```
- 커스텀훅을 사용하면 상태 로직은 공유할 수 있지만 state 자체는 공유하는 것이 아니다!



## Passing reactive values between Hooks
> 훅 사이에 반응형 값 전달하기


```js
export default function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  // 🌟 채팅 연결하는 useEffect 내부 로직을 커스텀 훅으로 빼고 싶을 때 + 반응형 값
  useEffect(() => {
    const options = {
      serverUrl: serverUrl,
      roomId: roomId
    };
    const connection = createConnection(options);
    connection.on('message', (msg) => {
      showNotification('New message: ' + msg);
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, serverUrl]);

  return (
    <>
      <label>
        Server URL:
        <input value={serverUrl} onChange={e => setServerUrl(e.target.value)} />
      </label>
    </>
  );
}
```
- useEffect의 경우 의존성배열로 roomId, serverUrl이 필요하다. 반응형 값을 가지고 Effect 내부 로직을 커스텀 훅으로 추출하는 방법
  ```js
  // 🌟 커스텀 훅 인자로 의존성 배열에 있는 반응형값을 전달한다
  export const function useChatRoom({ serverUrl, roomId }) {
    useEffect(() => {
      const options = {
        serverUrl: serverUrl,
        roomId: roomId
      };
      const connection = createConnection(options);
      connection.connect();
      connection.on('message', (msg) => {
        showNotification('New message: ' + msg);
      });
      return () => connection.disconnect();
    }, [roomId, serverUrl]);
  }
  ```
  ```js
  export default function ChatRoom({ roomId }) {
    const [serverUrl, setServerUrl] = useState('https://localhost:1234');

    // 🌟 반응형값을 인수로 전달하고 커스텀훅을 호출
    useChatRoom({
      roomId: roomId,
      serverUrl: serverUrl
    });

    return (
      <>
        <label>
          Server URL:
          <input value={serverUrl} onChange={e => setServerUrl(e.target.value)} />
        </label>
        <h1>Welcome to the {roomId} room!</h1>
      </>
    );
  }
  ```



## Passing event handlers to custom Hooks
> 커스텀훅에게 이벤트 핸들러 전달하기


```js
export function useChatRoom({ serverUrl, roomId, onReceiveMessage }) {
  // ✅ useEffectEvent 로 감싸 의존성을 제거하여 다시 호출되는 것을 방지
  const onMessage = useEffectEvent(onReceiveMessage);

  useEffect(() => {
    const options = {
      serverUrl: serverUrl,
      roomId: roomId
    };
    const connection = createConnection(options);
    connection.connect();
    connection.on('message', (msg) => {
      onMessage(msg);
    });
    return () => connection.disconnect();
  }, [roomId, serverUrl]);
}
```

```js
export default function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useChatRoom({
    roomId: roomId,
    serverUrl: serverUrl,
    onReceiveMessage(msg) {
      showNotification('New message: ' + msg);
    }
  });
  // ...
}
```
- 커스텀 훅이 수신한 이벤트 핸들러는 Effect Event로 감싸서 의존성을 제거하여 재호출되는 것을 방지해야한다.


## When to use custom Hooks
> 커스텀 훅을 사용하는 때

- 중복되는 코드라고 해서 무조건 커스텀 훅으로 추출할 필요는 없다. 하지만 고민해보는 것은 좋다.
- Effect를 사용해야할 때 커스텀 훅으로 감싸서 의도를 명확하게 하는 것은 좋은 방법이다.

```js
function ShippingForm({ country }) {
  const [cities, setCities] = useState(null);

  useEffect(() => { // 1️⃣ fetch 하는 로직이 중복됨
    let ignore = false;
    fetch(`/api/cities?country=${country}`)
      .then(response => response.json())
      .then(json => {
        if (!ignore) {
          setCities(json);
        }
      });
    return () => {
      ignore = true;
    };
  }, [country]);

  const [city, setCity] = useState(null);
  const [areas, setAreas] = useState(null);

  useEffect(() => { // 2️⃣ fetch 하는 로직이 중복됨
    if (city) {
      let ignore = false;
      fetch(`/api/areas?city=${city}`)
        .then(response => response.json())
        .then(json => {
          if (!ignore) {
            setAreas(json);
          }
        });
      return () => {
        ignore = true;
      };
    }
  }, [city]);
}
```
- `useFetch` 커스텀 훅을 추출하여 패치하는 로직 중복을 제거할 수 있다.
  ```js
  function useData(url) {
    const [data, setData] = useState(null);

    useEffect(() => {
      if(url) {
        let ignore = false;
        fetch(url)
        .then(response => response.json())
        .then(json => {
          if(!ignore) {
            setData(json);
          }
        });
        return () => {
          ignore = true;
        };
      }
    }, [url]);
    return data;
  }
  ```
  - `useData()`커스텀 훅 사용시 데이터의 흐름을 명시적으로 만들 수 있다.
    ```js
    function ShippingForm({ country }) {
      const cities = useData(`/api/cities?country=${country}`);
      const [city, setCity] = useState(null);
      const areas = useData(city ? `/api/areas?city=${city}` : null);
      // ...
    }
    ```


### Keep your custom Hooks focused on concrete high-level use cases
> 커스텀 훅은 구체적인 고수준 사용 사례에 집중하기

- 커스텀 훅의 네이밍은 무엇을 하는지, 무엇을 반환하는지 명확하게 드러나야한다. => *용도에 따라 이름을 지정*
- 외부시스템과 동기화하는 커스텀 훅은 더 기술적이고 관련 전문용어를 사용할 수도 있다. (ex. `useMediaQuery` - query, `useSocket` - url, `useIntersectionObserver` - (ref, options))
- 커스텀훅은 수행하는 작업을 보다 선언적으로 표현할 수 있어야 제대로 생성한 것이라고 할 수 있다. (추상적인 커스텀훅은 좋지않음, 범용적으로 사용하기 보단 지역적으로 사용하는 것이 더 좋음)


## Custom Hooks help you migrate to better patterns
> 커스텀 훅은 더 나은 패턴으로 마이그레이션하는데 도움을 줍니다.

### `useSyncExternalStore`
> 외부 동작(구독)과 반응하여 동기화시켜주는 훅

```js
import { useSyncExternalStore } from 'react';

function subscribe(callback) {
  window.addEventListener('online', callback);
  window.addEnvetListener('offline', callback);
  return () => {
    window.removeEventListener('online', callback);
    window.removeEventListener('offline', callback);
  }
}

export function useOnlineStatus () {
  return useSyncExternalStore(
    subscribe,
    () => navigator.onLine,
    () => true,
  );
}
```
- 커스텀 훅 내부에서 `useSyncExternalStore`를 사용하여 마이그레이션을 하여도 컴포넌트 내부 로직은 따로 수정할 필요가 없다.


#### Effect를 커스텀훅으로 감싸는 것이 좋은 이유
1. Effect와 데이터 흐름을 명확하게 할 수 있다.
2. 컴포넌트의 코드가 Effect의 의도에 집중할 수 있다.
3. 새로운 기능을 추가할 때 컴포넌트를 수정할 필요가 없고 Effect만 수정하면 된다.



## There is more than one way to do it
> 여러가지 방법이 있습니다

```js
function Welcome() {
  const ref = useRef(null);

  useEffect(() => {
    const duration = 1000;
    const node = ref.current;

    let startTime = performance.now();
    let frameId = null;

    function onFrame(now) {
      const timePassed = now - startTime;
      const progress = Math.min(timePassed / duration, 1);
      onProgress(progress);
      if (progress < 1) {
        frameId = requestAnimationFrame(onFrame);
      }
    }

    function onProgress(progress) {
      node.style.opacity = progress;
    }

    function start() {
      onProgress(0);
      startTime = performance.now();
      frameId = requestAnimationFrame(onFrame);
    }

    function stop() {
      cancelAnimationFrame(frameId);
      startTime = null;
      frameId = null;
    }

    start();
    return () => stop();
  }, []);

  return (
    <h1 className="welcome" ref={ref}>
      Welcome
    </h1>
  );
}

export default function App() {
  const [show, setShow] = useState(false);
  return (
    <>
      <button onClick={() => setShow(!show)}>
        {show ? 'Remove' : 'Show'}
      </button>
      <hr />
      {show && <Welcome />}
    </>
  );
}
```
-  `requestAnimationFrame`을 사용한 애니메이션 구현 로직 커스텀훅으로 분리하기
    ```js
    export function useFadeIn(ref, duration) {
      useEffect(() => {
        const node = ref.current;

        let startTime = performance.now();
        let frameId = null;

        function onFrame(now) {
          const timePassed = now - startTime;
          const progress = Math.min(timePassed / duration, 1);
          onProgress(progress);
          if (progress < 1) {
            frameId = requestAnimationFrame(onFrame);
          }
        }

        function onProgress(progress) {
          node.style.opacity = progress;
        }

        function start() {
          onProgress(0);
          startTime = performance.now();
          frameId = requestAnimationFrame(onFrame);
        }

        function stop() {
          cancelAnimationFrame(frameId);
          startTime = null;
          frameId = null;
        }

        start();
        return () => stop();
      }, [ref, duration]);
    }

