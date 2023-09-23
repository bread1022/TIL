# Removing Effect Dependencies

## Dependencies should match the code
> 의존성은 코드와 일치해야한다.

1. Effect 내부 로직에는 시작과 중지하는 방법을 먼저 지정해야한다.
2. 의존성에 필요한 요소를 올바르게 지정해야한다.
3. 반응형 값을 받으면 의존성에 지정해야하고, Effect는 동기화되어야한다.


## To remove a dependency, prove that it’s not a dependency
> 의존성을 제거하려면 의존성이 아님을 증명해야한다.

- **반응형 값** : props와 컴포넌트 내부에서 선언된 모든 변수 및 함수
  - Effect에서 이 반응형값을 읽으면 의존성 목록에 꼭 추가해야한다.
- 의존성 제거를 위해 컴포넌트 외부로 이동이 가능한지 확인하고 리렌더링시에도 변경되지 않음이 확인된다면 의존성 목록에서 제거할 수 있다.


## To change the dependencies, change the code
> 의존성을 변경하려면 주변 코드를 변경해야한다.

- 의존성을 변경하기 위해선 먼저 Effect 코드, 반응형 값 선언방식을 변경해보고 변경한 코드에 맞게 의존성을 조정하는 것이 좋다.


## Removing unnecessary dependencies
> 불필요한 의존성 제거하기

- 다른 조건에서 Effect의 다른 부분을 다시 실행하고 싶을 수도 있다.
- 일부 의존성 변경에 반응하지 않고 최신 값만 읽고싶을 수도 있다.
- 의존성은 객체나 함수이기 때문에 의도치않게 너무 자주 변경될 수도 있다.


1. #### Should this code move to an event handler?  
   (이벤트 핸들러로 옮겨야 하는가?)
     - 가장 먼저 고려해야할 것 : 이 코드가 Effect가 되어야하는 지 여부
     - 특정 상호작용에 의한 응답으로 일부 코드를 실행하는 경우 이벤트 핸들러가 더 적합하다.
2. #### Is your Effect doing several unrelated things?  
   (Effect가 여러 관련 없는 작업을 하고 있는가?)
     - Effect가 여러 작업을 수행하는 경우, 각 작업에 대한 Effect를 각각 생성해야한다.
       ```js
       function ShippingForm({ country }) {
         const [cities, setCities] = useState(null);
         const [city, setCity] = useState(null);
         const [areas, setAreas] = useState(null);

         useEffect(() => {
           let ignore = false;
           fetch(`/api/cities?country=${country}`)
             .then(response => response.json())
             .then(json => {
               if (!ignore) {
                 setCities(json);
               }
             });
           if (city) {
             fetch(`/api/areas?city=${city}`)
               .then(response => response.json())
               .then(json => {
                 if (!ignore) {
                   setAreas(json);
                 }
               });
           }
           return () => ignore = true;
         }, [country, city]); // country에 의해 city가 변경되고, city변경에 의해 country의 불필요한 패칭이 발생함
         // ❌ 서로 관련없는 두가지를 동기화하고 있다는 것이 문제!
         // ...
       }
       ```

       ```js
       function ShippingForm({ country }) {
         const [cities, setCities] = useState(null);
         useEffect(() => {
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
         }, [country]); // ✅ country가 변경될 때만 실행됨

         const [city, setCity] = useState(null);
         const [areas, setAreas] = useState(null);
         useEffect(() => {
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
         }, [city]); // ✅ city가 변경될 때만 실행됨
         // ...
       }
       ```
       - 개별 의존성 목록으로 분리해야 의도치 않게 서로의 로직을 촉발하지 않는다.
       - 코드 자체의 길이가 길어지더라도 각 Effect는 독립적인 동기화 프로세스를 나타내야하므로 Effect를 분할하는 것이 중요하다.

<br>

## `useEffectEvent`

### Wrapping an event handler from the props
> props를 이벤트 핸들러로 감싸기

`useEffect`에서 동작하지만, 값의 변경에 따라 반응하지 않고 읽기만 할 땐 (Effect에서 반응해서는 안되는 로직 = 비반응로직) Effect Event로 옮겨야 한다. 


```js
  // 이벤트핸들러를 prop으로 받았을 때
function ChatRoom({ roomId, onReceiveMessage }) {
  const [messages, setMessages] = useState([]);

  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on('message', (receivedMessage) => {
      onReceiveMessage(receivedMessage);
    });
    return () => connection.disconnect();
  }, [roomId, onReceiveMessage]);
  // 부모 컴포넌트가 렌더링할 때마다 다른 이벤트핸들러를 전달하면
  // 의존성변경에 의해 리렌더링때마다 Effect가 다시 동기화되어 채팅이 또 다시 연결되는 문제가 발생한다.
  // ...
}
```

```js
function ChatRoom({ roomId, onReceiveMessage }) {
  const [messages, setMessages] = useState([]);

  // ✅ Effect Event는 비반응로직이므로 의존성에서 제거해야한다.
  // 부모 컴포넌트가 렌더링될 때마다 다른 이벤트핸들러가 전달되어도
  // Effect가 다시 실행되지 않는다!!!!
  const onMessage = useEffectEvent(receivedMessage => {
    onReceiveMessage(receivedMessage);
  });

  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on('message', (receivedMessage) => {
      onMessage(receivedMessage);
    });
    return () => connection.disconnect();
  }, [roomId]);
  // ...
}
```


### 🌟 Separating reactive and non-reactive code
> 반응형 코드와 비반응형 코드 분리

```js
function Chat({ roomId, notificationCount }) {
  // ✅ 이벤트핸들러를 반응하지 않게 하기 위해서 Effect Event로 분리해야한다.
  const onVisit = useEffectEvent(visitedRoomId => {
    logVisit(visitedRoomId, notificationCount);
  });

  useEffect(() => {
    onVisit(roomId);
  }, [roomId]); // roomId가 변경될 때마다 방문을 기록하는 로직
  // ...
}
```


## Does some reactive value change unintentionally?
> 일부 반응형 값이 의도치 않게 변경될 떄!


```js
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  const options = {
    serverUrl: serverUrl,
    roomId: roomId
    // message를 입력할 때마다 리렌더링되고
    // 리렌더링될때마다 options가 새로 생성되어 (다른 것이라고 판단) Effect가 다시 동기화된다.
  };

  useEffect(() => {
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [options]);
  // ❌ 객체 및 함수 의존성으로 인해 Effect가 필요 이상으로 자주 재동기화됨
  // ...
}
```

의존성배열에 객체와 함수를 넣을 경우, 안에 내용이 동일할 수 있더라도 새로 생성된 객체와 함수가 다르다고 판단하여 Effect가 필요이상으로 자주 재동기화되는 문제가 발생할 수 있다. 그렇기때문에 가능하면 객체와 함수는 Effect의 의존성으로 사용하지 않는 것이 좋다.  
대신에 컴포넌트 외부나 Effect내부, 원시값으로 추출하는 등 다른 방법을 사용해야한다.


### Move static objects and functions outside your component
> 정적 객체와 함수를 컴포넌트 외부로 이동

```js
// 컴포넌트 외부에서 선언되므로 반응형 값이 아님
const options = {
  serverUrl: 'https://localhost:1234',
  roomId: 'music'
};

function ChatRoom() {
  const [message, setMessage] = useState('');

  useEffect(() => {
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, []); // ✅ 의존성에서 options를 제거하고 컴포넌트 외부로 이동
}
```

### Move dynamic objects and functions inside your Effect
> Effect 내에서 동적 객체 및 함수 이동


```js
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  useEffect(() => {
    // ✅ 리렌더링의 결과로 변경될 수 있는 roomId(반응형)값에 의존하는 경우에는 Effect코드 내부로 options 이동시킴
    const options = {
      serverUrl: serverUrl,
      roomId: roomId
    };
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);
  // ...
}
```


### Read primitive values from objects
> 객체에서 원시 값 읽기


```js
function ChatRoom({ options }) {
  const [message, setMessage] = useState('');

  useEffect(() => {
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [options]);
  // ...
}

// 부모컴포넌트에서 객체를 생성하여 props로 전달할 경우
<ChtRoom
  roomId={roomId}
  // ❌ 부모컴포넌트가 리렌더링될 때마다 하위컴포넌트의 Effect도 다시 연결됨
  options={{
    serverUrl: serverUrl,
    roomId: roomId
  }}
/>
```
- 부모컴포넌트에서 객체를 생성하여 prop으로 전달할 경우 렌더링 중 객체를 생성하고  
  하위컴포넌트는 새로운 객체를 받기 때문에 리렌더링때마다 Effect가 다시 연결된다.
  이 문제를 해결하기 위해서는 Effect 외부에서 객체를 구조분해할당하여 중첩된 원시값을 읽어 의존성을 제거해야한다.


```js
function ChatRoom({ options }) {
  const [message, setMessage] = useState('');

  const { roomId, serverUrl } = options;
  // ✅ 중첩된 원시값을 읽고 Effect에서 동일한 값임을
  // 확인한 객체를 생성하기 때문에 의도치않은 리렌더링을 방지할 수 있음
  // options.roomId, options.serverUrl의 값이 변경되지 않는 한 Effect는 다시 동기화되지 않음

  useEffect(() => {
    const connection = createConnection({
      roomId: roomId,
      serverUrl: serverUrl
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, serverUrl]);
  // ...
}
```

### Calculate primitive values from functions
> 함수에서 원시값 계산

```js
function ChatRoom({ getOptions }) {
  const [message, setMessage] = useState('');

  const { roomId, serverUrl } = getOptions();  // ✅ 의존성 제거

  useEffect(() => {
    const connection = createConnection({
      roomId: roomId,
      serverUrl: serverUrl
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, serverUrl]);
  // ...
}

// 부모컴포넌트에서 함수를 props로 전달할 경우
<ChatRoom
  roomId={roomId}
  getOptions={() => {
    return {
      serverUrl: serverUrl,
      roomId: roomId
    };
  }}
/>
```
- 부모컴포넌트에서 함수를 prop으로 전달하여 하위컴포넌트에서 그 prop에 대한 의존성을 제거하기위해서는 Effect 외부에서 호출하여 값을 읽어야 렌더링중에 호출해도 안전하다.
- **함수가 이벤트 핸들러이지만 변경사항에 의해 Effect가 다시 동기화되는 것을 원하지 않는 경우 `Effect Event`로 감싸야한다.**