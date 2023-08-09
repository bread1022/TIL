# Generics
> 타입을 마치 함수의 파라미터처럼 사용하는 것을 의미


- 한가지 타입보다 여러가지 타입에서 동작하는 컴포넌트 생성에 사용된다.
- 보통 API를 호출한 뒤 응답에 대한 규격을 정의할 때 가장 많이 사용된다!
  ```ts
  function fetchContacts(): Promise<Response> {
    // ... Promise 반환타입을 제네릭으로 지정하면 타입 추론이 가능하다
  }
  ```
- 함수를 호출하는 시점에 제네릭값으로 사용할 타입을 넘겨서 사용한다.
    ```ts
    function logText(text: string) {
      return text;
    }

    function logNumber(num: number) {
      return num;
    }

    logText('a');
    logNumber(10);
    // 같은 로직으로 동작하는 함수지만 타입에 따라
    // 중복으로 생성해야되는 문제점이 있다. (유지보수 측면에서 좋지않음)
    // 이를 해결하기 위해 제네릭을 사용
    ```

    ```ts
    function logText(text: string | number) {
      return text; // string과 number의 교집합 메서드만 사용가능
    }

    function log<T>(text: T): T {
      return text; // 함수를 호출할 때 타입을 정의
    }

    log<string>('a'); // 제네릭을 사용하여 타입을 정의
    log('a');         // 이렇게 사용해도 타입 추론이 됨
    log<number>(10);
    ```

    ```ts
    interface Email {
      value: string;
      selected: boolean;
    }

    const email: Email[] = [
      { value: 'naver.com', selected: true },
      { value: 'gmail.com', selected: false },
      { value: 'hanmail.net', selected: false },
    ];

    interface ProductNumber {
      value: number;
      selected: boolean;
    }

    const NumberOfProduct: ProductNumber[] = [
      { value: 1, selected: true },
      { value: 2, selected: false },
      { value: 3, selected: false },
    ];

    const createDropdownItem = (item: Email | ProductNumber) => {
      const option = document.createElement('option');
      option.value = item.value.toString();
      option.innerText = item.value.toString();
      option.selected = item.selected;
      return option;
    };
    // 언제든지 타입이 추가될 수 있기 때문에 확장성을 위해서 제네릭을 사용함
    ```


#### interface를 제네릭으로 선언하는 방법

```ts
// 인터페이스 마다 타입을 따로 지정하는 게 아니라
interface Dropdown<T> { // 사용할 때 마다 제네릭 타입을 지정하면 된다!
  value: T;
  selected: boolean;
}

const email: Dropdown<string>[] = [
  { value: 'naver.com', selected: true },
  { value: 'gmail.com', selected: false },
  { value: 'hanmail.net', selected: false },
];

const obj: Dropdown<number> = { value: 10, selected: false };
const obj2: Dropdown<string> = { value: 'abc', selected: true };

const createDropdownItem = (item: Dropdown<string> | Dropdown<number>) => {
  const option = document.createElement('option');
  option.value = item.value.toString();
  option.innerText = item.value.toString();
  option.selected = item.selected;
  return option;
};

// 또는 Union type도 제거하기 위해서 사용하는 함수에 제네릭을 사용할 수 있음
const createDropdownItem<T> = (item: Dropdown<T>) => {
  const option = document.createElement('option');
  option.value = item.value.toString();
  option.innerText = item.value.toString();
  option.selected = item.selected;
  return option;
};

emails.forEach((email) => {
  const item = createDropdownItem<string>(email);
  const selectTag = document.querySelector('#email-dropdown');
  selectTag.appendChild(item);
});
```

### 제네릭의 타입 제한

#### `extends`

```ts
const logTextLength = <T>(text: T): T => {
  text.length; // Error: text에 length속성이 있을지 알 수 없기때문
  return text;
};
```

```ts
// length 속성의 type을 extends하여 타입을 제한함
interface LengthType {
  length: number;
}

// 이 제네릭은 length 속성이 있는 타입만 받을 수 있음
const logTextLength = <T extends LengthType>(text: T): T => {
  text.length;
  return text;
};

logTextLength('a'); // ⭕️ string은 length 속성이 있기 때문에
logTextLength(10);  // ❌ number는 length 속성이 제공되지 않기때문
```


#### `keyof`


```ts
interface ShoppingItem {
  name: string;
  price: number;
  stock: number;
}

// ShoppingItem의 key값 중 한가지만 제네릭으로 받을 수 있음
const getShoppingItemOption = <T extends keyof ShoppingItem>(itemOption: T): T => {
  return itemOption;
};

getShoppingItemOption('abc'); // ❌ ShoppingItem에 없는 key값이기 때문에
getShoppingItemOption('name');
getShoppingItemOption('price');
getShoppingItemOption('stock');
```


```ts
// 매개변수로 obj를 받고, 두번째 인자로는 obj의 key값만을 받음
// 첫번째 인자 객체에 없는 속성 접근을 제한함
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const obj = { a: 1, b: 2, c: 3, d: 4 };
getProperty(obj, 'a');
getProperty(obj, 'e'); // ❌ obj에 없는 key값이기 때문에
```