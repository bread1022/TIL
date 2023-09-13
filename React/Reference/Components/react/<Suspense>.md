# `<Suspense>`
> 로딩이 완료될 때까지 fallback에 전달된 컴포넌트를 표시한다.

## Props

- `fallback`: 로딩중일 때 실제UI를 대체해서 렌더링할 UI 컴포넌트
- `children`: Suspense 내부에 렌더링할 실제 UI 컴포넌트
  ```js
  <Suspense fallback={<로딩중에 대체하려는 UI>}>
    <렌더링하려는 실제 UI> // children
  </Suspense>
  ```
  - children이 로딩중인 경우엔 fallback으로 전환되고, 로딩이 완료되면 children으로 전환된다.

### 주의사항

- 첫 마운트되기전 일시중단이 되면 렌더링 도중의 state를 보존하지 않고 재시작한다.
- 일시중단되어 이미 표시된 콘텐츠를 숨겨야되는 경우, 콘텐츠트리에서 layout effect를 클린업한다.
<!-- ??? 주의사항..무슨소리지 -->
- Suspense를 사용할 수 있는 경우
  - Next.js, Relay 같은 Suspense를 도입한 프레임워크에서
  - React.lazy()를 사용한 지연 로딩 컴포넌트에서
- Effect나 이벤트 핸들러 내부에서 패칭하는 경우는 감지하지 않는다.

<br>

### 콘텐츠 로딩동안 fallback을 표시하는 방법

- 데이터를 로드할 때까지 대체컴포넌트를 표시하고 싶은 경우 React의 `<Suspense>` 컴포넌트를 사용한다.
- 렌더링 준비가 다 될 때까지 React는 가장 가까운 상위 Suspense의 fallback을 전환하여 표시하고 이후 데이터가 로드되면 fallback을 숨기고 표시하려던 자식 컴포넌트를 렌더링한다.


### 콘텐츠를 한번에 드러내는 방법

```js
<Suspense fallback={<Loading />}>
  <Biography />
  <Panel>
    <Albums />
  </Panel>
</Suspense>
```
- children의 모든 항목이 준비되면 한번에 표시된다.
- 함께 동작해야되는 특별한 이유가 없다면 로드에 오래걸리는 요소만 children에 넣으면 된다.



### 중첩된 콘텐츠가 로드될 때 표시

```js
<Suspense fallback={<BigSpinner />}>
  <Biography /> // 가장 가까운 Suspense의 fallback으로 전환
  <Suspense fallback={<AlbumsGlimmer />}> // 내부 항목 로딩에만 fallback을 사용
    <Panel>
      <Albums />
    </Panel>
  </Suspense>
</Suspense>
```
- 컴포넌트가 일시중단되면 가장 가까운 상위 Suspense의 fallback으로 전환된다.
- 여러 Suspense가 중첩되어있는 경우, 가장 가까운 Suspense의 fallback으로 전환된다.


### `useDeferredValue`: 새 콘텐츠가 로드될 때 오래된 콘텐츠로 대체
> UI 일부의 업데이트를 지연시킬 수 있는 React Hook

```js
export default function App() {
  const [query, setQuery] = useState('');
  const deferredQuery = useDeferredValue(query);

  return (
    <>
      <label>
        Search albums:
        <input value={query} onChange={e => setQuery(e.target.value)} />
      </label>
      <Suspense fallback={<h2>Loading...</h2>}>
      // ✅ 오래된 값을 반환해놓고 새로운 값을 로드한다.
        <div style={{ opacity: isStale ? 0.5 : 1 }}> // 시각적인 효과를 추가함
          <SearchResults query={deferredQuery} />
        </div>
      </Suspense>
    </>
  );
}
```
- 매개변수로 지연시키려는 값을 넣고 최상위 컴포넌트에서 호출하면 지연된 버전의 값을 반환한다.
    ```js
    import { useState, useDeferredValue } from 'react';

    function SearchPage() {
      const [query, setQuery] = useState('');
      const deferredQuery = useDeferredValue(query);
      // ...
    }
    ```

<!-- ?? 질문 이런경우에는 그럼 fallback에 로딩 컴포넌트를 넣지 않아도 되는 거 아닌가 ..!! 처음에 저장된 값 없을때 필요해서 추가해야되는가.. -->


### `useTransition`, `startTransition`

- `useTransition`: 트랜지션이 발생중임을 나타내는 React Hook
  - 반환값 : `isPending`
    - 트랜지션이 발생중임을 나타내는 boolean
- `startTransition`: state 업데이트를 transition으로 표시할 수 있는 React API
  - `startTransition`을 사용하면 탐색 state 업데이트를 transition으로 표시할 수 있다.
  - 리액트에게 state 전환이 급한게 아니면 이미 표시된 콘텐츠를 숨기지않고 이전 페이지를 표시하도록 알려준다.
  - transition을 사용하면 느린 기기에서도 사용자 인터페이스 업데이트를 반응성있게 유지할 수 있다. (리렌더링중에도 반응성이 유지됨)
    ```js
    import { startTransition } from 'react';

    function TabContainer() {
      const [tab, setTab] = useState('about');

      function selectTab(nextTab) {
        startTransition(() => {
          setTab(nextTab);
        });
      }
    }
    ```


```js
export default function App() {
  return (
    <Suspense fallback={<BigSpinner />}>
    // ❌ Router가 일시중단되면 표시되었던 콘텐츠들도 숨겨짐
      <Router />
    </Suspense>
  );
}

function Router() {
  const [page, setPage] = useState('/');

  function navigate(url) {
    setPage(url);
  }

  let content;
  if (page === '/') {
    content = (
      <IndexPage navigate={navigate} />
    );
  } else if (page === '/the-beatles') {
    content = (
      <ArtistPage
        artist={{
          id: 'the-beatles',
          name: 'The Beatles',
        }}
      />
    );
  }
  return (
    <Layout>
      {content}
    </Layout>
  );
}
```
```js
export default function App() {
  return (
    <Suspense fallback={<BigSpinner />}>
      <Router />
    </Suspense>
  );
}

function Router() {
  const [page, setPage] = useState('/');
  // 🌟 isPending 불리언값을 사용하여 트랜지션을 표시함
  const [isPending, startTransition] = useTransition();

  function navigate(url) {
    // 🌟 state 업데이트를 트랜지션으로 표시하게함
    startTransition(() => {
      setPage(url);
    });
  }

  let content;
  if (page === '/') {
    content = (
      <IndexPage navigate={navigate} />
    );
  } else if (page === '/the-beatles') {
    content = (
      <ArtistPage
        artist={{
          id: 'the-beatles',
          name: 'The Beatles',
        }}
      />
    );
  }
  return (
    // isPending 값을 받아서 Layout에 넘겨줌
    <Layout isPending={isPending}>
      {content}
    </Layout>
  );
}

// Layout은 props로 isPending을 받아서 로딩중일 때 opacity를 설정할 수 있음
export default function Layout({ children, isPending }) {
  return (
    <div className="layout">
      <section className="header" style={{
        opacity: isPending ? 0.7 : 1
      }}>
        Music Browser
      </section>
      <main>
        {children}
      </main>
    </div>
  );
}
```

### 서버 오류 전용 fallback

- 스트리밍 서버 렌더링 API를 사용하는 경우 서버에서 발생한 오류를 처리하기 위해 `<Suspense> Boundary`를 사용하기도 한다. 컴포넌트가 서버에서 에러를 던져도 React는 렌더링을 중단하지 않고 가장 가까운 Suspense의 fallback을 표시한다.
- 클라이언트에서는 다시 동일한 컴포넌트를 리렌더링하려고 시도하고, 에러가 발생하면 가장 가까운 `<Error> Boundary`를 표시하고 에러가 발생하지 않으면 성공적으로 렌더링을 하게되는 것이다.

#### 업데이트중에 UI가 폴백으로 대체되는 것을 방지하는 방법 : `startTransition`

- 업데이트하다 컴포넌트 일시중단으로 인한 `<Suspense>` fallback이 표시되는 것을 방지하기위해 `startTransition`을 사용하여 데이터 로드가 충분해질 때까지 state 업데이트를 트랜지션으로 표시한다.
- 기존 데이터를 유지하여 UI를 숨기지 않고 있다가 새 데이터가 준비되면 UI를 업데이트한다.


<!--?? 질문 1. useFetch에서 Loading state를 이용해서 로딩중을 표시한 이유 (suspense를 사용하지 않은 이유) -->
<!-- ?? startTransition?? -->