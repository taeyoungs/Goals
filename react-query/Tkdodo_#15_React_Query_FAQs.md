# #15 React Query FAQs

> 원본 글  
> https://tkdodo.eu/blog/react-query-fa-qs

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
- [#11 React Query Error Handling](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%2311_React_Query_Error_Handling.md)
- [#12 Mastering Mutations in React Query](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%2312_Mastering_Mutations_in_React_Query.md)
- [#13 Offline React Query](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%2313_Offline_React_Query.md)
- [#14 React Query and Forms](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%2314_React_Query_and_Forms.md)
- #15 React Query FAQs (현재)
- [#16 React Query meets React Router](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%2316_React_Query_meets_React_Router.md)
- [#17 Seeding the Query Cache](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%2317_Seeding_the_Query_Cache.md)

**목차**

- [#15 React Query FAQs](#15-react-query-faqs)
  - [개요](#개요)
  - [How can I pass parameters to refetch?](#how-can-i-pass-parameters-to-refetch)
    - [Loading states](#loading-states)
  - [Why are updates not shown?](#why-are-updates-not-shown)
    - [1: Query Keys are not matching](#1-query-keys-are-not-matching)
    - [2: The QueryClient is not stable](#2-the-queryclient-is-not-stable)
  - [Why should I useQueryClient()…](#why-should-i-usequeryclient)
    - [1: useQuery uses the hook too](#1-usequery-uses-the-hook-too)
    - [2: It decouples your app from the client](#2-it-decouples-your-app-from-the-client)
    - [3: You sometimes can’t export](#3-you-sometimes-cant-export)
  - [Why do I not get errors ?](#why-do-i-not-get-errors-)
    - [The fetch API](#the-fetch-api)
    - [Logging](#logging)

## 개요

필자는 지난 18개월 동안 **React Query**와 관련하여 많은 질문에 답변해 왔다. 커뮤니티에 참여하고 질문에 답하는 것이 처음에 오픈 소스를 만들게 된 계기가 되었고, 이 **React Query** 관련 시리즈 포스팅을 작성하는 데에도 큰 요인이 되었다.

I'm still excited to answer questions, especially if they are well formulated and of the non-standard kind. Please see my post [How can I?](https://tkdodo.eu/blog/how-can-i) if you don't know what I mean or want to know what makes a question a good question.

그러나 대답하기 간단하면서도 글로 쓰기에는 약간의 노력이 필요한 여러 반복되는 질문들을 봐았다. 이러한 반복되는 질문들이 현재 포스팅에서 다루고자 하는 것들이다. 즉, 필자가 그러한 질문들이 다시 보게 됐을 때 사람들에게 줄 수 있는 리소스를 위해서이다.

다음은 주요 질문들이다.

## How can I pass parameters to refetch?

간단한 답변은 "**그렇 수 없다**"이다. 여기엔 중요한 이유가 있다. 매번 `refetch`에 파라미터를 전달하고 싶을 때마다 실제로는 굳이 전달하지 않아도 되는 상황일 것이다.

```jsx
const { data, refetch } = useQuery(['item'], () => fetchItem({ id: 1 }));

<button
  onClick={() => {
    // 🚨 this is not how it works
    refetch({ id: 2 });
  }}
>
  Show Item 2
</button>;
```

파라미터 또는 변수들은 `query`에 종속되어 있다. 위 코드를 살펴보면, 우리는 **Query Key**를 `['item']`이라고 정의하고 있다. 따라서 우리가 패칭해 온 데이터가 무엇이든지 해당 키 아래에 저장된다. 만약 다른 `id`로 `refetch`를 했다고 해도 캐시의 동일한 공간에 저장될 것이다. 왜냐하면 키가 이전과 동일하기 때문이다. 그래서 `id`를 2로 해도 `id`가 1인 캐시 데이터 공간에 덮어써질 것이며 만약 `id`를 1로 다시 전환할 경우 해당 데이터는 사라진다.

다른 **Query Key** 아래에 다른 응답 데이터를 캐싱하는 것은 **React Query**의 가장 큰 장점 중 하나이다. "**refetch-with-parameters**" API는 이러한 장점을 없앤다. 이것이 `refetch`가 동일한 변수와 함께 요청을 다시 발생시키기 위한 기능인 이유다. 따라서, 본질적으로는 `refetch`를 원할 것이 아니라 다른 `id`로 새로운 데이터 패칭을 하길 원해야 한다.

**React Query**를 효과적으로 사용하기 위해선 선언적(declarative) 접근 방식을 받아들여야 한다. **Query Key**는 데이터를 패칭하기 위한 `query` 함수의 모든 의존성을 정의한다. 이 점을 잘 고수한다면 `refetch` 하기 위해서 우리가 해줄 것은 의존성을 업데이트 해주는 것 뿐이다. 보다 현실적인 예는 다음과 같다.

```jsx
const [id, setId] = useState(1);

const { data } = useQuery(['item', id], () => fetchItem({ id }));

<button
  onClick={() => {
    // ✅ set id without explicitly refetching
    setId(2);
  }}
>
  Show Item 2
</button>;
```

`setId`는 컴포넌트를 리렌더링 할 것이고 **React Query**는 새로운 **Query Key**를 가지고 해당 키에 대한 데이터 패칭을 시작할 것이다. 이는 `id`가 1인 데이터가 캐시된 공간과 별도의 공간에 캐시된다.

또한 선언적 접근 방식은 우리가 어디에서 어떻게 `id`를 업데이트 하던 항상 `id`와 `query` 데이터를 동기화한다. 따라서, 우리는 생각하는 방식을 "**버튼을 누르면 `refetch`를 하고 싶다**"에서 "**항상 현재 id에 대한 데이터를 보고 싶다**"와 같이 바꿔야 한다.

`useState`를 이용하여 `id`를 저장해서도 안된다. (`zustand`, `redux`와 같은 클라이언트 사이드 상태 저장 포함) 위 예제의 경우 **URL**이 `id`를 저장하기 좋은 곳이다.

```jsx
const { id } = useParam();

const { data } = useQuery(['item', id], () => fetchItem({ id }));

// ✅ change url, make useParams pick it up
<Link to="/2">Show Item 2</Link>;
```

이러한 접근 방식의 가장 좋은 점은 상태(ex. 위 예제의 `id`)를 관리할 필요가 없다는 점, 공유가능한 **URL**을 가져오면 된다는 점 그리고 브라우저의 뒤로 가기 버튼이 항목 사이를 이동하는 것과 같이 동작한다는 점이다.

### Loading states

이미 눈치챘을 수도 있겠지만 **Query Key**를 전환한다는 것은 `query`가 다시 **hard** 로딩 상태가 된다는 것을 의미한다. 이는 우리가 변경한 Query Key에 대한 캐시 데이터가 없기 때문이다. (해당 **Query Key**로 데이터 패칭을 한 적이 없는 경우를 말한다)

**Query Key**에 대한 `placeholderData`를 설정하거나 미리 새로운 **Query Key**에 대한 데이터를 `prefetching` 하는 것과 같이 **Query Key** 전환을 용이하게 하기 위한 여러 가지 방법들이 있다. 이 문제를 해결하기 위한 좋은 접근 방법은 `query`에게 이전 데이터를 유지하도록 지시하는 것이다.

```jsx
const { data, isPreviousData } = useQuery(
  ['item', id],
  () => fetchItem({ id }),
  // ⬇️ like this️
  { keepPreviousData: true }
);
```

이 플래그를 설정해두면 **React Query**는 `id: 2`에 대한 데이터를 패칭하는 동안 `id: 1`에 대한 데이터를 보여줄 것이다. 추가적으로 `query` 결과에 들어 있는 `isPreviousData` 플래그는 `true`로 설정될 것이기에 이를 가지고 **UI**에 작업도 가능하다. 데이터와 함께 백그라운드 로딩 스피너를 표시하거나 표시된 데이터에 불투명도를 추가하여 데이터가 `stale` 상태라는 것을 나타낼 수 있습니다. That is totally up to you - React Query just gives you the means to do that. 🙌

## Why are updates not shown?

필자는 가끔 `mutation`의 응답 데이터로 업데이트를 수행하고 싶거나 `mutation` 후에 `invalidate`를 하고 싶은 경우와 같이 직접적으로 **Query Cache**와 상호 작용할 때 "업데이트가 화면에 반경되지 않는다"거나 단순히 "동작하지 않는다"는 제보를 받는다. 이런 경우 대개 주로 두 가지 이유로 귀결된다.

### 1: Query Keys are not matching

**Query Key**들은 결국엔 해시 처리되기 때문에 **referential stability**를 유지하거나 객체의 키 순서를 고려할 필요가 없다. 그러나 `queryClient.setQueryData`를 호출할 때는 **Query Key**가 반드시 기존에 존재하는 **Query Key**와 일치해야만 한다. 다음 예와 같은 두 가지 **Query Key**는 일치하지 않는다.

```jsx
['item', '1'][('item', 1)];
```

**Query Key** 배열의 두 번째 값은 첫 번째 예제에선 `string`이고 두 번째 예제에선 `number`다. 이런 경우는 **Query Key**를 주로 `number` 타입으로 사용하고 있는데 `useParams`로 읽어온 **URL**이 `string` 타입일 때 발생할 수 있다.

이런 경우 제일 유용한 친구는 **React Query DevTools**다. 이를 통해 어떤 **Query Key**가 존재하고 어떤 **Query Key**가 데이터 패칭 중인지 알 수 있다. 그래도 성가신 세부 사항들은 주의해야 한다.

I recommend using [TypeScript](https://www.typescriptlang.org/) and [Query Key Factories](https://tkdodo.eu/blog/effective-react-query-keys) to help with that problem.

### 2: The QueryClient is not stable

대부분의 예제에서 `queryClient`는 `App` 컴포넌트 밖에서 생성한다. 이는 `queryClient`를 참조적으로 안정적이게 만들기 위해서다.

```jsx
// ✅ created outside of the App
const queryClient = new QueryClient();

export default function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <Example />
    </QueryClientProvider>
  );
}
```

`QueryClient`는 **Query Cache**를 관리하고 있기 때문에 새로운 `queryClient`를 만드는 경우 비어 있는 새로운 캐시를 갖게 된다. 만약 `App` 컴포넌트 내부로 `queryClient`의 생성을 옮길 경우 경로 변경과 같은 어떠한 이유로 인해 컴포넌트가 리렌더링이 발생한다면 캐시 데이터는 전부 날아가 버릴 것이다.

```jsx
export default function App() {
  // 🚨 this is not good
  const queryClient = new QueryClient();

  return (
    <QueryClientProvider client={queryClient}>
      <Example />
    </QueryClientProvider>
  );
}
```

만약 `App` 컴포넌트 내부에서 `queryClient`를 생성하고 싶다면 `ref` 인스턴스나 **React** `state`를 사용하여 참조적으로 안정적이게 만들어 줘야 한다.

```jsx
export default function App() {
  // ✅ this is stable
  const [queryClient] = React.useState(() => new QueryClient());

  return (
    <QueryClientProvider client={queryClient}>
      <Example />
    </QueryClientProvider>
  );
}
```

I do have a separate blog post on that topic: [useState for one-time initializations](https://tkdodo.eu/blog/use-state-for-one-time-initializations).

## Why should I useQueryClient()…

`queryClient`를 `import` 해올 수 있다면 ..?

`QueryClientProvider`는 생성된 `queryClient`를 **React** `Context`에 집어넣어 애플리케이션 전체에 배포한다. 따라서, queryClient를 useQueryClient를 통해 읽을 수 있다. 이는 추가적인 subscription을 생성하지 않고 추가적인 리렌더링을 발생시키지 않는다. (위 예제와 같이 참조적으로 안정적이라면) 그저 **prop-drilling**을 피해 `queryClient`를 아래로 전달한다.

또는 `queryClient`를 `export`하고 원하는 곳에서 `import` 할 수도 있다.

```jsx
// ⬇️ exported so that we can import it
export const queryClient = new QueryClient();

export default function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <Example />
    </QueryClientProvider>
  );
}
```

훅(`useQueryClient`)을 사용하는 방식이 더 선호되는 이유는 다음과 같다.

### 1: useQuery uses the hook too

`useQuery`를 호출할 때, 내부적으로 `useQueryClient`를 호출한다. 이를 통해 **React** `Context`에 있는 가장 가까운 queryClient를 찾을 것이다. 큰 문제는 아니겠지만, `import`해서 가져온 `queryClient`가 내부적으로 `useQueryClient`를 사용하여 가져오는 `queryClient`와 다른 경우 피할 수 있는 버그를 추적하기 어려워 질 것이다.

### 2: It decouples your app from the client

The client you define in your `App` is your production client. It might have some default settings that work well in production. However, in testing, it might make sense to use different default values. One example is [turning off retries](https://tkdodo.eu/blog/testing-react-query#turn-off-retries) during testing, because testing erroneous queries might time out the test otherwise.

의존성 주입 메커니즘이 사용될 때 **React** `Context`의 가장 큰 이점은 의존성에서 애플리케이션을 분리한다는 것이다. `useQueryClient`는 어느 특정 `queryClient`가 아닌 트리 상에 있는 `queryClient`를 가져온다. 프로덕션 `queryClient`를 직접적으로 `import` 하면 이러한 이점을 잃어버리게 될 것이다.

### 3: You sometimes can’t export

때로는 `App` 컴포넌트 내부에서 `queryClient`를 생성할 필요가 있다(위 예제에서 보이는 바와 같이). 이러한 예로는 **SSR**, 즉 서버 사이드 렌더링을 사용할 때이다. 왜냐하면 다수의 사용자가 동일한 `queryClient`를 공유하는 것을 피하고 싶기 때문이다.

이는 **microfrontend**로 작업할 때도 동일하다. `App` 컴포넌트들은 고립되어 있어야만 한다. 만약 `App` 컴포넌트 바깥에 `queryClient`를 생성한다면 하나의 페이지에서 동일한 애플리케이션을 두 번 사용할 경우 `queryClient`를 공유하게 된다.

마지막으로 `queryClient`의 기본 값에 다른 훅을 사용하고 싶은 경우 `App` 컴포넌트 내부에서 `queryClient`를 생성해야 한다. 실패한 모든 `mutation`에 토스트 메시지창을 보여주고 싶다면 전역 에러 핸들러를 고려해보자.

```jsx
export default function App() {
  // ✅ we couldn't useToast outside of the App
  const toast = useToast();
  const [queryClient] = React.useState(
    () =>
      new QueryClient({
        mutationCache: new MutationCache({
          // ⬇️ but we need it here
          onError: (error) => toast.show({ type: 'error', error }),
        }),
      })
  );

  return (
    <QueryClientProvider client={queryClient}>
      <Example />
    </QueryClientProvider>
  );
}
```

따라서, 이와 같이 `queryClient`를 생성하는 경우 `queryClient`를 `export` 하고 애플리케이션 내에서 `import` 할 수 있는 방법이 없다.

필자가 추측하기에 `queryClient`를 `export` 하고자 하는 이유는 **Query**를 `invalidation` 해야하는 레거시 클래스 컴포넌트와 함께 작업 중인 상황이다. 이런 경우 훅 자체를 사용할 수 없다. 만약 이런 때 함수 컴포넌트로 간단하게 리펙토링할 수 없는 경우라면 **render props** 버전을 만드는 것에 대해 고려해보자.

```jsx
const UseQueryClient = ({ children }) => children(useQueryClient());

<UseQueryClient>
  {(queryClient) => <button onClick={() => queryClient.invalidateQueries(['items'])}>invalidate items</button>}
</UseQueryClient>;
```

useQuery나 다른 훅에 대해서도 동일한 작업을 할 수 있다.

```jsx
const UseQuery = ({ children, ...props }) => children(useQuery(props));

<UseQuery queryKey={['items']} queryFn={fetchItems}>
  {({ data, isLoading, isError }) => (
    // 🙌 return jsx here
  )}
</UseQuery>
```

## Why do I not get errors ?

만약 네트워크 요청이 실패한 경우 이상적이라면 `query`는 `error` 상태로 전환될 것이다. 만약 `error` 상태로 전환되지 않고 대신에 여전히 성공적인 `query`가 보여진다면 이는 `queryFn`이 실패한 **Promise**(`reject`된)를 반환하지 않는다는 것을 뜻한다.

기억하자. **React Query**는 상태 코드 또는 네트워크 요청에 대해 전혀 알지 못한다(또는 신경쓰지 않는다). **React Query**는 `queryFn`이 제공하는 `resolve`되거나 `reject`된 `Promise`가 필요하다.

만약 **React Query**가 `reject`된 `Promise`를 보았다면 잠재적으로 데이터 패칭을 다시 시작하고 `offline` 상태라면 `query`를 일시 정지하며 결국 `query`를 `error` 상태로 만들 수 있다. so it's quite an important thing to get right.

### The fetch API

운좋게도 **axios** 또는 **ky**와 같은 많은 데이터 패칭 라이브러리들이 `4xx` 또는 `5xx`와 같은 거대한 상태 코드들을 실패한 `Promise`로 변환시켜주기 때문에 만약 네트워크 요청이 실패한 경우 `query` 또한 실패할 것이다. 대표적인 예외가 하나 있는데 바로 빌트인 **fetch** **API**이다. 이는 네트워크 오류로 인해 요청이 실패한 경우에만 실패한 `Promise`(`reject`된) 제공한다.

This is of course [documented here](https://tanstack.com/query/v4/docs/guides/query-functions#usage-with-fetch-and-other-clients-that-do-not-throw-by-default), but it's still a stumbling block if you've missed this.

```jsx
useQuery(['todos', todoId], async () => {
  const response = await fetch('/todos/' + todoId);
  // 🚨 4xx or 5xx are not treated as errors
  return response.json();
});
```

이를 해결하기 위해선 응답이 `ok`인지 아닌지 확인하고 `ok`가 아닐 경우 `reject`된 `Promise`로 변환해줘야 한다.

```jsx
useQuery(['todos', todoId], async () => {
  const response = await fetch('/todos/' + todoId);
  // ✅ transforms 4xx and 5xx into failed Promises
  if (!response.ok) {
    throw new Error('Network response was not ok');
  }
  return response.json();
});
```

### Logging

필자가 많이 본 두 번째 이유는 `queryFn` 내부에서 로깅 목적으로 에러를 잡고 있기 때문이었다.

```jsx
useQuery(['todos', todoId], async () => {
  try {
    const { data } = await axios.get('/todos/' + todoId);
    return data;
  } catch (error) {
    console.log(error);
    // 🚨 here, an "empty" Promise<void> is returned
  }
});
```

이와 같이 하고 싶다면 에러를 다시 던져줘야 한다는 것을 기억하자.

```jsx
useQuery(['todos', todoId], async () => {
  try {
    const { data } = await axios.get('/todos/' + todoId);
    return data;
  } catch (error) {
    console.log(error);
    // ✅ here, a failed Promise is returned
    throw error;
  }
});
```

오류를 처리하기 위한 대안으로는 `useQuery`의 `onError` 콜백을 사용하는 것이다.

```jsx
useQuery(
  ['todos', todoId],
  async () => {
    const { data } = await axios.get('/todos/' + todoId);
    return data;
  },
  { onError: (error) => console.error(error) }
);
```

I definitely prefer the callbacks, and you can read more about differnt ways to handle errors in [#11: React Query Error Handling](https://tkdodo.eu/blog/react-query-error-handling).
