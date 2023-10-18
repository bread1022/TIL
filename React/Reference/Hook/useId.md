# useId

> 접근성 속성에 전달할 수 있는 고유 ID를 생성하는 React Hook

```js
import { useId } from 'react';

const passwordId = useId();
```

컴포넌트의 최상단에서 호출하여 고유한 ID를 생성하고 HTML 접근성 속성에 전달할 수 있다.  
하드코딩으로 고유ID를 생성했을 때 발생할 수 있는 문제를 방지할 수 있는 장점을 갖는다.!  
서버렌더링을 하다보면 클라이언트 컴포넌트가 하이드레이션 되는 순서가 서버 HTML이 생성된 순서와 일치하지 않아 id가 일치하지 않을 수 있다.  
하지만 useId를 사용하여 호출하면 하이드레이션이 작동하고 서버와 클라이언트 출력물이 서로 일치하는지 확인을 하며,  
리액트 내부에서 useId는 호출한 컴포넌트의 부모 경로에서 생성되기때문에 클라이언트와 서버 트리가 동일하다면 렌더링 순서와 상관없이 부모 경로가 일치하고, useId도 일치한다.


## Parameters

- 매개변수 없음

## Returns

- 특정 컴포넌트 내 특정 useId 와 관련된 고유 ID를 반환한다.

### 접근성 속성에 전달할 수 있는 고유 ID

```js
import { useId } from 'react';

function Password () {
  const passwordHintId = useId();

  return (
    <>
      <input type="password" aria-describedby={passwordHintId} />
      // 🌟 id를 지정하여 두 태그가 서로 연관되어 있음을 표시할 수 있다 !
      <p id={passwordHintId}>
    </>
  )
}
```
```js
<label>
  Password:
  <input
    type="password"
    aria-describedby="password-hint"
  />
</label>
<p id="password-hint">
  The password should contain at least 18 characters
</p>
```

```js
export default function Form() {
  const id = useId();
  return (
    <form>
    // ✅ 관련 요소일 경우 prefix 를 붙여서 표시할 수도 있다~
      <label htmlFor={id + '-firstName'}>First Name:</label>
      <input id={id + '-firstName'} type="text" />
      <hr />
      <label htmlFor={id + '-lastName'}>Last Name:</label>
      <input id={id + '-lastName'} type="text" />
    </form>
  );
}
```