# Passing Props to a Component

- 부모컴포넌트와 자식컴포넌트는 props를 이용해 `HTML attribute` 뿐만아니라 `Object`, `Array`, `Function을` 포함한 모든 JavaScript 값을 가진 데이터를 전달할 수 있다.
- **`props`란 JSX 태그에 전달하는 정보**로 컴포넌트에 대한 유일한 인자이다.
  - ex. `<img> 태그` 에서 `className`, `src`, `alt`, `width`, `height` 등을 말한다.
- 리액트 컴포넌트 함수는 하나의 인자, `props 객체`를 전달 받아 사용한다.
- ReactDOM은 HTML [Standard](https://html.spec.whatwg.org/multipage/)를 준수한다.


### Step 1: Pass props to the child component
> 자식 컴포넌트에 props 전달하기

```jsx
// 부모 컴포넌트 Profile
export default function Profile() {
  return (
    <Avatar
      person={{ name: 'Lin Lanying', imageId: '1bX5QH6' }}
      size={100}
    />
  );
}
```
- `<Avatar> 컴포넌트`에 `person (객체)` 와 `size (숫자)` 를 전달한다.

### Step 2: Read props inside the child component
> 자식 컴포넌트 내부에서 props 읽기

```jsx
// 자식 컴포넌트 Avatar
function Avatar({ person, size }) {
  // person and size are available here
}
```
- 자식 컴포넌트 함수의 하나의 인자로 전달받고, 이 `props 객체`를 사용하기 위해 구조분해할당을 진행한다.
- 자식 컴포넌트의 괄호안에 `({ ... })` 중괄호를 사용하여 `person` 과 `size` 를 구조분해할당한다.
- 보통은 props 객체 자체를 필요로 하지 않기때문에 구조분해할당하여 사용한다. 만약 구조분해할당을 사용하지 않으면 함수의 매개변수의 속성으로 접근해야한다.
  ```jsx
  function Avatar(props) {
    let person = props.person;
    let size = props.size;
    // ...
  }
  ```

## Specifying a default value for a prop
> prop의 기본값 지정하기

- prop에서 기본값을 주기 원하면, 변수 바로 `=` 을 사용하여 기본값을 넣을 수 있다.
- 기본값을 지정해준 **prop의 값이 `undefined`일 때 지정된 기본값을 사용**한다. 하지만 `null` 이거나 `0` 으로 전달되면 기본값은 사용하지 않는다.

  ```jsx
  function Avatar({ person, size = 100 }) { // size의 기본값은 100 이다.
    // ...
  }
  ```


## Forwarding props with the JSX spread syntax
> JSX 전개 구문으로 props 전달하기

일부 컴포넌트는 부모 컴포넌트에서 받은 props을 구조분해할당하여 사용하지 않고 그 props 전체를 자식 컴포넌트에 전달해야할 땐 전개구문을 사용하여 코드의 반복을 줄이고 가독성을 높일 수 있다.

```jsx
function Profile({ person, size, isSepia, thickBorder }) {
  return (
    <div className="card">
      <Avatar
        person={person}
        size={size}
        isSepia={isSepia}
        thickBorder={thickBorder}
      />
    </div>
  );
}
```
- Profile 컴포넌트의 모든 `props`를 한번에 Avatar에 전달해야하는 경우 `{...props}`로 전달한다.
  ```jsx
  function Profile(props) {
    return (
      <div className="card">
        <Avatar {...props} />
      </div>
    );
  }
  ```

## Passing JSX as children
> 자식을 JSX로 전달하기

- JSX 태그 내에 콘텐츠를 중첩하면 부모 컴포넌트는 해당 컨텐츠를 `children` 으로 인식한다.
    ```jsx
    <Card>
      <Avatar />
    </Card>
    ```


```jsx
import Avatar from './Avatar.js';

function Card({ children }) {
  return (
    <div className="card">
      {children}
    </div>
  );
}

export default function Profile() {
  return (
    <Card>
    // Avatar를 Card 컴포넌트의 자식 컴포넌트로 인식한다.
      <Avatar
        size={100}
        person={{
          name: 'Katsuko Saruhashi',
          imageId: 'YfeOqp2'
        }}
      />
    </Card>
  );
}
```

## How props change over time
> 시간에 따라 props가 변하는 방식

- **props는 변경할 수 없다**. 컴포넌트 내부에서 props로 전달받은 객체를 변경해야하는 경우 부모 컴포넌트에 다른 props로 새로운 객체를 전달해야하며 변경 전에 전달받은 props는 가비지컬렉터의 대상이 된다.