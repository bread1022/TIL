# Importing and Exporting Components

## The root component file
> 루트 컴포넌트 파일

`CRA(Create React App)`에서는 `App.js`라는 root컴포넌트 파일이 존재하며, 앱 전체가 `src/App.js`에서 실행된다. 설정에 따라 root컴포넌트가 다른 파일에 위치할 수 도 있다.


## Exporting and importing a component
> 컴포넌트 import 및 export하기

보통 한 파일에는 하나의 컴포넌트만 두고 파일명과 선언하는 컴포넌트명을 동일하게 네이밍하는 것이 좋다.

- 재사용을 하기 위한 컴포넌트는 `export default` 로 컴포넌트를 내보낸다.
- 예외적으로 한 파일에서 여러 컴포넌트를 선언해서 사용하는 것이 좋은 경우도 있다.
  1. 오직 한 컴포넌트 내에서만 사용할, 종속성이 높은 내부 컴포넌트
  2. 하나의 묶음으로 처리하는 것이 좋을 경우 (ex. Modal)

  ```jsx
  // Modal.jsx
  export default function Modal() {
    return (<div>모달 컴포넌트</div>)
  }
  Modal.Header = function () {}
  Modal.Content = function () {}
  Modal.Footer = function () {}
  ```
  - 함수는 각각의 컴포넌트지만, 구분되는 요소를 분리하여 사용할 수 도 있다.
  ```jsx
  import { Modal } from './Modal';
  ...
  <>
    <Modal.Header />
    <Modal.Content />
    <Modal.Footer />
  </>
  ```
  ```jsx
  // AutoComplete.jsx
  export default function AutoComplete() {
    return (<div>자동완성컴포넌트</div>)
  }
  AutoComplete.SearchBox = function () {}
  AutoComplete.Tooltip = function () {}
  ```
  - 하나의 컴포넌트에서 파생된 컴포넌트를 분리하여 생성한 뒤 각각을 불러 사용할 수 있다 !


<p align="center"><img src="https://react-ko.dev/images/docs/illustrations/i_import-export.svg" alt="export 비교사진"></p>

Default export | named exports
---|---
하나의 파일에서 default export는 하나만 존재해야한다. | 하나의 파일에서 여러개 존재할 수 있다.
사용시 `import Button from './Button.js'` | 사용시 중괄호로 감싸야 한다. `import { Button } from './Button.js'`
import 시 다른 이름으로 값을 가져올 수 있다. | 양쪽 파일에서 사용하고자하는 값의 이름이 같아야한다. (=> named import)
보편적으로 한 파일에서 하나의 컴포넌트만 export할 때 `default export` 방식을 사용 | 여러 컴포넌트를 export 하는 경우 `named export` 방식을 사용