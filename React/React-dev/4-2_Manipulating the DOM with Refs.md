# Manipulating the DOM with Refs


React는 렌더링 출력과 일치하도록 DOM을 자동 업데이트하므로 컴포넌트가 조작할 필요가 거의 없다.  
하지만 때때로 focus, scroll 등 DOM요소에 접근해야 할 때 `ref`를 사용한다.


## Getting a ref to the node

```jsx
import { useRef } from 'react';

const MyComponent = () => {
  const inputRef = useRef(null);
  // inputRef.current값은 null로 초기화된다.

  const handleClick = () => {
    inputRef.current.focus();
  };

  return (
    <div>
      // DOM 노드를 가져올 JSX 요소에 ref 어트리뷰트를 추가함
      <input type="text" ref={inputRef} />
      <button onClick={handleClick}>Focus</button>
    </div>
  );
};
```
- JSX태그의 `ref`속성에 `ref`객체를 할당하여 DOM 노드를 가져올 수 있다.
- 초기의 `ref.current`값은 initialValue로 설정된다.
- React가 DOM 노드를 생성하면  
  **`ref.current`의 값으로 지정된 노드에 대한 참조를 할당**한다.
  (`ref.current` = DOM 노드에 대한 참조)
- 이벤트 핸들러에서 `ref.current`로 부터 DOM 노드에 접근할 수 있고,  
  DOM노드에 정의된 빌트인 [브라우저 API](https://developer.mozilla.org/ko/docs/Web/API/Element)로 DOM 노드를 조작할 수 있다.


### Example: Scrolling to an element

- 컴포넌트에서 여러개의 `ref`를 사용할 수 있다.


#### `scrollIntoView()`
- 선택한 element가 브라우저의 보이는 영역에 위치하도록 스크롤을 이동시킨다.  
- 매개변수(alignToTop): `true` - 맨 위에, `false` - 맨 아래에
- 매개변수(Option)
  - `behavior`: 움직이는 애니메이션 지정 옵션
  - `block`: 위아래
  - `inline`: 좌우

```jsx
import { useRef } from 'react';

export default function CatFriends() {
  const firstCatRef = useRef(null);
  const secondCatRef = useRef(null);
  const thirdCatRef = useRef(null);

  function handleScrollToFirstCat() {
    firstCatRef.current.scrollIntoView({
      behavior: 'smooth',
      block: 'nearest',
      inline: 'center'
    });
  }

  function handleScrollToSecondCat() {
    secondCatRef.current.scrollIntoView({
      behavior: 'smooth',
      block: 'nearest',
      inline: 'center'
    });
  }

  function handleScrollToThirdCat() {
    thirdCatRef.current.scrollIntoView({
      behavior: 'smooth',
      block: 'nearest',
      inline: 'center'
    });
  }

  // 함수가 중복되는 문제점이 있음
  return (
    <>
      <nav>
        <button onClick={handleScrollToFirstCat}>
          Tom
        </button>
        <button onClick={handleScrollToSecondCat}>
          Maru
        </button>
        <button onClick={handleScrollToThirdCat}>
          Jellylorum
        </button>
      </nav>
      <div>
        <ul>
          <li>
            <img
              src="첫번째사진"
              ref={firstCatRef}
            />
          </li>
          <li>
            <img
              src="두번째사진"
              ref={secondCatRef}
            />
          </li>
          <li>
            <img
              src="세번째사진"
              ref={thirdCatRef}
            />
          </li>
        </ul>
      </div>
    </>
  );
}
```

```jsx
import { useRef } from 'react';

const Carousel = () => {
  const carouselRef = useRef(null);
  const prevButtonRef = useRef(null);
  const nextButtonRef = useRef(null);

  const scrollToNext = () => {
    carouselRef.current.scrollLeft += 200;
  };

  const scrollToPrev = () => {
    carouselRef.current.scrollLeft -= 200;
  };

  return (
    <div>
      <button onClick={scrollToPrev} ref={prevButtonRef}>
        Prev
      </button>
      <div className="carousel"
        ref={carouselRef}
        style={{ overflow: 'scroll', width: '400px' }}
      >
      <button onClick={scrollToNext} ref={nextButtonRef}>
        Next
      </button>
    </div>
  );
}
```


### How to manage a list of refs using a ref callback
> ref콜백을 사용하여 refs 목록을 관리하는 방법

- `useRef` hook은 최상위 컴포넌트에서만 사용이 가능하다. (반복문 or map()메서드 내부에서 사용불가)
- **반복문 내부에서 `ref`를 사용하려면 `ref콜백(ref속성에 함수를 전달)`을 사용**해야 한다.
- React는 `ref`를 설정할 때 DOM 노드를 지정하고,  
  지울땐 `null`로 `ref`콜백을 호출한다.

```jsx
import { useRef } from 'react';

export default function CatFriends() {
  const itemsRef = useRef(null);
  // HTML element 하나를 위해 생성한 게 아니라
  // HTML elements 여러개를 객체를 가지기 위해 생성한 것임!!!

  function scrollToId(itemId) {
    const map = getMap(); // 삭제, 추가 하기 쉽게 하려고 맵 객체 사용
    const node = map.get(itemId);
    node.scrollIntoView({
      behavior: 'smooth',
      block: 'nearest',
      inline: 'center'
    });
  }

  function getMap() {
    if (!itemsRef.current) {
      itemsRef.current = new Map();
    }
    return itemsRef.current;
  }

  return (
    <>
      <nav>
        <button onClick={() => scrollToId(0)}>
          Tom
        </button>
        <button onClick={() => scrollToId(5)}>
          Maru
        </button>
        <button onClick={() => scrollToId(9)}>
          Jellylorum
        </button>
      </nav>
      <div>
        <ul>
          {catList.map(cat => (
            // HTML LI element node 에 접근할 수 있게됨
            <li
              key={cat.id}
              ref={(node) => {
                const map = getMap();
                if (node) map.set(cat.id, node);
                else map.delete(cat.id);
              }}
            >
              <img
                src={cat.imageUrl}
                alt={'Cat #' + cat.id}
              />
            </li>
          ))}
        </ul>
      </div>
    </>
  );
}
```


## Accessing another component’s DOM nodes

#### `React.forwardRef()`

- 함수 컴포넌트에서 `ref`를 전달할 수 있게 해주는 함수
- 첫번째 인수에 props,
- 두번째 인수에 `ref`
- 부모컴포넌트에서 자식 중 하나에게 `ref`를 전달할 수 있게한다.
    ```jsx
    import { forwardRef, useRef } from 'react';

    const MyInput = forwardRef((props, ref) => {
    // forwardRef 를 사용하여 자식 컴포넌트에 ref를 전달하도록함
      return <input {...props} ref={ref} />;
    });

    export default function Form() {
      const inputRef = useRef(null);

      function handleClick() {
        inputRef.current.focus();
      }

      return (
        <>
          <MyInput ref={inputRef} />
          <button onClick={handleClick}>
            Focus the input
          </button>
        </>
      );
    }
    ```
    ```js
    function MyInput(props) {
      return <input {...props} />;
    }

    <MyInput ref={inputRef} /> // ❌ Error: ref는 Prop에 포함되지 않음

    function MyInput2(props) {
      return <input ref={props.myRef} />;
    }

    <MyInput2 myRef={inputRef}> // ⭕️ 변수명을 다르게하면 Prop으로 사용이 가능하긴함
    ```


### Exposing a subset of the API with an imperative handle

#### `useImperativeHandle()`

- 첫번째 인자로 `ref`
- 두번째 인자로 실행해서 객체를 반환하는 함수
- 반환값 : `ref`에 지정할 메서드만 사용가능한 요소(지정한 `ref`에는 다른 메서드의 접근은 제한하고, 남겨둔 메서드만 접근가능하게 함)

  ```jsx
  import {
    forwardRef,
    useRef,
    useImperativeHandle
  } from 'react';

  const MyInput = forwardRef((props, ref) => {
    const realInputRef = useRef(null);
    useImperativeHandle(ref, () => ({
      focus() {
        realInputRef.current.focus();
      },
    }));
    // focus메서드만 사용가능한 input element를 반환
    return <input {...props} ref={realInputRef} />;
  });
  ```


### `useRef()` hook

- 렌더링 과정 중에는 `ref`에 접근하면 안된다.  
  첫번째 렌더링 중에는 DOM노드가 생성되지 않았으므로 `ref.current = null`이 반환될 수 있기때문이다.
- React는 커밋 과정 동안 `ref.current` 의 초기값 `null`을 설정하였다가 DOM이 업데이트 된 직후 지정된 DOM 노드를 할당한다.
- 따라서 `ref`는 컴포넌트가 마운트된 이후 접근해야하므로  
  보통 이벤트 핸들러로 `ref`에 접근한다.  
  이때 `ref`를 조작해야하지만 수행할 이벤트가 없다면 그때 `useEffect` hook을 사용한다.
  - 시점이 약간 늦을 수 있음
  - 디버깅하고 조작하는 처리가 복잡할 수 있음
- `ref`를 사용하면 좋은 때
  1. DOM 노드에 접근해야 할 때 (focus, scroll 등)  
    (이때 node를 직접 수정하려고하면 React와 충돌이 날 수 있으므로 React가 관리하는 DOM노드를 직접 변경하지 말아야함!!, React가 업데이트 할 이유가 없는 DOM이면 상관없음..!)
  2. React가 노출하지 않는 브라우저 API호출할 때


------


#### 과제1 (`video play()/pause()`)

```jsx
import { useState, useRef } from 'react';

export default function VideoPlayer() {
  const [isPlaying, setIsPlaying] = useState(false);
  const videoRef = useRef(null);

  function handleVideoButtonClick() {
    setIsPlaying((pre) => !pre);
    // video tag의 메서드를 재생하거나 정지시키도록함
    if(!isPlaying) videoRef.current.play();
    else videoRef.current.pause();
  }

  return (
    <>
      <button onClick={handleVideoButtonClick}>
        {isPlaying ? 'Pause' : 'Play'}
      </button>
      <video
        width="250"
        ref={ref}
        // 브라우저 빌트인 컨트롤 처리
        onPlay={() => setIsPlaying(true)}
        onPause={() => setIsPlaying(false)}
      >
        <source
          src="https://interactive-examples.mdn.mozilla.net/media/cc0-videos/flower.mp4"
          type="video/mp4"
        />
      </video>
    </>
  )
}
```

-------

#### 과제3 (carousel🌟)

```jsx
import { useRef, useState } from 'react';
import { flushSync } from 'react-dom';

export default function CatFriends() {
  const selectedRef = useRef(null);
  const [index, setIndex] = useState(0);

  return (
    <>
      <nav>
        <button onClick={() => {
          flushSync(() => {
            if (index < catList.length - 1) {
              setIndex(index + 1);
            } else {
              setIndex(0);
            }
          });
          selectedRef.current.scrollIntoView({
            behavior: 'smooth',
            block: 'nearest',
            inline: 'center'
          });
        }}>
          Next
        </button>
      </nav>
      <div>
        <ul>
          {catList.map((cat, i) => (
            <li
              key={cat.id}
              // 조건부로 선택된 li element에 ref를 지정함
              ref={index === i ?
                selectedRef :
                null
              }
            >
              <img
                className={
                  index === i ?
                    'active'
                    : ''
                }
                src={cat.imageUrl}
                alt={'Cat #' + cat.id}
              />
            </li>
          ))}
        </ul>
      </div>
    </>
  );
}
```