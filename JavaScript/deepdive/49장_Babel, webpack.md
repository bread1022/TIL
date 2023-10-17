# 49장. Babel, Webpack을 이용한 ES6+/ES.NEXT 개발환경 구축

대부분의 브라우저는 ES6 사양을 지원하지만 IE에서 지원율은 현저히 낮기때문에 IE를 포함한 구형 브라우저에서 최신 ECMAScript 사양을 사용한 코드를 실행시키기 위해서 개발 환경 구축이 필요하다.  
이때 사용하는 도구가 바로 Babel과 Webpack이다.

### Babel : transpilter

자바스크립트 컴파일러로 이전 브라우저 또는 환경에서 ECMAScript2015+사양의 코드를 이전 버전의 JavaScript로 변환하는데 사용된다.

```js
[1,2,3].map((n) => n ** n);
```
- ES6의 화살표함수, ES7 지수연산자를 모든 브라우저에서 사용하기 위해 Babel을 사용하여 ES5사양으로 변환 할 수 있는 것이다.
  ```js
  "use strict";

  [1, 2, 3].map(function (n) {
    return Math.pow(n, n);
  });
  ```


### Webpack : bundler

JavaScript App을 위한 정적 모듈 번들러로 의존 관계에 있는 JavaScript, CSS, Image resource를 하나(또는 여러개의) 파일로 번들링하는 모듈 번들러이다. Webpack을 사용하면 의존 모듈이 하나의 파일로 번들링되므로 별도의 모듈로더가 필요없고 script 태그로 여러개의 js파일을 로드하지 않아도 된다.

