# #2 React Query Data Transformations

> 원본 글  
> https://tkdodo.eu/blog/react-query-data-transformations

**Series**

- [#1 Practical React Query](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%231_Practical_React_Query.md)
- #2 React Query Data Transformations (현재)
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
- [#15 React Query FAQs](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%2315_React_Query_FAQs.md)

**목차**

- [#2 React Query Data Transformations](#2-react-query-data-transformations)
  - [Data Transformation](#data-transformation)
  - [0. On the backend](#0-on-the-backend)
  - [1. In the queryFn](#1-in-the-queryfn)
  - [2. In the render function](#2-in-the-render-function)
  - [3. using the select option](#3-using-the-select-option)

아래 문서는 가장 흔하고 중요한 과업 중 하나인 `Data Transformation`에 대해 설명합니다.

## Data Transformation

하나 짚고 넘어가보자. 대부분의 사람들은 여전히 `REST API`를 사용하고 있다. 즉, **GraphQL**을 사용하고 있지 않다. 혹시나 **GraphQL** 대신 `REST API`를 잘 사용하고 있다면 스스로를 칭찬해주자. 왜냐하면 데이터 요청하기 위한 REST API에 대한 설계를 그만큼 잘 했다는 뜻이기 때문이다.

우리가 REST와 함께 무언가를 작업하는 이상 우리는 백엔드가 반환하는 응답에 제약을 받을 수 밖에 없다. 그래서 우리가 **React Query**를 사용할 때 데이터를 변환하기 가장 좋은 방법과 방법을 사용하는 위치는 어디일까?

> It depends.
> — Every developer, always

그러하다 ..

여기선 각각의 장단점이 있는 데이터를 변환할 수 있는 위치에 대한 3+1 접근 방법에 대해 설명한다.

## 0. On the backend

여유가 된다면, 이 방법은 제가 가장 좋아하는 접근법이다. (블로그글 저자가 좋아하는)

만약 백엔드의 응답 데이터의 구조가 우리가 원하는 구조로 온다면 여기서 우리가 더 해줘야할 것은 없다. 여러 경우(ex. public REST API를 사용하는 경우 등)에서 이게 비현실적으로 들릴 수도 있겠지만 엔터프라이즈 애플리케이션에서도 충분히 가능한 방법이다.

만약에 백엔드를 제어하고 있으며 정확한 use-case에 맞게 데이터를 반환하는 endpoint가 있는 경우 예상한 방식으로 데이터를 전달하는 것을 택한다.

> If you are in control of the backend and have an endpoint that returns data for your exact use-case, prefer to deliver the data the way you expect it.
> ⇒ 원문은 이건데 .. 백엔드를 건드릴 수 있고 생각하고 있는 use-case에 맞는 데이터를 반환해주는 API를 갖고 있다면 기존에 생각했던 대로(보내는 데이터의 구조 ?) 백엔드에 데이터를 전달하는 방법을 선호한다는 말인듯 .. 하다.

🟢  - 프론트엔드에서 추가적인 작업이 필요없음

🔴  - 항상 가능한 방법은 아님

## 1. In the queryFn

`queryFn은` `useQuery` 훅에 전달하는 함수이다. 이는 `Promise를` 반환하고 해당 함수의 결과는 `queryCache에` 저장된다. 그러나 결과가 저장된다는 말이 곧 백엔드에서 제공하는 구조로 무조건 저장해야 한다는 뜻은 아니다. 우리는 이 데이터들을 캐시에 저장하기 전에 변환할 수 있다.

```tsx
const fetchTodos = async (): Promise<Todos> => {
  const response = await axios.get('todos');
  const data: Todos = response.data;

  return data.map((todo) => todo.name.toUpperCase());
};

export const useTodosQuery = () => useQuery(['todos'], fetchTodos);
```

위와 같은 변환 작업을 거친 후에 프론트엔드에서 마치 백엔드에서 애초에 이러한 형태로 데이터가 왔던 것처럼 해당 데이터 구조를 가지고 작업을 진행할 수 있다. 이러면 이름이 대문자가 아닌 todo로 작업할 일은 없게 된다. 또한 원본 객체의 구조에 접근할 일도 없어진다. 만약 react-query-devtools를 보고 있다면, 변환된 구조를 볼 수 있을 것이다. 만약 네트워크 탭을 보고 있다면, 원본 객체의 구조를 볼 수 있을 것이다. 헷갈릴 수도 있으니 잘 기억해서 사용하자.

다만, 여기서 **React Query**가 할 수 있는 최적화는 없다. 매번 패치가 실행됐을 때마다 위와 같은 변환 작업이 실행될 것이기 때문이다. 만약 이러한 변환 작업의 비용이 높다면 다른 대안을 고민해보자.

> Some companies also have a shared api layer that abstracts data fetching

몇몇 회사들에서 데이터 패칭을 추상화하는(미들웨어 같이 중간 다리가 하나 계층으로 존재할 수도 있다는 말일듯? 추상화 작업을 발생시키는 ..) 공유 API 계층이 존재할 수도 있기 때문에 이러한 변환 작업을 위해 해당 계층에 접근하지 못할 수도 있다.

🟢 very "close to the backend" in terms of co-location

`co-location` 측면이라는 말을 조금 풀어보면 결국 `co-location`이 예를 들면 데이터 센터 장소를 대여하는 서비스를 말하기 때문에 백엔드로부터 받은 데이터를 변환하는 공간을 만들어서 프론트엔드에서 사용할 수 있게 만드므로 이러한 변환 작업하는 공간을 만든다. 대여한다. 라는 의미에선 백엔드에 근접하다 뭐 이런 말인듯?

🟡 캐시에는 변환된 구조가 저장되기 때문에, 원본 객체 구조에 접근할 수 없다.

🔴 매 패치마다 작업이 발생한다.

🔴 자유롭게 수정할 수 없는 공유 API 레이어가 있는 경우 사용이 불가능하다.

## 2. In the render function

Part 1에 따르면, 만약 커스텀 훅을 만든다면, 쉽게 변환 작업을 추가할 수 있다.

```tsx
const fetchTodos = async (): Promise<Todos> => {
  const response = await axios.get('todos');
  return response.data;
};

export const useTodosQuery = () => {
  const queryInfo = useQuery(['todos'], fetchTodos);

  return {
    ...queryInfo,
    data: queryInfo.data?.map((todo) => todo.name.toUpperCase()),
  };
};
```

위 예제는 매 패치 함수가 실행될 때 뿐만 아니라 매 렌더링마다 실행된다(해당 렌더링에 데이터 패칭이 포함되지 않음에도 불구하고). 이 자체로 문제가 되진 않지만, 문제가 있다면 `useMemo`를 통해 최적화를 진행할 수 있다. 의존성 배열을 정의할 때 최대한 좁게 해야한다는 걸 인지하자. `queryInfo` 내의 `data`는 `data`가 진짜로 바뀌지(변환 작업을 다시 실행하고 싶은 경우) 않는 한 참조적으로 안정적이지만 `queryInfo`는 그렇지 않다. 만약 `queryInfo`를 의존성 배열에 추가할 경우 변환 작업은 매 렌더링마다 다시 실행될 것이다.

```tsx
export const useTodosQuery = () => {
  const queryInfo = useQuery(['todos'], fetchTodos);

  return {
    ...queryInfo,
    // 🚨 don't do this - the useMemo does nothing at all here!
    data: React.useMemo(() => queryInfo.data?.map((todo) => todo.name.toUpperCase()), [queryInfo]),

    // ✅ correctly memoizes by queryInfo.data
    data: React.useMemo(() => queryInfo.data?.map((todo) => todo.name.toUpperCase()), [queryInfo.data]),
  };
};
```

특히 만약 커스텀 훅에 데이터 변환 작업과 함께 특정 로직을 추가하고 싶다면, 이건 좋은 옵션이다. `data` 객체 자체는 잠재적으로 `undefined`일 수 있으니, 이를 작업할 때 `optional chaning`을 사용하도록 하자.

🟢 `useMemo`를 통한 최적화 가능

🟡 `devtools`를 통해서 정확한 구조는 관측 불가

🔴 문법 자체는 조금 복잡도가 상승

🔴 `data`에 `undefined`가 할당될 가능성이 발생

## 3. using the select option

v3에서 소개된 빌트인 `selector`로 데이터를 변환하는 작업에 주로 사용된다.

```tsx
export const useTodosQuery = () => {
  useQuery(['todos'], fetchTodos, {
    select: (data) => data.map((todo) => todo.name.toUpperCase()),
})
```

`selector`는 데이터가 있을 때만 호출될 것이기에 `undefined` 상태에 대해선 걱정할 필요가 없다. 위 예제와 같은 `Selector`는 매 렌더링마다 실행될 것인데 이는 함수가 새로 만들어졌기 때문이다(inline function). 만약 변환 작업의 비용이 높다면, `useCallback`을 이용하여 `memoization`을 진행하거나 함수 자체를 추출하여 함수 자체가 한번만 정의되도록 만들어야 한다.

```tsx
const tarnsformTodoNames = (data: Todos) =>
  data.map((todo) => todo.name.toUppercase())

export const useTodosQuery = () =>
  useQuery(['todos'], fetchTodos, {
  // ✅ use a stable function reference
  select: transformTodoNames,
})

export const useTodosQuery = () =>
  useQuery(['todos'], fetchTodos, {
  // ✅ memoizes with useCallback
  select: React.**useCallback**(
    (data: Todos) => data.map((todo) => todo.name.toUpperCase()),
    []
  ),
})
```

또한, `select` 옵션을 이용하여 데이터의 일부부만 구독하도록 만들 수도 있다. 이게 바로 이 접근법을 특별하게 만든다. 다음 예제를 한번 살펴보자.

```tsx
export const useTodosQuery = (select) => useQuery(['todos'], fetchTodos, { select });

export const useTodosCount = () => useTodosQuery((data) => data.length);
export const useTodo = (id) => useTodosQuery((data) => data.find((todo) => todo.id === id));
```

위 예제를 보면, 사용자가 원하는 `selector`를 `useTodosQuery`에 전달하여 API와 같은 `useSelector`를 만들었다. 커스텀 훅은 여전히 이전과 같이 동작할 것이며 만약 `select` 파라미터를 전달하지 않아서 `undefined`로 설정된다면 `data` 객체 전체가 반환될 것이다.

하지만 만약 `selector`를 인자값으로 전달할 경우 `selector` 함수의 결과만을 구독하게 될 것이다. 이건 꽤나 강력한 기능인데, 이는 우리가 `todo`의 **이름**을 업데이트 하더라도 `useTodosCount`를 통해 `count`를 구독하고 있는 컴포넌트들은 리렌더링이 일어나지 않는다는 것을 의미하기 때문이다. count가 변하지 않았기 때문에 **React Query**는 `observer`들에게 업데이트 사항을 알리지 않는다. (다만 알아두어야 할 것은 여기서의 기술적인 설명은 단순화 됐기 때문에 렌더링 최적화에 대한 부분은 Part 3에서 더 자세히 알아보도록 하자)

🟢 가장 좋은 최적화 방법

🟢 `data` 객체 내부 속성 중 일부만 구독 가능

🟡 `observer` 마다 다른 구조를 가질 수도 있음

`selector`를 전달하여 데이터 객체 중 일부만을 구독하도록 만들었으니 `observer` 마다 다른 데이터 구조를 구독하고 있을 확률이 높다.

🟡 구조적인 공유를 두 번 수행하고 있다. (이에 대한 더 자세한 사항은 Part 3에서 진행)
