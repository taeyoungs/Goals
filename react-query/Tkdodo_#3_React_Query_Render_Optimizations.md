# #3 React Query Render Optimizations

> 원본 글  
> https://tkdodo.eu/blog/react-query-render-optimizations

**Series**

- [#1 Practical React Query](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%231_Practical_React_Query.md)
- [#2 React Query Data Transformations](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%232_React_Query_Data_Transformations.md)
- #3 React Query Render Optimizations (현재)
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
- [#15 React Query FAQs](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%2315_React_Query_FAQs.md)

**목차**

- [#3 React Query Render Optimizations](#3-react-query-render-optimizations)
    - [개요](#개요)
  - [isFetching transition](#isfetching-transition)
  - [notifyOnChangeProps](#notifyonchangeprops)
  - [Staying in sync](#staying-in-sync)
  - [Tracked Queries](#tracked-queries)
  - [Structural sharing](#structural-sharing)

### 개요

렌더링 최적화는 어느 애플리케이션에서나 고급 개념에 해당하는 부분이다. **React Query**는 기본적으로 이미 좋은 최적화를 제공하기 때문에 대부분의 경우 최적화에 대한 추가적인 작업이 필요없다. `불필요한 리렌더링`이라는 주제는 많은 사람들이 주의 깊게 살피는 주제이기 때문에 여기서 해당 주제를 다뤄보고자 한다. 하지만 다시 한번 말하지만 대부분의 앱에서 렌더링 최적화는 생각보다 신경쓰지 않아도 되는 부분이다. 리렌더링은 좋은 것이다. 이건 우리의 애플리케이션을 최신 상태로 만들어 준다. 필자는 `있어야만 하는데 없는 누락된 렌더링`을 할바에 매일 `불필요한 리렌더링`을 할 것이다. 해당 주제에 대해 더 많은 정보를 얻고 싶다면 다음의 포스팅을 살펴보자.

- [https://kentcdodds.com/blog/fix-the-slow-render-before-you-fix-the-re-render](https://kentcdodds.com/blog/fix-the-slow-render-before-you-fix-the-re-render)
- [https://medium.com/@ryanflorence/react-inline-functions-and-performance-bdff784f5578](https://medium.com/@ryanflorence/react-inline-functions-and-performance-bdff784f5578)

---

필자는 #2의 select 옵션에서 이미 렌더링 최적화에 대해 조금이나마 다뤘다. 그러나 데이터가 변경되지 않았음에도 불구하고 컴포넌트를 두 번 리렌더링하는 이유에 대한 질문을 필자가 가장 많이 받았기 때문에 이에 대해 깊게 한번 설명해보려고 한다.

## isFetching transition

이전 예제에서 todos의 길이가 변경될 때만 컴포넌트가 리렌더링 된다고 말했던 때 말하지 않은 부분이 존재한다.

```tsx
export const useTodosQuery = (select) => useQuery(['todos'], fetchTodos, { select });
export const useTodosCount = () => useTodosQuery((data) => data.length);

function TodosCount() {
  const todosCount = useTodosCount();

  return <div>{todosCount.data}</div>;
}
```

매번 **background refetch**를 발생시킬 때, 위 컴포넌트는 다음의 ₩ 정보를 가지고 두 번 리렌더링될 것이다.

```json
{ status: 'success', data: 2, isFetching: true }
{ status: 'success', data: 2, isFetching: false }
```

이는 **React Query**가 매 쿼리마다 만들어내는 많은 양의 메타 데이터 때문이며 `isFetching`은 그 메타 데이터들 중 하나이다. 이 플래그 변수는 네트워크 요청이 진행 중일 때는 항상 `true` 값일 것이다. 이것은 만약 우리가 **background loading indicator**를 보여주길 원할 때 꽤 유용하다. 하지만 이러한 작업을 원치 않는다면 불필요하기도 하다.

## notifyOnChangeProps

이러한 때를 위하여 **React Query**를 `notifyOnChangeProps` 옵션을 가지고 있다. 이를 이용하여 R**eact Query**에게 이러한 변화를 알리는 `Observer`를 설정할 수 있다. `Observer`는 오직 이러한 `props` 중 선택한 속성에 대해 변화가 있을 때만 **React Query**에게 알린다. 이 옵션을 `['data']`로 설정하면 우리가 원했던 최적화 버전을 찾을 수 있을 것이다. (`data` 속성이 변할 때만 리렌더링이 발생할 것이다)

```tsx
export const useTodosQuery = (select, notifyOnChangeProps) =>
  useQuery(['todos'], fetchTodos, { select, notifyOnChangeProps });
export const useTodosCount = () => useTodosQuery((data) => data.length, ['data']);
```

You can see this in action in the [optimistic-updates-typescript](https://github.com/tannerlinsley/react-query/blob/9023b0d1f01567161a8c13da5d8d551a324d6c23/examples/optimistic-updates-typescript/pages/index.tsx#L35-L48) example in the docs.

## Staying in sync

위 코드는 잘 동작할지 몰라도 동기화는 쉽지 않을 수 있다. 만약 우리가 `error` 객체에도 반응하길 원한다면? 또는 우리가 `isLoading` 플래그를 사용하기 시작한다면? 우리는 계속해서 컴포넌트에서 사용하고자 하는 필드들과 `notifyOnChangeProps` 리스트가 동기화되도록 유지해야한다. 만약 우리가 이 작업을 까먹기라도 한다면 React Quyery는 `data` 속성만 지켜보고 있을 것이고 우리가 보여주고자 하는 `error`가 발생했다고 하더라도 컴포넌트는 리렌더링하고 이전 상태로 남아있을 것이다. 이는 만약 우리가 커스텀 훅에서 이를 하드 코딩한 상태라면 더 문제가 되는데 커스텀 훅 자체는 컴포넌트에서 실제로 무엇을 사용하게 될지 모르기 때문이다.

```tsx
export const useTodosCount = () => useTodosQuery((data) => data.length, ['data']);

function TodosCount() {
  // 🚨 we are using error, but we are not getting notified if error changes!
  const { error, data } = useTodosCount();

  return (
    <div>
      {error ? error : null}
      {data ? data : null}
    </div>
  );
}
```

이 포스팅 시작 부분에서 추가적으로 설명했듯이, 필자는 위 같은 상황이 가끔씩 발생하는 불필요한 리렌더링보다 더 문제라고 생각한다. 물론, 우리가 커스텀 훅에 옵션을 전달하는 방식으로 작성할 수도 있지만 여전히 수동적이고 상용구처럼 느껴진다. 이러한 작업을 자동으로 해주는 방법은 없을까?

## Tracked Queries

> React Query v4에서는 `tracked`가 기본 설정이며 이를 해제하고 싶다면 `notifyOnChangeProps`를 `all`로 주면 된다.

만약 우리가 `notifyOnChangeProps`에 `tracked`라고 설정하면, React Query는 렌더링 중에 사용 중인 필드를 추적할 것이고 이를 기반으로 리스트를 만들어 내서 사용할 것이다. 이 방법은 우리가 이러한 리스트에 대해 고민할 필요도 없게 만들어 주는 점을 제외하면 리스트를 직접 명시하는 것과 정확히 동일한 최적화를 만들어 낼 것이다. 다음과 같이 전역적으로 모든 쿼리에 대해서 적용할 수도 있다.

```tsx
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      notifyOnChangePRops: 'tracked',
    },
  },
});

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <Example />
    </QueryClientProvider>
  );
}
```

이와 함께라면 우리는 리렌더링이 다시 발생하는 것에 대해서 전혀 생각하지 않아도 된다. 물론, 렌더링 시 사용되는 속성만 추적하는 것은 과부화를 발생시키기 때문에 신중하게 사용해야만 한다. 또한 쿼리 추적에는 몇 가지 제한 사항이 존재하기에 이 기능은 옵트인 기능이다.

- 만약 `spread` 연산자를 이용한 객체 구조 분해 할당을 사용하고 있다면, 모든 필드를 효율적으로 지켜볼 수 있다. 일반적은 구조 분해는 괜찮지만 다음과 같이 해서는 안된다.

```tsx
// 🚨 will track all fields
const { isLoading, ...queryInfo } = useQuery(...)

// ✅ this is totally fine
const { isLoading, data } = useQuery(...)
```

- 쿼리 추적은 오직 렌더링 중에만 동작한다. 만약 effects 중에 필드들에게 접근하고 싶다면, 이는 추적되지 않을 것이다. This is quite the edge case though because of dependency arrays

```tsx
const queryInfo = useQuery(...)

// 🚨 will not corectly track data
React.useEffect(() => {
  console.log(queryInfo.data)
})

// ✅ fine because the dependency array is accessed during render
React.useEffect(() => {
  console.log(queryInfo.data)
}, [queryInfo.data])
```

- 쿼리 추적은 매 렌더마다 초기화되지 않는다. 따라서, 필드를 한번 추적하면 `Observer`는 평생 해당 필드를 추적하게 된다.

```tsx
const queryInfo = useQuery(...)

if (someCondition()) {
  // 🟡 we will track the data field if someCondition was true
  // in any previous render cycle
  return <div>{queryInfo.data}</div>
}
```

## Structural sharing

**React Query**에 기본적으로 활성화되어 있는 것 중 다르지만 조금은 덜 중요한 렌더링 최적화는 `structural sharing`이다. 이 기능을 사용하면 모든 수준에서 데이터에 대한 참조값을 유지할 수 있다. 예를 들어, 다음과 같은 데이터 구조를 가지고 있다고 가정해보자.

```json
[
  { id: 1, name: 'Learn React', status: 'active' },
  { id: 2, name: 'Learn React Query', status: 'todo' },
];
```

이제 첫 번째 todo를 `done` 상태로 전환하고 백그라운드 리패치를 발생시켰다고 가정해보자.

우리는 백엔드로부터 완전히 새로운 json 데이터를 가져오게 될 것이다.

```json
[
  -{ id: 1, name: 'Learn React', status: 'active' },
  +{ id: 1, name: 'Learn React', status: 'done' },
  { id: 2, name: 'Learn React Query', status: 'todo' },
];
```

이제 **React Query**는 이전 상태값과 새로운 상태값을 비교하고 최대한 많은 이전 상태값을 유지하려는 시도를 할 것이다. 위 예제에서, 우리가 todo를 업데이트 했기 때문에 todos 배열은 새로운 것이 됐다. id 1을 가진 객체는 새로운 것이 됐지만 id 2를 가진 객체는 이전 상태값 중 하나와 동일한 참조를 갖는다. **React Query**는 변경되지 않은 참조에 대해서는 이를 복사하여 새로운 결과값에 집어 넣는다.

이러한 작업은 selectors for partial subscriptions을 사용할 때 매우 편리하다.

```tsx
// ✅ will only re-render if _something_ within todo with id:2 changes
// thanks to structural sharing
const { data } = useTodo(2);
```

앞서 언급했듯이, `selector`에 대해 **structural sharing**은 두 번 발생할 것이다. queryFn에서 반환된 결과값에 대해 한 번 변경된 사항이 있는지 확인한 다음 `selector` 함수의 결과에 대해 한 번 더 확인할 것이다. 몇 가지 상황에서 특히나 매우 많은 데이터셋을 다루는 때에 **structural sharing**은 병목 현상을 만들 수 있으며 이는 또한 `JSON` 직렬화가 가능한 데이터에 대해서만 동작한다. 만약 이러한 최적화 과정이 필요없다면 어느 쿼리에나`structuralSharing: false`을 설정해서 이를 꺼버릴 수 있다.

Have a look at the [replaceEqualDeep tests](https://github.com/tannerlinsley/react-query/blob/80cecef22c3e088d6cd9f8fbc5cd9e2c0aab962f/src/core/tests/utils.test.tsx#L97-L304) if you want to learn more about what happens under the hood.
