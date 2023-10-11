# Profiler

> Tree의 렌더링 성능 측정에 쓰임

```js
<Profiler id="App" onRender={onRender}>
  <App />
</Profiler>
```

- 컴포넌트 트리를 <Profiler>로 감싸서 렌더링 성능을 측정한다.
- Props
  - `id` : 측정할 UI를 식별할 문자열
  - `onRender Callback` : 프로파일링된 트리 내 컴포넌트가 업데이트될 때마다 React가 호출하는 onRender Callback function
    - 렌더링된 내용과 소요된 시간에 대한 정보를 인자로 받는다.
    ```js
    function onRender(
      id,
      phase,
      actualDuration,
      baseDuration,
      startTime,
      commitTime
    ) {
      // Aggregate or log render timings...
      // 렌더링 타이밍을 집계하거나 로그를 남깁니다...
    }
    ```

### onRender Callback Parameters

- `id`: 방금 커밋한 Profiler 트리의 "id"
- `phase`: "mount" | "update" | "nested-update" - 마운트 시점, 리렌더링 시점을 알 수 있다.
- `actualDuration`: 렌더링에 걸린 시간 -> 하위 트리의 메모화가 얼마나 잘 쓰였는지 알 수 있다 (최초 마운트 이후 리렌더링 시간이 많이 감소됐을 경우 Good)
- `baseDuration`: 최적화 없이 전체 트리를 렌더링하는데 걸리는 시간 (ms)
- `startTime`: 렌더링을 시작한 시점에 대한 타임스탬프
- `endTime`: 업데이트를 커밋한 시점의 타임스탬프

### 렌더링 성능 측정방법

```js
<App>
  // 🌟 측정이 필요한 컴포넌트를 감싼다.
  <Profiler id="Sidebar" onRender={onRender}>
    <Sidebar />
  </Profiler>
  <Profiler id="Content" onRender={onRender}>
    <Content>
      <Profiler id="Editor" onRender={onRender}>
        <Editor />
      </Profiler>
      <Preview />
    </Content>
  </Profiler>
</App>
```

- 트리내의 컴포넌트가 업데이트를 "커밋"할 때마다 React가 호출하는 id, onRender callback을 사용한다
- 감싸는 작업으로 렌더링 시점을 측정할 수 있는 가벼운 컴포넌트이지만 사용할 때마다 App의 CPU, memory overhead가 발생하기 때문에 필요한 경우에만 사용을 해야하고 이때문에 상용 빌드에서는 비활성화되어있으므로 개발모드에서만 사용가능하다. (상용빌드를 원하는 경우 특수 상용 빌드를 설정해야됨)
