# Updating Arrays in State

- [ ] React state 안의 배열 항목을 추가, 제거 또는 변경하는 방법
- [ ] 배열 내부의 객체를 업데이트 하는 방법
- [ ] Immer로 덜 반복적으로 배열을 복사하는 방법


## Updating arrays without mutation
> 변이 없이 배열 업데이트하기

- 배열도 객체와 마찬가지로 읽기전용으로 취급해야하고 배열 내부 항목을 직접 재할당하거나 `push()`, `pop()`메서드를 사용하면 안된다.
- `state`의 원래배열에서 `filter()`, `map()`같은 비변이 메서드를 사용하여 새배열을 생성해야한다.
- 배열의 요소를 추가할 때: `spread syntax`
- 배열의 요소를 제거할 때: `filter()`, `slice()`
- 배열의 요소를 교체할 때: `map()`
- 배열을 정렬할 때: copy 후 `sort()`


## Adding to an array

- 배열의 맨 뒤에 추가하기 **`spread syntax`**
  ```jsx
  setArtists([
      ...artists,
      { id: nextId++, name: name }
  ]);
  ```
- 배열의 맨 앞에 추가하기 **`spread syntax`**
  ```jsx
  setArtists([
    { id: nextId++, name: name },
    ...artists // Put old items at the end
  ]);
  ```

## Removing from an array

- 특정 항목을 포함하지 않는 새배열 생성하기 **`filter()`**
  ```jsx
  setArtists(
    artists.filter(a => a.id !== artist.id)
  );
  ```

## Transforming an array

- 배열의 특정 항목만을 변경하기 **`map()`**
  ```jsx
  setArtists(
    artists.map(a => {
      if (a.id === artist.id) {
        return artist;
      } else {
        return a;
      }
    })
  );
  ```

## Replacing items in an array

- 배열에서 하나 이상의 항목을 변경하기 **`map()`**
  ```jsx
  const nextCounters = counters.map((c, i) => {
    if (i === index) {
      return c + 1;
    } else return c;
  });
  ```

## Inserting into an array

- 배열에서 맨 앞, 맨 뒤가 아닌 특정 위치에 항목을 삽입하기 **`slice()`** + **`spread syntax`**
  ```jsx
  const nextCounters = [
    ...counters.slice(0, index),
    0,
    ...counters.slice(index)
  ];
  ```
  - 항목 삽입 지점 앞에 `slice`를 `spread`한 다음 새 항목을 추가하고, 나머지 원래 배열을 펼치는 배열을 생성한다.

## Making other changes to an array

- 배열을 정렬하고 싶을 땐 원래 배열을 복사한 다음 `sort()`를 호출한다.
  ```jsx
  const nextCounters = [...counters].sort();
  ```


## Updating objects inside arrays

- 중첩된 배열 내부 객체도 업데이트 하려는 지점부터 최상위 수준까지 복사본을 만들고 업데이트해야한다.
  ```jsx
  const initialList = [
    { id: 0, title: 'Big Bellies', seen: false },
    { id: 1, title: 'Lunar Landscape', seen: false },
    { id: 2, title: 'Terracotta Army', seen: true },
  ];
  ```
  ```jsx
  const myNextList = [...myList];
  const artwork = myNextList.find(a => a.id === artworkId);
  artwork.seen = nextSeen; // 변이 발생
  setMyList(myNextList);
  ```
  ```jsx
  setMyList(myList.map(artwork => { // 방금 만든 객체를 변이
    if (artwork.id === artworkId) {
      return { ...artwork, seen: nextSeen };
    } else {
      return artwork;
    }
  }));
  ```
  - 아예 새로운 객체를 넣는 것이 아니라 기존의 `state`를 이용해야한다면 배열을 변이하는 대신 새로운 버전을 생성하고 `state`를 업데이트해야한다.
