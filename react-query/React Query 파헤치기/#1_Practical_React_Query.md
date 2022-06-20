### 원본 글

https://tkdodo.eu/blog/practical-react-query

2018년엔 Apollo Client와 GraphQL이 등장했다. 등장한 것과 별개로 등장했을 때 GraphQL과 Apollo Client에게 던지는 가장 많은 질문 중 하나는 리덕스를 대체할 수 있냐는 질문이었다.

조금만 자세히 생각해보면 뭔가 이상한 질문이라는 것을 알 수 있다. 전역 상태를 컨트롤하기 위한 라이브러리를 데이터 패칭을 도와주는 라이브러리를 통해 대체할 수 있느냐고 묻는 질문이기 때문이다. 왜 이런 질문은 사람들은 하게 되는 걸까?

## Client State vs Server State

Apollo는 서버의 데이터를 패칭하고 이를 캐싱한다. 이 말은 동일한 `useQuery` 훅을 통해 여러 컴포넌트에서 사용하고 이 훅은 데이터를 패칭하며 이 후에는 캐시에서 이 데이터를 가져올 수 있다는 말이다.

뭔가 익숙하지 않은가? 리덕스의 목적에 대해서 생각해보자. 리덕스는 서버로부터 패칭한 데이터를 전역 즉, 서버로부터 가져온 데이터를 어디에서든 접근하여 사용할 수 있게 하기 위하여 만들어 졌다.

여기서 우리는 서버에서 가져온 데이터를 클라이언트 데이터처럼 다루고 있었다는 사실을 하나 발견할 수 있다. 애플리케이션 내에서 해당 데이터를 갖고 있지 않고 해당 화면에 렌더링하는 용도로만 쓸 것임에도 불구하고 말이다.

이제 감이 좀 오는가? Apollo는 서버에서 온 데이터를 클라이언트에서 전역 상태 라이브러리와 같은 도구로 애플리케이션 전체에 저장하지 않고도 서버 상태를 캐시를 통해 관리할 수 있게 만든다. 이는 많은 애플리케이션에서 리덕스의 사용을 줄이거나 크게는 제거할 수 있도록 만들어 줬다.

## React Query

REST API가 그렇듯 GraphQL도 장단점이 있으며 REST API를 이미 설계단부터 잘 만들어서 관리하고 있었다면 over-fetching과 같은 문제를 거의 겪을 일이 없다. 따라서 REST API를 벗어나고자 GraphQL을 사용하겠다! 라는 식의 전환은 일반적으로 흔하지 않다.

하지만 데이터 패칭 관리는 다르다. 프론트엔드에서 기존의 데이터 패칭 로직을 작성하는 것은 상당히 시간 소요와 귀찮음을 만드는 일 중 하나였다. 로직이 일관되지 않을 수도 있으며 로딩이나 에러 핸들링이 이쁘게 되지 않을 수도 있기 때문이다. React Query는 GraphQL의 일관적이고 간단한 데이터 패칭 관리와 로딩, 에러 핸들링을 보고 이를 REST API와 함께 써먹고 싶어서 만들어지기 시작했다.

## The Defaults explained

첫 번째로 React Query는 **queryFn을 매 리렌더링마다 호출하지 않는다. staleTime이 0일지라도**

애플리케이션은 다양한 이유로 리렌더링이 발생하기 때문에 매 리렌더링마다 데이터 패칭을 시도하는 것은 좋은 생각이 절 대 아니다.

만약 예상하지 못한 곳에서 `refetch`를 목격했다면 이는 React Query가 `refetchOnWindowFouch`를 수행하고 있는 상황에서 window가 focus된 상황일지도 모른다. 이 기능은 프로덕션 상에서 아주 좋은 기능 중 하나로 유저가 다른 브라우저 탭으로 이동했다가 애플리케이션으로 돌아온 경우 백그라운드에서 자동으로 `refetch`가 수행된다. 유저가 다른 탭을 갔다가 돌아온 시간 사이에 서버의 데이터의 변화가 생겼다면 업데이트된 데이터로 스크린을 업데이트 시킨다. 이러한 일련의 과정들은 로딩 스피너 없이 이루어 지며, refetch로 가져온 데이터가 캐시에 존재하는 데이터와 동일할 경우 리렌더링은 발생하지 않는다.

애플리케이션을 개발할 때는 위에서 언급한 일련의 과정이 더욱 더 빈번하게 발생하고 개발자 도구와 애플리케이션을 왔다 갔다 하는 사이에도 발생하므로 이 점을 염두해두자.

두 번째로 `cacheTime`과 `staleTime`을 확실하게 정리할 필요가 있다.

### StaleTime

StaleTime은 Query의 상태가 최신 상태에서 stale 상태로 전환될 때까지의 시간을 말한다. Query의 상태가 최신인 이상 데이터는 항상 캐시에서 먼저 읽어온다. 즉, 네트워크 요청이 발생하지 않는다. 다만, 특정 상황에서는 백그라운드 refetch가 발생할 수 있다. (ex. 새 쿼리 인스턴스가 생성된 경우)

### CacheTime

`CacheTime`은 캐시에서 사용되지 않는 쿼리가 제거되기 까지의 시간을 말한다. 이 시간은 기본값이 5분으로 지정되어 있다. 해당 쿼리를 더 이상 사용하는 곳이 없는 순간 쿼리는 비활성 상태가 되고 해당 쿼리를 사용하는 모든 컴포넌트는 언마운트된다.

일반적으로 `staleTime`에 대해서는 조작할 경우가 생기지만 `cacheTime`를 건드릴 경우는 거의 발생하지 않는다.

## Use the React Query DevTools

Reat Query DevTools는 캐시에 어떤 데이터가 저장되어 있는 지와 같은 정보를 손 쉽게 디버깅할 수 있게 해준다. 추가적으로 네트워크 스로틀링을 발생시킬 수 있으며 이는 백그라운드 refetch를 인지하는데 도움을 준다.

## Treat the query key like a dependency array

`queryKey`와 useEffect의 의존성 배열은 왜 비슷할까?

왜냐하면 React Query는 queryKey가 변경될 때마다 refetch를 발생시키기 때문이다. 그래서 우리가 queryFn에 변수를 전달할 때는 거의 값이 변경되어 데이터를 패칭하고 싶을 때이다. `refetch`를 일일히 직접 발생시키는 대신 queryKey를 조작하는 방법도 있다.

```typescript
type State = "all" | "open" | "done";
type Todo = {
  id: number;
  state: State;
};
type Todos = ReadonlyArray<Todo>;

const fetchTodos = async (state: State): Promise<Todos> => {
  const response = await axios.get(`todos/${state}`);
  return response.data;
};

export const useTodosQuery = (state: State) =>
  useQuery(["todos", state], () => fetchTodos(state));
```

자, 필터링 옵션을 가진 Todo 리스트가 있다고 상상해보자. 필터링 옵션은 로컬 상태로 관리하고 있으며 유저가 필터링 옵션을 변경할 경우 로컬 상태가 업데이트 되면서 queryKey가 변경되기 때문에 자동으로 refetch가 발생한다. 유저의 필터링 옵션을 계속해서 동기화하고 있는 위 상황은 useEffect의 의존성 배열과 유사하다.

### A new cache entry

queryKey는 cache의 key 값으로 사용되기 때문에 위 예제에서 State를 `all`에서 `done`으로 변경하는 것은 새로운 캐시 진입점을 가져오겠다는 것과 동일한 말이다. State를 `all`에서 `done`으로 변경하는 순간 로딩 상태 이후 새로운 결과가 화면에 렌더링될 것이다.

위 방법은 이상적인 방법은 아니다. 사용자 경험을 조금 더 상승시키고 싶다면 `keepPreviousData` 옵션을 사용하거나 `initalData`를 통해 생성될 캐시 데이터를 미리 생성할 수도 있다. 아래의 예제는 initialData를 이용하여 위 예제에서 Todo 리스트에 대한 클라이언트 사이드 pre-filtering을 구현한 것이다.

```typescript
type State = "all" | "open" | "done";
type Todo = {
  id: number;
  state: State;
};
type Todos = ReadonlyArray<Todo>;

const fetchTodos = async (state: State): Promise<Todos> => {
  const response = await axios.get(`todos/${state}`);
  return response.data;
};

export const useTodosQuery = (state: State) =>
  useQuery(["todos", state], () => fetchTodos(state), {
    initialData: () => {
      const allTodos = queryCache.getQuery<Todos>(["todos", "all"]);
      const filteredData =
        allTodos?.filter((todo) => todo.state === state) ?? [];

      return filteredData.length > 0 ? filteredData : undefined;
    },
  });
```

이제, 유저가 필터링 옵션을 변경할 때마다 우리가 데이터를 갖고 있지 않더라도 todos와 all을 쿼리 키로 갖는 쿼리의 캐시 데이터를 이용해서 데이터를 미리 채워줄 것이다. 우리는 이 과정을 통해 유저가 필터링 옵션을 done으로 변경한다고 해도 업데이트된 목록을 보여줄 수 있고 백그라운드 패칭이 완료되어도 여전히 업데이트된 목록을 보여줄 수 있게 된다. (사용자가 느끼기엔 변화가 없다는 점이 중요하다)

initialData의 코드 몇 줄로 인해 사용자는 로딩 스피너를 보지 않아도 되는데 이는 사용자 경험을 상승시킬 수 있다는 점이 아주 중요한 포인트이다.

## Keep server and client state separate

만약 useQuery를 통해 데이터를 서버로부터 가져왔다면 로컬 상태에 해당 데이터를 넣어 유지하려고 하지 않아야 한다. 왜냐하면 로컬 상태에 넣어서 유지하는 순간 React Query가 백그라운드 업데이트로 업데이트하는 범위 내에서 벗어나기 때문이다. (로컬 상태에 넣는 순간 결국 복사한 상태이기 때문)

폼의 default value들을 패칭하고 해당 value들이 존재할 경우 폼을 렌더링하려는 경우 이 방법은 괜찮은 방법될 수 있다. 이 경우에는 폼도 초기화된 상태이니 이후에 백그라운드 업데이트를 발생시킬 필요가 없다. 따라서, 이럴 때는 백그라운드 refetch가 발생하지 않도록 `staleTime`을 `Infinity`로 줘버린다.

```typescript
const App = () => {
	const { data } = useQuery('key', queryFn, { staleTime: Infinity })

	return data ? <MyForm initialData={data} /> : null
}

const MyForm = ({ initialData }) => {
	const [data, setData] = React.useState(initialData)
	...
}
```

이러한 방식의 가장 중요한 포인트는 React Query로 가져온 데이터를 절 대 로컬 상태에 저장하지 않는다는 점이다. 이렇게 함으로써 우리는 항상 최신 데이터를 볼 수 있게 된다.

## The enabled option is very powerful

**useQuery** 훅은 쿼리의 동작을 커스터마이징할 수 있는 다양한 옵션을 갖고 있는데 그 중 강력한 옵션 중 하나인 `enabled` 옵션에 대해 짧고 굵게 알아보자.

- enabled 속성을 통해 특정 속성에 의존하는 쿼리를 만들 수 있다.
  ex. 로그인 상태일 때만 쿼리가 동작하게 하는 등
- 쿼리의 첫 동작 이후 두 번째 동작부터는 첫 번째 쿼리로부터 성공적으로 받아왔을 때만 진행되도록 만들 수 있다.
- 쿼리의 동작을 시작할 수도 멈출 수도 있다.
  ⇒ 첫 번째랑 거의 비슷한 느낌인데 Modal이 활성화 됐을 때와 되지 않았을 때와 같이 특정 속성에 아예 의존하는 것이 아닌 일시적으로 쿼리의 동작을 컨트롤할 수 있다 라는 뜻인 것 같다.
- 사용자의 입력을 기다릴 수 있다.
- 사용자의 입력 중 특정 입력값에 대해서는 쿼리가 동작하지 않게 만들 수 있다.

⇒ 결국 다 비슷한 소리이긴 한데 그 경우의 수를 설명해준 느낌이다. “`enabled`라는 속성 하나로 위와 같이 다양한 경우에 따라 쿼리의 동작을 컨트롤할 수 있다.” 정도로 이해하고 넘어가자.

## Don’t use the queryCache as a local state manager

`queryCache`(ex. queryCache.setData)를 로컬 상태 매니저(ex. Redux, MobX 등)처럼 사용하지 말 것

queryCache는 오직 낙관적인 업데이트 또는 mutation 이후에 백엔드로부터 받는 응답 데이터를 이용해 업데이트하는 데에만 사용되어야 한다.

기억하자. 모든 백그라운드 refetch는 데이터를 오버라이딩한다. 따라서, 로컬 상태 관리는 다른 방법을 이용할 것

## Create custom hooks

만약 특정 queryFn을 사용하는 useQuery 호출이 하나 이상의 컴포넌트에서 사용이 된다면 커스텀 훅으로 만들어서 재사용하는 방법이 효과적이다.

왜냐하면

- You can keep the actual data fetching out of the ui, but co-located with your *useQuery* call.
  ⇒ 동일한 queryFn을 호출하는 useQuery를 커스텀 훅으로 사용하면 서로 다른 UI 화면에서 동일한 useQuery 로직을 손쉽게 사용할 수 있다. 정도로 이해하고 넘어가자.
- 하나의 queryKey에 대한(잠재적인 타입 정의, 타입스크립트를 사용할 경우) 관리를 하나의 파일 내에서 할 수 있게 된다.
- 일부 설정이 변경되거나 데이터 변환을 추가해야 하는 경우에도 여전히 한 파일 내에서 수행할 수 있게 된다.

```typescript
const useRandomValue = () => {
  const [draft, setDraft] = React.useState(undefined);
  const { data, ...queryInfo } = useQuery(
    "random",
    async () => {
      await sleep(1000);
      return Promise.resolve(String(Math.random()));
    },
    {
      enabled: !draft,
    }
  );

  return {
    value: draft ?? data,
    setDraft,
    queryInfo,
  };
};
```
