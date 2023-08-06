# Union Type
> 특정 타입 이상 사용할 수 있게끔 연산자를 사용하는 타입

```ts
function logText(text: string | number) {
  // ...
}
```
```ts
type status = string | number | boolean;
```
- JS의 OR연산자와 같이 `|`을 사용하여 타입을 여러개 연결하는 방식이다.
- 유니언타입을 사용할 땐 타입가드를 잘 적용해야 오류를 방지할 수 있다.
    ```ts
    interface Person {
      name: string;
      age: number;
    }
    interface Developer {
      name: string;
      skill: string;
    }
    function introduce(someone: Person | Developer) {
      someone.name; // O 정상 동작
      someone.age; // X 타입 오류
      someone.skill; // X 타입 오류
    }
    ```
    - 별도의 타입가드(Type Guard)를 이용하여 타입의 범위를 좁히지 않는 이상 기본적으로는 Person과 Developer 두 타입에 공통적으로 들어있는 속성인 name만 접근할 수 있게된다.



# Intersection Type
> 여러 타입을 모두 만족하는 하나의 타입

```ts
interface Person {
  name: string;
  age: number;
}

interface Developer {
  name: string;
  skill: string;
}

type Capt = Person & Developer;
  // {
  //   name: string;
  //   age: number;
  //   skill: string;
  // }
```
- `&`을 사용하여 타입을 하나로 합치는 방식이다.
