## tsconfig.json
> 타입스크립트 설정파일

- TypeScript를 JavaScript로 변환할 때 설정을 정의해놓은 파일

```json
{
  "compilerOptions": {
    "allowJs": true,
    "checkJs": true,
    "target": "es5",
    "lib": ["es2015", "dom", "dom.iterable"],
    "noImplicitAny": true, // any라도 무조건 타입을 선언해야함 true
    "strict": true,
    "strictFunctionTypes": true
  },
  "include": ["./src/**/*"]
}
```


## eslintrc.js

```js
module.exports = {
  root: true,
  env: {
    browser: true,
    node: true,
    jest: true,
  },
  extends: [
    'plugin:@typescript-eslint/eslint-recommended',
    'plugin:@typescript-eslint/recommended',
  ],
  plugins: ['prettier', '@typescript-eslint'],
  rules: {
    'prettier/prettier': [
      'error',
      {
        singleQuote: true,
        semi: true,
        useTabs: false,
        tabWidth: 2,
        printWidth: 80,
        bracketSpacing: true,
        arrowParens: 'avoid',
      },
    ],
    '@typescript-eslint/no-explicit-any': 'off',
    "@typescript-eslint/explicit-function-return-type": 'off',
    'prefer-const': 'off',
  },
  parserOptions: {
    parser: '@typescript-eslint/parser',
  },
};
```

## Module System

```ts
// types.ts
interface PhoneNumberDictionary {
  [phone: string]: {
    num: number;
  };
}

interface Contact {
  name: string;
  address: string;
  phones: PhoneNumberDictionary;
}

enum PhoneType {
  Home = 'home',
  Office = 'office',
  Studio = 'studio',
}

export { Contact, PhoneType };
```

- 타입들을 정의한 types.ts 파일을 만들고 export를 통해 외부에서 사용할 수 있도록 한다.
- `tsconfig.json`에 설정한 컴파일러 모드에 따라 모듈코드가 다르게 변환된다.


## d.ts
> 타입스크립트 선언파일

- TypeScript 코드의 타입 추론을 돕는 파일
- 전역 변수에 선언한 변수를 특정 파일에서 import 구문없이 사용하는 경우 해당 변수를 인식하지 못함


------

## 유틸리티타입
> 이미 정의해놓은 타입을 변환할 때 사용하기 좋은 타입 문법



### Partial
> 특정 타입의 부분집합 타입을 정의

```ts
interface Address {
  email: string;
  address: string;
}

type MayHaveEmail = Partial<Address>;
const all: MayHaveEmail = { email: 'abc@gmail.com', address: 'seoul' }; // 가능
const email: MayHaveEmail = { email: 'abc@gmail.com' }; // 가능
const address: MayHaveEmail = { address: 'seoul' }; // 가능
const nothing: MayHaveEmail = {}; // 가능
```


### Pick
> 특정 타입에서 몇 개의 속성을 선택(pick)하여 타입을 정의

```ts
interface AddressBook {
  name: string;
  phone: number;
  address: string;
  company: string;
}

type PhoneBook = Pick<AddressBook, 'name' | 'phone'>; // name, phone만 선택
const phone1: PhoneBook = { name: '김댕댕', phone: 0211112222 }; // 가능
const phone2: PhoneBook = { name: '이똘똘', phone: 021234567, address: 'seoul' }; // 불가능
```


### Omit
> 특정 타입에서 지정된 속성만 제거한 타입을 정의

```ts
interface AddressBook {
  name: string;
  phone: number;
  address: string;
  company: string;
}

type PhoneBook = Omit<AddressBook, 'address'>; // address만 제거
const phone1: PhoneBook = { name: '김댕댕', phone: 1234567890, company: 'google' }; // 가능
