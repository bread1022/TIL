# 45장. Promise

비동기 처리를 위한 패턴 중 하나로 콜백함수의 단점을 보완하고, 비동기 처리 시점을 명확하게 표현할 수 있는 방법

## 비동기처리 콜백 패턴의 단점

비동기 함수를 호출하면 함수 내부의 비동기 코드의 완료와 상관없이 호출 후 즉시 종료된다.  
따라서, 비동기 함수 내부의 비동기로 동작하는 코드에서 처리 결과를 외부로 반환하거나 상위 스코프의 변수에 할당하면 기대한 대로 동작하지 않을 것이다.

#### 비동기 함수 실행 순서

1. 비동기 함수가 호출되면 함수 코드 평가 과정에서 비동기 함수 실행 컨텍스트가 생성되고
2. 실행컨텍스트 스택에 push된다.
3. 함수 코드 실행 과정에서 이벤트핸들러 프로퍼티에 이벤트 핸들러는 바인딩되고
4. 비동기 함수가 종료되면 실행 컨텍스트 스택에서 pop된다.
5. 이벤트가 발생되면 이벤트 루프에 의해 이벤트핸들러는 `task queue`에 저장되어 대기하고 있다가
6. 실행컨텍스트 스택(`Call stack`)이 비면 이벤트 루프에 의해 이벤트핸들러가 `Call stack`으로 push되어 실행된다.

비동기함수는 비동기 처리결과가 나오기 전에 종료되기때문에 외부에 반환값을 전달할 수 없고 상위 스코프의 변수에 할당할 수도 없다.  
따라서 비동기 함수에 처리결과에 대한 후속 처리를 하는 콜백함수를 전달하여, 성공할 경우 호출될 콜백함수와 실패할 경우 호출될 콜백함수까지 전달할 수 있다.

## 에러 처리의 한계

비동기 처리를 위한 콜백함수의 가장 큰 문제점은 콜백헬이 발생하면 에러처리가 곤란하다는 점이다.  
에러 처리를 위해 주로 사용하는 `try...catch...finally`는  
try 코드 블럭에서 에러가 발생하면 catch 문의 err 변수에 에러가 전달되고 catch 내부 코드 블럭이 실행되며 finally 코드 블럭은 에러 발생과 상관없이 한번 실행된다.

```js
try {
  // ❌에러를 캐치하지 못한다.
  setTimeout(() => {
    throw new Error('Error:');
  }, 1000);
} catch (e) {
  console.error('캐치한 에러', e);
}
```

하지만 try 코드 블럭 내의 setTimeout 비동기 함수는 호출 즉시 실행 컨텍스트를 생성한 뒤 종료되고,  
내부 콜백함수는 `task queue`에 저장되어 대기하다가 실행 컨텍스트 스택이 비면 실행되기 때문에 에러가 발생해도 catch문에서 캐치할 수 없다.  
일반적인 에러는 호출한 방향으로 흐르기 때문에 비동기 내부 콜백함수에서 발생한 에러는 캐치되지 않는다.  
이러한 문제를 해결하기 위해 도입된 것이 Promise이다.

## Promise

Promise는 비동기 처리에 사용되는 객체로, Promise 생성자 함수는 비동기 처리를 수행할 콜백함수 (resolve, reject)를 인수로 전달받는다.

#### Promise 생성

```js
const promise = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve(1);
  }, 1000);
});

promise.then((n) => {
  console.log(n);
});
```

Promise는 성공할 수도 있고 실패할 수도 있기때문에, 성공할 때 실행할 resolve 콜백함수와 실패할 때 실행할 reject 콜백함수를 인수로 전달하고  
실행 결과에 따라 resolve 콜백함수는 then 메서드의 콜백함수로 전달되고, reject 콜백함수는 catch 메서드의 콜백함수로 전달되어 실행된다.

#### Promise는 비동기 처리 상태와 처리 결과를 관리하는 객체이다.

Promise의 상태는 생성 직후 pending상태에서 비동기 처리 뒤 resolve, reject 함수 호출에 따라 결정된다.

| 프로미스의 상태 정보 | 의미                    | 상태 변경 조건                   |
| -------------------- | ----------------------- | -------------------------------- |
| pending              | 아직 수행되지 않은 상태 | 프로미스가 생성된 직후 기본 상태 |
| fulfilled            | 수행된 상태(성공)       | resolve 함수 호출                |
| rejected             | 수행된 상태(실패)       | reject 함수 호출                 |

- settled 상태를 fulfilled 또는 rejected 상태를 통칭하는데 이는 pending이 아닌 비동기 처리가 수행된 상태를 의미한다.
- promise는 settled 상태로 변화하면 다른 상태로 변화할 수 없다.

<br>

## Promise 후속 처리 메서드

Promise의 비동기 처리 상태가 변하면, Promise의 처리 결과가 후속 처리 메서드에 인수로 전달되어 결과에 따라 선택적으로 호출된다.  
모든 후속처리 메서드는 프로미스를 반환하고, 이또한 비동기로 동작한다.

### `Promise.prototype.then`

- then 메서드는 2개의 콜백함수를 인수로 전달받는다. (result, error)
- 첫번째 콜백함수는, Promise가 fulfilled 상태로 resolve가 호출되면 호출되는 Callback.
  - 이 첫번째 콜백함수는 Promise의 비동기 처리 result를 인수로 전달받는다.
- 두번째 콜백 함수는, Promise가 rejected 상태로 reject가 호출되면 호출되는 Callback.
  - 이 두번째 콜백 함수는 Promise의 error를 인수로 전달받는다.

### `Promise.prototype.catch`

- catch 메서드는 1개의 콜백함수를 인수로 전달받고 이는 Promise가 rejected 상태일 때만 호출된다.

### `Promise.prototype.finally`

- finally 메서드는 1개의 콜백 함수를 인수로 전달받으며 Promise 성공, 실패 여부와 상관없이 무조건 마지막에 한번 호출되어 실행된다.

## Promise의 에러 처리

Promise의 에러처리는 후속처리 메서드의 then 또는 catch 메서드로 할 수 있다.

- then 메서드의 두번째 콜백함수로 에러처리하는 경우
  ```js
  const wrongUr1 = 'https://xxx.com/xxx1';
  promiseGet(wrongUrl).then(
    (res) => console.log(res),
    (err) => console.error(err)
  ); // Error:404
  ```
- catch 메서드로 에러처리하는 경우: 비동기 처리에서 발생한 에러뿐만아니라 then 메서드 내부에서 발생한 에러까지 모두 캐치할수 있고 가독성이 훨씬 명확하여 권장하는 메서드이다.
  ```js
  const wrongUr1 = 'https://xxx.com/xxx1';
  promiseGet(wrongUrl)
    .then((res) => console.log(res))
    .catch((err) => console.error(err)); // Error:404
  ```

## Promise 체이닝

Promise 는 후속처리 메서드를 통해 콜백헬을 해결할 수 있다. 이 메서드들은 모두 콜백함수가 반환한 Promise를 반환하고 만약 Promise가 아닌 값을 반환하더라도 암묵적으로 resolve, reject 하여 Promise 생성해 반환한다. 이 같은 Promise 체이닝을 통해 비동기 처리 결과를 전달받아 후속 처리를 계속해서 연결할 수 있기때문에 콜백헬을 해결할 수 있었지만, 이 또한 콜백함수를 사용하는 건 마찬가지이므로 가독성 측면에선 async/await 문법을 사용하는 것이 좋다.

## Promise 정적 메서드

### Promise.resolve /Promise.reject

> Promise를 생성하기 사용하는 메서드

- `Promise.resolve` : 인수로 전달받은 result를 resolve하는 Promise를 생성
- `Promise.reject` : 인수로 전달받은 error를 reject하는 Promise를 생성

### Promise.all

> 비동기 처리를 병렬로 진행할 때 사용하는 메서드

`Promise.all` 메서드에 전달한 배열의 모든 Promise가 fulfilled 상태가 되면 then 메서드가 호출되고,  
하나라도 rejected 상태가 되면 나머지 비동기 처리 결과를 기다리지않고 즉시 종료되고 catch 메서드가 호출된다.  
이때 반환되는 결과를 다시 배열로 저장해 그 배열을 resolve하는 새로운 Promise를 반환하여 처리 결과를 전달하기때문에 순서가 보장된다.

### Promise.race

`Promise.all`와의 차이는 모든 비동기가 fulfilled 상태되기를 기다리는 것이 아니라 가장 먼저 fulfilled된 Promise의 처리결과를 resolve한다.  
(rejected 상태의 경우 `Promise.all` 동일하게 처리된다)

### Promise.allSettled

`Promise.allSettled` 메서드는 배열로 전달된 모든 Promise가 settled 상태이면,  
처리 결과를 배열로 반환하는 메서드로 이 처리 결과 배열에는 fulfilled, rejected 상태와 상관없이 모든 Promise의 처리 결과가 담겨있다.

- Promise가 fulfilled 상태인 경우 : 비동기 처리에 대한 status 프로퍼티, 처리결과에 대한 value 프로퍼티
- Promise가 rejected 상태인 경우 : 비동기 처리에 대한 status 프로퍼티, 에러에 대한 reason 프로퍼티

## 마이크로태스크 큐

비동기 함수는 호출 즉시 실행 컨텍스트를 생성하고, 내부에 있는 콜백함수는 `task queue`에 저장되어 대기하다가 call stack이 비면 실행됐었다.  
하지만 Promise의 후속 처리 메서드의 콜백함수는 `task queue`가 아니라 `microtask queue`에 저장되어 대기하다가 call stack이 비면 실행된다. (그 외 콜백함수나 이벤트핸들러는 태스크 큐)

> 콜백함수나 이벤트 핸들러 같이 일시 저장된다는 점이 동일하지만,
>
> 마이크로태스크 큐는 태스크큐보다 우선순위가 높기때문에 이벤트 루프는 콜스택이 비면
>
> 먼저 마이크로태스크 큐에 있는 함수를 콜스택으로 가져와 실행하고 마이크로태스크 큐가 비었을 때 태스크 큐에 있는 함수를 콜스택으로 가져와 실행한다.

## fetch (Web API)

> fetch는 HTTP 요청을 생성하는 Web API로, HTTP 응답을 나타내는 Response 객체를 래핑한 Promise 객체를 반환하는 함수

#### Response.prototype.json

> Response객체에 포함되어있는 메서드로
>
> 응답받은 Response body를 역직렬화하여 반환하는 메서드

```js
fetch('https://www.~~~~~.com/api/~~')
  .then((res) => {
    if (!res.ok) throw new Error(res.status); // HTTP Error 처리를 진행하지 않으면 기본 서버에러를 캐치할 수 없음
    res.json(); // Response body를 JSON으로 역직렬화
  })
  .then((users) => console.log(users));
```

- fetch 함수의 단점은 기본적인 404 Not Found 나 500 Internal Server Error와 같은 HTTP 에러를 reject하지 않는다는 점이다.  
  (오프라인, 네트워크 장애, CORS에러 같은 요청이 완료되지 못한 경우에만 Promise를 reject한다)
  이 HTTP에러를 캐치하기 위해선 Response 객체의 ok 프로퍼티를 사용하여 직접 에러처리를 해줘야하는 번거로움이 있다.
- 좀 더 편리함을 위해 axios 라이브러리를 사용하면 모든 HTTP 에러를 캐치할 수 있어 에러처리가 편리하고 다양한 기능을 제공하는 장점을 가진다.
