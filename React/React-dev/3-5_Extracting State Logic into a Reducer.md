# Extracting State Logic into a Reducer

여러 이벤트 핸들러에 `state` 업데이트가 분산되어있는 경우 컴포넌트 과부하가 발생할 수 있어  
`reducer`를 사용하여 `state` 업데이트 로직을 통합•관리하는 것이 좋다.


## Consolidate state logic with a reducer
> reducer로 state 로직 통합하기

- `state`가 업데이트되는 경우를 한눈에 파악하기 어려울 때  
  복잡성을 줄이고 분산되어있는 로직들을 한 곳에 모으기 위해
  컴포넌트 외부의 단일함수`reducer`로 옮길 수 있다.
- `useState`를 `useReducer`로 마이그레이션하는 방법
  1. `state`설정을 action전달 로직으로 변경
  2. reducer 함수 작성
  3. 컴포넌트에서 reducer 사용


### Step 1: Move from setting state to dispatching actions
> state 설정을 action들의 전달로 바꾸기

```jsx
let nextId = 3;
const initialTasks = [
  {id: 0, text: 'Visit Kafka Museum', done: true},
  {id: 1, text: 'Watch a puppet show', done: false},
  {id: 2, text: 'Lennon Wall pic', done: false},
];

export default function TaskApp() {
  const [tasks, setTasks] = useState(initialTasks);

  function handleAddTask(text) {
    setTasks([
      ...tasks,
      {
        id: nextId++,
        text: text,
        done: false,
      },
    ]);
  }

  function handleChangeTask(task) {
    setTasks(
      tasks.map((t) => {
        if (t.id === task.id) return task;
        return t;
      })
    );
  }

  function handleDeleteTask(taskId) {
    setTasks(tasks.filter((t) => t.id !== taskId));
  }

  return (
    <>
      <h1>Prague itinerary</h1>
      <AddTask onAddTask={handleAddTask} />
      <TaskList
        tasks={tasks}
        onChangeTask={handleChangeTask}
        onDeleteTask={handleDeleteTask}
      />
    </>
  );
}
```
- state 설정 로직을 제거하고 action을 전달하는 로직으로 변경
    ```jsx
    function handleAddTask(text) {
      dispatch({ // dispatch 함수를 통해 action 객체를 전달
        type: 'add', // action : 추가해라~
        id: nextId++,
        text: text,
      });
    }

    function handleChangeTask(task) {
      dispatch({
        type: 'change',  // action : 변경해라!
        task: task,
      });
    }

    function handleDeleteTask(taskId) {
      dispatch({
        type: 'delete',  // action : 삭제!~
        id: taskId,
      });
    }
    ```
    - dispatch함수의 action객체에는 type(무슨 일이 일어났는지 설명), payload(업데이트에 필요한 데이터)가 포함되어야한다.


### Step 2: Write a reducer function
> reducer 함수 작성하기

- `reducer`함수
  - 매개변수
    - (1) `현재 state`
    - (2) `action 객체`
  - 반환값 : `다음 state`
  - 순수함수여야한다.
  - 테스트하기 좋다.


```jsx
function tasksReducer(tasks, action) {
  switch(action.type) {
    case 'add': { // 중괄호로 감싸야 변수 중복 선언이 가능
      return [ // return을 해야 case를 빠져나옴
        ...tasks,
        {
          id: action.id,
          text: action.text,
          done: false,
        },
      ];
    }
    case 'change': {
      return tasks.map((t) => {
        if (t.id === action.task.id) return action.task;
        return t;
      });
    }
    case 'delete': {
      return tasks.filter((t) => t.id !== action.id);
    }
    default: {
      return throw Error('Unknown action: ' + action.type);
    }
  }
}
```
- state를 매개변수로 받기때문에 컴포넌트 밖에서 reducer 함수를 선언하고 사용할 수 있고  
  이렇게 되면 depth도 줄이고 코드를 읽기 쉽게 만들 수 있다.


### Step 3: Use the reducer from your component
> 컴포넌트에서 reducer 사용하기

#### `useReducer` Hook

- 인자
  - `reducer함수`
  - `초기state`
- 반환값
  - `state값`
  - `dispatch 함수` : action을 reducer에 전달하는 역할

```jsx
import { useReducer } from 'react'; // Hook import

const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);
```

```jsx
// tasksReducer.js
export default function tasksReducer(tasks, action) {
  switch (action.type) {
    case 'added': {
      return [
        ...tasks,
        {
          id: action.id,
          text: action.text,
          done: false,
        },
      ];
    }
    case 'changed': {
      return tasks.map((t) => {
        if (t.id === action.task.id) {
          return action.task;
        } else {
          return t;
        }
      });
    }
    case 'deleted': {
      return tasks.filter((t) => t.id !== action.id);
    }
    default: {
      throw Error('Unknown action: ' + action.type);
    }
  }
}
```

```jsx
import { useReducer } from 'react';
import tasksReducer from './tasksReducer.js';

let nextId = 3;
const initialTasks = [
  {id: 0, text: 'Visit Kafka Museum', done: true},
  {id: 1, text: 'Watch a puppet show', done: false},
  {id: 2, text: 'Lennon Wall pic', done: false},
];

export default function TaskApp() {
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);

  function handleAddTask(text) {
    dispatch({
      type: 'added',
      id: nextId++,
      text: text,
    });
  }

  function handleChangeTask(task) {
    dispatch({
      type: 'changed',
      task: task,
    });
  }

  function handleDeleteTask(taskId) {
    dispatch({
      type: 'deleted',
      id: taskId,
    });
  }

  return (
    <>
      <h1>Prague itinerary</h1>
      <AddTask onAddTask={handleAddTask} />
      <TaskList
        tasks={tasks}
        onChangeTask={handleChangeTask}
        onDeleteTask={handleDeleteTask}
      />
    </>
  );
}
```
- reducer를 사용하면 이벤트핸들러에 action만을 전달하여 state를 업데이트할 수 있다.



## Comparing useState and useReducer
> useState와 useReducer 비교하기

_|useState|useReducer|
|---|---|---|
코드크기 | 미리 작성해야하는 코드가 간결 | reducer함수와 action객체를 모두 작성해야함. But 이벤트핸들러가 비슷한 방식으로 state를 업데이트하는 경우 useReducer를 사용하는 것이 좋음
가독성 | 간단한 state업데이트할 땐 좋음. But 복잡해지면 코드양이 많아질 수 있음 | state업데이트 로직이 어떻게 동작하는 지 이벤트 핸들러를 통해 어떤 action을 하는지 한눈에 파악하기 좋음
디버깅 | 어려움 | 순수함수이기때문에 테스트 하기 쉽고, action이 정확하면 버그가 reducer에 있다는 것을 알 수 있음


## Writing reducers well
> reducer 잘 작성하기

- `reducer`는 순수함수여야한다. (입력값이 같으면 결과값도 항상 같아야함)
- 요청을 보내거나 sideEffect를 수행해서는 안된다.
- `state`를 직접 수정하면 안된다. (변이없이 업데이트해야함)
- action은 데이터가 여러개 변경되어도 하나의 action으로 표현할 수 있어야한다.  
  (5개의 개별 필드 action을 5번 전달하기보단 1개의 action으로 전달하는 것이 좋음. 로그가 명확해야함)


## Writing concise reducers with Immer
> Immer를 사용하여 간결한 reducer 작성하기


```json
{
  "dependencies": {
    "immer": "1.7.3",
    "react": "latest",
    "react-dom": "latest",
    "react-scripts": "latest",
    "use-immer": "0.5.1"
  },
  "scripts": {
  },
  "devDependencies": {}
}
```

```jsx
import { useImmerReducer } from 'use-immer';

function tasksReducer(draft, action) {
  switch (action.type) {
    case 'added': {
      draft.push({
        id: action.id,
        text: action.text,
        done: false,
      });
      break;
    }
    case 'changed': {
      const index = draft.findIndex((t) => t.id === action.task.id);
      draft[index] = action.task;
      break;
    }
    case 'deleted': {
      return draft.filter((t) => t.id !== action.id);
    }
    default: {
      throw Error('Unknown action: ' + action.type);
    }
  }
}

export default function TaskApp() {
  const [tasks, dispatch] = useImmerReducer(tasksReducer, initialTasks);

  function handleAddTask(text) {
    dispatch({
      type: 'added',
      id: nextId++,
      text: text,
    });
  }

  function handleChangeTask(task) {
    dispatch({
      type: 'changed',
      task: task,
    });
  }

  function handleDeleteTask(taskId) {
    dispatch({
      type: 'deleted',
      id: taskId,
    });
  }

  return (
    <>
      <h1>Prague itinerary</h1>
      <AddTask onAddTask={handleAddTask} />
      <TaskList
        tasks={tasks}
        onChangeTask={handleChangeTask}
        onDeleteTask={handleDeleteTask}
      />
    </>
  );
}
```
- `push`요소를 추가하거나 `arr[i]=x`값을 할당하여 state를 변이할 수 있다.
- Immer의 `draft`객체는 state의 복사본을 생성하여 안전하게 변이할 수 있는 객체로  
  reducer함수에서 따로 `state`를 반환할 필요 없다.

-----

#### 과제

```jsx
const useReducer = (reducer, initialState) => {
  const [state, setState] = useState(initialState);
  const dispatch = (action) => {
    // 다음 렌더링 까지 전달된 action이 대기열에 대기하기 때문에 updater function을 사용해야함
    setState((pre)=> reducer(pre, action));
  }
  return [state, dispatch];
}
```