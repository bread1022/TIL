# JavaScript in JSX with Curly Braces

JSX를 사용하면 JavaScript 파일 내에 마크업을 작성하여 렌더링과 관련된 로직을 콘텐츠 내부에 유지할 수 있다. 마크업 내부에 JavaScript 로직을 추가하거나 동적 프로퍼티를 참조하고싶을 경우에 JSX에서 `중괄호 {}`를 사용하여 JavaScript 로직과 변수를 마크업으로 가져올 수 있다.

### JavaScript 문법 사용하는 방법

1. 따옴표로 문자열을 전달하는 방법
   - `작은 따옴표 ''` 또는 `큰 따옴표 ""`를 사용하여 JSX에 문자열 속성을 전달할 수 있다.
2. 중괄호를 사용하여 JSX 내에서 JavaScript 변수를 참조하는 방법
   - 변수에 문자열을 할당한 뒤 중괄호에 `{변수}`를 참조하여 속성을 전달할 수 있다.
   - 중괄호를 사용하면 마크업에서 바로 JavaScript 로 동작한다.
   - ```jsx
      export default function Avatar() {
        const avatar = 'https://i.imgur.com/7vQD0fPs.jpg';
        const description = 'Gregorio Y. Zara';
        return (
          <img
            className="avatar"
            src={avatar}
            alt={description}
          />
        );
      }
      ```
3. 중괄호를 사용하여 JSX 내에서 JavaScript 함수를 호출하는 방법
   - 중괄호 안에서 함수를 호출하여 작성하면 JavaScript 표현식이 동작한다.


### Where to use curly braces
> 중괄호 사용 위치

1. JSX 태그 안에서 직접 **텍스트를 사용**하기 위해서
    ```jsx
    <h2>{name}'s BirthDay </h2>
    ```

2. `=` 기호 뒤에 오는 **속성을 전달**하기 위해서
    ```jsx
    src={avatar} // avatar 라는 변수를 전달
    ```
    ```jsx
    src="{avatar}" // "{avatar}" 라는 문자열을 전달
    ```


## Using “double curlies”: CSS and other objects in JSX
> “이중 중괄호” 사용: JSX 내에서의 CSS 및 다른 객체

- JSX에서 JS 객체를 전달하기위해선 다른 중괄호 쌍으로 객체를 감싸야한다. 객체는 중괄호로 표시할 수 있기 때문에 JSX 내에서 다른 객체를 사용하기 위해서는 `이중 중괄호 {{...}}` 를 사용한다.
- CSS 스타일을 인라인으로 적용하기위해서도 style attribute 에 객체를 전달해서 사용한다.
  - 인라인 CSS style property 는 카멜케이스로 작성한다.
  - `<ul style={{ backgroundColor: 'black }}`


```jsx
export default function TodoList() {
  return (
    <ul style={{
      backgroundColor: 'black',
      color: 'pink'
    }}>
      <li>Improve the videophone</li>
      <li>Prepare aeronautics lectures</li>
      <li>Work on the alcohol-fuelled engine</li>
    </ul>
  );
}
```



## More fun with JavaScript objects and curly braces
> JavaScript 객체와 중괄호로 더 재미있게 즐기기

여러 표현식을 하나의 객체로 생성하면 컴포넌트에서 하나의 객체로 값을 사용할 수 있다.

```jsx
const person = {
  name: 'Gregorio Y. Zara',
  theme: {
    backgroundColor: 'black',
    color: 'pink'
  }
}

<div style={person.theme}>
  <h1>{person.name}'s Todos</h1>
```