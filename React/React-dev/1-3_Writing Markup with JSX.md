# Writing Markup with JSX

리액트 컴포넌트는 리액트가 브라우저에 마크업을 렌더링할 수 있는 JavaScript 함수로 JSX라는 구문 확장자를 사용하여 마크업을 표현한다.
  - JSX : 구문 확장
  - 리액트 : 자바스크립트 라이브러리



## JSX: Putting markup into JavaScript
> JSX: JavaScript에 마크업 넣기

<p align="center"><img src="https://react-ko.dev/_next/image?url=%2Fimages%2Fdocs%2Fdiagrams%2Fwriting_jsx_sidebar.png&w=750&q=75" /></p>

이전까지 바닐라JS로 생성한 앱은 HTML로 컨텐츠를, CSS로 디자인을, JavaScript로 로직을 작성되었고, JavaScript가 동작하는동안 HTML안에서는 컨텐츠가 마크업되었다.  
하지만 *웹이 인터렉티브해지면서 로직이 컨텐츠를 결정하는 경우가 많아졌고 JavaScript가 HTML을 담당하게 되면서 리액트에서 렌더링 로직과 마크업이 같은 위치의 컴포넌트에 함께 존재하게 되었다*.

- 예를 들어, 버튼의 렌더링 로직과 마크업이 함께 존재하면 서로 동기화 상태를 유지할 수 있다. 반대로 버튼과 사이드바 같이 서로 관련이 없는 경우에는 서로 분리되어 있으므로 개별적으로 변경하는 것이 더 안전하다.

## The Rules of JSX
> JSX 규칙


1. 컴포넌트에서 element를 여러개 반환하는 경우 하나의 부모 태그로 감싸야한다.
   - 하나의 태그로 감싸야되는 이유 : JSX가 내부적으로는 JavaScript 객체로 변환되기때문
2. 태그를 명시적으로 닫아야한다.
   - ex. `<img />`, `<li> ... </li>`
3. 네이밍은 보통 카멜 케이스로 작성한다.
   - JSX 는 JavaScript 로 바뀌고 JSX로 작성된 attribute 는 JavaScript 객체의 키가 된다. 예약어를 사용할 수 없기때문에 HTML과 SVG attribute 는 카멜 케이스로 작성해야한다.
   - ex. `stroke-width` => `strokeWidth`, `class` => `className`, `<label for="">` => `<label htmlFor="">`



### 참고

기존 HTML 을 JSX 로 변환해주는 [사이트](https://transform.tools/html-to-jsx) 도 있으니 참고해본다.