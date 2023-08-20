# 37장. Set 과 Map

## 37.1 Set 객체
> 중복되지 않는 유일한 값들의 집합

- 배열과 유사하지만 동일한 값을 중복해서 포함할 수 없다.
- 요소 순서에 의미가 없어 인덱스로 접근할 수 없다.
- 수학적 집합을 구현하기 위한 자료구조로 사용되어 집합 연산을 수행할 수 있다.

### 37.1.1 Set 객체의 생성

- `new` 연산자와 함께 `Set` 생성자 함수를 호출하여 생성한다.
- 인수를 전달하지 않으면 빈 Set객체가 생성되고, 이터러블을 인수로 전달받으면 전달받은 이터러블 내 중복된 값을 제거하여 Set객체를 생성한다.
  ```js
  const set = new Set();
  console.log(set); // Set(0) {}
  ```
  ```js
  const set1 = new Set([1, 2, 3, 4]);
  console.log(set1); // Set(4) {1, 2, 3, 4}
  ```
- 중복을 허용하지 않는 특성을 이용해 배열 내 중복제거할 때 유용하게 쓰인다.
  ```js
  const filter = (...args) => [...new Set(args)];
  console.log(filter(1, 2, 3, 4, 4, 4, 5)); // [1, 2, 3, 4, 5]
  ```

### 37.1.2 Set 객체 메서드

- **요소 개수 확인** : **`Set.prototype.size`**
    ```js
    const { size } = new Set([1, 2, 3]);
    console.log(size); // 
    ```
- **요소 추가** : **`Set.prototype.add`**
  ```js
  const set = new Set(); // Set(0) {}
  set.add(10);
  console.log(set); // Set(1) {10}
  ```
  - 중복된 요소는 추가되지 않고 무시된다.
    ```js
    const set = new Set(); // Set(0) {}
    set.add(10).add(10).add('a').add(undefined).add(NaN);
    console.log(set); // Set(4) {10, 'a', undefined, NaN}
    ```
- **요소 존재 여부 확인** : **`Set.prototype.has`** boolean값 반환
  ```js
  const set = new Set([1, 2, 3]);
  console.log(set.has(2)); // true
  console.log(set.has(4)); // false
  ```
- **요소 삭제** : **`Set.prototype.delete`** 삭제 성공 여부에 대한 boolean값 반환
  ```js
  const set = new Set([1, 2, 3]);
  console.log(set.delete(2)); // true
  console.log(set); // Set(2) {1, 3}
  console.log(set.delete(4)); // false 에러가 발생하지는 않음
  console.log(set); // Set(2) {1, 3}
  ```
- **요소 일괄 삭제** : **`Set.prototype.clear`** 항상 `undefined` 반환
  ```js
  const set = new Set([1, 2, 3]);
  console.log(set.clear()); // undefined
  console.log(set); // Set(0) {}
  ```
- **요소 순회**
  - **`Set.prototype.forEach`**
    - 첫번째인수 : 현재 요소
    - 두번째인수 : 현재 요소
    - 세번째인수 : 현재 Set객체 자체
      ```js
      const set = new Set([1, 2, 3]);
      set.forEach((v1, v2, set) => console.log(v1, v2, set));
      // 1 1 Set(3) {1, 2, 3}
      // 2 2 Set(3) {1, 2, 3}
      // 3 3 Set(3) {1, 2, 3}
      ```
  - **`for...of`문**
    ```js
    const set = new Set([1, 2, 3]);
    for (const value of set) {
      console.log(value); // 1 2 3
    }
    ```
  - `Set.prototype.entries()` : [key, value]쌍의 반복가능한 이터러블 객체를 반환
  - `Set.prototype.keys()` : 각 요소의 key를 모은 이터러블 객체를 반환
  - `Set.prototype.values()` : 각 요소의 value를 모은 이터러블 객체를 반환
- **집합 연산**
  - **교집합**
    ```js
    const set1 = new Set([1, 2, 3]);
    const set2 = new Set([2, 3, 4]);
    const intersection = new Set([...set1].filter(v => set2.has(v)));
    console.log(intersection); // Set(2) {2, 3}
    ```
  - **합집합**
    ```js
    const set1 = new Set([1, 2, 3]);
    const set2 = new Set([2, 3, 4]);
    const union = new Set([...set1, ...set2]);
    console.log(union); // Set(4) {1, 2, 3, 4}
    ```
  - **차집합**
    ```js
    const set1 = new Set([1, 2, 3]);
    const set2 = new Set([2, 3, 4]);
    const difference = new Set([...set1].filter(v => !set2.has(v)));
    console.log(difference); // Set(1) {1}
    ```


<br>

## 37.2 Map
> 키와 값의 쌍으로 이루어진 컬렉션

- 객체와 유사하지만 Map객체는 이터러블에 속한다.
- 객체의 키는 반드시 문자열 또는 심벌값이어야하지만 Map의 키는 모든 값을 허용한다.


### 37.2.1 Map 객체의 생성

- `new` 연산자와 함께 `Map` 생성자 함수를 호출하여 생성한다.
- 인수를 전달하지 않으면 빈 Map객체가 생성되고, 이터러블을 인수로 전달해야하는데 이때 키와 값의 쌍으로 이루어진 요소로 구성되어야한다.
  ```js
  const map = new Map();
  console.log(map); // Map(0) {}
  ```
  ```js
  const map = new Map([
    ['key1', 'value1'],
    ['key2', 'value2'],
  ]);
  console.log(map); // Map(2) {'key1' => 'value1', 'key2' => 'value2'}
  ```


### 37.2.2 Map 객체 메서드

- **요소 개수 확인** : **`Map.prototype.size`**
    ```js
    const { size } = new Map([['key1', 'value1'], ['key2', 'value2']]); // Map(2) {'key1' => 'value1', 'key2' => 'value2'}
    console.log(size); // 2
    ```
- **요소 추가** : **`Map.prototype.set`**
  ```js
  const map = new Map();
  map.set('key1', 'value1');
  console.log(map); // Map(1) {'key1' => 'value1'}
  ```
  ```js
  const map = new Map();
  map.set('key1', 'value1').set('key2', 'value2');
  console.log(map); // Map(2) {'key1' => 'value1', 'key2' => 'value2'}
  ```
  - Map 객체의 Key는 중복을 허용하지 않기 때문에 기존에 존재하는 키를 추가하면 기존 키에 해당하는 값이 나중에 추가된 값으로 갱신된다.
    ```js
    const map = new Map();
    map.set('key1', 'value1').set('key1', 'value100');
    console.log(map); // Map(1) {'key1' => 'value100'}
    ```
  - Key 타입 제한이 없어 객체도 키로 사용할 수 있다.
    ```js
    const map = new Map();
    const keyObj = { name: 'Kim' };
    map.set(keyObj, 'value');
    console.log(map); // Map(1) {{name: 'Lee'} => 'value'}
    ```
- **요소 얻기** : **`Map.prototype.get`** 키에 해당하는 값을 반환하고 없으면 `undefined`를 반환
  ```js
  const map = new Map([['key1', 'value1'], ['key2', 'value2']]);
  console.log(map.get('key1')); // value1
  console.log(map.get('key10')); // undefined
  ```

- **요소 존재 여부 확인** : **`Map.prototype.has`** boolean값 반환
  ```js
  const map = new Map([['key1', 'value1'], ['key2', 'value2']]);
  console.log(map.has('key1')); // true
  console.log(map.has('key2')); // true
  console.log(map.has('key100')); // false
  ```
- **요소 삭제** : **`Map.prototype.delete`** 삭제 성공 여부에 대한 boolean값 반환
  ```js
  const map = new Map([['key1', 'value1'], ['key2', 'value2']]);
  console.log(map.delete('key1')); // true
  console.log(map); // Map(1) {'key2' => 'value2'}
  console.log(map.delete('key1000')); // false 에러가 발생하지는 않음
  console.log(map); // Map(1) {'key2' => 'value2'}
  ```

- **요소 일괄 삭제** : **`Map.prototype.clear`** 항상 `undefined` 반환
  ```js
  const map = new Map([['key1', 'value1'], ['key2', 'value2']]);
  console.log(map.clear()); // undefined
  console.log(map); // Map(0) {}
  ```

- **요소 순회**
  - **`Map.prototype.forEach`**
    - 첫번째인수 : 현재 요소 값
    - 두번째인수 : 현재 요소의 키
    - 세번째인수 : 현재 Map객체 자체
      ```js
      const map = new Map([['key1', 'value1'], ['key2', 'value2']]);
      map.forEach((value, key, map) => console.log(value, key, map));
      // value1 key1 Map(2) {'key1' => 'value1', 'key2' => 'value2'}
      // value2 key2 Map(2) {'key1' => 'value1', 'key2' => 'value2'}
      ```
  - **`for...of`문**
    ```js
    const map = new Map([['key1', 'value1'], ['key2', 'value2']]);
    for (const [key, value] of map) {
      console.log(key, value); // key1 value1 key2 value2
    }
    ```
  - `Map.prototype.entries()` : [key, value]쌍의 반복가능한 이터러블이면서 동시에 이터레이터인 객체를 반환
  - `Map.prototype.keys()` : 각 요소의 key를 모은 이터러블이면서 동시에 이터레이터인 객체를 반환
  - `Map.prototype.values()` : 각 요소의 value를 모은 이터러블이면서 동시에 이터레이터인 객체를 반환