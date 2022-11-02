# #10 React Query as a State Manager

> 원본 글  
> https://tkdodo.eu/blog/react-query-as-a-state-manager

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
- #10 React Query as a State Manager (현재)

**목차**

- [#10 React Query as a State Manager](#10-react-query-as-a-state-manager)
  - [개요](#개요)
  - [An Async State Manager](#an-async-state-manager)
  - [A data synchronization tool](#a-data-synchronization-tool)
    - [Before React Query](#before-react-query)
  - [Stale While Revalidate](#stale-while-revalidate)
  - [Smart refetches](#smart-refetches)
    - [`refetchOnMount`](#refetchonmount)
    - [`refetchOnWindowFocus`](#refetchonwindowfocus)
    - [`refetchOnReconnect`](#refetchonreconnect)
  - [Letting React Query do its magic](#letting-react-query-do-its-magic)
  - [Customize `staleTime`](#customize-staletime)
    - [Bonus: using setQueryDefaults](#bonus-using-setquerydefaults)
  - [A note on separation of concerns](#a-note-on-separation-of-concerns)
  - [Takeaways](#takeaways)

### 개요

**React Query**는 React 애플리케이션에서 데이터 패칭을 대폭 간소화해주기 때문에 많은 사랑을 받고 있다. 따라서, **React Query**가 사실은 데이터 패칭 라이브러리가 아니다라고 말하면 놀랄 수도 있다.

**React Query**는 직접 데이터 패칭을 하지 않으며 아주 작은 기능들의 모음만이 네트워크에 연결되어 있을 뿐이다(**React Query**의 `OnlineManage`, `refetchOnReconnect` 등과 같은 기능들). 이는 `queryFn`을 처음 작성해질 때 분명해지며, 데이터를 실제로 가져오기 위해서는 `fetch`, `axios`, `ky` 또는 `graphql-request`와 같은 무언가를 사용해야만 한다.

그래서 만약 **React Query**가 데이터 패칭 라이브러리가 아니라면, 그럼 무엇일까?

## An Async State Manager

**React Query**는 비동기 상태 매니저다. 이는 모든 형태의 비동기 상태를 관리할 수 있다. 맞다. 대부분의 경우 우리는 데이터 패칭을 통해 `Promise`를 생성하기에 **React Query**가 여기서 빛을 발하게 되는 것이다. **React Query**는 그저 로딩이나 에러 상태를 관리해주는 것보다 더 많은 것들을 해주기에 **전역 상태 관리자**(`global state manager`)이기도 하다. `QueryKey`는 내가 정의한 **Query**를 고유하게 식별하기 때문에 다른 두 장소에서 동일한 `Key`로 **Query**를 호출할 경우 두 **Query**는 동일한 데이터를 가져올 것이다. 이는 실제로 데이터 패칭 함수에 두 번 접근할 필요가 없도록 커스텀 훅으로 잘 추상화할 수 있다.

```jsx
export const useTodos = () => useQuery(['todos'], fetchTodos);

function ComponentOne() {
  const { data } = useTodos();
}

function ComponentTwo() {
  // ✅ will get exactly the same data as ComponentOne
  const { data } = useTodos();
}

const queryClient = new QueryClient();

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <ComponentOne />
      <ComponentTwo />
    </QueryClientProvider>
  );
}
```

위 컴포넌트들(`ComponentOne`, `ComponentTwo`)은 컴포넌트 트리의 모든 위치에 존재할 수 있다. 해당 컴포넌트들이 동일한 `QueryClientProvider` 아래에 있는 한 동일한 데이터를 가져올 것이다. **React Query**는 동시에 발생하는 요청에 대해서도 중복을 제거하기 때문에 위 예제에서 두 컴포넌트가 동일한 데이터를 요청하더라도 네트워크 요청은 하나만 있을 것이다.

## A data synchronization tool

**React Query**가 비동기 상태를 관리하기 때문에 (또는 서버 상태에 대한 데이터 패칭 측면을) 프론트엔드 애플리케이션은 데이터(서버 데이터)를 소유하지 않는다고 가정한다. 맞는 말인 것이 만약 우리가 **API**를 통해 가져온 데이터를 화면에 표시한다고 하면, 우리는 오직 데이터의 `Snapshot`을 보여줄 뿐이다.

> 여기서 `Snapshot`이란 API를 통해 데이터를 검색했을 가져와지는 그때의 데이터를 말한다.

그렇기에 여기서 우리가 스스로에게 던져야할 질문은

**우리가 가져온 데이터는 패칭이 끝난 후에도 정확할까?**

답은 전적으로 "우리가 어떻게 하느냐에 따라 다르다"라는 것이다. 만약 우리가 특정 트위터 포스팅의 좋아요와 댓글 수를 패칭해 왔다면, 이는 금세 오래된(구식의, 최신 상택가 아닌) 데이터가 될 수도 있다. 만약 매일 업데이트되는 환율 정보를 패칭해 왔다면, 해당 데이터는 얼마 동안은 데이터 패칭을 다시 하지 않아도 정확할 것이다.

**React Query**는 실제 데이터의 주인(백엔드)과 동기화하는 수단을 제공한다. 이러한 동기화 수단으로 인해 충분히 자주 업데이트하지 않는 것보다 자주 업데이트하는 측면에서 오류가 발생한다.

> 백엔드 서버가 가진 데이터와 동기화하는 수단을 제공하기 때문에 자주 업데이트하지 않아서 에러가 발생하는 이전 상황에 비해 오히려 자주 업데이트함으로 인해 오류가 발생한다 라는 뜻인 것 같다. 에러가 발생하는 측면 또는 상황이 바뀌었다는 뜻

### Before React Query

데이터 패칭을 하기 위한 두 가지 접근 방식은 **React Query**와 같은 라이브러리가 와서 도와주기 전가지는 꽤나 일반적인 방식이었다.

- **한번 패칭하고 전역에 저장한 뒤 업데이트를 드물게 하는 방식**

  이는 **Redux**를 사용해서 많이 해왔던 작업들이다. 어디선가 주로 애플리케이션이 `mount`될 때 데이터 패칭을 시작하는 `action`을 `dispatch`한다. 데이터를 가져온 후에 애플리케이션 내 어디서나 해당 데이터에 접근할 수 있도록 전역 상태 관리자에 데이터를 저장한다. 그리고 나서 많은 컴포넌트들이 `Todo` 리스트에 접근한다.
  우리는 데이터를 다시 요청할까(`refetch`)? 아니, 우리는 이미 데이터를 `다운로드`했다. 이미 데이터를 가지고 있는 상태인데 굳이 다시 요청해야 할까?
  아마 우리가 백엔드에 `POST` 요청을 날릴 경우, 백엔드는 `최신` 상태를 우리에게 보내줄 것이다. 더 정확한 것을 원하면 언제나 브라우저를 새로고침할 수도 있다.

- `mount`될 때마다 데이터를 패칭하고 로컬에 데이터를 유지하는 방식

  이따금 우리는 전역 상태에 패칭한 데이터를 넣는 것은 `과하다`라고 생각할지도 모른다. 데이터가 **Modal Dialog**에서만 필요하다면 **Dialog**가 열렸을 때 바로 데이터를 패칭해도 되지 않을까? 그 다음은 말 안해도 알 것이다. `useEffect`의 의존성에 빈 배열을 위치시키고 `setLoading(true)`를 하고 … 물론 이제 우리는 **Dialog**가 열릴 때마다 데이터 패칭이 완료되기 전까지 로딩 스피너를 봐야한다. What else can we do, the local state is gone…

두 가지 접근 방식 모두 차선책이다. 첫 번째 방식은 로컬 캐시를 충분히 자주 업데이트하지 못한다. 반면에 두 번째 방식은 잠재적으로 너무 자주 `refetch`가 발생하고 두 번째 데이터 패칭 때 이전에 가져온 데이터가 없기 때문에 의심스러운 **UX**를 계속 경험해야 한다.

**React Query**는 이러한 문제들을 어떻게 풀어냈을까?

## Stale While Revalidate

이전에 들어서 알고 있겠지만 이는 **React Query**가 사용하고 있는 캐싱 메커니즘이다. It's nothing new - you can read about the [HTTP Cache-Control Extensions for Stale Content here](https://datatracker.ietf.org/doc/html/rfc5861).

요약하자면, **React Query**는 데이터가 더 이상 최신 상태가 아닐지라도(오래된) 데이터를 캐싱하고 필요할 때 제공한다. **React Query**가 갖는 원칙은 `데이터가 없다` 라는 건 로딩 스피너를 의미하고 이는 사용자에게 `**느리다**` 라는 인식을 심어줄 것이기 때문에 "**데이터가 없는 것보다는 오래된 데이터라도 있는게 낫다**"라는 것이다. 동시에 해당 데이터의 유효성을 다시 확인하기 위해 `Background refetch`를 시도한다.

## Smart refetches

**Cache invalidation**은 꽤나 힘들다. 그렇다면 백엔드에게 새로운 데이터를 다시 요청하기 위한 시간은 언제로 해야할까? 당연히 우리는 `useQuery`를 호출하는 컴포넌트가 리렌더링될 때마다 매번 백엔드에게 새로운 데이터를 요청할 순 없다. 이는 현대를 기준으로도 상당히 높은 비용의 작업일 것이다.

그래서 **React Query**는 `refetch`를 다시 발생시키기 위한 전략적 지점을 선택했다. 좋은 지표라고 생각되는 전략적 지점 즉, 데이터를 패칭하기 좋은 때들은 다음과 같다.

### `refetchOnMount`

`useQuery`를 호출하는 새로운 컴포넌트가 `mount`될 때마다 **React Query**는 `revalidation`을 수행할 것이다.

### `refetchOnWindowFocus`

우리가 브라우저 탭을 누를 때마다 `refetch`를 수행할 것이다. 이는 필자가 생각하기에 `revalidation`을 수행하기 가장 좋은 지점이라고 생각하지만 종종 오해를 사곤 한다. 개발 중에는 브라우저 탭을 자주 변경하기에 우리는 `refetch`가 너무 과하다라고 인지할 수도 있다. 그러나 프로덕션 중에는 사용자가 탭을 눌렀다라는 것이 사용자가 메일을 확인하거나 트위터를 읽다가 다시 우리의 애플리케이션으로 돌아왔음을 나타낸다. 이 상황에서 최신 업데이트를 보여주는 것이 제일 적합한 타이밍이다.

### `refetchOnReconnect`

만약 네트워크 연결이 끊기거나 다시 연결된 경우도 화면에 표시되는 내용을 `revalidate`하기 좋은 지표가 된다.

마지막으로 만약 애플리케이션의 개발자가 `revalidate`하기 좋은 지점을 알고 있다면 `queryClient.invalidateQueries`를 통하여 명시적으로 `invalidation`을 시도할 수도 있다. 이는 보통 `mutation`을 수행한 후에 이뤄지곤 한다.

## Letting React Query do its magic

필자는 이 [기본적인 옵션들](https://tanstack.com/query/v4/docs/guides/important-defaults?from=reactQueryV3&original=https://react-query-v3.tanstack.com/guides/important-defaults)을 사랑하지만 이전에도 말했듯이 이러한 옵션들은 네트워크 요청의 양을 최소화하기 위한 것이 아니라 최신 상태로 유지하는 데 중점을 둔다. 주로 `staleTime`은 기본적으로 `0`으로 설정되어 있기에 이는 예를 들어, 새로운 컴포넌트가 `mount` 될 때마다 **Background refetch**가 발생한다는 것을 의미한다. 이러한 작업을 많이 수행하는 경우 특히나 동일한 렌더링 사이클에 있지 않은 짧으면서 연속된 `mount`의 경우 네트워크 탭에서 엄청 많은 양의 데이터 패칭을 볼 수도 있을 것이다. 이는 **React Query**가 다음과 같은 상황에 중복을 제거할 수 없기 때문이다.

```jsx
function ComponentOne() {
  const { data } = useTodos();

  if (data) {
    // ✋ mounts conditionally, only after we already have data
    return <ComponentTwo />
  }
  return <Loading />;
}

function ComponentTwo() {
  // ✋ will thus trigger a second network request
  const { data } useTodos();
}

const queryClient = new QueryClient();

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <ComponentOne />
    </QueryClientProvider>
  )
}
```

> What's going on here, I just fetched my data 2 seconds ago, why is there another network request happening? This is insane!
>
> — Legit reaction when using React Query for the first time

위 예제와 같은 상황에선 `props`를 통하여 데이터를 전달하거나 **prop drilling**을 피하기 위해 **React Context**에 데이터를 넣거나 `refetchOnMount`와 `refetchOnWindowFocus` 플래그를 꺼두는 것이 모두 좋은 생각처럼 보일 수 있다. 왜냐하면 이대로 놔둘 경우 너무 많은 데이터 패칭이 발생하기 때문이다.

일반적으로 `data` 객체를 `props`로 전달하는 것은 아무런 문제가 없다. 이는 우리가 할 수 있는 가장 명확한 일이고 위와 같은 예제에서는 잘 동작할 것이다. 그러나 좀 더 실제 상황에 맞게 예제를 수정하면 어떻게 될까?

```jsx
function ComponentOne() {
  const { data } = useTodos();
  const [showMore, toggleShowMore] = React.useReducer((value) => !value, false);

  // yes, I leave out error handling, this is "just" an example
  if (!data) {
    return <Loading />;
  }

  return (
    <div>
      Todo count: {data.length}
      <button onClick={toggleShowMore}>Show More</button>
      // ✅ show ComponentTwo after the button has been clicked
      {showMore ? <ComponentTwo /> : null}
    </div>
  );
}
```

이 예제에선, 우리의 `ComponentTwo` 컴포넌트(`Todo` 데이터에 의존하는)가 오직 사용자가 버튼을 클릭한 후에만 `mount` 될 것이다. 이제 사용자가 몇 분 지나서 해당 버튼을 클릭했다고 상상해보자. 사용자가 몇 분 후에 버튼을 클릭한 이 상황에서 `Todo` 목록의 최신 값을 볼 수 있도록 **Background refetch**를 수행하는 것이 좋지 않을까?

앞서 언급한 방법 중 기본적으로 **React Query**가 원하는 것을 우회하는 방법을 선택한 경우에는 이 작업이 불가능하다.

So how can we have our cake and eat it, too?

## Customize `staleTime`

아마 필자가 가고자 하는 방향을 이미 예상했을지도 모른다. 해결 방식은 각자의 특정한 **use-case**에 맞게 `staleTime`의 값을 변경하는 것이다. 여기서 알아야 할 점은

---

💡 데이터가 최신 상태인 이상, 데이터는 항상 캐시에서 가져올 것이다. 우리가 얼마나 자주 검색하느냐에 상관없이 새로운 데이터를 위한 네트워크 요청을 볼 수 없을 것이다.

---

`staleTime`에 **옳은** 값이란 없다. 대부분의 경우 기본 옵션(`staleTime: 0`)은 정말로 잘 동작할 것이다. 개인적으로는 해당 시간의 프레임에서 중복된 요청을 제거하기 위해 최소 **20초**로 `staleTime`을 설정하지만 `staleTime`을 어떻게 설정하냐는 전적으로 **React Query**를 사용하는 사람에게 달려있다.

### Bonus: using setQueryDefaults

**React Query** **v3** 이후로 **React Query**는 `QueryClient.setQueryDefaults`를 통하여 각 **Query Key** 별로 옵션의 기본값을 설정하는 아주 좋은 방법을 지원하고 있다. 그래서 만약 필자가 [#8: Effective React Query Keys](https://tkdodo.eu/blog/effective-react-query-keys)에서 언급한 패턴을 따르고 있다면 원하는 만큼 세부적으로 **Query Key**에 대한 기본값을 설정할 수 있다. 왜냐하면 `setQueryDefaults`에 전달하는 **Query Key** 또한 `the standard partial matching`(**Query Filters**도 사용하는 Query Key 매칭 방법)을 따르고 있기 때문이다.

```jsx
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      // ✅ globally default to 20 seconds
      staleTime: 1000 * 20,
    },
  },
});

// 🚀 everything todo-related will have a 1 minute staleTime
queryClient.setQueryDefaults(todoKeys.all, { staleTime: 1000 * 60 });
```

## A note on separation of concerns

`useQuery`와 같은 훅을 애플리케이션 내 모든 계층의 컴포넌트에 추가하면 컴포넌트가 수행해야 하는 작업의 책임이 뒤섞인 것처럼 보이는 것은 합당한 우려다. 예전엔 "smart-vs-dumb", "container-vs-presentational" 컴포넌트 패턴이 어디에나 있었다. 이러한 패턴들은 **명확한 분리**, **decoupling**, **재사용성** 그리고 **테스트 용이성**을 확보해줬었다. 왜냐하면 `presentational` 컴포넌트가 그저 `props`를 가져오기만 했기 때문이다. 바로 앞에서 말했던 점들을 제외하고도 많은 `prop drilling`, `boilerplate`, 정적 타입이 어려웠던 패턴들 그리고 임의의 컴포넌트 분리들 또한 가져왔었다.

이러한 점들은 훅이 등장하면서 많이 바뀌었다. 우리는 이제 `useContext`, `useQuery` 또는 `useSelector`(만약 **Redux**를 사용하고 있다면)를 어디에서나 사용할 수 있으며 훅을 통해 컴포넌트에 의존성을 주입할 수 있다. 물론 훅을 사용하면 컴포넌트가 더욱 더 결합된다고 말할 수 있다. 또한 애플리케이션에서 자유롭게 이동할 수 있고 그 제차로도 잘 동작할 것이기에 더 독립적이라고도 말할 수 있다.

I can totally recommend watching [Hooks, HOCS, and Tradeoffs (⚡️) / React Boston 2019](https://www.youtube.com/watch?v=xiKMbmDv-Vw) by redux maintainer [Mark Erikson](https://twitter.com/acemarke).

요약하자면, 이것들은 모두 **tradeoffs**라는 것이다. 아무것도 하지 않고 무언가를 얻을 수는 없다(There is no free lunch). 어떤 상황에서 효과가 있던 방법은 다른 상황에서 전혀 효과가 없을 수도 있다. 재사용 가능한 버튼이 데이터 패칭까지 수행해야만 할까? 아마도 아닐 것이다. `Dashboard`를 데이터를 전달하기 위해서 `DashboardView`와 `DashboardContainer`로 나누는 것은 합리적일까? 이 또한 아마도 아닐 것이다. 따라서 장단점을 파악하고 올바른 작업에 올바른 도구를 적용하는 것은 우리의 몫이다.

## Takeaways

**React Query**는 애플리케이션 내에서 비동기 전역 상태를 관리하기에 좋은 방법이다. 적합한 **use-case**에서만 `refetch` 플래그를 끄고 다른 상태 관리자와 서버 상태를 동기화하고 싶은 마음을 참아라. 보통의 경우 `statleTime`을 임의로 지정하는 것이 백그라운드 업데이트가 발생하는 빈도를 제어하는 동시에 훌륭한 **UX**를 얻는 데 필요한 전부일 것이다.
