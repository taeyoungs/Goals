# BEM

> 원본 글  
> https://sparkbox.com/foundry/bem_by_example

**목차**

- [BEM](#bem)
  - [예제로 이해하는 BEM](#예제로-이해하는-bem)
    - [왜 BEM인가?](#왜-bem인가)
    - [BEM은 어떻게 동작하는가?](#bem은-어떻게-동작하는가)
  - [예제](#예제)
    - [요소 또는 변경자가 없는 단순한 구성 요소](#요소-또는-변경자가-없는-단순한-구성-요소)
    - [변경자가 있는 구성 요소](#변경자가-있는-구성-요소)
    - [하위 요소가 있는 구성 요소](#하위-요소가-있는-구성-요소)
    - [변경자가 있는 요소](#변경자가-있는-요소)
    - [구성 요소 변경자 기반으로 하위 요소 스타일 처리하기](#구성-요소-변경자-기반으로-하위-요소-스타일-처리하기)
    - [여러 단어 명명법(케밥 케이스, 카멜 케이스 접목)](#여러-단어-명명법케밥-케이스-카멜-케이스-접목)

## 예제로 이해하는 BEM

**BEM**은 **CSS** 클래스 명명 규칙입니다. **BEM**의 기본 개념은 단순하고 직관적이지만 BEM을 처음 접할 때 저지르기 쉬운 오류들을 예제를 통해 설명합니다.

**BEM**(**Block-Element-Modifier**)은 **CSS** 클래스 이름을 위한 명명 규칙 표준입니다. **BEM**은 여러 웹 사이트에서 꽤나 많이 채택되었으며 **CSS**를 쉽게 읽고, 쉽게 이해하고, 확장하기에 용이합니다.

### 왜 BEM인가?

**BEM** 명명 규칙은 세 가지 뚜렷한 이점을 제공합니다.

1. BEM은 목적 또는 기능을 전달한다.
2. BEM은 구성 요소의 구조를 전달한다.
3. BEM은 선택자 특이성을 항상 낮은 수준으로 유지한다.

### BEM은 어떻게 동작하는가?

**BEM** 클래스 이름은 최대 세 가지로 구성됩니다.

1. 블록(Block): 구성 요소의 가장 바깥쪽 상위 요소를 블록으로 정의한다.
2. 요소(Element): 구성 요소 안쪽에는 하나 또는 그 이상의 요소가 있을 수 있다.
3. 변경자(Modifier): 블록 또는 요소는 변경자를 이용하여 변형을 표시할 수 있다.

세 가지 형식을 모두 클래스 이름에 사용하면 이렇게 보일 겁니다.

```
.block__element--modifier
```

소개는 이정도로 하고 구체적인 예제를 살펴보겠습니다.

## 예제

### 요소 또는 변경자가 없는 단순한 구성 요소

간단한 구성 요소는 단일 요소와 단일 클래스를 사용하는 것만으로도 블록이 될 수 있습니다.

```html
*<!-- 역자 주 / btn은 하나의 구성 요소다 -->*
<button class="”btn”"></button>

<style>
  **.btn** {
  }
</style>
```

### 변경자가 있는 구성 요소

구성 요소에 변형이 있을 수 있다. 변형은 변경자 클래스로 구현합니다.

```html
<!-- DO THIS / 역자 주 / btn 구성 요소에 --submit 변경자를 추가해서 확장했다 -->
<button class="btn btn--submit"></button>

<style>
  .btn {
    display: inline-block;
    color: blue;
  }
  .btn--submit {
    color: green;
  }
</style>
```

변경자 클래스만 사용하면 안 됩니다. 변경자 클래스는 기본 클래스를 대체하기 위한 것이 아니라 확장하기 위한 것이기 때문입니다.

```html
<!-- DON'T DO THIS / 이렇게 하지 마세요 -->
<button class="btn--secondary"></button>

<style>
  .btn--secondary {
    display: inline-block;
    color: green;
  }
</style>
```

### 하위 요소가 있는 구성 요소

일반적으로 복잡한 구성 요소에는 하위 요소가 있습니다. 따라서, 스타일이 필요한 각 하위 요소에는 명명된 클래스가 있어야 합니다.

BEM의 목적 중 하나는 **특이성을 낮추고 일관성 있게 유지**하는 것입니다.

- **HTML**의 하위 요소에서 클래스 이름을 생략하지 않아야 한다.
- 클래스 이름을 생략하면 구성 요소 내부에 있는 이름 없는 요소의 스타일을 처리하기 위해 특이성이 더 높은 선택자를 사용해야 한다. (아래 `img` 및 `figcaption` 요소 참고).

클래스 이름을 생략하면 당장은 장황한 클래스 이름을 사용하지 않아서 더 간결해 보일 수 있지만 결국 미래엔 특이성 증가 때문에 위기를 맞게 됩니다. **BEM**의 목표 중 하나는 대부분의 선택자가 단일 클래스 이름만 사용하는 것입니다.

> **역자 주**: **BEM**이 선택자 중첩을 금지한다는 의미는 아니다. 하위 요소에 클래스 선택자를 사용하면 `!important` 같은 끔찍한 규칙을 덜 사용하도록 돕는다.

```html
<!-- DO THIS -->
<figure class="photo">
  <img class="photo__img" src="me.jpg" />
  <figcaption class="photo__caption">Look at me!</figcaption>
</figure>

<style>
  .photo {
  } /* 특이성 10 */
  .photo__img {
  } /* 특이성 10 */
  .photo__caption {
  } /* 특이성 10 */
</style>

<!-- DON'T DO THIS / 이렇게 하지 마세요 -->
<figure class="photo">
  <img src="me.jpg" />
  <figcaption>Look at me!</figcaption>
</figure>

<style>
  .photo {
  } /* 특이성 10 */
  .photo img {
  } /* 특이성 11 */
  .photo figcaption {
  } /* 특이성 11 */
</style>
```

구성 요소에 여러 수준의 하위 요소가 있는 경우 클래스 이름에서 각 수준을 나타내려고 하면 안 됩니다.

**BEM**은 구조의 깊이를 전달하지 않습니다.

**구성 요소의 하위 요소를 나타내는 BEM 클래스 이름은 오직 기본 블록 이름과 하나의 요소 이름만 허용합니다.** 따라서, 아래 예시에서 `photo__caption__quote`는 BEM을 잘못 사용한 것으로 `photo__quote`가 더 적합합니다.

```html
<!-- DO THIS -->
<figure class="photo">
  <img class="photo__img" src="me.jpg" />
  <figcaption class="photo__caption">
    <blockquote class="photo__quote">Look at me!</blockquote>
  </figcaption>
</figure>

<style>
  .photo {
  }
  .photo__img {
  }
  .photo__caption {
  }
  .photo__quote {
  }
</style>

<!-- DON'T DO THIS / 이렇게 하지 마세요 -->
<figure class="photo">
  <img class="photo__img" src="me.jpg" />
  <figcaption class="photo__caption">
    <blockquote class="photo__caption__quote">
      <!-- 클래스 이름에 여러 하위 요소(또는 구조)를 명명하지 말 것 -->
      Look at me!
    </blockquote>
  </figcaption>
</figure>

<style>
  .photo {
  }
  .photo__img {
  }
  .photo__caption {
  }
  .photo__caption__quote {
  } /* 역자 주: 초심자의 가장 흔한 실수다 */
</style>
```

### 변경자가 있는 요소

경우에 따라 구성 요소의 하위 단일 요소에 변형을 주려고 할 수 있는데 이런 경우 구성 요소를 따로 추가하는 대신 하위 요소에 변경자를 추가하면 됩니다.

```html
<figure class="photo">
  <img class="photo__img photo__img--framed" src="me.jpg" />
  <figcaption class="photo__caption photo__caption--large">Look at me!</figcaption>
</figure>

<style>
  .photo__img--framed {
    /* incremental style changes */
  }
  .photo__caption--large {
    /* incremental style changes */
  }
</style>
```

### 구성 요소 변경자 기반으로 하위 요소 스타일 처리하기

동일한 구성 요소의 하위 요소를 동일한 방식으로 일관성 있게 수정하려면 기본 구성 요소에 변경자를 추가하고 하나의 변경자를 기반으로 각 하위 요소 스타일을 처리합니다. 이렇게 하면 특이성이 높아지지만 구성 요소 수정이 훨씬 수월해집니다.

```html
<!-- DO THIS / 역자 주 / BEM이 선택자 중첩을 금지하는 건 아니다. 다만 지금보다 중첩이 더 깊어지면 곤란하다. -->
<figure class="photo photo--highlighted">
  <img class="photo__img" src="me.jpg" />
  <figcaption class="photo__caption">Look at me!</figcaption>
</figure>

<style>
  .photo--highlighted .photo__img {
  }
  .photo--highlighted .photo__caption {
  }
</style>

<!-- DON'T DO THIS / 이렇게 하지 마세요 -->
<figure class="photo">
  <img class="photo__img photo__img--highlighted" src="me.jpg" />
  <figcaption class="photo__caption photo__caption--highlighted">Look at me!</figcaption>
</figure>

<style>
  .photo__img--highlighted {
  }
  .photo__caption--highlighted {
  }
</style>
```

### 여러 단어 명명법(케밥 케이스, 카멜 케이스 접목)

BEM 이름은 의도적으로 `블록__요소--변경자`를 분리하기 위해 단일 밑줄 대신 이중 밑줄과 이중 하이픈을 사용합니다. 단일 하이픈을 단어 구분 기호로 사용할 수 있기 때문입니다. 클래스 이름은 읽기 쉬워야 하기 때문에 보편적으로 인식할 수 없는 약어는 바람직하지 않습니다.

```html
<!-- DO THIS / 역자 주 / 케밥 케이스를 접목한 명명법 -->
<div class="some-thesis some-thesis--fast-read">
  <div class="some-thesis__some-element"></div>
</div>

<style>
  .some-thesis {
  }
  .some-thesis--fast-read {
  }
  .some-thesis__some-element {
  }
</style>

<!-- DO THIS / 역자 주 / 이렇게 카멜 케이스를 접목해도 된다 -->
<div class="someThesis someThesis--fastRead">
  <div class="someThesis__someElement"></div>
</div>

<style>
  .someThesis {
  }
  .someThesis--fastRead {
  }
  .someThesis__someElement {
  }
</style>

<!-- DON'T DO THIS / 이렇게 하지 마세요 -->
// These class names are harder to read
<div class="somethesis somethesis--fastread">
  <div class="somethesis__someelement"></div>
</div>

<style>
  .somethesis {
  }
  .somethesis--fastread {
  }
  .somethesis__someelement {
  }
</style>
```
