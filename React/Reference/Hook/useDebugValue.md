# useDebugValue

> React 개발자 도구에서 Custom hook에 label을 추가해주는 Hook

디버그의 값을 표시하려면 Custom hook 최상위 레벨에서 `useDebugValue`를 호출하여 사용한다.

```js
import { useDebugValue } from 'react';

function useOnlineStatus() {
  // ...
  useDebugValue(isOnline ? 'Online' : 'Offline');
  // ...또는
  useDebugValue(date, (date) => date.toDateString());
}
```

## Parameters

- `value`: 개발자 도구에 표시할 값
- `format?`: 포맷팅 함수를 지정하면 컴포넌트 검사시 포매팅 함수를 호출한 다음 디버그 값을 매개변수로 받고 변환된 값을 반환한다. (디버그값의 형식 지정을 연기할 수 있음)

## Return

- 없음

## Custom Hook에 Label 추가하기

```js
import { useDebugValue } from 'react';

function useOnlineStatus() {
  useDebugValue(isOnline ? 'Online' : 'Offline');
}
```

- `useDebugValue`를 호출하지 않으면 true만 표시되고, 호출하면 value로 전달한 값이 표시된다.
- 모든 커스텀 훅에 디버그를 추가하기 보단 내부 데이터가 복잡한 커스텀훅에 유용하다.

<!-- ? 근데 왜 커스텀훅에만 유용한가..? 일반 컴포넌트는 안하는건가??..-->
