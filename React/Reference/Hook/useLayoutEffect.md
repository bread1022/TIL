# useLayoutEffect

> 렌더링에 레이아웃 정보를 사용해야할 때 브라우저 리페인팅 이전에 실행되는 Effect Hook

```js
useLayoutEffect(setup, dependencies?)
```

## Parameter

- `setup function`: Effect 로직이 포함된 함수로 선택적으로 clean up 함수를 반환할 수 있다.
  - 컴포넌트가 DOM에 추가되면 setup 함수가 실행되고,
  - 의존성이 변경되어 리렌더링할때마다 이전 값으로 clean up 함수를 실행하고 그 다음 새 값으로 setup 함수를 실행한다.
  - 컴포넌트가 DOM에서 제거되기 전에 React는 clean up 함수를 한번 더 실행한다.
- `dependencies?`: setup 코드 내에 참조된 모든 반응형 값들의 목록

## Returns

- `undefined`를 반환한다.

## 주의사항

- 컴포넌트 최상위 레벨에서만 호출할 수 있다.
- 서버렌더링 중에는 실행되지 않고 client에서만 실행되는 로직으로 브라우저가 리페인팅하기전에 내부 코드와 모든 state 업데이트가 처리되도록 보장한다.
- 이로써 리렌더링하지 않은 상태에서 필요한 요소의 레이아웃을 측정하고 이 값으로 리렌더링할 수 있게 하는 것이다.
- 브라우저 렌더링을 차단하는 Hook이기 때문에 과도하게 사용하면 성능이 느려질 수 있으므로 주의해야한다.
- 의존성 배열에 객체 또는 함수가 포함된 경우 Effect가 필요이상으로 자주 실행될 수 있으므로 의존성에서 제거하거나 비반응형 로직을 추출하는 방법을 고려해야한다.

## Repaint 전에 레이아웃 측정

- 이벤트 적용을 위해 DOM에 추가되기 전에 공간이 충분한지 확인이 필요해 레이아웃 측정을 하고 올바른 위치에 렌더링하기 위해서는 브라우저가 화면을 그리기 전에 동작해야한다.
- 브라우저가 화면을 리페인팅하기 전에 `useLayoutEffect`를 호출하면 레이아웃 측정을 수행할 수 있다.
- ex. 드롭다운패널을 띄워야하는데 맨 가장자리에 있는 버튼의 드롭다운이 뷰포트 밖으로 벗어나는 이슈가 있었음 -> 이럴 때 사용하면 좋을 것같다!

```js
function Tooltip() {
  const ref = useRef(null);
  const [tooltipHeight, setTooltipHeight] = useState(0);

  // 🌟 리페인팅 전에 값을 측정하고, 그 값을 이용하여 리렌더링함
  useLayoutEffect(() => {
    const { height } = ref.current.getBoundingClientRect();
    setTooltipHeight(height);
  }, []);

  let tooltipX = 0;
  let tooltipY = 0;
  if (targetRect !== null) {
    tooltipX = targetRect.left;
    tooltipY = targetRect.top - tooltipHeight;
    if (tooltipY < 0) {
      tooltipY = targetRect.bottom;
    }
  }

  return createPortal(
    <TooltipContainer x={tooltipX} y={tooltipY} contentRef={ref}>
      {children}
    </TooltipContainer>,
    document.body
  );
}
```

- 동작방식
  1. 초기에는 `tooltipHeight = 0`으로 설정되어 있으므로 위치가 잘못될 수 있음
  2. React에서는 useLayoutEffect를 호출하여 DOM에 추가되기 전에 ref로 지정된 DOM의 레이아웃을 측정하고 즉시 리렌더링을 트리거한다.
  3. 그럼 측정된 값으로 리렌더링하여 올바르게 배치하고 DOM을 업데이트하여 최종위치에 표시된다.

### `useLayoutEffect`가 서버에서 동작하지 않을 경우, does nothing on the server

- 일반적으로 레이아웃 정보에 의존하는 컴포넌트는 서버에서 렌더링할 필요가 없고 client 상호작용에 의존하기 때문에 서버에서는 동작하지 않는다. (서버에는 레이아웃 정보가 없으므로!)
- 해결방법
  - useEffect로 대체하면 리페인팅을 막지 않고 초기 렌더링결과를 표시할 수 있다.
  - client-only 컴포넌트로 생성하여 <Suspense>에 의해 로딩 폴백을 표시할 수도 있다.
  - 서버렌더링을 지원하는 `useSyncExternalStore` 을 사용할 수도 있다.
