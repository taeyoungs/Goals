# Main Concepts (v6.4.3)

> 원본 글  
> https://reactrouter.com/en/main/start/concepts#main-concepts

> React Router v6.4.3 기준

**목차**

- [Main Concepts (v6.4.3)](#main-concepts-v643)
  - [Overview](#overview)
  - [Definitions](#definitions)
  - [History and Locations](#history-and-locations)
    - [History Object](#history-object)
    - [Locations](#locations)
  - [Matching](#matching)
    - [Defining Routes](#defining-routes)
    - [Match Params](#match-params)
    - [Pathless Routes](#pathless-routes)
    - [Route Matches](#route-matches)
  - [Rendering](#rendering)
    - [Outlets](#outlets)
    - [Index Routes](#index-routes)
    - [Layout Routes](#layout-routes)
  - [Navigating](#navigating)
    - [Link](#link)
    - [Navigate Function](#navigate-function)
  - [Data Access](#data-access)
  - [Review](#review)

## Overview

**React Router**가 정확히 무슨 일을 할까? 어떻게 애플리케이션을 빌드하는데 도움이 될까? `router`란 정확히 무엇일까?

만약 위와 같은 질문을 떠올린 적이 있거나 라우팅의 근본을 알고 싶다면 여기에 그 답이 있을 것이다. 이 문서에는 **React Router**에 구현된 라우팅 이면의 모든 핵심 개념에 대한 자세한 설명이 담겨 있다.

이 문서에 압도될 필요는 없다. **React Router**를 사용하는 방법 자체는 간단하기 때문에 단순히 **React Router**를 사용하기 위해서라면 이렇게 깊게 알아볼 필요는 없다.

**React Router**는 단순히 **URL**을 함수나 컴포넌트에 매칭시키는 것이 아니다. **React Router**는 **URL**에 매핑되는 전체 사용자 인터페이스를 구축하는 것이므로 평소에 알고 있던 것보다 더 많은 개념이 포함될 수 있다.

**React Router**의 3가지 주요 작업에 대해 자세히 알아보자.

1. **history stack**을 구독하고 조작하는 것
2. **URL**에 `route`를 매칭시키는 것
3. 매칭된 `route`에 중첩된 **UI**를 렌더링하는 것

## Definitions

하지만 먼저, 몇 가지 정의에 대해 짚고 넘어가자. 백엔드와 프론트엔드 프레임워크에서는 라우팅에 대한 많고 다양한 아이디어가 있다. 따라서, 가끔 사용되는 용어의 차이가 발생할 수 있다.

아래에는 **React Router**에 대해서 이야기할 때 자주 사용하는 용어들이다. 뒤에서 이 용어들에 대해서 더 자세히 설명한다.

- **URL** - 주소창에 있는 **URL**을 말한다. 많은 사람들이 `URL`과 `route`를 같은 의미의 용어처럼 사용하지만 **React Router**에서 `route`는 `URL`이 아니다.
- **Location** - 브라우저의 빌트인 `window.location` 객체를 기반으로 하는 **React Router**의 특정 객체를 말한다. 이 객체는 사용자가 어디에 머무르고 있는지를 나타낸다. 이는 대개 URL의 객체 표현이지만 단순히 URL 말고도 조금 더 많은 정보를 가지고 있다.
- **Location State** - **URL**에서 인코딩되지 않은 `location`을 유지하는 값. 해시와 **search params**(**URL**에서 인코딩된 데이터)와 비슷하지만 보이지 않게 브라우저의 메모리에 저장된다.
- **History Stack** - 사용자가 `navigate` 할 때마다, 브라우저는 `stack`에 각 `location`을 쌓아서 추적한다. 만약 브라우저의 뒤로 가기 버튼을 클릭하거나 누르고 있으면 브라우저의 **history stack**을 볼 수 있다.
- **Client Side Routing (CSR)** - `plain` **HTML** 문서는 다른 **HTML** 문서와 연결될 수 있고 브라우저는 직접 **history stack**을 처리한다. 클라이언트 사이드 라우팅은 개발자가 서버에게 **HTML** 문서를 요청하지 않고도 브라우저 **history stack**을 조작할 수 있게 해준다.
- **History** - **React Router**가 **URL**의 변경에 대해 구독할 수 있게 하고 프로그래밍 방식으로 브라우저의 **history stack**을 조작하는 **API**를 제공하는 객체다.
- **History Action** - 사용자들은 `POP`, `PUSH`, 또는 `REPLACE` 중 하나의 **history action**을 통해 **URL**에 도달할 수 있다. **PUSH**는 새로운 `history`가 **history stack**에 추가된다(일반적으로 링크를 클릭하거나 프로그래머가 강제로 `navigation` 시킬 때 **PUSH**가 이루어진다). **REPLACE**는 `stack`의 마지막 `history`가 새롭게 추가되는 `history`로 대체되는 것을 제외하면 **PUSH**와 비슷하다. 마지막으로 **POP**은 **Chrome** 브라우저에서 사용자가 뒤로 가기나 앞으로 가기 버튼을 불렀을 때 발생한다.
- **Segment** - `'/'` 문자 사이에 **URL** 일부 또는 **path pattern**을 의미한다. 예를 들어, **URL**이 `/users/123`인 경우 두 가지 `segment`(`users`, `123`)를 가지고 있는 상태다.
- **Path Pattern** - **URL**처럼 보이지만 **dynamic segments**(`"/users/:userId"`) 또는 **star segments**(`"/docs/*"`)와 같이 **URL**을 경로에 일치시키기 위한 특수 문자가 있을 수 있다. **dynamic**이나 **star segment**는 **URL**이 아니라 **React Router**가 매치시킬 패턴들이다.
- **Dynamic Segment** - **path pattern** 중 하나인 **dynamic segment**로 **segment**에 어떠한 값이 오든 매치가 가능하다. 예를 들어, `/users/:userId` 패턴은 `/users/123`과 같은 **URL**과 매치된다.
- **URL Params** - **URL**에서 **dynamic segment**와 매칭되어 파싱된 값들을 말한다.
- **Router** - **Stateful**하며 모든 컴포넌트들과 훅이 동작하도록 만드는 최상단 컴포넌트다.
- **Route Config** - A tree of **routes objects** that will be ranked and matched (with nesting) against the current location to create a branch of **route matches**.
- **Route** - 객체 또는 **Route Element**를 뜻한다. 일반적으로 `{ path, element }`
   또는 `<Route path element>`와 같은 형태를 취하고 있다. 여기서 `path` 속성은 **path pattern**이다. **path pattern**이 현재 **URL**에 매치될 때 `element` 속성이 렌더링될 것이다.
- **Route Element** - 또는 `<Route>`. 이 요소의 속성은 `<Routes>`에 의해 `route`를 생성하기 위해 읽히지만 `route`를 생성할 때를 제외하곤 아무런 일도 하지 않는다.
- **Nested Routes** - `route`는 **children route**를 가지고 각 `route`는 `segment`를 통하여 **URL**의 일부를 정의하기 때문에 단일 **URL**은 중첩된 `route`들 중에 다수의 `route`들과 매치될 수도 있다.
- **Relative links** - `'/'`로 시작하지 않는 링크는 렌더링된 가장 가까운 경로를 상속한다. 이렇게 하면 전체 경로를 알고 빌드할 필요 없이 더 깊은 **URL**에 쉽게 연결할 수 있습니다.
- **Match** - `route`가 **URL**과 일치할 때 **URL Params**나 매칭된 `pathname`과 같은 정보를 가지고 있는 객체
- **Matches** - 현재 `location`에 매칭된 `route`들(or branch of the [route config](https://reactrouter.com/en/main/start/concepts#route-config)
  )이 담긴 배열이다. This structure enables [nested routes](https://reactrouter.com/en/main/start/concepts#nested-routes).
- **Parent Route** - **child route**를 가지고 있는 `route`
- **Outlet**  - A component that renders the next match in a set of [matches](https://reactrouter.com/en/main/start/concepts#match).
- **Index Route** - `path` 속성이 없어서 부모 `route`의 **URL**가 매칭되어 부모 `route`의 `Outlet`에서 렌더링되는 **child route**다.
- **Layout Route** - `path` 속성이 없는 **parent route**로, 특정 레이아웃 내에서 **child route**를 그룹화하는 데만 사용된다.

## History and Locations

**React Router**가 무엇이든 할 수 있으려면 먼저 브라우저 **history stack**에 대한 변경 사항에 대해 구독할 수 있어야 한다.

브라우저는 사용자가 `navigate` 할 때 고유의 **history stack**을 유지한다. 이러한 **history stack**이 뒤로 가기 그리고 앞으로 가기 버튼이 동작할 수 있는 이유다. 전통적인 웹사이트(**JavaScript**가 없는 **HTML** 문서)의 브라우저는 사용자가 링크를 클릭하거나 `form`을 `submit` 하거나 뒤로 가기 그리고 앞으로 가기 버튼을 누를 때마다 서버에게 요청을 만들어 냈었다.

예를 들어,

1. `/dashboard`라는 링크를 클릭
2. `/accounts`라는 링크를 클릭
3. `/customers/123`라는 링크를 클릭
4. 뒤로 가기 버튼 클릭
5. `/dashboard`라는 링크를 클릭

**history stack**은 굵게 표시된 항목이 현재 **URL**을 나타내며 다음과 같이 변경된다.

1. **`/dashboard`**
2. `/dashboard`, **`/accounts`**
3. `/dashboard`, `/accounts`, `/customers/123`
4. `/dashboard`, `**/accounts**`, `/customers/123`
5. `/dashboard`, `/accounts`, **`/dashboard`**

### History Object

클라이언트 사이드 라우팅을 통해 개발자들은 브라우저 **history stack**을 프로그래밍 방식으로 조작할 수 있게 된다. 예를 들어, 서버에게 요청을 만들어 내는 브라우저의 기본 동작 없이 **URL**을 변경하기 위해서 다음과 같이 코드를 작성할 수 있다.

```jsx
<a
  href="/contact"
  onClick={(event) => {
    // stop the browser from changing the URL and requesting the new document
    event.preventDefault();
    // push an entry into the browser history stack and change the URL
    window.history.pushState({}, undefined, '/contact');
  }}
/>
```

> ✋ **주의**
>
> 설명을 위한 코드일 뿐이니 `window.history.pushState`를 **React Router**에 직접 사용하지는 말 것

위 코드는 **URL**을 변경하기는 하지만 **UI**에는 아무런 일도 일어나지 않는다. 따라서, UI가 연락처 페이지로 변하려면 어딘가에서 상태값 일부를 변경하는 코드를 추가적으로 작성해야 한다. 문제는 브라우저가 우리에게 URL의 변경 사항에 대해 구독할 수 있는 방법을 제공해주지 않는다는 것이다.

완전히 맞는 말은 아닌 것이 **history action** 중 `POP` 이벤트에 대해서는 **URL**이 변하는 것을 알 수 있다.

```jsx
window.addEventListener('popstate', () => {
  // URL changed!
});
```

하지만 이는 오직 사용자가 뒤로 가기 버튼이나 앞으로 가기 버튼을 눌렀을 때만이다. 프로그래머가 `window.history.pushState`나 `window.history.replaceState`를 호출했을 때 발생하는 이벤트는 없다.

바로 여기에서 **React Router**의 `history` 객체가 작동한다. 이는 **history action**이 `PUSH`, `POP`, `REPLACE`인지 아닌지에 상관없이 **URL** 변경에 대해 알 수 있는 방법을 제공한다.

```jsx
let history = createBrowserHistory();
history.listen(({ location, action }) => {
  // this is called whenever new locations come in
  // the action is POP, PUSH, or REPLACE
});
```

애플리케이션은 애플리케이션 고유의 `history` 객체를 설정할 필요가 없다. 이는 `<Router>`가 할 일이다. 이 객체 중 하나를 설정하고 history stack의 변경 사항을 구독하며 마지막으로 **URL**이 변경되면 상태값을 업데이트한다. 이는 애플리케이션에 리렌더링을 일으키고 올바른 UI를 보여준다. The only thing it needs to put on state is a `location`, everything else works from that single object.

### Locations

브라우저는 `window.location` 속성이라는 `location` 객체를 가지고 있다. 해당 객체에는 URL에 대한 정보와 이를 변경할 수 있는 메서드도 존재한다.

```jsx
window.location.pathname = window.location.hash; // /getting-started/concepts/ // #location
window.location.reload(); // force a refresh w/ the server
// and a lot more
```

**React Router**는 `window.location`을 사용하는 대신에 `window.location` 보다 패턴화되어 있지만 더 간단한 `location` 이라는 개념을 가지고 있다.

이는 다음과 같이 이루어져있다.

```jsx
{
  pathname: '/bbq/pig-pickins',
  search: '?campaign=instagram',
  hash: '#menu',
  state: null,
  key: 'aefz24ie'
}
```

위 객체에서 `{ pathname, search, hash }`는 `window.location`이 가지고 있던 속성들과 동일하다. 만약 이 셋을 합치면 사용자가 브라우저에서 볼 수 있는 URL을 얻을 수 있다.

```jsx
location.pathname + location.search + location.hash;
// /bbq/pig-pickins?campaign=instagram#menu
```

나머지 두 개 `{ state, key }`는 **React Router**만의 것이다.

- **Location Pathname**

  `origin` 다음에 오는 **URL**의 일부로 `https://example.com/teams/hotspurs` 여기에서 `pathname`은 `/teams/hotspurs`가 된다.
  This is the only part of the location that routes match against.

- **Location Search**

  이는 URL의 일부로 이를 설명하는 용어는 상당히 많다.

  - location search
  - search params
  - URL search params
  - query string
    **React Router**에선 **location search**라고 부른다. 하지만 **location search**는 `URLSearchParams`의 직렬화된 버전이기 때문에 **URL search params**라고도 부른다.

  ```jsx
  // given a location like this:
  let location = {
    pathname: '/bbq/pig-pickins',
    search: '?campaign=instagram&popular=true',
    hash: '',
    state: null,
    key: 'aefz24ie',
  };

  // we can turn the location.search into URLSearchParams
  let params = new URLSearchParams(location.search);
  params.get('campaign'); // "instagram"
  params.get('popular'); // "true"
  params.toString(); // "campaign=instagram&popular=true"
  ```

  좀 더 정확하게 말하면, 직렬화된 문자열 버전을 `search`라고 하고 파싱된 버전을 `search params`라고 부른다. 하지만 이 정도로 구분 짓는 것이 중요하지 않을 때는 두 용어를 비슷하게 사용하는 것이 일반적이다.

- **Location Hash**

  **URL**에서 `hash`는 현재 페이지에서 `Scroll` 위치를 나타낸다. `window.history.pushState`가 도입되기 전에 웹 개발자들은 URL의 해시 부분으로 클라이언트 사이드 라우팅을 수행했는데 이는 당시 서버에게 새로운 요청을 만들어내지 않고 **URL**을 조작할 수 있는 유일한 방법이었기 때문이다. 하지만 오늘날 우리는 hash를 애시당초 설계된 목적으로 사용할 수 있다(**Scroll Position**).

- **Location State**

  `window.history.pushState()` **API**는 왜 **push state**라고 부를까 궁금해 해본적이 있을 수도 있다. **State**? 우리는 그저 **URL**을 변경하는게 아니었나? 그렇게 따지면 `history.push`가 되어야 하지 않을까? **API**가 설계될 당시에 그 자리에 있던게 아니라서 왜 `state`라는 말이 포함되어 있는지는 모른지만 이 기능이 브라우저가 가진 멋진 기능이라는 것엔 변함이 없다.

  ```jsx
  window.history.pushState('look ma!', undefined, '/contact');
  window.history.state; // 'look ma!'
  // user clicks back
  window.history.state; // undefined
  // user clicks forward
  window.history.state; // 'look ma!'
  ```

  **React Router**는 브라우저 기능을 활용하여 이를 약간 추상화하고 `history` 대신에 `location`이라는 값을 사용한다.
  `location.state`는 **URL**에 값을 집어 넣는 대신 프로그래머만 알고 있는 것을 URL에 숨긴다는 것을 제외하면 `location.hash` 나 `location.search` 처럼 생각할 수 있다.
  **location state**의 좋은 **use-case**는 다음과 같다.

  - 사용자가 어디에서 왔는지 다음 페이지에서 알리고 UI를 분기한다. The most popular implementation here is showing a record in a modal if the user clicked on an item in a grid view, but if they show up to the URL directly, show the record in its own layout (pinterest, old instagram).
  - 다음 스크린에 리스트의 일부 레코드 정보를 보내어 데이터의 일부를 즉시 렌더링할 수 있고 나머지 데이터는 이후에 데이터 패칭한다.

  **location state**는 두 가지 방법(`<Link>` 또는 `navigate`)을 통해 설정할 수 있다.

  ```jsx
  <Link to="/pins/123" state={{ fromDashboard: true }} />;

  let navigate = useNavigate();
  navigate('/users/123', { state: partialUser });
  ```

  위 정보는 다음 페이지에서 `useLocation`을 통해 접근할 수 있다.

  ```jsx
  let location = useLocation();
  location.state;
  ```

  > ✋ **주의**  
  > **location state**의 값들은 직렬화되기 때문에 `new Date`와 같은 값들은 문자열로 변환될 것이다.

- **Location Key**

  각 `location`은 고유의 `key` 값을 가진다. 이는 **`location` 기반 스크롤 관리**, **클라이언트 사이드 데이터 캐싱** 등과 같은 응용 케이스에서 유용하다. 새로운 `location`은 각기 다른 고유의 `key` 값을 갖기 때문에 일반 객체, `new Map` 또는 `locationStorage`에 정보를 저장하는 추상화(abstractions)를 구축할 수 있다.

  예를 들어, 아주 일반적인 클라이언트 사이드 데이터 캐시가 `location key`를 값으로 저장하고 **URL**로 데이터를 패칭하며 사용자가 뒤로 가기를 클릭했을 때 데이터 패칭을 건너뛸 수 있다고 해보자.

  ```jsx
  let cache = new Map();

  function useFakeFetch(URL) {
    let location = useLocation;
    let cacheKey = location.key + URL;
    let cached = cache.get(cacheKey);

    let [data, setData] = useState(() => {
      // initialize from the cache
      return cached || null;
    });

    let [state, setState] = useState(() => {
      // avoid the fetch if cached
      return cached ? 'done' : 'loading';
    });

    useEffect(() => {
      if (state === 'loading') {
        let controller = new AbortController();
        fetch(URL, { signal: controller.signal })
          .then((res) => res.json());
          .then((data) => {
            if (controller.signal.aborted) return;
            // set the cache
            cache.set(cacheKey, data);
            setData(data);
          })
        return () => contoller.abort();
      }
    }, [state, cacheKey]);

    useEffect(() => {
      setState('loading');
    }, [URL]);

    return data;
  }
  ```

## Matching

초기 렌더링 그리고 **history stack**이 변경될 때 **React Router**는 렌더링할 `matches`(`route`들이 담긴 배열) 가져오기 위하여 **route config**와 `location`을 매칭할 것이다.

### Defining Routes

**route config**는 다음과 같이 생긴 `route`의 트리이다.

```jsx
<Routes>
  <Route path="/" element={<App />}>
    <Route index element={<Home />} />
    <Route path="teams" element={<Teams />}>
      <Route path=":teamId" element={<Team />} />
      <Route path=":teamId/edit" element={<EditTeam />} />
      <Route path="new" element={<NewTeamForm />} />
      <Route index element={<LeagueStandings />} />
    </Route>
  </Route>
  <Route element={<PageLayout />}>
    <Route path="/privacy" element={<Privacy />} />
    <Route path="/tos" element={<Tos />} />
  </Route>
  <Route path="contact-us" element={<Contact />} />
</Routes>
```

`<Routes>` 컴포넌트는 `props.children`을 통하여 재귀하고 `props`를 제거하여 다음과 같은 객체를 생성한다.

```jsx
let routes = [
  {
    element: <App />,
    path: '/',
    children: [
      {
        index: true,
        element: <Home />,
      },
      {
        path: 'teams',
        element: <Teams />,
        children: [
          {
            index: true,
            element: <LeagueStandings />,
          },
          {
            path: ':teamId',
            element: <Team />,
          },
          {
            path: ':teamId/edit',
            element: <EditTeam />,
          },
          {
            path: 'new',
            element: <NewTeamForm />,
          },
        ],
      },
    ],
  },
  {
    element: <PageLayout />,
    children: [
      {
        element: <Privacy />,
        path: '/privacy',
      },
      {
        element: <Tos />,
        path: '/tos',
      },
    ],
  },
  {
    element: <Contact />,
    path: '/contact-us',
  },
];
```

`<Routes>` 대신에 `useRoutes('routesGoHere')` 훅을 사용해서 **Router**를 생성할 수도 있다. 이게 `<Routes>`가 하는 전부다.

위 예제에서 볼 수 있듯이 `route`는 `:teamId/edit`과 같은 다수의 `segment`로 정의하거나 `:teamId`와 같이 하나로 정의할 수도 있다. **route config**의 분기점 아래에 있는 `segment`들이 함께 모여 `route`에 대한 최종적인 **path pattern**을 만들어 낸다.

### Match Params

`:teamId`와 같은 `segment`를 **path pattern** 중에서 **dynamic segment**라고 부르며 이는 **URL**에 정적으로 매칭되지 않고 동적으로 매칭된다는 것을 의미한다. `:teamId`에는 어떤 값이 와도 상관없다. `/teams/123` 또는 `/teams/cupcakes` 모두 일치할 것이다. 파싱된 값은 URL params라고 부른다. 따라서, `teamId`에 담기는 `param`의 값은 "123" 또는 "cupcakes"가 될 것이다. 이를 애플리케이션의 렌더링 섹션에서 어떻게 사용하는지 살펴보자.

```jsx
['/', '/teams', '/teams/:teamId', '/teams/:teamId/edit', '/teams/new', '/privacy', '/tos', '/contact-us'];
```

`/teams/new` 이라는 **URL**이 있을 때 이는 어떤 패턴과 일치할까?

총 두 가지 패턴과 일치한다.

```
/teams/new
/teams/:teamId
```

**React Router**는 두 가지 중에서 어떤 패턴을 사용해야 할지 결정을 내려야 한다. 클라이언트 사이드와 서버 사이드를 포함한 많은 라우터들이 정의된 순서에 따라서 패턴을 처리한다. 정의되어 있는 순서에서 먼저 일치하면 장땡이다. 따라서, 정의된 순서대로 일치한다고 했을 때 위 예제에선 `"/"` 패턴과 일치할 것이며 `<Home />` 컴포넌트가 렌더링될 것이다.

이는 우리가 원하는 바가 아니다. 이러한 방식의 Router를 사용하려면 우리가 원하는 결과를 얻기 위하여 경로를 완벽하게 정렬해야 한다. **React Router** **v6** 이전까지 이 방식을 사용했지만 **v6**부터는 더 똑똑해진 방식을 사용한다.

위 패턴들을 봤을 때, `/teams/new` 패턴은 `/teams/new` **URL**과 일치하길 바란다는 것을 직관적으로 알 수 있다. 해당 패턴과 URL이 완벽하게 일치한다는 것을 React Router도 알고 있다. 따라서, 매칭할 때 `segment`의 수, **static segment**, **dynamic segment**, **star pattern** 등에 따라 `route`의 순위를 매기고 이 중 가장 일치하는 `route`를 선택한다. 경로를 정렬하는 것에 대해 전혀 생각할 필요가 없게 됐다.

### Pathless Routes

이전 예제에서 다음과 같은 이상한 `route`를 본 적이 있을 것이다.

```jsx
<Route index element={<Home />} />
<Route index element={<LeagueStadings />} />
<Route element={<PageLayout />} />
```

`path` 속성이 존재하지 않는데 어떻게 `route`가 될 수 있을까? 이는 **React Router**에서 `route`라는 단어가 느슨하게(pretty loosely) 사용되는 곳이다. `<Home />`과 `<LeagueStadings />`은 **index route**이며 `<PageLayout />`는 **layout route**다. 렌더링 섹션에서 두 `route`가 어떻게 동작하는지 살펴볼 것이다. 두 `route` 모두 `route` 매칭과는 관련이 없다.

### Route Matches

`route`가 **URL**과 일치하는 경우 `match` 객체로 표현된다. `<Route path=":teamId" element={<Team/>}/>`에 대한 `match` 객체는 다음과 같이 생겼다.

```jsx
{
  pathname: '/teams/firebirds',
  params: {
    teamId: 'firebirds'
  },
  route: {
    element: <Team />,
    path: ':teamId'
  }
}
```

`pathname`은 현재 `route`와 일치하는 **URL**의 일부분을 가지고 있다. `params`는 **dynamic segment**와 일치하는 파싱된 값을 가지고 있다. `params`의 객체 `key` 값은 `segment`의 이름과 직접적으로 매핑된다. 위 예쩨의 경우 `:teamId`와 `params.teamId`가 그러하다.

`route`는 트리 구조를 갖고 있기 때문에 단일 **URL**의 경우 트리의 전체 분기와 일치할 수 있다. `/team/firebirds` **URL**의 경우 다음과 같은 **route branch**와 일치한다.

```jsx
<Routes>
  <Route path="/" element={<App />}>
    <Route index element={<Home />} />
    <Route path="teams" element={<Teams />}>
      <Route path=":teamId" element={<Team />} />
      <Route path=":teamId/edit" element={<EditTeam />} />
      <Route path="new" element={<NewTeamForm />} />
      <Route index element={<LeagueStandings />} />
    </Route>
  </Route>
  <Route element={<PageLayout />}>
    <Route path="/privacy" element={<Privacy />} />
    <Route path="/tos" element={<Tos />} />
  </Route>
  <Route path="contact-us" element={<Contact />} />
</Routes>
```

**React Router**는 위와 같이 일치하는 `route`들과 **URL**을 배열로 만들어 중첩된 `route`와 일치하는 중첩 **UI**를 렌더링할 수 있다.

```jsx
[
  {
    pathname: '/',
    params: null,
    route: {
      element: <App />,
      path: '/',
    },
  },
  {
    pathname: '/teams',
    params: null,
    route: {
      element: <Teams />,
      path: 'teams',
    },
  },
  {
    pathname: '/teams/firebirds',
    params: {
      teamId: 'firebirds',
    },
    route: {
      element: <Team />,
      path: ':teamId',
    },
  },
];
```

## Rendering

마지막 개념인 렌더링이다. 애플리케이션의 `entry`가 다음과 같이 생겼다고 해보자.

```jsx
const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(
  <BrowserRouter>
    <Routes>
      <Route path="/" element={<App />}>
        <Route index element={<Home />} />
        <Route path="teams" element={<Teams />}>
          <Route path=":teamId" element={<Team />} />
          <Route path="new" element={<NewTeamForm />} />
          <Route index element={<LeagueStandings />} />
        </Route>
      </Route>
      <Route element={<PageLayout />}>
        <Route path="/privacy" element={<Privacy />} />
        <Route path="/tos" element={<Tos />} />
      </Route>
      <Route path="contact-us" element={<Contact />} />
    </Routes>
  </BrowserRouter>
);
```

`/teams/firebirds` **URL**을 예시로 다시 사용해보자. `<Routes>`는 **route config**에서 `location`을 매칭하고 `match` 객체들이 담긴 배열을 가져온 뒤 다음과 같은 **React** `element` 트리를 렌더링할 것이다.

```jsx
<App>
  <Teams>
    <Team />
  </Team>
</App>
```

부모 `route` 요소 내에서 렌더링된 각 `match`는 강력한 추상화(abstraction)다. 대부분의 웹사이트와 애플리케이션은 이 특성을 공유한다. - boxes inside of boxes inside of boxes, each with a navigation section that changes a child section of the page.

### Outlets

This nested element tree won't happen automatically. `<Routes>`는 첫 번째로 매칭되는 요소를 렌더링할 것이다(위 예제에서는 `<App />`). 이 다음 일치하는 요소는 `<Teams>`이다. 해당 요소를 렌더링하기 위해서는 `App` 내에 `Outlet`이 필요하다.

```jsx
function App() {
  return (
    <div>
      <GlobalNav />
      <Outlet />
      <GlobalFooter />
    </div>
  );
}
```

`Outlet` 컴포넌트는 다음에 매치되는 요소를 항상 렌더링할 것이다. 이 말인 즉슨 `<Teams>` 또한 `<Team />`을 렌더링하기 위해선 `Outlet` 컴포넌트가 필요하다.

만약 **URL**이 `/contact-us` 였다면 **element tree**는 다음과 같이 변경된다.

```jsx
<ContactForm />
```

왜냐하면 **contact form**은 `<App>`이라는 `route`의 하위에 위치하지 않았기 때문이다.

만약 **URL**이 `/teams/firebirds/edit` 였다면 **element tree**는 다음과 같이 변경된다.

```jsx
<App>
  <Teams>
    <EditTeam />
  </Teams>
</App>
```

`Outlet`은 일치하는 새로운 하위 요소로 기존 하위 요소를 교체하지만 부모의 레이아웃은 유지된다. It's subtle but very effective at cleaning up your components.

### Index Routes

Remember the [route config](https://reactrouter.com/en/main/start/concepts#route-config) for `/teams`

```jsx
<Route path="teams" element={<Teams />}>
  <Route path=":teamId" element={<Team />} />
  <Route path="new" element={<NewTeamForm />} />
  <Route index element={<LeagueStandings />} />
</Route>
```

만약 **URL**이 `/teams/firebirds` 였다면 **element tree**는 다음과 같이 변경된다.

```jsx
<App>
  <Teams>
    <Team />
  </Teams>
</App>
```

하지만 만약 URL이 `/teams` 였다면 **element tree**는 다음과 같다.

```jsx
<App>
  <Teams>
    <LeagueStandings />
  </Teams>
</App>
```

`LeagueStandings` ? 어떻게 여기서 `<Route index element={<LeagueStandings>}/>`이 튀어나오는 걸까? `LeagueStandings`엔 `path` 속성이 없는데 말이다.

그 이유는 `LeagueStandings`가 **index route**이기 때문이다. **index route**는 **URL**이 부모 `route`의 경로와 일치할 때 부모 `route`의 `Outlet`에 렌더링된다.

이렇게 생각해보자. 만약 어떤 하위 `route`의 경로와도 일치하지 않는다면 `<Outlet>`은 UI에 아무것도 렌더링하지 않을 것이다.

```jsx
<App>
  <Teams />
</App>
```

If all the teams are in a list on the left then an empty outlet means you've got a blank page on the right! 이때 **UI**가 빈공간을 채울 무언가가 필요로 할텐데 이를 **index route**가 도와줄 수 있다.

Another way to think of an index route is that it's the default child route when the parent matches but none of its children do.

유저 인터페이스에 따라 **index route**가 필요하지 않을 수도 있지만 부모 `route`에 일종의 **persistent navigation**가 있는 경우 사용자가 항목 중 하나를 클릭하지 않았을 때 공간을 채우기 위해 아마 **index route**를 사용하게 될 것이다.

> Depending on the user interface, you might not need an index route, but if there is any sort of persistent navigation in the parent route you'll most likely want index route to fill the space when the user hasn't clicked one of the items yet.

### Layout Routes

아직 다루지 않은 부분이 하나 있는데 바로 **URL**이 `/privacy` 일 경우이다. **route config**를 다시 살펴봤을 때 강조된 부분이 `/privacy`와 매칭되는 `route`들이다.

```jsx
<Routes>
  <Route path="/" element={<App />}>
    <Route index element={<Home />} />
    <Route path="teams" element={<Teams />}>
      <Route path=":teamId" element={<Team />} />
      <Route path=":teamId/edit" element={<EditTeam />} />
      <Route path="new" element={<NewTeamForm />} />
      <Route index element={<LeagueStandings />} />
    </Route>
  </Route>
  <Route element={<PageLayout />}>
    .
    <Route path="/privacy" element={<Privacy />} />
    <Route path="/tos" element={<Tos />} />
  </Route>
  <Route path="contact-us" element={<Contact />} />
</Routes>
```

**element tree**는 다음과 같이 렌더링된다.

```jsx
<PageLayout>
  <Privacy />
</PageLayout>
```

`PageLayout`이라는 `route`는 확실히 이상하다. 이 `route`는 **layout route**라고 부르는데 이는 매칭에 전혀 참여하지 않는다. **layout route**는 오직 다수의 하위 요소를 동일한 레이아웃으로 간단히 래핑하기 위해 존재한다. 만약 layout route를 사용하고 싶지 않은 경우 레이아웃 처리를 위해선 두 가지 방법을 사용할 수 있다.

- `route`가 레이아웃 작업을 하거나
- 애플리케이션 전체에 걸쳐 레이아웃 컴포넌트를 통해 반복적으로 일일히 작업하거나

다음과 같이 코드를 작성할 수 도 있겠지만 **layout route**의 사용을 더 추천한다.

`This is a bad example`

```jsx
<Route path="/" element={<App />}>
    <Route index element={<Home />} />
    <Route path="teams" element={<Teams />}>
      <Route path=":teamId" element={<Team />} />
      <Route path=":teamId/edit" element={<EditTeam />} />
      <Route path="new" element={<NewTeamForm />} />
      <Route index element={<LeagueStandings />} />
    </Route>
  </Route>
  <Route
    path="/privacy"
    element={
      <PageLayout>
        <Privacy />
      </PageLayout>
    }
  />
  <Route
    path="/tos"
    element={
      <PageLayout>
        <Tos />
      </PageLayout>
    }
  />
  <Route path="contact-us" element={<Contact />} />
</Routes>
```

So, yeah, the semantics of a layout "route" is a bit silly since it has nothing to do with the URL matching, but it's just too convenient to disallow.

## Navigating

**URL**이 변경되는 것을 `navigation`이라고 부른다. **React Router**에서 `navigate` 하기 위한 방법은 2가지가 있다.

- `<Link>`
- `navigate`

### Link

This is the primary means of navigation. `<Link>` 컴포넌트를 렌더링하는 것은 사용자로 하여금 이를 클릭하여 **URL**을 변경할 수 있도록 한다. **React Router**는 브라우저의 기본 동작을 막고 **history stack**에 새로운 항목을 집어넣도록 `history`에게 이를 알린다. `location`이 변경되고 새로운 `matches`(URL과 일치하는 `route`의 배열)가 렌더링될 것이다.

However, links are accessible in that they

- **keyboard**, **focusability**, **SEO** 등과 같은 기본적인 접근성 문제들이 충족되도록 여전히 `<a href>`를 렌더링한다.
- 만약 "**새 탭에서 열기**"를 오른쪽 클릭 또는 `command`/`control` 클릭을 통해서 진행할 경우 브라우저의 기본 동작을 막지 않는다.

중첩 `route`는 단순히 레이아웃 렌더링에 관한 것이 아니며 "**relative links**"를 가능하게도 한다.

Consider our `teams` route from before

```jsx
<Route path="teams" element={<Teams />}>
  <Route path=":teamId" element={<Team />} />
</Route>
```

`<Teams>` 컴포넌트는 다음과 같은 링크를 통하여 렌더링할 수도 있다.

```jsx
<Link to="psg" />
<Link to="new" />
```

전체 `path`는 `/teams/psg` 그리고 `/teams/new`가 될 것이다. `Link` 컴포넌트는 렌더링되는 `route`를 상속받는다. 이는 `route` 컴포넌트가 애플리케이션 내에 나머지 `route`들에 대해서 실제로 알 필요 없게 해준다. 매우 많은 양의 링크일지라도 한 `segment`만 더 깊게 이동한다. You can rearrange your whole [route config](https://reactrouter.com/en/main/start/concepts#route-config) and these links will likely still work just fine. This is very valuable when building out a site in the beginning and the designs and layouts are shifting around.

### Navigate Function

이 함수는 `useNavigate` 훅이 반환하며 프로그래머가 언제든지 **URL**을 변경할 수 있도록 해준다.

`timeout` 내에서도 사용이 가능하다.

```jsx
let navigate = useNavigate();

useEffect(() => {
  setTimeout(() => {
    navigate('/logout');
  }, 3000);
}, []);
```

또는 `form`이 `submit` 되고 나서도 가능하다.

```jsx
<form onSubmit={event => {
  event.preventDefault();
  let data = new FormData(event.target);
  let urlEncoded = new URLSearchParams(data);
  navigate('/create', { state: urlEncoded });
}}>
```

`Link` 컴포넌트와 같이 `navigate`도 중첩 `route`에 한해서 "**relative links**"가 가능하다.

```jsx
navigate('psg');
```

`<Link>` 컴포넌트 대신 `navigate`를 사용할 때는 합당한 이유가 있어야 한다.

다음은 올바르지 못한 예다.

`This is a bad example`

```jsx
<li onClick={() => navigate('/somewhere')} />
```

`link`와 `form`을 제외하고 **URL**을 변경하는 상호 작용은 거의 없다. 왜냐하면 이는 접근성과 사용자 경험성에 대한 복잡성을 야기하기 때문이다.

## Data Access

마지막으로 애플리케이션은 전체 **UI**를 구축하기 위해 **React Router**에게 몇 가지 정보를 필요로 하는데 이러한 정보들은 **React Router**가 몇 가지 훅을 통해서 제공한다.

```jsx
let location = useLocation();
let urlParams = useParams();
let [urlSearchParams] = useSearchParams();
```

## Review

Let's put it all together from the top!

1. You render your app:

   ```jsx
   const root = ReactDOM.createRoot(document.getElementById('root'));
   root.render(
     <BrowserRouter>
       <Routes>
         <Route path="/" element={<App />}>
           <Route index element={<Home />} />
           <Route path="teams" element={<Teams />}>
             <Route path=":teamId" element={<Team />} />
             <Route path="new" element={<NewTeamForm />} />
             <Route index element={<LeagueStandings />} />
           </Route>
         </Route>
         <Route element={<PageLayout />}>
           <Route path="/privacy" element={<Privacy />} />
           <Route path="/tos" element={<Tos />} />
         </Route>
         <Route path="contact-us" element={<Contact />} />
       </Routes>
     </BrowserRouter>
   );
   ```

2. `<BrowserRouter>`는 `history`를 만들고 `state`에 초기 `location`을 집어 넣은 뒤 **URL**을 구독한다.
3. `<Routes>`는 **route config**를 구축하기 위해 **child route**들을 재귀하며 `location`에 일치하는 `route`를 찾아 `matches`를 생성한 뒤 일치하는 `match` 중 첫 번째 `route` 요소를 렌더링한다.
4. 각 부모 `route` 내에 `Outlet`을 렌더링한다.
5. `Outlet`은 일치하는 `match` 중 3번에서 렌더링한 요소를 제외한 다음 `route` 요소를 렌더링한다.
6. 사용자는 링크를 클릭한다.
7. 링크는 `naviagate()`를 호출한다.
8. history는 URL을 변경하고 이를 `<BrowserRouter>`에게 알린다.
9. `<BrowserRouter>`는 렌더링한 후에 2번부터 다시 시작한다.
