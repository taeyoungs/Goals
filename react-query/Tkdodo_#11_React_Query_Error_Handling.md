# #11 React Query Error Handling

> 원본 글  
> https://tkdodo.eu/blog/react-query-error-handling

**Series**

- [#1 Practical React Query](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%231_Practical_React_Query.md)
- [#2 React Query Data Transformations](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%232_React_Query_Data_Transformations.md)
- [#3 React Query Render Optimizations](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%233_React_Query_Render_Optimizations.md)
- [#4 Status Checks in React Query](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%234_Status_Checks_in_React_Query.md)
- [#6 React Query and TypeScript](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%236_React_Query_and_TypeScript.md)
- [#7 Using WebSockets with React Query](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%237_Using_WebSockets_with_React_Query.md)
- [#8 Effective React Query Keys](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%238_Effective_React_Query_Keys.md)
- [#8a Leveraging the Query Function Context](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%238a_Leveraging_the_Query_Function_Context.md)
- [#9 Placeholder and Initial Data in React Query](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%239_Placeholder_and_Initial_Data_in_React_Query.md.md)
- [#10 React Query as a State Manager](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%2310_React_Query_as_a_State_Manager.md)
- #11 React Query Error Handling (현재)
- [#12 Mastering Mutations in React Query](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%2312_Mastering_Mutations_in_React_Query.md)
- [#13 Offline React Query](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%2313_Offline_React_Query.md)
- [#14 React Query and Forms](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%2314_React_Query_and_Forms.md)
- [#15 React Query FAQs](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%2315_React_Query_FAQs.md)
- [#16 React Query meets React Router](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%2316_React_Query_meets_React_Router.md)

**목차**

- [#11 React Query Error Handling](#11-react-query-error-handling)
    - [개요](#개요)
- [Prerequisites (전제 조건)](#prerequisites-전제-조건)
  - [The standard example](#the-standard-example)
  - [Error Boundaries](#error-boundaries)
  - [Showing error notifications](#showing-error-notifications)
  - [The onError callback](#the-onerror-callback)
  - [The global callbacks](#the-global-callbacks)
- [Putting it all together](#putting-it-all-together)

### 개요

에러 핸들링은 비동기 데이터, 특히나 데이터 패칭을 작업하는 데 있어서 필수적인 부분이다. 우리는 모든 요청이 성공할 순 없다는 사실과 모든 `Promise`가 `fulfilled` 상태로 바뀌지 않을 수 있다는 사실을 인지하고 있어야 한다.

Oftentimes, it is something that we don't focus on right from the beginning though. 우리는 에러 핸들링을 나중에 고려하는 `sunshine cases`를 먼저 처리하는 것을 좋아한다.

> 낙관적인 케이스들을 말하는 듯 ..? 성공할 때만을 가정한 케이스들을 말하는 건가 ..

그러나 에러 핸들링에 대해 생각하지 않으면 사용자 경험에 부정적인 영향을 끼칠 수 있다. 이를 피하기 위해 에러 핸들링을 하기 위해 **React Query**가 제공하는 옵션들에 대해서 알아보자.

# Prerequisites (전제 조건)

**React Query**는 에러를 올바르게 핸들링하기 위해서 `reject`된 `Promise`가 필요하다. 운좋게도 이는 **axios**와 같은 라이브러리를 사용하고 있을 때 얻을 수 있는 것이다.

만약 **fetch API**를 사용하거나 `4xx` 또는 `5xx`과 같은 에러 상태 코드에 대해서 `reject`된 `Promise`를 제공하지 않는 다른 라이브러리를 사용한다면, `queryFn`을 직접 변환해야 한다. 이와 관련된 사항은 다음 [링크](https://tanstack.com/query/v4/docs/guides/query-functions?from=reactQueryV3&original=https://react-query-v3.tanstack.com/guides/query-functions#usage-with-fetch-and-other-clients-that-do-not-throw-by-default)를 참조하자.

## The standard example

에러를 표시하는 가장 흔한 예시부터 살펴보자.

```jsx
function TodoList() {
  const todos = useQuery(['todos'], fetchTodos);

  if (todos.isLoading) {
    return 'Loading ...';
  }

  // ✅ standard error handling
  // could also check for: todos.status === 'error'
  if (todos.isError) {
    return 'An error occurred';
  }

  return (
    <div>
      {todos.data.map((todo) => (
        <Todo key={todo.id} {...todo} />
      ))}
    </div>
  );
}
```

위 예제에선 **React Query**가 우리에게 주는 `isError` **boolean** 플래그(`status` **enum**으로부터 나오는)를 체크하여 에러 상황을 처리하고 있다.

이는 몇 가지 시나리오에선 괜찮지만 몇 가지 단점 또한 존재한다.

1. **백그라운드에서 발생하는 에러를 잘 다루지 못한다.**

   우리가 과연 **Background refetch**가 실패했을 때 전체 `Todo List`를 `unmount`하고 싶을까? **API**에 일시적으로 문제가 생겼거나 속도 제한에 걸렸을 경우 이는 몇 분 내로 다시 동작할 수도 있다. 이러한 상황을 개선하고 싶다면 다음 [링크](https://tkdodo.eu/blog/status-checks-in-react-query)를 참고하자.

2. **쿼리를 사용하고자 하는 모든 컴포넌트에 위 같은 작업을 수행해야 하는 경우 매우 번거로울 수 있다.**

두 번째 단점을 해결하기 위해서 우리는 **React**가 직접적으로 제공하는 아주 좋은 기능을 사용할 수 있다.

## Error Boundaries

[Error Boundaries - React](https://reactjs.org/docs/error-boundaries.html#introducing-error-boundaries)

**Error Boundaries**는 렌더링 중 발생하는 런타임 에러를 잡기 위한 React의 일반적인 컨셉으로 이를 통해 런타임 에러에 적절하게 반응하고 `fallback` **UI**를 대신 보여줄 수도 있다.

**Error Boundaries**는 우리가 원하는 만큼 세분하여 컴포넌트들을 **Error Boundaries**로 래핑하고 이를 통해 나머지 UI는 에러의 영향을 받지 않을 수 있게 할 수 있기 때문에 좋다.

**Error Boundaries**가 할 수 없는 한 가지는 비동기 에러를 잡는 것이다. 왜냐하면 비동기 오류는 렌더링 중에 발생하지 않기 때문이다. 따라서, **React Query**에서 **Error Boundaries**가 동작하도록 만들기 위해선 라이브러리가 내부적으로 에러를 잡고 **Error Boundaries** 에러를 인지할 수 있도록 다음 렌더링 주기 때 해당 에러를 다시 `throw` 해줘야 한다.

필자는 이 방법이 에러 핸들링에 대한 매우 똑똑하면서 간단한 접근 방식이라고 생각한다. 이 작업을 수행하게 만들기 위해선 `useErrorBoundary` 플래그를 쿼리에 전달하기만 하면 된다(또는 **default config**를 통해 제공하거나).

```jsx
function TodoList() {
  // ✅ will propagate all fetching errors to the nearest Error Boundary
  const todos = useQuery(['todos'], fetchTodos, { useErrorBoundary: true });

  if (todos.data) {
    return (
      <div>
        {todos.data.map((todo) => (
          <Todo key={todo.id} {...todo} />
        ))}
      </div>
    );
  }

  return 'Loading ...';
}
```

**v3.23.0**부터 `useErrorBoundary`에 함수를 제공하여 어떤 오류가 **Error Boundary**로 가야하고 어떤 오류는 로컬에서 처리하고 싶은지 커스터마이징할 수도 있다.

```jsx
useQuery(['todos'], fetchTodos, {
  // 🚀 only server errors will go to the Error Boundary
  useErrorBoundary: (error) => error.response?.status >= 500,
});
```

이는 또한 `mutation`에서도 동작하며 `form`을 제출할 때 상당히 도움이 될 것이다. `4xx` 범위에 있는 에러들은 로컬에서 처리하는(예를 들어, 백엔드에서 유효성 검사 실패 응답이 오는 경우) 반면에 `5xx` 범위에 있는 서버 에러는 **Error Boundary**로 흘러가게 만들 수 있다.

## Showing error notifications

---

몇 가지 **use-case**들의 경우, 화면에 알림 배너를 렌더링 하는 대신에 어디엔가 표시되는 에러 토스트창을 보여주는 것(그리고 자동으로 사라지는)이 나을 수도 있다. 이런 토스트창은 `react-hot-toast`에서 제공하는 것과 같은 명령형 **API**로 주로 열린다.

```jsx
import toast from 'react-hot-toast';

toast.error('Something went wrong');
```

그렇다면 **React Query**에서 오류가 발생할 때 어떻게 해야 할까?

## The onError callback

---

```jsx
const useTodos = () =>
  useQuery(['todos'], fetchTodos, {
    // ✋ looks good, but is maybe _not_ what you want
    onError: (error) => toast.error(`Something went: wrong: ${error.message}`),
  });
```

언뜻 보기에 `onError` 콜백은 데이터 패칭이 실패했을 때 우리가 원하는 `side effect` 같고 이를 수행할 수 있을 것 처럼 보인다(커스텀 훅을 한번만 사용하는 한).

`useQuery`의 `onError` 콜백은 모든 `Observer`에 대해 호출된다. 즉, 만약 애플리케이션 내에서 `useTodos`가 두 번 호출되는 경우, 네트워크 요청이 한번 실패했을 지라도 에러 토스트창은 두 번 발생할 것이다.

이론적으로 `onError` 콜백 함수는 `useEffect`와 비슷하다고 생각할 수 있다. 그래서 만약 위 예제를 해당 구문으로 확장할 경우 모든 고객에 대해 에러 토스트가 실행될 것이라는 게 명확해진다.

```jsx
const useTodos = () => {
  const todos = useQuery(['todos'], fetchTodos);

  // 🚨 effects are executed for every component
  // that uses this custom hook individually
  React.useEffect(() => {
    if (todos.error) {
      toast.error(`Something went wrong: ${todos.error.message}`);
    }
  }, [todos.error]);

  return todos;
};
```

물론 콜백을 커스텀 훅에 추가하지 않고 훅을 호출하는 것은 문제가 없다. 하지만 만약 모든 `Observer`에게 데이터 패칭이 실패한 사실에 대해 알리고 싶지 않고 숨겨진 데이터 패칭(**Background refetch**를 말하는 듯 ?)에 대한 실패를 사용자에게 한번만 알리고 싶다면? 이런 경우를 위해 **React Query**는 다른 레벨의 콜백을 가지고 있다.

## The global callbacks

**Global callbacks**는 `QueryCache`를 생성할 때 제공되어야 하며, 이는 `new QueryClient`를 생성할 때 암묵적으로 만들어지지만 우리는 이를 커스터마이징할 수도 있다.

```jsx
const queryClient = new QueryClient({
  queryCache: new QueryCache({
    onError: (error) => toast.error(`Something went wrong: ${error.message}`),
  }),
});
```

이제 각 쿼리에 대해 에러 토스트창은 한번만 보여질 것이며 이는 우리가 정확히 원하는 것이었다. 또한 요청당 한 번만 실행되며 `defaultOptions`와 같이 덮어쓸 수 없기 때문에 우리가 수행하려는 모든 종류의 에러 트래킹 또는 모니터링을 배치하기에 가장 좋은 곳일 수도 있다.

# Putting it all together

**React Query**에서 에러를 핸들링하는 주된 방법 3가지는 다음과 같다.

- `useQuery`로부터 반환받은 `error` 속성을 활용하는 방법
- `onError` 콜백을 활용하는 방법 (**Query** 자체 또는 전역 `QueryCache` / `MutationCache` 에 있는 `onError` 콜백)
- **Error Boundaries**를 사용하는 방법

위 방법들을 원하는 대로 조합할 수 있으며, 개인적으로 필자가 좋아하는 것은 **Background refetch**에 대한 에러 토스트창을 보여주고(`stale`한 **UI**를 그대로 유지하기 위해) 로컬 또는 **Error Boundaries**를 사용하여 나머지 모든 것을 처리하는 것이다.

```jsx
const queryClient = new QueryClient({
  queryCache: new QueryCache({
    onError: (error, query) => {
      // 🎉 only show error toasts if we already have data in the cache
      // which indicates a failed background update
      if (query.state.data !== undefined) {
        toast.error(`Something went wrong: ${error.message}`);
      }
    },
  }),
});
```
