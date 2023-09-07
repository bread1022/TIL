# createContext
> 컴포넌트 외부에서 호출하여
>
> Context 를 생성할 수 있는 React API


```js
import { createContext } from 'react';

const My = createContext(defaultValue);
```


## Parameters

- `defaultValue`: `useContext`를 사용하였을 때 수신되는 기본값
  - `useContext`를 호출하여 context를 읽는 컴포넌트에서 상위 트리를 Provider로 감싸지 않아 value를 탐색할 수 없는 경우 이 값을 대체하여 사용한다.
  - 절대 변경되지 않는 값이다. (컴포넌트에 동적으로 값을 제공해야될 때는 useState와 Provider를 사용해야함)
  - 렌더링이 안되는 에러를 방지할 수도 있지만 명시적으로 `null`을 넣어서 에러처리를 유도할 수도 있다.


## return

- createContext로 생성된 `Context 객체`를 반환한다.
  - 어떠한 정보도 포함하지 않는 객체
  - 다른 컴포넌트가 읽거나 제공할 수 있는 컨텍스트를 나타낸다.
  - property
    - `Context.Provider`: 컴포넌트에 컨텍스트 값을 제공할 수 있다.
    - (`Context.Consumer`: 컨텍스트 값을 읽을 수 있는 속성이지만 거의 사용하지 않는 방법)



### SomeContext.Provider

- 컴포넌트를 Context.Provider로 감싸고 props로 값을 전달하면, 내부 모든 컴포넌트에서 이 Context의 값을 읽을 수 있다.

  ```js
  function App() {
    const [theme, setTheme] = useState('light');
    // ...
    return (
      <ThemeContext.Provider value={theme}>
        <Page />
      </ThemeContext.Provider>
    );
  }
  ```
- Provider의 props: 해당 Provider 내에서 context를 읽는 모든 하위 컴포넌트에 전달하려는 값
  - 내부 컴포넌트에서 `useContext`를 호출했을 때 가장 가까운 위치의 `Context.Provider`의 value를 탐색하여 사용한다.


### context 생성

```js
import { createContext } from 'react';

const ThemeContext = createContext('light');
const AuthContext = createContext(null);

function App() {
  const [theme, setTheme] = useState('dark');
  const [currentUser, setCurrentUser] = useState({ name: 'Taylor' });

  // ...

  return (
    <ThemeContext.Provider value={theme}>
      <AuthContext.Provider value={currentUser}>
        <Page /> // 🌟 Provider로 감싸진 컴포넌트에서 Context를 사용할 수 있고,
        // 이 Context가 변경될 경우 Context를 읽는 컴포넌트도 리렌더링됨
      </AuthContext.Provider>
    </ThemeContext.Provider>
  );
}
```