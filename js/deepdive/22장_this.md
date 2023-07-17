# 22장. this


## 22.1 this 키워드
> this란,
> 자신이 속한 객체 또는 생성할 인스턴스의 프로퍼티나 메서드를 참조할 수 있는 자기 참조 변수를 말한다.

- `this`는 자바스크립트 엔진에 의해 암묵적 생성되며 코드 어디서든 참조할 수 있다. (전역에서, 함수내부에서 참조가능하다)
- 자바스크립트의 this는 함수가 호출되는 방식에 따라 `this`가 가리키는 값, `this 바인딩`이 동적으로 결정된다. (상황에 따라 가리키는 대상이 다르다)
  - **`this 바인딩`** : 식별자와 값을 연결하는 과정으로 `this`와 `this`가 가리킬 객체를 바인딩하는 것을 말한다.

## 22.2 함수 호출 방식과 this 바인딩

- `this 바인딩`은 함수 호출 시점에 결정된다.
- **함수 호출 방식에 따른 this 바인딩**
  1. 일반 함수 호출 = 전역객체
  2. 메서드 호출 = 메서드를 호출한 객체
  3. 생성자 함수 호출 = 생성자 함수가 생성할 인스턴스
  4. `Function.prototype.apply`/`call`/`bind`에 의한 간접 호출 = 메서드 첫번째 인수로 전달한 객체

```js
// 1. 일반 함수 호출
// foo 함수 내부의 this는 전역 객체 window를 가리킨다.
foo(); // window

// 2. 메서드 호출 (foo 함수를 프로퍼티 값으로 할당하여 호출)
// foo 함수 내부의 this는 메서드를 호출한 객체 obj를 가리킨다.
const obj = { foo };
obj.foo(); // obj

// 3. 생성자 함수 호출 (foo 함수를 new 연산자와 함께 생성자 함수로 호출)
// foo 함수 내부의 this는 생성자 함수가 생성한 인스턴스를 가리킨다.
new foo(); // foo {}

// 4. Function.prototype.apply/call/bind 메서드에 의한 간접 호출
// foo 함수 내부의 this는 인수에 의해 결정된다.
const bar = { name: 'bar' };

foo.call(bar);   // bar
foo.apply(bar);  // bar
foo.bind(bar)(); // bar
```

## 22.2.1 일반 함수 호출

- 기본적으로 `this`에는 전역객체가 바인딩된다.
- 일반 함수로 호출된 모든 함수(중첩함수, 콜백함수 포함) 내부의 `this`에는 전역객체가 바인딩된다.
- ```js
  function foo() {
    console.log('foo': this); // window 전역 객체가 바인딩 됨
    function bar() {
      console.log('bar': this); // window
    }
    bar();
  }
  foo();
  ```
- strict mode가 적용된 일반 함수 내부의 `this`의 경우 `undefined`가 바인딩된다.
  - ```js
    function foo() {
      'use strict';

      console.log('foo': this); // undefined
      function bar() {
          console.log('bar': this); // undefined
      }
      bar();
    }
    foo();
    ```

#### this바인딩을 메서드의 this와 일치시키는 방법
1. `that`
    ```js
    var value = 1;
    const obj = {
      value: 100,
      foo() {
        // this 바인딩(obj)을 변수 that에 할당한다.
        const that = this;
        // 콜백 함수 내부에서 this 대신 that을 참조한다.
        setTimeout(function () {
          console.log(that.value); // 100
        }, 100);
      }
    };
    obj.foo();
    ```
2. `화살표함수`
    ```js
    var value = 1;

    const obj = {
      value: 100,
      foo() {
        // 화살표 함수 내부의 this는 상위 스코프의 this를 가리킨다.
        setTimeout(() => console.log(this.value), 100); // 100
      }
    };
    obj.foo();
    ```
3. `Function.prototype.apply`
4. `Function.prototype.call`
5. `Function.prototype.bind`
    ```js
    var value = 1;

    const obj = {
      value: 100,
      foo() {
        // 콜백 함수에 명시적으로 this를 바인딩한다.
        setTimeout(function () {
          console.log(this.value); // 100
        }.bind(this), 100);
      }
    };
    obj.foo();
    ```


## 22.2.2 메서드 호출

- 메서드 내부의 `this`에는 메서드를 호출한 객체, 메서드 앞 `마침표(.)연산자` 앞에 있는 객체가 바인딩된다.
- 메서드 내부의 `this`는 메서드를 소유한 객체가 아닌 **메서드를 호출한 객체에 바인딩**된다.
  ```js
  const person = {
    name: 'Lee',
    getName() {
      // 메서드 내부의 this는 메서드를 호출한 객체에 바인딩된다.
      return this.name;
    }
  };
  // 메서드 getName을 호출한 객체는 person이다.
  console.log(person.getName()); // Lee
  ```

## 22.2.3 생성자 함수 호출

- 생성자 함수 내부의 `this`에는 생성자 함수가 생성할 인스턴스가 바인딩된다.
- `new 연산자`와 함께 생성자 함수를 호출해야 생성자 함수로 동작하고, 그렇지 않으면 일반 함수로 호출되기때문에 일반 함수로 호출하게 되면 그때의 `this`는 전역객체를 가리킨다.
  ```js
  // 생성자 함수
  function Circle(radius) {
    // 생성자 함수 내부의 this는 생성자 함수가 생성할 인스턴스를 가리킨다.
    this.radius = radius;
    this.getDiameter = function () {
      return 2 * this.radius;
    };
  }

  // 반지름이 5인 Circle 객체를 생성
  const circle1 = new Circle(5);
  console.log(circle1.getDiameter()); // 10

  // new 연산자와 함께 호출하지 않으면 생성자 함수로 동작하지 않는다. 즉, 일반적인 함수의 호출이다.
  const circle3 = Circle(15);
  // 일반 함수로 호출된 Circle에는 반환문이 없으므로 암묵적으로 undefined를 반환한다.
  console.log(circle3); // undefined
  // 일반 함수로 호출된 Circle 내부의 this는 전역 객체를 가리킨다.
  console.log(radius); // 15
  ```

## 22.2.4 apply / call / bind 메서드에 의한 간접 호출

- `Function.prototype`의 메서드인 `apply`, `call`, `bind` 메서드는 모든 함수가 상속받아 사용이 가능하다.
- 이 메서드들은 `this`로 사용할 객체와 인수 리스트를 인수로 전달받아 함수를 호출한다.
- **`Function.prototype.apply(thisArg[, argArray])`**
  - 첫번째 인수로 `this`로 사용할 객체
  - 두번째 인수로 호출할 함수의 인수를 배열로 묶어 전달
- **`Function.prototype.call(thisArg[, arg1[, arg2[, ...]]])`**
  - 첫번째 인수로 `this`로 사용할 객체
  - 두번째 인수로 호출할 함수의 인수를 쉼표로 구분한 리스트 형식으로 전달
- `apply`와 `call`메서드는 함수를 호출하면서 첫번째 인수로 this로 사용할 객체를 전달하면서 함수를 호출한다는 공통점을 가진다.
- **`Function.prototype.bind(thisArg)`**
  - apply와 call과 달리 함수를 호출하지 않고
  - 첫번째 인수로 전달한 값으로 this바인딩이 교체된 함수를 새롭게 생성해 반환한다.
  - bind메서드를 사용하면 this와 메서드 내부 중첩 함수, 콜백함수의 this 불일치 문제를 해결할 수 있다.
  - ```js
    const person = {
      name: 'Lee',
      foo(callback) { // Hi Lee. -> this는 foo 메서드를 호출한 객체 = person 객체를 가리킴
        setTimeout(callback, 100);
      }
    };
    person.foo(function() {
      console.log(`Hi ${this.name}.`); // Hi . -> 일반 함수로 호출된 콜백 함수 내부의 this.name은 브라우저 환경에서 window.name과 같다. = 전역객체를 가리킴
    })
    ```