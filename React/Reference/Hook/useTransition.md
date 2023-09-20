# useTransition
> UI를 차단하지 않고 state 업데이트할 수 있는 React Hook

```js
import { useTransition } from 'react';

const [isPending, startTransition] = useTransition();
```

## Parameters

- 매개변수를 받지 않는다.

## Returns

- 2개의 값을 반환한다.
  - `isPending`: 트랜지션이 진행중일 때 true를 반환하는 boolean 값
  - `startTransition function`: state 업데이트를 트랜지션으로 나타낼 수 있는 함수


## startTransition function

- `useTransition`이 반환하는 startTransition함수를 사용하면 state업데이트를 트랜지션으로 표시할 수 있다.
- 매개변수 : set 함수를 호출하여 state를 업데이트하는 함수 하나 이상
  - 동기적으로 예약된 state 업데이트를 트랜지션으로 나타낸다.
  - UI를 차단하지 않고 Non-blocking으로 state를 업데이트할 수 있고,   
  - 원치않는 로딩 표시를 방지할 수 있다.
  - 매개변수로 전달한 함수는 비동기로 동작하지 않고 즉시 실행되며 함수가 실행되는 동안 예약된 state업데이트를 트랜지션으로 나타내는 것이다. (비동기X)
- 반환값 : 없음

## 주의사항

- `useTransition`은 리액트 컴포넌트 내부에서만 호출이 가능하다.
- 다른 데이터 라이브러리에서 트랜지션을 사용할 땐 `startTransition`을 따로 호출해야한다.


## UI 표시를 Non-blocking 하면서 state 업데이트하는 방법

```js
import { useState, useTransition } from 'react';

function TabContainer () {
  const [isPending, startTransition] = useTransition();
  const [tab, setTab] = useState('home');

  const selectTab = (nextTab) => {
    // 🌟 transition을 사용하면 UI blocking을 막을 수 있음
    // 느린 디바이스에서도 사용자 인터페이스 업데이트 반응성 유지가 가능!!
    startTransition(() => {
      setTab(nextTab);
    });
  }
}
```

## 'Pending' state 표시하여 트랜지션 진행중을 표시하는 방법

```js
export default function TabButton({ children, isActive, onClick }) {
  const [isPending, startTransition] = useTransition();
  if (isActive) {
    return <b>{children}</b>
  }
  if (isPending) {
    return <b className="pending">{children}</b>;
  }
  // isActive도 아니고, isPending도 아닐 때 버튼 반환
  return (
    <button onClick={() => {
      startTransition(() => {
        onClick();
      });
    }}>
      {children}
    </button>
  );
}
```

## 원치않는 로딩 표시 방지하기

```js
import { Suspense, useState } from 'react';
import TabButton from './TabButton.js';
import AboutTab from './AboutTab.js';
import PostsTab from './PostsTab.js';
import ContactTab from './ContactTab.js';

export default function TabContainer() {
  const [tab, setTab] = useState('about');
  return (
    //  🌟 Suspense 사용하면 내부 컴포넌트들이 로딩중일 때 fallback을 대체한다.
    // 이때 작은 컴포넌트 단위로 pending을 이용하면 원치않는 로딩 표시를 방지할 수 있다.
    <Suspense fallback={<h1>🌀 Loading...</h1>}>
      <TabButton
        isActive={tab === 'posts'}
        onClick={() => setTab('posts')}
      >
        Posts
      </TabButton>
      <hr />
      {tab === 'posts' && <PostsTab />} // 계산이 느린 컴포넌트일 경우,
    </Suspense>
  );
}
```
```js
export default function TabButton({ children, isActive, onClick }) {
  const [isPending, startTransition] = useTransition();
  if (isActive) {
    return <b>{children}</b>
  }
  if (isPending) {
    return <b className="pending">{children}</b>;
  }
  return (
    <button onClick={() => {
      // ✅ 해당 컴포넌트에 한해서 pending 상태에 대한 트랜지션을 표시할 수 있다.
      startTransition(() => {
        onClick();
      });
    }}>
      {children}
    </button>
  );
}
```


## 페이지 네비게이션을 트랜지션으로 표시하는 방법

리액트로 라우터를 구축하는 경우 페이지 네비게이션을 트랜지션으로 표시하는 것이 좋다.  
트랜지션은 중단이 가능하여 렌더링이 완료 될 때까지 사용자 인터렉션 반응성을 유지할 수 있고  
원하지 않은 로딩표시와 갑작스러운 네비게이션 이동을 방지할 수 있다.  

#### 트랜지션을 네비게이션에 적용한 예제
```js
export default function App() {
  return (
    <Suspense fallback={<BigSpinner />}>
      <Router />
    </Suspense>
  );
}

const Router = () => {
  const [page, setPage] = useState('/');
  const [isPending, startTransition] = useTransition();

  const navigate = (url) => {
    // 🌟 트랜지션 사용
    startTransition(() => setPage(url));

    let content;
    if (page === '/') {
      content = (
        <IndexPage navigate={navigate} />
      );
    } else if (page === '/the-beatles') {
      content = (
        <ArtistPage/>
      );
    }
    return (
      <Layout isPending={isPending}>
        {content}
      </Layout>
    );
  }
}
```

### input state 업데이트에는 트랜지션을 적용할 수 없다

<!-- 영상 보고 다시하기 -->

```js
const [text, setText] = useState('');

function handleChange(e) {
  // ❌ 트랜지션 적용할 수 없음
  startTransition(() => {
    setText(e.target.value);
  });
}

return <input value={text} onChange={handleChange} />;
```

### 비동기함수와 트랜지션을 사용해야할 때

startTarnsition 매개변수의 scope은 동기 함수만 전달이 가능하다. 만약 비동기로 state를 업데이트 해야할 경우에는 비동기함수의 콜백함수에 startTransition을 전달해야한다.

```js
startTransition(() => {
  // ❌ 비동기함수 트랜지션으로 표시할 수 없음
  setTimeout(() => {
    setPage('/about');
  }, 2000);
});
```
```js
// ⭕️ 트랜지션을 비동기함수의 콜백함수로 전달하면 트랜지션으로 표시할 수 있다.
setTimeout(() => {
  startTransition(() => {
    setPage('/about');
  });
}, 2000);
```

```js
startTransition(async () => {
  // ❌ async, await 을 적용할 수 없음
  await someAsyncFunction();
  setPage('/about');
});

await someAsyncFunction();
// ⭕️ 함수를 분리하는 방법으로 해결할 수 있음
startTransition(() => {
  setPage('/about');
});
```
