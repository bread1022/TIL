# useState
> 컴포넌트에 state변수를 추가할 수 있게하는 React Hook

```js
const [state, setState] = useState(initialState);
```

### Parameters

- `initialState`를 매개변수로 받는다. 이 인자는 초기 렌더링 이후에는 무시되는 값이다.
- 값의 타입으로는 모든 타입이 가능하다.
  - 함수를 `initialState`로 전달할 경우 이를 초기화 함수로 취급한다.
  - 초기화 함수는 반드시 순수함수이어야 한다.
  - 초기화 함수는 인자를 받지 않아야하며 반드시 값을 반환해야한다.


### Return

2개의 값을 반환한다.
- **`current state`**(`현재state`)
- state를 업데이트하고 리렌더링을 트리거하는 **`set function`**(`설정자함수`)


### 주의사항

- `useState`는 Hook이기 때문에 컴포넌트의 최상위 레벨이나 커스텀훅에서만 호출할 수 있고, 조건문이나 반복문 안에서는 호출할 수 없다.
- Strict Mode에서 React는 의도치않은 부작용을 찾기위해 초기화 함수를 두번 호출한다.


## set function

- `setState`는 새로운 state를 받아 컴포넌트를 리렌더링할 수 있다.
- 매개변수로 `next state`를 전달하거나 `prev state`를 계산하여 다음 state를 반환하는 `updater function`을 전달할 수도 있다.
  - `updater function`은 순수함수 이어야한다.
  - 인수로 대기중인 state를 사용하고, 다음 state를 반환한다.
  - 리액트는 대기열에 있는 모든 updater 를 이용하여 `prev state`에 적용한 다음 state를 계산한다.
- 다음 렌더링에 대한 state변수만 업데이트를 한다. 그렇기 때문에 호출 후 state변수에는 호출 전 화면에 있던 이전 값이 남아있다.
- `Object.is`메서드로 입력값과 `current state`를 비교하여 같다면 Reacts는 컴포넌트와 그 자식컴포넌트들을 렌더링하지 않는다.
- React는 state업데이트를 일괄처리한다.  
  모든 이벤트핸들러가 실행되고 set함수를 호출한 뒤 화면을 업데이트하기 때문에 단일 이벤트에 의해 여러번 리렌더링되는 일을 방지할 수 있다.
  - DOM 접근을 위해 React가 화면을 더 일찍 업데이트해야되는 경우 `flushSync`를 사용할 수 있다.
- Strict mode에서 React가 의도지않은 부작용을 찾기위해 `setState`를 두번 호출한다. 이때 updater function이 순수함수가 아니면 오류가 발생할 수 있기때문에 꼭 순수함수이어야한다.


<br>

#### set function은 다음 렌더링에서 반환할 useState에만 영향을 준다.
```js
const [name, setName] = useState('Nanii');

function handleClick() {
  setName('Robin');
  console.log(name); // 아직 'Nanii'
}
```
- set function 은 다음 렌더링에 대한 state변수를 업데이트한다.
- 이미 실행 중인 코드에서는 state 변수가 업데이트되지 않는다.
  ```js
  const [age, setAge] = useState(0);
  function handleClick() {
    setAge(age + 1); // age는 아직 0
    setAge(age + 1); // age는 아직 0
    setAge(age + 1); // age는 아직 0
  } // age는 1
  ```
- 이러한 문제를 해결하기위해 `updater function`을 사용해야한다.
  ```js
  function handleClick() {
    setAge((prev) => prev + 1); // age 0 => 1
    setAge((prev) => prev + 1); // age 1 => 2
    setAge((prev) => prev + 1); // age 2 => 3
  }
  ```


#### useState 기본예시
- text field
  ```js
  export default function MyInput () {
    const [text, setText] = useState('hello');

    function handleChange(e) {
      setText(e.target.value);
    }

    return (
      <>
        <input value={text} onChage={handleChage}>
        <p>typed : {text}</p>
      </>
    )
  }
  ```

- check box
  ```js
  export default function MyCheckbox () {
    const [checked, setChecked] = useState(false);

    function handleChange(e) {
      setChecked(e.target.checked);
    }

    return (
      <>
        <input
          type="checkbox"
          checked={checked}
          onChange={handleChange}
        />
        <p>checked : {checked}</p>
      </>
    )
  }
  ```

