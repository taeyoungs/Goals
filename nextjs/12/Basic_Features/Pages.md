# Basic Features: Pages

> 원본 글  
> https://nextjs.org/docs/basic-features/pages

**목차**

- [Basic Features: Pages](#basic-features-pages)
  - [개요](#개요)
    - [Pages with Dynamic Routes](#pages-with-dynamic-routes)
  - [Pre-rendering](#pre-rendering)
    - [Two forms of Pre-rendering](#two-forms-of-pre-rendering)
  - [Static Generation (Recommended)](#static-generation-recommended)
    - [Static Generation without data](#static-generation-without-data)
    - [Static Generation with data](#static-generation-with-data)
    - [When Should I use Static Generation?](#when-should-i-use-static-generation)
  - [Server-side Rendering](#server-side-rendering)
  - [요약](#요약)

## 개요

> Next.js 다음 버전에서는 라우팅 방식이 바뀔 예정 (물론 현재 라우팅 방식도 여전히 지원)  
> https://nextjs.org/blog/layouts-rfc?ck_subscriber_id=1780635791

`Next.js`에서 페이지를 구현하는 방식은 `pages` 디렉토리 하위에서 `js`, `jsx`, `ts`, `tsx`에 해당하는 파일 확장자로 `React Component`를 `export`하는 방식이다.

예를 들어, 다음과 같이 `pages/about.js`를 생성한 후에 해당 파일에서 `React Component`를 다음과 같이 `export`한다면 `about` 페이지에 접근할 수 있을 것이다.

```jsx
function About() {
  return <div>About</div>;
}

export default About;
```

### Pages with Dynamic Routes

`Next.js`는 동적 라우팅을 지원한다. 예를 들어, 파일을 `pages/posts/[id].js`와 같이 생성한다면 `posts/1`, `posts/2` 등과 같은 경로로 접근할 수 있게 된다.

## Pre-rendering

기본적으로 `Next.js`는 모든 페이지를 `pre-rendering`한다. 이말인 즉슨 클라이언트 사이드에서 자바스크립트로 모든 것을 하는 대신에 Next.js가 미리 각 페이지를 위한 HTML을 만들어 낸다는 것이다. `Pre-rendering`은 더 좋은 성능과 SEO를 얻을 수 있게 만든다.

미리 생성된 HTML들은 각 페이지에서 필요한 최소한의 자바스크립트 코드와 연결된다. 브라우저에 의해 페이지가 불러와질 때 자바스크립트 코드가 돌아가고 이는 페이지를 인터렉티브하게 만들어 준다. (이러한 과정을 `hydration`이라고 부른다)

### Two forms of Pre-rendering

`Next.js`에는 두 가지 형태의 `Pre-rendering` 방식을 가지며 이는 `Static Generation`과 `Server Side Rendering`이다. 두 방식의 차이점은 페이지를 위한 HTML이 생성되는 시점이다.

- **Static Generation (추천)**: HTML은 빌드 시에 생성되며 매 요청마다 이를 재사용한다.
- **Server-side Rendering**: HTML이 매 요청마다 생성된다.

중요한 것은 `Next.js`가 각 페이지에 사용하고 싶은 `pre-rendering` 방식을 선택할 수 있게 만들어 주는 것이다. 이로 인해 우리는 대부분의 페이지에는 `Static Generation`을 사용하고 이를 제외한 나머지 페이지에는 `Server-side Rendering`을 사용하는 **하이브리드** `Next.js` 애플리케이션을 만들 수 있다.

Next.js 측은 퍼포먼스 이유로 Server-side Rendering 보다는 Static Generation을 사용하는 것을 추천한다. 정적으로 생성된 페이지는 추가적인 설정 작업없이 CDN에 의한 캐시 처리로 높은 퍼포먼스를 발휘할 수 있다. 하지만 몇몇 케이스에서는 `Server-side Rendering`이 유일한 선택지일 수도 있다.

우리는 `Static Generation` 또는 `Server-side Rendering`과 함께 `Client-side Rendering` 또한 사용할 수 있다. 이는 어떤 부분에서는 완전히 자바스크립트에 의해 클라이언트 사이드에서 페이지를 렌더링시킬 수 있다는 말이기도 하다.

## Static Generation (Recommended)

만약 페이지를 `Static Generation`한다면 해당 페이지의 HTML은 빌드 시에 생성될 것이다. 이는 `production` 상황에서 `next build` 명령어를 실행시킬 때 페이지의 HTML이 생성된다는 뜻이다. 이렇게 생성된 HTML은 매 요청시마다 재사용될 것이며 이는 `CDN`에 의해서 캐시 처리될 수 있다.

`Next.js`에서는 **데이터가 있던 없던** 페이지를 정적으로 생성할 수 있다. 그러면 각 상황을 살펴보자.

### Static Generation without data

기본적으로 `Next.js`는 데이터 패칭없이 `Static Generation`을 이용하여 페이지를 `pre-rendering`한다.
예를 들어

```jsx
function About() {
  return <div>About</div>;
}

export default About;
```

위 예제를 보면 미리 렌더링하기 위해 가져올 어떠한 외부 데이터도 존재하지 않는다. 이러한 케이스에서 `Next.js`는 빌드 시에 각 페에지마다 하나의 HTML 파일을 만들어 낸다.

### Static Generation with data

몇몇 페이지는 미리 렌더링할 외부 데이터를 패칭하는 과정이 필요하다. 아래에는 두 가지 시나리오가 있으며 하나 또는 두 가지 시나리오 모두 적용될 수가 있다. 각 케이스에서 우리는 `Next.js`에서 제공되는 다음의 함수들을 사용해야 한다.

- **Scenario 1: Your page content depends on external data**

  예를 들어, 우리의 블로그가 CMS(여기서 CMS란 [컨텐츠 관리 시스템](https://ecommerce-platforms.com/ko/glossary/content-management-system-cms)를 뜻한다)로부터 블로그 포스팅 리스트를 패칭해와야 하는 상황이라고 해보자.

  ```jsx
  // TODO: Need to fetch `posts` (by calling some API endpoint)
  //       before this page can be pre-rendered.
  function Blog({ posts }) {
    return (
      <ul>
        {posts.map((post) =>
          <li>{post.titls}</li>
        ))}
      </ul>
    )
  }

  export default Blog;
  ```

  `pre-render`할 데이터를 패칭하기 위해 `Next.js`에선 우리에게 `getStaticProps`라고 불리는 `async` 함수를 제공하며 이는 같은 파일 내에서 `export` 해야 한다. 이 함수는 빌드 시점에 호출되며 `pre-render`때 패칭해온 데이터를 페이지의 `props` 속성에 집어 넣어준다.

  ```jsx
  function Blog({ posts }) {
    // Render posts
  }

  // This function gets called at build time
  export async function getStaticProps() {
    // Call an external API endpoint to get posts
    const res = await fetch('https://.../posts');
    const posts = await res.json();

    // By returning { props: { posts } }, the Blog componen
    // will receive `posts` as a prop at build time
    return {
      props: {
        posts,
      },
    };
  }
  ```

- **Scenario 2: Your page paths depend on external data**

  `Next.js`는 동적 라우팅과 함께 페이지를 생성할 수 있게 해준다. 예를 들어, 우리는 `pages/posts/[id].js`와 같이 파일을 생성하면 `id`에 기반하는 개별 블로그 포스팅을 볼 수 있게 된다. 이는 우리에게 `posts/1`로 접근할 때 `id: 1`과 함께 블로그 포스팅을 볼 수 있게 만들어 준다.
  하지만 빌드 시간에 `pre-render` 하려는 `id` 값은 외부 데이터에 따라 달라질 수 있다.
  예를 들어, 우리가 데이터베이스에 오직 하나의 블로그 포스팅(`id`값이 1인)만 추가한 상황이라고 가정해보자. 이 상황에서는 빌드 시 `posts/1`만 `pre-render`되길 원할 것이다.
  나중에 우리가 두 번째 포스팅(`id`값이 2인)을 추가했다고 해보자. 그러고 나면 우리는 `posts/2` 또한 `pre-render`되길 원할 것이다.
  따라서, 우리의 페이지 경로들은 외부의 데이터에 따라 `pre-rendering`된다. 이를 다루기 위해서 Next.js는 우리에게 동적 페이지를 위한 `getStaticPaths`라고 불리는 `async` 함수를 제공하며 이를 `export` 할 수 있게 해준다. (현재의 케이스에서 동적 페이지란 `pages/posts/[id].js`를 말한다) 이 함수는 빌드 시점에 호출되며 `pre-render`하고 싶은 경로를 지정할 수 있게 해준다.

  ```jsx
  // This function gets called at build time
  export async function getStaticPaths() {
    // Call an external API endpoint to get posts
    const res = await fetch('https://.../posts');
    const posts = await res.json();

    // Get the paths we want to pre-render based on posts
    const paths = posts.map((post) => ({
      params: { id: post.id },
    }));

    // We'll pre-render only these paths at build time
    // { fallback: false } means other routes should 404
    return { paths, fallback: false };
  }
  ```

  또한 `pages/posts/[id].js` 안에서, 우리는 `id`에 해당하는 `post` 데이터를 패칭하고 이를 `pre-rendering`된 페이지에서 사용할 수 있도록 `getStaticProps`를 `export` 할 수도 있다.

  ```jsx
  function Post({ post }) {
    // Render post ...
  }

  export async function getStaticPaths() {
    // ...
  }

  // This also gets called at build time
  export async function getStaticProps({ params }) {
    // params contains the post "id".
    // If the route is like /posts/1, then params.id is 1
    const res = await fetch(`https://.../posts/${params.id}`);
    const post = await res.json();

    // Pass post data to the page via props
    return { props: { post } };
  }

  export default Post;
  ```

### When Should I use Static Generation?

Next.js 측은 가능하면 언제나 `Static Generation`(데이터가 있든 없든)을 사용하는 것을 추천한다. 왜냐하면 우리의 페이지가 오직 한번만 만들어 질 수도 있고 매 요청마다 서버에서 페이지를 새로 렌더링하는 것보다 CDN을 통해 처리되는 것이 훨씬 퍼포먼스적으로 빠를 수 있기 때문이다.

우리는 다음 목록을 포함한 다양한 타입의 페이지에서 Static Generation을 사용할 수 있다.

- Marketing pages
- Blog posts and portfolios
- E-commerce product listings
- Help and documentation

우리는 먼저 우리 스스로에게 물어봐야만 한다. `사용자가 요청하기 전에 특정 페이지를 pre-rendering 할 수 있는가?` 만약 `예`라고 대답할 수 있다면 `Static Generation`을 선택해야한다.

반면에 만약 사용자가 요청하기 전에 특정 페이지를 `pre-rendering` 할 수 없다면 `Static Generation`은 좋은 생각이 아니다. 만약 우리의 페이지가 빈번하게 업데이트되는 데이터를 보여줘야 한다면 페이지의 컨텐츠는 매 요청마다 바뀌어야 할 것이다.

이러한 케이스들에서는 다음과 같은 선택지가 있다.

- **`Static Generation`과 `Client-side Rendering` 사용하기**

  몇몇 페이지에서는 `pre-rendering`을 건너뛰고 이를 클라이언트 사이드 자바스크립트를 이용하여 컨텐츠를 채우도록 할 수도 있다.

- **`Server-side Rendering` 사용하기**

  `Server-side Rendering`을 사용하면 Next.js는 매 요청마다 페이지를 새로 `pre-rendering`한다. 이는 `CDN`에 의해서 캐시될 수 없기 때문에(매번 새롭게 만들어지므로 캐시 처리가 의미가 없다) 느릴 수도 있지만 `pre-rendering`된 페이지들은 항상 최신 상태를 유지할 것이다.

## Server-side Rendering

`SSR` 또는 `Dynamic Rendering`이라고도 불린다.

만약 페이지가 Server-side Rendering을 사용하고 있다면, 해당 페이지의 HTML은 매 요청마다 새롭게 생성된다.

페이지를 `Server-side Rendering` 하기 위해서는 `gerServerSideProps`라고 불리는 `async` 함수를 `export` 해야 한다. 이 함수는 매 요청마다 서버에서 호출될 것이다.

예를 들어, 우리의 페이지가 빈번하게 업데이트되는 데이터(외부 API로부터 패칭된 데이터)를 `pre-rendering` 하고 있다고 가정해보자. 우리는 `getStaticServerProps`를 사용하여 데이터를 패칭할 수 있으며 이를 `Page`에 전달할 수 있다.

```jsx
function Page({ data }) {
  // Render data...
}

// This gets called on every request
export async function getServerSideProps() {
  // Fetch data fron external API
  const res = await fetch(`https://.../data`);
  const data = await res.json();

  // Pass data to the page via props
  return { props: { data } };
}

export default Page;
```

위에서 볼 수 있듯이 `getServerSideProps`는 `getStaticProps`와 비슷하다. 여기서 두 함수의 차이점은 `getServerSideProps`은 빌드 시점에 실행되는 대신에 매 요청마다 실행된다는 것이다.

## 요약

---

우리는 지금까지 `Next.js`에서 제공하는 두 가지 `pre-rendering` 방식에 대해 살펴봤다.

- **Static Generation (Recommended)**

  이 방식에서는 HTML이 빌드 시점에 생성되며 매 요청마다 이를 재사용한다. 페이지가 `Static Generation`을 사용하게 만들기 위해서는 페이지 컴포넌트를 `export` 하면서 `getStaticProps` 함수도 `export` 해야 한다. (필요하다면 `getStaticPaths`까지) 이러한 방식은 사용자의 요청에 앞서 `pre-rendering` 해야 하는 페이지에 적합하다. 또한 추가적인 데이터를 가져오는 `Client-side Rendering`도 같이 사용할 수 있다.

- **Server-side Rendering**

  이 방식에서는 매 요청마다 HTML은 새롭게 생성된다. 페이지가 `Server-side Rendering`을 사용하게 만들기 위해서는 `getServerSideProps` 함수를 `export` 해야 한다. `Server-side Rendering`은 퍼포먼스 측면에서 `Static Generation`보다 느리기 때문에 반드시 필요한 경우에만 사용해야 한다.
