# Updating Objects in State


## What’s a mutation?
> 변이란?

- JavaScript에서 원시값은 불변이나, 참조형은 객체 자체의 내용 변경이 가능하다.
- 객체의 내용을 변경하는 것을 **변이**라고 한다.


## Treat state as read-only
> state를 읽기 전용으로 취급하세요

```jsx
onPointerMove={e => {
  position.x = e.clientX;
  position.y = e.clientY;
}}
```
```jsx
onPointerMove={e => {
  // 새로운 객체로 덮어씌워야한다.
  setPosition({
    x: e.clientX,
    y: e.clientY
  });
}}
```

- `state`에 객체를 저장하면 객체를 변이해도 `setState`를 하지 않으면 리액트는 객체가 변경되었는지 알지 못한다.
- 따라서 리렌더링을 트리거하기위해서는 **새객체를 생성하고 `setState`에 전달**해야한다.
  - 설정자 함수는 새 객체로 바꾸고 리렌더링을 지시하는 역할을 한다.

#### 지역변이 (local mutation)

- 기존객체의 `state`를 변경하지 않고 방금 생성한 새로운 객체를 변경하면 해당 객체에 의존하는 다른 객체에 영향이 가지 않는다.
- 렌더링하는 동안에 지역변이를 수행하는 것이기때문에 다른 코드가 아직 참조하지 않으므로 안전하다.


## Copying objects with the spread syntax
> 전개 구문을 사용하여 객체 복사하기

- 새로운 객체를 생성할 때 기존 객체의 일부를 복사하여 일부만 업데이트하고 다른 필드는 이전 값을 유지하고 싶을 때 `spread syntax`를 사용한다. (얕은 복사)
  ```jsx
  const handleNameChage = (e) => {
    setPerson({
      firstName: person.firstName,
      lastName: e.target.value,
      email: person.email
    });
  }
  ```

#### `Object.assign()`
- 첫번째 인자에 타겟 객체를 두고, 대상 객체들의 `property`를 하나씩 꺼내 복사하여 넣는다. (불변 객체 업데이트)
  ```jsx
  const handleNameChage = (e) => {
    setPerson(Object.assign({}, person, {
      lastName: e.target.value
    }));
  }
  ```

### Spread syntax
- 모든 속성을 개별적으로 복사하지 않고 객체를 복사할 수 있다.
  ```jsx
  const handleNameChage = (e) => {
    setPerson({
      ...person,
      lastName: e.target.value
    });
  }
  ```


### Closure 를 이용하여 단일 이벤트 핸들러 사용하기

```jsx
function handleFirstNameChange(e) {
  person.firstName = e.target.value;
}

function handleLastNameChange(e) {
  person.lastName = e.target.value;
}

function handleEmailChange(e) {
  person.email = e.target.value;
}
```
```jsx
const handleChage = (target) => (e) => {
  setPerson({
    ...person,
    [target]: e.target.value
  });
}
```

### 속성을 이용하여 단일 이벤트 핸들러 사용하기

```jsx
export default function Form() {
  const [person, setPerson] = useState({
    firstName: 'Barbara',
    lastName: 'Hepworth',
    email: 'bhepworth@sculpture.com'
  });

  function handleChange(e) {
    setPerson({
      ...person,
      [e.target.name]: e.target.value
    });
  }

  return (
    <>
      <label>
        First name:
        <input
          name="firstName"
          // 속성 name을 이용하여 target.name 접근을 가능하게 한다.
          value={person.firstName}
          onChange={handleChange}
        />
      </label>
      <label>
        Last name:
        <input
          name="lastName"
          value={person.lastName}
          onChange={handleChange}
        />
      </label>
      <label>
        Email:
        <input
          name="email"
          value={person.email}
          onChange={handleChange}
        />
      </label>
      <p>
        {person.firstName}{' '}
        {person.lastName}{' '}
        ({person.email})
      </p>
    </>
  );
}
```


## Updating a nested object
> 중첩된 객체 업데이트하기

- 중첩된 객체를 업데이트하기위해서는 중첩 객체의 위쪽 위치에서 새로운 객체로 교체하고 그 다음 위치의 중첩 객체를 새 객체로 교체하면서 가장 위쪽까지 복사본을 만들어야 업데이트가 적용된다.

  ```jsx
  const nextArtwork = { ...person.artwork, city: 'New Delhi' };
  const nextPerson = { ...person, artwork: nextArtwork };
  setPerson(nextPerson);
  ```
  ```jsx
  setPerson({
    ...person,
    artwork: {
    // 대체할 artwork를 제외한 다른 필드를 복사합니다.
      ...person.artwork,
      city: 'New Delhi'
    // New Delhi'는 덮어씌운 채로 나머지 artwork 필드를 복사합니다!
    }
  });
  ```
  ```jsx
  const handleAddressChange = (e) => {
    setPerson({
      ...person,
      address: {
        // 중첩된 객체의 내부 필드를 복사한다.
        ...person.address,
        [e.target.name]: e.target.value // 덮어씌운다
      }
    });
  }
  ```



## Write concise update logic with Immer
> Immer로 간결한 업데이트 로직 작성


- `Immer`는 불변성을 유지하면서 객체를 업데이트하는 라이브러리로 `Immer`의 `draft`는 프록시라는 특수한 유형의 객체로 `draft`의 변경된 부분을 파악하고 변경된 부분만 편집하여 새로운 객체를 반환한다. 전개구문을 사용하지 않고 객체를 변이하는 것처럼 작성할 수 있다.


### React에서 state 변이를 권장하지 않는 이유

- 렌더링사이 `state`가 어떻게 변경되었는지 명확하게 확인할 수 있다.
- `state`가 동일한 경우 렌더링을 건너뛰고 이전 결과를 재사용할 수 있다.


