# useEffect fire twice in React 18 is not bug

> 관련 링크  
> https://github.com/facebook/react/issues/24502#

## Updates to Strict Mode

**React** 팀은 이전 상태를 보존하면서 UI의 섹션을 추가하거나 지울 수 있는 기능을 추가하고자 한다. 예를 들어, 사용자가 스크린에서 벗어났다가 다시 돌아왔을 때(뒤로 가기를 말하는 것 같기도 ?) **React**는 그 즉시 이전 스크린을 보여줘야만 한다. 이 작업을 진행하기 위해서, **React**는 이전과 동일한 컴포넌트 상태를 이용하여 **DOM** 트리를 `ummount` 그리고 `remount` 해야한다.

이 기능은 기본적으로 **React**가 더 나은 퍼포먼스를 낼 수 있게 한다. 다만, 이러한 기능을 위해 컴포넌트는 여러 번 `mounted` 되고 `destoryed` 되는데 조금 더 유연해져야 한다. (아마 위 기능이 원할히 동작하기 위해서는 컴포넌트가 여러 번 마운트 혹은 언마운트 되어야 하는데 이러한 과정 속에서 발생하는 여러 사이드 이펙트를 조금 더 탄력적이고 유연하게 대응될 필요가 있다는 뜻 같다) 대부분의 `effect`들은 아무런 변화를 주지 않겠지만 특정 `effect`들은 한번만 `mounted` 되거나 `destoryed` 될 수도 있다.

이러한 표면적인 문제들을 해결하기 위해, **React 18**은 개발 모드 전용 **Strict Mode** 체크 기능을 제공한다. 이 새로운 체크 기능은 모든 컴포넌트가 `unmount` 되거나 `remount` 되는 것을 자동으로 체크하며 컴포넌트가 처음 `mount`될 때 상태를 기반으로 두 번째 `mount` 때 이전 상태를 복원한다.

변경 전에는, React는 컴포넌트를 `mount`한 후 `effect`를 만들었다.

```
* React mounts are components.
  * Layout effects are created.
  * Effect effects are created.
```

**React 18**의 **Strict Mode**에선, 개발 모드 내라면 **React**가 컴포넌트가 `unmount`하고 `remount`하는 것을 시뮬레이션하고 있을 것이다.

```
* React mounts the component.
  * Layout effects are created.
  * Effect effects are created.
* React simulates unmounting the component
  * Layout effects are destoryed.
  * Effects are destroyed.
* React simulates mounting the component with the previous state.
  * Layout effect setup code runs
  * Effect setup code runs
```
