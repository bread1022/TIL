# 3. Your First Component

## Components: UI building blocks
> 컴포넌트 : UI 구성요소

리액트를 사용하면 마크업, CSS, JavaScript 를 재사용이 가능한 UI요소인 컴포넌트로 결합할 수 있고 이미 작성한 컴포넌트들을 재사용하여 구성하면서 개발속도를 향상시킬 수 있는 장점을 가진다.


## Defining a component
> 컴포넌트 정의하기

**React의 컴포넌트란, 마크업으로 뿌릴 수 있는 JavaScript 함수**이다.

<br>

### Step 1: Export the component
> 컴포넌트 내보내기

일반적으로 `export default` 를 사용하여 내보낸다.

### Step 2: Define the function
> 함수 정의하기

일반적으로 컴포넌트명은 대문자로 시작한다.


### Step 3: Add markup
> 마크업 추가하기

JSX란, JavaScript 파일 안에 HTML과 유사한 마크업을 작성할 수 있는 JavaScript를 확장한 문법이다.  
리액트 컴포넌트에서 마크업을 반환할 때 return 문과 같은 라인에 있지않은 경우에는 괄호로 묶어서 반환해야한다.

```js
return <img src="http://img.jpg" alt="이미지">
```


```js
return (
  <div>
    <img src="http://img.jpg" alt="이미지">
  </div>
)
```


## Using a component
> 컴포넌트 사용하기

컴포넌트를 정의했으면 다른 컴포넌트 안에 중첩하여 사용할 수 있다.  
소문자로 시작하는 경우에는 리액트 HTML태그를 가리킨다고 이해할 수 있고,  
대문자로 시작하는 경우에는 리액트 컴포넌트를 사용한다고 이해할 수 있다.

```jsx
<section>
  <h1>제목</h1>
  <img src="http://img.jpg" alt="이미지">
</section>
```
```jsx
<Profile />
```

## Nesting and organizing components
> 컴포넌트 중첩 및 구성

- 컴포넌트는 한번 정의하면 다음 원하는 곳에서 원하는 만큼 여러번 사용이 가능하며,
- 부모 컴포넌트 내부에 자식 컴포넌트를 호출하여 중첩해서 사용이 가능하다.
- 컴포넌트는 다른 컴포넌트를 렌더링할 수 있지만 중첩해서 정의할 수는 없다.
- 자식 컴포넌트에 부모 컴포넌트의 일부 데이터가 필요한 경우, 정의를 중첩하는 게 아니라 props로 데이터를 전달해야한다.

```jsx
export default function Gallery() {
  // 🔴 컴포넌트 내부에서 다른 컴포넌트를 정의할 수 없다 ! ! !
  function Profile() {
    // ...
  }
  // ...
}
```

```jsx
export default function Gallery() {
  // ...
}

// ✅ 컴포넌트는 최상위 레벨에서 정의해야한다 !!!
function Profile() {
  // ...
}
```

### 과제
1. 컴포넌트 내보내기
   - `export default` 를 사용한다!
    ```js
      export default function Profile() {
      return (
        <img
          src="https://i.imgur.com/lICfvbD.jpg"
          alt="Aklilu Lemma"
        />
      );
    }
    ```
2. `return` 문 수정하기
   - return 문과 같은 라인에 마크업을 반환하거나, 다른 라인에 작성하는 경우 `()`괄호로 마크업을 감싸다음 반환해야한다.
    ```js
    export default function Profile() {
      return (
        <img src="https://i.imgur.com/jA8hHMpm.jpg" alt="Katsuko Saruhashi" />
      )
    }
    ```
3. 실수를 찾아내기
   - 일반 HTML 태그와 달리 리액트 컴포넌트는 대문자로 시작한다.
    ```js
    function Profile() {
      return (
        <img
          src="https://i.imgur.com/QIrZWGIs.jpg"
          alt="Alan L. Hart"
        />
      );
    }

    export default function Gallery() {
      return (
        <section>
          <h1>Amazing scientists</h1>
          <Profile />
          <Profile />
          <Profile />
        </section>
      );
    }
    ```