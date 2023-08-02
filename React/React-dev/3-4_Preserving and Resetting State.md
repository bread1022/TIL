# Preserving and Resetting State


## The UI tree
> UI 트리

컴포넌트에서 React는 JSX로부터 **UI트리**를 생성하고, React DOM은 해당 UI트리와 일치하도록 브라우저 DOM element를 업데이트한다.


## State is tied to a position in the tree
> state는 트리의 한 위치에 묶입니다

- 보통 컴포넌트에 `state`를 생성할 때 내부에 존재한다고 생각할 수 있지만  
  실제론 (컴포넌트 밖) React 내부 별도의 공간에 모든 컴포넌트들의 `state`를 모아서 관리한다.  
  (`state`를 위한 별도의 공간이 존재함)
- React는 UI트리에서 해당 컴포넌트 위치에 따라 보유하고 있는 각 `state`를 연결해주는 것이다.
- React는 같은 컴포넌트를 같은 위치에 렌더링하는 한 그 `state`를 유지하고  
  컴포넌트가 제거되거나 같은 위치에 다른 컴포넌트가 렌더링되면 React는 해당 `state`를 삭제한다.


## Same component at the same position preserves state
> 동일한 위치의 동일한 컴포넌트는 state를 유지합니다

- 동일한 위치란 JSX마크업이 아니라 UI트리에서의 위치가 동일하다는 것이다.  
  (JSX태그에 보관되는게 아니라 JSX를 넣은 트리의 위치와 연관되어 있음)
- 같은 위치에 있는 같은 컴포넌트는 `state`를 업데이트하여도  
  컴포넌트를 재설정하지않고(이전값을 리셋하지x) `state`만 변경되는 것이다.


## Different components at the same position reset state
> 동일한 위치의 다른 컴포넌트는 state를 초기화합니다

- 같은 위치에 서로 다른 컴포넌트를 위치시키고 삼항연산자같은 로직으로 컴포넌트들을 전환하게 되면  
  DOM에서 제거되고 그 전체 하위 트리의 `state`도 재설정된다. (트리에서 컴포넌트를 제거할 때 `state`도 함께 제거됨)
- 리렌더링사이에 `state`를 유지하기 위해선 트리의 구조도 일치해야한다.


## Resetting state at the same position
> 동일한 위치에서 state 재설정하기

- React는 같은 위치에 있는 동안 컴포넌트의 `state`를 유지한다.
  ```jsx
  {isPlayerA ? (
    <Counter person="Taylor" />
  ) : (
    <Counter person="Sarah" />
  )}
  // 같은 위치에 있지만 내용상 별도의 Counter컴포넌트가 필요함
  ```
  - 전환하는 로직이 필요할 때 `state`를 재설정하는 방법
    1. 컴포넌트를 **다른 위치에 렌더링**시킨다.
    2. 각 컴포넌트에 **`key` prop을 추가**하여 명시적인 identity를 부여한다.

### Option 1. Rendering a component in different positions
> 컴포넌트를 다른 위치에 렌더링하기

```jsx
{isPlayerA && <Counter person="Taylor" />}
{!isPlayerA && <Counter person="Sarah" />}
```
<p align="center"><img src="https://github.com/bread1022/TIL/assets/107349637/47ce66af-65c1-4007-b637-ee04f067d6ea" /></p>

- 다른 위치에 독립적으로 렌더링을 하게 되면 각 `state`는 독립적으로 동작한다. (유지x)
- DOM에서 제거될 때마다 `state`도 함께 제거되어 초기화된다.
- 이 방법은 독립적이고 적은 수의 컴포넌트를 렌더링할 때만 편리한 방법이다. (2개정도~)


### Option 2. Resetting state with a key
> key로 state 재설정하기

```jsx
{isPlayerA ? (
  // key를 추가함으로써 다른 컴포넌트로 인식하게 됨
  <Counter key="Taylor" person="Taylor" />
) : (
  <Counter key="Sarah" person="Sarah" />
)}
```
- `key`를 사용하여 React가 컴포넌트를 구분할 수 있도록하면  
  JSX에서 같은 위치에 있더라도 `state`를 공유하지 않게 된다. (서로 다른 key를 부여했기 때문)
- React에서 `key`자체 위치를 사용하도록 지시하기때문에 JSX에서 같은 위치에 렌더링을 하더라도  
  React관점에서는 서로 다른 컴포넌트로 인식한다.
- `key`는 전역으로 고유한 값은 아니고 부모컴포넌트 내에서만 위치만 지정하는 고유의 값이다.



## Resetting a form with a key
> 키로 form 재설정하기


- form 데이터를 다룰 때 key로 `state`를 재설정하면  
  form 컴포넌트가 하위 트리의 모든 `state`를 포함하여 처음부터 다시 생성하고,
  React는 DOM element를 재사용하는 대신 다시 생성한다.

### Preserving state for removed components
> 제거된 컴포넌트에 대한 state 보존

- 간단한 UI컴포넌트 구성일 경우 CSS를 이용하여 보관할 수 있지만,
- 트리 복잡한 DOM노드를 포함한 경우에는 부모컴포넌트에 state를 끌어올려서 보관할 수도 있다.
- 또한 localStorage를 이용하여 보관할 수도 있다.


-------

### 과제

```jsx

export default function App() {
  const [reverse, setReverse] = useState(false);
  let checkbox = (
    <label>
      <input
        type="checkbox"
        checked={reverse}
        onChange={e => setReverse(e.target.checked)}
      />
      Reverse order
    </label>
  );
  if (reverse) {
    return (
      <>
        <Field label="Last name" />
        <Field label="First name" />
        {checkbox}
      </>
    );
  } else {
    return (
      <>
        <Field label="First name" />
        <Field label="Last name" />
        {checkbox}
      </>
    );
  }
}
```
- reverse 값이 변경되면 리렌더링 동안 컴포넌트의 state가 올바르게 유지되지 않는다.

- reverse 값이 변경되어도 리렌더링 동안 컴포넌트의 state가 key에 맞게 유지된다!!!
  ```jsx
  export default function App() {
    const [reverse, setReverse] = useState(false);
    let checkbox = (
      <label>
        <input
          type="checkbox"
          checked={reverse}
          onChange={e => setReverse(e.target.checked)}
        />
        Reverse order
      </label>
    );
    if (reverse) {
      return (
        <>
          <Field key="lastName" label="Last name" /> // key를 추가하여 state를 일치시킴
          <Field key="firstName" label="First name" />
          {checkbox}
        </>
      );
    } else {
      return (
        <>
          <Field key="firstName" label="First name" />
          <Field key="lastName" label="Last name" />
          {checkbox}
        </>
      );
    }
  }
  ```
  ```jsx
  export default function App() {
    const [reverse, setReverse] = useState(false);
    let checkbox = (
      <label>
        <input
          type="checkbox"
          checked={reverse}
          onChange={e => setReverse(e.target.checked)}
        />
        Reverse order
      </label>
    );

      const order = (reverse) => reverse ? 'First name' : 'Last name';
      return (
        <>
          <Field label={order(reverse)} key={order(reverse)} /> 
          <Field label={order(!reverse)} key={order(!reverse)} />
          {checkbox}
        </>
      )
  }
  ```


-------

