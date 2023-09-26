# 44장. REST API

REST는 HTTP를 기반으로 클라이언트가 서버의 리소스에 접근하는 방식을 규정한 아키텍처이며  
REST API는 REST를 기반으로 서비스 API를 구현한 것을 말한다.

## REST API의 구성

REST API는 `자원(Resource)`, `행위(Verb)`, `표현(Representations)`의 3가지 요소로 구성된다.  
REST는 자체 표현 구조로 구성되어 REST API만으로 HTTP 요청의 내용을 이해할 수 있다



구성요소 | 내용 | 표현 방법
---|---|---
`자원(Resource)` | 자원 | URI(엔드포인트)
`행위(Verb)` | 자원에 대한 행위 | HTTP 요청 메서드
`표현(Representations)` | 자원에 대한 행위의 구체적 내용 | 페이로드


## REST API 설계 원칙

#### REST 에서 중요한 기본 원칙
1. URI는 리소스를 표현하는데 집중해야한다.
    - 리소스를 식별할 수 있는 이름은 동사보다는 명사를 사용한다.
    - 행위에대한 표현은 하지 않아야한다.
2. 리소스에 대한 행위는 HTTP 요청 메서드로 표현해야한다.
    - GET, POST, PUT, PATCH, DELETE 같은 주요 HTTP 메서드를 사용하여 CRUD를 구현한다.

      HTTP 요청메서드 | 종류 | 목적 | 페이로드
      ---|---|---|---|
      GET | index/retrieve | 모든/특정 리소스 취득 | X
      POST | create | 리소스 생성 | O
      PUT | replace | 리소스의 전체 교체 | O
      PATCH | modify | 리소스의 일부 수정 | O
      DELETE | delete | 모든/특정 리소스 삭제 | X

## REST API의 사용

### JSON Server
> 로컬의 json 파일을 사용하여 가상 REST API 서버를 구축할 수 있는 도구


1. `npm install json-server` 로 json-server 설치
2. 프로젝트 폴더에 리소스를 제공하는 데이터 베이스역할을 할 `db.json` 파일 생성
3. `json-server --watch db.json` 명령어로 json-server 실행
    ```js
    json-server --watch db.json // 기본 포트 3000

    json-server --watch db.json --port 4000 // 포트를 변경할 경우
    ```


#### GET 요청


```js
const xhr = new XMLHttpRequest();

xhr.open('GET', '/todos'); // HTTP 요청 초기화
xhr.send(); // HTTP 요청 전송
xhr.onload = () => { // 요청이 성공적으로 완료되면 load이벤트가 발생함
  if (xhr.status === 200 || xhr.status === 201) {
    document.querySelector('pre').textContent = xhr.response;
    return;
  }
  console.error(xhr.responseText);
};
```


#### POST 요청


```js
const xhr = new XMLHttpRequest();

xhr.open('POST', '/todos'); // HTTP 요청 초기화
xhr.setRequestHeader('content-type', 'application/json'); // HTTP 요청 헤더 설정
// request body로 전송할 페이로드의 MIME 타입을 지정함
xhr.send(JSON.stringify({ id: 4, content: 'JavaScript', completed: false })); // HTTP 요청 전송
xhr.onload = () => {
  if (xhr.status === 200 || xhr.status === 201) {
    document.querySelector('pre').textContent = xhr.response;
    return;
  }
  console.error(xhr.responseText);
};
```

#### PUT 요청
> 특정 리소스의 전체를 교체할 때 사용

#### PATCH 요청
> 특정 리소스의 일부를 수정할 때 사용

#### DELETE 요청
> 특정 리소스를 삭제할 때 사용