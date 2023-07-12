# Element Rendering

index.html 파일 내 `<div id="root"></div>` 루트 노드는 일반적으로 하나 존재한다. React element를 렌더링 하기 위해서는 DOM element를 `ReactDOM.createRoot()`에 전달한다음 React element를 `root.render()`에 전달해야한다.

```js
const root = ReactDOM.createRoot(
  document.getElementById('root')
);
const element = <h1>Hello, world</h1>;
root.render(element);
```

JS로 UI를 업데이트하는 방법은 new element 를 생성하고 이를 `root.render()`에 전달하는 것이다.

React DOM은 해당 element와 그 자식 element를 이전의 element와 비교하여 원하는 상태에 필요한 경우에만 DOM을 업데이트한다.

# Components

재사용이 가능한 UI를 개별적인 여러 조각으로 나눈 단위로 React의 핵심 개념 중 하나이다. MVC의 view를 독립적으로 구성하여 재사용할 수 있고 이를 통해 새 컴포넌트를 쉽게 만들 수 있는 장점을 가진다.

컴포넌트는 props로 데이터를 전달받아 state에 따라 DOM node를 출력하는 함수로 생성하며 컴포넌트 이름은 대문자로 시작하도록 한다.

### 컴포넌트의 구성요소

- **props** (property)
  - 상위컴포넌트에서 하위컴포넌트에 전달되는 데이터객체
  - 읽기 전용 스냅샷으로 렌더링할 때 마다 새로운 버전의 props를 받는다
  - 컴포넌트에서 props는 변경할 수 없다 (동일 입력에 동일 출력을 반환해야 한다)
  - **단방향 데이터 흐름**을 갖는다
  - 모든 React 컴포넌트는 자신의 props를 다룰 때 반드시 순수 함수처럼 동작해야한다
- **state** : 컴포넌트의 상태를 저장하고 수정이 가능한 데이터
- **context** : 상위컴포넌트에서 생성하여 모든 하위컴포넌트에게 전달하는 데이터

### 컴포넌트의 종류
- **함수컴포넌트** : JS 함수기반 컴포넌트로 function으로 컴포넌트를 정의하고 return 문에 JSX 코드를 반환한다.
    ```js
    function Welcome(props) {
      return <h1>Hello, {props.name}</h1>;
    }
    ```
  - React v16이후 React Hooks을 통해 state(`useState`), lifecycle(`useEffect`) 관리가 가능해지면서 사용을 많이 하게 되었다.
  - 코드가 직관적이고 메모리자원을 효율적으로 사용할 수 있다
- **클래스컴포넌트** : Class 로 정의하고 render()함수에서 JSX 코드를 반환한다.
  ```js
  class Welcome extends React.Component {
    render() {
      return <h1>Hello, {this.props.name}</h1>;
    }
  }
    ```
  - React ComponentClass의 상속이 필요하고 render()메서드를 필수로 사용해야한다.

## 컴포넌트 렌더링

DOM 태그 외에 사용자 정의 컴포넌트로도 React element를 나타낼 수 있고, React가 사용자 정의 컴포넌트로 작성한 엘리먼트를 발견하면 JSX 어트리뷰트와 자식을 해당 컴포넌트에 단일 객체로 전달한다.

```js
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}

const root = ReactDOM.createRoot(document.getElementById('root'));
const element = <Welcome name="Sara" />;
root.render(element);
```

## 컴포넌트 나누기
컴포넌트가 중첩 구조로 이루어져있으면 변경, 재사용이 어렵기때문에 중복해서 사용하는 요소는 컴포넌트로 분리하는 것이 좋다.
```js
function Comment(props) {
  return (
    <div className="Comment">
      <div className="UserInfo">
        <img className="Avatar"
          src={props.author.avatarUrl}
          alt={props.author.name}
        />
        <div className="UserInfo-name">
          {props.author.name}
        </div>
      </div>
      <div className="Comment-text">
        {props.text}
      </div>
      <div className="Comment-date">
        {formatDate(props.date)}
      </div>
    </div>
  );
}
```
- imgage 태그, userInfo Div 태그 분리해보기

  ```js
  function Avatar(props) {
    return (
      <img className="Avatar"
        src={props.user.avatarUrl}
        alt={props.user.name}
      />
    );
  }
  ```

  ```js
  function UserInfo(props) {
    return (
      <div className="UserInfo">
        <Avatar user={props.user} />
        <div className="UserInfo-name">
          {props.user.name}
        </div>
      </div>
    );
  }
  ```

  ```js
  function Comment(props) {
    return (
      <div className="Comment">
        <UserInfo user={props.author} />
        <div className="Comment-text">
          {props.text}
        </div>
        <div className="Comment-date">
          {formatDate(props.date)}
        </div>
      </div>
    );
  }
  ```

## Props 전달

### default props 지정
props가 누락되거나 `undefined`인 경우를 대비해 기본값을 지정해줄 땐 변수 바로 뒤에 `=`와 함께 기본값을 넣어 구조분해할당할 수 있다.
```js
function Avatar({ person, size = 100 }) {
  // ...
}
```

### 전개구문으로 props 전달
반복적인 props 또는 상위컴포넌트에서 자식 컴포넌트에게 모든 props를 전달할 땐 "spread syntax"를 사용하여 가독성을 높일 수 있다.
```js
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
- Profile의 모든 props를 Avatar에게 전달
    ```js
    function Profile(props) {
      return (
        <div className="card">
          <Avatar {...props} />
        </div>
      );
    }
    ```

### props가 많은 경우 객체로 전달
```js
function Profile({
  imageId,
  name,
  profession,
  awards,
  discovery,
  imageSize = 70
}) {
  return (
    <section className="profile">
      <h2>{name}</h2>
      <img
        className="avatar"
        src={getImageUrl(imageId)}
        alt={name}
        width={imageSize}
        height={imageSize}
      />
      <ul>
        <li><b>Profession:</b> {profession}</li>
        <li>
          <b>Awards: {awards.length} </b>
          ({awards.join(', ')})
        </li>
        <li>
          <b>Discovered: </b>
          {discovery}
        </li>
      </ul>
    </section>
  );
}

export default function Gallery() {
  return (
    <div>
      <h1>Notable Scientists</h1>
      <Profile
        imageId="szV5sdG"
        name="Maria Skłodowska-Curie"
        profession="physicist and chemist"
        discovery="polonium (chemical element)"
        awards={[
          'Nobel Prize in Physics',
          'Nobel Prize in Chemistry',
          'Davy Medal',
          'Matteucci Medal'
        ]}
      />
      <Profile
        imageId='YfeOqp2'
        name='Katsuko Saruhashi'
        profession='geochemist'
        discovery="a method for measuring carbon dioxide in seawater"
        awards={[
          'Miyake Prize for geochemistry',
          'Tanaka Prize'
        ]}
      />
    </div>
  );
}
```

```js
function Profile({ person, imageSize = 70 }) {
  const imageSrc = getImageUrl(person)

  return (
    <section className="profile">
      <h2>{person.name}</h2>
      <img
        className="avatar"
        src={imageSrc}
        alt={person.name}
        width={imageSize}
        height={imageSize}
      />
      <ul>
        <li>
          <b>Profession:</b> {person.profession}
        </li>
        <li>
          <b>Awards: {person.awards.length} </b>
          ({person.awards.join(', ')})
        </li>
        <li>
          <b>Discovered: </b>
          {person.discovery}
        </li>
      </ul>
    </section>
  )
}

export default function Gallery() {
  return (
    <div>
      <h1>Notable Scientists</h1>
      <Profile person={{
        imageId: 'szV5sdG',
        name: 'Maria Skłodowska-Curie',
        profession: 'physicist and chemist',
        discovery: 'polonium (chemical element)',
        awards: [
          'Nobel Prize in Physics',
          'Nobel Prize in Chemistry',
          'Davy Medal',
          'Matteucci Medal'
        ],
      }} />
      <Profile person={{
        imageId: 'YfeOqp2',
        name: 'Katsuko Saruhashi',
        profession: 'geochemist',
        discovery: 'a method for measuring carbon dioxide in seawater',
        awards: [
          'Miyake Prize for geochemistry',
          'Tanaka Prize'
        ],
      }} />
    </div>
  );
}
```