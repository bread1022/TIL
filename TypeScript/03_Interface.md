# Interface


#### interface로 정의할 수 있는 범주
- 객체의 속성와 속성 타입
- 함수의 파라미터, 반환값에 대한 타입
- 배열과 객체를 접근하는 방식
- 클래스


## 옵션속성

- 인터페이스에 정의된 속성을 모두 사용하지 않아도 되게끔하는 문법이다.
  ```ts
  interface Person {
    name: string;
    age?: number; // ?를 붙여 옵션속성으로 만든다. => age는 있어도 되고 없어도됨
  }
  ```
- 속성을 선택적으로 적용할 수 있고 인터페이스에 정의되어있지 않은 속성에 대해 인지시켜줄 수 있다.


## 읽기전용속성

- 인터페이스로 객체를 생성할 때만 값을 할당하고 그 이후에는 값을 변경할 수 없게끔 하는 문법이다.
- `readonly` 키워드를 사용한다.
  ```ts
  interface CraftBeer {
    readonly brand: string;
  }
  CraftBeer.brand = 'abc'; // error (수정하려고 하면 에러남)
  ```


## 딕셔너리패턴

```ts
interface StringRegexDictionary {
  [key: string]: RegExp;
}

const obj: StringRegexDictionary = {
  sth: 'abc', // error
  cssFile: /\.css$/, // key는 string, value는 RegExp
  jsFile: /\.js$/,
};

obj['cssFile'] = 'a'; // error - value = Regex가 아니므로 에러
```

## 인터페이스 확장

```ts
interface Person {
  name: string;
  age: number;
}

interface Developer extends Person {
  language: string; // Person의 속성을 상속받음
}
```


## 객체 선언과 관련된 타입 체킹

- 인터페이스를 이용하여 객체를 선언하면 속성 검사를 더 엄밀히 진행한다.
  ```ts
  interface CraftBeer {
    brand: string;
  }

  function makeCraftBeer(beer: CraftBeer): void {
    // ...
  }

  makeCraftBeer({ brend: 'abc' }); // error (오탈자 점검 오류 발생)
  ```
  - 타입 추론을 무시하고 싶을 땐 **`as` 키워드를 사용**한다.
    ```ts
    makeCraftBeer({ brend: 'abc' } as CraftBeer);
    ```
    ```ts
    const myBeer = { brend: 'abc' };
    makeCraftBeer(myBeer as CraftBeer);
    ```