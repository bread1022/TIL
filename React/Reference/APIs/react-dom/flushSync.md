# flushSync

> 강제로 제공된 Callback의 모든 업데이트를 동기적으로 비워야할 때 사용하는 API

```js
flushSync(callback);
```

React가 보류중이던 작업을 강제로 비우고 DOM을 동기적으로 업데이트한다.

## Parameters

- `callback`: 즉시 실행할 Callback 함수로 여기에 포함된 모든 업데이트를 동기적으로 비우고 DOM을 업데이트한다.  
  또한 보류중인 모든 업데이트, Effect 또는 Effect 내부의 업데이트를 flush할 수도 있다.  
  이 flushSync 호출 결과로 업데이트가 일시 중단되면 폴백이 다시 표시될 수 있다.

## Returns

- `undefined`를 반환한다.

## 주의사항

- 성능을 크게 저하시킬 수 있으므로 주의해서 사용해야 한다.
- 보류중인 Suspense를 강제 fallback state 표시할 수 있다.
- 보류중인 Effect들을 실행하고 반환하기 전 포함된 모든 업데이트를 동기적으로 적용할 수 있다.
- 콜백 내부의 업데이트를 비우기위해 필요한 경우 콜백 외부의 업데이트도 비워질 수 있다.
- 거의 사용하지 않지만 보통 브라우저 API나 UI라이브러리와 같은 서드파드 코드와 통합할 때  
  이 콜백이 끝날 때 콜백 내부의 결과가 DOM에 동기적으로 업데이트되도록 할 수 있다.

  ```js
  import { useState, useEffect } from 'react';
  import { flushSync } from 'react-dom';

  export default function PrintApp() {
    const [isPrinting, setIsPrinting] = useState(false);

    useEffect(() => {
      function handleBeforePrint() {
        flushSync(() => setIsPrinting(true));
      }

      function handleAfterPrint() {
        setIsPrinting(false);
      }

      // 🌟 beforeprint 이벤트가 발생하기 직전에 사용자 지정 인쇄 스타일을 적용할 때 사용
      window.addEventListener('beforeprint', handleBeforePrint);
      window.addEventListener('afterprint', handleAfterPrint);
      return () => {
        window.removeEventListener('beforeprint', handleBeforePrint);
        window.removeEventListener('afterprint', handleAfterPrint);
      };
    }, []);

    return (
      <>
        <h1>isPrinting: {isPrinting ? 'yes' : 'no'}</h1>
        <button onClick={() => window.print()}>Print</button>
      </>
    );
  }
  ```
