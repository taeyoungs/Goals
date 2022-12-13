# Trunk-based Development

> 원본 글  
> https://www.atlassian.com/continuous-delivery/continuous-integration/trunk-based-development

**목차**

- [Trunk-based Development](#trunk-based-development)
  - [개요](#개요)
  - [What is trunk-based development?](#what-is-trunk-based-development)
  - [Git flow vs. trunk-based development](#git-flow-vs-trunk-based-development)
  - [Benefits of trunk-based development](#benefits-of-trunk-based-development)
    - [Allow continuous code integration](#allow-continuous-code-integration)
    - [Ensures continuous code review](#ensures-continuous-code-review)
    - [Enables consecutive production code releases](#enables-consecutive-production-code-releases)
    - [Trunk-based development and CI/CD](#trunk-based-development-and-cicd)
  - [Trunk-based development best practices](#trunk-based-development-best-practices)
    - [Develop in small batches](#develop-in-small-batches)
    - [Feature flags](#feature-flags)
    - [Implement comprehensive automated testing](#implement-comprehensive-automated-testing)
    - [Perform asynchronous code reviews](#perform-asynchronous-code-reviews)
    - [Have three or fewer active branches in the application’s code repository](#have-three-or-fewer-active-branches-in-the-applications-code-repository)
    - [Merge branches to the trunk at least once a day](#merge-branches-to-the-trunk-at-least-once-a-day)
    - [Reduced number of code freezes and integration phases](#reduced-number-of-code-freezes-and-integration-phases)
    - [Build fast and execute immediately](#build-fast-and-execute-immediately)
  - [In conclusion](#in-conclusion)

## 개요

**Trunk-based development**(이하 TBD)는 개발자가 핵심 "**truck**" 또는 **Main branch**에 작고 빈번한 업데이트를 머지하는 **version control management** 방법이다. **TBD**는 머지 및 통합 단계를 간소화하기 때문에 **CI**/**CD**를 달성하고 소프트웨어 제공 및 조직 성과를 높이는 데 도움이 된다.

소프트웨어 개발 초기에 프로그래머는 최신 버전 제어 시스템을 사용할 수 없었다. 오히려 그 당시 프로그래머들은 변경 사항을 추적하고 필요한 경우 되돌릴 수 있는 수단으로 두 가지 버전의 소프트웨어를 동시에 개발했었다. 시간이 지남에 따라 이러한 프로세스는 노동 집약적이고 비용은 많이 들며 비효율적인 것으로 판명됐다.

버전 제어 시스템이 발전하면서 다양한 개발 스타일이 등장하여 프로그래머가 버그를 더 쉽게 찾고 동료와 함께 코딩하고, 릴리즈 주기를 가속화할 수 있게 되었다. 오늘날 대부분의 프로그래머는 **Git flow** 및 **TBD**라는 두 가지 개발 모델 중 하나를 활용하여 고품질의 소프트웨어를 제공한다.

가장 먼저 대중화된 **Git flow**는 특정 개인만이 메인 코드에 대한 변경을 승인할 수 있는 더 엄격한 개발 모델이다. 이는 코드 품질을 유지하고 버그의 수를 최소화한다. **TBD**는 모든 개발자가 메인 코드에 접근할 수 있으므로 보다 개방적인 모델이다. 이를 통해 팀은 빠르게 반복하고 **CI**/**CD**를 구현할 수 있다.

## What is trunk-based development?

**TBD**는 개발자가 핵심 "**truck**" 또는 **Main branch**에 작고 빈번한 업데이트를 머지하는 **version control management** 방법이다. TBD는 머지 및 통합 단계를 간소화하기 때문에 **DevOps** 팀과 **DevOps**의 **lifecycle** 사이에서 일반적인 관습이다. 실제로 **TBD**는 **CI**/**CD**의 필수 사례로 개발자들은 수명이 긴 다른 **branch** 전략에 비해 약간의 커밋으로 수명이 짧은 **branch**를 만들 수 있다. 코드베이스 복잡성과 팀의 규모가 커짐에 따라 **TBD**는 프로덕션 릴리즈 주기를 유지하는 데 도움이 된다.

## Git flow vs. trunk-based development

**Git flow**는 수명이 긴 **Feature branch**와 다수의 주 **branch**를 사용하는 대체 **Git branch** 모델이다. **Git flow**는 **TBD**보다 더 오래 지속되는 **branch**와 더 큰 커밋을 가지고 있다. 이 모델에서 개발자는 **Feature branch**를 만들고 해당 기능이 완료될 때까지 **Main Trunk branch**에 머지하는 것을 연기한다. 이러한 수명이 긴 **Feature branch**는 **Trunk branch**에서 벗어나고 충돌하는 업데이트를 만들어낼 위험이 높기 때문에 머지하려면 더 많은 협업이 필요하다.

**Git flow**는 또한 개발, 핫픽스, 기능 그리고 릴리즈를 위한 별도의 주요 **branch**를 가지고 있다. 이러한 **branch**들 간의 커밋을 머지하기 위한 다양한 전력이 존재한다. 효율적으로 조직하고 관리해야할 **branch**가 더 많기 때문에 팀에서 추가적인 계획을 위한 세션과 리뷰가 필요한 더 복잡한 경우들이 많다.

**TBD**는 수정과 릴리즈로 **Main branch**에 초점을 맞추기 때문에 훨씬 더 간단하다. **TBD**에서 **Main branch**는 항상 안정적이고 이슈가 없으며 배포할 준비가 되어 있다고 가정한다.

## Benefits of trunk-based development

**TBD**는 **CI**를 위한 필수 사례다. 빌드와 테스트 프로세스가 자동화 되었어도 개발자가 공유 브랜치에 자주 통합되지 않는 격리되고 수명이 긴 **Feature branch**에서 작업하는 경우 **CI**는 잠재력을 발휘하지 못한다. **TBD**는 코드를 통합하는 데 있어서 발생하는 마찰을 완화한다. 개발자가 새로운 작업을 끝내면, 새로운 코드는 **Main branch**로 머지해야만 한다. 그러나 개발자들은 빌드가 성공적이라는 것을 확인할 때까지 변경 사항을 머지해서는 안된다. 이 단계에서 새로운 작업이 시작된 후에 수정 사항이 생긴 경우 충돌이 발생할 수도 있다. 특히 이러한 충돌은 개발팀이 커지고 코드베이스의 규모가 확장됨에 따라 점점 더 복잡해진다. 이는 개발자가 **Source branch**에서 벗어난 별도의 **branch**를 만들고 다른 개발자가 동시에 겹치는 코드를 머지할 때 발생한다. 다행히도 **TBD** 모델은 이러한 충돌을 줄인다.

### Allow continuous code integration

**TBD** 모델에선, **Main branch**로 꾸준히 유입되는 커밋 스트림이 있는 레포가 있다. 이 커밋 스트림에 대한 자동화된 테스트 세트 및 코드 커버리지에 대한 모니터링을 추가하면 **CI**가 가능하다. 새로운 코드가 **Trunk**에 머지되면 자동화된 통합 및 코드 커버리지 테스트가 수행되어 코드 품질을 검증한다.

### Ensures continuous code review

TBD의 빠르고 작은 커밋들은 코드 리뷰를 보다 효율적인 프로세스로 만든다. 작은 **branch**들을 통해 개발자들은 작은 변경 사항들을 빠르게 확인하고 리뷰할 수 있다. 이는 리뷰어가 코드의 페이지를 읽거나 코드 변경이 일어난 넓은 범위를 일일히 검사해야 하는 수명이 긴 **Feature branch**에 비해 훨씬 쉽다.

### Enables consecutive production code releases

팀은 **Main branch**에 매일 머지를 자주 수행해야 한다. **TBD**는 **Trunk branch**를 "**녹색**"으로 유지하기 위해 노력한다. 즉, 모든 모든 커밋에서 배포할 준비가 되어 있다는 뜻이다. 자동화된 테스트, 코드 커버리지, 그리고 코드 리뷰는 언제든지 프로덕션에 배포할 준비가 된 **TBD** 프로젝트를 제공한다. 이를 통해 팀은 프로덕션에 자주 배포하고 일일 프로덕션 릴리즈의 추가 목표를 설정할 수 있는 능력을 제공한다.

### Trunk-based development and CI/CD

**CI**/**CD**의 인기가 높아짐에 따라 branch 모델이 개선되고 최적화되어 TBD를 진행하는 곳들이 증가했다. 이제 **TBD**는 **CI**의 요구 사항 중 하나다. **CI**를 통해 개발자는 **Trunk**에 대한 커밋 진행 후에 실행되는 자동화된 테스트와 함께 **TBD**를 수행한다. 이렇게 하면 프로젝트가 항상 잘 동작한다.

## Trunk-based development best practices

**TBD**를 통해 팀은 코드를 빠르고 일관되게 릴리즈할 수 있다. 다음은 팀의 주기를 조정하고 최적화된 릴리즈 일정에서 개발하는 데 도움이 되는 예시들이다.

### Develop in small batches

TBD는 빠른 리듬을 따라 프로덕션에 코드를 전달한다. 만약 TBD가 음악과 같다면 빠른 스타카토가 될 것이다. - short, succinct notes in rapid succession, with the repository commits being the notes. 커밋과 **branch**를 작게 유지하면 머지와 배포 속도가 더 빨라진다.

몇 가지 커밋으로 인한 작은 변경 사항들 또는 몇 줄의 코드 수정으로 cognitive overhead가 최소화된다. 이는 팀이 코드의 제한된 영역과 광범위한 변경 사항을 리뷰할 때 의미 있는 대화를 나누고 신속한 결정을 내리기가 훨씬 쉽다.

### Feature flags

**Feature flag**는 개발자가 비활성화 코드의 경로에 새로운 변경 사항을 래핑하고 나중에 이를 활성화할 수 있도록 하여 **TBD** 개발을 훌륭하게 보완한다. 이를 통해 개발자는 별도의 레포에 **Feature branch**를 생성하는 대신에 새로운 **Feature flag** 경로 내 **Main branch**에 직접 새로운 기능 코드를 커밋할 수 있다.

**Feature flag**는 소규모 배치 업데이트를 권장한다. **Feature branch**를 생성하고 완전한 스펙이 빌드되는 것을 기다리는 대신 개발자는 **Feature flag**를 도입하고 **flag** 내에서 기능을 구축하는 새로운 **Trunk** 커밋을 `push` 하는 **Trunk** 커밋을 만들 수 있다.

### Implement comprehensive automated testing

**CI**/**CD**를 달성하려는 최신 소프트웨어 프로젝트에는 자동화된 테스트가 필요하다. 릴리즈 파이프라인의 여러 단계에서 실행되는 여러 유형의 자동화된 테스트가 있다. 짧게 실행되는 유닛 테스트와 통합 테스트는 개발 중에 그리고 코드가 머지될 때 수행된다. 길게 실행되는 풀스택, E2E 테스트는 나중에 Full Staging 또는 프로덕션 환경에 대한 파이프라인 단계에서 수행된다.

자동화된 테스트는 개발자가 새로운 커밋을 머지할 때 small batch rhythm을 유지함으로써 **TBD**에 도움이 된다. 자동화된 테스트 세트는 모든 이슈에 대한 코드를 리뷰하고 자동으로 해당 코드를 승인하거나 거부한다. 이를 통해 개발자는 커밋을 빠르게 생성하고 자동화된 테스트틀 통해 해당 커밋에 새로운 이슈가 있는지 확인할 수 있다.

### Perform asynchronous code reviews

**TBD**에서는 코드 리뷰를 즉시 수행해야만 하며 나중에 리뷰하기 위한 비동기식 시스템을 도입하지 말아야 한다. 자동화된 테스트는 사전에 코드를 리뷰하는 계층을 제공한다. 개발자가 팀원의 **PR**을 리뷰할 준비가 되면 먼저 자동화된 테스트를 통과하는지 그리고 코드 커버리지는 증가했는지 확인할 수 있다. 이를 통해 리뷰어는 새로운 코드가 특정 스펙을 만족한다는 즉각적인 확신을 얻을 수 있게 되기에 최적화에 집중할 수 있다.

### Have three or fewer active branches in the application’s code repository

**Branch**가 머지되면 삭제하는 것이 가장 좋다. 활성화된 **branch**가 많이 있는 레포의 경우 **side effect**가 존재한다. 팀에서 활성화된 **branch**를 검사하여 진행 중인 작업을 확인하는 것이 도움이 될 순 있지만 여전히 **stale**하고 비활성화된 **brnach**가 있는 경우 이러한 도움도 얻지 못할 수 있다. Some developers use Git user interfaces that may become unwieldy to work with when loading a large number of remote branches.

### Merge branches to the trunk at least once a day

높은 퍼포먼스를 발휘하는 **TBD** 팀은 적어도 매일 열려 있고 머지할 준비가 된 **branch**를 마무리하고 머지하해야 한다. 이러한 작업은 릴리즈 추적을 위한 리듬을 유지하고 주기를 설정하는 데 도움을 준다. The team can then tag the main trunk at the end of day as a release commit, which has the helpful side effect of generating a daily agile release increment.

### Reduced number of code freezes and integration phases

애자일 **CI**/**CD** 팀은 통합 단계를 위해 계획된 코드에 대한 동결 또는 일시 중지가 필요하지 않아야 한다. 단, 조직에서 다른 이유로 필요로 할 수는 있다. **CI**/**CD**의 "**continuous**"는 업데이트를 지속적으로 진행하고 있음을 의미한다. **TBD** 팀은 코드의 동결로 인한 업데이트 차단을 피하고 릴리즈 파이프라인이 중단되지 않도록 적절하게 계획해야 한다.

### Build fast and execute immediately

빠른 릴리즈 주기를 유지하려면 빌드 및 테스트 수행 시간을 최적화해야 한다. **CI**/**CD** 빌드 도구는 정적 리소스에 대해 비용이 많이 드는 계산을 피하기 위해 적절한 캐싱 계층을 사용해야 한다. 서드 파티 서비스에 적절한 스텁을 사용하도록 테스트를 최적화 해야 한다.

## In conclusion

**TBD**는 단순화된 **Git branch** 전략을 사용하여 소프트웨어 릴리즈 주기를 설정하기 유지하기 때문에 높은 퍼모먼스를 위한 엔지니어링 팀에 대한 표준이다. 또한 **TBD**를 통해 엔지니이렁 팀은 최종 사용자에게 소프트웨어를 제공하는 방법에 대한 더 많은 유연성과 제어 기능을 제공한다.
