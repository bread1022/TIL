# useInsertionEffect

> DOM 변이 이전에 실행되는 useEffect Hook

```js
useInsertionEffect(setup, dependencies);
```

- Hook을 호출하여 DOM 변이 전에 스타일을 주입할 수 있다.

## Parameter

- `setup function`: Effect 로직이 포함된 함수로 선택적으로 clean up 함수를 반환할 수 있다.
  - 컴포넌트가 DOM에 추가되면 setup 함수가 실행되고,
  - 의존성이 변경되어 리렌더링할 때마다 이전 값으로 setup 함수를 실행한 다음 새로운 값으로 setup 함수를 실행한다.
  - 컴포넌트가 DOM에서 제거되기 전에 React는 clean up 함수를 한번 더 실행한다.
- `dependencies?`: setup 코드 내에 참조된 모든 반응형 값들의 목록

## Returns

- `undefined`를 반환한다.

## 주의사항

- 서버렌더링 중에는 실행되지 않고 client에서만 실행되는 로직으로 useInsertionEffect가 실행될 때는 DOM이 업데이트 되지 않은 상태이기때문에 refs를 사용할 수 없다.
- 내부 로직에서 state 업데이트를 할 수 없다.

<br>

## CSS-in-JS 라이브러리에서 동적 스타일 삽입하는 방법

### CSS-in-JS 의 접근 방식

1. 컴파일러를 사용하여 CSS파일을 생성

   ```css
   .success {
     color: green;
   }
   ```

   ```js
   <button className="success">버튼</button>
   ```

   - 정적 스타일의 경우 CSS 파일로

1. 인라인 스타일로 적용

   ```js
   <div style={{ opacity: 1 }}>
   ```

   - 동적 스타일의 경우 inline style로 적용하는 것이 좋음

1. 런타임에 `<style>` 태그 삽입
   - _Run time에 스타일을 주입하면 브라우저에서 스타일 적용을 위한 계산을 계속 해야하므로 권장하지 않는 방법이다!_
   - React cycle의 잘못된 시점에서 style을 주입하면 속도 저하가 발생할 수 있다.
   - 런타임에 주입을 해야되는 상황이 오면 useInsertionEffect를 사용하여 DOM 변경 전에 스타일을 주입하면서, 시점으로 인한 속도 저하를 방지할 수 있다.

### `useInsertionEffect` VS `useLayoutEffect`

- 렌더링중이거나 Non-blocking update 중에 스타일을 주입하면 브라우저는 컴포넌트 트리를 렌더링하는 동안 모든 프레임 스타일 계산 과정이 필요하다.
- useInsertionEffect는 다른 Effect 컴포넌트에서 실행될 때 style 태그가 이미 주입되어있고 이전 오래된 값으로 레이아웃 계산을 잘못하지 않기때문에 다른 Effect 보다 더 좋은 방법이 될 수 있다.


