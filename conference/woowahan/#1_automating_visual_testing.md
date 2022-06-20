## 목차

1. [소프트웨어 테스트란?](#1-소프트웨어-테스트란)
2. [어떤 테스트를 했는가?](#2-어떤-테스트를-했는가)
3. [어떻게 테스트했는가?](#3-어떻게-테스트했는가)
4. [테스트 구축하기](#4-테스팅-구축하기)
5. [테스트 구성하기](#5-테스트-구성하기)

## 1. 소프트웨어 테스트란?

널리 사용되기 전에 어떤 것의 품질, 성능 또는 신뢰성을 확립하기 위한 절차

ex.

- 화면이 잘 그려지는가?
- 기능이 잘 동작하는가?

### 안정성의 필요성

테스트가 필요한 이유? 안정성이 중요시 되는 이유? 모두 프로덕션 단으로 갔을 때 문제가 발생하면 안되기 때문이다. 현재 세미나에서 설명하고 있는 만다오 앱을 예로 들어보자 만다오를 통해 만든 이벤트 배너가 있다. 이 배너가 배포되고 프로덕션 단으로 갔을 때 과연 깨지지 않는다고 장담할 수 있을까? 적어도 어느 정도까지는 안전하다고 말할 수 있을까?

그래서 위에서 언급했던 `화면이 잘 그려지는가?`나 `기능이 잘 동작하는가?`에 대해 어느 정도의 보장이 필요한 것이다. 오류를 완벽하게 잡겠다는 게 아니다. 사람에 의해 만들어질 수 있는 에러를 어느 정도 선까지는 테스트 코드를 통해 막겠다는 것이다.

## 2. 어떤 테스트를 했는가?

### 시나리오 테스트

유저가 어떤 동작을 할지에 대한 시나리오를 작성하고 해당 시나리오를 테스트한다. 예를 들어

1. 이벤트 시작기간 선택
2. 이벤트 종료기간 선택
3. 이벤트 기간 확인
4. 쿠폰 생성
5. 쿠폰 기간 확인
6. ...

위와 같은 사용자가 이벤트 기간을 설정하는 시나리오를 작성하고 이에 따라 테스트를 한 뒤 결과물이 자신이 설정한 의도대로 쿠폰이 만들어지는지 검사한다.

### 시각적 회귀 테스트

렌더링 결과물이 기존과 똑같이 생성되었는지 테스트한다. 예를 들어 변경 전 상태의 렌더링 결과물 스냅샷을 가지고 있다가 새로 생성된 렌더링 결과물과 비교하여 변경점이 존재하는 지 확인한다.

## 3. 어떻게 테스트했는가?

관련되어 있는 시스템이 일정 이상을 넘어가는 순간 모든 시스템을 테스트하는 테스트 코드를 작성할 수는 없다. 따라서, 모든 시스템을 테스트하는 테스트 코드를 작성하는 것이 아닌 `블랙박스`와 `화이트박스` 테스트를 실행한다.

### 블랙박스 테스트

블랙박스 테스트란 모든 모듈 사이의 상관 관계를 테스트하는 것이 아니라 어떤 인풋이 들어가면 어떤 아웃풋이 나온다 정도의 관점으로 테스트를 작성하는 것을 말한다.

예를 들어

- 결제가 잘 동작하는가?
- 버튼은 잘 눌러지는가?
- 에러 핸들링은 잘 되는가?

### 화이트박스 테스트

화이트박스 테스트란 코드 안에 있는 모든 모듈이 제대로 동작하는지, 시나리오를 통과하는지와 같이 상당히 꼼꼼하게 작성하는 테스트이다. 물론 커버리지라는 말이 있듯이 모든 코드를 커버할 수 있는 테스트 코드를 작성하기란 정말 쉽지 않은 이야기겠지만 그 정도로 꼼꼼히 작성하는 테스트라고 생각하고 넘어가자.

## 4. 테스팅 구축하기

### Jest

- React, Vue 등의 프로젝트 테스트
- 비동기/병렬 테스트
- 테스트 커버리지 확인
- API mocking
- ...

ex.

`sample.test.js`

```javascript
const sum = require("./sum");

test("adds 1 + 2 to equal 3", () => {
  expect(sum(1, 2)).toBe(3);
});
```

### Puppeteer

Chromium 기반 자동화 도구, Puppeteer가 필요한 이유는 결국 이 세미나에서 하고자 하는 테스트는 시각적 요소를 테스트이기 때문이다. 시각적 요소를 테스트하기 위해서는 브라우저가 필요하고 특정 요소를 렌더링해야 한다.

브라우저에 대한 API 프로토콜이라고 공식 문서에서는 말하고 있다. 브라우저와 상호 작용할 수 있다는 것이 포인트

- 유저 행동 코드화
- DOM 탐색 및 인터렉션
- 브라우저 다이얼로그(e.g. alert, confirm) 상호작용
- ...

ex.

```javascript
const puppeteer = require('puppeteer'');

(async () => {
	const browser = await puppeteer.launch();
	const page = await browser.newPage();
	await page.goto('https://example.com');

	// Get the "viewport" of the page, as reported by the page.
	const dimensions = await page.evalutate(() => {
		return {
			width: document.documentElement.clientWidth,
			height: document.documentElement.clientHeight,
			deviceScaleFactor: window.devicePixelRatio,
		};
	});

	console.log('Dimensions:', dimensions);

	await browser.close();
})();
```

## 5. 테스트 구성하기

### package.json

- `@types/jest-image-snapshot`
- `cross-env`
- `jest`
- `**jest-image-snapshot**`: jest를 이용해서 브라우저 화면의 스냅샷을 만들고 이를 통해 비교를 할 수 있게 해주는 라이브러리
- `**jest-puppeteer**`: jest와 puppeteer를 연동하기 위한 라이브러리
- `puppeteer`
- `ts-jest`

### Config 작성

`jest.config.js`

```javascript
// ~src/test/e2e/jest.config.js

module.exports = {
  // 디렉토리 설정
  rootDir: "../../",
  roots: ["./test/e2e"],
  // 타임스크립트 컴파일
  transform: { "^.+\\.ts?$": "ts-jest" },
  // 테스트 코드 특정
  testMatch: ["**/?(*.)+(spec|test).ts"],
  // 테스트코드를 찾지 않을 경로
  testPathIgnorePatterns: ["/node_modules/", "dist"],
  // 테스트 타임아웃
  testTimeout: 100000,
  // 개별 테스트 결과 표시
  verbose: true,
  // 프리셋(puppeteer 사용)
  preset: "jest-puppeteer",
  // 테스트 셋업 후 실행할 스크립트
  setupFilesAfterEnv: ["./test/e2e/jest.image.ts"],
};
```

`jest-puppeteer.config.js`

```javascript
// ~src/test/e2e/jest-puppeteer.config.js

module.exports = {
  server: {
    // Jest 실행 시 서버 서빙을 위해 실행할 커맨드
    command: `npm run testdev`,
    // 서버 포트 번호 (package.json의 "testdev" 스크립트에 설정됨.)
    port: 4321,
    // 로컬호스트이므로 https가 아닌 http 사용
    protocol: "http",
    // 서버 실행 타임아웃
    launchTimeout: 120000,
    debug: true,
  },
  launch: {
    // headless 모드
    headless: true,
    // 브라우저 실행 타임아웃
    timeout: 120000,
  },
};
```

`jest.image.ts`

```javascript
// ~src/test/e2e/jest.image.ts

import { toMatchImageSnapshot } from "jest-image-snapshot";

expect.extend({ toMatchImageSnapshot });
```

`jest.image.ts`만 타입스크립트 확장자인 이유는 `config.js` 두 파일은 jest 인스턴스가 실행될 때 뜨는 것 즉, 자바스크립트를 노드 프로세스가 읽어 와서 실행을 하게 된다. 그에 비해 `jest.image.ts` 파일은 노드 프로세스가 뜬 후 해당 파일을 찾아가서 읽는다. 즉, 타입스크립트 인터프리터가 이미 존재하는 상황이라는 것이다.

`scripts`

```json
"testdev": "export PORT=4321 && react-scripts start",
"test": "cross-env JEST_PUPPETEER_CONFIG=./src/test/e2e/jest-puppeteer.config.js jest --config=./src/test/e2e/jest.config.js",
"test-update": "cross-env JEST_PUPPETEER_CONFIG=./src/test/e2e/jest-puppeteer.config.js jest --config=./src/test/e2e/jest.config.js --updateSnapshot",
```
