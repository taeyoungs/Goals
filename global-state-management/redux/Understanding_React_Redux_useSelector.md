# Understanding React Redux useSelector

> 원본 글  
> https://react-redux.js.org/api/hooks#useselector

**목차**

- [Understanding React Redux useSelector](#understanding-react-redux-useselector)
  - [Equality Comparisons and Updates](#equality-comparisons-and-updates)
  - [useSelector Examples](#useselector-examples)
    - [Basic usage](#basic-usage)
    - [Using memoizing selectors](#using-memoizing-selectors)

```tsx
const result: any = useSelector(selector: Function, equalityFn?: Function)
```

`Selector` 함수를 이용하여 **Redux** **store state**로부터 데이터를 추출할 수 있다.

> Selector 함수는 잠재적으로 임의의 시점에 여러 번 실행될 수 있기 때문에 순수 함수여야 한다.

`Selector` 함수는 개념적으로 `connect` 함수의 `mapStateToProps` 인자값과 거의 동일하다. `Selector` 함수는 **Redux store state** 전체를 유일한 인자값으로 하여 호출될 것이다. `Selector` 함수는 함수 컴포넌트가 렌더링될 때마다 실행될 것이다. (만약 `state`의 참조값이 컴포넌트의 이전 렌더링 때와 변함이 없다면, `Selector` 함수의 재실행없이 `useSelector` 훅은 캐시된 결과값을 반환할 것이다) 또한 `useSelector` 훅의 호출은 **Redux store**를 구독하게 될 것이고 `action`이 `dispatch` 될 때마다 `Selector` 함수를 실행할 것이다.

그러나 `mapState` 함수와 `useSelector` 훅의 호출에 전달되는 `Selector` 사이에는 몇 가지 차이점이 존재한다.

- `Selector`는 객체 뿐만이 아니라 어떠한 값이라도 반환할 수 있다. `Selector`의 반환값은 `useSelector` 훅의 반환값으로 사용될 것이다.
- `action`이 `dispatch` 됐을 때, `useSelector` 훅은 이전 `Selector`의 반환값과 현재 반환값의 참조를 비교할 것이다. 만약 두 참조값이 다른 경우, 컴포넌트는 강제로 리렌더링 될 것이다. 만약 참조값이 동일하다면 컴포넌트는 리렌더링되지 않을 것이다.
- `Selector` 함수는 `ownProps`를 인자값으로 받지 않는다. 그러나, `props`는 `Closure` 또는 **curried selector**를 이용함으로써 사용될 수 있다.
- **메모제이션** 처리된 `Selector`를 사용할 때 더 각별한 주의가 필요하다.
- `useSelector`는 **Shallow Equality**가 아닌 정적 `===` 참조 동등 체크를 기본으로 한다

단일 함수 컴포넌트 내에서 `useSelector`는 여러 번 호출될 수 있다. `useSelctor`의 각 호출은 **Redux store**의 개별 구독을 만들어 낸다. 왜냐하면 **React Redux** v7에서 사용된 **React** 배치 업데이트 동작으로 인해, 동일한 컴포넌트 내의 여러 개의 `useSelector`가 새로운 값을 반환하도록 하는 `dispatch`된 `action`은 오직 한번만 리렌더링 되어야만 하기 때문이다.

## Equality Comparisons and Updates

함수 컴포넌트가 렌더링될 때, `Selector` 함수는 호출될 것이고 함수의 반환값은 `useSelector` 훅에서 반환될 것이다. (컴포넌트의 이전 렌더링과 함수 참조값이 동일할 경우 `Selector` 함수의 재실행없이 훅은 캐시된 결과값을 반환할 수도 있다)

그러나, **Redux store**로 `action`이 `dispatch` 됐을 때, `useSelector`는 만약 `Selector`의 결과값이 마지막 결과값과 다를 경우에만 리렌더링을 강제할 것이다. 결과값 비교는 기본적으로 정적 `===` 참조 비교로 이루어진다. 이것은 리렌더링이 필요한지 여부를 결정하기 위해 `mapState` 호출 결과에 대해 **Shallow Equality** 검사를 사용하는 `connect`와는 다르다. 이것은 `useSelector`를 어떻게 사용해야 하는지에 대한 몇 가지 의미를 보여준다.

`mapState`에선, 모든 개별 필드가 하나의 객체로 합쳐져서 반환된다. 이는 반환 객체가 새로운 참조값을 가지던 가지지 않던 신경쓰지 않는다. - `connect`는 그저 각 개별 필드를 비교한다. `useSelector`에선 매번 새 객체가 반환되어 기본적으로 항상 리렌더링이 강제될 것이다. 만약 **store**에서 여러 값을 검색하려는 경우 다음과 같이 할 수 있다.

- 단일 필드 값을 반환하는 `useSelector`를 여러 번 호출한다.
- **Reselect** 또는 이와 유사한 라이브러리를 사용하여 하나의 개체에 여러 값을 반환하지만 값 중 하나가 변경된 경우에만 새 개체를 반환하는 **메모제이션** `Selector`를 만든다.
- 다음과 같이 `useSelector`에 `equalityFn` 인자값을 넘겨 **React-Redux**의 `shallowEqual` 함수를 사용한다.

```tsx
import { shallowEqual, useSelector } from 'react-redux';

// later
const selectedData = useSelector(selectorReturningObject, shallowEqual);
```

**optional comparison function**는 **Lodash**의 `isEqual` 함수 또는 **Immutable.js**의 비교 능력과 같은 무언가로도 사용 가능하다.

> `Selector` 함수에 별다른 **메모이제이션** 처리가 없다면 매번 새로운 객체(참조값이 다른)를 반환해줄 것이므로 **Redux store**의 특정 상태를 구독하고 있는 경우 해당 상태가 변하지 않았음에도 새로운 객체를 반환하니 리렌더링이 발생한다.
>
> 따라서, 구독하고 있는 **Redux Store** 생태값이 변할 때만 컴포넌트에 리렌더링을 발생시키기 위해선 `Reselect` 혹은 이와 같은 **메모제이션** 처리가 된 `Selector` 함수를 만들어서 사용해야 한다는 뜻이다.

## useSelector Examples

### Basic usage

```tsx
import React from 'react';
import { useSelector } from 'react-redux';

export const CounterComponent = () => {
  const counter = useSelector((state) => state.counter);

  return <div>{counter}</div>;
};
```

**Closure**를 통한 `props`를 사용으로 추출할 값을 결정할 수 있다.

```tsx
import React from 'react';
import { useSelector } from 'react-redux';

export const TodoListItem = (props) => {
  const todo = useSelector((state) => state.todos[props.id]);

  return <div>{todo.text}</div>;
};
```

### Using memoizing selectors

위와 같이 `Selector` 함수를 인라인으로 사용할 경우, 컴포넌트가 렌더링될 때마다 `Selector`의 새로운 인스턴스가 생성될 것이다. 이러한 작업이 계속되는 한 `Selector`는 어떠한 상태값도 유지하지 못한다. 그러나, **memoizing selectors**(예를 들어, **Reselect**의 `createSelector`를 통해 생성된)는 내부 상태값을 가지고 있기에 사용할 때 더 조심해야 한다. 아래를 보면 **memoizing selectors**의 일반적인 사용법을 확인할 수 있다.

`Selector`가 상태값(Redux Store)에만 의존하는 경우, 각 렌더링에 동일한 `Selector` 인스턴스가 사용되도록 컴포넌트 밖에서 `Selector`를 선언했는지 확인하자.

```tsx
import React from 'react';
import { useSelector } from 'react-redux';
import { createSelector } from 'reselect';

const selectNumCompletedTodos = createSelector(
  (state) => state.todos,
  (todos) => todos.filter((todo) => todo.completed).length
);

export const CompletedTodosCounter = () => {
  const numCompletedTodos = useSelector(selectNumCompletedTodos);

  return <div>{numCompletedTodos}</div>;
};

export const App = () => {
  return (
    <>
      <span>Number of completed todos:</span>
      <CompletedTodosCounter />
    </>
  );
};
```

컴포넌트의 `props`에 의존하는 `Selector`의 경우에도 동일하지만 오직 단일 컴포넌트의 단일 인스턴스에서만 사용된다.

```tsx
import React from 'react';
import { useSelector } from 'react-redux';
import { createSelector } from 'reselect';

const selectCompletedTodosCount = createSelector(
  (state) => state.todos,
  (_, completed) => completed,
  (todos, completed) => todos.filter((todo) => todo.completed === completed).length
);

export const CompletedTodosCount = ({ completed }) => {
  const matchingCount = useSelector((state) => selectCompletedTodosCount(state, completed));

  return <div>{matchingCount}</div>;
};

export const App = () => {
  return (
    <>
      <span>Number of done todos:</span>
      <CompletedTodosCount completed={true} />
    </>
  );
};
```

그러나 `Selector`가 여러 컴포넌트 인스턴스에서 사용되고 여러 컴포넌트들의 `props`에 의존하고 있는 경우, 각 컴포넌트 인스턴스가 고유의 `Selector` 인스턴스를 갖고 있는지 확실히 할 필요가 있다.

```tsx
import React, { useMemo } from 'react';
import { useSelector } from 'react-redux';
import { createSelector } from 'reselect';

const makeSelectCompletedTodosCount = () =>
  createSelector(
    (state) => state.todos,
    (_, completed) => completed,
    (todos, completed) => todos.filter((todo) => todo.completed === completed).length
  );

export const CompletedTodosCount = ({ completed }) => {
  const selectCompletedTodosCount = useMemo(makeSelectCompletedTodosCount, []);

  const matchingCount = useSelector((state) => selectCompletedTodosCount(state, completed));

  return <div>{matchingCount}</div>;
};

export const App = () => {
  return (
    <>
      <span>Number of done todos:</span>
      <CompletedTodosCount completed={true} />
      <span>Number of unfinished todos:</span>
      <CompletedTodosCount completed={false} />
    </>
  );
};
```
