# createPortal

> 일부 자식 요소를 DOM의 다른 영역에 렌더링할 수 있는 API

```js
createPortal(children, DOMNode, key?)
```

포탈을 사용하면 지정한 DOM 위치에 컴포넌트의 일부 children 요소를 렌더링할 수 있다.  
Portal을 사용하면 컴포넌트의 일부 요소가 어디있든 DOM 노드를 지정하여 물리적 배치를 변경할 수 있다.  
포탈을 사용하면 배치만 변경하고 부모 트리가 제공하는 Context에 접근할 수 있으며 이벤트는 React트리에 따라 자식에서 부모로 버블링된다.

또한 리액트가 정적 페이지, 서버 페이지의 일부분일 때  
root를 하나만 사용하는 것과 비교하여 포탈을 이용하면 App의 state를 공유하면서 단일 React 트리로 처리할 수 있다는 장점을 가진다!


## Parameters

```js
import { createPortal } from 'react-dom';

function MyComponent() {
  return (
    <div>
      <p>Hello</p>
      {createPortal(
        <p>포탈로 생성한 요소</p>,
        document.body // 🌟 이미 존재하는 DOM 노드를 지정해야함!
      )}
    </div>
  );
}
```
```js
<body>
  <div id="root">
    ...
      <div>
        <p>Hello</p>
      </div>
    ...
  </div>
  <p>포탈로 생성한 요소</p> //✅ document.body 안에 생성됨
</body>
```


- `children`: 포탈로 생성할 React 요소
- `DOMNode`: 포탈을 표시할 DOM 노드 (**이미 존재하고 있는 DOM 노드**여야 함)
- `key`?: 포탈의 키

## Returns

- JSX에 포함되거나 React 컴포넌트에서 반환될 수 있는 React 노드를 반환한다.

## 주의사항

- Portal Event는 React 트리를 따라 전파된다. (DOM 트리 형태로 전파❌)
- 포탈이 `<div onClick>`으로 감싸져있을 때 포탈 내부를 클릭하면 감싸져있는 `onClick`이벤트가 실행된다.  
  만약 이런 문제가 발생하면 포탈 내부 이벤트 전파를 중지하거나 포탈을 React 트리 위로 옮겨야한다.
