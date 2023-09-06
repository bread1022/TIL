# useReducer
> 최상위 컴포넌트에 호출하여
>
> reducer로 state를 관리할 수 있게하는 React Hook

```js
const [state, dispatch] = useReducer(reducer, initialArg, init?);
```

### Parameters

- **`reducer function`**: state와 action을 매개변수로 받아 새로운 state를 반환하는 `reducer` 함수
  - **`state`**: 현재 state
  - **`action`**: dispatch로 전달된 객체
    - `type`: action의 타입
    - `payload`: action의 데이터
  - `return`: 다음 state
- `initialArg`: 초기 state
- `init fuction?`: 지정할 경우, 초기 state를 계산하여 반환하는 함수로 init(initailArg)를 호출한 결과가 초기 state로 설정된다.
  - `initialArg`: `initialArg` 매개변수의 값
  - `return`: 초기 state


### Return

2개의 요소를 가진 배열을 반환한다.
- **`current state`**(`현재state`)
  - 첫번째 렌더링에는 `init(initialArg)`의 반환값으로 설정되고 (init이 없는 경우, initialArg)  
  - 이후 렌더링에는 reducer의 반환값으로 설정된다.
- state를 업데이트하고 리렌더링을 트리거하는 **`dispatch function`**(`디스패치함수`)
  - 사용자가 수행한 작업을 표시하는 객체(action)을 사용하여 호출하고, 이를 통해 리렌더링을 트리거한다.
  - 리액트는 현재state와 action을 reducer함수에 전달하고, 다음 state를 계산하여 반환한다. 그럼 이 값을 다음 state로 저장하여 리렌더링하는 것이다.


### 주의사항

- `useReducer`는 Hook이기 때문에 컴포넌트의 최상위 레벨이나 커스텀훅에서만 호출할 수 있고, 조건문이나 반복문 안에서는 호출할 수 없다.  
  -> 필요할 땐 새컴포넌트를 추출하고 state를 옮겨야한다.
- Strict Mode에서 React는 의도치않은 부작용을 찾기위해 reducer와 초기화 함수를 두번 호출한다. 따라서 순수함수로 작성해야한다.

<br>


## dispatch function

- `useReducer`가 반환하는 함수
- 유일한 인수로 **`action`** 을 받고 state를 업데이트하여 리렌더링을 트리거한다.
- 리액트는 reducer함수에 `현재 state`와 `dispatch한 action`을 전달하여 다음 state를 계산한다.
- 매개변수
  - **`action`** : state를 업데이트하는데 필요한 정보를 가진 객체로 사용자가 수행한 작업을 나타낸다.
    - `type`: action을 식별할 수 있는 타입
    - `payload`: action의 데이터
- 반환값은 따로 없다.
- dispatch함수는 다음 state를 위한 계산을 하기때문에 함수를 호출했을 때 state 변수를 읽으면 호출 전 화면에 있던 이전 state가 남아있다!
- 새로 업데이트된 state를 `Object.is`메서드로 비교했을 때 `current state`와 같다면 React는 리렌더링하지않는다.
- React는 state 업데이트를 일괄처리한다. 모든 이벤트핸들러가 실행되고 set함수를 호출한 후 화면을 업데이트하기때문에 단일 이벤트에 의해 여러번 리렌더링되는 일을 방지할 수 있다.
  - DOM 접근을 위해 React가 화면을 더 일찍 업데이트해야되는 경우 `flushSync`를 사용할 수 있다.


<br>


#### useReduer를 컴포넌트에서 사용하는 방법

```js
import { useReducer } from 'react';

// 2️⃣ 매개변수로 state, action을 전달받아 다음 state를 반환하는 리듀서함수
function reducer(state, action) {
  switch (action.type) {
    case 'incremented_age': {
      return {
        name: state.name,
        age: state.age + 1
      };
    }
    case 'changed_name': {
      return {
        name: action.nextName,
        age: state.age
      };
    }
  }
  throw Error('Unknown action: ' + action.type);
}

// 1️⃣ 초기 state, 관련이 있는 항목들을 객체로 관리
const initialState = { name: 'Taylor', age: 42 };

export default function Form() {
  // 3️⃣ useReducer를 호출하여 작성해둔 reducer함수와 초기 state를 전달
  const [state, dispatch] = useReducer(reducer, initialState);

  function handleButtonClick() {
    // 4️⃣ dispatch함수를 호출하여 action을 전달
    dispatch({ type: 'incremented_age' });
  }

  function handleInputChange(e) {
    // 4️⃣ dispatch함수를 호출하여 action을 전달 (다음 state계산에 데이터가 필요한 경우, payload도 함께 전달)
    dispatch({
      type: 'changed_name',
      nextName: e.target.value
    });
  }

  return (
    <>
      <input
        value={state.name}
        onChange={handleInputChange}
      />
      <button onClick={handleButtonClick}>
        Increment age
      </button>
      <p>Hello, {state.name}. You are {state.age}.</p>
    </>
  );
}
```
- 컴포넌트의 최상위 레벨에서 `useReducer`를 호출하여 reducer로 state를 관리한다.
- 관리해야하는 state가 여러개이면서 관련있는 경우 `useReducer`를 사용하는 것이 좋다.
- `useReducer`는 이벤트 핸들러의 state업데이트 로직을 컴포넌트 외부로 분리할 수 있게한다.
- `useReducer`는 `current state`와 `dispatch function`을 배열로 반환한다.


### reducer function 작성 방법

1. 매개변수로 state와 action을 전달받고
2. 다음 state를 반환하는 로직을 작성한다.
3. action type에 따라 수행해야되는 로직이 다르기때문에 일반적으로 switch문으로 작성한다.
4. 이때 action에는 다음 state를 계산하기 위한 필수 정보를 포함하고 있어야한다.  
   액션을 식별할 수 있는 `type`과 다음 state를 계산할 때 필요한 데이터를 포함하는 `payload`가 필요하다.
    ```js
    function Form() {
      const [state, dispatch] = useReducer(reducer, { name: 'Taylor', age: 42 });

      function handleButtonClick() {
        dispatch({ type: 'incremented_age' });
        // ✅ 액션 타입은 컴포넌트에 로컬로 지정됨
      }

      function handleInputChange(e) {
        // ✅ 디스패치함수에 액션에 대한 타입과 페이로드를 전달
        dispatch({
          type: 'changed_name',
          nextName: e.target.value
        });
      }
      // ...
    }
    ```
5. reducer 함수의 type 내부 로직에서 state를 업데이트할 때 state를 바로 변이하면 안되고 reducer에서 새로운 객체를 반환하도록 해야한다.
    ```js
    function reducer(state, action) {
      switch (action.type) {
        case 'incremented_age': {
          // ❌ 절대로 state를 바로 변이하면 안됨!
          state.age = state.age + 1;
          return state;
        }
        case 'incremented_age': {
          // ✅ 새로운 객체를 반환하여 state를 업데이트해야한다! only 순수함수!!
          return {
            ...state,
            age: state.age + 1
          };
        }
      }
    }
    ```


### 렌더링마다 초기 state 재생성 방지를 위한 초기화 함수

```js
function createInitialState(username) {
  // ...
}
```
```js
function TodoList({ username }) {
  // ❌ 초기 state를 계산하는 함수를 두번째 인수로 전달하면 렌더링마다 호출됨
  // (함수를 전달하는 게 아니라 계속해서 함수를 호출해서 계산해야되기때문!!!)
  const [state, dispatch] = useReducer(reducer, createInitialState(username));
  // ...
}
```
```js
function TodoList({ username }) {
  // 🌟 두번째 인수에는 state만 전달하고, 세번째 인수로 초기화 계산 함수를 전달 !!
  const [state, dispatch] = useReducer(reducer, username, createInitialState);
  // 🌟 초기화 함수에 아무런 정보가 필요하지 않을땐 두번째 인자로 null을 전달
  const [state, dispatch] = useReducer(reducer, null, createInitial);
  // ...
}
```

- `useReducer` Hook은 초기 state를 한번만 계산한다.
- 두번째 인수로 전달하는 초기 state를 함수를 통해서 계산해야되게끔 전달할 경우 초기 렌더링뿐만아니라 모든 렌더링에서 호출되어 계산하게 된다.(초기화 함수를 호출하는 것이 아니라 함수를 호출해서 계산해야되기때문!! 매우 비효율적)  
  -> 이를 방지하기 위해서는 `useReducer`의 세번째 인수로 초기화 함수를 전달해야한다.

<br>


#### action을 dispatch했는데도 이전 state가 읽히는 경우

- 코드를 실행 중일 땐 dispatch함수를 호출해도 state가 바로 업데이트되지 않는다.
- React의 state는 스냅샷처럼 동작하기 때문에 state를 업데이트하면 업데이트된 state로 리렌더링을 트리거하지만 실행중인 이벤트핸들러 내부의 변수에는 영향을 미치지 않기 때문이다.
- dispatch함수를 호출하면 React는 다음 state를 계산하는 것이므로 내부 로직에서 읽히는 state는 호출 이전 state이다. 다음 state를 확인하려면 직접 reducer를 호출하고 수동으로 계산해서 읽어야된다.
    ```js
    const action = { type: 'incremented_age' };
    dispatch(action);

    const nextState = reducer(state, action);
    console.log(state);     // { age: 42 }
    console.log(nextState); // { age: 43 }
    ```


#### 🌟 action을 dispatch했는데도 화면에 업데이트 되지 않는 경우

- 보통 객체나 배열 형태의 state를 변이했을 때 주로 발생하는 현상으로,  
  이전 state와 다음 state가 같을 땐 React에서 업데이트를 무시한다.
    ```js
    function reducer (state, action) {
      switch (action.type) {
        case 'increase_age': {
          // ❌ state를 직접 변이하면 안됨!
          state.age++;
          return state;
        }
        case 'chage_name': {
          // ❌ state를 직접 변이하면 안됨!
          state.name = action.name;
          return state;
        }
      }
    }
    ```
  - 기존 state 객체를 변경한 것이기때문에 `Object.is`메서드로 비교했을 때 같은 객체이다.
  - 따라서 업데이트를 명시하기 위해서는 항상 *새로운 객체를 반환해야 다르다*고 판단한다!
    ```js
    function reducer(state, action) {
      switch (action.type) {
        case 'incremented_age': {
          // ✅ 직접 변이하는 대신 항상 교체를 해야됨!! (새로운 객체를 반환)
          return {
            ...state,
            age: state.age + 1
          };
        }
        case 'changed_name': {
          // ✅ 새로운 객체를 반환
          return {
            ...state,
            name: action.nextName
          };
        }
        // ...
      }
    }
    ```


#### dispatch했을 때 reducer state의 일부분이 undefined가 되는 경우

- 새로운 객체를 반환할 때 기존 field를 모두 복사한 뒤 반환해야한다.
- 안그러면 다음 state에는 새로 교체한 state만 남고 나머지 feild가 undefined될 수 있다.

```js
function reducer(state, action) {
  switch (action.type) {
    case 'increase_age': {
      return {
        // ❌ age만 업데이트되고 나머지 field가 없어짐..!!!!
        age: state.age + 1
      };
    }
    case 'changed_name': {
      return {
        ...state, // ✅ 업데이트하는 field외의 영역을 복사해야됨!
        name: action.nextName
      };
    }
  }
}
```


#### dispatch했을 때 모든 state가 undefined가 되는 경우

- action이나 state에 문제발생하여 다음 state를 업데이트할 수 없는 경우에는 reducer 함수의 switch 외부에서 오류를 발생시키도록 해야한다.
- 예기치 못한 state를 업데이트하지 않도록 하기 위함이다.
    ```js
    function reducer(state, action) {
      switch (action.type) {
        case 'incremented_age': {
          // ...
        }
        case 'edited_name': {
          // ...
        }
      }
      // ✅ type이 일치하지 않는 경우 switch문 외부에서 오류 발생시키기
      throw Error('Unknown action: ' + action.type);
    }
    ```


#### "Too many re-renders" 오류 발생한 경우

- 컴포넌트 렌더링 - dispatch (state 업데이트) - 컴포넌트 렌더링 - dispatch (state 업데이트) - ...  
  이런식으로 무한히 반복되는 경우 발생한다.
- 이벤트핸들러를 지정하는 과정에서 실수가 없었는지 확인하고,  
  추가적으로 원인을 찾기위해서는 JS stack을 살펴보면서 오류의 원인이 되는 dispatch 함수 호출을 찾아야한다.
    ```js
    <button onClick={handleClick()}>Click<button> // ❌ 렌더링마다 핸들러가 호출됨!!
    <button onClick={handleClick}>Click<button> // ✅ 핸들러를 지정하는 방법
    <button onClick={e => handleClick(e)}>Click<button> // ✅ 핸들러를 지정하는 방법
    ```


#### reducer, 초기화 함수가 두번 실행되는 경우

- Strict Mode에 의해서 함수 호출이 두번 될 수 있어 reducer 함수가 순수함수가 아니면 예기치 못한 오류가 발생할 수 있다.
- reducer function, 컴포넌트, 초기화 함수는 반.드.시 순수해야한다! (이벤트핸들러는 아님)
    ```js
    function reducer(state, action) {
      switch (action.type) {
        case 'added_todo': {
          // ❌ 배열을 변이하면 안됨! No push!
          state.todos.push({ id: nextId++, text: action.text });
          return state;
        }
        case 'added_todo': {
          return {
            // ✅ 기존 객체를 복사하여 새로운 객체를 반환
            ...state,
            todos: [
              ...state.todos,
              { id: nextId++, text: action.text }
            ]
          };
        }
      }
    }
    ```

    ```js
    function reducer(state, action) {
      switch (action.type) {
        case 'added_todo': {
          // ✅ Correct: replacing with new state
          return {
            ...state,
            todos: [
              ...state.todos,
              { id: nextId++, text: action.text }
            ]
          };
        }
        // ...
      }
    }
    ```