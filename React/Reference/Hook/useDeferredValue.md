# useDeferredValue

> 컴포넌트 최상위 레벨에서 호출하여 지연된 값을 사용하여 UI를 지연 업데이트할 수 있는 Hook

```js
const deferredValue = useDeferredValue(value);
```

## Parameters

```js
import { useState, useDeferredValue } from 'react';

function SearchPage() {
  const [query, setQuery] = useState('');
  const deferredQuery = useDeferredValue(query);
  // ...
}
```

- `value`: 지연시키려는 값

## Returns

- 초기 렌더링 반환값은 초기값과 동일하고, 업데이트가 발생하면 React에서 이전 값으로 리렌더링을 한 다음 백그라운드에서 새 값으로 리렌더링을 시도하여 업데이트된 새 값을 반환한다.

## 주의사항

- 전달하는 value는 원시값이거나 컴포넌트 외부에서 생성된 객체이어야한다.
- 렌더링 중 새로운 객체를 생성하고 전달하면 렌더링때마다 값이 달라지기때문에 불필요한 백그라운드 리렌더링이 발생할 수 있다
- useDeferredValue가 현재 렌더링 값 외에 다른 값을 받으면 백그라운드에서는 새 값으로 렌더링을 예약한다.

  - 값에 대한 또 다른 업데이트가 있으면 백그라운드 리렌더링은 중단될 수 있고,
  - React는 백그라운드 리렌더링을 처음부터 다시 시작한다.

  - 이때 _지연된 값을 받는 속도보다 입력 속도가 더 빠를 경우 입력이 멈춘 후에만 다시 렌더링되는 것이다.!!_

    ```js
    const [value, setValue] = useState(0);
    const deferredValue = useDeferredValue(value);

    setState(3); // 3으로 변경되면서 리렌더링 (deferredValue = 0 상태 / 백그라운드에서 3으로 변경, 리렌더링을 예약함)
    // 예약해둔 상태에서 사용자가 setState를 변경하면
    setState(6); // 앞선 예약은 무시되고 6으로 변경되면서 리렌더링 (deferredValue = 0 상태 / 백그라운드에서 6으로 변경, 리렌더링을 예약함)
    // (현재 값이랑, 지연값이 동일해졌을때) 6으로 업데이트한 상태로 리렌더링을 시도한다.
    ```

- `<Suspense>`와 통합하여 사용하면 새로운 값으로 인한 백그라운드 업데이트로 UI가 일시중단됐을 때 사용자에게 fallback이 표시되지 않고 데이터가 로드될 때까지 기존 지연된 값이 계속 표시된다.
- 화면을 업데이트 하지않을 뿐 데이터 변경을 위한 네트워크 요청을 방지하는 것은 아니다.
  - *계속해서 네트워크 요청이 진행되어 데이터 패칭을 진행했을 때 화면에 보여지는 데이터가 준비될 때까지 전달을 지연하는 것임*
  - `useDeferredValue`자체로 추가 네트워크 요청을 방지하지는 않는다.
- React는 리렌더링을 완료하자마자 `new deferred value`로 백그라운드 리렌더링을 시작하기때문에 `useDeferredValue`자체로 인해서 지연을 발생시키진 않지만, 이벤트로 인한 업데이트는 백그라운드 리렌더링을 보다 업데이트를 먼저 진행하여 지연을 발생시킬 수 있다.
- `useDeferredValue`로 인한 백그라운드 리렌더링은 화면이 커밋될 때까지 Effect를 실행하지 않고 백그라운드 리렌더링이 일시 중단되면 데이터가 로드되고 UI가 업데이트된 후 해당 Effect가 실행된다.

### 업데이트하는 동안 지연된 값 사용이 유용한 경우 ?

1. Suspense 를 지원하는 프레임워크에서 데이터 패칭할 때
2. lazy loading을 사용하여 컴포넌트를 지연 로딩할 때


### Suspense와 통합하여 사용하기

```js
import { Suspense, useState } from 'react';
import SearchResults from './SearchResults.js';

export default function App() {
  const [query, setQuery] = useState('');
  const deferredQuery = useDeferredValue(query);

  const isStale = query !== deferredQuery; // 🌟 명시적으로 값을 비교해서 CSS 차이를 적용할 수도 있다

  const handleChange = (e) => setQuery(e.target.value);

  return (
    <>
      <label>
        Search albums:
              //✅ value는 즉시 업데이트
        <input value={query} onChange={handleChange} />
        되지만
      </label>
      <Suspense fallback={<h2>Loading...</h2>}> // ✅ fallback 대신 이전 값으로 렌더링된다 !!
        <div style={{
          opacity: isStale ? 0.5 : 1,
          transition: isStale ? 'opacity 0.2s 0.2s linear' : 'opacity 0s 0s linear'
        }}>
        <SearchResults query={deferredQuery} />
        // ✅ deferredQuery는 지연된 값이므로 데이터가 로드될 때까지 이전 값을 유지함
        // 업데이트된 데이터가 준비될 때까지 오래된 지연 값을 사용하고, 데이터가 준비되면 업데이트된 값을 사용한다.
        </div>
      </Suspense>
    </>
  );
}
```


### 성능 최적화 적용하기

UI 일부 리렌더링 속도가 느려서 다른 나머지 UI를 차단하지 않도록 하려면 `useDeferredValue`을 사용해서 값 자체를 업데이트한느 것은 지연하되 UI업데이트는 즉시 발생하도록 할 수 있다..!!!!!!  
ex. 검색바에 입력할 때, 유저가 입력하는 값은 즉시 보여야되지만 유저가 입력하는 동안 검색 결과는 약간 느려도 될 때 사용!  
(차이를 두지않으면 유저가 입력할 때마다 업데이트가 되어 불필요한 리렌더링이 발생할 수 있다.)  

```js
// 🌟 react memo를 통해 props가 변경될 때만 리렌더링되는 컴포넌트
const SlowList = memo(function SlowList({ text })) {
  return text.map((char, i) => <span key={i}>{char}</span>);
}
```

```js
function App() {
  const [text, setText] = useState('');
  const deferredText = useDeferredValue(text);
  return (
    <>
      <input value={text} onChange={e => setText(e.target.value)} />
      <SlowList text={deferredText} />
      //🌟 memo를 사용하여 props가 변경되지 않으면 리렌더링이 발생하지 않기때문에
      // deferredText가 변경되지 않으면 리렌더링을 건너뛸 수 있는 것이다!!!!!!!!!!!!!
    </>
  );
}
```

### Debounce, throttle 과 useDeferredValue 의 차이

- **`디바운스`** : 사용자가 입력을 멈출 때까지 기다렸다가 마지막 입력을 처리
- **`스로틀`** : 일정한 시간 간격으로 입력을 처리
  - 개발자가 직접 고정된 지연을 적용하여 처리하고,
  - 디바운스와 스로틀을 잘 사용하면 네트워크 요청을 사용자의 입력에 비례하지 않도록 제어할 수 있다.
- **`useDeferredValue`** : 입력이 발생할 때마다 백그라운드에서 새 값을 렌더링하고, 이전 값으로 UI를 업데이트한다.
  - 사용자 기기의 성능에 따라 지연된 리렌더링이 보일수도 안보일 수도 있고,
  - `useDeferredValue`에 의해 수행되는 리렌더링은 다른 UI 업데이트가 발생하면 중단될 수도 있는 작업이다.
