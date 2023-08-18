# 30장. Date

내장 객체 Date를 활용하면 날짜를 측정할 수도 저장, 출력할 수 있다.

### 날짜 구성 요소

#### new Date()

인수 없이 생성자 함수를 호출하면 현재 날짜와 시간을 가지는 Date 객체를 반환한다.

```js
const now = new Date();
console.log(now); // 현재 시각 2023-08-18T07:23:00.000Z
```


#### getFullYear()
> 연도(4자릿수) 정보를 반환한다.

```js
const now = new Date();
now.getFullYear(); // 2023
```


#### getMonth()
> 월(0 ~ 11) 정보를 반환한다.


```js
const now = new Date();
now.getMonth(); // 7 (8월)
```

0부터 1월, 11까지 12월을 나타낸다.


#### getDate()
> 일(1 ~ 31) 정보를 반환한다.


```js
const now = new Date();
now.getDate(); // 18
```

#### getDay()
> 요일(0 ~ 6) 정보를 반환한다.

```js
const now = new Date();
now.getDay(); // 4 (목요일)
```

0부터 일요일, 6까지 토요일을 나타낸다.



#### getHours(), getMinutes(), getSeconds(), getMilliseconds()
> 시, 분, 초, 밀리초 정보를 반환한다.



#### Date.prototype.toLocaleString()
> 인수로 전달한 local(국가)기준으로 Date객체의 날짜와 시간을 문자열로 반환


```js
const now = new Date('2023-08-18');
now.toLocaleString(); // 2023. 8. 18. 오전 9:00:00
now.toLocaleString('ko-KR'); // '2023. 8. 18. 오전 9:00:00'
now.toLocaleString('en-us'); // '8/18/2023, 9:00:00 AM'
```


#### Date.prototype.toLocaleTimeString()
> 인수로 전달한 local(국가)기준으로 Date객체의 시간을 문자열로 반환

```js
const now = new Date('2023-08-18');
now.toLocaleTimeString(); // '오전 9:00:00'
now.toLocaleTimeString('ko-KR'); // '오전 9:00:00'
now.toLocaleTimeString('en-us'); // '9:00:00 AM'
```