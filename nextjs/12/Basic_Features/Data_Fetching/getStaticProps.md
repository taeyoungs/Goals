# getStaticProps

**목차**

- [getStaticProps](#getstaticprops)
  - [개요](#개요)
  - [When should I use getStaticProps?](#when-should-i-use-getstaticprops)
  - [When does getStaticProps run](#when-does-getstaticprops-run)
  - [Using getStaticProps to fetch data from a CMS](#using-getstaticprops-to-fetch-data-from-a-cms)
  - [Write server-side code directly](#write-server-side-code-directly)
  - [Statically generates both HTML and JSON](#statically-generates-both-html-and-json)
  - [Where can I use getStaticProps](#where-can-i-use-getstaticprops)

## 개요

`getStaticProps`(**Static Site Generation**)라고 불리는 함수를 페이지 컴포넌트 파일에서 `export` 할 경우 `Next.js`는 `getStaticProps`에서 반환된 `props`를 사용하여 빌드 시에 페이지를 `pre-rendering` 할 것이다.

```jsx
export async function getStaticProps(0 {
  return {
    props: {}, // will be passed to the page component as props
  }
}
```

## When should I use getStaticProps?

만약 다음과 같은 상황이라면 `getStaticProps`를 사용해야 한다.

- 페이지를 렌더링하는데 필요한 데이터를 사용자의 요청 이전 빌드 시에 사용할 수 있는 경우
- **headless CMS**로부터 데이터를 가져오는 경우
- 페이지가 SEO를 위해 pre-rendering 되어야만 하고 매우 빨라야 하는 경우
  `getStaticProps`는 `HTML`와 `JSON` 파일을 생성하며 이 둘은 모두 `CDN`에 의해 캐시 처리될 수 있다.
- 데이터가 공개적으로 캐시 처리될 수 있는 경우. 이 조건은 미들웨어를 사용하여 경로를 다시 작성하여 특정 상황에서 우회할 수 있다.

## When does getStaticProps run

`getStaticProps`은 항상 서버에서 실행되며 절대 클라이언트에서 실행되지 않는다. 이 또한 앞서 정리한 두 가지 함수와 동일하게 클라이언트 사이드 번들 파일 내에서는 사라진다.

- `getStaticProps`는 항상 빌드되는 동안에 실행된다.
- `revalidate`를 사용할 때는 백그라운드 상에서 실행된다.
- `unstable_revalidate`를 사용할 때는 백그라운드 상에서 요청할 때 실행된다.

`Incremental Static Regeneration`와 결합할 때 `getStaticProps`는 최신 상태가 아닌 페이지가 다시 검증될 때 백그라운드 상에서 실행될 것이며 이로 인해 최신 상태의 페이지가 브라우저에 전달될 것이다.

`getStaticProps`는 정적 **HTML**을 생성하므로 들어오는 요청(예를 들어 쿼리 파라미터 또는 **HTTP** **header** 정보)에 접근할 수는 없다. 만약 페이지 요청에 접근해야 하는 경우 `getStaticProps`에 미들웨어를 추가하는 것을 고려해보자.

## Using getStaticProps to fetch data from a CMS

다음 예제는 **CMS**로부터 블로그 포스팅 목록을 패칭하는 방법에 대한 것이다.

```jsx
// posts will be populated at build time by getStaticProps()
function Blog({ posts }) {
  return (
    <ul>
      {posts.map((post) => (
        <li>{post.title}</li>
      ))}
    </ul>
  );
}

// This function gets called at build time on server-side.
// It won't be called on client-side, so you can even do
// direct database queries
export async function getStaticProps() {
  // Call an external API endpoint to get posts.
  // You can use any data fetching library
  const res = await fetch('https://.../posts');
  const posts = await res.json();

  // By returning { props: { posts } }, the Blog component
  // will receive `posts` as a prop at build time
  return {
    props: {
      posts,
    },
  };
}

export default Blog;
```

## Write server-side code directly

`getStaticProps`는 오직 서버 사이드에서만 실행되며 절대 클라이언트 사이드에서 실행되지 않을 것이다. 브라우저를 위한 JS 번들 파일에도 포함되지 않기 때문에 브라우저로 보내지 않고 직접 데이터베이스 쿼리를 작성할 수도 있다.

즉, `getStaticProps`로부터 API 경로를 가져오는 대신에 직접 서버 사이드 코드를 작성할 수 있다는 말이다.

다음 예제를 확인해보자. **API** 경로는 **CMS**로부터 특정 데이터를 패칭하는데 사용되고 있다. 해당 **API** 경로는 `getStaticProps`에서 직접 호출된다. 이는 추가적인 요청을 만들어내고 퍼포먼스를 떨어트린다. 대신에 lib/ 저장소를 이용하여 **CMS**로부터 데이터를 가져오는 로직을 공유할 수 있게 만들 수 있다. 그러면 이는 `getStaticProps`와 공유할 수 있다.

```jsx
// lib/fetch-posts.js

// The following function is shared
// with getStaticProps and API routes
// from a `lib/` directory
export async function loadPosts() {
  // Call an external API endpoint to get posts
  const res = await fetch('https://.../posts/');
  const data = await res.json();

  return data;
}

// pages/blog.js
import { loadPosts } from '../lib/load-posts';

// This function runs only on the server side
export async function getStaticProps() {
  // Instead of fetching your `/api` route you can call the same
  // function directly in `getStaticProps`
  const posts = await loadPosts();

  // Props returned will be passed to the page component
  return { props: { posts } };
}
```

API Route를 사용해서 데이터를 패칭하지 않는 경우엔 fetch API를 직접 `getStaticProps` 내에서 사용하는 방법으로 대신할 수도 있다.

## Statically generates both HTML and JSON

페이지를 `getStaticProps`와 함께 빌드 시에 `pre-rendering` 할 때 `Next.js`는 페이지에 필요한 **HTML** 파일 외에도 `getStaticProps`의 실행 결과를 포함하는 **JSON** 파일을 생성한다.

이 JSON 파일은 `next/link` 또는 `next/router`를 통한 클라이언트 사이드 라우팅에 사용될 것이다. 우리가 `getStaticProps`를 사용하여 `pre-rendering`된 페이지로 이동했을 때, `Next.js`는 **JSON** 파일(빌드시에 미리 만들어진)을 가져오고 이를 페이지 컴포넌트의 `props`로 사용한다. 이말인 즉슨 클라이언트 사이드 페이지 전환은 `getStaticProps`를 호출하는 것이 아니라 `export` 된 **JSON**이 사용된다는 것을 뜻한다.

`Incremental Static Generation`을 사용할 때 `getStaticProps`는 클라이언트 사이드 네비게이션에 필요한 **JSON** 파일을 생성하기 위해 백그라운드에서 실행된다. 이를 동일한 페이지에 대한 다수의 요청으로 볼 수도 있으나 이는 최종 사용자가 느끼는 성능에 아무런 영향을 미치지 않는다.

## Where can I use getStaticProps

`getStaticProps`는 오직 페이지 컴포넌트 파일 내에서 `export` 되어야만 한다. 페이지 컴포넌트 파일이 아닌 곳에서 이를 `export` 할 수 없다.

이러한 제한의 이유 중 하나는 페이지가 렌더링 되기 전에 React에 필요한 모든 데이터가 있어야하기 때문이다.

또한, `getStaticProps`를 독립된 함수 형태로 `export` 해야한다. 페이지 컴포넌트 내에 속성으로 추가할 경우 정상적으로 동작하지 않을 것이다.
