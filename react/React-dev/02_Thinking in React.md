# 2. Thinking in React

#### 리액트로 사고하는 방법
1. 컴포넌트 계층 구조를 분리하기
2. 정적인 UI 생성하기
3. 최소한의 state 정의하기
4. state의 위치 파악하기
5. 역방향 데이터 흐름 추가하기


### Step 1: Break the UI into a component hierarchy
> UI를 컴포넌트 계층 구조로 나누기

- 컴포넌트 분할은 **단일 책임 원칙** 기법을 적용하여 컴포넌트는 이상적으로 한가지 일만 수행하도록 컴포넌트가 늘어나면 더 작은 하위 컴포넌트로 분해해야한다.
- 컴포넌트 분할 예시
  <p align="center"><img src="https://react-ko.dev/images/docs/s_thinking-in-react_ui_outline.png" width=500px></p>

       FilterableProductTable
        └ SearchBar
        └ ProductTable
          └ ProductCategoryRow
          └ ProductRow


### Step 2: Build a static version in React
> React로 정적인 UI 만들기

- 컴포넌트를 계층구조로 나눈다음에는 정적 버전을 생성한다.  
  재사용하는 컴포넌트를 만들고 props를 이용하여 데이터를 전달한다. (state는 아직 사용하지 말 것)
- 컴포넌트를 생성하는 방향은 계층구조에서 상위 컴포넌트로부터 하향식으로 만들거나, 하위 컴포넌트부터 상향식으로 만들 수 있다. 보통 하향식이 더 쉽지만 대규모 프로젝트에서는 상향식으로 진행하는 것이 더 쉽다.


### Step 3: Find the minimal but complete representation of UI state
> 최소한의 완전한 UI state 찾기

- 상호작용되는 UI를 만들기 위해서는 유저가 데이터모델을 변경할 수 있어야한다. 이때 리액트에서 `state`를 사용한다.
- **`state`는 App이 기억해야하는 최소한의 변화하는 데이터 집합**이다.
- `state`를 구조화할 때 가장 중요한 원칙은 [직접 반복하지 않는 것](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself)이다. (중복을 최소화)
- **state가 아닌 조건**
  - 시간이 지나도 변하지 않는다 ?
  - 부모로부터 props를 통해 전달된다 ?
  - 컴포넌트의 기존 state 또는 props를 가지고 계산이 가능하다 ?
  - 그럼 state가 아니다!


#### Props 와 State

- Props
  - 함수가 전달받는 인자와 같다
  - 부모 컴포넌트가 자식 컴포넌트에 데이터를 넘기는 것
- State
  - 컴포넌트의 메모리와 같다
  - 컴포넌트가 일부 정보를 계속해서 추적하고 상호작용하여 변화할 수 있게한다


### Step 4: Identify where your state should live
> state가 어디에 있어야 할지 파악하기

- 타겟 state를 기반으로 렌더링하는 컴포넌트를 식별하고 가장 가까운 공통 상위 컴포넌트를 찾는다.
- 공통 부모에 state를 둘 수 있고, 혹은 공통 부모보다 더 상위 컴포넌트에 둘 수도 있으며 적절한 컴포넌트가 없다면 새로운 컴포넌트를 생성하여 공통 부모 컴포넌트보다 상위에 추가하여 state를 관리할 수 있다.
```jsx
function FilterableProductTable({ products }) {
  const [filterText, setFilterText] = useState('');
  const [inStockOnly, setInStockOnly] = useState(false);

  return (
    <div>
      <SearchBar
        filterText={filterText}
        inStockOnly={inStockOnly} />
      <ProductTable
        products={products}
        filterText={filterText}
        inStockOnly={inStockOnly} />
    </div>
  );
}
```


### Step 5: Add inverse data flow
> 역방향 데이터 흐름 추가하기

- 사용자 입력에 따라 state를 변경하려면 역방향으로도 데이터가 흐를 수 있게 해야한다.
- 상위에서 상태를 생성하고, 하위에 함수를 props로 전달한 다음  
  하위 컴포넌트에서 전달받은 함수를 호출하면서, 상위 컴포넌트에서 함수가 실행되게 한다.
    ```jsx
    function FilterableProductTable({ products }) {
      const [filterText, setFilterText] = useState('');
      const [inStockOnly, setInStockOnly] = useState(false);

      return (
        <div>
          <SearchBar
            filterText={filterText}
            inStockOnly={inStockOnly}
            onFilterTextChange={setFilterText}
            onInStockOnlyChange={setInStockOnly} />
            ...
        <div>
      )}
    ```
    ```jsx
    function SearchBar({
      filterText,
      inStockOnly,
      onFilterTextChange,
      onInStockOnlyChange
    }) {
      return (
        <form>
          <input
            type="text"
            value={filterText}
            placeholder="Search..."
            onChange={(e) => onFilterTextChange(e.target.value)} />
          <label>
            <input
              type="checkbox"
              checked={inStockOnly}
              onChange={(e) => onInStockOnlyChange(e.target.checked)} />
            {' '}
            Only show products in stock
          </label>
        </form>
      );
    }
    ```