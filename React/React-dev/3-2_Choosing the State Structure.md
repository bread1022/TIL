# Choosing the State Structure

## Principles for structuring state
> state 구조화 원칙

1. 관련 상태를 그룹화한다.  
   **`state변수`를 동시에 업데이트하는 경우 단일 `state변수`로 병합하는 것이 좋다.**
2. `state`의 모순을 피한다.  
   여러 `state` 조각이 **서로 모순되거나 불일치할 수 있는 방식으로 `state`를 구성하지 않는다.**
3. 컴포넌트의 props나 기존 `state변수`에서 계산이 가능한 값인 경우 `state`로 저장하지 않는다.
4. 동일한 데이터가 여러 `state변수`, 중첩객체에 중복되면 동기화 상태를 유지하기 어렵기때문에 중복을 피해야한다.
5. 너무 깊은 중첩은 `state`업데이트가 어렵기 때문에 가능하면 중첩은 얕게 유지한다.

## Group related state
> 관련 state 그룹화하기

```jsx
const [x, setX] = useState(0);
const [y, setY] = useState(0);
```
```jsx
const [position, setPosition] = useState({ x: 0, y: 0 });
```

- 단일, 다중 state변수를 고민할 때 첫번째 접근 방법은 업데이트 시점이 같은지 확인하는 것이다.
- 두개의 `state변수`가 항상 **함께 변경되는 경우 하나**의 `state변수`로 통합하여 이벤트에 따라 동기화 상태를 유지할 수 있다.
   ```jsx
   export default function MovingDot() {
   const [position, setPosition] = useState({
      x: 0,
      y: 0
   });
   return (
      <div
         onPointerMove={e => {
         setPosition({
            x: e.clientX,
            y: e.clientY
         });
         }}
         style={{
         position: 'relative',
         width: '100vw',
         height: '100vh',
         }}>
         <div style={{
         position: 'absolute',
         backgroundColor: 'red',
         borderRadius: '50%',
         transform: `translate(${position.x}px, ${position.y}px)`,
         left: -10,
         top: -10,
         width: 20,
         height: 20,
         }} />
      </div>
   )
   }
   ```
- 필요한 state의 조각수를 모를 때 데이터를 개체나 배열로 그룹화하는 것이 좋으며 사용자 정의 필드를 추가할 수 있는 양식이 있을 때 유용하다.
- state변수가 객체인 경우, 한 필드의 값을 업데이트할 땐 다른 필드를 명시적으로 복사해야한다.
   ```jsx
   setPosition({ ...position, x: 100 }) // ⭕️ x의 값을 변경하려면 다른 필드도 명시적으로 복사해야함
   ```
   ```jsx
   setPosition({ x: 100 }) // ❌ x의 값만 업데이트할 수 없음
   ```


## Avoid contradictions in state
> state의 모순을 피하세요

- 동시에 발생가능한 `state`변수를 업데이트하는 경우 `state`를 하나로 통합하여 모순을 피하는 것이 좋다.
```jsx
export default function FeedbackForm() {
  const [text, setText] = useState('');
  const [isSending, setIsSending] = useState(false);
  const [isSent, setIsSent] = useState(false);

  async function handleSubmit(e) {
    e.preventDefault();
    setIsSending(true);
    await sendMessage(text);
    setIsSending(false);
    setIsSent(true);
    // isSending과 isSent가 동시에 true 되는 상황이 발생할 수 있음
  }
  if (isSent) {
    return <h1>Thanks</h1>
  }
  return (
    <form onSubmit={handleSubmit}>
      <textarea
        disabled={isSending}
        value={text}
        onChange={e => setText(e.target.value)}
      />
      {isSending && <p>Sending...</p>}
    </form>
  );
}
```
```jsx
export default function FeedbackForm() {
  const [text, setText] = useState('');
  const [status, setStatus] = useState('typing');
  // 따라서 status를 하나의 변수로 대체함
  // typing, sending, sent 라는 하나의 변수로 대체하여 모순을 피함!!!

  async function handleSubmit(e) {
    e.preventDefault();
    setStatus('sending');
    await sendMessage(text);
    setStatus('sent');
  }

  const isSending = status === 'sending'; // 가독성을 위한 상수 선언
  const isSent = status === 'sent';

  if (isSent) {
    return <h1>Thanks</h1>
  }
  return (
    <form onSubmit={handleSubmit}>
      <textarea
        disabled={isSending}
        value={text}
        onChange={e => setText(e.target.value)}
      />
      {isSending && <p>Sending...</p>}
    </form>
  );
}
```

## Avoid redundant state
> 불필요한 state를 피하세요


#### props를 state에 그대로 사용하는 것을 지양해야하는 이유 (그대로 미러링 하지X)

- props나 기존 `state변수`로 계산이 가능한 데이터는 `state`로 저장하지 않는다.
- 특별하게 state의 업데이트를 막으려는 의도가 아니라면 props를 state에 넣지않는 것이 좋다.
   ```jsx
   function Message({ messageColor }) {
   const [color, setColor] = useState(messageColor);
   // 초기상태는 props 와 같음
   // 이러면, props가 변경되어도 color state 변수의 값은 변경되지 않음 (setState하지않았으니까)
   // 동기화 하기 위해선 추가적으로 useEffect 내부 콜백으로 useState(() => setColor(messageColor), [messageColor]) 등록해줘야함 => 혼란을 야기할 수 있음
   }
   ```
- 부모 컴포넌트가 나중에 messageColor값을 변경해도 Message 컴포넌트는 state변수를 업데이트하지 않기 때문에 변경된 값이 반영되지 않는다.  
  (첫번째 렌더링 중에만 초기화됨)
   ```jsx
   function Message({ messageColor: color }) {
    ... // 구조분해할당 활용하기!!!!!!!!!
   }

  // 또는

   function Message({ messageColor }) {
   // 상수를 사용하면 부모컴포넌트에서 전달된 prop과 동기화되지않음
   const color = messageColor;
   }
   ```
   ```jsx
   function Message({ messageColor: initialColor }) {
   // props를 state에 그대로 사용하면 특정 prop에 대한 업데이트를 무시하려고 의도할 때만 의미가 있기때문에
   // prop의 이름을 initial 또는 default로 시작하여 값이 무시되는 것을 명확하게 해야함
   const [color, setColor] = useState(initialColor);
   }
   ```



## Avoid duplication in state
> state 중복을 피하세요


```jsx
const initialItems = [
  { title: 'pretzels', id: 0 },
  { title: 'crispy seaweed', id: 1 },
  { title: 'granola bar', id: 2 },
];

export default function Menu() {
  const [items, setItems] = useState(initialItems);
  const [selectedItem, setSelectedItem] = useState(items[0]);
  // items = [{ id: 0, title: 'pretzels'}, ...]
  // selectedItem = {id: 0, title: 'pretzels'} 중복되는 정보가 계속해서 복사됨

  function handleItemChange(id, e) {
    setItems(items.map(item => {
      // items에 관련된 중복 state인데
      // setItems 했을 때 setSelectedItem를 하지않아 동기화되지 않는 문제 발생
      if (item.id === id) {
        return {
          ...item,
          title: e.target.value,
        };
      }
      return item;
    }));
  }

  return (
    <>
      <ul>
        {items.map((item, index) => (
          <li key={item.id}>
            <input
              value={item.title}
              onChange={e => {
                handleItemChange(item.id, e)
              }}
            />
            {' '}
            <button onClick={() => {
              setSelectedItem(item);
            }}>Choose</button>
          </li>
        ))}
      </ul>
      <p>You picked {selectedItem.title}.</p>
    </>
  );
}
```
  - **선택**과 같은 UI 패턴의 경우 객체 자체보단 배열의 index, 또는 id를 사용하는 것 좋다 !!!

    ```jsx
    import { useState } from 'react';

    const initialItems = [
      { title: 'pretzels', id: 0 },
      { title: 'crispy seaweed', id: 1 },
      { title: 'granola bar', id: 2 },
    ];

    export default function Menu() {
      const [items, setItems] = useState(initialItems);
      const [selectedId, setSelectedId] = useState(0);
      // items = [{ id: 0, title: 'pretzels'}, ...]
      // selectedId = 0 중복을 제거하고 필수 state만 유지

      // items 내부 중복된 데이터를 복사하지않고 선택한 인덱스에 대한 state변수를 생성하여 중복을 피함
      const selectedItem = items.find(item =>
        item.id === selectedId
      );

      function handleItemChange(id, e) {
        setItems(items.map(item => {
          if (item.id === id) {
            return {
              ...item,
              title: e.target.value,
            };
          }
          return item;
        }));
      }

      return (
        <>
          <ul>
            {items.map((item, index) => (
              <li key={item.id}>
                <input
                  value={item.title}
                  onChange={e => {
                    handleItemChange(item.id, e)
                  }}
                />
                {' '}
                <button onClick={() => {
                  setSelectedId(item.id);
                }}>Choose</button>
              </li>
            ))}
          </ul>
          <p>You picked {selectedItem.title}.</p>
        </>
      );
    }
    ```


## Avoid deeply nested state
> 깊게 중첩된 state는 피하세요

- 중첩된 `state`는 업데이트하기위해 변경된 부분부터 위쪽까지 객체의 복사본을 계속해서 생성해야한다.
- `state`가 너무 중첩되어있을 땐 `state`를 "flat(정규화)"하게 만들어 (필요한 데이터를 맵핑하여 사용) `state`를 쉽게 업데이트하고 중첩된 객체의 중복을 방지할 수 있다~!!
    ```js
    const initialTravelPlan = {
      id: 0,
      title: '(Root)',
      childPlaces: [{
        id: 1,
        title: 'Earth',
        childPlaces: [{
          id: 2,
          title: 'Africa',
          childPlaces: [{
            id: 3,
            title: 'Botswana',
            childPlaces: []
          }, {
            id: 9,
            title: 'South Africa',
            childPlaces: []
          }]
        }, {
          id: 35,
          title: 'Oceania',
          childPlaces: [{
            id: 36,
            title: 'Australia',
            childPlaces: [],
          }]
        }]
      }, {
        id: 47,
        title: 'Mars',
        childPlaces: [{
          id: 48,
          title: 'Corn Town',
          childPlaces: []
        }, {
          id: 49,
          title: 'Green Hill',
          childPlaces: []
        }]
      }]
    };
    ```

    ```js
    const initialTravelPlan = {
      0: {
        id: 0,
        title: '(Root)',
        childIds: [1, 43, 47],
      },
      1: {
        id: 1,
        title: 'Earth',
        childIds: [2, 10, 19, 27, 35]
      },
      2: {
        id: 2,
        title: 'Africa',
        childIds: [3, 4, 5, 6 , 7, 8, 9] // 부모 - 자식 관계를 맺는 childIds를 추가하여 중첩을 방지
      },
    };
    ```



### Improving memory usage
> 메모리 사용량 개선하기

- Immer를 사용하면 객체 복사하지않고 간결한 로직으로 state를 업데이트할 수 있다...~


---------

### 과제1

```jsx
export default function Clock(props) {
  const [color, setColor] = useState(props.color);
  // props 가 변경되어도 color가 업데이트 되지 않음 (state를 사용하고 있기 때문!)
  return (
    <h1 style={{ color: color }}>
      {props.time}
    </h1>
  );
}
```
- 만약 state를 사용하고 싶다면 props 값에 따라 state를 업데이트 하기 위해선 useEffect를 사용해야함 (전혀 좋지 않은 방법~!)
  ```jsx
  useEffect(() => {
    setColor(props.color);
  }, [props.color]);
  ```

```jsx
export default function Clock({ color, time } ) {
  return (
    <h1 style={{ color: color }}>
      {time}
    </h1>
  );
}
```

------

### 과제2


```jsx
export default function TravelPlan() {
  const [items, setItems] = useState(initialItems);
  const [total, setTotal] = useState(3); // items를 이용하여 계산할 수 있는 정보 (=중복)
  const [packed, setPacked] = useState(1); // items를 이용하여 계산할 수 있는 정보 (=중복)

  function handleAddItem(title) {
    setTotal(total + 1);
    setItems([
      ...items,
      {
        id: nextId++,
        title: title,
        packed: false
      }
    ]);
  }

  function handleChangeItem(nextItem) {
    if (nextItem.packed) {
      setPacked(packed + 1);
    } else {
      setPacked(packed - 1);
    }
    setItems(items.map(item => {
      if (item.id === nextItem.id) {
        return nextItem;
      } else {
        return item;
      }
    }));
  }

  function handleDeleteItem(itemId) {
    setTotal(total - 1);
    setItems(
      items.filter(item => item.id !== itemId)
    );
  }

  return (
    <>
      <AddItem
        onAddItem={handleAddItem}
      />
      <PackingList
        items={items}
        onChangeItem={handleChangeItem}
        onDeleteItem={handleDeleteItem}
      />
      <hr />
      <b>{packed} out of {total} packed!</b>
    </>
  );
}
```

```jsx
const [total, setTotal] = useState(3); // items를 이용하여 계산할 수 있는 정보 (=중복)
const [packed, setPacked] = useState(1); // items를 이용하여 계산할 수 있는 정보 (=중복)

// 중복된 state를 제거하고 변수로 사용하면 이벤트 핸들러는 setItems의 호출에만 동작하고,
// 다음 렌더링 중 items의 값에 의해 total, packed의 값을 계산하여 항상 최신 상태를 유지할 수 있음!!!
const total = items.length;
const packed = items.filter(item => item.packed).length;
```


-------

### 과제4

```jsx
export default function MailClient() {
  const [selectedId, setSelectedId] = useState(new Set()); // 중복을 제거하고 새로운 값을 덮어씌울 수 있음
  const selectedCount = selectedId.size;

  function handleToggle(toggledId) {
    setSelectedId((prev) => {
      const newSelectedId = new Set(prev);
      if(prev.has(toggledId)) newSelectedId.delete(toggledId);
      else newSelectedId.add(toggledId);
      return newSelectedId;
    })
  }

  return (
    <>
      <h2>Inbox</h2>
      <ul>
        {letters.map(letter => (
          <Letter
            key={letter.id}
            letter={letter}
            isSelected={
              letter.id === selectedId.has(letter.id)
            }
            onToggle={handleToggle}
          />
        ))}
        <hr />
        <p>
          <b>
            You selected {selectedCount} letters
          </b>
        </p>
      </ul>
    </>
  );
}
```