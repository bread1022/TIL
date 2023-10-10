# 46장. async, await

## 제너레이터

> 코드 블록 실행을 중지 (yield) 시켰다가 필요 시점에 재개 (next) 할 수 있는 특수함수

#### 일반 함수와 제너레이터의 차이

1. 함수의 제어권을 함수 호출자에 양도할 수 있다.
   ```js
   function* genDecFunc() {
     yield 1;
   }
   ```
   - `function*`로 함수를 정의하고 하나 이상의 `yield 표현식`을 포함한다.
   - _함수 호출자에게 제어권을 양도하기때문에 필요한 시점에서 함수를 재개할 수 있는 것!_
2. 제너레이터는 함수 호출자와 양방향 통신이 가능하다.
3. 제너레이터 함수를 호출하면 제너레이터 객체를 반환한다.
   - 일반 함수는 코드 블럭을 일괄 실행하고 값을 반환하지만  
     제너레이터함수는 함수 코드를 실행하는 것이 아니라 이터러블이면서 이터레이터인 제너레이터 객체를 반환한다.

### 제너레이터 함수

```js
// 제너레이터 함수 선언문
function* genDecFunc() {
  yield 1;
}

// 제너레이터 함수 표현식
const genExpFunc = function* () {
  yield 1;
};

// 제너레이터 메서드
const obj = {
  *genObjMethod() {
    yield 1;
  },
};

// 제너레이터 클래스 메서드
class MyClass {
  *genClsMethod() {
    yield 1;
  }
}
```

```js
// ❌ 화살표함수로는 정의할 수 없음
const genArrowFunc = * () => {
  yield 1;
}// syntaxError: Unexpected token '*'

function* genFunc() {
  yield 1;
}
// ❌ 생성자함수로 호출할 수 없음
new genFunc(); // TypeError: genFunc is not a constructor
```

### 제너레이터 객체

> `Symbol.iterator`메서드를 상속받는 이터러블이면서 `value`, `done` 프로퍼티를 갖는 이터레이터 리절트 객체를 반환하는 next 메서드를 소유하는 이터레이터

이터러블이면서 `next`메서드를 갖는 이터레이터이고  
일반 이터레이터에 없는 `return`, `throw `메서드도 있다.

- `next` 메서드 호출시, 제너레이터 함수의 `yield` 표현식 까지 코드 블록을 실행하고 일시 중지되며 제어권이 함수호출자에게 양도되고 호출자가 다시 `next`메서드를 호출하면 재개함
  - `value 프로퍼티` 값으로 **yield된 값**을
  - `done 프로퍼티` 값으로 **fase를 갖는 이터레이터 리절트 객체**를 반환
- `return` 메서드 호출시,
  - `value 프로퍼티` 값으로 **인수로 전달된 값**을
  - `done 프로퍼티` 값으로 **true를 갖는 이터레이터 리절트 객체**를 반환
- `throw` 메서드 호출시,
  - `value 프로퍼티` 값으로 인수로 전달받은 에러를 발생시키고 **`undefined`** 를
  - `done 프로퍼티` 값으로 **true를 갖는 이터레이터 리절트 객체**를 반환

<br>

제너레이터 함수를 호출하면 코드 실행이 아니라 이터러블이면서 이터레이터이자 `next`메서드를 갖는 제너레이터 객체를 반환한다. 이 제너레이터 객체의 `next`메서드를 호출하면 제너레이터 함수의 `yield` 표현식까지만 실행하고 일시 중지된다...!!  
`yield` 키워드는 제너레이터 함수의 실행을 일시 중지시키거나 `yield` 키워드 뒤에 오는 표현식의 평가 결과를 제너레이터 함수 호출자에게 반환하는 역할을 한다.

`next` 메서드를 반복호출하면서 `yield` 표현식까지의 실행과 중지를 반복하다가 제너레이터 함수가 끝까지 실행되면 `next` 메서드가 반환하는 이터레이터 리절트 객체의 `value` 프로퍼티에 제너레이터 함수의 반환값이 할당되고 `done` 프로퍼티에는 제너레이터 실행이 끝났다는 의미로 true가 할당된다.

> 따라서 제너레이터함수의 next 메서드와 yield 표현식을 통해 함수 호출자와 제너레이터 함수의 상태를 주고 받으면서 yield 표현식까지 함수를 실행시켜 yield된 값을 꺼내올 수도 있고, next 메서드에 인수를 전달하여 제너레이터 객체에 상태를 할당할 수도 있다.  
> 이러한 특징을 활용하여 비동기 처리를 동기로 표현할 수 있다.

### 제너레이터를 활용하여 비동기 처리

제너레이터 함수의 `next`메서드와 `yield` 표현식을 통해 함수 호출자와 함수 상태를 주고받는 특징을 활용하여 비동기 처리를 동기처럼 표현할 수 있다.  
Promise의 후속처리메서드로 콜백함수를 계속해서 실행하는 방법말고 비동기처리 결과만 반환하도록 구현이 가능하다!

```js
const async = generatorFunc => {
  const generator = generatorFunc(); // 2️⃣

  const onResolved = arg => {
    const result = generator.next(arg); // 5️⃣

    return result.done
      ? result.value  // 9️⃣
      : result.value.then(res -> onResolved(res)); // 7️⃣
  }
  return onResolved; // 3️⃣
}

(async (function* fetchTodo() {  // 1️⃣
  const url = 'https://www.~~./~';
  const res = yield fetch(url); // 6️⃣
  const todo = yield res.json(); // 8️⃣
  console.log(todo);
})()); // 4️⃣
```

<!-- * 다시 읽기  -->

## async, await

제너레이터보다 간단하고 가독성 좋게 비동기 처리를 동기 처리처럼 동작할 수 있는 문법으로  
Promise 기반으로 동작하기때문에 Promise의 후속 처리 메서드 없이 동기 처리처럼 Promise가 처리 결과를 반환하도록 구현할 수 있다.

```js
// 🌟 async를 사용하여 함수를 선언하고
async function fetchTodo() {
  const url = 'https://www.~~./~';
  const res = await fetch(url);
  // 🌟 await 키워드를 사용하여 프로미스가 처리 (settled)될 때까지 대기하다가 처리 결과를 반환
  const todo = await res.json();
  console.log(todo);
}
```

- `async` 함수는 언제나 Promise를 반환한다. (명시적으로 Promise를 생성하지 않아도 암묵적으로 반환값을 resolve하는 Promise를 생성하여 반환)
- `await` 키워드는 반드시 `async` 함수 내부에서 사용해야하고 반드시 Promise앞에서 사용해야한다.
  - Promise가 settled상태가 될 때까지 대기하다가 settled상태가 되면 Promise가 resolve한 처리 결과를 반환한다.
- `async`, `await`을 통해 비동기처리를 동기처럼 할 순 있지만 모든 Promise에 `await`키워드를 사용하면 관련이 없는 작업도 순차적으로 실행되기 때문에 비효율적이다.
  - 따라서 서로 연관이 없이 개별적으로 비동기 처리를 해야될 경우에는 `Promise.all`메서드를 사용하여 병렬로 처리하도록 구현하는 것이 좋다.

### async, await를 통한 에러처리

비동기 작업을 위한 콜백함수에서 발생한 에러는 호출자방향으로 전파된다.  
즉, Call stack의 아래방향인 실행 컨텍스트가 push되기 직전에 push된 실행 컨텍스트 방향으로 전파되는건데! 비동기 함수의 콜백함수를 호출한 것은 비동기 함수가 아니기 때문에 `try...catch`문으로 에러를 처리할 수 없다.  
하지만 `async` 함수는 호출자가 명확하기 때문에 `async` 함수를 호출한 함수에서 `try...catch`문으로 에러를 처리할 수 있다!!!! async 함수 내에서 catch문을 사용해 에러 처리를 하지않으면 async 함수는 발생한 에러를 reject하는 Promise를 반환하므로 후속처리메서드를 사용할 수도 있다.

```js
async function foo() {
  try {
    const url = 'https://www.~~./~';
    const res = await fetch(url);
    const data = await res.json();
    console.log(data);
  } catch (err) {
    // ✅ try 코드 내에서 발생한 비동기 처리 error를 catch문으로 잡을 수 있음!
    console.error(err); // TypeError: Failed to fetch
  }
}

foo();
```

```js
async function foo() {
  const url = 'https://www.~~./~';
  const res = await fetch(url);
  const data = await res.json();
  // ✅ async 키워드를 사용한 함수는 성공시 resolve하는 Promise를 반환하고,
  // 에러가 발생하면 reject하는 Promise를 반환하기 때문에
  return data;
}

foo()
  .then((res) => console.log(res))
  // ✅ 후속처리 메서드를 사용하여 에러를 캐치할 수 있다!
  .catch((err) => console.error(err)); // TypeError: Failed to fetch
```
