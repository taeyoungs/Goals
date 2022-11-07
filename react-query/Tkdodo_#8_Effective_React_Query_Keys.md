# #8 Effective React Query Keys

> 원본 글  
> https://tkdodo.eu/blog/effective-react-query-keys

**Series**

- [#1 Practical React Query](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%231_Practical_React_Query.md)
- [#2 React Query Data Transformations](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%232_React_Query_Data_Transformations.md)
- [#3 React Query Render Optimizations](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%233_React_Query_Render_Optimizations.md)
- [#4 Status Checks in React Query](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%234_Status_Checks_in_React_Query.md)
- [#6 React Query and TypeScript](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%236_React_Query_and_TypeScript.md)
- [#7 Using WebSockets with React Query](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%237_Using_WebSockets_with_React_Query.md)
- #8 Effective React Query Keys (현재)
- [#8a Leveraging the Query Function Context](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%238a_Leveraging_the_Query_Function_Context.md)
- [#9 Placeholder and Initial Data in React Query](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%239_Placeholder_and_Initial_Data_in_React_Query.md.md)
- [#10 React Query as a State Manager](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%2310_React_Query_as_a_State_Manager.md)
- [#11 React Query Error Handling](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%2311_React_Query_Error_Handling.md)
- [#12 Mastering Mutations in React Query](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%2312_Mastering_Mutations_in_React_Query.md)
- [#13 Offline React Query](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%2313_Offline_React_Query.md)

**목차**

- [#8 Effective React Query Keys](#8-effective-react-query-keys)
    - [개요](#개요)
  - [Caching Data](#caching-data)
  - [Automatic Refetching](#automatic-refetching)
  - [Manual Interaction](#manual-interaction)
  - [Effective React Query Keys](#effective-react-query-keys)
    - [Colocate](#colocate)
    - [Always use Array Keys](#always-use-array-keys)
    - [Structure](#structure)
    - [Use Query Key factories](#use-query-key-factories)

### 개요

**Query Key**는 **React Query**에서 정말 중요한 개념이다. **Query Key**는 **React Query**가 내부적으로 데이터를 올바르게 캐시하고 **Query**에 대한 `dependency`가 변경될 때 자동으로 `refetch` 할 수 있도록 하기 위해 필요하다. 또한 **Query Cache**와 필요할 때 직접 상호 작용하기 위해서도 필요할 것이다. 예를 들어, **Mutation** 이후에 데이터를 업데이트하고자 할 때 또는 특정 **Query**들을 직접 `invalidate` 하고자 할 때

이러한 작업을 보다 효과적으로 수행할 수 있도록 개인적으로 **Query Key**를 구성하는 방법을 보여주기 전에 이 세 가지 사항이 의미하는 바를 빠르게 살펴보자.

## Caching Data

내부적으로 **Query Cache**는 직렬화된 **Query Key**인 `Key`와 메타데이터를 더한 **Query Data**인 `value`로 이루어진 **JavaScript** 객체이다. `Key`들은 `deterministic`한 방법으로 해시 처리되기에 `Key`에 객체를 사용해도 된다(on the top level, keys have to be strings or arrays though).

> 여기서 `deterministic way`라는 건 객체가 들어왔을 때 객체 내부 속성들의 순서에 상관없이 속성들이 동일하다면 같은 **Query Key**로 보는 방법을 말한다.

여기서 중요한 점은 **Query**의 `Key` 값은 고유한 값이어야 한다는 점이다. 만약 **React Query**가 캐시에서 동일한 `Key` 값을 찾는다면 해당 `Key`에 담긴 데이터를 그대로 사용한다. 주의해야할 점은 `useQuery`와 `useInfiniteQuery`에 동일한 **Key** 값을 사용하지 못한다는 점이다. 두 **Hooks**에 동일한 `Key` 값을 사용할 경우 오직 하나의 **Query Cache**만 존재하며(**Hooks**의 네이밍이 다르다고 동일한 `Key` 값에 대해 다른 **Query Cache**를 유지하고 있지 않는다) 이를 공유한다. 이러한 상황은 좋지 못한 것이 일반적인 **Query**(`useQuery`를 이용한)와 **infinite Query**(`useInfiniteQuery`를 이용한)는 구조가 근본적으로 다르기 때문이다.

```tsx
useQuery(['todos'], fetchTodos);

// 🚨 this won't work
useInfiniteQuery(['todos'], fetchInfiniteTodos);

// ✅ choose something else instead
useInfiniteQuery(['infiniteTodos'], fetchInfiniteTodos);
```

## Automatic Refetching

> _Queries are declarative._

이는 아무리 강조해도 지나치지 않은 매우 중요한 개념이며, and it's also something that might take some time to "click". 대부분의 사람들은 **Query**, 특히나 **Refetching**에 대해서 명령적인(선언적의 반대) 방법이라고 생각한다.

**Query**가 있다고 가정해보자. 그리고 해당 **Query**는 어떤 데이터를 가져온다. 버튼을 클릭해서 `refetch`를 발생은 시키고 싶지만 다른 파라미터와 함께 발생시키고 경우에 대해 코드를 작성한다고 했을 때 필자는 다음과 같은 코드들을 많이 접했었다.

```tsx
function Component() {
  const { data, refetch } = useQuery(['todos'], fetchTodos)

  // ❓ how do I pass parameters to refetch ❓
  return <Filters onApply={() => refetch(???)} />
}
```

여기에 대한 대답은 **그렇겐 할 수 없다.** 이다

`refetch`는 위 같은 상황을 위해 만들어진 기능이 아니다. - `refetch`는 처음 호출할 당시 사용된 파라미터들과 동일한 파라미터들을 이용하여 데이터를 다시 가져오기 위한 것이다.

만약 데이터를 변경하는 특정 상태값이 존재하는 경우, 우리가 해야하는 일은 **Query Key**에 해당 상태값을 넣는 것이다. 왜냐하면 **React Query**가 해당 `Key`가 변경될 때 자동으로 `refetch`를 발생시킬 것이기 때문이다. 따라서, 만약 필터를 적용하고 싶다면, **클라이언트 상태**를 변경하면 되는 것이다.

```tsx
function Component() {
  const [filters, setFilters] = React.useStater()

  const { data } = useQuery(['todos'], filters], () => fetchTodos(filters))

  // ✅ set local state and let it "drive" the query
  return <Filters onApply={setFilters} />
}
```

`setFilters` 호출에 의해 발생한 리렌더링은 **React Query**에 다른 **Query Key**를 전달할 것이고 이는 `refetch`를 발생시킬 것이다. 이러한 상황에 더 자세한 예제는 다음 링크를 참조하자. [#1: Practical React Query - Treat the query key like a dependency array](https://tkdodo.eu/blog/practical-react-query#treat-the-query-key-like-a-dependency-array)

## Manual Interaction

`Query Cache`와의 상호 작용은 **Query Key**의 구조가 가장 중요한 곳이다. **Query Filter**를 지원하는 `invalidateQueries` 또는 `setQueriesData`와 같은 대부분의 **Query Cache** 상호 작용 메서드들은 **Query Key**를 fuzzily(?)하게 일치시키는 것이 가능하다.

> `fuzzily` - 유사 ? 흐릿 ?
>
> **Query Filter**: `type?: 'active' | 'inactive' | 'all'`와 같이 **Query**에 대한 조건을 걸 수 있다. default는 `all`이고 `active` 또는 `inactive`로 설정할 경우 활성화 또는 비활성화 **Query**들만을 대상으로 할 수 있다.

## Effective React Query Keys

이 블로그에 나오는 모든 것을 포함하여 이러한 측면들은 필자의 개인적인 의견이라는 점을 기억하자. 따라서, **Query Key**로 작업할 때 여기서 나오는 방법이 무조건적인 정답이 아니란 것을 알고 있어야 한다. 필자는 이러한 전략은 앱이 더 복잡해질 때 가장 잘 작동하고 확장도 잘 된다는 것을 알고 있다. 그렇기에 Todo App과 같은 작은 규모의 애플리케이션에서는 이러한 전략이 필요하지 않을 수도 있을 것이다.

### Colocate

> [https://kentcdodds.com/blog/colocation](https://kentcdodds.com/blog/colocation)

필자는 `/src/utils/queryKeys.ts`와 같은 곳에 전역적으로 모든 **Query Key**를 저장하는 것이 이롭다고 생각하지 않는다. 필자는 다음과 같이 `feature` 폴더와 같은 위치에 있는 각각의 **Query** 옆에 **Query Key**를 작성하여 유지한다.

```
- src
  - features
    - Profile
    - index.tsx
    - queries.ts
  - Todos
    - index.tsx
    - queries.ts
```

이러한 `queries.ts` 파일은 **React Query**와 관련된 모든 것을 포함한다. 필자는 주로 커스텀 훅만 `export` 하기에 실제 **Query** 함수들 뿐만 아니라 **Query** **Key**들도 로컬에 유지할 것이다.

### Always use Array Keys

> **React Query** v4에서부터는 무조건 **Query** **Key**에 배열을 전달하게 되어 있으므로 그냥 **Query** **Key**는 배열로 전달하자!

### Structure

**Query Key**는 **Query Key**들 사이에 필요하다고 생각되는 점까지 세세하게 아주 일반적인 것부터 아주 구체적인 것까지 구조화해야 한다. 다음은 필자가 Todo 리스트에서 필터 가능한 리스트와 상세 페이지에 대한 **Query Key** 구조화 방법이다.

```
['todos', 'list', { filters: 'all' }]
['todos', 'list', { filters: 'done' }]
['todos', 'detail', 1]
['todos', 'detail', 2]
```

위 **Query Key** 구조와 함께라면, 필자는 모든 리스트 또는 모든 상세 데이터와 같이 `['todos']`와 관련된 모든 todo 뿐만 아니라 정확하게 `Key`를 알고 있는 특정 리스트와 같은 단일 데이터 모두를 `invalidate` 할 수 있다. 또한 이 구조와 함께라면 필요할 때마다 모든 리스트를 특정지을 수 있기 때문에 **Mutation**의 응답 데이터를 이용한 업데이트는 훨씬 더 유연해진다.

```tsx
function useUpdateTitle() {
  return useMutation(updateTitle, {
    onSuccess: (newTodo) => {
      // ✅ update the todo detail
      queryClient.setQueryData(['todos', 'detail', newTodo.id], newTodo)

      // ✅ update all the lists that contain this todo
      queryClient.setQueriesData(['todos', 'list'], (previous) =>
      previous.map((todo) => (todo.id === newTodo.id ? newTodo: todo))
    }
  }
}
```

위 코드는 리스트와 상세 데이터의 구조가 많이 다를 경우 제대로 동작하지 않을 수도 있기 때문에, 이를 대신해 모든 리스트를 `invalidate` 할 수도 있다.

```tsx
function useUpdateTitle() {
  return useMutation(updateTitle, {
    onSuccess: (newTodo) => {
      queryClient.setQueryData(['todos', 'detail', newTodo.id], newTodo)

      // ✅ just invalidate all the lists
      queryClient.invalidateQueries(['todos', 'list'])
    }
  }
}
```

만약 현재 어떤 목록이 있는지 알고 있는 경우 (예를 들어, url에서 필터들을 읽어들여 정확한 **Query Key**를 구성할 수 있는 경우) 이 두 메서드를 결합하고 리스트에 `setQueryData`를 호출한 뒤 나머지에 대해선 `invalidate` 할 수도 있다.

```tsx
function useUpdateTitle() {
  // imagine a custom hook that returns the current filters,
  // stored in the url
  const { filters } = useFilterParams();

  return useMutation(updateTitle, {
    onSuccess: (newTodo) => {
      queryClient.setQueryData(['todos', 'detail', newTodo.id], newTodo);

      // ✅ update the list we are currently on instantly
      queryClient.setQueryData(['todos', 'list', { filters }], (previous) =>
        previous.map((todo) => (todo.id === newTodo.id ? newTodo : todo))
      );

      // 🥳 invalidate all the lists, but don't refetch the active one
      queryClient.invalidateQueries({
        queryKey: ['todos', 'list'],
        refetchActive: false,
      });
    },
  });
}
```

**Update**: v4에서 `refetchActive`는 `refetchType`으로 대체됐다. 위 예제에 적용한다면 `refetchType: 'none'`이 될 것이다. 왜냐하면 **아무것도** `refetch` 하지 않길 원하기 때문이다.

### Use Query Key factories

위 예제를 참고했을 때, 우리는 직접 일일이 **Query Key**를 선언하는 경우가 많았다. 이는 잠재적인 오류가 될 수 있을 뿐더러 나중에 수정하기 힘들어 진다. 예를 들어, 만약 **Query Key**에 세분화 단계를 높이려고 하는 경우 수정이 더 힘들어질 것이다.

그렇기에 필자는 기능별 **Query Key factory**를 추천한다. 이는 `Entity`들과 함께인 단순한 객체이며 함수들은 **Query Key**를 생산하는 함수들을 커스텀 훅을 통해 사용할 수 있다. 위 예제의 **Query Key** 구조를 이에 적용하면 다음과 같다.

```tsx
const todoKeys = {
  all: ['todos'] as const,
  lists: () => [...todoKeys.all, 'list'] as const,
  list: (filters: string) => [...todoKeys.lists(), { filters }] as const,
  details: () => [...todoKeys.all, 'detail'] as const,
  detail: (id: number) => [...todoKeys.details(), id] as const,
};
```

This gives me a lot of flexibility, as each level builds on top of another, but is still independently accessible:

```tsx
// 🕺 remove everything related to the todos feature
queryClient.removeQueries(todoKeys.all);

// 🚀 invalidate all the lists
queryClient.invalidateQueries(todoKeys.lists());

// 🙌 prefetch a single todo
queryClient.prefetchQueries(todoKeys.detail(id), () => fetchTodo(id));
```
