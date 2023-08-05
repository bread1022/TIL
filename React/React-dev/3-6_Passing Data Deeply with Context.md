# Passing Data Deeply with Context

부모컴포넌트가 props를 명시적으로 전달하지 않고 하위에 있는 모든 컴포넌트에서 props를 사용할 수 있게 해주는 것이 Context이다.


## The problem with passing props
> props 전달의 문제

- props는 UI 트리를 통해 필요로하는 데이터를 컴포넌트에 명시적으로 전달하는데 사용된다.
- UI 트리의 depth가 깊거나, 컴포넌트가 많아질 수록 props 전달이 복잡해진다.
- “Prop drilling”이 발생한다.


## Context: an alternative to passing props
> Context: props 전달의 대안

- Context를 사용하면 상위 컴포넌트의 데이터를 하위 모든 컴포넌트에 제공할 수 있다.

<p align="center"><img src="https://react-ko.dev/_next/image?url=%2Fimages%2Fdocs%2Fdiagrams%2Fpassing_data_context_far.png&w=640&q=75" width="400px"/></p>


#### Step 1: Create the context

```jsx
import { createContext } from 'react';

export const LevelContext = createContext(1);
```
- `createContext(defaultValue)`의 **인수에 기본값**을 전달하고 Context를 만든다.


#### Step 2: Use the context

```jsx
import { useContext } from 'react';
```
- React에서 context를 사용하기 위해 `useContext()` Hook을 사용한다.

#### Step 3: Provide the context

```jsx
export default function Section({ level, children }) {
  return (
    <LevelContext.Provider value={level}>
      {children}
    </LevelContext.Provider>
  );
}
```
- `Context Provider`로 감싸서 context를 제공하면  
  컴포넌트는 상위의 UI 트리에서 가장 가까운 `Context.Provider`의 value를 사용한다. (level prop이 필요없어짐)


#### `useContext()` Hook
- React 컴포넌트 최상단에서만 Hook을 호출할 수 있다.
- `createContext()`로 만든 Context 객체를 인수로 전달한다.
- `useContext(Context객체)`을 호출하면 depth에 상관없이 모든 하위 컴포넌트에서 접근이 가능하다.
- Context를 생성한 컴포넌트 하위 컴포넌트를 <Context.Provider value={value}>로 감싸서 value를 전달한다.
- context를 사용하기에 앞서 먼저 Props로 데이터 흐름을 명확하게 하고,  
  컴포넌트를 추출하고 JSX를 children으로 전달하면서 props를 전달하고,  
  바로 하위컴포넌트가 아니라 여러 레이어를 거쳐 전달한다면 `<Layout><Post posts={posts} /></Layout>` 같이 children을 prop으로 사용하도록 만들어본다. 이 두가지 접근 방식이 적합하지 않을 때 context를 사용한다.
- 주로 Theme, Account, Route, State 관리 등에 많이 사용된다.
  - Theme: App 상단에 context provider 배치하여 하위 컴포넌트에서 theme을 사용할 수 있게 한다.
  - Current Account : 로그인 사용자 정보를 많은 컴포넌트에서 사용하게 될 때 context를 사용하면 트리 어느 곳에서 편리하게 사용자 정보를 읽을 수 있다.
  - State Management : reducer를 함께 사용하면서 복잡한 state를 관리하고 멀리 떨어진 컴포넌트에 state를 전달한다.