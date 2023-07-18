# 15장.let, const 키워드와 블록 레벨 스코프

## 15.1 var 변수의 문제점

### (1) 변수의 중복 선언 허용
var 키워드로 선언한 변수는 중복 선언이 가능하다. 중복 선언을 했을 때 초기화문이 있는 변수는 JS 엔진에 의해 var 키워드가 없는 것 처럼 동작하여 값이 재할당되어 변경될 수 있고, 초기화문이 없는 변수 선언문은 에러를 발생시키지 않고 변수 선언문을 그냥 무시한다. 따라서 var 키워드 변수는 중복 선언으로 의도치 않은 재할당을 주의해야한다.

```js
var x = 1;
var y = 1;

var x = 100;      // -> 초기화문이 있는 변수 선언문은 var 키워드가 없는 것처럼 동작한다.
var y;            // -> 초기화문이 없는 변수 선언문은 무시된다.

console.log(x);   // 100
console.log(y);   // 1
```

### (2) 함수 레벨 스코프
var 키워드로 선언한 변수는 함수의 코드블록만을 지역 스코프로 인정하여 함수 외부에서 var 키워드로 선언한 변수는 코드 블록 내에서 선언해도 모두 전역 변수가 된다. 함수 레벨 스코프는 전수를 남발할 가능성을 높여 의도치않은 중복 선언이 발생할 수 있다.

```js
var i = 10;

// for 문에서 선언한 i는 전역 변수다. 이미 선언된 전역 변수 i가 있으므로 중복 선언된다.
for(var i = 0; i < 5; i++){
  console.log(i);	// 0 1 2 3 4
}

console.log(i);	// 5 의도치않게 i 변수의 값이 변경되었다.
```

### (3) 변수 호이스팅
var 키워드로 선언한 변수는 변수 호이스팅에 의해 변수 선언문이 스코프의 선두로 끌어 올려진 것처럼 동작하여 변수 선언문 이전에 참조할 수 있고 할당문 이전에 변수를 참조하면 `undefined`를 반환한다. 하지만 변수 호이스팅을 이용해 변수 선언문 이전에 변수를 참조하는 것은 가독성을 떨어뜨리고 오류를 발생시킬 여지를 남긴다.

```js
// 이 시점에는 변수 호이스팅에 의해 이미 foo 변수가 선언되었다. (1.선언 단계)
// 변수 foo는 undefined로 초기화된다. (2.초기화 단계)
console.log(foo);	// undefined

// 변수에 값을 할당 (3.할당 단계)
foo = 100;

console.log(foo);	// 100

// 변수 선언은 런타임 이전에 자바스크립트 엔진에 의해 암묵적으로 실행된다.
var foo;
```



<br>

## 15.2 let 변수

var 키워드의 단점을 보완하기 위해 ES6에서는 새로운 변수 선언 키워드인 let, const를 도입했다.

### (1) 변수의 중복 선언 금지
중복선언을해도 에러를 발생시키지않는 var 와 달리 let 키워드로 선언한 변수는 이름이 같은 변수를 중복 선언하면 **문법 에러(SyntaxError)가 발생**하기때문에 같은 스코프 내에서 중복 선언을 허용하지 않는다.
```js
let foo = 100;
let foo = 200; // SyntaxError: Identifier 'foo' has already been declared
```

### (2) 블록 레벨 스코프
함수 레벨 스코프를 따른 var 와 달리 let 키워드로 선언한 변수는 모든 코드 블록을 지역 스코프로 인정하는 **블록 레벨 스코프**를 따른다.

```js
let foo = 1;	// 전역 변수

{
  let foo = 2;	// 지역 변수
  let bar = 3;	// 지역 변수
}

console.log(foo);	// 1
console.log(bar);	// ReferenceError: bar is not defined
```
    전역에서 선언된 foo 변수와 코드 블록 내에서 선언된 foo 변수는 서로 다른 별개의 변수이고
    코드 블록 내에서 선언된 bar 변수도 블록 레벨 스코프를 갖는 지역변수이므로 전역에서 참조할 수 없다.


<p align="center"><img src="https://velog.velcdn.com/images/kozel/post/677ed533-e045-4338-8c17-a25674425847/image.jpeg" width="450px"></p>

*함수도 코드 블록이므로 스코프를 생성하고 함수내의 코드블록은 함수 레벨 스코프에 중첩된다.*


### (3) 변수 호이스팅

선언단계에서 스코프에 변수 식별자를 등록해 JS 엔진에 의해 초기화 단계에 의해 `undefined`로 변수를 초기화했던 var 키워드와 달리 let 키워드는 선언단계와 초기화단계까 분리되어 진행된다. 런타임 이전에 JS엔진에 의해 암묵적으로 선언단계가 실행되지만 초기화단계는 변수선언문에 도달했을 때 실행되어 변수 선언문 이전에 참조하면 **참조 에러(ReferenceError)가 발생**한다. let 키워드로 선언한 변수도 호이스팅이 발생하기 때문이다.

```js
console.log(foo);	// undefined

var foo;
console.log(foo);	// undefined

foo = 1;	        // 할당문에서 할당 단계가 실행된다.
console.log(foo);	// 1
```

```js
console.log(foo);    // ReferenceError: foo is not defined

let foo;             // 변수 선언문에서 초기화 단계가 실행된다.
console.log(foo);    // undefined

foo = 1;             // 할당문에서 할당 단계가 실행된다.
```

<p align="center"><img src="https://velog.velcdn.com/images/kozel/post/b6df99c3-8c6b-44d0-a9d4-94d7668ab612/image.jpeg" width="400px"><img src="https://velog.velcdn.com/images/kozel/post/aec01fe9-b458-4ca1-9015-cef821d84b2b/image.jpeg" width="400px"></p>

let 키워드로 선언한 변수는 스코프 시작 시점부터 초기화 단계 시작 지점(변수선언문)까지 변수를 참조할 수 없다. 이 스코프 시작 지점부터 초기화 시작 지점까지 변수를 참조할 수 없는 구간을 **일시적 사각지대**(Temporal Dead Zone)라고 부른다.


### (4) 전역객체와 let

var 키워드로 선언한 전역 변수, 전역함수, 암묵적 전역은 모두 `전역객체 window`의 프로퍼티가 된다. 하지만 let 키워드로 선언한 전역 변수는 전역 객체의 프로퍼티가 아니기때문에 window 프로퍼티로 접근할 수 없다.

```js
// 이 예제는 브라우저 환경에서 실행해야 한다.

var x = 1;    // 전역 변수
y = 2;        // 암묵적 전역

console.log(window.x);	// 1
console.log(x);			    // 1
console.log(window.y);	// 2
console.log(y);			    // 2

let z = 1;

console.log(window.z);	// undefined
console.log(x);			    // 1
```
    let 전역 변수는 보이지 않는 개념적인 블록 내에 존재하게 된다.

<br>


## 15.3 const 변수

### (1) 선언과 초기화

const 키워드로 선언한 변수는 **선언과 동시에 초기화**해야한다. 그렇지 않으면 **문법 에러(SyntaxError)가 발생**한다.

```js
const foo = 1;
const bar;     // SyntaxError: Missing initializer in const declaration
```

```js
{
  // 변수 호이스팅이 발생하지 않는 것처럼 동작한다.
  console.log(foo);  // ReferenceError: Cannot access 'foo' before initialization
  const foo = 1;
  console.log(foo);  // 1
}
// 블록 레벨 스코프를 갖는다.
console.log(foo);   // ReferenceError: foo is not defined
```


const 키워드로 선언한 변수는 let 키워드로 선언한 변수와 마찬가지로 블록 레벨 스코프를 가지지만 변수 호이스팅이 발생하지 않는 것 처럼 동작한다.

### (2) 재할당 금지

var, let 키워드로 선언한 변수와 다르게 const 키워드로 선언한 변수는 재할당이 금지된다.

```js
const foo = 1;
foo = 2;       // TypeError: Assignment to constant variable.
```

### (3) 상수

상수는 재할당이 금지된 변수를 말한다. 상수도 값을 저장하기 위한 메모리 공간이 필요하므로 변수라고 할 수 있으며 변수는 재할당이 가능하지만 상수는 재할당이 금지된다.

```js
const TAX_RATE = 0.1;
let preTaxPrice = 100;
```
    코드 내에서 상수 0.1 이나 100 이 어떤 의미로 사용했는지 명확히 알기 어려워 가독성이 나빠진다.
    이를 해결하기위해 상수로 정의하여 값의 의미를 쉽게 파악하고 변경될 수 없는 고정값으로 사용할 수 있다.

    일반적으로 상수의 이름은 대문자로 선언하고 여러 단어일 경우 스네이크 케이스로 표현하여 언더스코어(_)로 구분한다.


### (4) const 키워드와 객체

const 키워드로 선언된 변수에 객체를 할당할 경우 값을 변경할 수 있다. 변경 불가능한 값인 원시값은 재할당 없이 변경할 수 있는 방법이 없지만 변경 가능한 값인 객체는 재할당 없이도 직접 변경이 가능하기 때문이다. 즉, 재할당을 금지할 뿐 "불변"을 의미하지 않아 프로퍼티 동적 생성, 삭제, 값 변경을 통해 객체를 변경하는 것이 가능하다.

```js
const person = {
  name: "Kozel"
};

person.name = 'Kim'
console.log(person);  // {name: "Kim"}
```


## 15.4 var vs. let vs. const

- ES6를 사용한다면 var 키워드는 사용하지 않는다.
- 재할당이 필요한 경우 스코프를 최대한 좁게하여 let 키워드를 사용한다.
- 변경이 발생하지 않고 읽기 전용으로 사용하는 원시 값과 객체에는 const 키워드를 사용한다.