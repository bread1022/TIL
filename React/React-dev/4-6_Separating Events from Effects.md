# Separating Events from Effects


사용자와의 상호작용에 의해 실행되는 이벤트 핸들러와 달리  
Effect는 prop 또는 state변수와 같은 일부 값을 마지막 렌더링때와 다른 값으로 읽게 되면 다시 동기화된다.  
값에 대한 응답으로 재실행되는 Effect와 그렇지 않은 Effect를 혼합해야하는 경우도 있다.


## Choosing between event handlers and Effects
> 이벤트 핸들러와 Effect 중 선택하기

- 이벤트 핸들러와 Effect중 선택할 때 첫번째로 고려해야하는 사항으로 **코드가 왜 실행되어야하는 지**가 판단해야한다.
- **이벤트 핸들러** : 특정 상호작용에 대한 응답으로 실행된다. (수동)
- **Effect** : 동기화가 필요할 때마다 실행된다. (자동)


## Reactive values and reactive logic
> 반응형 값 및 반응형 로직

- **이벤트 핸들러** 내부 로직은 **반응형이 아니다**.  
  사용자가 동일한 상호작용을 수행하지 않는 한 다시 실행되지 않는다.
- **Effect** 내부 로직은 **반응형(리렌더링의 결과로 변경될 수 있음)**이다.  
  Effect에서 반응형 값을 읽는 경우 의존성으로 지정하여 반응형 값을 지켜보고 있다가 리렌더링에 의해 값이 변경되면 새로운 값으로 Effect를 다시 실행한다.



## Extracting non-reactive logic out of Effects
> Effect에서 비반응형 로직 추출하기

```js
import { useState, useEffect } from 'react';
import { createConnection, sendMessage } from './chat.js';
import { showNotification } from './notifications.js';

const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId, theme }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // 채팅을 연결하는 Effect
    connection.on('connected', () => {
      showNotification('Connected!', theme); // 테마를 변경하는 Effect
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, theme]);

  return <h1>Welcome to the {roomId} room!</h1>
}
```
- 의존성 배열에 있는 `roomId`와 `theme`는 반응형 값이다.  
  하지만 `theme`만 변경되어도 채팅의 연결이 끊어졌다가 다시 연결되는 문제가 발생한다. (같은 Effect 내부에 존재하기 때문)
- 따라서 주변 반응형 Effect로 부터 비반응형 로직을 분리해야하는데,  
  `useEffectEvent()`를 사용하면 Effect 내부에서 비반응형 로직을 추출할 수 있다.


### `useEffectEvent()`
> 비반응형 로직을 Effect에서 추출할 때 사용하는 Hook
> 출시 전

```js
function ChatRoom({ roomId, theme }) {
  const onConnected = useEffectEvent(() => { // Effect Event (비반응형 로직)
    showNotification('Connected!', theme);
  });

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.on('connected', () => {
      onConnected();
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // ✅ 의존성 배열에는 roomId만 존재
  // ...
}
```
- 비반응형 로직은 Effect 내부 로직이지만 이벤트 핸들러처럼 동작한다.  
  Effect 내부에서 사용자에 의해 트리거되기 때문이다.
- 그 내부 로직은 반응형으로 동작하지 않고 항상 Props와 state의 최신값을 확인한다.
- 비반응형 로직에 사용되는 변수는 반응형이 아니기 때문에 의존성에서 생략해야하고 리렌더링에 관여하지 않는다.


## Reading latest props and state with Effect Events
> Effect Event로 최신 props 및 state 읽기


<!-- 동영상 보고 다시  -->


## Limitations of Effect Events
> Effect Event의 제한사항

- Effect 내부에서만 호출가능하다. (Effect Event를 사용하는 Effect 바로 앞에 선언하는 것이 좋음.)
  - Effect Event가 Effect코드의 비반응형 조각이기 때문
- 다른 컴포넌트나 Hook에 전달하면 안된다!