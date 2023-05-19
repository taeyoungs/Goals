# Memoization and React

> 원본 글: https://epicreact.dev/memoization-and-react/

**목차**

- [Memoization and React](#memoization-and-react)
  - [개요](#개요)
  - [React's memoization](#reacts-memoization)
  - [The value of memoization in React](#the-value-of-memoization-in-react)

## 개요

**Memoization**는 캐싱과 관련이 있다. 다음은 **Memoization**을 아주 간단하게 구현한 예시다.

```javascript
const cache = {};

function addTwo(input) {
  if (!cache.hasOwnProperty(input)) {
    cache[input] = input + 2;
  }
  return cache[input];
}
```

**Memoization**의 기본 아이디어는 입력과 관련 출력을 매달아 두었다가 동일한 입력으로 호출되면 입력에 해당하는 출력을 다시 반환하는 것이다.

반환되는 결과가 캐시된 값을 다시 계산하지 않도록 하는 것이 핵심이다. 위 예제의 경우 input에 2를 더하는 계산을 피할 수 있다.

```javascript
addTwo(3); // 5
addTwo(3); // 5, 동일하게 5를 반환하지만 이번에는 캐시로부터 반환된 값을 얻었다. 🤓
// 즉, 다시 계산하지 않았다는 것이다.
// 🤓 <= 요 이모지는 무엇인가 Memoization 됐을 경우 다시 나타날 예정이다.
```

이러한 계산을 다시 할 가치가 전혀 없어서 **Memoization** 하는 경우도 있겠지만, 계산에 비용이 많이 들어서 **Memoization**을 하는 경우도 있을 수 있다.

**Memoization**의 또 다른 흥미로운 측면은 캐시된 값이 이전에 얻은 값과 동일하다는 사실이다.

```javascript
// 내용이 일치하는 배열을 반환하는 함수가 있다고 가정해보자.
// "post" objects:

// getPostsNoMemo가 Memoization 되지 않은 경우
const posts1 = getPostsNoMemo('search term');
const posts2 = getPostsNoMemo('search term');
posts1 === posts2; // false (unique arrays)

// getPostsNoMemo가 Memoization 된 경우
const posts1 = getPostsMemo('search term');
const posts2 = getPostsMemo('search term');
posts1 === posts2; // true (identical array) 🤓
```

This has interesting implications for React we'll talk about in a second...

거기서부터 캐시 무효화에 대해 이야기해야 한다. **Memoization** 하려는 함수가 순수 함수라면 캐시를 영원히 보관할 수 있지만, 캐시가 얼마나 커지느냐에 따라 '메모리 부족' 문제가 발생할 수 있다. 캐시 무효화는 까다롭기 때문에 현재 포스팅에서는 다루지 않는다.

## React's memoization

React는 **Memoization**을 위한 3가지 API(`memo`, `useMemo` 그리고 `useCallback`)를 가지고 있다. **React가 채택한 캐싱 전략의 크기는 1이다.** 즉, 입력과 결과의 가장 최근 값만 유지한다. 이러한 결정에는 여러 가지 이유가 있지만, React Context에서 **Memoization**의 주요 사용 사례를 충족한다.

따라서, React의 **Memoization**은 다음과 같다.

```javascript
let prevInput, prevResult;

function addTwo(input) {
  if (input !== prevInput) {
    prevResult = input + 2;
  }
  prevInput = input;
  return prevResult;
}
```

이를 통해

```javascript
addTwo(3); // 5가 계산된다.
addTwo(3); // 5가 캐시로부터 반환된다. 🤓
addTwo(2); // 4가 계산된다.
addTwo(3); // 5가 계산된다.
```

분명히 말하지만, React의 경우 prevInput을 !==로 비교하지 않는다. 각 prop과 각 dependency가 동일한지 개별적으로 확인한다. 각각을 확인해 보자:

```javascript
// React.memo's `prevInput` is props and `prevResult` is react elements (JSX)
const MemoComp = React.memo(Comp)

// Memoization 처리한 컴포넌트를 렌더링하게 되면:
<MemoComp prop1="a" prop2="b" /> // 새로운 elements를 렌더링

// 동일한 props와 함께 다시 렌더링하게 되면:
<MemoComp prop1="a" prop2="b" /> // 이전 elements를 렌더링 🤓

// 다른 props와 함께 다시 렌더링을 하게 되면:
<MemoComp prop1="a" prop2="c" /> // 새로운 elements를 렌더링

// 처음과 동일한 props와 함께 다시 렌더링을 하게 되면:
<MemoComp prop1="a" prop2="b" /> // 새로운 elements를 렌더링
```

```javascript
// React.useMemo's `prevInput` is the dependency array
// and `prevResult` is whatever your function returns
const posts = React.useMemo(() => getPosts(searchTerm), [searchTerm]);
// searchTerm = 'puppies'과 함께 초기 렌더링:
// - getPosts가 호출되고
// - posts은 새로운 포스팅 목록의 배열이 된다.

// searchTerm = 'puppies'와 함께 리렌더링:
// - getPosts은 *호출되지 않고*
// - posts은 이전 값과 동일한 값이 된다. 🤓

// searchTerm = 'cats'와 함께 리렌더링:
// - getPosts가 호출되고
// - posts은 새로운 포스팅 목록의 배열이 된다.

// (다시) searchTerm = 'puppies'와 함께 리렌더링:
// - getPosts가 호출되고
// - posts은 새로운 포스팅 목록의 배열이 된다.
```

```javascript
// React.useCallback's `prevInput` is the dependency array
// and `prevResult` is the function
const launch = React.useCallback(() => launchCandy({ type, distance }), [type, distance]);
// type = 'twix'과 distance = '15m'과 함께 초기 렌더링:
// - launch는 이번 렌더링에 useCallback에 전달된 콜백과 동일

// type = 'twix' and distance = '15m'과 함께 리렌더링 :
// - launch는 마지막 렌더링에 useCallback에 전달된 콜백과 동일 🤓

// type = 'twix' and distance '20m'과 함께 리렌더링:
// - launch는 이번 렌더링에 useCallback에 전달된 콜백과 동일

// type = 'twix' and distance = '15m'과 함께 리렌더링 :
// - launch는 이번 렌더링에 useCallback에 전달된 콜백과 동일
```

## The value of memoization in React

무언가를 **Memoization** 하고 싶은 이유는 보통 두 가지가 있다:

- 값비싼 컴포넌트를 다시 렌더링하거나 값비싼 함수를 호출하는 등 값비싼 계산을 피하여 성능을 개선하기 위해
- Value stability를 위해

첫 번째 요점은 이미 다뤘다고 생각하지만, **Value stability**의 이점에 대해 명확히 하고 싶다. React Context에서 이 **Value stability**은 side-effect 뿐만 아니라 다른 값의 메모리에도 매우 중요하다.

간단한 예시를 한번 살펴보자.

```typescript
function App() {
  const [body, setBody] = React.useState();
  const [status, setStatus] = React.useState('idle');
  const fetchConfig = {
    method: 'POST',
    body,
    headers: { 'content-type': 'application/json' },
  };

  const makeFetchRequest = () => (body ? fetch('/post', fetchConfig) : null);

  React.useEffect(() => {
    const promise = makeFetchRequest();
    // if no promise was returned, then we didn't make a request
    // so we'll exit early
    if (!promise) return;

    setStatus('pending');
    promise.then(
      () => setStatus('fulfilled'),
      () => setStatus('rejected')
    );
  }, [makeFetchRequest]);

  function handleSubmit(event) {
    event.preventDefault();
    // get form input values
    setBody(formInputValues);
  }

  return <form onSubmit={handleSubmit}>{/* form inputs and other neat stuff... */}</form>;
}
```

> 참고: 각자 생각하기에 위 코드가 form sumit 코드를 작성하는 방식이 아닐 수도 있고, 본인 또한 그렇게 하지 않을 것이다... 사실, 개인적으로는 리액트 쿼리를 사용하겠지만 예제의 목적을 위해 조금만 참아보도록 하자...

무슨 일이 일어날지 추측해 보자. "runaway side-effect loop"를 생각했다면 정답이다! 그 이유는 의존성 배열의 개별 요소가 변경될 때마다 React.useEffect가 주어진 이펙트 콜백 함수에 대한 호출을 트리거하기 때문이다. 위 코드에서 유일한 의존성 요소는 `makeFetchRequest`이고, `makeFetchRequest`는 컴포넌트 내에서 생성되므로 렌더링할 때마다 새로 만들어 진다.

그래서 **Memoization**의 **Value stability**가 React에서 중요한 역할을 하는 부분입니다. 그럼 `makeFetchRequest`를 `useCallback`으로 **Memoization** 해보자.

```typescript
const makeFetchRequest = React.useCallback(
  () => (body ? fetch('/post', fetchConfig) : null),
  [body, fetchConfig]
);
```

`useCallback`은 의존성이 변경될 때만 새 함수를 반환한다. 그 덕분에 `makeFetchRequest`는 렌더링 사이에 안정적인 값을 갖는다. 안타깝게도 `fetchConfig`도 컴포넌트 내에서 생성되므로 렌더링할 때마다 새로 만들어 진다. 따라서 **Value stability**을 위해 `fetchConfig`도 **Memoization** 해두자.

```typescript
const fetchConfig = React.useMemo(() => {
  return {
    method: 'POST',
    body,
    headers: { 'content-type': 'application/json' },
  };
}, [body]);
```

🎉 이제 `fetchConfig`와 `makeFetchRequest`는 모두 안정적이며 우리가 원하는 대로 본문이 변경될 때만 변경된다. 이 코드의 최종 버전은 다음과 같다:

```typescript
function App() {
  const [body, setBody] = React.useState();
  const [status, setStatus] = React.useState('idle');
  const fetchConfig = React.useMemo(() => {
    return {
      method: 'POST',
      body,
      headers: { 'content-type': 'application/json' },
    };
  }, [body]);

  const makeFetchRequest = React.useCallback(
    () => (body ? fetch('/post', fetchConfig) : null),
    [body, fetchConfig]
  );

  React.useEffect(() => {
    const promise = makeFetchRequest();
    // if no promise was returned, then we didn't make a request
    // so we'll exit early
    if (!promise) return;

    setStatus('pending');
    promise.then(
      () => setStatus('fulfilled'),
      () => setStatus('rejected')
    );
  }, [makeFetchRequest]);

  function handleSubmit(event) {
    event.preventDefault();
    // get form input values
    setBody(formInputValues);
  }

  return <form onSubmit={handleSubmit}>{/* form inputs and other neat stuff... */}</form>;
}
```

`makeFetchRequest`에 대한 `useCallback`이 제공하는 **Value stability**은 side-effect가 실행되는 시기를 제어할 수 있도록 도와준다. 그리고 `fetchConfig`에 대해 `useMemo`가 제공하는 **Value stability**은 `makeFetchRequest`의 **Memoization** 특성을 보존하여 작동할 수 있도록 도와준다.

이 작업을 자주 할 필요는 없지만, 필요할 때 방법을 알면 좋다. 앞서 말했듯이, 이런 종류의 작업에는 리액트 쿼리를 사용하겠지만, 원하지 않는다면 실제로는 이렇게 작성할 것이다(직접 추상화하지 않고):

```typescript
function App() {
  const [body, setBody] = React.useState();
  const [status, setStatus] = React.useState('idle');

  React.useEffect(() => {
    // no need to do anything if we don't have a body to send
    // so we'll exit early
    if (!body) return;

    setStatus('pending');
    const fetchConfig = {
      method: 'POST',
      body,
      headers: { 'content-type': 'application/json' },
    };
    fetch('/post', fetchConfig).then(
      () => setStatus('fulfilled'),
      () => setStatus('rejected')
    );
  }, [body]);

  function handleSubmit(event) {
    event.preventDefault();
    // get form input values
    setBody(formInputValues);
  }

  return <form onSubmit={handleSubmit}>{/* form inputs and other neat stuff... */}</form>;
}
```
