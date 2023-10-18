# useSyncExternalStore

> 외부 스토어를 구독할 수 있는 React Hook

```js
const snapshot = useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot?)
```

컴포넌트 최상위 레벨에서 호출하여 외부 데이터 저장소에서 값을 읽을 수 있다.  
`subscribe` 함수를 통해 외부 데이터 저장소의 변경사항을 구독하고,  
`getSnapshot` 함수를 통해 외부 데이터 저장소의 현재 값을 읽어온다.

주로 기존의 비React코드와 통합해야할 때 유용하며 useState, useReducer와 함께 사용하는 것이 좋다.

## Parameters

- `subscribe`: callback을 받아 스토어를 구독하는 함수
  - 리렌더링사이에 구독함수에 의한 성능 문제가 발생하거나 스토어 재구독을 피하고 싶다면 `useCallback`을 사용하거나 컴포넌트 외부로 이동하는 것이 좋다.
- `getSnapshot`: 스토어 데이터의 스냅샷을 반환하는 함수 (클린업). 스토어가 변경되지 않은 상태에선 동일값을 반환해야하고 저장소가 변경되어 반환값이 달라지면 리렌더링한다.
- `getServerSnapshot`: 스토어에 있는 데이터 초기 스냅샷을 반환하는 함수 (리턴값).
  - 실제로 변경된 사항이 있는 경우에만 다른 객체를 반환해야한다. 스토어에 불변 데이터가 포함된 경우 해당 데이터를 직접 반환할 수 있다.
  - 오직 서버에서 렌더링할때 & 클라이언트에서 하이드레이트할 때에 초기값을 설정에 사용된다. (React가 서버 HTML을 생성할 때 서버에서 실행됨)
  - App이 상호하기 전에 사용될 초기 스냅샷을 제공할 수 있고 서버렌더링에 의미있는 초기값이 없을 경우 컴포넌트가 client에서만 렌더링되게 강제 설정할 수도 있다.
  <!--!!!! 동영상보고 다시 - 초기클라이언트 렌더링과 서버에서 반환한 초기값이 동일한지 확인하는 과정이 필요하므로 `<script>` 태그를 생성하여 클라이언트에서 글로벌값을 가져오는 방법을 사용하는 것이 좋다. -->


## Returns

- 렌더링에 사용할 수 있는 스토어 현재 스냅샷

## 주의사항

- `getSnapshot`이 반환하는 스토어 스냅샷은 불변이어야한다.
- 가변인 경우, 값이 변경되면 새로운 불변 스냅샷을 반환하도록 해야한다.
- `subscribe`함수는 컴포넌트 외부에 작성하여 리렌더링할때마다 새로운 함수가 생성되지 않도록 해야한다.


### 브라우저 API 구독하는 방법 (ex. 네트워크 연결 활성화 여부를 표시해야 될 때)

브라우저에서 시간이 지남에 따라 네트워크 연결 상태를 구독하고 싶을 때,  
React가 알지못하는 사이에 변경될 수 있으므로 `useSyncExternalStore`를 사용하여 구독한다.

```js
import { useSyncExternalStore } from 'react';

// 1️⃣ 브라우저 API에서 현재 네트워크 상태를 읽는 함수
function getSnapshot() {
  return navigator.onLine;
}

// 2️⃣ navigator.onLine이 변경되었을 때 브라우저에서 실행하는 이벤트를 구독하고, 클린업을 반환하는 함수
function subscribe(callback) {
  window.addEventListener('online', callback);
  window.addEventListener('offline', callback);
  return () => {
    window.removeEventListener('online', callback);
    window.removeEventListener('offline', callback);
  };
}

// 3️⃣ useSyncExternalStore의 매개변수로 구독함수와 반환값을 생성하는 함수를 전달하여 외부 스토어를 구독하는 기능을 구현할 수 있다! ! ! !
function useNetworkStatus() {
  const isOnline = useSynExternalStore(subscribe, getSnapshot);
  // ...
  return <h1>{isOnline ? '✅ Online' : '❌ Disconnected'}</h1>;
}
```

### 구독하는 커스텀훅 생성방법

일반 컴포넌트에서 `useSyncExternalStore`를 호출하여 구독하는 것보단,  
커스텀훅으로 생성하여 서로 다른 컴포넌트에서 동일한 외부스토어를 구독하게 할 수 있다.

```js
import { useSyncExternalStore } from 'react';

function useNetworkStatus() {
  const status = useSynExternalStore(subscribe, getSnapshot);
  // ...
  return status;
}

function getSnapshot() {
  return returnValue;
}

function subscribe(callback) {
  setupfuction();
  return () => {
    cleanupFunction();
  };
}
```

```js
function ComponentA() {
  const status = useNetworkStatus(); // ✅ 사용해야하는 컴포넌트에서 커스텀훅을 호출하여 외부스토어를 구독할 수 있따!~!
  // ...
}
```