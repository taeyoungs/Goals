# #16 React Query meets React Router

> 원본 글  
> https://tkdodo.eu/blog/react-query-meets-react-router

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
- [#15 React Query FAQs](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%2315_React_Query_FAQs.md)
- #16 React Query meets React Router (현재)

**목차**

- [#16 React Query meets React Router](#16-react-query-meets-react-router)
  - [개요](#개요)
- [A router that fetches data](#a-router-that-fetches-data)
  - [It’s not a cache](#its-not-a-cache)
  - [Fetching early](#fetching-early)
  - [Fetching too often](#fetching-too-often)
  - [Querifying the example](#querifying-the-example)
    - [The loader needs access to the QueryClient](#the-loader-needs-access-to-the-queryclient)
    - [getQueryData ?? fetchQuery](#getquerydata--fetchquery)
    - [A TypeScript tip](#a-typescript-tip)
  - [Invalidating in actions](#invalidating-in-actions)
    - [await is the lever](#await-is-the-lever)
  - [Summary](#summary)

## 개요

[Remix](https://remix.run/) is changing the game

**Remix** 내에 있던 데이터 패칭 컨셉을 **React Router v6.4**로 순수 클라이언트 아시드 렌더링 애플리케이션에 안착시켰다. **React Router** 공식 홈페이지에 소개된 튜토리얼에서는 이들이 보여주고자 하는 컨셉이 잘 나와있었고 작지만 기능이 풍부한 애플리케이션을 얼마나 빠르게 만들 수 있는지 잘 설명이 되어 있었다.

**React Router**가 데이터 패칭 기능을 가져오면서 이 기능이 어떻게 **React Query**와 같은 기존 데이터 패칭 및 캐싱 라이브러리들과 어떻게 경쟁 관계를 갖는지 또는 상관 관계가 있는지 자연스레 관심이 갈 것이다. 따라서, 궁금한 점을 바로 파헤쳐 보자.

# A router that fetches data

다시 짚고 넘어가면, **React Router**는 특정 경로에 방문했을 때 호출될 `[loaders](https://reactrouter.com/en/6.4.0/route/loader)`를 정의할 수 있게 해줄 것이다. `route` 컴포넌트 자체에서 `useLoaderData()`를 사용하여 데이터에 접근할 수 있다. 데이터를 업데이트하는 것은 `action` 함수를 호출함으로써 `Form`을 **submit**하는 것 만큼 간단하다. `Action`은 활성화된 모든 `loader`들을 `invalidate`하여 우리가 자동으로 스크린에서 업데이트된 데이터를 볼 수 있게 할 것이다.

만약 위 말들이 `queries` 그리고 `mutations`과 매우 흡사하게 들린다면 그게 맞고 실제로도 그러하다. 그래서 **Remixing React Router** 발표 이후 뜨는 질문은 무엇이냐면

- 특정 경로에서 데이터 패칭을 할 수 있는 **React Query**가 여전히 쓰일 것인지
- 만약 이미 **React Query**를 사용하고 있다면, 우리가 새로운 **React Router** 기능을 사용하길 바라는지 또는 어떻게 사용할 수 있는지

## It’s not a cache

**Tkdodo** 형님의 생각은 두 질문 모두 YES이다. **Remix** 팀 소속 개발자도 트윗으로 대답을 해주기도 했다.

["React Router is not a cache"](https://twitter.com/ryanflorence/status/1561731634419773447?ref_src=twsrc%5Etfw%7Ctwcamp%5Etweetembed%7Ctwterm%5E1561731634419773447%7Ctwgr%5Eb1667e259271a0497bf7b54ec3057972be7e7161%7Ctwcon%5Es1_&ref_url=https%3A%2F%2Ftkdodo.eu%2Fblog%2Freact-query-meets-react-router)

"**가능한 한 빨리**" 데이터를 패칭하는 것은 최고의 사용자 경험을 제공하기 위한 중요한 컨셉 중 하나이다.

**Next.js** 또는 **Remix**와 같은 풀스택 프레임워크에서 이러한 과정을 서버로 옮긴 것은 서버가 가장 빠른 시점의 진입점이기 때문이다. 클라이언트 사이드 렌더링 애플리케이션에서는 그러한 방법이 없다.

## Fetching early

우리가 주로 하는 것 중 하나는 데이터가 처음 필요할 때 컴포넌트가 마운트 되는 시점에 데이터를 패칭하는 것이다. 이건 사용자 경험 측면에서 좋은 방법은 아니다. 왜냐하면 우리가 데이터를 처음 가져오는 한 사용자에게 로딩 스피너를 보여주기 때문이다. [Prefetching](https://tanstack.com/query/v4/docs/guides/prefetching)이 도움이 될 수도 있지만 이는 후속 네비게이션에 한정되며 이는 이동하려는 모든 경로에 일일히 작업을 해줘야 한다.

그러나 `router`는 우리가 방문하고자 하는 페이지를 항상 알고 있는 첫 번째 컴포넌트이며 이제 `loader`가 있기 때문에 해당 페이지에서 렌더링해야 하는 데이터가 무엇인지 알기까지 한다. 이는 페이지에 처음 방문할 때는 상당히 좋지만 문제는 `loader`가 페이지를 매번 방문할 때마다 호출된다는 점이다. `router`는 캐시가 없기 때문에 우리가 따로 무언가를 해주지 않는 이상 서버에 새로운 네트워크 요청을 한다.

예를 들어 연락처 목록이 있다고 가정해보자. 그리고 만약 연락처 중 하나를 클릭한 경우 해당 연락처의 상세 정보가 보여진다.

`src/routes/contacts.jsx`

```jsx
import { useLoaderData } from 'react-router-dom';
import { getContact } from '../contacts';

// ⬇️ this is the loader for the detail route
export async function loader({ params }) {
  return getContact(params.contactId);
}

export default function Contact() {
  // ⬇️ this gives you data from the loader
  const contact = useLoaderData();
  // render some jsx
}
```

`src/main.jsx`

```jsx
import Contact, { loader as contactLoader } from './routes/contact';

const router = createBrowserRouter([
  {
    path: '/',
    element: <Root />,
    children: [
      {
        path: 'contacts',
        element: <Contacts />,
        children: [
          {
            path: 'contacts/:contactId',
            element: <Contact />,
            // ⬇️ this is the loader for the detail route.
            loader: contactLoader,
          },
        ],
      },
    ],
  },
]);
```

만약 `contacts/1`로 이동했을 경우 해당 연락처(`id: 1`)에 대한 데이터는 컴포넌트가 렌더링되기 전에 데이터 패칭될 것이다. 연락처를 보여주려고 하는 시점에는 `useLoaderData`에서 데이터를 쉽게 읽을 수 있을 것이다. 이는 사용자 경험(UX)뿐만 아니라 연관된 데이터 패칭과 덴더링에 대한 개발자 경험까지 향상시키기 때문에 굉장한 기능이라고 말할 수 있다. I love it. 🥰

## Fetching too often

캐시가 없다는 큰 단점은 우리가 `contacts/2`에서 `contacts/1`으로 다시 돌아갈 때 나타난다. 만약 **React Query**를 사용하고 있다면 `contacts/1`에 대한 데이터가 이미 캐시되었다는 사실을 알고 있다. 따라서 해당 데이터를 즉시 보여주고 해당 데이터가 `stale` 상태라면 **Background refetch**를 실행한다. `loader` 접근 방식을 이용하는 경우 우리가 이전에 동일한 데이터에 대해서 데이터 패칭을 진행했음에도 불구하고 다시 데이터 패칭을 진행해야 한다. 그리고 데이터 패칭이 완료될 때까지 기다리기도 해야한다.

여기가 바로 **React Query**가 등장하는 부분이다.

만약 `loader`를 사용하여 **React Query** 캐시를 미리 채우긴 하지만 여전히 컴포넌트 내에서 `refetchOnWindowFocus`와 같은 모든 **React Query**의 기능을 가져오고 즉시 `stale` 데이터를 보여줄 수 있다면 어떨까? 필자는 이 방법이 두 라이브러리 모두에게 최선의 방법같이 느껴진다. `router`는 만약 캐시에 데이터가 없는 경우 데이터를 조기에 패칭하여 가져오는 역할을 하고 **React Query**는 데이터를 최신 상태로 캐싱하고 유지하는 역할을 한다.

## Querifying the example

위 설명을 예제로 옮겨보자.

`src/routes/contacts.jsx`

```jsx
import { useQuery } from '@tanstack/react-query';
import { getContact } from '../contacts';

// ⬇️ define your query
const contactDetailQuery = (id) => ({
  queryKey: ['contact', 'detail', id],
  queryFn: async () => getContact(id),
});

// ⬇️ needs access to queryClient
export const loader =
  (queryClient) =>
  async ({ params }) => {
    const query = contactDetailQuery(params.contactId);
    // ⬇️ return data or fetch it
    return queryClient.getQueryData(query.queryKey) ?? (await queryClient.fetchQuery(query));
  };

export default function Contact() {
  const params = useParams();
  // ⬇️ useQuery as per usual
  const { data: contact } = useQuery(contactDetailQuery(params.contactId));
  // render some jsx
}
```

`src/main.jsx`

```jsx
const queryClient = new QueryClient();

const router = createBrowserRouter([
  {
    path: '/',
    element: <Root />,
    children: [
      {
        path: 'contacts',
        element: <Contacts />,
        children: [
          {
            path: 'contacts/:contactId',
            element: <Contact />,
            // ⬇️ pass the queryClient to the route
            loader: contactLoader(queryClient),
          },
        ],
      },
    ],
  },
]);
```

위 예제에서 몇 가지 작업들이 있었는데 이를 더 자세히 한 번 탐구해보자.

### The loader needs access to the QueryClient

`loader`는 훅이 아니기에 `useQueryClient`를 사용할 순 없다. **Query Client**를 직접적으로 `import` 하는 방식은 [추천하지 않는다](https://tkdodo.eu/blog/react-query-fa-qs#why-should-i-usequeryclient). 따라서 명시적으로 전달하는 것이 가장 좋은 대안인 것 같다.

### getQueryData ?? fetchQuery

우리는 페이지가 처음 로딩될 때 사용자 경험 향상을 위해서 `loader`가 데이터가 준비될 때까지 기다리고 이를 반환해주길 원한다. 또한 `[errorElement](https://reactrouter.com/en/6.4.0/route/error-element)`로 에러가 전달되기를 원하기 때문에 `fetchQuery`가 최적의 옵션이다. 참고로 `prefetchQuery`는 아무것도 반환하지 않으며 내부적으러 에러를 캐치한다. 즉, `prefetchQuery`를 사용하면 이미 내부적으로 에러를 캐치하고 다시 전파하지 않기 때문에 `errorElement`까지 에러를 전달할 수 없다. (이를 제외하면 `prefetchQuery`와 `fetchQuery`는 동일하다)

`getQueryData`는 캐시에 존재하는 데이터가 `stale` 상태일지라도 이를 반환하는 작업을 수행한다. 이를 통해 이전에 방문했던 페이지에 재방문하는 경우 데이터를 즉시 보여줄 수 있게 된다. `getQueryData`가 `undefined`를 반환하는 경우(캐시에 아무런 데이터도 존재하지 않는 경우)에만 실제로 데이터 패칭을 수행한다.

다른 방법으로는 `fetchQuery`에 `staleTime`을 설정하는 것이다.

```jsx
export const loader =
  (queryClient) =>
  ({ params }) =>
    queryClient.fetchQuery({
      ...contactDetailQuery(params.contactId),
      staleTime: 1000 * 60 * 2,
    });
```

`staleTime`을 **2분**으로 설정한다는 것은 만약 데이터가 사용 가능한 상태고 2분이 지나지 않은 상태라면 `fetchQuery`가 데이터 패칭 대신 해당 데이터를 사용하도록 만드는 것을 뜻한다 . 만약 2분이 넘었다면 데이터 패칭이 실행된다. `stale` 상태의 데이터가 컴포넌트에 보이는 것이 괜찮다면 이는 좋은 대안이다.

`staleTime`을 `Infinity`로 설정한다는 것은 `staleTime` 보다 명시적인 `query invalidation`이 우선시 된다는 것만 제외하면 `getQueryData`. 접근 방법과 거의 동일하다. 따라서 코드의 양이 조금 더 증가할 지라도 `getQueryData` 접근 방법이 더 좋다.

### A TypeScript tip

위 방식을 통해 컴포넌트 내에서 `useQuery`를 호출하면 `useLoaderData`를 호출하는 것과 같이 일부 데이터를 사용할 수 있게 된다. 그러나 **TypeScript**는 이런 부분을 전혀 알 방도가 없다. 데이터의 반환 타입이 `Contact | undefined`이 된다.

> `loader`를 통해 `initalData`를 제공해주고 있지만 **TypeScript**로는 `initialData`가 제공되는지 모르기 때문에 반환타입이 `undefined`를 포함한 `union` 타입으로 추론된다.

Thanks to [Matt Pocock](https://twitter.com/mattpocockuk) and his [contribution](https://github.com/TanStack/query/pull/3834) to React Query v4, 이제 `initialData`가 제공되는 경우 `union` 타입에서 `undefined`를 제외할 수 있게 됐다.

`initalData`를 어디에서 얻을 수 있을까? 바로 `useLoaderData`다. 따라서, `loader` 함수로부터 타입 추론을 할 수 있다.

```tsx
export default function Contact() {
  const initialData = useLoaderData() as Awaited<ReturnType<ReturnType<typeof loader>>>;
  const params = useParams();
  const { data: contact } = useQuery({
    ...contactDetailQuery(params.contactId),
    initialData,
  });
  // render some jsx
}
```

`loader`는 함수를 반환하는 함수이기 때문에 작성해야할 게 좀 많지만 그래도 하나의 유틸로 뺄 수 있다. 현재로서는 **type assertions**을 사용하는 것이 `useLoaderData`의 반환 타입을 좁히는 유일한 방법인 것 같다. But it will nicely narrow the type of the *useQuery* result, which is what we want. 🙌

## Invalidating in actions

다음은 `query invalidation`이다. 아래는 **React Query** 없이 연락처를 업데이트하는 방법에 대한 `action` 예제이다.

```tsx
export const action = async ({ reqeust, params }) => {
  const formData = await request.formData();
  const updates = Object.fromEntries(formData);
  await updateContact(params.contactId, updates);
  return redirect(`/contacts/${params.contactId}`);
};
```

`action`이 `loader`를 `invalidate`하고 있지만 `loader`가 항상 캐시 데이터를 반환받도록 설정해놨기 때문에 우리가 캐시를 `invalidate` 하지 않는 이상 업데이트 결과를 볼 수가 없다.

It's just one line of code really:

```tsx
export const action =
  (queryClient) =>
  async ({ request, params }) => {
    const formDAta = await request.formData();
    const updates = Object.fromEntries(formData);
    await updateContact(params.contactId, updates);
    await queryClient.invalidateQueries(['contacts']);
    return redirect(`/contacts/${params.contactId}`);
  };
```

[fuzzy matching of invalidateQueries](https://tanstack.com/query/v4/docs/guides/query-invalidation#query-matching-with-invalidatequeries)는 `action`이 완료되고 상세 페이지로 리다이렉트될 때까지 리스트와 상세 페이지가 캐시로부터 새로운 데이터를 받을 수 있도록 해준다.

### await is the lever

그러나 이런 식으로 하게 되면 action 함수의 작업 시간이 길어지고 전환이 막혀버린다. 그렇다면 `invalidation`을 발생시키지 말고 상세 페이지로 리다이렉트한 다음에 우선 `stale` 데이터를 보여주고 새로운 데이터를 사용할 수 있게 되면 백그라운드에서 업데이트하게 할 순 없을까?

당연히 가능하다. 그저 `await` 키워드만 없애면 된다.

```tsx
export const action =
  (queryClient) =>
  async ({ request, params }) => {
    const formDAta = await request.formData();
    const updates = Object.fromEntries(formData);
    await updateContact(params.contactId, updates);
    queryClient.invalidateQueries(['contacts']);
    return redirect(`/contacts/${params.contactId}`);
  };
```

await 말 그래도 어느 방향으로든 당길 수 있는 레버가 된다.

(This analogy is based on Ryan's great talk [When To Fetch](https://www.youtube.com/watch?v=95B8mnhzoCM). Please watch it if you haven't already)

> 정말 좋은 컨퍼런스 영상, 필히 정리해서 블로그에 포스팅할 것.

- 가능한 한 빨리 상세 페이지로의 전환이 중요하다면 `await` 하지 말것
- stale 상태의 데이터를 보여줄 때 잠재적인 **layout shift**를 피하는 게 중요하거나 새 데이터를 얻을 때까지 `action`을 `pending` 상태로 유지하는 게 중요하다면 `await`를 사용할 것

만약 다수의 `invalidation`이 존재하는 경우 위 두 가지 접근 방법을 혼합하여 중요한 `refetch`는 기다리고 덜 중요한 것은 백그라운드에서 이루어지게 만들 수도 있다.

## Summary

I'm very excited about the new React Router release. It's a great step forward to enable all applications to trigger fetches as early as possible. However, it is not a replacement for caching - so go ahead and combine React Router with React Query to get the best of both worlds. 🚀

If you want to explore this topic some more, I've implemented the app from the tutorial and added React Query on top of it - you can find it in [the examples of the official docs](https://tanstack.com/query/v4/docs/examples/react/react-router).
