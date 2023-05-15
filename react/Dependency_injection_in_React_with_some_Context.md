# Dependency injection in React with some Context

> 원본 글
> https://blog.codeminer42.com/di-with-some-context/

**목차**

- [Dependency injection in React with some Context](#dependency-injection-in-react-with-some-context)
  - [개요](#개요)
  - [Dependency injection이란 무엇인가?](#dependency-injection이란-무엇인가)
  - [Context API](#context-api)
    - [The problem](#the-problem)
  - [DI와 Context API를 통한 해결 방법](#di와-context-api를-통한-해결-방법)
  - [해결 방법 테스트하기](#해결-방법-테스트하기)

## 개요

의존성 주입(Dependency injection)은 애플리케이션의 두 컴포넌트 간의 상호 의존성을 제거하는 데 사용되는 기술이지만, 어떤 이유에서인지 React 애플리케이션에서는 일반적으로 사용되지 않는다. 이 글에서는 이를 수행하는 몇 가지 편리한 방법에 대해 알아본다. 하지만 React와 관련된 부분을 파헤치기 전에 의존성 주입이 무엇인지 이해해 보자.

## Dependency injection이란 무엇인가?

앞서 설명한 것처럼 의존성 주입(일명 DI)은 애플리케이션의 두 단위 간의 상호 의존성을 제거하는 데 사용되는 기술이다. 일반적으로 유닛을 생성하는 동안 의존성을 인수로 전달한다.

의존성의 생성에 대한 예시와 DI를 사용하여 의존성을 해결하는 방법을 살펴보자.

```typescript
// createUser.ts

import { User, create, isValidUser } from '../User';
import { UserRepository } from '../repositories/UserRepository'; // coupling
import { MailService } from '../services/MailService'; // coupling

const createUser = async (userData: User) => {
  const userRepository = new UserRepository();
  const mailService = new MailService();

  if (!isValidUser(userData)) {
    throw new Error('Error');
  }

  const user = create(userData);

  await userRepository.save(user);
  await mailService.sendWelcomeMail(user.email);

  return user;
};

export { createUser };
```

위 코드는 일부 애플리케이션에서 유저를 생성하는 플로우를 담고 있다. 유저를 생성하기 위해 `createUser` 파일은 `UserRepository와` `MailService를` import하여 플로우를 구축한다. 주석으로 **coupling**이라고 표시한 것처럼 `createUser` 함수는 `UserRepository와` `MailService의` 구체적인 구현에 의존하기 때문에 코드의 유연성이 떨어질 뿐만 아니라 테스트 작성을 어렵게 만든다.

여기서 코드가 서로 의존하는 것은 당연하지만 **제어의 반전(inversion of control)**을 통해 **coupling**을 피할 수 있다. 예를 들어, `createUser`가 `createUser`를 호출한 측에 새로운 유저를 생성할 것으로 예상되는 것을 알려주도록 강제하여 **coupling**을 피할 수 있다.

의존성 주입은 코드에서 **coupling**을 제거하는 데 도움을 준다. 어떻게 도움을 주는지 한번 살펴보자.

```typescript
// @aplication/useCases/CreateUser.ts

import { User, create, isValidUser } from '../domain/user/User';

type Dependencies = {
  userRepository: { save: (user: User) => void };
  mailService: { sendWelcomeMail: (email: User.email) => void };
};

const makeCreateUser =
  ({ userRepository, mailService }: Dependencies) =>
  async (userData: User) => {
    if (!isValidUser(userData)) {
      throw new Error('Error');
    }

    const user = create(userData);

    await userRepository.save(user);
    await mailService.sendWelcomeMail(user.email);

    return user;
  };

export { makeCreateUser };
```

위 코드는 앞선 예제 코드를 함께 이야기 해보고자 사항을 반영한 것이다. 예시 코드에는 다음과 같은 변경 사항을 포함하고 있다.

- 유저 생성에 필요한 모듈을 선언한 `Dependencies` 타입을 만들었다.
- 의존성과 `userData`를 전달받을 수 있도록 고차 함수로 변경했다.

이제 이러한 변경 사항으로 인해 `createUser`는 인터페이스에만 의존하며 더 이상 유저 도메인 외에는 아무것도 import 해올 필요가 없다. 또한 고차 함수F의 이름을 `makeCreateUser`로 지정한 것을 눈치채셨을 텐데, 이는 해당 고차 함수가 `createUser`를 빌드하는 방법, 즉 의존성과 내부 작동 방식만 알고 있기 때문에 일부러 그렇게 지었다.

그렇다면 의존성을 어디에 주입해야 할까? 이 위치를 **container** 또는 **composition root**라고도 한다.

```typescript
// src/container.ts

import { makeCreateUser } from '../application/useCases/CreateUser';

import { userRepository } from '../infra/repositories/UserRepository';
import { mailServices } from '../services/MailServices';

const container = {
  createUser: makeCreateUser({ userRepository, mailServices }),
};

export { container };
```

👍 이제 `userData`만 인수로 전달하면 `createUser` 함수를 사용할 수 있다.

```typescript
// ../controllers/userController.ts

import { app } from '../hhtp/server';
import { createUser } from '../container';

app.post('/users', async (req, res) => {
  const { name, email } = req.body;

  /*
    container 파일에서 의존성을 주입했으므로 우리는 userData, 즉 이름과
    이메일만 createUser에 전달할 수 있다.
  */
  const user = await createUser({ name, email });

  res.status(201).json(user);
});
```

## Context API

이제 DI에 대해 더 잘 이해하게 되었으니, 더 흥미진진하게 Context API가 프론트엔드 애플리케이션에서 동일한 접근 방식을 구현하는 데 어떻게 도움이 되는지 알아보자.

### The problem

앞선 예제에서와 마찬가지로 Context API를 DI 컨테이너로 사용하여 문제를 해결하고 이를 해결하는 방법을 알아보자.

```tsx
// HomePage.tsx
import { useState } from 'React';

import api from '../apiClient';
import { User } from '../domain/User';

const HomePage = () => {
  const [user, setUser] = useState<User | null>(null);

  const createUser = () => (user: User) => {
    const newUser = api
      .post('/user', user)
      .then((data) => setUser(data))
      .catch((e) => console.log(e));
  };

  return (
    <div class="home-container">
      /// ...more UI hre
      <button class="btn" onClick={createUser({ name, email })}>
        Create user
      </button>
      // ... more UI here
    </div>
  );
};
```

앞선 예제에서 살펴본 것처럼 이 서비스(API)의 변경 사항이 HomePage 컴포넌트에 영향을 미치는 방식으로 HomePage와 API라는 두 컴포넌트 사이에 **coupling**이 있다는 것을 눈치챘을 것이다.

게다가 이 컴포넌트에는 많은 책임이 있다. 이상적으로 React의 컴포넌트는 UI만 처리해야 한다. 컴포넌트를 분리하고 더 간결하게 만들 수 있는 두 가지 개념이 있는데, 이는 바로 **Dependency Injection(의존성 주입)**과 **Custom Hooks**다.

## DI와 Context API를 통한 해결 방법

먼저 `makeCreateUser` 함수를 다시 생성해야 하지만 이번에는 다음과 같이 API 의존성을 삽입해야 한다.

```typescript
// @aplication/useCases/CreateUser.ts

import { User, create, isValidUser } from '../domain/user/User';

type Dependencies = {
  api: ApiClientType;
};

const makeCreateUser =
  ({ api }: Dependencies) =>
  async (userData: User) => {
    if (!isValidUser(user)) {
      throw new Error('Error');
    }

    const user = await api.post('/users', userData);

    if (!user) {
      throw new Error("Can't create user");
    }

    return user;
  };

export { makeCreateUser };
```

이제 컨테이너 파일에서 의존성을 조립할 수 있다.

```typescript
// container.ts
import { User } from '../domain/User';
import { api } from '../apiClient';
import { makeCreateUser } from '../useCases/CreateUser';

export const container = {
  createUser: makeCreateUser({ api }),
};

export const Container = typeof container;
```

마지막으로 애플리케이션에 해당 함수를 제공하기 위한 Context를 만든다.

```tsx
// ContainerContext.tsx
import { createContext } from 'React';
import { container, Container } from '../container';

export const ContainerContext = createContext<Container>(container);

const ContainerProvider = ({ children }: { children: React.ReactNode }) => {
  return <ContainerContext.Provider value={container}>{children}</ContainerContext.Provider>;
};
```

이제 이를 HomePage 컴포넌트에서 사용해보자.

```tsx
// HomePage.tsx
import { useContext } from 'React';
import { ContainerContext } from '../container';

const HomePage = () => {
  // more stuff here ...
  const { createUser } = useContext(ContainerContext);
  // more stuff here ...

  return (
    <div class="home-container">
      <button class="btn" onClick={() => createUser({ name, email })}>
        Create user
      </button>
      // ... more UI here
    </div>
  );
};
```

잠깐, 커스텀 훅은 어떻게 사용하는거지? 라고 아마 떠올렸을 것이다. 앞서 말했듯이 React의 컴포넌트는 UI만 처리해야 하며, 적절한 레이어는 모든 비즈니스 규칙을 처리해야 한다.

React Hook 덕분에 우리는 로직과 비즈니스 규칙을 어려움 없이 추상화할 수 있다. 새 훅을 생성하려면 React 문서에 설명된 몇 가지 규칙을 준수해야 한다. 우리의 경우, 커스텀 훅은 Context API 자체 내에서 생성될 것이다.

```tsx
// ContainerContext.tsx
import { createContext, useContext } from "React";
import { container, Container } from '../container'

export const ContainerContext = createContext<Container>(container)

const ContainerProvider = ({ children }: { children React.ReactNode }) => {

  return (
    <ContainerContext.Provider value={container}>
      {children}
    </ContainerContext.Provider>
  );
};

export const useContainer = () => useContext(ContainerContext);
```

이제 커스텀 훅도 HomePage 컴포넌트에서 사용해보자.

```tsx
// HomePage.tsx
import { useContainer } from '../container';

const HomePage = () => {
  // more stuff here ...
  const { createUser } = useContainer();
  // more stuff here ...

  return (
    <div class="home-container">
      <button class="btn" onClick={() => createUser({ name, email })}>
        Create user
      </button>
      // ... resto da UI
    </div>
  );
};
```

## 해결 방법 테스트하기

이 접근 방식을 사용하면 타사 라이브러리 없이도 원하는 만큼 많은 시나리오에서 테스트를 작성할 수 있다(복잡성과 가독성이 떨어질 수 있음).

```typescript
// HomePage.ts
import { act } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

import { HomePage } from './HomePage';
import { ApiClientType } from '../../apiClient';
import { ContainerContext } from '../ContainerContext';
import { makeCreateUser } from '../../useCases/CreateUser';

const api: ApiClientType = jest.fn().mockReturnValue({
  post: jest.fn(),
});

const createUser = makeCreateUser({ api });

const fakeContainer = { createUser };

describe('<HomePage />', () => {
  it('creates a user', async () => {
    const { findByRole, findByTestId } = render(
      <ContainerContext.Provider value={fakeContainer}>
        <HomePage />
      </ContainerContext.Provider>
    );

    const nameInput = findByTestId('sign-up-name-input');
    const emailInput = findByTestId('sign-up-email-input');
    const signUpButton = findByRole('button', { name: /Create Account/ });

    act(() => {
      userEvent.type(nameInput, 'Taeyoung Jang');
      userEvent.type(emailInput, 'xoxodudwkd@gmail.com');

      userEvent.click(signUpButton);
    });

    expect(await findByText('Welcome, Taeyoung Jang')).toBeInTheDocument();
  });
});
```
