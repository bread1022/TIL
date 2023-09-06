# useContext
> context를 읽고 구독할 수 있게 하는 Hook

```js
const value = useContext(cutomContext);
```

## Parameters

- `context`: `createContext`로 생성한 context
  - `createContext`: 컴포넌트가 제공하거나 읽을 수 있는 컨텍스트를 생성하는 React API
    - 컴포넌트 외부에서 호출하여 컨텍스트를 생성하면 context 객체를 반환한다.
    - 반환된 context 를 읽기위해 `useContext`에 전달한다.


## Return

- `useContext`는 호출하는 컴포넌트에서 트리상 가장 가까운 Context.Provider에 전달된 value 를 반환한다.


## 주의사항

<!--?? 같은 컴포넌트에서 반환된 프로파이더의 영향을 받지 않는다? -->
- 동일한 컴포넌트에서 반환된 provider의 영향을 받지 않는다.
- `useContext`를 호출하는 컴포넌트 위에 `<Context.Provider>`두고 감싸야한다.
- React는 변경된 value를 받는 provider부터 시작해서 해당 context 를 사용하는 자식들에게 전달되어 자동으로 리렌더링된다.
- `memo`로 props가 변경되지 않았을 때 리렌더링을 막더라도 context 값이 변경되어 발생하는 리렌더링은 막을 수 없다.
<!--?? 빌드 시스템이 출력 결과에 중복 모듈을 생성하는 경우(심볼릭 링크를 사용하는 경우 발생할 수 있음) context가 손상될 수 있습니다. context를 통해 무언가를 전달하는 것은 === 비교에 의해 결정되는 것처럼 context를 제공하는 데 사용하는 SomeContext와 context를 읽는 데 사용하는 SomeContext가 정확하게 동일한 객체인 경우에만 작동합니다. -->


<br>


### 컴포넌트 트리 밑으로 데이터를 전달하는 방법

```js
// 1️⃣ context 생성
import { createContext } from 'react';

const ThemeContext = createContext(null);

function MyPage() {
  return (
    // 2️⃣ context가 필요한 컴포넌트 트리를 provider로 감싸기
    // 현재는 value가 정적인 data임 (상태X, 변하지 않는 값임)
    <ThemeContext.Provider value="dark">
      <Form />
    </ThemeContext.Provider>
  );
}
```
```js
import { useContext } from 'react';

function Button () {
  // 3️⃣ context를 사용해야될 때 useContext를 호출하여 context를 읽고 구독
  // ThemeContext.Provider value 'dark'를 읽어옴
  const theme = useContext(ThemeContext);
  const className = 'button-' + theme;
  return (
    <button className={className}>
      {children}
    </button>
  );
}
```

1. `createContext`로 context를 생성하고 컴포넌트 트리에 `<ThemeContext.Provider>`를 사용하여 value를 전달한다.
2. 최상위 레벨의 컴포넌트에서 `useContext`를 호출하고 context를 읽고 구독한다.
3. `useContext`는 전달한 context에 대한 context value를 반환할 때  
  컴포넌트 트리에서 가장 가까운 context provider를 찾은 다음  
  `<ThemeContext.Provider>`의 value를 가져온다.
  - 호출하는 컴포넌트의 내부 Provider와 상관없이 해당 컴포넌트의 상위로 올라 가장 가까운 Provider를 탐색한다!



### 데이터를 전달하여 context를 업데이트하는 방법

```js
function Page () {
  const [theme, setTheme] = useState('dark');

  const handleSwitchTheme = () => setTheme(theme === 'dark' ? 'light' : 'dark');

  return (
    // ✅ value로 state를 전달하면 context를 업데이트할 수 있다.
    <ThemeContext.Provider value={theme}>
      <Form/>
      <Button onClick={handleSwitchTheme}>Switch theme</Button>
    </ThemeContext.Provider>
  )
}
```

- context가 변경되어야하는 경우 state를 사용하여 context를 업데이트한다.
- Provider를 생성하는 컴포넌트에서 state 변수를 선언하고 context value로 state를 전달해야한다!


#### 다중 context 사용하기

#### 객체 state를 context에 전달하는 방법

#### 단일 컴포넌트로 providers 추출하는 방법

#### context와 reducer를 함께 사용하는 방법


<br>


### fallback 기본값 지정하기

- context를 사용하는 컴포넌트가 Provider를 찾지 못할 때  
  `useContext`는 context의 기본값을 반환한다.
- 기본값은 `createContext`로 context 생성할 때 지정한 값이다!
  ```js
  const ThemeContext = createContext(defaultValue);
  ```
  - 기본값은 절대 변경되지 않는 값이다.
  - null값 대신 의미있는 값으로 사용하는 것이 좋은데, 그 이유는 실수로 Provider 없이 일부 컴포넌트를 업데이트하여 렌더링해도 중단되지 않기 때문이다.

```js
import { createContext, useContext, useState } from 'react';

const ThemeContext = createContext('dark'); // ❓ 초기값을 전달했고

export default function MyApp() {
  const [theme, setTheme] = useState('light');
  // ❓ 여기서 전달하는 state도 있는데 중복이 아닌가 ?? 근데 보면 createContext에 전달한 값을 초기값을 받고 있음
  // 다른 값이라고 생각해야되나???
  return (
    <>
      <ThemeContext.Provider value={theme}>
        <Form />
      </ThemeContext.Provider>
      <Button onClick={() => {
        setTheme(theme === 'dark' ? 'light' : 'dark');
      }}>
        Toggle theme
      </Button>
    </>
  )
}

function Form({ children }) {
  return (
    <Panel title="Welcome">
      <Button>Sign up</Button>
      <Button>Log in</Button>
    </Panel>
  );
}

function Panel({ title, children }) {
  const theme = useContext(ThemeContext);
  const className = 'panel-' + theme;
  return (
    <section className={className}>
      <h1>{title}</h1>
      {children}
    </section>
  )
}

function Button({ children, onClick }) {
  const theme = useContext(ThemeContext);
  const className = 'button-' + theme;
  return (
    <button className={className} onClick={onClick}>
      {children}
    </button>
  );
}
```


### 컴포넌트 트리에서 일부에만 context 재정의하는 방법

- 필요한 만큼 provider를 중첩하고 재정의할 수 있다.
    ```js
    import { createContext, useContext } from 'react';

    const ThemeContext = createContext(null);

    export default function MyApp() {
      return (
        <ThemeContext.Provider value="dark">
          <Form />
        </ThemeContext.Provider>
      )
    }

    function Form() {
      return (
        <Panel title="Welcome">
          <Button>Sign up</Button>
          <Button>Log in</Button>
          <ThemeContext.Provider value="light"> // ✅ 필요한 곳에만 재정의하기
            <Footer />
          </ThemeContext.Provider>
        </Panel>
      );
    }
    ```


### 객체, 함수를 전달할 때 리렌더링을 최적화하는 방법: `useCallback`, `useMemo`

```js
function MyApp() {
  const [currentUser, setCurrentUser] = useState(null);

  function login(response) {
    storeCredentials(response.credentials);
    setCurrentUser(response.user);
  }

  return (
    // ❌ 객체, 함수는 컴포넌트가 렌더링될 때마다 새로운 값이라고 간주함
    <AuthContext.Provider value={{ currentUser, login }}>
      <Page />
    </AuthContext.Provider>
  );
}
```

- context를 통해 객체, 함수를 전달하면 context value는 전달한 객체, 함수를 프로퍼티로 갖는 JS 객체가 된다.
- React에서 리렌더링할 때마다 전달된 객체, 함수는 새로운 값으로 간주되어 `useContext`를 호출하는 깊숙한 트리까지 모든 컴포넌트를 리렌더링하게 된다.
- 이를 방지하기 위해서는 context에 전달하는 함수를 `useCallback`, 객체를 `useMemo`로 감싸면 된다. (소규모앱에서는 문제되지 않음..)

    ```js
    import { useCallback, useMemo } from 'react';

    function MyApp() {
      const [currentUser, setCurrentUser] = useState(null);

      // ✅ 전달함수를 useCallback으로 감싼다
      const login = useCallback((response) => {
        storeCredentials(response.credentials);
        setCurrentUser(response.user);
      }, []);

      // ✅ 전달하는 JS 객체를 useMemo로 감싼다
      const contextValue = useMemo(() => ({
        currentUser,
        login
      }), [currentUser, login]);
      // 🌟 currentUser가 변경되는게 아니면 리렌더링을 유발하지 않음

      return (
        <AuthContext.Provider value={contextValue}>
          <Page />
        </AuthContext.Provider>
      );
    }
    ```


#### 기본값이 다른데도 context에서 undefined만 반환할 때

- 트리에 value가 없는 provider가 있는 경우에는 value가 `undefined`로 간주된다.
  - value 지정을 하지 않으면 `value={undefined}`를 전달하는 것과 같음
    ```js
    // ❌ value를 전달하지 않음 => 당연히 에러나겠지..
    // value={undefined}가 전달됨!!
    <ThemeContext.Provider>
      <Button />
    </ThemeContext.Provider>
    ```
- 또는 prop을 value로 하지않은 경우에도 value값을 가져오지 못한다.
    ```js
    // ❌ prop은 다른 이름을 사용할 수 없음!
    // value={theme}으로 전달해야됨!!
    <ThemeContext.Provider theme={theme}>
      <Button />
    </ThemeContext.Provider>
    ```
- `createContext(defaultValue)`할 때 설정해놓은 기본값을 왜 사용하지 않느냐?  
  이 경우는 provider가 상위 트리 컴포넌트에 일치하는 provider가 없을 때만 적용되는 것이기때문에 먼저 감싸져있는 provider에 value가 `undefined`라면 `undefined`가 반환되는 것이다. (상위 컴포넌트를 탐색하고~ 탐색해도 없으면 `defaultValue`를 반환하는 것임)

  <!-- ?? 진짜 그런지 확인해보기 -->