# JSX란

React에서 렌더링 로직과 UI로직이 관련 작업을 할 때 시각적으로 보기 좋게 표시하는 Javascript에 XML을 추가한 확장한 문법이다. 브라우저에서 실행하기 전에 바벨을 사용하여 일반 자바스크립트 형태의 코드로 변환된다. 리액트를 사용할 때 JSX문법을 이용하면 하나의 파일에 Javascript 와 HTML 을 동시에 작성할 수 있어 편리하다.

## JSX에 표현식 포함하는 방법

`{ }` 중괄호로 감싸 JSX안에 JS 표현식을 넣어 사용할 수 있다.

```js
function formatName(user) {
  return user.firstName + ' ' + user.lastName;
}

const user = {
  firstName: 'Harper',
  lastName: 'Perez'
};

const element = (
  <h1>
    Hello, {formatName(user)}!
  </h1>
);

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(element);
```

## JSX 속성 정의

속성을 정의할 땐 중괄호나 따옴표 둘 중 하나만 사용하며, JSX는 HTML보단 JS에 가깝기때문에 카멜케이스 프로퍼티 명명규칙을 사용한다.


```js
const element = <img src={user.avatarUrl}></img>
```

## JSX 조건부연산자 사용

if구문과 for루프는 표현식이 아니기 때문에 조건부 렌더링을 하기위해서 JSX 삼항연산자를 사용하거나 JSX코드밖에서 if, for문을 이용하여 조건 처리를 해야한다.

1. **외부에서 조건 연산**
   ```js
   function App() {
    let result = '';
    const check = true;
    if(check) result = <div>완료</div>;
    else result = <div>취소</div>;

    return (
      <>{result}</>
    )
   }
   ```
2. **내부에서 조건부연산**
    ```js
    function App() {
    const check = true;
    return (
      <>
      {check ? (<div>완료</div>) : (<div>취소</div>)}
      </>
    )
   }
    ```
3. **&&(AND)연산자**
    ```js
    function App() {
    const check = true;
    return (
      <>
      {check && (<div>완료</div>)}
      </>
    )
   }
    ```
4. **즉시실행함수**
    ```js
    function App() {
    const check = true;
    return (
      <>
      {
        (() => {
          if(check) return (<div>완료</div>);
          else return (<div>취소</div>);
        })()
      }
      </>
    )
   }
    ```


## Style 적용
1. **css style inline**

   스타일 적용도 객체로 전달해야한다
    ```js
    function App() {
      const style = {
        padding: 20,
        fontSize: '12px',
      }

      return (
        <div style={style}>내용</div>
      )
    }
    ```

    ```js
    function App() {
      let name = "nanii";

      const style = {
        App: {
          textAlign: 'center',
          backgroundColor: 'antiquewhite',
        },
        h2: {
          color: 'white',
        }
      }

      return (
        <div style={style.App}>
          <MyHeader/>
          <header>
            <h2 style={style.h2}>안녕 {name}</h2>
          </header>
          <MyFooter/>
      </div>
      );
    }
    ```



2. **className**


   컴포넌트명과 클래스네임을 일치시켜 더 직관적으로 컴포넌트생성할 수 있다
    ```js
    function DiaryEditor() {
      return (
        <div className="DiaryEditor">내용</div>
      )
    }
    ```

     - 중괄호 내부에는 문자열만 가능하여 className이 여러개인 경우 배열을 넣어 문자열로 형변환 시켜주는 방법도 있다
        ```js
        <button className={["MyButton", `MyButton_${type}`].join(' ')} onClick={onClick}>
        ```