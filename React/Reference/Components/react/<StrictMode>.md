# StrictMode

> 개발 중에 컴포넌트에서 발생하는 버그를 조기에 발견할 수 있게 해주는 도구

```js
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';

const root = createRoot(document.getElementById('root'));
root.render(
  <StrictMode>
    <App />
  </StrictMode>
);
```

- 리액트 프로젝트를 생성하면 `main.tsx`에는 자동으로 `<StrictMode>` 태그가 적용되어있다.
- `<StrictMode>` 태그를 사용하면 <StrictMode> 컴포넌트 내부의 전체 컴포넌트 트리에 대해 개발 전용 검사를 추가로 수행하게 되어 초기에 컴포넌트 버그를 발견할 수 있다.
  - 렌더링으로 인한 버그를 찾기위해 한번 더 렌더링하는 과정이 있다.
    - 개발환경에서 불순한 코드를 찾기위해 일부 순수함수를 2번 호출한다.
    - Component function body, useState, set Function, useMemo, useReducer에 전달한 함수들이 2번 호출된다.
  - Effect clean up이 누락되어 발생한 버그를 찾기위해 Effect를 한번 더 실행한다.
  - 트리내에 더이상 지원하지 않는 API를 사용하고 있지는 않은지 검사한다.

### 개발환경에서 이중 렌더링으로 발견된 버그 수정 방법

배열을 업데이트 해야할 땐, 배열의 복사본을 만들어서 그 복사본만 수정해야한다. (그래야 외부 객체나 변수에 영향이 가지 않음)

```js
export default function StoryTray({ stories }) {
  // ✅ 배열을 복사하여 사용해야한다!
  const items = stories.slice();
  items.push({ id: 'create', label: 'Create Story' });
}
```

### 개발환경에서 Effect를 재실행하여 발견된 버그 수정 방법

Effect에는 셋업 코드가 있으면 클린업 코드도 있어야 한다. React 컴포넌트가 마운트될 때 셋업함수를 호출하고 컴포넌트가 언마운트될 때 클린업을 호출한다.  
Strict 모드가 켜져있으면 React는 개발 환경에서 모든 Effect에 대해 set up + clean up 함수를 한번 더 실행한다.

```js
const serverUrl = 'https://localhost:1234';
const roomId = 'general';

export default function ChatRoom() {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    // ⭕️ Clean up 함수를 작성하지 않으면 Chat connect이 끊기지 않고 계쏙해서 연결되므로 꼭 clean up함수를 작성해야 누수를 방지할 수 있다.
    return () => {
      connection.disconnect();
    };
  }, []);
  return <h1>Welcome to the {roomId} room!</h1>;
}
```
