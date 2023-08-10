# Type Inference (타입추론)
> TypeScript가 코드를 해석해 나가는 동작

타입을 따로 지정하지 않아도 변수, 속성, 인자의 기본값, 함수의 반환값 등을 설정할 때 타입 추론이 발생한다.


```ts
interface Dropdown<T> {
  value: T;
  title: string;
}

interface DetailedDropdown<K> extends Dropdown<K> {
  description: string;
  tag: K;
}

const detailedItem: DetailedDropdown<string> = {
  title: 'abc',
  description: 'ab',
  value: 'a',
  tag: 'a',
};
```

<br>

# Type Assertion (타입 단언)
> 개발자가 타입을 확신할 때 사용하는 타입 지정 방식

- 기본적으로 `as`키워드를 사용하여 정의한다.
- 타입 단언은 타입스크립트 컴파일러에게 타입을 알려주는 역할을 한다. (개발자가 타입을 컴파일러보다 더 잘 알고있을 때 사용)
- DOM API를 조작할 때 주로 사용한다.
- #### 타입 단언 사용 예시
    ```ts
    const div = document.querySelector('div');
    if(div) { // null 값일 수도 있기때문에 type을 보장해주기 위해 if문으로 null 체크
      div.innerText;
    }
    ```
    ```ts
    const div = document.querySelector('div') as HTMLDivElement;
    div.innerText;
    ```
- 
    ```ts
    interface Hero {
      name: string;
      age: number;
    }

    const capt: Hero = {}; // X. 오류 발생
    capt.name = '캡틴';
    capt.age = 100;
    ```

    ```ts
    const capt = {} as Hero; // as를 사용하여 타입 단언하면 타입 오류를 해결할 수 있음
    ```

<br>


# Type Compatibility (타입 호환성)
> TypeScript가 코드에서 특정 타입이 다른 타입에 잘 맞는지, 속성끼리 서로 호환이 되는지 점검하는 방식


```ts
interface Developer {
  name: string;
  skill: string;
}

interface Person {
  name: string;
}

let developer: Developer;
let person: Person;
```
```ts
developer = person; // ❌에러 발생 (오른쪽에 있는 타입이 구조적으로 더 커야 왼쪽과 호환이 됨)
person = developer; // 에러 발생 안함
```

