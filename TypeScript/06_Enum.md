# Enums
> 특정 값들의 집합을 의미하는 자료형

- 정해진 값들을 관리할 때 유용
- 정확한 코드를 작성하고 예외처리에 유용
- 런타임시점에서의 이넘은 객체로 존재한다.
- 컴파일시점에서의 이넘은 keyof 사용시 `keyof typeof enumName` 으로 사용해야한다.
  ```ts
  enum LogLevel {
    ERROR, WARN, INFO, DEBUG
  }

  // 'ERROR' | 'WARN' | 'INFO' | 'DEBUG';
  type LogLevelStrings = keyof typeof LogLevel;
  ```


## 숫자형 이넘

```ts
enum Direction {
  Up      // 0
  Down,   // 1
  Left,   // 2
  Right,  // 3
}
```
- 이넘을 선언하면 0부터 차례로 1씩 증가한 값이 할당된다.
  - 별도의 값을 지정하지 않으면 숫자형 이넘으로 선언된다.
- 선언시 초기값을 함께 넣어주면 초기값부터 차례로 1씩 증가한 값이 할당된다.
  ```ts
  enum Direction {
    Up = 3,
    Down,  // 4
    Left,  // 5
    Right  // 6
  }
  ```
- 
  ```ts
  enum Response {
    No = 0,
    Yes = 1,
  }

  function respond(recipient: string, message: Response): void {
    // ...
  }

  respond("Captain Pangyo", Response.Yes);
  ```
- Reverse Mapping이 가능하다.
  ```ts
  enum Enum {
    A,
    B
  }

  let a = Enum.A; // key의 값으로 value 접근
  let nameOfA = Enum[a]; // "A" value로 key 접근
  ```



## 문자형 이넘

- 숫자형 이넘과 다르게 모든 값을 특정 String 또는 다른 enum 값으로 초기화해야한다.  
  (복합이넘이 되긴하지만 최대한 같은 타입으로 이루어진 이넘을 사용하는 것이 좋음)
- 숫자형 이넘과 다르게 auto-increment가 되지 않는다.
- 디버깅시 항상 명확한 값이 나와 읽기 좋다.

```ts
enum Shoes {
  Nike = '나이키',
  Adidas = '아디다스',
}

const myShoes = Shoes.Nike;
console.log(myShoes); // '나이키'
```

```ts
enum Answer {
  Yes = 'Y',
  No = 'N',
}

function askQuestion(answer: Answer) {
  if (answer === Answer.Yes) { // answer === 'yes' 이런식의 하드코딩을 줄일 수 있음
    console.log('정답입니다.');
  }
  if (answer === Answer.No) {
    console.log('오답입니다.');
  }
}

askQuestion(Answer.Yes);
askQuestion('Yes'); // 타입이 다르기 때문에 Error
```