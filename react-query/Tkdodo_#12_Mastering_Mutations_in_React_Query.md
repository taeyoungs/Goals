# #12 Mastering Mutations in React Query

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
- [#11 React Query Error Handling](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%2311_React_Query_Error_Handling.md)
- #12 Mastering Mutations in React Query (현재)

**목차**

- [#12 Mastering Mutations in React Query](#12-mastering-mutations-in-react-query)
  - [개요](#개요)
  - [What are mutations?](#what-are-mutations)
  - [Similarities to useQuery](#similarities-to-usequery)
  - [Differences to useQuery](#differences-to-usequery)
  - [Tying mutations to queries](#tying-mutations-to-queries)
    - [Invalidation](#invalidation)
    - [Direct updates](#direct-updates)
  - [Optimistic updates](#optimistic-updates)
    - [Example](#example)
  - [Common Gotchas](#common-gotchas)
    - [awaited Promises](#awaited-promises)
    - [Mutate or MutatAsync](#mutate-or-mutatasync)
    - [Mutation only take one argument for variables](#mutation-only-take-one-argument-for-variables)
    - [Some callbacks might not fire](#some-callbacks-might-not-fire)

## 개요

앞선 포스팅들로 **React Query**가 제공하는 많은 기능들과 컨셉들에 대해서 다루었다. 앞서 다루었던 기능과 컨셉들은 대부분 `useQuery` 훅을 통하여 데이터 검색에 대한 것들이었다.

**React Query**에는 데이터를 가지고 작업하기 위한 필수적인 부분이 또 있는데 이는 바로 데이터를 업데이트 하는 것이다.

이러한 **use-case**를 위해 **React Query**는 `useMutation` 훅을 제공한다.

## What are mutations?

일반적으로 `mutation`은 **side effect**를 가지고 있는 함수다. **side effect**를 가진 함수의 예로는 배열의 메서드인 `push`를 있다. `push` 메서드는 값을 다음과 같이 집어 넣어 원본 배열을 변경하는 **side effect**를 발생시킨다.

```jsx
const myArray = [1];
myArray.push(2);

console.log(myArray); // [1, 2]
```

원본 배열을 건드리지 않고 불변성을 지키는 메서드로는 `concat`이 있다. `concat`은 `push` 메서드처럼 배열에 값을 추가할 수 있지만 원본 배열을 직접 변형하는 대신 값이 추가된 새로운 배열을 반환해 준다.

```jsx
const myArray = [1];
const newArray = myArray.concat(2);

console.log(myArray); // [1]
console.log(newArray); // [1, 2]
```

이름에서 알 수 있듯이, `useMutation`도 일종의 **side effect**를 가진다. 우리는 서버 상태를 관리하는 **React Query**의 `context`에 있기 때문에 `mutation`은 서버에 **side effect**를 발생시킬 수 있는 함수를 말한다. 데이터베이스에 `Todo`를 생성하는 것은 `mutation`이다. 사용자가 로그인하는 것도 전통적인 `mutation`을 뜻한다. 왜냐하면 사용자에 대한 토큰을 생성하는 것에서 **side effect**가 발생하기 때문이다.

어떤 측면에서 `useMutation`은 `useQuery`와 비슷하지만 한편으론 꽤나 다르다.

## Similarities to useQuery

`useMutation`은 `useQuery`가 `query`에 대해 그러하는 것처럼 `mutation`의 상태를 추적할 것이다. `useMutation`은 우리가 사용자에게 무슨 일이 일어나고 있는지 쉽게 보여줄 수 있도록 `loading`, `error` 그리고 `status` 필드를 제공할 것이다.

또한 `useQuery`가 가진 유용한 콜백(`onSuccess`, `onError` 그리고 `onSettled`)들도 사용할 수 있다.

하지만 비슷한 점이 이게 끝이다.

## Differences to useQuery

> **`useQuery`는 선언적이고 `useMutation`은 명령적이다.**

이말인 즉슨 `query`들은 대부분 자동으로 실행된다. 의존성을 정의하긴 하지만 **React Query**는 자동으로 `query`를 실행하고 필요한 경우 **Background update**까지 수행한다. 우리는 화면에 보여지는 것들이 백엔드의 실제 데이터들과 동기화되어 유지되길 원하기 때문에 이런 부분들은 `query`들에게 상당히 유용한 부분들이다.

하지만 `mutation`의 경우 그렇지 않다. 우리가 브라우저 윈도우에 포커징할 때마다 새로운 `Todo`가 생성된다고 상상해봐라. 따라서, **React Query**는 `mutation`을 호출 즉시 실행하는 대신 `mutation`을 발생시키고 싶을 때 호출할 수 있는 함수를 제공한다.

```jsx
function AddComment({ id }) {
  // this doesn't really do anything yet
  const addComment = useMutation((newComment) => axios.post(`/posts/${id}/comments`, newComment));

  return (
    <form
      onSubmit={(event) => {
        event.preventDefault();
        // ✅ mutation is invoked when the form is submitted
        addComment.mutate(new FormData(event.currentTarget).get('comment'));
      }}
    >
      <textarea name="comment" />
      <button type="submit">Comment</button>
    </form>
  );
}
```

또 다른 차이점 중 하나는 `mutation`은 `useQuery`와 같이 `state`를 공유하지 않는다는 것이다. 서로 다른 컴포넌트들에서 동일한 `useQuery`를 여러 번 호출할 수 있고 이로 인해 반환되는 값은 동일한 캐시 데이터일 것이다.

그러나 `mutation`은 그렇게 동작하지 않는다.

## Tying mutations to queries

`Mutation`은 설계에 따라 `query`에 직접 연결되지 않는다. 블로그 포스팅에 좋아요를 누르는 것과 같은 `mutation`은 블로그 포스팅 데이터를 패칭하는 `query`와는 관련이 없다. 이게 가능해지도록 하려면 **React Query**가 가지고 있지 않은 일종의 **underlying schema**가 필요하다.

> 블로그 좋아요 기능에 대한 mutation을 발생시킨다고 블로그 포스팅 데이터에 대한 네트워크 요청이 발생하진 않는다. 하지만 좋아요를 업데이트 했다는 말은 블로그 포스팅 데이터에 대한 업데이트가 필요하다는 뜻이고 네트워크 요청을 발생시켜야만 한다.
>
> 따라서, `mutation`과 연계해서 `query` 데이터를 업데이트 시키기 위한 방법을 **React Query** 측에서 제공하고 있다.

`mutation`이 `query`들에게 변경사항을 반영하게 만들기 위해서 **React Query**는 두 가지 방법을 제공한다.

### Invalidation

`invalidation`은 이론적으로 화면을 최신 상태로 만드는 가장 간단한 방법이다. 기억을 되살려보자. 서버 상태는 우리가 네트워크 요청을 한 시점의 데이터 **Snapshot**을 보여줄 뿐이다. 물론 **React Query**가 데이터를 최신 상태로 유지하기 위해서 노력할 것이다. 다만, `mutation`을 통해 의도적으로 서버 상태를 바꾼 경우 **React Query**에게 캐시된 데이터가 이제 더 이상 유효하지 않다 라고 알려주기 적합한 때라는 것이다. 그리하면 **React Query**는 해당 데이터가 사용 중인 경우 `refetch`를 발생시켜 데이터 패칭이 완료된 순간 화면을 자동으로 업데이트할 것이다. 우리가 할 일은 어느 `query`를 `invalidate` 할 건지 라이브러리에게 알려주는 것 뿐이다.

```jsx
const useAddComment = (id) => {
  const queryClient = useQueryClient();

  return useMutation((newComment) => axios.post(`/posts/${id}/comments`, newComment), {
    onSuccess: () => {
      // ✅ refetch the comments list for our blog post
      queryClient.invalidateQueries(['posts', id, 'comments']);
    },
  });
};
```

**Query Invalidation**은 꽤나 똑똑하다. 모든 **Query Filters**와 같이 **Query Key**에 `fuzzy` 매칭을 사용한다. 그래서 만약 댓글 목록에 대해 여러 키를 가지고 있는 경우 모두 `invalidate`될 것이다. 단, 해당 키를 가진 `query` 중 오직 현재 활성 상태인 것들만 `refetch`될 것이다. 나머지는 `stale`로 표시되어 있다가 다음에 해당 키들이 사용될 때 `refetch`가 발생할 것이다.

예를 들어, 댓글들에 대해 정렬 옵션을 가지고 있다고 가정해보자. 그리고 이때 새로운 댓글이 추가됐고 우리의 캐시 데이터엔 댓글에 대한 두 가지 `query`가 존재하는 상황이다.

```
['posts', 5, 'comments', { sortBy: ['data', 'asc'] }]
['posts', 5, 'comments', { sortBy: ['author', 'desc'] }]
```

화면엔 이 두 `query` 중 하나만 표시하고 있기 때문에 `invalidateQueries`는 `query` 중 하나만 `refetch`할 것이고 나머지 하나는 `stale`로 표시해둘 것이다.

### Direct updates

가끔은 데이터를 `refetch`하지 않고 싶을 때가 있다. 특히나 `mutation`을 통해 이미 우리가 원하는 데이터의 모든 것을 반환받은 상황일 경우 말이다.

만약 블로그 포스팅의 타이틀을 업데이트하는 `mutation`을 발생시켰고 백엔드가 해당 `mutation`에 대한 응답으로 업데이트된 블로그 포스팅 데이터를 반환해준 경우 `setQueryData`를 통해 `query` 캐시 데이터를 직접 업데이트할 수도 있다.

```jsx
const useUpdateTitle = (id) => {
  const queryClient = useQueryClient();

  return useMutation((newTitle) => axios.patch(`/posts/${id}`, { title: newTitle }).then((response) => response.data), {
    // 💡 response of the mutation is passed to onSuccess
    onSuccess: (newPost) => {
      // ✅ update detail view directly
      queryClient.setQueryData(['posts', id], newPost);
    },
  });
};
```

`setQueryData`를 통하여 캐시 데이터에 데이터를 직접 전달하는 것은 마치 백엔드로부터 데이터를 반환받은 것과 같이 동작한다. 이 말인 즉슨 해당 `query`를 사용하고 있는 모든 컴포넌트들은 리렌더링될 것이라는 말이다.

I'm showing some more examples of direct updates and the combination of both approaches in [#8: Effective React Query Keys](https://tkdodo.eu/blog/effective-react-query-keys#structure).

필자는 개인적으로 대부분의 경우 `invalidation`이 캐시 데이터를 직접 업데이트하는 `setQueryData`와 같은 작업보다 우선시 되어야 한다고 생각한다. 물론 **use-case**에 따라 다르겠지만 캐시 데이터를 직접 업데이트하는 것이 확실하게(또는 안전하게) 동작하기 위해선 프론트엔드 측에 더 많은 코드가 필요하고 백엔드에선 이에 대한 어느 정도의 중복된 로직이 필요하게 된다. 예를 들어 정렬된 목록은 직접 업데이트하기가 꽤나 어렵다. 업데이트로 인해 내 항목의 위치가 잠재적으로 업데이트 됐을 수도 있기 때문이다. 전체 목록에 대해서 `invalidate` 하는 것이 더 "안전한" 접근 방법이다.

## Optimistic updates

낙관적 업데이트는 **React Query** `mutation`을 사용하는 주요 포인트 중 하나이다. `useQuery` 캐시는 쿼리 간에 전환될 때 캐시 데이터를 즉시 주는데 특히 `prefetching`과 결합될 때 더욱 더 그러하다. 우리가 가진 전체적인 UI는 이로 인해 빠르게 느껴진다. 그렇다면 `mutation`에서도 이와 같은 이점을 얻는건 어떨까?

대부분의 경우 우리는 업데이트가 잘 진행될 것이라고 확신한다. 사용자는 왜 우리가 UI에 결과를 보여주기 위해 백엔드에서 승인을 받을 때까지 몇 초 동안 기다려야 할까? 낙관적 업데이트의 아이디어는 `mutation`을 서버로 보내기도 전에 `mutation`이 성공했다고 거짓말하는 것이다. `mutation`에 대해 성공적인 응답을 받았다면, 실제 데이터를 보기 위해 `invalidate` 하기만 하면 된다. 요청이 실패한 경우 `mutation`을 보내기 이전 상태의 **UI**로 돌아가야 한다.

낙관적 업데이트는 즉각적인 사용자 피드백이 실제로 필요한 작은 단위의 `mutation`에서 유용하다. 요청을 수행하기 위한 토글 버튼이 있는 것 보다 나쁠게 없으며 해당 요청이 완료될 때까지 반응하지 않는다. 사용자가 두 번 혹은 그 이상 버튼을 클릭해도 그저 "렉이 있구나" 생각할 뿐이다.

### Example

I've decided to *not* show an additional example. The [official docs](https://react-query.tanstack.com/guides/optimistic-updates) cover that topic very well, and they also have a codesandbox example [in JavaScript](https://react-query.tanstack.com/examples/optimistic-updates) and [in TypeScript](https://react-query.tanstack.com/examples/optimistic-updates-typescript).

필자는 낙관적 업데이트가 약간 과도하게 사용된다고 생각한다. 모든 `mutation`이 항상 낙관적일 순 없다. 항상 낙관적일 순 없고 롤백에 대한 사용자 경험성. 즉 **UX**는 좋지 않기 때문에 실패하는 경우가 거의 없다는 것을 확실히 해야한다. 양식을 제출하면 닫히는 Dialog 내에 있는 Form 또는 업데이트 후에 상세 페이지에서 목록 페이지로 리다이렉트되는 것들을 상상해보자. 이러한 작업들은 조기에 완료되면 취소하기 어려운 작업들이다.

그리고 즉각적인 피드백(위 토글 버튼 예제와 같은)이 과연 정말로 필요한지 확인하자. 낙관적 업데이트를 하기 위해 필요한 코드는 일반적인 `mutation` 코드와 비교해 별반 차이가 없다. 낙관적 업데이트를 할 경우 백엔드에서 이루어지는 작업을 따라해야 하는데 이는 배열의 아이템을 추가하거나 `Boolean` 값을 뒤집는 것과 같이 간단하지 않고 훨씬 더 복잡할 수도 있다.

- 만약 `Todo`를 추가하고자 한다면 `id`가 필요할텐데 이 `id`는 어디서 얻어야 할까?
- 만약 현재 보고 있는 목록이 정렬된 상태라면 새로운 항목을 삽입할 때 올바른 위치에 집어 넣을 수 있을까?
- 이러한 작업 도중 다른 사용자가 또 다른 항목을 추가했다면? 우리가 낙관적 업데이트로 추가한 항목은 `refetch` 이후에 위치를 변경됐을까?

이러한 엣지 케이스들은 버튼을 비활성화하고 mutation이 실행 중인 동안 로딩 애니메이션을 보여주는 것만으로도 충분한 몇몇 상황에서 사용자 경험성(**UX**)을 오히려 안좋게 만들 수도 있다. 항상 말하지만 알맞은 곳에 알맞은 도구를 골라라.

## Common Gotchas

마지막으로 `mutation`을 다룰 때 알아두면 좋은 몇 가지를 살펴볼 것이다.

### awaited Promises

`mutation` 콜백으로부터 반환된 `Promise`는 **React Query**에 의해 `await`되며 이에 따라 `invalidateQueries`도 `Promise`를 반환한다. 만약 관련 `query`가 업데이트되는 동안 `mutation`을 로딩 상태로 유지하고 싶은 경우 콜백에서 `invalidateQueries`의 결과를 반환하면 된다.

```jsx
{
  // 🎉 will wait for query invalidation to finish
  onSuccess: () => {
    return queryClient.invalidateQuries(['posts', id, 'comments']);
  };
}
{
  // 🚀 fire and forget - will not wait
  onSuccess: () => {
    queryClient.invalidateQueries(['posts', id, 'comments']);
  };
}
```

### Mutate or MutatAsync

`useMutation`은 두 가지 함수(`mutate`와 `mutateAsync`)를 제공한다. 이 둘의 차이점은 무엇이고 무엇을 어떤 때에 선택해서 써야할까?

`mutate`는 어떤 것도 반환하지 않는 반면에 `mutateAsync`은 `mutation`의 결과를 포함하는 `Promise`를 반환한다. 그래서 `mutation`의 응답 데이터에 접근하고 싶을 때 `mutateAsync`를 사용하고 싶을 수도 있겠지만 여전히 필자는 웬만하면 `mutate`를 사용해야 한다고 주장한다.

왜냐하면 `mutate`를 사용하더라도 여전히 콜백을 통해 데이터 또는 에러에 접근할 수 있으며 에러 핸들링에 대해 걱정할 필요가 없다. `mutateAsync`는 `Promise`를 제어할 수 있도록 하기 때문에 에러를 일일히 잡아줘야 한다. 그렇지 않으면 **unhandled promise rejection** 알림을 받게 될 수도 있다.

```jsx
const onSubmit = () => {
  // ✅ accessing the response via onSuccess
  myMutation.mutate(someData, {
    onSuccess: (data) => history.push(data.url),
  });
};

const onSubmit = async () => {
  // 🚨 works, but is missing error handling
  const data = await myMutation.mutateAsync(someData);
  history.push(data.url);
};

const onSubmit = async () => {
  // 😕 this is okay, but look at the verbosity
  try {
    const data = await myMutation.mutateAsync(someData);
    history.push(data.url);
  } catch (error) {
    // do nothing
  }
};
```

`mutate`에 대한 에러 핸들링은 필요없다. 왜냐하면 **React Query**가 내부적으로 에러를 잡아서 처리하기 때문이다. It is literally implemented with: *mutateAsync().catch(noop)*😎

필자가 `mutataAsync`가 더 유용하다고 느낀 유일한 상황은 `Promise`를 갖기 위해 `Promise`가 정말로 필요한 경우이다. 이는 여러 `mutation`을 동시에 실행하고 `mutation`들이 모두 완료될 때까지 기다리려는 경우(`Promise.all` 같은) 또는 콜백과 함께 콜백 헬에 들어가는 종속적인 `mutation`이 있는 경우에 필요할 수 있다.

> This can be necessary if you want to fire off multiple mutations concurrently and want to wait for them all to be finished, or if you have dependent mutations where you'd get into callback hell with the callbacks.

### Mutation only take one argument for variables

`mutate`의 마지막 인자는 옵션 객체이기 때문에 `useMutation`은 현재 변수에 대해 오직 하나의 인자만을 다룰 수 있다. 이는 제한사항 같아 보이지만 객체를 사용하여 쉽게 해결할 수 있다.

```jsx
// 🚨 this is invalid syntax and will NOT work
const mutation = useMutation((title, body) => updateTodo(title, body));
mutation.mutate('hello', 'word');

// ✅ use an object for multiple variables
const mutation useMutation(({ title, body }) => updateTodo(title, body));
mutation.mutate({ title: 'hello', body: 'word' });
```

To read more on why that is currently necessary, have a look at [this discussion](https://github.com/tannerlinsley/react-query/discussions/1226).

### Some callbacks might not fire

`useMutation`이 갖는 콜백들을 `mutate`에도 갖게 할 수 있다. 여기서 중요한 점은 `useMutation`의 콜백이 `mutate`의 콜백보다 먼저 실행된다는 것이다. 더 나아가 `mutate`가 완료되기 전에 컴포넌트가 `unmount` 되는 경우 `mutate`의 콜백은 실행되지 않을 수도 있다.

그렇기 때문에 콜백에서 우려 사항을 분리하는 것이 좋다.

- `useMutation` 콜백에선 `query invalidation`과 같이 로직과 관련되고 절대적으로 필요한 것들을 수행하자.
- 리다이렉트 또는 토스트 알림을 보여주는 것과 같이 UI와 관련된 것들은 `mutate` 콜백에서 수행하자. 만약 사용자가 `mutation`이 완료되기 전에 현재 화면을 벗어나려고 한다면 해당 콜백은 의도적으로 실행되지 않을 것이다.

이러한 분리는 커스텀 훅으로 래핑된 `useMutation`의 경우 특히 깔끔하다. 이렇게 하면 `React Query` 로직과 관련된 `query`는 커스텀 훅 내에서 유지하고 **UI**와 관련된 작업들은 **UI** 단에서 있을 수 있다. 이는 커스텀 훅을 더 재사용 가능하게 만들어 준다. 왜냐하면 **UI**와 상호 작용하는 부분들은 상황에 따라 다를 것이기 때문이다. (`invalidation` 로직은 항상 동일할 것이다)

> **UI** 관련된 코드들은 **UI** 단에서 유지하고 커스텀 훅엔 **React Query**와 관련된 로직만 남기 때문에 커스텀 훅을 더 다양한 상황에 맞춰서 재사용할 수 있다.

```jsx
const useUpdateTodo = () =>
  useMutation(updateTodo, {
    // ✅ always invalidate the todo list
    onSuccess: () => {
      queryClient.invalidateQuries(['todos', 'list']);
    },
  });

// in the component

const updateTodo = useUpdateTodo();
updateTodo.mutate(
  { title: 'newTitle' },
  // ✅ only redirect if we're still on the detail page
  // when the mutation finishes
  { onSuccess: () => history.push('/todos') }
);
```
