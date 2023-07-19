# Conditional Rendering

리액트에서 조건에 따라 다른 것을 보여줘야할 때 `if문` 대신 `&&`, `||`, `삼항연산자` 같은 JavaScript 문법을 사용해 조건부 렌더링을 한다.

## Conditionally returning JSX
> 조건부로 반환하는 JSX

컴포넌트 내부 요소를 렌더링할 때 조건을 판단하면서 렌더링을 해야할 때,
`if...else`문으로 사용할 수 도 있다. 하지만 이렇게 되면, 조건 하나 때문에 불필요한 JSX 코드를 작성해야할 수 있다.


### Conditionally returning nothing with null
> null을 사용해 조건부로 아무것도 반환하지 않기

아무것도 렌더링하고 싶지 않을 땐 `null`을 반환한다. 하지만 `null` 값을 사용하는 것보다 조건부나 단축표현으로 JSX 를 작성하는 것이 좋다.

```jsx
if (isPacked) {
  return <li className="item">{name} ✔</li>;
}
return <li className="item">{name}</li>;
```


### Conditional (ternary) operator (? : )
> 조건(삼항) 연산자(? : )

삼항연산자를 사용하면 `조건문`을 작성하고 ? `조건문이 true일 때 실행할 표현식` : `조건문이 false일 때 실행할 표현식`을 순서대로 작성한다.


```jsx
return (
  <li className="item">
    {isPacked ? name + ' ✔' : name}
  </li>
);
```



### Logical AND operator (&&)
> 논리 AND 연산자(&&)

리액트 컴포넌트 내에서 조건이 참일 경우 JSX를 렌더링하거나  
그렇지 않으면 아무것도 렌더링하지 않게끔 할 때 자주 사용된다.

```jsx
return (
  <li className="item">
    {name} {isPacked && '✔'}
  </li>
);
```
- `&&` 좌측의 조건이 true이면 오른쪽의 값을 반환하고 false일 경우 아무것도 렌더링하지 않는다.
- *이때 주의해야할 점은 `&&`의 좌측에 숫자를 넣으면 값을 렌더링하기때문에 숫자를 `&&`좌측에 넣지 않아야한다*.
  ```jsx
  messageCount && <p>New messages</p> // 이런식으로 좌측에 숫자가 오는 값을 넣지 말기 !
  ```
  ```jsx
  messageCount > 0 && <p>New messages</p> // 좌측에 불리언값을 넣기 !!!
  !!messageCount && <p>New messages</p> // NOT 논리연산자 (!) 사용하기 !!!
  ```


### Conditionally assigning JSX to a variable
> 변수에 조건부로 JSX 할당하기

`중괄호 {}`를 사용하여 표현식을 값으로 사용할 수 있다.

```jsx
function Item({ name, isPacked }) {
  let itemContent = name;
  if (isPacked) {
    itemContent = name + " ✔"; // 가장 유연한 방법
  }
  return (
    <li className="item">
      {itemContent}
    </li>
  );
}
```