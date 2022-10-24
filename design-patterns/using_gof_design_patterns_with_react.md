### 원본 글

https://medium.com/bitsrc/using-gof-design-patterns-with-react-c334f3ea3147

---

디자인 패턴은 소프트웨어 새발에서 주된 이슈에 대해 재사용 가능하고 입증된 해결책을 제공한다. 디자인 패턴을 사용하는 것은 개발에 있어서 수없이 많은 시간을 아낄 수 있게 하며 프로덕션 기능을 더 빠르게 만들 수 있게 도움을 준다.

개발자로서 디자인 패턴들을 이해하는 것은 필수적이며 해당 패턴들을 어떻게 사용하냐에 따라 개발 생산성이 달라질 수 있다.

따라서, 이 문서는 Gof 디자인 패턴과 이를 React로 어떻게 구현하는 지에 대해 이야기한다.

### 목차

1. [Design Pattern 1: Singleton](#design-pattern-1-singleton)
2. [Design Pattern 2: Observer](#design-pattern-2-observer)
3. [Design Pattern 3: Facade](#design-pattern-3-facade)

## Desgin Patterns: Gang-of-Four

앞서 언급했듯이, 디자인 패턴은 소프트웨어 개발에 일반적인 디자인 문제에 대하여 언어 독립적이고 재사용 가능하며 확장성 있는 해결책을 제시하는데 도움을 준다. The Gang-of-Four refers to four authors who wrote the book — “[Design Patterns: Elements of Reusable Object-Oriented Software](<http://en.wikipedia.org/wiki/Design_Patterns_(book)>)” which fundamentally proposes 23 design patterns based on the principles of Object-Oriented Programming.

4명의 저자는 디자인 패턴을 3가지 그룹으로 나눴다.

1. **Creational Patterns**: 이 패턴들은 객체를 인터턴스화하여 제공하는 것(컴포넌트를 생성)을 통해 구현부를 숨긴다.
2. **Structural Patterns**: 이 패턴들은 클래스들(컴포넌트들) 간에 관계를 효과적으로 정의하는 데 도움을 준다.
3. **Behavioral Patterns**: 이 패턴들은 컴포넌트들 간의 통신과 관련이 있다.

이러한 모든 패턴들은 언어 독립적이다. 개발자들은 React나 Vue와 같은 프레임워크 내에서 확장성 있고 읽을 수 있는 React Component를 디자인하는 데 이러한 패턴들을 적용할 수 있다.

## Implementing Design Patterns with React

이제 개발자들이 왜 디자인 패턴들을 이해해야 하고 사용해야만 하는 지 이해했다. 그러므로 이제 이러한 디자인 패턴들을 React 애플리케이션에서 어떻게 사용할 수 있는지 살펴보자.

아래에 나올 예제를 위해, React와 TypeScript의 사용을 권장하는데 이는 인터페이스, 상속 그리고 타입 지원을 하는 정적 언어가 이러한 패턴을 빠르게 구현하는데 도움을 주기 때문이다.

### Pre-requisites

진행하기에 앞서 Node.js가 설치되어 있는지 확인해보자. 설치를 확인했으면 다음 명령어를 통해 프로젝트를 생성하자.

```shell
npx create-react-app react-design-patterns --template typescript
```

## Design Pattern 1: Singleton

### What is a Singleton?

싱글톤 패턴은 creational 디자인 패턴 중 하나로 클래스가 인스턴스를 오직 하나만 생성하도록 만드는 것을 보장하는 접근법이다.

### When should you use Singleton?

예를 들어, React 애플리케이션에서 로그인한 유저 정보를 보관하고 있는 전역 설정 객체를 활용하고 있다고 가정해보자. 이상적으로, 애플리케이션은 애플리케이션 전체에 걸쳐서 현재 유저 정보를 얻기 위해서는 이 정보를 반드시 재사용해야 한다.

### Implementing Singleton with React

우리가 싱글톤 패턴을 구현할 때는 반드시 두 가지 영역을 다뤄야 한다.

1. 객체는 반드시 싱글톤 인스턴스여야 한다.
2. 단일 지점에서만 접근 가능해야만 한다.

```typescript
// use type
interface SingletonConfigValues {
  name?: string;
  id?: string;
  email?: string;
  token?: string;
}

// private variable accessible only within current module
let loggedInUserStore: SingletonConfigValues | undefined = undefined;
const userActions = {
  // configure the single instance logged in user
  initializeUser: (user: SignletonConfigValues) => {
    loggedInUserStore = user;
  },
  // retireve the single instance of the logged in user
  getUserInformation: () => {
    return loggedInUserStore;
  },
};

// export the methods so that components can use the single instance
export default userActions;
```

위 예제에서는 싱글 인스턴스의 private 변수의 상태를 관리하는 타입스크립트 모듈을 보여주고 있다. 여기서 loggedInUserStore는 두 가지 메서드를 사용하고 있는데 하나는 싱글 인스턴스를 가져오는 `getUserInformation` 메서드이고 다른 하나는 싱글 인스턴스를 설정하는 `initializeUser` 메서드다.

이 모듈은 사용자 정보를 전역으로 관리하기 위한 싱글톤 접근법을 제공한다. 컴포넌트 내에서 이 모듈을 사용하기 위해서는 `actions`를 import하고 메서드들을 아래와 같이 호출하면 된다.

```typescript
import { useEffect, useState } from "react";
// obtain the single instance - directly user var is not accessible (private)
import userRetriever from "../../store/custom-singleton";

export const ComponentA = () => {
  const [userInformation, setUserInformation] = useState<any>(undefined);

  useEffect(() => {
    setUserInformation(userRetriever.getUserInformation());
  }, [userInformation]);

  return (
    <div>
      <h1>Component A</h1>
      <p>
        {userInformation && (
          <>
            <span>Name: {userInformation.name}</span>
            <br />
            <span>Id: {userInformation.id}</span>
            <br />
            <span>Email: {userInformation.email}</span>
            <br />
            <span>Token: {userInformation.token}</span>
          </>
        )}
      </p>
    </div>
  );
};
```

위 예제는 싱글 인스턴스를 가져오고 있는 컴포넌트를 보여준다.

```typescript
import { useEffect } from "react";
import { ComponentA } from "./component-a";
// retrieve the single instance - directly user var is not accessible (private)
import userInformation from "../../store/custom-singleton";

export const Singleton = () => {
  useEffect(() => {
    // initalize the single instance variable
    userInformation.initializeUser({
      name: "John Doe",
      id: "123",
      email: "johndoe@gmail.com",
      token: "123456789",
    });
  }, []);

  return (
    <div className="App">
      <div>
        <ComponentA />
      </div>
    </div>
  );
};
```

위 예제는 useEffect 내에서 싱글 인스턴스에 사용자 정보를 구성하는 것을 보여준다.

### Pros of Singleton pattern

- 코드를 확장성 있고 유지보수 용이하게 만들어 준다.
- 다수의 컴포넌트 전체에 결쳐 코드 중복을 피할 수 있게 해준다.

### Cons of Singleton parttern

- 싱글톤의 가장 큰 단점은 우리가 작성한 코드를 테스트 하기 어렵게 만든다는 것이다. 싱글톤 패턴으로 인한 전역 상태로 인해 단위 테스트 간에 상태가 유지되기 때문에 코드를 테스트하기 어렵게 만들 수 밖에 없다.

## Design Pattern 2: Observer

### What is the Observer?

Observer 패턴이란 관찰하고 있는 대상의 변경 사항을 관찰자들(Observers, 구독하고 이를 바라보고 있는 대상들)에게 알리는 subscription mechanism을 정의하는데 도움을 주는 behavioral pattern이다.

### When should you use Observer?

React 애플리케이션에서 Observer 패턴을 사용하기 가장 좋은 상황은 애플리케이션이 브라우저 이벤트에 의존하고 있을 때이다. 예를 들어, React 애플리케이션이 인터넷 상황에 따라 기능의 전환이 필요한 컴포넌트를 가지고 있다고 가정해보자. 컴포넌트는 브라우저의 인터넷 이벤트를 구독한 뒤 이에 대한 알림을 받고 알림에 따라 상태를 업데이트 할 수 있다.

### Implementing Observer in React

Observer 패턴을 구현할 때

1. 첫 번째로 컴포넌트는 이벤트를 구독해야 한다.
2. 그리고 나서, 컴포넌트가 언마운트 됐을 때 해당 이벤트에 대한 구독을 해제해야만 한다.

다음 예제를 함께 살펴보자.

```typescript
import { useEffect, useState } from "react";

export const InternetAvailabilityObserver = () => {
  const [isOnline, setOnline] = useState<any>(navigator.onLine);

  useEffect(() => {
    // subscribe to two events -> online and offline

    // when online -> set online to true
    // when offline -> set online to false
    window.addEventListener("online", () => setOnline(true));
    window.addEventListener("offline", () => setOnline(false));

    return () => {
      // when component gets unmounted, remove the event listeners to prevent memory leaks
      window.removeEventListener("online", () => setOnline(true));
      window.removeEventListener("offline", () => setOnline(false));
    };
  });

  return (
    <>
      <h1>Internet Availability Observer</h1>
      <p>
        {isOnline ? (
          <span>
            You are <b>online</b>
          </span>
        ) : (
          <span>
            You are <b>offline</b>
          </span>
        )}
      </p>
    </>
  );
};
```

위 예제는 React에서 Observer 패턴의 구현을 보여준다. 첫 번째로, 두 가지 대상에 대해 구독을 생성한다(online과 offline 이벤트). 그리고 구독한 대상에 대해 변경 사항이 발생했을 때, 콜백 함수들이 컴포넌트 상태의 업데이트와 함께 실행된다.

### Pros of Observer pattern

- 구독 대상(subject)과 관찰자(observer) 간의 코드를 분리하여 유지보수 효율을 높인다.
- 다수의 관찰자들이 어느 때나 구독 대상에 대해서 구독을 하거나 구독을 해지할 수 있다.

### Cons of Observer pattern

- 만약 구독이 해지되지 않는다면, React는 계속해서 변경 사항의 발생에 대해 기다리고 있을 것이고 이는 메모리 손실을 발생시키며 퍼포먼스 측면에서의 손해이다.

## Design Pattern 3: Facade

### What is the Facade pattern?

`Facade` 패턴은 **structural pattern**으로 하나의 API를 통해 다수의 컴포넌트들이 상호 작용하는 간단한 방법을 제공하는데 초점이 맞춰져있다. 게다가 이 패턴은 내부 동작의 복잡함을 숨겨 코드를 더 읽기 쉽게 만든다. 이 패턴으로 구현하기 위해선, 인터페이스와 클래스가 필요하지만 React에서 `Facade` 패턴의 기본 원칙을 활용하여 구현할 수도 있다.

### When should you use Facade?

React 애플리케이션에선 애플리케이션의 복잡도가 올라갈 때 `Facade` 패턴을 통한 구현을 고민해볼 수 있다. 예를 들어, React 내에서 Facade 패턴을 통한 구현의 좋은 예는 컴포넌트 상태를 관리해야 할 때이다.

예를 들어, 애플리케이션 사용자(`add`, `remove`, `fetch`, `fetch one`)를 관리하는 컴포넌트가 있다고 해보자. 일반적으로 이러한 경우 많은 상태 변수와 HTTP 요청 로직으로 인해 코드는 어지럽혀지며 가독성이 떨어지게 된다. 아래 코드 예제를 보자.

```typescript
export const NoFacade: FC = () => {
	const [users, setUsers] = useState<any>([]);

	const fetchUsers = useCallback(async () => {
		const resp = await axios.get(`/api/users`);
		setUsers(resp.data);
	}, []);

	useEffect(() => {
		fetchUsers();
	}, [fetchUsers]);

	const handleUserDelete = async (id: string) => {
		await axios.delete(`/api/users/${id}`);
		setUsers(users.filter((user: any) => user.id !== id));
	};

	const handleCreateUser = async (user: any) => {
		if (!users.find((u: any) => u.id === user.id) {
			await axios.post(`/api/users`, user);
			setUsers([...users, user]);
		} else {
			console.log('User already exists');
		}
	};

	return (
		<>
			<UserTable users={users} onDelete={handleUserDelete} />
			<UserCreateModal onCreate={handleCreateUser} />
		</>
	)
}
```

위 코드를 보면 HTTP 요청 로직과 상태 변수의 복잡함으로 인해 읽기가 쉽지가 않다. 하지만 개발자들은 HTTP 요청 로직들과 상태 변수들을 그룹화할 수 있으면 데이터를 다루는 하나의 API만 노출시킬 수 있다. 이러한 때가 바로 `Facade` 패턴을 사용할 수 있는 때이다.

React에서, 개발자들은 커스텀 훅을 통해 복잡한 코드를 그룹화하고 클라이언트가 사용할 메서드만 훅을 통해 노출함으로써 `Facade` 패턴을 구현할 수 있다.

### Implementing Facade in React

그러면 커스텀 훅을 통해 복잡한 코드 블럭을 대체해보자. 커스텀 훅은 사용자를 생성하고 제거하고 사용자 정보를 가져오는 메서드를 포함할 것이다. 컴포넌트는 이러한 기능들을 순차적으로 실행되는 복잡한 코드를 보지 않고도 메서드를 통해 사용할 수 있게 된다.

변경한 코드는 다음과 같다.

```typescript
import axios from "axios";
import { useState } from "react";

export const useFacadeUserAPI = () => {
  const [users, setUsers] = useState<any>([]);
  const [actionExecuting, setActionExecuting] = useState<boolean>(false);

  // expose one method to get users
  async function getUsers() {
    setActionExecuting(true);

    try {
      const resp = await axios.get("/api/users");
      setUsers(resp.data);
    } catch (err) {
      console.log(err);
    } finally {
      setActionExecuting(false);
    }
  }

  // expose method to create user
  async function createUser(user: any) {
    setActionExecuting(true);

    try {
      await axios.post("/api/users", user);
      setUsers([...users, user]);
    } catch {
      console.log(err);
    } finally {
      setActionExecuting(false);
    }
  }

  // expose method to delete a user
  async function deleteUser(id: string) {
    setActionExecuting(true);

    try {
      await axios.delete(`/api/users/${id}`);
      setUsers(users.filter((user: any) => user.id !== id));
    } catch {
      console.log(err);
    } finally {
      setActionExecuting(false);
    }
  }

  // return the methods that encapsulate the complex code
  // resulting in a clearner client code
  return {
    // the users
    users,
    // boolean to indicate if action occurs
    actionExecuting,
    // method to mutate the users via HTTP requests
    getUsers,
    createUsers,
    deleteUsers,
  };
};
```

위 코드를 보면 사용자를 관리하는 커스텀 훅인 것을 알 수 있다. 훅은 컴포넌트에서 API와 상호 작용하는데 사용되는 메서드들을 반환해준다.

커스텀 훅을 활용하여 업데이트한 컴포넌트 코드는 다음과 같다.

```typescript
export const Facade: FC = () = {
  const userFacade = userFacadeUserAPI();
  const { createUser, deleteUser, getUsers, users } = userFacade;

  const fetchUsers = useCallback(async () => {
	// replace with facade API method to simplify code
	await getUsers();
  }, [getUsers]);

  useEffect(() => {
	fetchUsers();
  }, [fetchUsers]);

  const handleUserDelete = async (id: string) => {
	// replace with a facade method to hide complex code required in deleting
	await deleteUser(id);
  };

  const handleCreateUser = async (user: any) => {
	// replace with a facade method to hide complex code required in creating
	await createUser(user);
  };

  return (
    <>
	  <UserTable users={users} onDelete={handleUserDelete} />
	  <UserCreateModal onCreate={handleCreateUser} />
	</>
  )
}
```

위 코드는 커스텀 훅을 바탕으로 컴포넌트 코드를 업데이트한 것이다. 코드를 통해 알 수 있듯이 커스텀 훅은 더 읽기 쉽고 복잡도가 낮은 코드를 만들며 컴포넌트로부터 복잡한 비즈니스 로직을 모두 숨긴다.

### Pros of Facade pattern

- 코드를 재사용 가능하게 만들며 코드의 중복을 피할 수 있게 도와준다.
- 컴포넌트들의 비즈니즈 로직을 분리하고 개발자에게 함수 단위의 단위 테스트 작성을 가능하게 만들어 준다.

### Cons of Facade pattern

- 기존 코드를 리팩토링하고 이를 커스텀 훅 내로 캡슐화하는 건 쉽지 않다. 하지만 이 패턴은 정확하게 구현됐을 경우 높은 재사용성과 확장성을 가진 코드를 만들어 준다.
