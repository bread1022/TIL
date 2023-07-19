# Rendering Lists

## Rendering data from arrays
> 배열에서 데이터 렌더링하기

데이터 배열을 컴포넌트 배열로 반환하기 위해서는 `map()` 과 `filter()` 같은 메서드를 사용하여 컴포넌트 목록을 렌더링할 수 있다.

#### `map()`
```jsx
export default function List() {
  const listItems = people.map(person => <li>{person}</li>);
  return <ul>{listItems}</ul>;
}
```

#### `filter()`
```jsx
export default function List() {
  const chemists = people.filter(person =>
    person.profession === 'chemist'
  );

  const listItems = chemists.map(person =>
    <li>
      <img
        src={getImageUrl(person)}
        alt={person.name}
      />
      <p>
        <b>{person.name}:</b>
        {' ' + person.profession + ' '}
        known for {person.accomplishment}
      </p>
    </li>
  );

  return <ul>{listItems}</ul>;
}
```

## Keeping list items in order with key
> key로 목록의 항목 순서 유지하기

key는 각 컴포넌트가 배열의 어떤 항목에 해당하는지 리액트에 알려주어 매칭해주는 역할을 한다.  
DOM 트리를 업데이트할 때 *리렌더 과정에서 리스트 내부에서 컴포넌트 항목들을 정확하게 식별하기 위해 (리렌더링 최적화를 위해)* 데이터 배열을 배열 메서드를 사용하여 컴포넌트를 생성할 땐 꼭 key prop을 명시적으로 전달해야한다.


### Rules of keys
> Key 규칙

- key 값은 DB의 데이터 값 중 고유한 key/id를 사용할 수 도 있고
- 로컬에서 생성되고 유지되는 경우 항목을 만들때 증분카운터를 사용하거나 `crypto.randomUUID()`또는 `uuid`같은 패키지를 사용하여 생성할 수 있다.
- key는 형제 컴포넌트간 고유한 값이여야하며 변경되지 않아야한다.
- 주의해야할 점으로
  - 배열의 인덱스를 사용하지 않아야한다 => 렌더링하는 과정중에 항목이 추가, 삭제되어 순서가 변경될 수 있기때문이다.
  - 즉석에서 `Math.random()` 메서드로 난수를 사용하지 않아야한다 => 렌더링될 때마다 key가 일치하지 않기때문에 매번 모든 컴포넌트와 DOM이 다시 생성되기 때문이다.
  - 따라서 데이터에 기반한 안정적인 id를 사용해야한다.



### 과제

#### 방법1
```jsx
import { people } from './data.js';
import { getImageUrl } from './utils.js';

export default function List() {
  const chemists = people.filter(({profession}) => profession === 'chemist');
  const others = people.filter(({profession}) => profession !== 'chemist');

  const listItems = (group) => group.map(person =>
    <li key={person.id}>
      <img
        src={getImageUrl(person)}
        alt={person.name}
      />
      <p>
        <b>{person.name}:</b>
        {' ' + person.profession + ' '}
        known for {person.accomplishment}
      </p>
    </li>
  );

  return (
    <article>
      <h1>Scientists</h1>
      <h3>Chemists</h3>
      <ul>{listItems(chemists)}</ul>
      <h3>everyoneElse</h3>
      <ul>{listItems(others)}</ul>
    </article>
  );
}
```


#### 방법2
```jsx
import { people } from './data.js';
import { getImageUrl } from './utils.js';

function ListSection ({title, people}) {
  return (
    <>
      <h3>{title}</h3>
      <ul>
      {people.map(({ id, name, profession, accomplishment, imageId }) =>
        <li key={id}>
          <img
            src={getImageUrl(person)}
            alt={name}
          />
          <p>
            <b>{name}:</b>
            {' ' + profession + ' '}
            known for {accomplishment}
          </p>
        </li>
      )}
      </ul>
    </>
  )
}

export default function List() {
  const chemists = people.filter(({profession}) => profession === 'chemist');
  const others = people.filter(({profession}) => profession !== 'chemist');

  return (
    <article>
      <h1>Scientists</h1>
      <ListSection
        title="Chemists"
        people={chemists}
      />
      <ListSection
        title="Everyone Else"
        people={everyoneElse}
      />
    </article>
  );
}
```

#### 방법3
```jsx
import { people } from './data.js';
import { getImageUrl } from './utils.js';

let chemists = [];
let everyoneElse = [];
// people이 절대 변하지 않는 배열이라면 컴포넌트 밖으로 옮겨서 만들어도 상관없다! (배열을 어떻게 생성했는지는 중요하지 않음)
people.forEach(person => {
  if (person.profession === 'chemist') {
    chemists.push(person);
  } else {
    everyoneElse.push(person);
  }
});

function ListSection({ title, people }) {
  return (
    <>
      <h2>{title}</h2>
      <ul>
        {people.map(person =>
          <li key={person.id}>
            <img
              src={getImageUrl(person)}
              alt={person.name}
            />
            <p>
              <b>{person.name}:</b>
              {' ' + person.profession + ' '}
              known for {person.accomplishment}
            </p>
          </li>
        )}
      </ul>
    </>
  );
}

export default function List() {
  return (
    <article>
      <h1>Scientists</h1>
      <ListSection
        title="Chemists"
        people={chemists}
      />
      <ListSection
        title="Everyone Else"
        people={everyoneElse}
      />
    </article>
  );
}
```