# Element Rendering

index.html 파일 내 `<div id="root"></div>` 루트 노드는 일반적으로 하나 존재한다. React element를 렌더링 하기 위해서는 DOM element를 `ReactDOM.createRoot()`에 전달한다음 React element를 `root.render()`에 전달해야한다.

```js
const root = ReactDOM.createRoot(
  document.getElementById('root')
);
const element = <h1>Hello, world</h1>;
root.render(element);
```

JS로 UI를 업데이트하는 방법은 new element 를 생성하고 이를 `root.render()`에 전달하는 것이다.

React DOM은 해당 element와 그 자식 element를 이전의 element와 비교하여 원하는 상태에 필요한 경우에만 DOM을 업데이트합니다.

# Components

# Props

