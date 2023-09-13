# lazy
> 컴포넌트의 로딩을 지연시킬 수 있는 API

```js
const component = lazy(load);
```
```js
import { lazy } from 'react';

// 동적 import
const MarkdownPreview = lazy(() => import('./MarkdownPreview.js'));
```

- 크기가 큰 리액트App에서 첫 페이지를 로드하는 즉시 모든 JS 번들을 다운로드하게되면 페이지 성능에 영향을 줄 수 있으므로 코드를 분할 하고 지연로딩을 적용해야한다.
- React.lazy 함수는 JS 청크로 분리하는 방법을 제공하여 코드 분할 요소를 동적으로 로드할 수 있고  
  `<Suspense>`는 로드가 완료될 때까지 fallback을 표시할 수 있게한다.
- 이때 약간의 지연이 발생할 수 있으므로 `<Suspense>` 컴포넌트와 함께 사용하여 로딩 상태를 표시하는 것이 좋다.

## Parameters

- `load`: `Promise` 또는 다른 `thenable 객체(then 메서드를 가진 Promise 유사객체)`를 반환하는 함수
  - 반환된 컴포넌트를 처음 렌더링할 때까지는 이 함수를 실행하고 있지 않다가 렌더링해야되는 시점에 함수를 호출한다.
  - load함수를 처음 호출한 뒤 resolve될 때까지 기다린 다음 그 결과를 캐시하고 컴포넌트로 렌더링한다. (이후에는 캐시된 결과를 사용하기때문에 load를 두번이상 호출하지 않음)
  - reject되면 가장 가까운 `<Error> Boundary` 로 error를 전달한다.


## Return

- React 컴포넌트를 반환한다.
- lazy Comoponent가 로딩되는 동안 렌더링을 시도하면 렌더링이 일시중단되므로 `<Suspense>`를 사용하여 로딩중임을 표시할 수도있다.


## load function

- 매개변수는 없으며 반환값으로 Promise 또는 다른 thenable 객체를 반환해야한다.


### Suspense가 있는 지연 로딩 컴포넌트

```js
import { useState, Suspense, lazy } from 'react';
import Loading from './Loading.js';

// ✅ 모듈의 최상위 컴포넌트의 밖에서 지연 컴포넌트를 선언한다!
const MarkdownPreview = lazy(() => delayForDemo(import('./MarkdownPreview.js')));

export default function MarkdownEditor() {
  const [showPreview, setShowPreview] = useState(false);
  const [markdown, setMarkdown] = useState('Hello, **world**!');

  return (
    <>
      <textarea value={markdown} onChange={e => setMarkdown(e.target.value)} />
      <label>
        <input type="checkbox" checked={showPreview} onChange={e => setShowPreview(e.target.checked)} />
        Show preview
      </label>
      <hr />
      {showPreview && (
        // MarkdownPreview 가 로드될 때까지 Loading 컴포넌트를 렌더링한다.
        <Suspense fallback={<Loading />}>
          <h2>Preview</h2>
          <MarkdownPreview markdown={markdown} />
        </Suspense>
      )}
    </>
  );
}
```
- 처음 렌더링될 때까지 로딩을 하지 않고, 실제로 필요할 때 렌더링 시키고자 할 때 lazy를 사용하여 로딩을 지연시킬 수 있다.
- 로드되는 작업이 비동기로 처리되기 때문에 Suspense 컴포넌트로 감싸서 로드되는 동안 표시할 fallback을 지정할 수 있다.
  - `<Suspense>`로는 지연컴포넌트나 부모컴포넌트를 감싸면 된다.
- 최상위 컴포넌트의 밖에서 선언해야한다. 컴포넌트 내부에 선언할 경우 리렌더링할 때마다 컴포넌트를 새로 불러오는 것이기때문에 state가 초기화된다.
  ```js
  import { lazy } from 'react';

  function Editor() {
    // ❌ 리렌더링마다 컴포넌트 내부 state가 초기화된다.
    const MarkdownPreview = lazy(() => import('./MarkdownPreview.js'));
    // ...
  }
  ```

