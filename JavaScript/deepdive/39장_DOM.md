# 39장. DOM (Document Object Model)

브라우저 렌더링 엔진은 웹 문서를 로드한 후 파싱하여 브라우저가 이해할 수 있는 자료 구조로 구성하여 메모리에 적재하는데 이를 **DOM**이라 한다. DOM은 프로그래밍언어가 접근하고 수정할 수 있도록 프로퍼티와 메서드를 갖는 JS 객체인 DOM API를 제공한다.


## 39.1 노드

HTML element 는 HTML 문서를 구성하는 개별적인 요소를 의미한다.  
이는 렌더링엔진에 의해 파싱되어 DOM 구성 노드 객체로 변환된다.  
요소의 어트리뷰트는 어트리뷰트 노드로, 텍스트는 텍스트 노드로 변환되고,  
HTML 요소간 중첩관계를 형성하면서 HTML element를 객체화한 모든 노드 객체들이 트리자료구조로 구성된다.


### 트리 자료구조

<p align="center"><img src="https://info343.github.io/img/html/dom-tree.jpg" width="300px">
<img src="https://github.com/bread1022/TIL/assets/107349637/2b40bc84-83ea-44c1-bf16-bf088c5525f2" width="300px">
</p>

트리자료구조는 노드들의 계층적 구조를 표현하는 자료구조를 말한다.  
하나의 최상위 노드에서부터 시작하여 여러 노드가 부모-자식, 형제 관계로 서로 연결되어 있고,  
마지막 자식 노드가 없는 노드를 리프노드라고 한다.


### 노트 타입

<p align="center"><img src="https://poiemaweb.com/img/HTMLElement.png" width="300px"></p>

1. **문서노드**
  - DOM 트리의 최상위에 존재하는 루트노드로 DOM 트리에 접근하기 위한 진입점(Entry Point)역할
  - document 객체 (브라우저가 렌더링한 HTML문서 전체를 가리키는 객체)
  - window.document로 접근 가능 (script 태그가 여러개로 분리되어있어도 하나의 전역 객체 window를 공유함)
2. **요소노드**
  - HTML 요소를 가리키는 객체
  - 요소간 중첩관계를 통해 정보를 구조화함
  - 문서의 구조를 표현
3. **어트리뷰트 노드**
  - HTML 요소의 어트리뷰트를 가리키는 객체
  - 어트리뷰트가 지정된 HTML요소의 노드에만 연결되어있다. (부모와 연결되어있지 않음)
  - 어트리뷰트 노드에 접근하기 위해서는 요소 노드에 먼저 접근하고, 어트리뷰트 노드에 접근하여 어트리뷰트를 참조하거나 변경해야한다.
4. **텍스트 노드**
  - HTML 요소의 텍스트를 가리키는 객체
  - 요소 노드의 자식 노드로 텍스트 노드는 자식 노드를 가질 수 없는 리프 노드이다.
  - DOM 트리의 최종단
  - 텍스트 노드에 접근하기 위해서는 요소 노드에 먼저 접근한 다음 텍스트 노드에 접근해야한다.
  - 문서의 정보를 표현
5. 그 외
  - 주석노드, 문서타입노드, 문서프로세싱노드, CDATA노드 등


### 노드 객체의 상속 구조

DOM 노드 객체는 자신의 구조와 정보를 제어할 수 있는 DOM API를 사용하여 노드를 탐색하고 조작할 수 있다.  
DOM을 구성하는 노드객체는 빌트인 객체가 아니라 브라우저 환경에서 추가적으로 제공하는 호스트 객체이다.  
이는 JS 객체이므로 프로토타입에 의한 상속 구조를 갖기때문에 프로토타입 체인에 있는 모든 프로토타입의 프로퍼티, 메서드들을 상속받아 사용할 수 있다.


노드 객체에는 노드 타입에 상관없이 모든 노드 객체가 공통으로 갖는 기능도 있고 타입에 따라 고유한 기능도 있다.  
- 모든 노드 객체
  - 공통적으로 EventTarget 인터페이스에 의해 이벤트를 발생시킬 수 있다.
  - Node 인터페이스에 의해 트리 탐색기능 (parentNode, childNodes, firstChild, lastChild, nextSibling, previousSibling), 노드 정보 제공기능 (nodeType, nodeName, nodeValue)이 필요하다.
- HTML 요소 노드 객체
  - HTMLElement 인터페이스에 의해 HTML 요소의 종류에 따른 고유한 기능을 제공한다.

즉, DOM API는 노드 타입에 따라 필요한 기능을 프로퍼티와 메서드의 집합으로 제공한다. 이를 통해 HTML의 구조, 내용, 스타일을 조작할 수 있는 것이다.

<br>


## 39.2 요소 노드 취득

### (1) id
> id 어트리뷰트 값을 갖는 하나의 요소 노드를 탐색하여 반환한다.

```js
Document.prototype.getElementById(id)
```
- HTML 문서 내에 id값은 유일한 값이어야하므로 중복된 id값을 갖는 HTML요소가 여러개 존재하더라도 에러가 발생하지 않고 첫번째 요소 노드만을 반환한다.
- 요소가 존재하지 않을 경우에는 null을 반환한다.


### (2) tagName
> tagName을 갖는 모든 요소 노드를 탐색하여 HTMLCollection 객체에 담아 반환한다.

#### root 노드인 document를 통해 DOM 전체에서 요소 노드를 탐색할 때

```js
Document.prototype.getElementsByTagName(tagName)
```
- HTMLCollection 객체는 유사배열객체이면서 이터러블이다.
- HTML문서 내 모든 요소를 취득할 땐 인수로 '*'를 전달한다.
- 요소가 존재하지 않을 경우에는 빈 HTMLCollection 객체를 반환한다.

#### 특정 요소 노드의 자손 노드 중에서 요소 노드를 탐색할 때

```js
Element.prototype.getElementsByTagName(tagName)
```
- 특정 노드를 호출하고, 그의 자식 노드를 탐색할 땐 Element.prototype을 사용한다.
  ```js
  const $fruits = document.getElementById('fruits');
  const $li = $fruits.getElementsByTagName('li'); // HTMLCollection(3) [li, li, li]
  ```

### (3) className
> class 값을 갖는 모든 요소 노드를 탐색하여 HTMLCollection 객체에 담아 반환한다.

#### root 노드인 document를 통해 DOM 전체에서 요소 노드를 탐색할 때

```js
Document.prototype.getElementsByClassName(className)
```
- 요소가 존재하지 않을 경우에는 빈 HTMLCollection 객체를 반환한다.

#### 특정 요소 노드의 자손 노드 중에서 요소 노드를 탐색할 때

```js
Element.prototype.getElementsByClassName(className)
```


### (4) CSS선택자를 이용한 querySelector, querySelectorAll
> 인수로 전달한 CSS selector를 만족하는 HTML 요소를 탐색하여 NodeList 객체에 담아 반환한다.


```js
Document.prototype.querySelector('.className')
```
- 인수로 전달한 CSS 선택자를 만족시키는 요소 노드가 여러개 존재하더라도 첫번째 요소 노드만을 반환한다.


```js
Document.prototype.querySelectorAll('.className')
```
- 인수로 전달한 CSS 선택자를 만족시키는 모든 요소 노드를 객체로 갖는 NodeList를 반환한다.


#### CSS 선택자 문법 메서드와 getElementBy~ 메서드의 차이

`querySelector`, `querySelectorAll` 메서드는 `getElementById`, `getElementsBy~` 메서드 보다 탐색 속도가 느리지만, 구체적이고 일관된 조건으로 요소 노드를 취득할 수 있다는 장점을 가진다. 따라서 id 어트리뷰트가 있는 요소를 취득할 때를 제외하고는 `querySelector`, `querySelectorAll` 메서드를 사용하는 것이 좋다고 한다.


### (5) 특정 노드를 취득할 수 있는지 확인하는 matches()
> 인수로 전달한 CSS selector를 통해 특정 요소 노드를 취득할 수 있는지 확인하는 메서드

```js
Element.prototype.matches()
```



### HTMLCollection과 NodeList

DOM 컬렉션 객체인 HTMLCollection과 NodeList는 DOM API가 여러 개의 결과값을 반환하기 위한 DOM 컬렉션 객체이다.  
HTMLCollection과 NodeList는 모두 유사 배열 객체이면서 이터러블이다.  
따라서 `for...of`문으로 순회할 수 있으며 스프레드 문법을 사용하여 간단히 배열로 변환할 수 있다.

#### HTMLCollection

- `getElementsByTagName`, `getElementsByClassName` 메서드가 반환하는 객체 (live객체)
- 실시간으로 Node의 상태 변경을 반영한다. (반복문 사용시 주의해야함)
- loop를 돌면서 DOM을 조작해야되는 경우, 반복문을 역방향으로 돌리거나 while문을 사용하거나 HTMLCollection을 배열로 변환하여 사용하는 것이 좋다.
    ```js
    const elems = document.getElementsByClassName('red');

    // 유사 배열 객체인 HTMLCollection을 배열로 변환한다.
    // 배열로 변환된 HTMLCollection은 더 이상 live하지 않다.
    console.log([...elems]); // [li#one.red, li#two.red, li#three.red]
    [...elems].forEach(elem => elem.className = 'blue');
    ```

#### NodeList

- `querySelectorAll` 메서드가 반환하는 객체 (non-live객체, 특정 경우에만 live객체)
- HTMLCollection 객체와 다르게 실시간으로 노드 객체의 상태변경을 반영하지 않는 객체이다.
- childNodes 프로퍼티를 사용하여 반환되는 객체는 Nodelist이지만 live객체이므로 주의해야한다.
  ```js
  const $fruits = document.getElementById('fruits');
  const { childNodes } = $fruits;
  console.log(childNodes); // NodeList(7) [text, li, text, li, text]
  ```


## 39.3 노드 탐색

DOM 트리의 노드를 탐색할 때는 Node, Element 인터페이스에 정의된 프로퍼티를 사용한다.
- Node.prototype 은 모두 읽기 전용 접근자 프로퍼티이다.
- HTML문서상 공백 문자는 **`공백 텍스트 노드`** 를 생성하므로 노드 탐색시 주의해야한다.

### 자식 노드 탐색 프로퍼티

프로퍼티 | 설명
---|---
`Node.prototype.childNodes` | 자식을 모두 탐색하여 NodeList로 반환 (text node 포함)
`Node.prototype.firstChild` | 첫번째 자식 노드 반환 (element node 또는 text node)
`Node.prototype.lastChild` | 마지막 자식 노드 반환 (element node 또는 text node)
`Element.prototype.children` | 요소 노드만 탐색하여 HTMLCollection으로 반환
`Element.prototype.firstElementChild` | 첫번째 자식 요소 노드 반환
`Element.prototype.lastElementChild` | 마지막 자식 요소 노드 반환
`Node.prototype.hasChildeNodes()` | 자식 노드가 있는지 확인하여 boolean 값을 반환 (text node 포함)
`Element.prototype.childElementCount`, `children.length` | 자식 요소 노드의 개수를 반환 (text node를 제외하고 요소 노드가 있는지 확인할 때 사용)


### 그외 노드 탐색 프로퍼티

프로퍼티 | 설명
---|---
`Node.prototype.parentNode` | 부모 노드 반환
`Node.prototype.previousSibling` | text node를 포함한 이전 형제 노드 반환
`Node.prototype.nextSibling` | text node를 포함한 다음 형제 노드 반환
`Element.prototype.previousElementSibling` | 형제 요소 노드 중에서 이전 형제 요소 노드만 반환
`Element.prototype.nextElementSibling` | 형제 요소 노드 중에서 다음 형제 요소 노드만 반환


## 39.4 노드 정보 취득 프로퍼티

프로퍼티 | 설명
---|---
`Node.prototype.nodeType` | 노드 타입을 나타내는 상수 반환 (1: 요소노드, 3: 텍스트노드, 9: 문서노드)
`Node.prototype.nodeName` | 노드의 이름을 문자열로 반환 (요소노드: 대문자 문자열, 텍스트노드: #text, 문서노드: #document)



## 39.5 요소 노드의 텍스트 조작

### (1) nodeValue
> 노드의 값을 반환, 할당하는 프로퍼티
>
> 텍스트 노드의 유일한 프로퍼티이다.

- 텍스트 노드의 텍스트 값을 반환하는 메서드로 텍스트 노드에만 텍스트를 반환하고
- 텍스트 노드가 아닌 노드 (문서노드, 요소노드)에 nodeValue 프로퍼티를 참조할 경우 null을 반환한다. (텍스트 노드가 아닌 노드에는 의미X)
- *텍스트 노드의 경우는 문자열, 요소 노드의 경우 null 반환*
- 텍스트 노드의 nodeValue property에 값을 할당하면 텍스트 노드값을 변경할 수 있다.
  ```js
  // ❌ 문서 노드의 nodeValue 프로퍼티를 참조
  console.log(document.nodeValue); // null
  // ❌ 요소 노드의 nodeValue 프로퍼티를 참조
  const $foo = document.getElementById('foo');
  console.log($foo.nodeValue); // null

  // ✅ 텍스트 노드의 nodeValue 프로퍼티를 참조할 수 있음
  const $textNode = $foo.firstChild;
  console.log($textNode.nodeValue); // Hello

  // nodeValue 프로퍼티를 사용하여 텍스트 노드의 값을 변경할 수 있음
  $textNode.nodeValue = 'World';
  console.log($textNode.nodeValue); // World
  ```


### (2) textContent
> 요소 노드의 텍스트와 자식 요소 노드의 텍스트를 모두 반환, 할당하는 프로퍼티

- 요소 노드의 텍스트를 모두 반환한다. (마크업 무시)
- nodeValue를 사용했을 땐 텍스트 노드가 아닌 노드일땐 텍스트 값을 취득하는 과정이 더 복잡하지만 textContent 프로퍼티를 사용하면 한번에 요소의 콘텐츠 영역에 있는 텍스트 값을 취득할 수 있다.
- 요소 노드의 textContent 프로퍼티에 문자열을 할당하면 해당 요소의 자식 요소를 제거한 뒤 할당한 문자열을 텍스트로 할당된다.
  - 순수한 텍스트만 지정해야 하며 마크업을 포함시키면 문자열로 인식되어 그대로 출력된다.
  - HTML 파싱X


```js
<div id="foo">Hello <span>world!</span></div>

// #foo 요소 노드의 모든 자식 노드가 제거되고 할당한 문자열이 파싱되지않고 텍스트로 추가됨
document.getElementById('foo').textContent = 'Hi <span>there!</span>';
```

***마크업이 포함된 콘텐츠를 추가하는 행위는 크로스 스크립팅 공격(XSS: Cross-Site Scripting Attacks)에 취약하므로 주의가 필요하다.***


### (3) innerText

- textContent와 유사한 동작을 하지만 CSS에 의해 숨겨진 요소의 텍스트는 반환하지 않는다.
- innerText는 CSS에 순종적이기때문에 textContent보다 성능이 떨어진다.
- 비표준이다.



## 39.6 DOM 조작
> 새로운 노드를 생성하여 추가하거나 기존 노드를 삭제 또는 변경하는 것

- DOM이 조작되어 새노드가 추가, 삭제되는 경우 reflow, repaint가 발생한다.  
  (노드의 속성이 변경되는 경우 repaint만 발생)



### (1) innerHTML
> 요소 노드의 콘텐츠 영역에 대한 HTML 마크업을 문자열로 반환, 할당하는 프로퍼티

- 해당 요소의 모든 자식 요소를 포함하는 모든 콘텐츠를 하나의 문자열로 취득할 수 있다. (마크업 포함)
  ```js
  // #foo 요소의 콘텐츠 영역 내의 HTML 마크업을 문자열로 취득
  console.log(document.getElementById('foo').innerHTML); // "Hello <span>world!</span>"
  ```
- textContent와의 차이점 : HTML 마크업이 포함된 문자열을 그대로 반환한다. (textContent는 마크업 제외하고 텍스트만 반환)
- innerHTML 프로퍼티에 문자열을 할당하면 자식 노드 제거하고 할당한 문자열에 마크업이 있으면 파싱되어 요소의 자식 노드로 DOM에 반영할 수 있어 HTML 마크업 문자열로 DOM 조작이 매우 간편하다는 장점을 가진다. (추가, 교체, 삭제가 매우 쉬움)
    ```js
    <ul id="fruits">
      <li class="apple">Apple</li>
    </ul>

    const $fruits = document.getElementById('fruits');
    // 노드 추가
    $fruits.innerHTML += '<li class="banana">Banana</li>';
    // 노드 교체
    $fruits.innerHTML = '<li class="orange">Orange</li>';
    // 노드 삭제
    $fruits.innerHTML = '';
    ```
- 렌더링 엔진에 의해 파싱된 요소 노드가 DOM에 바로 반영할 수 있지만, 한편으론 사용자로부터 입력받은 데이터를 바로 할당하는 것은 XSS(Cross-Site Scripting Attacks) 공격에 취약하므로 주의해야한다. (JS에 악성 코드가 있을 때 파싱 과정에 그대로 실행될 수 있는 위험성이 있다.)
- DOM조작이 간단하고 직관적인 반변 크로스 사이트 스크립팅 공격(XSS)에 취약한 단점을 갖는다.

### (2) outerHTML
> outerHTML에서 지정한 요소 태그도 포함하여 값을 가져오는 프로퍼티
> 자기자신을 포함한다.

```js
alert(document.getElementById('test').outerHTML); // <div id="test">TEST</div>
```

### (3) insertAdjacentHTML(position, text)
> 특정 위치의 DOM 트리에 원하는 node를 추가하는 메서드

- 인자로 전달한 텍스트를 HTML로 파싱하고 그 결과로 생성된 노드를 DOM 트리의 지정된 위치에 삽입한다.

- 매개변수
  - position : 삽입할 위치를 지정하는 문자열로, 다음 중 하나를 지정할 수 있다.
    - 'beforebegin' : 특정 요소 앞에 삽입 (컨테이너 밖도 가능)
    - 'afterbegin' : 특정 요소의 첫번째 자식으로 삽입
    - 'beforeend' : 특정 요소의 마지막 자식으로 삽입
    - 'afterend' : 특정 요소 뒤에 삽입 (컨테이너 밖도 가능)
<p align="center"><img src="https://poiemaweb.com/img/insertAdjacentHTML-position.png" width="300px"/></p>

- 기존 요소에는 영향을 주지않고 삽입될 요소만 파싱하여 자식 요소로 추가하는 메서드이기때문에 innerHTML 프로퍼티보다 효율적이로 빠른 장점을 가진다. 하지만 HTML을 파싱하기 때문에 XSS 공격에 취약하다는 단점은 동일하다.

- 참고 : [insertAdjacentText](https://developer.mozilla.org/en-US/docs/Web/API/Element/insertAdjacentText), [insertAdjacentElement](https://developer.mozilla.org/en-US/docs/Web/API/Element/insertAdjacentElement)


### 39.6.3 노드 생성, 추가

DOM 은 노드를 직접 생성, 삽입, 삭제, 치환하는 메서드를 제공한다.  
(이전의 HTML 마크업 문자열을 파싱하여 노드를 생성하고 DOM에 반영하는 방식과 다름)

```js
const $fruits = document.getElementById('fruits');

// 1. 요소 노드 생성
const $li = document.createElement('li');

// 2. 텍스트 노드 생성
const textNode = document.createTextNode('Banana');

// 3. 텍스트 노드를 $li 요소 노드의 자식 노드로 추가
$li.appendChild(textNode);

// 4. 생성한 li 요소 노드를 #fruits 요소 노드의 마지막 자식 노드로 추가
$fruits.appendChild($li);
```

### (1) Document.prototype.createElement(tagName)
> 요소 노드를 생성하는 메서드

- 매개변수로 태그 이름을 전달하여 요소 노드를 생성한다.
- DOM 추가되지는 않고 생성만 된 상태로 따로 자식 노드들을 추가해줘야한다.

### (2) Document.prototype.createTextNode(text)
> 텍스트 노드를 생성하는 메서드

- 매개변수로 텍스트를 전달하여 텍스트 노드 값으로 사용한다.
- 자식 노드로 추가되지는 않고 텍스트 노드를 생성한 상태로 요소 노드에 추가해줘야한다.

### (3) Node.prototype.appendChild(childnode)
> 자식 노드를 추가하는 메서드

- 매개변수로 추가할 자식 노드를 전달하면 호출한 노드의 마지막 자식 노드로 추가한다.
- 텍스트 노드를 생성하여 부모 요소 노드에 텍스트 노드를 마지막 자식 노드로 추가할 수 있지만 자식 노드가 아예 없는 부모 요소의 경우에는 노드를 생성하는 과정이 필요없는 textContent 프로퍼티를 사용해서 바로 할당하는 방법이 더 간편하다.
- 이미 존재하는 노드를 DOM에 다시 추가하는 경우 현재 위치에서 제거, 새위치에 추가하게 되어 노드 이동 기능을 한다.


#### 노드를 여러개 생성하고 추가해야할 때 리플로우, 리페인트를 최소화하는 방법

- createElement 메서드로 요소를 생성하고 appendChild 메서드로 DOM에 추가하는 과정을 반복하면 반복할 수록 DOM이 그만큼 변경되어 리플로우, 리페인트가 발생한다. (비용 ⇧)
- **`DocumentFragment`** 를 사용한다.
  - DocumentFragment 노드를 생성하고, DOM에 추가할 요소 노드들을 생성하고 자식 노드로 추가한 다음 기존 DOM에 DocumentFragment 노드를 추가한다.
- 
    ```js
    const $fruits = document.getElementById('fruits');

    // ✅ DocumentFragment 노드 생성한 뒤 반복해서 생성
    const $fragment = document.createDocumentFragment();

    ['Apple', 'Banana', 'Orange'].forEach(text => {
      // 1. 요소 노드 생성
      const $li = document.createElement('li');
      // 2. 텍스트 노드 생성
      const textNode = document.createTextNode(text);
      // 3. 텍스트 노드를 $li 요소 노드의 자식 노드로 추가
      $li.appendChild(textNode);
      // 4. $li 요소 노드를 DocumentFragment 노드의 마지막 자식 노드로 추가
      $fragment.appendChild($li);
    });

    // 5. DocumentFragment 노드를 마지막 자식 노드로 추가
    $fruits.appendChild($fragment);
    ```
- 이렇게 하면 DOM은 1번만 변경되고 리플로우, 리페인트가 1번만 발생하므로 훨씬 효율적이다.

<p align="center"><img src="https://github.com/bread1022/TIL/assets/107349637/34f8e088-089a-4530-a61e-dc1323b81986" width="450px"/></p>



### (4) Node.prototype.insertBefore(newNode, childNode)
> 특정 위치에 노드 삽입하는 메서드

- 첫번째 인수로 받은 노드를 두번째 인수로 전달받은 노드 바로 앞에 삽입한다.
- 따라서 두번째 인수로 전달받은 인수는 메서드를 호출한 노드의 자식이어야한다.
- 두번째 인수가 null 이면 호출한 노드의 마지막 자식으로 추가된다 (= appendChild처럼 동작)
- 이미 존재하는 노드를 DOM에 다시 추가하는 경우 현재 위치에서 제거, 새위치에 추가하게 되어 노드 이동 기능을 한다.

  ```js
  const $fruits = document.getElementById('fruits');

  // 요소 노드 생성
  const $li = document.createElement('li');

  // 마지막 자식 노드로 추가
  $li.appendChild(document.createTextNode('Orange'));

  // 마지막 자식 노드로 추가
  $fruits.insertBefore($li, null);
  ```


### (5) Node.prototype.cloneNode([deep: true | false])
> 노드를 복제하는 메서드

- deep 인수를 생략하거나 false를 전달하면 노드를 얕은 복사하여 노드 자신만의 사본을 생성(자손 노드 포함X)하고
- deep 인수를 true로 전달하면 노드를 깊은 복사하여 모든 자손 노드가 포함된 사본을 생성한다.


### (6) Node.prototype.replaceChild(newChild, oldChild)
> 노드를 교체하는 메서드

- 호출한 노드의 자식노드를 다른 노드로 교체한다.
- 첫번째 인수에는 새로 교체할 노드를, 두번째 인수에는 호출 노드의 자식 노드에서 교체될 노드를 전달한다.


### (7) Node.prototype.removeChild(childNode)
> 노드를 삭제하는 메서드

- 인수로 전달한 노드를 DOM에서 삭제한다.
- 호출한 노드의 자식노드를 전달해야한다.


#### innerHTML vs. DOM 조작 방식 vs. insertAdjacentHTML()

innerHTML | DOM 조작 방식 | insertAdjacentHTML()
---|---|---
문자열로 정의한 여러 요소를 DOM에 추가 | 노드를 생성하여 DOM에 추가 | 문자열로 정의된 여러 요소를 DOM에 추가, 삽입 위치를 선정할 수 있음
HTML 파싱과정이 있음 | 특정 노드 한개 추가할 때 적합 | HTML 파싱과정이 있음
XSS 공격에 취약 | innerHTML보다 느리고 더 많은 코드가 필요함 | XSS 공격에 취약


## 39.7 어트리뷰트

- 어트리뷰트는 읽기 전용 프로퍼티로
- 요소 노드 attribute 프로퍼티는 모든 노드의 어트리뷰트 참조가 담긴 NamedNodeMap 객체를 반환한다.

  ```js
  // 요소 노드의 attribute 프로퍼티는 요소 노드의 모든 어트리뷰트 노드의 참조가 담긴 NamedNodeMap 객체를 반환한다.
  const { attributes } = document.getElementById('user');
  console.log(attributes);

  // NamedNodeMap {0: id, 1: type, 2: value, id: id, type: type, value: value, length: 3}

  // 어트리뷰트 값 취득
  console.log(attributes.id.value); // user
  console.log(attributes.type.value); // text
  console.log(attributes.value.value); // ungmo2
  ```
- attribute 값을 참조: `Element.prototype.getAttribute(attributeName)`
- attribute 값을 변경: `Element.prototype.setAttribute(attributeName, value)`
- attribute 존재하는 지 확인: `Element.prototype.hasAttribute(attributeName)`
- attribute 값을 제거: `Element.prototype.removeAttribute(attributeName)`


<!-- 39.7.3 HTML어트리뷰트, DOM 프로퍼티 -->

### 🌟 data 어트리뷰트와 dataset 어트리뷰트

- data 어트리뷰트와 dataset 프로퍼티를 사용하면 HTML요소에 정의한 사용자 정의 어트리뷰트와 JS 데이터를 교환할 수 있다.
- data attribute 값은 HTMLElement.dataset 프로퍼티로 취득하고 값을 변경할 수 있다.
  - HTML 요소의 모든 data attribute 정보를 제공하는 DOMStringMap 객체를 반환한다.
- data 어트리뷰트는 data-user-id, data-role과 같이 data- 접두사 다음에 임의의 이름을 붙여 사용한다.


```js
<ul class="users">
  <li id="1" data-user-id="1번" data-role="admin">Lee</li>
  <li id="2" data-user-id="2번" data-role="subscriber">Kim</li>
</ul>

const users = [...document.querySelector('.users').children];

// user-id가 '1번'인 요소 노드를 취득
const user = users.find(user => user.dataset.userId === '1번');

// user-id가 '1번'인 요소 노드의 data-role 값을 취득
console.log(user.dataset.role); // "admin"

// 값을 변경
user.dataset.role = 'subscriber';

// dataset 프로퍼티는 DOMStringMap 객체를 반환
console.log(user.dataset); // DOMStringMap {userId: "1번", role: "subscriber"}
```

## 39.8 스타일

### HTMLElement.prototype.style
> 요소 노드의 인라인 스타일을 취득, 추가, 변경하는 프로퍼티

- style 프로퍼티를 사용하면 inline 스타일 선언을 생성한다.
- 특정요소에 inline 스타일을 지정해야될 때 사용한다.
- style 프로퍼티를 참조하면 CSSStyleDeclaration 객체를 반환한다.
  - 이 프로퍼티에 값을 할당하면 CSS 프로퍼티가 인라인 스타일로 HTML 요소에 추가되거나 변경된다.
  - CSS 프로퍼티는 케밥케이스로, CSSStyleDeclaration 객체의 프로퍼티는 카멜케이스로 작성한다.
  - 단위가 필요한 프로퍼티의 경우에는 반드시 단위를 지정해야되고 지정 하지않으면 적용되지 않는다.

```js
<div style="color: red">Hello World</div>

const $div = document.querySelector('div');

// 인라인 스타일 취득
console.log($div.style); // CSSStyleDeclaration { 0: "color", ... }

// 인라인 스타일 변경
$div.style.color = 'blue';

// 인라인 스타일 추가
$div.style.width = '100px'; // 단위 지정 필수
$div.style.height = '100px';
$div.style.backgroundColor = 'yellow';
$div.style['background-color'] = 'green'; // 케밥케이스로 작성하려면 대괄호표기법을 사용해야함
```

### Element.prototype.className
> HTML요소의 class 어트리뷰트 값을 취득, 변경하는 프로퍼티

- className 프로퍼티를 참조하면 어트리뷰트 값을 문자열로 반환하고, 값을 할당하면 어트리뷰트 값을 변경한다.
  ```js
  const $box = document.querySelector('.box');
  // .box 요소의 class 어트리뷰트 값을 취득
  console.log($box.className); // 'box red'
  // .box 요소의 class 어트리뷰트 값 중에서 'red'만 'blue'로 변경
  $box.className = $box.className.replace('red', 'blue');
  ```


### Element.prototype.classList
> HTML요소의 class 어트리뷰트 정보를 담은 DOMTokenList 객체를 반환하는 프로퍼티

- `element.classList.add(...className)`: 1개 이상의 class 어트리뷰트를 추가
- `element.classList.remove(...className)`: 1개 이상의 class 어트리뷰트를 제거 (일치하는 클래스가 없을 경우 에러없이 무시됨)
- `element.classList.item(index)`: 인수로 전달한 인덱스에 해당하는 클래스 어트리뷰트 값을 반환
- `elemeent.classList.contains(...className)`: 인수로 전달한 클래스 어트리뷰트가 존재하는지 확인하여 boolean 값을 반환
- `element.classList.replace(oldClass, newClass)`: 인수로 전달한 클래스를 두번째 인수로 전달한 클래스로 교체
- `element.classList.toggle(...className)`: 인수로 전달한 클래스 어트리뷰트가 존재하면 제거하고, 존재하지 않으면 추가한다.


### window.getComputedStyle(element, [pseudoElt])
> HTML요소에 적용되어 있는 모든 CSS 스타일을 참조해야할 때 사용하는 메서드

- 첫번째 인수로 전달한 요소 노드에 적용되어 평가된 스타일을 CSSStyleDeclaration 객체로 반환하는 메서드이다.
- 첫번째 인수로 참조할 HTML 요소 노드를 전달하고, 두번째 인수로는 가상 요소를 지정할 수 있다.
- 평가된 스타일이란, 요소 노드에 적용되어 있는 모든 스타일 (link, inline, user agent 등등)이 조합되어 최종적으로 적용된 스타일을 말한다.

  ```css
  <style>
    body {
      color: red;
    }
    .box {
      width: 100px;
      height: 50px;
      background-color: cornsilk;
      border: 1px solid black;
    }
  </style>
  ```
  ```js
  const $box = document.querySelector('.box');

  // CSSStyleDeclaration 객체 (.box의 모든 CSS스타일)
  const computedStyle = window.getComputedStyle($box);

  // 임베딩 스타일
  console.log(computedStyle.width); // 100px
  console.log(computedStyle.height); // 50px
  console.log(computedStyle.backgroundColor); // rgb(255, 248, 220)
  console.log(computedStyle.border); // 1px solid rgb(0, 0, 0)
  // 상속 스타일(body -> .box)
  console.log(computedStyle.color); // rgb(255, 0, 0)
  // 기본 스타일
  console.log(computedStyle.display); // block
  ```