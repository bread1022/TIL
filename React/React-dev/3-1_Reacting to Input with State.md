# Reacting to Input with State

## How declarative UI compares to imperative
> 선언형 UI와 명령형 UI의 차이점

- 명령형
  ```js
  async function handleFormSubmit(e) {
    e.preventDefault();
    disable(textarea);
    disable(button);
    show(loadingMessage);
    hide(errorMessage);
    try {
      await submitForm(textarea.value);
      show(successMessage);
      hide(form);
    } catch (err) {
      show(errorMessage);
      errorMessage.textContent = err.message;
    } finally {
      hide(loadingMessage);
      enable(textarea);
      enable(button);
    }
  }
  ```
- 선언형
  ```jsx
  export default function Form({
    status = 'empty'
  }) {
    if (status === 'success') {
      return <h1>That's right!</h1>
    }
    return (
      <>
        <h2>City quiz</h2>
        <p>
          In which city is there a billboard that turns air into drinkable water?
        </p>
        <form>
          <textarea />
          <br />
          <button>
            Submit
          </button>
        </form>
      </>
    )
  }
  ```

## Thinking about UI declaratively
> UI를 선언적인 방식으로 생각하기

#### 리액트의 선언적인 방식으로 UI 구현하는 방법

1. 컴포넌트의 **모든 시각적 상태를 식별**한다.
2. 상태 **변화를 일으키는 요소**를 찾는다.
3. `useState`로 상태를 표현한다.
4. 비필수적인 `state변수`를 제거한다.
5. 이벤트핸들러에 `setState`를 연결한다.



### Step 1: Identify your component’s different visual states
> Step 1: 컴포넌트의 다양한 시각적 상태 식별하기

- 먼저 유저에게 표시될 수 있는 UI의 다양한 상태를 모두 시각화해야한다.
  - Empty : Input이 비어있을 땐 버튼 비활성화
  - Typing : Input에 입력중일 땐 버튼 활성화
  - Submitting : Input을 제출중일 땐 form이 완전히 비활성화되고 Spinner 표시
  - Success : Input 제출이 성공하면 form이 사라지고 성공 메시지 표시
  - Error : '입력중'상태와 동일하지만 오류 메세지를 표시

- 한 컴포넌트에 시각적 상태가 여러가지 있는 경우, 한 페이지에 모두 표시하는 게 편리할 때도 있다. (`living styleguide` 또는 `storybook`이라 부름)
  ```jsx
  let statuses = [
    'empty',
    'typing',
    'submitting',
    'success',
    'error',
  ];

  export default function App() {
    return (
      <>
        {statuses.map(status => (
          <section key={status}>
            <h4>Form ({status}):</h4>
            <Form status={status} />
          </section>
        ))}
      </>
    );
  }
  ```


### Step 2: Determine what triggers those state changes
> 상태 변경을 촉발하는 요인 파악하기

- 사람의 입력이나 컴퓨터의 입력에 의해 상태 변경이 트리거될 수 있기때문에 `state변수`를 설정해야 UI를 업데이트할 수 있다.
  - 사람의 입력 : 버튼 클릭, 필드 입력, 링크 이동 등
  - 컴퓨터의 입력: 네트워크 응답, 시간초과, 이미지 로딩 등
- UI의 업데이트
  - 유저가 text입력을 변경하면 비어있다가 -> 입력중인 상태로, 또는 입력중 -> 비어있는 상태로 전환해야합니다.
  - 유저가 제출버튼을 클릭하면 입력중 -> 제출중 상태로 전환해야합니다.
  - 네트워크 응답이 성공되면 제출중 -> 성공 상태로 전환해야합니다.
  - 네트워크 요청이 실패되면 제출중 -> 오류메세지와 함께 오류 상태로 전환해야합니다.
  - 이런 UI 상태 전환을 시각화하면 흐름을 잘 파악할 수 있을 뿐만아니라 버그를 분류하는데 도움이 된다!


### Step 3: Represent the state in memory with useState
> 메모리의 상태를 useState 로 표현하기

- 상태는 가능한 적은 수로, 반드시 필요한 `state`부터 컴포넌트의 시각적인 상태들을 `useState`로 표현한다.
  - 예를 들어, 입력에 대한 `answer`를 저장하고 마지막 `error`가 존재한다면 우선 두 상태를 저장한다.
    ```jsx
    const [answer, setAnswer] = useState('');
    const [error, setError] = useState(null);
    ```
  - 또는, 모든 시각적 상태를 충분한 `state`로 추가하고 추후에 비필수적인 `state변수`를 제거한다.
    ```jsx
    const [isEmpty, setIsEmpty] = useState(true);
    const [isTyping, setIsTyping] = useState(false);
    const [isSubmitting, setIsSubmitting] = useState(false);
    const [isSuccess, setIsSuccess] = useState(false);
    const [isError, setIsError] = useState(false);
    ```

### Step 4: Remove any non-essential state variables
> 비필수적인 state 변수 제거하기

- 여러 state가 모순적인 상태인지 아닌지 확인한다.
  - 예) `isTyping`, `isSubmitting`, `success`는 모두 동시에 true가 될 수 없다.
- 다른 state에 중복으로 포함되어 있는지 확인한다.
  - 예) `isEmpty`, `isTyping`은 동시에 true가 될 수 없으며 state를 분리하게되면 동기화가 바로 되지않아 버그가 발생할 수 있다.
  - 이때 `isEmpty`는 `answer.length`로 확인할 수 있다.
- 다른 state를 뒤집으면 동일한 정보를 얻을 수 있는지 확인한다.
  - 예) `isError`는 `error !== null`로 확인할 수 있다.
- `state`를 조금 더 명확하게 모델링하기위해선 `reducer`로 분리하면 좋다! `reducer`를 사용하면 여러 `state변수`를 하나의 객체로 통합하고 관련 로직도 합칠 수 있따는 장점을 가진다.


### Step 5: Connect the event handlers to set state
> 이벤트 핸들러를 연결하여 state 설정하기

- 마지막으론 이벤트핸들러를 생성하여 `state변수`를 설정한다.

