# 20장. Strict mode

## 20.1 strict mode, 엄격모드 란

```js
function foo() {
  x = 10;
}
foo();

console.log(x); // 10
```

ES5부터 도입된 모드로 자바스크립트언어의 문법을 더 엄격히 적용하여 오류를 발생시킬 가능성이 높거나 자바스크립트 엔진의 최적화 작업에 문제를 일으킬 수 있는 코드에 대해 명시적인 에러를 발생시킨다.  
ESLint 같은 린트 도구를 사용하면 정적 분석기능을 통해 소스코드를 실행하기 전 소스코드를 스캔하여 문법적 오류뿐만아니라 잠재적 오류까지 찾아내고 오류의 원인을 리포팅한다. 린트 도구는 strict mode가 제한하는 오류는 물론 코딩 컨벤션을 설정파일형태로 정의하고 강제할 수 있기때문에 더욱 강력한 효과를 얻을 수 있다.


## 20.2 strict mode의 적용

ES6에서 도입된 클래스와 모듈은 기본적으로 strict mode가 적용된다.
strict mode를 적용하려면 전역의 선두 또는 함수 몸체 선두에 use strict 를 추가하여 적용할 수 있다.
- 전역의 선두에 추가하면 스크립트 전체에 strict mode가 적용된다.
  ```js
  'use strict';
  function foo() {
  x = 10; // ReferenceError: x is not defined
  }
  foo();
  ```
- 함수 몸체의 선두에 추가하는 경우, 해당 함수와 중첩 함수에 strict mode가 적용된다.
  ```js
  function foo() {
  'use strict';
  x = 10; // ReferenceError: x is not defined
  }
  foo();
  ```
- 코드의 선두에 'use strict'를 위치시키지않으면 strict mode가 제대로 동작하지 않는다.
  ```js
  function foo() {
  x = 10; // 에러를 발생시키지 않는다.
  'use strict';
  }
  foo();
  ```

#### 전역에 엄격모드를 적용하는 경우

- 전역에 적용한 strict mode 는 스크립트 단위로 적용되므로, 스크립트 단위로 적용된 엄격모드는 다른 스크립트에 영향을 주지않고 해당 스크립트에 한정되어 적용된다.
- strict mode 스크립트와 non-strict mode 스크립트를 혼용하는 것은 오류를 발생시킬 수 있으므로 전역에 strict mode를 적용하는 것은 바람직하지 않다.



#### 함수단위로 엄격모드를 적용하는 경우

- 함수단위로 strict mode를 적용할 수 있어 어떤 함수는 strict mode를 적용하고 어떤 함수는 strict mode를 적용하지 않을 수 있다. 이렇게 되면 모든 함수에 일일이 strict mode를 적용해야할 수 도 있어 번거로운 작업을 유발할 수 있다. strict mode가 적용된 함수가 참조할 함수 외부의 컨텍스트에 strict mode를 적용하지 않는다면 또 문제가 발생할 수 있기때문에 strict mode는 즉시 실행 함수로 감싼 스크립트 단위로 적용하는 것이 바람직하다.


## 20.5 strict mode가 발생시키는 에러

1. 선언하지 않은 변수를 참조한 경우 ReferenceError
2. delete 연산자로 변수, 함수, 매개변수를 삭제한 경우 SyntaxError
3. 중복된 매개변수 이름을 사용한 경우 SyntaxError
4. with문을 사용한 경우 SyntaxError
   ```js
   (function() {
    'use strict';

    // SyntaxError: Strict mode code may not include a with statement
    with({ x: 1 }) {
      console.log(x);
    }
   }());
   ```
   - with문은 전달된 객체를 스코프 체인에 추가한다. with문은 동일한 객체의 프로퍼티를 반복해서 사용할 때 객체 이름을 생략할 수 있어서 코드가 간단해지는 효과가 있지만 가독성이 나쁘기때문에 사용하지 않는 게 좋다.


## 20.6 strict mode적용에 의한 변화

#### 일반함수의 this

strict mode에서 함수를 일반 함수로서 호출하면  
생성자 함수가 아닌 일반 함수 내부에서는 this를 사용할 필요가 없기때문에  
this에 `undefined`가 바인딩된다.  
(이때 에러는 발생하지 않는다. 만약 strict mode가 아닌경우 window 전역객체가 바인딩된다 )


#### arguments 객체

strict mode에서 매개변수에 전달된 인수를 재할당하여 변경해도 arguments객체에 반영되지 않는다.
```js
  (function (a) {
    'use strict';
    // 매개변수에 전달된 인수를 재할당하여 변경
    a = 2;

    // 변경된 인수가 arguments 객체에 반영되지 않는다.
    console.log(arguments); // { 0: 1, length: 1 }
  }(1));
```