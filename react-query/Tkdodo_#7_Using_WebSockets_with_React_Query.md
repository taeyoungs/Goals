# #7 Using WebSockets with React Query

> 원본 글  
> https://tkdodo.eu/blog/using-web-sockets-with-react-query

**Series**

- [#1 Practical React Query](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%231_Practical_React_Query.md)
- [#2 React Query Data Transformations](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%232_React_Query_Data_Transformations.md)
- [#3 React Query Render Optimizations](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%233_React_Query_Render_Optimizations.md)
- [#4 Status Checks in React Query](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%234_Status_Checks_in_React_Query.md)
- [#6 React Query and TypeScript](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%236_React_Query_and_TypeScript.md)
- #7 Using WebSockets with React Query (현재)
- [#8 Effective React Query Keys](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%238_Effective_React_Query_Keys.md)
- [#8a Leveraging the Query Function Context](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%238a_Leveraging_the_Query_Function_Context.md)
- [#9 Placeholder and Initial Data in React Query](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%239_Placeholder_and_Initial_Data_in_React_Query.md.md)
- [#10 React Query as a State Manager](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%2310_React_Query_as_a_State_Manager.md)
- [#11 React Query Error Handling](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%2311_React_Query_Error_Handling.md)
- [#12 Mastering Mutations in React Query](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%2312_Mastering_Mutations_in_React_Query.md)
- [#13 Offline React Query](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%2313_Offline_React_Query.md)
- [#14 React Query and Forms](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%2314_React_Query_and_Forms.md)
- [#15 React Query FAQs](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%2315_React_Query_FAQs.md)

**목차**

- [#7 Using WebSockets with React Query](#7-using-websockets-with-react-query)
    - [개요](#개요)
  - [What are WebSockets](#what-are-websockets)
  - [React Query integration](#react-query-integration)
  - [Step by Step](#step-by-step)
  - [Consuming data](#consuming-data)
  - [Partial data updates](#partial-data-updates)
  - [Increasing Stale Time](#increasing-stale-time)

### 개요

필자가 최근에 가장 많이 받는 질문 중 하나는 **React Query**와 **WebSocket**을 함께 사용하여 실시간 데이터를 다루는 방법에 대한 것이다. 그래서 필자가 한번 시도해보고 테스트한 것들 그리고 찾은 것들을 기록해볼까 한다.

## What are WebSockets

간단히 말해서, **WebSocket**을 사용하면 서버에서 클라이언트(브라우저)로 `Push` 메시지 또는 실시간 데이터를 보낼 수 있다. **HTTP**를 사용하면 일반적으로 클라이언트는 클라이언트가 원하는 특정 데이터를 서버에게 요청하게 되고 서버는 데이터 또는 에러와 함께 응답을 한 후에 연결이 닫힌다.

일반적인 **HTTP** 통신에서 클라이언트는 연결을 열고 요청을 시작하는 주체이기 때문에 서버가 업데이트 가능한 때라는 것을 알아도 클라이언트에게 데이터를 보낼 수 있는 방법이 없다.

여기서 **WebSocket**이 필요하다.

다른 HTTP 요청과 마찬가지로 브라우저는 연결을 시작하지만 해당 연결을 **WebSocket**으로 진행하고 싶다는 뜻을 서버에게 드러낸다. 만약 서버가 이를 받아들인다면, 서버는 프로토콜(**HTTP** ⇒ **WebSocket**)을 변경할 것이다. 이 연결은 클라이언트와 서버 둘 다 해당 연결을 종료하기로 결정하기 전까지 종료되지 않고 열린 채로 유지될 것이다. 이제 양측이 데이터를 전송할 수 있는 완전한 양방향 연결이 생겼다.

이 방법의 주요 장점은 이제 서버가 클라이언트에게 선택적 업데이트를 진행할 수 있다라는 점이다. 이는 다수의 사용자가 동일한 데이터를 바라보고 있는 상태에서 한 사용자가 업데이트를 만들어 냈을 때 유용할 수 있다. 일반적으로 다른 클라이언트들은 그들이 주체적으로 `refetch`를 진행하지 않는 이상 업데이트를 확인할 수 없다. 하지만 **WebSocket**을 사용하면 이러한 업데이트를 실시간으로 즉시 발생시킬 수 있다.

## React Query integration

**React Query**는 주로 **클라이언트 사이드** **비동기 상태 관리 라이브러리**이기 때문에 필자는 서버에서 **WebSocket**을 어떻게 셋업하는지에 대해서는 말하지 않을 것이다. 솔직히 필자는 해당 작업을 진행해본 적이 없으며, 이러한 작업은 우리가 사용하는 백엔드가 무엇이냐에 따라 다를 것이다.

**React Query**는 **WebSocket**을 위한 어떠한 빌트인 기능도 내장되어 있지 않다. 다만, 그렇다고 해서 **WebSocket**을 지원하지 않거나 관련 라이브러리와 잘 동작하지 않는다라는 말은 아니다. It's just that React Query is *very* agnostic when it comes to how you fetch your data: 모든건 `Promise`를 통한 작업이 **resolved** 되거나 **rejected**될 뿐이며 나머지는 우리에게 달려있다.

## Step by Step

일반적인 아이디어는 **WebSocket** 없이 작업했던 평소처럼 **Query**를 만드는 것이다. 대부분의 경우 우리는 `Entity`들을 **Query**하고 **Mutate**하는 HTTP 엔드포인트를 가지고 있다.

```tsx
const usePosts = () => useQuery(['posts', 'list'], fetchPosts);

const usePost = (id) => useQuery(['posts', 'detail', id], () => fetchPost(id));
```

여기 추가적으로, 우리는 **app-wide** 즉, **App**이 `init`되는 곳에서 `useEffect`를 통해 **WebSocket** 엔드포인트와 연결하는 작업을 셋업할 수 있다. 이러한 작업의 동작 방식은 우리가 사용하고자 하는 기술에 따라 다르다. 필자는 사람들이 [Hasura](https://github.com/TanStack/query/issues/171#issuecomment-649810136)를 통해 실시간 데이터를 구독하는 것도 봤었다. [Firebase](https://aggelosarvanitakis.medium.com/a-real-time-hook-with-firebase-react-query-f7eb537d5145)를 통해서 연결하는 방법에 대한 좋은 포스팅도 존재한다. 현재 예제에서는 간단하게 브라우저 native **WebSocket** API를 사용할 것이다.

```tsx
const useReactQuerySubscription = () => {
  React.useEffect(() => {
    const websocket = new WebSocket('wss://echo.websocket.org/');
    websocket.onopen = () => {
      console.log('connected');
    };

    return () => {
      websocket.close();
    };
  }, []);
};
```

## Consuming data

---

**WebSocket** 연결을 완료했다면 우리에겐 **WebSocket**을 통해 데이터가 들어왔을 때 호출될 일종의 콜백 함수가 있을 것이다. 다시 말하지만, 해당 데이터가 무엇인지는 전적으로 설정 방법에 따라 다르다. 필자는 Tanner Linsley의 이 [메시지](https://github.com/TanStack/query/issues/171#issuecomment-649716718)에서 영감을 받아 완전한 데이터 객체 대신 백엔드에서 이벤트를 보내는 것을 좋아한다.

```tsx
const useReactQuerySubscription = () => {
  const queryClient = useQueryClient();
  React.useEffect(() => {
    const websocket = new WebSocket('wss://echo.websocket.org/');
    websocekt.onopen = () => {
      console.log('connected');
    };
    websocket.onmessage = (event) => {
      const data = JSON.parse(event.data);
      const queryKey = [...data.entity, data.id].filter(Boolean);
      queryClient.invalidateQueries(querykey);
    };

    return () => {
      websocket.close();
    };
  }, [queryClient]);
};
```

> **WebSocket**으로 업데이트할 데이터 객체를 보내는 것이 아니라 `queryKey`에 대한 정보를 보내 해당 `Query`를 `invalidating`하여 최신 상태 데이터를 가져오는 식을 주로 택한다라는 것 같다.
>
> **React Query**를 사용하지 않는 상황에서라면 모르겠는데 **React Query**를 사용하고 있는 상황이라면 이 편이 훨씬 유용할 것 같다. 데이터 객체를 받아와서 어설프게 기존 데이터를 업데이트하는 것보다 중요한 것은 업데이트를 해야할 타이밍이니 기존 API를 활용해서 기존 데이터를 최신 상태로 만들어주는 것이다.

이벤트를 받을 때 상세 정보와 리스트를 업데이트하는 데 필요한 것들은 아래의 정보가 전부다.

- `{ "entitiy": ["posts", "list"] }` 은 포스팅 목록을 `invalidate` 할 것이다.
- `{ "entitiy": ["posts", "list"], id: 5 }` 단일 포스팅을 `invalidate` 할 것이다.
- `{ "entitiy": ["posts"] }`는 관련 포스팅 전부를 `invalidate` 할 것이다.

**Query invalidating**은 **WebSocket**과 함께 사용하면 정말 좋다. 이 접근 방법은 과도한 Pushing 문제를 피할 수 있다. 왜냐하면 우리가 현재 관심이 없는 `Entity`에 대한 이벤트를 수신하면 아무 일도 일어나지 않을 것이기 때문입니다. 예를 들어, 현재 **Profile** 페이지에 있고 포스팅에 대한 업데이트 이벤트를 수신하는 경우 `invalidateQueries`는 다음에 포스팅 페이지에 도달할 때 최신 상태의 포스팅 데이터를 가져오도록 `refetch` 될 것이다. 하지만 당장 `refetch`가 이루어지진 않을 것이다. 왜냐하면 지금 당장 활성화된 **observer**가 없기 때문이다. 만약 우리가 해당 페이지를 다시 가지 않는다면, 도착했던 업데이트 이벤트는 필요없게 될 테니까 말이다.

> **React Query**는 **Query**나 **Mutation**들에 대한 **Observer**를 관리하고 있는 상황이다. **Query**를 사용하는 페이지에 다시 가면 해당 **Query**에 대한 **Observer**가 활성화될테지만 페이지를 다시 가지 않는다면 **Observer**가 활성화될 일도 없고 새로운 데이터를 네트워크 요청으로 가져올 일도 없게 된다는 뜻이다.

## Partial data updates

물론 작지만 빈번한 업데이트 이벤트를 수신하는 거대한 데이터 더미가 있는 경우 여전히 **WebSocket**을 통해서 이벤트가 아닌 데이터의 일부분을 `Push`하고 싶을 수도 있다.

포스팅의 제목이 바뀐 상황 ? 제목을 `Push` 하면 된다. 수 많은 좋아요 ? `Push` 해버리면 된다.

이러한 부분 업데이트들을 위해, 우리는 특정 **Query**를 `invalidating` 하는 대신 **Query Cache**를 직접 `queryClient.setQueryData`를 이용해서 업데이트할 수 있다.

만약 동일한 데이터에 여러 **Query Key**를 가지고 있는 경우 이 작업은 조금 복잡해진다. 예를 들어, 만약 **Query Key**의 일부로 다수의 필터 기준이 존재한다거나 같은 메시지로 리스트 또는 상세 뷰를 업데이트하고자 하는 경우가 복잡해질 수 있다. [queryClient.setQueriesData](https://tanstack.com/query/v4/docs/reference/QueryClient?from=reactQueryV3&original=https://react-query-v3.tanstack.com/reference/QueryClient#queryclientsetqueriesdata)는 이러한 **use-case**도 잘 처리할 수 있도록 **React Query**에 비교적 최근에 추가됐다.

```tsx
const useReactQuerySubscription = () => {
  const queryClient = useQueryClient();

  React.useEffect(() => {
    const websocket = new WebSocket('wss://echo.websocket.org/');
    websocket.onopen = () => {
      console.log('connected');
    };
    websocket.onmessage = (event) => {
      const data = JSON.parse(event.data);
      queryClient.setQueriesData(data.entity, (oldData) => {
        const update = (entity) => (entity.id === data.id ? { ...entity, ...data.payload } : entity);
        return Array.isArray(oldData) ? oldData.map(update) : update(oldData);
      });
    };

    return () => {
      websocket.close();
    };
  }, [queryClient]);
};
```

> `Array.isArray(oldData) ? oldData.map(update) : update(oldData)`
>
> 배열이라면 `map`에 `update` 함수를 전달하여 각 객체별로 `update` 함수를 실행하고 그게 아니라 애초에 `oldData`가 단일 객체라면 `update`에 `oldData`를 인자값으로 전달하여 단일 객체 업데이트 실행

필자는 위 작업 자체가 너무 동적이고 추가 또는 삭제를 처리하지 않는다는 점 그리고 **TypeScript** 또한 위 작업에 대한 타입 추론이 올바르게 이루어지지 않는 점들을 때문에 `query invalidation`을 개인적으로는 더 선호한다.

그럼에도 불구하고 다음 [Codesandbox](https://codesandbox.io/s/react-query-websockets-ep1op) 예제에서는 두 가지 방식에 대해 모두 다루고 있다. (`invalidation`과 `partial updates`) 이 예제에서 커스텀 훅은 조금 복잡한데 동일한 **WebSocket**으로 `Server round trip`을 재현하고 있기 때문이다. 따라서 실 서버에서는 커스텀 훅의 복잡도가 다를테니 걱정하지 않아도 된다.

## Increasing Stale Time

**React Query**는 `staleTime`이 기본적으로 0으로 설정되어 있다. 이는 모든 **Query**가 즉각적으로 `stale` 상태가 된다는 것을 의미한다. 즉, 새로운 `subscriber`가 등장하거나 사용자가 **window**의 `focus`를 다시 발생시킬 때마다 `refetch`를 발생시킨다는 것이다. 필요에 따라 데이터를 최신 상태로 유지하는 것을 목표로 한다.

이러한 목표는 실시간으로 데이터를 업데이트하고자 하는 **WebSocket**과 많이 겹친다. 이 말인 즉슨 **React Query**는 기본적으로 이런 목표를 가지고 있고 목표에 맞는 처리가 기본적으로 이루어지고 있는데 서버에서 전용 메시지(이벤트 메시지)를 통하여 우리에게 업데이트하라고 말했다고 해서 우리가 직접 `invalidate`한 데이터들을 또 다시 `refetch` 해야할까?

어쨋든 만약 **WebSocket**을 통하여 모든 데이터를 업데이트하고자 한다면 `staleTime`을 높게 잡는 것을 고려해보자. 앞서 언급한 예제에서 필자는 `staleTime`을 `Infinity`로 잡아놨다. 이는 데이터를 처음엔 **useQuery**를 통하여 가져온 다음 그 이후엔 항상 캐시에서 가져오겠다는 것을 의미한다. `Refetching`은 오직 `query invalidation`을 직접 실행했을 때만 발생한다.

이 작업을 적용하는 제일 좋은 방법은 `QueryClient`가 생성될 때 전역 **Query** 옵션을 변경해주는 것이다.

```tsx
const queryClient = new QueryClient({
  defaultOpitons: {
    queries: {
      staleTime: Infinity,
    },
  },
});
```
