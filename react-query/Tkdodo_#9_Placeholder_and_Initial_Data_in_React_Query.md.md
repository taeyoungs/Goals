# #9 Placeholder and Initial Data in React Query

> 원본 글  
> https://tkdodo.eu/blog/placeholder-and-initial-data-in-react-query

**Series**

- [#1 Practical React Query](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%231_Practical_React_Query.md)
- [#2 React Query Data Transformations](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%232_React_Query_Data_Transformations.md)
- [#3 React Query Render Optimizations](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%233_React_Query_Render_Optimizations.md)
- [#4 Status Checks in React Query](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%234_Status_Checks_in_React_Query.md)
- [#6 React Query and TypeScript](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%236_React_Query_and_TypeScript.md)
- [#7 Using WebSockets with React Query](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%237_Using_WebSockets_with_React_Query.md)
- [#8 Effective React Query Keys](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%238_Effective_React_Query_Keys.md)
- [#8a Leveraging the Query Function Context](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%238a_Leveraging_the_Query_Function_Context.md)
- #9 Placeholder and Initial Data in React Query (현재)
- #10 React Query as a State Manager

**목차**

- [#9 Placeholder and Initial Data in React Query](#9-placeholder-and-initial-data-in-react-query)
  - [개요](#개요)
- [Similarities](#similarities)
  - [On cache level](#on-cache-level)
  - [On observer level](#on-observer-level)
- [Differences](#differences)
  - [Persistence](#persistence)
  - [Background refetches](#background-refetches)
  - [Error transitions](#error-transitions)
    - [InitialData](#initialdata)
    - [PlaceholderData](#placeholderdata)
- [When to use what](#when-to-use-what)

### 개요

이 포스팅은 **React Query**를 사용할 때 사용자 경험을 향상시킬 수 있는 모든 방법에 대한 이야기이다. 대부분의 경우, 우리(또는 사용자들)는 성가신 **Loading Spinner**를 좋아하지 않는다. **Loading Spinner**는 때때로 필요하긴 하지만 우리는 여전히 가능하다면 **Loading Spinner** 사용을 피하고 싶어 한다.

**React Query**는 이미 많은 상황에서 이러한 **Loading Spinner**를 제거하기 위한 툴을 제공하고 있다. 우리는 백그라운드에서 업데이트가 발생하는 동안 **Cache**로부터 `stale` 데이터를 가져올 수 있고 만약 특정 데이터가 필요하다는 사실을 미리 알 경우 데이터를 `prefetch` 할 수도 있으며 심지어 **QueryKey**가 변경됐을 때 심한 로딩 상태를 피하기 위해 이전 데이터를 유지할 수도 있다.

다른 방법으로는 잠재적으로 사용될 것이라 생각되는 **use-case** 데이터를 **Cache**에 미리 동기적으로 채워넣는 방법이다. 이러한 방법을 위해 **React Query**에서는 비슷하지만 다른 두 가지 접근 방법을 제공한다.

두 접근 방법의 차이점과 어느 상황에 어떤 접근 방법이 더 적합한지에 대해 알아보기 전에 두 방법이 가진 공통점에 대해 먼저 알아보자.

# Similarities

이미 암시했듯이, 두 접근 방법 모두 동기적으로 사용할 수 있는 데이터에 대해 **Cache**에 미리 해당 데이터를 채울 수 있는 방법을 제공한다. 이는 둘 중 하나의 접근 방법이라도 사용되는 상황일 경우 우리의 **Query**는 `loading` 상태가 아니라 바로 `success` 상태가 된다는 것을 의미한다. 또한 값을 계산하는 데 비용이 많이 드는 경우 값 또는 값을 반환하는 함수일 수 있다.

```tsx
function Component() {
  // ✅ status will be success even if we have not yet fetched data
  const { data, status } = useQuery(['number'], fetchNumber, {
    placeholderData: 23,
  });

  // ✅ same goes for initialData
  const { data, status } = useQuery(['number'], fetchNumber, {
    initialData: () => 42,
  });
}
```

마지막으로 **Cache**에 이미 데이터가 있는 경우 둘 다 효과가 없다. 그러면 두 방법을 사용하는데 있어서 차이점은 무엇일까? 이를 이해하려면 **React Query**의 옵션이 어떻게(그리고 어떤 `"level"`에서) 작동하는지 간략하게 살펴봐야 한다.

## On cache level

각 **QueryKey**에는 단 하나의 **Cache** 항목이 존재한다. 이러한 점은 **React Query**를 훌륭하게 만들어 주는 이유 중 하나인데 애플리케이션 내에서 동일한 데이터를 **전역적으로** 공유할 수 있게 하기 때문이다.

우리가 `useQuery`에 제공하는 몇 가지 옵션은 **Cache** 항목에 영향을 주는데 대표적으로 `statleTime`과 `cacheTime`이 그러하다. 각 **QueryKey**에 하나의 **Cache** 항목이 존재하기 때문에, 이러한 옵션들을 통해 캐시 항목이 `stale` 상태가 되어야 하는 시점 또는 가비지 컬렉팅 시점에 대해 명시할 수 있다.

## On observer level

**React Query**에서 말하는 `observer`란 일반적으로 단일 캐시 항목에 대해 생성된 `subscription`(구독)을 말한다. `observer`는 해당 캐시 항목의 변화를 지켜보고 있다가 변경 사항이 발생할 때마다 이에 대한 알림을 받는다.

`observer`를 생성하기 위한 기본적인 방법은 `useQuery`를 호출하는 것이다. `useQuery`를 호출할 때마다 우리는 `observer`를 생성하며, 이로 인해 컴포넌트는 데이터가 변경될 때 리렌더링을 될 것이다. 당연히도 이는 동일한 캐시 항목을 여러 `observer`가 지켜보고 있을 수도 있다는 것을 의미한다.

그건 그렇고, **React Query Devtools**의 **QueryKey** 왼쪽에 있는 숫자로 특정 **Query**에 몇 명의 `observer`가 있는지 확인할 수 있다.

![3명의 observer가 있는 상태](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3243f756-1961-44a3-bc6d-9b064cb515a7/Untitled.png)

3명의 observer가 있는 상태

**observer level**에서 동작하는 옵션은 `select` 또는 `keepPreviousData`가 있다. 사실 `data transformations`에 `select` 옵션이 좋은 이유는 단일 캐시 항목을 지켜보면서 다른 컴포넌트의 데이터 일부를 구독할 수 있는 기능 때문이다.

# Differences

`initialData`는 **cache level**에서 동작하는 반면에 `placeholderData`는 **observer level**에서 동작한다. 이러한 차이에는 몇 가지 의미가 있다.

## Persistence

우선, `initialData`는 **Cache**에서 유지된다. `initialData`는 **React Query**에게 말해주는 방법 중 하나인데 이게 무엇을 의미하냐면 **\*React Query**에게 이미 우리에게 **use-case**를 위한 데이터가 있으며, 이 데이터는 마치 백엔드에서 `fetch` 해온 것 마냥 좋은 데이터다.\* 와 같이 알리는 것이다. 이는 **cache level**에서 동작하기 때문에 `initialData`는 오직 하나만 있을 것이며 데이터는 캐시 항목이 생성되는 즉시 저장될 것이다. (여기서 캐시 항목이 생성될 때는 첫 `observer`가 `mount` 될 때를 의미한다) 만약 이때 다른 `initialData`와 함께 두 번째 `observer`를 `mount` 시킨다면, 아무 일도 일어나지 않을 것이다.

반면에 `PlaceholderData`는 절대 **Cache**에 유지되지 않는다. 필자는 `placeholderData`를 **fake-it-till-you-make-it**로 본다. 이는 진짜 데이터가 아니다. (마치 더미 데이터와 같은)

> **fake-it-till-you-make-it**: 될 때까지 그런 척을 하는 것을 말한다.
>
> 여기서는 데이터 패칭이 완료될 때까지 패칭해서 가져온 데이터인 척을 하는 **가짜 데이터**를 의미하는 것 같다.

**React Query**는 진짜 데이터를 패칭해서 가져오기 전까지 보여줄 가짜 데이터를 제공해준다. **observer level**에서 동작하기 때문에 이론적으로는 심지어 다른 컴포넌트에서 다른 `placeholderData` 가질 수도 있다.

## Background refetches

`placeholderData`를 사용하는 경우, `observer`가 처음 `mount` 될 때 항상 **background refetch**가 일어날 것이다. `placeholderData`로 만든 데이터는 진짜가 아니기 때문에 **React Query**는 진짜 데이터를 패칭해서 가져다 줄 것이다. 이런 일이 발생하는 동안 우리는 `useQuery`로부터 `isPlaceholderData` 플래그도 얻을 수 있게 된다. 우리는 `isPlaceholderData` 플래그를 사용하여 사용자로 하여금 사용자들이 보고 있는 데이터가 사실은 `placeholderData`임을 시각적으로 암시할 수도 있다. 해당 플래그는 진짜 데이터를 가져오는 즉시 `false`로 변환될 것이다.

반면에 `initialData`는 실제로 우리의 **Cache**에 저장된 유효한 데이터로 간주되기 때문에 `staleTime`에 영향을 받는다. 만약 `staleTime`을 `0`(`staleTime`의 기본값은 `0`이다)으로 만든다면, 우리는 여전히 **background refetch**가 발생되는 것을 볼 수 있을 것이다.

하지만 만약 우리가 **Query**에 `staleTime`(ex. 30초)을 다르게 설정한다면, **React Query**는 `initialData`를 보고 다음과 같이 동작할 것이다.

> _Oh, I'm getting fresh and new data here synchronously, thank you very much, now I don't need to go to the backend because this data is good for 30 seconds._
> — React Query when it sees *initialData* and *staleTime*
>
> `Cache`에 저장되어 유효한 데이터로 간주되는 `initialData`가 존재하기 때문에 `staleTime`이 0이 아닌 경우 **React Query**는 `staleTime` 동안 백엔드에 네트워크 요청을 보내지 않아도 된다고 인식한다.

만약 위와 같은 동작을 원하지 않는다면, **Query**에 `initialDataUpdateAt` 옵션을 제공하면 된다. 이 옵션을 통해 **React Query**에게 `initialData`가 생성될 때 **background refetch**의 발생도 같이 고려하라고 알려줄 수 있다. 이는 **dateUpdateAt timestamp**를 사용하여 기존에 존재하는 캐시 항목의 `initialData`로 사용하려는 경우 매우 유용하다.

```tsx
const useTodo = (id) => {
  const queryClient = useQueryClient();

  return useQuery(['todo', id], () => fetchTodo(id), {
    staleTime: 30 * 1000,
    initialData: () => queryClient.getQueryData(['todo', 'list'])?.find((todo) => todo.id === id),
    initialDataUpdatedAt: () =>
      // ✅ will refetch in the background if our list query data is older
      // than the provided staleTime (30 seconds)
      queryClient.getQueryState(['todo', 'list'])?.dataUpdateAt,
  });
};
```

[Initial Query Data | TanStack Query Docs](https://tanstack.com/query/v4/docs/guides/initial-query-data#staletime-and-initialdataupdatedat)

> 위 링크에 더 자세한 내용이 있지만 정리하자면, 우선 `initialData`는 **cache level**에서 동작하기 때문에 `staleTime`에 영향을 받는다. `staleTime`의 기본 설정(`staleTime: 0`)을 이용한다면 매번 새로운 네트워크 요청을 발생시키겠지만 `staleTime`이 `0`이 아니라면 `initialData`를 최신 상태라고 보고 `staleTime` 만큼의 시간이 흐른 후에 추가적인 상호작용이 발생할 경우 네트워크 요청이 이루어 진다.
>
> 만약 `staleTime`이 0이 아닌 상태에서도 네트워크 요청 발생 여부에 관여하고 싶다면 `initialDataUpdatedAt` 속성에 `timestamp` 형태의 값을 주면 해당 속성에 할당된 시간보다 `mount` 됐을 때의 시간이 `staleTime` 이상으로 더 늦는 경우 `stale` 상태라고 판단하고 네트워크 요청을 발생시키게 할 수 있다.
>
> 위 예제에서는 `initialDataUpdatedAt` 속성의 값으로 특정 `Query`의 마지막 업데이트 시간(`dataUpdateAt`)을 부여하고 있는 상황이다. 그래서 마지막 업데이트된 시간에 `staleTime`을 더한 시간보다 현재 시간이 더 늦는 경우 `stale` 상태의 **Query**라 판단하고 새로운 네트워크 요청을 발생시킨다.

## Error transitions

우리가 `initialData` 또는 `placeholderData`를 제공하고 있는 상황에서 **background refetch**가 발생했고 이 요청이 실패했다고 가정해보자. 각 상황에서 어떤 일이 일어날 것 같은가?

### InitialData

`initialData`는 캐시에서 유지되기 때문에, `refetch`로 발생한 에러는 여느 다른 백그라운드 에러와 같이 다뤄진다. **Query**는 에러 상태가 될 테지만 데이터는 여전히 캐시에 존재할 것이다.

### PlaceholderData

`placeholderData`는 가짜 데이터(**네트워크 요청이 완료될 때까지** 그럴듯하게 데이터가 있는 척을 하기 위한)기 때문에, 에러가 발생한 이상 더 이상 가짜 데이터를 만들어내지 않을 것이기에 해당 데이터를 더 이상 볼 수 없다. **Query**는 에러 상태가 될 것이며 데이터는 `undefined`가 될 것이다.

# When to use what

언제나 사용하는 사람의 마음에 달려있다. 필자는 개인적으로 다른 **Query**에 **Query**(초기 캐시 데이터를 말하는 듯?)를 미리 채우고 싶을 경우 `initialData`를 사용하고 나머지 모든 경우에 `placeholderData`를 사용한다.
