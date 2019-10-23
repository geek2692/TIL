### useEffect

이번에는 componentDidMount, componentDidUpdate, 그리고 componentWillUnmount와 같은 lifecycle method를 사용하는것과 같은 효과를 내는 Hook을 사용하여 특정작업을 처리하는 방법을 알아보겠습니다.

```javascript
useEffect(() => {
    document.title = `You clicked ${counts} times`; 
});
```

위와 같은 경우, 컴포넌트가 업데이트 될 때 마다 렌더링이 끝난후 useEffect가 호출될 것 입니다.

```javascript
//check out the variable count in the array at the end
useEffect(() => {
    document.title = `You clicked ${counts} times`; 
},[count]);
```

useEffect를 사용할때는 첫번째 인자로 함수, 두번째 인자로 의존값이 들어있는 배열(deps)를 넣습니다.
만약 배열을 비우게 되면 처음 컴포넌트가 나타날때만 호출됩니다. 
그리고 첫번째 인자만 사용할 경우 컴포넌트가 리렌더링 될 때마다 계속 호출되게 될 것입니다.

```javascript
const [height, setHeight] = useState(150);

useEffect(() => {
    document.title = `You height ${height}, you okay with that?`; 
}, [height]);

onClick = {() => setHeight(height + 10)}
```

onClick이 트리거 되면, useEffect method가 호출되고, DOM이 업데이트 되면서 title에 새로운 height이 보일것입니다.

```javascript
useEffect(() => {
   fetch('https://jsonplaceholder.typicode.com/todos/1')
    .then(results => results.json())
    .then((data) => { setTodos([{ text: data.title }]); });
}, []);
```

useEffect를 사용하기 좋은 경우는, 외부 API를 호출해야 될 경우입니다.



### useMemo

이번에는 성능 최적화를 위한 useMemo에 대해 알아보겠습니다. 
여기서 memo는 memoized를 의미하는데 이는 이전에 계산한 값을 재사용한다는 뜻을 갖고 있습니다.

```javascript
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
```

useMemo의 첫번째 인자에는 어떻게 연산할지에 대한 함수를 넣어주고, 두번째 인자로는 deps 배열을 넣어주는데
배열안의 내용이 바뀌면. 등록한 함수를 호출하여 값을 연산해주고, 바뀌지않으면 값을 재사용 하게됩니다.



### useCallback

useCallback은 앞에서 언급한 useMemo와 비슷한 Hook입니다.
다른점은 useMemo는 값을 재사용할때 사용하는 반면, useCallback은 특정 함수를 만들지 않고 재사용 하고 싶을때 사용합니다.

```javascript
const memoizedCallback = useCallback(
  () => {
    doSomething(a, b);
  },
  [a, b],
);
```

[두 개의 Hooks에 대해 설명을 마치며 useMemo, useCallback을 사용하는 경우에 대한 글입니다](https://rinae.dev/posts/review-when-to-usememo-and-usecallback)



### useReducer

앞서, 상태를 관리할때 useState를 사용하여 관리하였습니다.
이번에는 또 다른 방법인 useReducer에 대해 알아보겠습니다. 이 Hook함수를 이용하면 컴포넌트의 상태 업데이트 로직을 컴포넌트에서 분리하여 컴포넌트 밖에 작성하거나 다른 파일에서 불러와 사용 할 수도 있습니다.

reducer에 대해 간단히 설명하면 현재상태와 액션객체를 받아와 새로운 상태를 반환해주는 함수입니다.

```javascript
const [state, dispatch] = useReducer(reducer, initialState);
```

여기서 `state` 는 우리가 앞으로 컴포넌트에서 사용 할 수 있는 상태를 가르키게 되고, `dispatch` 는 액션을 발생시키는 함수라고 이해하시면 됩니다. 이 함수는 다음과 같이 사용합니다: `dispatch({ type: 'INCREMENT' })`.

그리고 `useReducer` 에 넣는 첫번째 파라미터는 reducer 함수이고, 두번째 파라미터는 초기 상태입니다.

```javascript
import React, { useReducer } from 'react';

function reducer(state, action) {
  switch (action.type) {
    case 'INCREMENT':
      return state + 1;
    case 'DECREMENT':
      return state - 1;
    default:      
      return state;
  }
}

function Counter() {
  const [number, dispatch] = useReducer(reducer, 0);

  const onIncrease = () => {
    dispatch({ type: 'INCREMENT' });
  };

  const onDecrease = () => {
    dispatch({ type: 'DECREMENT' });
  };

  return (
    <div>
      <h1>{number}</h1>
      <button onClick={onIncrease}>+1</button>
      <button onClick={onDecrease}>-1</button>
    </div>
  );
}

export default Counter;
```

끝으로 [useState와 useReducer를 비교한 글 입니다.](https://www.robinwieruch.de/react-usereducer-vs-usestate)

