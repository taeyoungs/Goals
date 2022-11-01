# #4 Status Checks in React Query

> 원본 글  
> https://tkdodo.eu/blog/status-checks-in-react-query

**Series**

- [#1 Practical React Query](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%231_Practical_React_Query.md)
- [#2 React Query Data Transformations](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%232_React_Query_Data_Transformations.md)
- [#3 React Query Render Optimizations](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%233_React_Query_Render_Optimizations.md)
- #4 Status Checks in React Query (현재)
- [#6 React Query and TypeScript](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%236_React_Query_and_TypeScript.md)
- [#7 Using WebSockets with React Query](<(https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%237_Using_WebSockets_with_React_Query.md)>)
- [#8 Effective React Query Keys](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%238_Effective_React_Query_Keys.md)
- [#8a Leveraging the Query Function Context](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%238a_Leveraging_the_Query_Function_Context.md)

**목차**

- [#4 Status Checks in React Query](#4-status-checks-in-react-query)
    - [개요](#개요)
  - [The standard example](#the-standard-example)
  - [Background errors](#background-errors)

### 개요

**React Query**의 장점 중 하나는 Query의 `status` 필드에 쉽게 접근이 가능하다는 점이다. 이를 통해 **Query**가 로딩 상태인지 에러 상태인지 즉시 알 수 있다. 이를 위해서 **React Query**에서는 내부 상태 머신으로부터 여러 `boolean` 플래그 값을 노출시킨다. [타입들](https://github.com/TanStack/query/blob/f2137dc4e4553256c4ebc1891b548fe35efe9231/src/core/types.ts#L250)을 살펴봤을 때 우리의 **Query**는 다음 중 하나의 상태를 가진다.

- `success`: **Query**는 성공했으며, 해당 **Query**에 대한 데이터를 보유한 상태
- `error`: **Query**가 정상적으로 동작하지 않았으며, `error` 값이 설정된 상태
- `loading`: **Query**가 가지고 있는 데이터는 없으며, 현재 처음으로 데이터를 가져오는 중인 상태
- `idle`: **Query**를 사용할 수 없기 때문에 한번도 실행되지 않은 상태

> **Update**: React Query v4에서 `idle` 상태를 제거될 예정

한 가지 짚고 넘어갈 점은 `isFetching` 플래그는 내부 상태 머신의 일부가 아니라는 점이다. 이는 요청 중일 경우 언제나 `true` 값으로 설정되는 플래그 값이다. 이는 다시 말해 Query는 데이터 패칭 중이면서 성공 상태일 수도 있지만 데이터 패칭 중이면서 실패한 상태일 수도 있다. 단, 내부 상태가 `loading`과 `success`를 동시에 가지는 경우는 없다. 내부 상태 머신이 이를 보장하기 때문이다.

> Update: **v4**에서 `isFetching` 플래그는 `fetchStatus`라는 두 번째 상태 객체 하위로 옮겨진다. 마치 새로운 `isPaused` 플래그처럼

## The standard example

`idle` 상태는 **Query**를 사용할 수 없는 상태에 해당되는 엣지 케이스이기 때문에 대부분 생략된다. 그래서 대부분의 예제는 다음과 같다.

> 위에서 언급했듯이 `idle` 상태값은 **v4**에서 제거된다.

```tsx
const todos = useTodos();

if (todos.isLoading) {
  return 'Loading...';
}

if (todos.error) {
  return 'An error has occurred: ' + todos.error.message;
}

return <div>{todos.data.map(renderTodo)}</div>;
```

위에서는 먼저 `loading` 상태와 `error` 유무를 먼저 체크하고 데이터를 렌더링한다. 이는 몇 가지 경우에는 괜찮을지 몰라도 나머지 경우에선 그렇지 않다. 여러 데이터 패칭 사용 방법에서, 특히 우리가 직접 작성하게 되는 코드들에서는 사용자의 상호 작용을 통한 `refetch`를 제외하고선 `refetch mechanism`이 존재하지 않는다.

하지만 **React Query**에는 존재한다.

**React Query**는 기본적으로 사용자의 `refetch` 요청 없이도 매우 적극적으로 `refetch`를 실행한다. `refetchOnMount`, `refetchOnWindowFocus`, `refetchOnReconnect`의 개념은 데이터를 정확하게 유지하는 데 좋지만 **automatic background refetch**가 실패하는 경우에는 **ux** 측면에서 혼란을 불러일으킬 수도 있다.

## Background errors

`background refetch`가 실패할 경우 이는 대개 조용히 무시될 수 있다. 그러나 위 코드는 그렇지 않다. 다음 두 가지 예제를 살펴보자.

- 사용자가 페이지를 열면 첫 번째 **Query**는 성공적으로 이뤄진다. 사용자가 일정 시간을 해당 페이지에 머물다가 이메일을 체크하기 위해 브라우저 탭을 변경했다. 몇 분 후 사용자가 다시 해당 페이지로 돌아왔을 때 **React Query**는 `background refetch`를 실행할 것이다. 그리고 이 데이터 패칭은 실패한다.
- 사용자가 리스트 뷰가 존재하는 페이지에 머물다가 리스트 중 하나의 아이템에 대한 상세 페이지로 이동하기 위해 하나를 클릭했다. 상세 페이지로의 이동은 정상적으로 이루어 졌기에 다시 리스트 뷰가 존재하는 페이지로 돌아왔다. 그리고 다시 상세 페이지로 이동했을 때 사용자는 캐시로부터 받은 데이터를 보게 될 것이다. 이 자체는 괜찮다. `background refetch`가 실패하는 경우만 제외하면

위 두 상황 모두, Query는 다음과 같은 상태를 갖게 될 것이다.

```json
{
  "status": "error",
  "error": { "message": "Something went wrong" },
  "data": [{ ... }]
}
```

보시다시피, 우리는 `error` 객체와 `stale` 상태의 데이터 모두 사용할 수 있게 된다. 이는 **React Query**의 장점이다. **React Query**는 `stale-while-revalidate` 캐싱 매커니즘을 따르기 때문에 오래된 데이터일지라도 데이터가 존재하기만 한다면 언제든지 데이터를 제공해준다.

이제 사용자에게 무엇을 보여줄 지는 우리에게 달려있다. 에러를 보여주는 것이 중요한가? 아니면 `stale` 상태의 데이터를 보여주는 것만으로 충분한가? 그것도 아니라면 조그만 **background error indicator**와 함께 두 가지 모두 보여주는 것이 맞는가?

이러한 질문에 정답은 없다. 이는 때에 따라 다르다. 그러나 위 2가지 예를 고려한다면, 데이터가 `error` 화면으로 대체되는 상황은 사용자 경험상 혼란을 가져다 줄 수도 있다.

이것은 **React Query**가 `exponential backoff`를 사용하여 기본적으로 실패한 쿼리를 세 번 다시 시도한다는 점을 고려할 때 훨씬 더 관련이 있다. 따라서 `stale` 상태의 데이터가 `error` 화면으로 대체될 때까지 몇 초가 걸릴 수 있다.

> This is even more relevant when we take into account that React Query will retry failed queries three times per default with exponential backoff, so it might take a couple of seconds until the stale data is replaced with the error screen.

이것이 필자가 주로 데이터 사용 가능 여부를 먼저 체크하는 이유이다.

```tsx
const todos = useTodos();

if (todos.data) {
  return <div>{todos.data.map(renderTodo)}</div>
}

if (todos.error) {
  return 'An error has occurred: ' todos.error.message;
}

return 'Loading...'
```

다시 말하지만, 때에 따라 다르기 때문에 명확한 원칙은 없다. 그저 **React Query**를 사용하는 모든 사람이 **React Query**의 공격적인 리패칭의 결과에 대해서 알고 있어야만 하며 위 예제들의 형태를 엄격하게 따르기 보다는 상황에 맞춰 코드를 구조화해야 한다.
