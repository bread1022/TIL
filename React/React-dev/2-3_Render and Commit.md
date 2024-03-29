# Render and Commit

## Step 1: Trigger a render

#### 컴포넌트의 렌더링 2가지

1. 컴포넌트의 첫 렌더링될 때
2. 컴포넌트의 `state` 가 업데이트 될 때

## Step 2: React renders your components

#### 렌더링이란?
> 리액트에서 컴포넌트를 호출하는 것

- 첫번째 렌더링에서 리액트는 `root 컴포넌트`를 호출한다.
- 첫번째를 제외한 렌더링에서 리액트는 `state`업데이트에 의해 렌더링이 트리거된 컴포넌트를 호출한다.
- 재귀적으로 업데이트된 컴포넌트가 다른 컴포넌트를 반환하면 그 다음 컴포넌트를 반환하는 과정을 중첩된 컴포넌트가 더 이상 없을 때까지 반복한다.
- 렌더링은 순수함수로 생성된 컴포넌트에 의해 트리거되어야한다.
  - 동일한 입력엔 항상 동일한 JSX출력을 반환해야한다.
  - 렌더링 이전 존재했던 `state`를 변경해서는 안된다.

## Step 3: React commits changes to the DOM

- 리액트는 컴포넌트 렌더링 후 렌더링 간에 차이가 있는 경우에만 DOM을 수정한다.
  - 부모 컴포넌트로 부터 전달된 props로 다시 렌더링하는 경우 전달 된 경우 props로 변경된 값만 업데이트하고 차이가 없는 경우에는 DOM을 수정하지 않는다.
  - 렌더링 결과가 이전과 같으면 리액트는 DOM을 수정하지 않는다.
- 초기 렌더링의 경우, `appendChild()` DOM API를 사용하여 생성한 DOM 노드를 화면에 표시한다.
- 리렌더링의 경우, DOM을 업데이트된 렌더링 출력과 일치하도록한다.
- 렌더링이 완료되면 리액트가 DOM을 업데이트한 후 **브라우저는 화면을 `painting`한다.**


### React App의 화면업데이트 3단계

**1. Trigger**
**2. Render**
**3. Commit**