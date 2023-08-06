# Type Aliases (타입 별칭)
> 특정 타입이나 인터페이스를 참조할 수 있는 타입 변수

```ts
// 일반적인 string 타입을 사용할 때
const name: string = 'hello';
```

- `type`키워드를 사용하여 타입 별칭을 지정할 수 있다.
  ```ts
  type MyStringType = string;
  const str: MyStringType = 'hello';
  ```
- 타입별칭은 새로운 타입 값을 생성하는 것이 아니라 정의한 타입에 대해 참고하기 쉽게 이름을 부여하는 것과 같다.

## 타입별칭과 인터페이스의 차이점

```ts
interface Person {
  name: string;
  age: number;
}

type PersonType = {
  name: string;
  age: number;
}

const nani: Person = {
  name: 'nani',
  age: 20,
};
```
- interface를 사용하여 타입을 정의하면 interface Person을 이용함을 확인할 수 있고
- type별칭을 사용하여 타입을 정의하면 내부 타입에 대한 정보를 확인할 수 있다.
- 가장 큰 차이점은 확장여부이다. **interface는 확장이 가능**하지만 type은 확장이 불가능하다. (interface로 선언하는 것이 좋음)