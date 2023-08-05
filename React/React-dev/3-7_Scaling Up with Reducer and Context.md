# Scaling Up with Reducer and Context


## Combining a reducer with context

1. Context를 생성한다.
2. state와 dispatch 함수를 context에 넣는다.
3. UI 트리 내부에서 context를 사용한다.


### Step 1: Create the context

```jsx
import { createContext } from 'react';

const StateContext = createContext(null);
const StateDispatchContext = createContext(null);
```
- 상위에서 선언한 state와 dispatch함수를 트리를 통해 하위 컴포넌트에 전달하기 위해서 두개의 별개의 context를 생성한다.


### Step 2: Put state and dispatch into context

```jsx
const [state, dispatch] = useReducer(reducer, initialState);
```
- `useReducer()` Hook은 현재 state와 dispatch 함수를 반환한다.

```jsx
export default function StateApp() {
  const [state, dispatch] = useReducer(stateReducer, initialState);
  // ...
  return (
    <StateContext.Provider value={state}>
      <StateDispatchContext.Provider value={dispatch}>
        ...
      </StateDispatchContext.Provider>
    </StateContext.Provider>
  );
}
```
- context를 사용하고자 하는 컴포넌트에서 import한 후,  
  하위 전체 트리를 `StateContext.Provider`로 감싼다.
- `value` prop으로 state와 dispatch 함수를 전달한다.


### Step 3: Use context anywhere in the tree


```jsx
import { createContext } from 'react';

const StateContext = createContext(null);
const StateDispatchContext = createContext(null);

export default function StateApp() {
  const [state, dispatch] = useReducer(stateReducer, initialState);
  // ...
  return (
    <StateContext.Provider value={state}>
      <StateDispatchContext.Provider value={dispatch}>
        ...
      </StateDispatchContext.Provider>
    </StateContext.Provider>
  );
}
```
- context를 통해 전달한 value는 필요한 컴포넌트에서 `useContext()` Hook을 이용하여 사용할 수 있다.
  ```jsx
  import { useState, useContext } from 'react';
  import { StateContext } from './StateContext.js';

  export default function StateList() {
    const states = useContext(StateContext);
    return (
      <ul>
        {states.map(state => (
          <li key={state.id}>
            <State state={state} />
          </li>
        ))}
      </ul>
    );
  }
  ```
- 최상위 컴포넌트에서는 `useReducer`로 state와 dispatch함수를 생성하고,  
  하위 컴포넌트에서는 `useContext`로 state와 dispatch함수를 사용한다.

## Moving all wiring into a single file

- Reducer가 복잡해지면 `StateProvider` 컴포넌트를 선언하여 사용할 수도 있다.
  ```jsx
  export function StateProvider({ children }) {
    const [state, dispatch] = useReducer(stateReducer, initialState);

    return (
      <StateContext.Provider value={state}>
        <StateDispatchContext.Provider value={dispatch}>
          {children}
        </StateDispatchContext.Provider>
      </StateContext.Provider>
    );
  }
  ```
  ```jsx
  import { StateProvider } from './StateContext.js';

  export default function StateApp() {
    return (
      <StateProvider>
        <AddState />
        <StateList />
      </StateProvider>
    );
  }

  export function useState() {
    return useContext(StateContext); // context를 사용하는 함수도 생성해서 export할 수 있음
  }

  export function useStateDispatch() {
    return useContext(StateDispatchContext);
  }
  ```
  - `useContext`Hook을 사용하는 함수들도 export할 수 있다. (context를 분리하고 함수에 로직을 추가하는 게 쉬워지는 장점)
    ```jsx
    const state = useState(); // 함수를 호출하여 바로 사용할 수 있음
    ```