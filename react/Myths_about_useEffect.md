# Myths about useEffect

> 원본 글  
> https://epicreact.dev/myths-about-useeffect/

**목차**

- [Myths about useEffect](#myths-about-useeffect)
  - [개요](#개요)
  - [❌ Lifecycles ❌ ✅ Synchronize Side Effects ✅](#-lifecycles---synchronize-side-effects-)
  - [I can ignore eslint-plugin-react-hooks/exhaustive-deps ❌](#i-can-ignore-eslint-plugin-react-hooksexhaustive-deps-)
  - [One giant useEffect ❌](#one-giant-useeffect-)
  - [Needlessly externally defined functions ❌](#needlessly-externally-defined-functions-)
  - [Conclusion](#conclusion)

## 개요

I've taught React to tens of thousands of developers. Before and after hooks were released. One thing I've observed is people tend to struggle with the **`useEffect`**
 hook and there are some common hang-ups for them that I'd like to address here.

## ❌ Lifecycles ❌ ✅ Synchronize Side Effects ✅

필자가 본 사람들 중 `useEffect`에 대해서 애를 먹는 사람들은 **React** 클래스 컴포넌트와 `constructor`, `componentDidMount`, `componentDidUpdate` 그리고 `componentWillUnmount`와 같은 **lifecycle** 훅을 경험해본 개발자들이었다. 이러한 훅들을 함수 컴포넌트의 훅에 매핑하여 생각할 수도 있겠지만 이러면 안된다.

다음에 나올 재밌는 **UI**를 통해서 왜 그러면 안되는지 알아보자.

![https://d33wubrfki0l68.cloudfront.net/5ee9bfb60fddcc2cf1cd4bcf57f6a46de53de69d/a841a/09be13feda555c17404ce714f2d1f013/dogs.gif](https://d33wubrfki0l68.cloudfront.net/5ee9bfb60fddcc2cf1cd4bcf57f6a46de53de69d/a841a/09be13feda555c17404ce714f2d1f013/dogs.gif)

아래 `DogInfo` 컴포넌트를 클래스 컴포넌트로 구현한 코드가 있다.

```jsx
class DogInfo extends React.Component {
  controller = null;
  state = { dog: null };

  // 간결함을 위해 에러나 로딩 상태는 무시
  fetchDog() {
    this.controller?.abort();

    this.controller = new AbortController();
    getDog(this.props.dogId, { siginal: this.controller.signal }).then(
      (dog) => {
        this.state({ dog });
      },
      (error) => {
        // handle the error
      }
    );
  }
  componentDidMount() {
    this.fetchDog();
  }
  componentDidUpdate(prevProps) {
    // handle the dogId change
    if (prevProps.dogId !== this.props.dogId) {
      this.fetchDog();
    }
  }
  componentWillUnmount() {
    // cancel the request on unmount
    this.controller?.abort();
  }

  render() {
    return <div>{/* render dog's info */}</div>;
  }
}
```

위 코드는 앞서 나온 UI와 같은 상호 작용 케이스를 위한 표준 클래스 컴포넌트이다. 이는 `constructor`, `componentDidMount`, `componentDidUpdate` 그리고 `componentWillUnmount` **lifecycle** 훅을 사용하고 있다. 만약 이를 별 생각없이 함수 컴포넌트의 훅에 매핑하게 되면 다음과 같은 코드의 형태를 취하게 된다.

```jsx
function DogInfo({ dogId }) {
  const controllerRef = React.useRef(null);
  const [dog, setDog] = React.useState(null);

  function fetchDog() {
    controllerRef.current?.abort();

    controllerRef.current = new AbortController();
    getDog(dogId, { signal: controllerRef.current.signal }).then(
      (d) => setDog(d),
      (error) => {
        // handle the error
      }
    );
  }

  // didMount
  React.useEffect(() => {
    fetchDog();
    // eslint-disable-next-line react-hooks/exhaustive-deps
  }, []);

  // didUpdate
  const previousDogId = usePrevious(dogId);
  useUpdate(() => {
    if (previousDogId !== dogId) {
      fetchDog();
    }
  });

  // willUnmount
  React.useEffecy(() => {
    return () => {
      controllerRef.current?.abort();
    };
  }, []);

  return <div>{/* render dog's info */}</div>;
}

function usePrevious(value) {
  const ref = useRef();
  useEffect(() => {
    ref.current = value;
  }, [value]);
  return ref.current;
}
```

There are still some holdouts on hooks. If this is what I thought hooks meant, then I would be a hooks holdout too.

문제의 요점은 **`useEffect`는 lifecycle 훅이 아니라는 것**이다. `useEffect`는 애플리케이션의 상태값에 대한 사이드 이펙트를 동기화하기 위한 메커니즘이다.

그렇기에 위 예제에서 실질적으로 신경써야 할 부분은 "`**dogId`가 변했을 때 새로운 `dog`의 정보를 패칭한다.\*\*"다. 여기에 포커즈를 맞춘다면 위 케이스에서의 `useEffect`는 훨씬 간단해진다.

```jsx
function DogInfo({ dogId }) {
  const [dog, setDog] = React.useState(null);

  React.useEffect(() => {
    const controller = new AbortController();
    getDog(dogId, { signal: controller.signal }).then(
      (d) => setDog(d),
      (error) => {
        // handle the error
      }
    );
    return () => controller.abort();
  }, [dogId]);

  return <div>{/* render dog's info */}</div>;
}
```

훨씬 나아지지 않았나? **React** 팀이 훅을 소개했을 때 **React** 팀의 목표는 단순히 함수 컴포넌트에 **liftcycle**을 더하는 것이 아니었다. 그들의 실질적인 목표는 애플리케이션 사이드 이펙트에 대한 **mental model**의 근본적인 향상이었다. And they did.

Big time.

So remember this gem by [Ryan Florence](https://twitter.com/ryanflorence/status/1125041041063665666):

> 질문은 "**언제 `effect`가 수행되나요?**"가 아니라 "**`effect`는 어떤 상태값과 동기화 되나요?**"가 되어야 한다.
>
> `useEffect(fn) // all state`  
> `useEffect(fn, []) // no state`  
> `useEffect(fn, [these, states])`

## I can ignore eslint-plugin-react-hooks/exhaustive-deps ❌

음, 기술적으로는 가능하다. 그리고 때로는 위 규칙을 무시해도 될만한 이유가 생기기도 한다. 그러나 대부분의 경우 위 규칙을 무시하는건 좋지 못하며 규칙을 무시함으로 인해서 나중에 버그가 발생할 수도 있다. 필자는 사람들이 "오직 컴포넌트가 `mount` 할 때만 특정 작업을 수행하고 싶어하는 경우" 위 규칙을 무시하고 싶어한다는 것을 안다. 하지만 이러한 생각은 `useEffect`를 **lifecycle**와 연관지어서 생각하고 있다는 것인데 이는 틀렸다. 만약 `useEffect`의 콜백이 의존성을 가진다면 의존성에 변경 사항이 생길 때마다 해당 콜백이 다시 수행될 수 있어야 한다. 그렇지 않으면, 애플리케이션의 상태값이 사이드 이펙트와 동기화되지 않을 것이다.

간단히 말해서, 이 규칙을 무시할 경우 버그가 있을 수 있다. 따라서, 규칙을 무시하면 안된다.

## One giant useEffect ❌

Honestly, I don't see this one a whole lot, but I want to include it just in case. No shame if this is you. It happens.

필자가 **lifecycle** 보다 `useEffect`를 좋아하는 점 중 하나는 관심사를 분리할 수 있게 해준다는 것이다. 간단한 예제를 살펴보자.

![https://d33wubrfki0l68.cloudfront.net/c85252aea24a0cfe309fda387d4fced32de8e246/180fc/1bba39e6cf722fa6bebfb8c66a49a2ba/geo-chat.gif](https://d33wubrfki0l68.cloudfront.net/c85252aea24a0cfe309fda387d4fced32de8e246/180fc/1bba39e6cf722fa6bebfb8c66a49a2ba/geo-chat.gif)

위 데모에 대한 **pseudo code**다.

```jsx
class ChatFeed extends React.Component {
  componentDidMount() {
    this.subscribeToFeed();
    this.setDocumentTitle();
    this.subscribeToOnlineStatus();
    this.subscribeToGeLocation();
  }

  componentWillUnmount() {
    this.unsubscribeFromFeed();
    this.restoreDocumentTitle();
    this.unsubscribeFromOnlineStatus();
    this.unsubscribeFromGeoLocation();
  }

  componentDidUpdate(prevProps, prevState) {
    // ... compare props and re-subscribe etc.
  }

  render() {
    return <div>{/* chat app UI */}</div>;
  }
}
```

4개의 관심사가 보이는가? 이들은 모두 섞여있다. 만약 위 코드를 다른 것과 공유하고 싶다면 이는 난장판이 될 것이다. 필자가 말하고자 하는건 **render props**도 좋지만 **hooks**는 더 좋다.

필자는 바로 위 예제에서 나온 관심사를 모두 수행하는 거대한 `useEffect` 훅을 본 적이 있다.

```jsx
function ChatFeed() {
  React.useEffect(() => {
    // subscribe to feed
    // set document title
    // subscribe to online status
    // subscribe to geo location
    return () => {
      // unsubscribe from feed
      // restore document title
      // unsubscribe from online status
      // unsubscribe from geo location
    };
  });

  return <div>{/* chat app UI */}</div>;
}
```

이러한 거대한 `useEffect` 훅은 각 관심사가 가지는 콜백 함수들을 매우 복잡하게 만든다. 필자는 다른 접근 방법을 권장한다. 논리적인 관심사를 개별 훅으로 분리할 수 있다는 사실을 잊지 말자.

```jsx
function ChatFeed() {
  React.useEffect(() => {
    // subscribe to feed
    return () => {
      // unsubscribe from feed
    };
  });

  React.useEffect(() => {
    // set document title
    return () => {
      // restore document title
    };
  });

  React.useEffect(() => {
    // subscribe to outline status
    return () => {
      // unsubscribe from online status
    };
  });

  React.useEffect(() => {
    // subscribe to geo location
    return () => {
      // unsubscribe from geo location
    };
  });
}
```

이 접근 방법을 이용하면 필요하거나 하고 싶은 각 개별 관심사별로 커스텀 훅으로 훨씬 더 간단하게 관련된 코드를 추출할 수 있다.

```jsx
function ChatFeed() {
  // NOTE: this is pseudo code,
  // you'd likely need to pass values and assign return values
  useeFeedSubScription();
  useDocumentTitle();
  useOnlineStatus();
  useGeoLocation();
  return <div>{/* chat app UI */}</div>;
}
```

The **self-encapsulation** of hooks in general is a huge win. Let's make sure we take advantage of that.

## Needlessly externally defined functions ❌

이 또한 몇 번 본 적이 있다. 이번엔 수정 전/후 코드로 바로 알아보자.

```jsx
// before. Don't do this!
function DogInfo({ dogId }) {
  const [dog, setDog] = React.useState(null);
  const controllerRef = React.useRef(null);

  const fetchDog = React.useCallback((dogId) => {
    controllerRef.current?.abort();
    controllerRef.current = new AbortController();

    return getDog(dogId, { signal: controllerRef.signal }).then(
      (d) => setDog(d),
      (error) => {
        // handle the error
      }
    );
  }, []);

  React.useEffect(() => {
    fetchDog(dogId);

    return () => controller.current?.abort();
  }, [dogId, fetchDog]);

  return <div>{/* render dog's info */}</div>;
}
```

앞선 예제를 통해 이미 얼마나 코드가 간단해지는지 봤었지만 다시 한번 더 간단하게 만들어 보자.

```jsx
// before. Don't do this!
function DogInfo({ dogId }) {
  const [dog, setDog] = React.useState(null);

  const fetchDog = React.useCallback(
    (dogId) => {
      const controller = new AbortController();

      getDog(dogId, { signal: controller.signal }).then(
        (d) => setDog(d),
        (error) => {
          // handle the error
        }
      );
      return () => controller.abort();
    },
    [dogId]
  );

  return <div>{/* render dog's info */}</div>;
}
```

여기서 구체적으로 말하고자 하는 것은 `useEffect` 콜백 밖에서 `fetchDog`와 같은 함수를 정의하는 아이디어에 대해서다. 이런 함수는 `useEffect` 콜백 외부에 있기 때문에 **stale closure**를 피하기 위해 의존성 배열에 추가해야 한다. 그리고 또한 무한 루프를 피하기 위하여 해당 함수를 메모이제이션 해야한다. 그런 다음 **Abort Controller**에 대한 ref를 만들어내야 했다.

Phew, seems like a lot of work. 호출할 `effect`에 대한 함수를 정의해야만 한다면 `effect` 콜백 외부가 아니라 내부에서 정의하자.

## Conclusion

**Dan Abramov**가 `useEffect`와 같은 **hooks**을 소개했을 때, 그는 **React** 컴포넌트를 `atoms`, 그리고 **hooks**를 `electrons`에 비교했다(각각을 원자랑 전자에 비유한듯 하다). **hooks**는 꽤나 **low-level primitive**이며 **primitive**함이 훅을 강력하게 만들어준다. **primitive**의 장점은 **hooks** 이전에 꽤나 골치 아팠던 문제들을 **hooks** 위에 추상화를 구축함으로써 해결할 수 있다는 점이다. Since the release of hooks, we've seen an explosion of innovation and progress of good ideas and libraries built on top of this primitive which ultimately helps us develop better apps.
