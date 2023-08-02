# Sharing State Between Components


하나의 `state`를 두개이상의 컴포넌트에서 공유하면서 항상 동기화되어야할 때,  
각각의 컴포넌트 내부에서 해당 `state`를 제거하고 가장 가까운 공통 부모 컴포넌트에 해당 `state`를 둔 다음  
props를 통해 `state`를 전달하면 된다. 이를 `lifting state up` 이라 한다.



## Lifting state up by example
> 예제로 알아보는 state 끌어올리기

```jsx
function Panel({ title, children }) {
  const [isActive, setIsActive] = useState(false);
  return (
    <section className="panel">
      {isActive ? (
        <p>{children}</p>
      ) : (
        <button onClick={() => setIsActive(true)}>
          Show
        </button>
      )}
    </section>
  );
}

export default function Accordion() {
  return (
    <>
      <h2>Almaty, Kazakhstan</h2>
      <Panel title="About">
        With a population of about 2 million, Almaty is Kazakhstan's largest city.
      </Panel>
      <Panel title="Etymology">
        The name comes from <span lang="kk-KZ">алма</span>, the Kazakh word for "apple" and is often translated as "full of apples".
      </Panel>
    </>
  );
}
```
- Panel 컴포넌트 자체 `isActive state`를 가지고 있기때문에 각각의 컴포넌트의 `isActive` 상태는 독립적으로 동작한다.
- 만약, 한번에 하나의 Panel만 열고싶을 땐 `isActive`state를 부모에서 하나만 가지고있어야한다.


### Step 1: Remove state from the child components
> 자식 컴포넌트에서 state 제거

- 자식 컴포넌트에서 개별적으로 가지고 있던 state를 제거하고,  
  부모 컴포넌트에서의 state를 prop으로 전달받을 수 있도록 매개변수에 추가한다.  
  (=> 부모컴포넌트에 의해 제어되는 컴포넌트가 됨)

### Step 2: Pass hardcoded data from the common parent
> 공통 부모에 하드 코딩된 데이터 전달하기

### Step 3: Add state to the common parent
> 공통 부모에 state 추가


```jsx
export default function Accordion() {
  const [activeIndex, setActiveIndex] = useState(0);
  // 0 이면 About, 1 이면 Etymology 둘 중에 하나만 보여지게끔함

  return (
    <>
      <h2>Almaty, Kazakhstan</h2>
      <Panel title="About" isActive={activeIndex === 0} onShow={() => setActiveIndex(0)}>
        With a population of about 2 million, Almaty is Kazakhstan's largest city.
      </Panel>
      <Panel title="Etymology" isActive={activeIndex === 1} onShow={() => setActiveIndex(1)}>
        The name comes from <span lang="kk-KZ">алма</span>, the Kazakh word for "apple" and is often translated as "full of apples".
      </Panel>
    </>
  );
}

function Panel({ title, children, isActive }) {
  return (
    <section className="panel">
      <h3>{title}</h3>
      {isActive ? (
        <p>{children}</p>
      ) : (
        <button onClick={() => setIsActive(true)}>
          Show
        </button>
      )}
    </section>
  );
}
```

<p align="center"><img src="https://react-ko.dev/_next/image?url=%2Fimages%2Fdocs%2Fdiagrams%2Fsharing_state_parent.png&w=1080&q=75" width="250px"/> <img src="https://react-ko.dev/_next/image?url=%2Fimages%2Fdocs%2Fdiagrams%2Fsharing_state_parent_clicked.png&w=1080&q=75" width="250px"/></p>


#### Controlled and uncontrolled components (state, props)

- 컴포넌트 자체 로컬 **state**를 가진 컴포넌트를 **비제어컴포넌트** (부모 컴포넌트의 제어되지x)
- 컴포넌트 **props**에 의해 동작하는 컴포넌트를 **제어컴포넌트** (부모 컴포넌트에 의해 동작, 제어o)


#### 제어 / 비제어 컴포넌트 (form 요소)

제어 | 비제어
---|---
useState | useRef
value가 실시간으로 동기화 | 실시간 동기화 x
액션마다 렌더링이 발생(불필요한 렌더링 발생) | 직접 트리거할 때까지 렌더링 x
ex. 유효성검사, 실시간 입력 형식에 적용 | ex. submit 이벤트 (form요소에만 onChange 이벤트핸들러를 등록)
제어컴포넌트에 의한 불필요한 해결방법으로 `Throttling`, `Debouncing` 을 적용 | _


## A single source of truth for each state
> 각 state의 단일 진실 공급원(SSOT)

- 트리구조의 컴포넌트들의 depth가 깊어져도 state를 상위에서 가지고 있다면 그 컴포넌트가 SSOT(single source of truth)가 된다..


