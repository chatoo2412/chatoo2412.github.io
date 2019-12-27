---
categories:
  - JavaScript
  - React
tags:
  - JavaScript
  - Library
  - React
date: 2019-11-12T13:49:00+09:00
last_modified_at: 2019-11-12T16:56:00+09:00

title: 상태 관리 관점에서 바라본 React Context
description: React Context의 간략한 개념과 용도를 다른 상태 관리 라이브러리와 비교해서 설명합니다.
excerpt: React Context의 간략한 개념과 용도를 다른 상태 관리 라이브러리와 비교해서 설명합니다.
header:
  # image: /assets/posts/logo-react.png
  # image_description:
  teaser: /assets/posts/logo-react.png
  og_image: /assets/posts/logo-react.png
---

{% raw %}

## Prop Drilling과 React Context

React context는 엄밀히 말해서 상태 관리와는 관련이 없습니다. 실제로 [공식 문서](https://reactjs.org/docs/context.html)에도 `state management`라는 용어는 등장하지도 않습니다. React Context는 근본적으로 [prop drilling](https://kentcdodds.com/blog/prop-drilling/) 문제점을 해결하기 위한 솔루션입니다.

> Context provides a way to pass data through the component tree without having to pass props down manually at every level. - [React](https://reactjs.org/docs/context.html)

React는 `props`를 이용해 데이터를 top-down으로 전달하는 심플한 구조로 설계된 UI 라이브러리입니다. 하지만 부모-자식 관계가 아닌 여러 컴포넌트에 걸쳐서 공유되는 데이터를 다루는 경우에는 굉장히 성가시게 됩니다.

예를 들어 다음과 같은 theme 예제를 생각해봅시다.

```jsx
import React from 'react';

const App = () => <Toolbar theme="dark" />;

const Toolbar = (props) => (
  <div>
    <ThemedButton theme={props.theme} />
  </div>
);

const ThemedButton = (props) => <Button theme={props.theme} />;
```

이 예제는 잘 동작하는 코드이지만 `Toolbar` 컴포넌트는 단지 자식 컴포넌트에게 전달하기 위해서 불필요한 `theme` props를 중계하고 있습니다. 이때 Context를 도입하면 구조를 단순화할 수 있습니다.

```jsx
import React, { createContext } from 'react';

const defaultValue = 'light';
const ThemeContext = React.createContext(defaultValue);

const App = () => (
  <ThemeContext.Provider value="dark">
    <Toolbar />
  </ThemeContext.Provider>
);

const Toolbar = () => (
  <div>
    <ThemedButton />
  </div>
);

const ThemedButton = () => <ThemeContext.Consumer>{(theme) => <Button theme={theme} />}</ThemeContext.Consumer>;
```

이렇게 Context를 사용해서 리팩토링한 코드를 보면 `Toolbar` 컴포넌트는 본인이 관심도 없는 `theme`를 중계해줄 필요가 없고, `ThemedButton` 가 `context`에서 직접 꺼내다 쓰면 됩니다. 관심사가 잘 분리되었고 훨씬 구조적인 모습입니다.

너무 단순한 예제이기 때문에 Context를 사용한 코드가 더 길고 난해해 보이긴 합니다. 하지만 실제 앱은 훨씬 복잡하며 그때 prop drilling 문제가 개발자를 지독하게 괴롭힌다는 사실을 명심하셔야 합니다.

### 다른 패턴들: Composition, HoC, Render Props

그러나 단지 prop drilling을 피하기 위해서라면 다른 방법이 얼마든지 존재합니다. 실제로 공식 문서에서도 Context를 도입하기 전에 [component composition을 고려해보라고 제안](https://reactjs.org/docs/context.html#before-you-use-context)하고 있습니다.

당연한 말이지만 이 패턴들은 각각 장단점이 있고, 용도도 조금씩 다릅니다. 예를 들어, 관심사가 UI에 가깝다면 render props 패턴이 낫고, 데이터와 비즈니스 로직에 집중하는 경우에는 Context가 더 적합합니다. 심지어는 서로 혼용해서 쓸 수도 있고, [그렇게 했을 때 훨씬 강력해집니다](https://velopert.com/3606).

또한, 비슷한 이유로 전역 상태 - 테마, locale, 로그인한 유저 정보 등 - 를 다루는 경우에는 Context가 확실히 우위에 있습니다.

## 상태 관리 도구로서의 React Context

그러면 상태 관리라는 목적에서 보면 경쟁자가 없느냐, 또 그것도 아닙니다. 프론트엔드 앱은 거의 항상 stateful하기 때문에 그 전역 상태를 어떻게 다루어야 할지 늘 고민해왔고, 그에 대한 솔루션도 발전해왔습니다. 현 시점에서 널리 알려진 상태 관리 도구로는 Redux와 MobX, Apollo Client 등이 있습니다.

그럼 이들과 비교해서 React Context는 어떤 공통점과 차이점이 있는지, 그리고 React Context가 이들을 대체할 수 있는지, 이런 의문이 자연스레 생길 겁니다. 그래서 그중 대표적인 Redux와 비교해서 React Context만의 특징을 알아보겠습니다.

### Built-In이다

[Flux(Redux)가 등장하게 된 배경](https://code-cartoons.com/a-cartoon-guide-to-flux-6157355ab207)을 한번 생각해봅시다. Facebook 앱을 전통적인 방식으로 구현하다보니, 데이터는 양방향으로 흐르는데, 그 흐름을 추적하기가 점점 어려워졌고, 복잡도는 기하급수적으로 증가해, 결국 디버깅조차 불가능해졌다, 그래서 이 문제를 해결하기 위해 만든 것이 바로 unidirectional data flow의 Flux입니다.

하지만 앞서 말했다시피 React는 태생부터 top-down의 unidirectional data flow 구조입니다. Flux가 해결하고자 한 문제점을 React는 근본적으로 가지고 있지 않습니다. 다만 컴포넌트 간에 데이터를 공유할 방법이 미비했을 뿐이었습니다. 그래서 등장한 것이 바로 Context입니다. React에 Context가 존재하지 않던 시절이라면 몰라도 지금 시대에 굳이 서드파티를 고집할 이유는 없습니다.

### 러닝 커브가 낮다

위의 theme 예제에서 알 수 있듯이 Context를 사용하기 위해서는 딱 3가지만 알면 됩니다. Context를 만들고(`createContext`), 거기에 데이터를 집어넣고(`Provider`), 필요한 곳에서 꺼내 쓰는(`Consumer`) 게 전부입니다. 다른 복잡한 룰은 없습니다.

Redux를 사용하려면 패턴 그 자체를 공부해야 하고, 사용 규칙을 숙지해야 하고, 제대로 쓰기 위해서는 여러가지 middleware들을 비교하고 도입해야 하는 것에 비하면 진입 장벽이 굉장히 낮습니다.

### 전역적이지 않은 상태를 다루기 용이하다

다른 상태 관리 라이브러리가 그렇듯이 Redux도 전역 상태에 포커스를 두고 있습니다. 반대로 말하면 전역적이지 않은 상태를 다루는 데에는 취약합니다.

#### Redux Single State Tree[^ssot]의 한계

> Single source of truth: The state of your whole application is stored in an object tree within a single store. - [Redux](https://redux.js.org/introduction/three-principles#single-source-of-truth)

[^ssot]: 개인적인 견해로는 single source of truth가 single state tree를 의미하지는 않으며, Redux는 SSoT라는 용어를 오용하고 있다고 생각합니다. 따라서 이 글에서는 single state tree라는 용어로 대체합니다.

[Redux의 제 1 원칙](https://redux.js.org/introduction/three-principles#single-source-of-truth)에 따르면 모든 상태는 하나의 store에 저장되어야 합니다. 하지만 과연 그게 최선일까요?

![Redux Single State Tree](/assets/posts/2019-11-12-react-context-from-a-state-management-perspective/single-state-tree.png)

예를 들어 React 컴포넌트 트리 중 특정 subtree(위 그림의 파란 사각형)에서만 공유되는 데이터가 존재한다고 가정해봅시다. Redux는 이러한 케이스를 커버하지 못하며 무조건 global state에 저장하는 수밖에 없습니다(왼쪽 그림). Redux 측에서는 얘기하기를 [reducer로 접근을 제어하기 때문에 문제 없다](https://redux.js.org/faq/store-setup#can-or-should-i-create-multiple-stores-can-i-import-my-store-directly-and-use-it-in-components-myself)고 합니다.

하지만 아무리 생각해도 single state tree는 장점보다는 단점이 훨씬 커 보입니다. 왜 지역적인 관심사를 저 멀리 global에 저장해야 하죠? 왜 서로 다른 도메인 데이터가 한 곳에 위치해야 하죠? 왜 굳이 자료를 [normalize](https://redux.js.org/recipes/structuring-reducers/normalizing-state-shape) - 라고 쓰고 flatten이라 읽는다 - 해줘야 하죠? 심지어는 [ORM을 쓰라고요?](https://redux.js.org/recipes/structuring-reducers/updating-normalized-data#redux-orm)

> Redux의 철학인 Single Source of Truth는 여러 State들을 한 곳에 몰아박는 결과를 만들어냈고, 여러가지 상태 중 컴포넌트에 필요한 상태만을 얻기 위해 복잡하게 구성되어 있는 상태를 다시 컴포넌트 내에서 풀어내야 하고, React Component에서 Redux를 사용하기 위해 명시해줘야 하는 API들은 코드의 양을 늘리는 데에만 기여했습니다. - [MobX 내부 살펴보기](http://www.secmem.org/blog/2019/03/09/mobx-internals/)

Redux도 이 문제점을 인식했는지 [만약 정말로 필요하다면 여러 개의 store를 만들어도 되긴 된다](https://redux.js.org/faq/store-setup#can-or-should-i-create-multiple-stores-can-i-import-my-store-directly-and-use-it-in-components-myself), 아니면 [sub-apps](https://redux.js.org/recipes/isolating-redux-sub-apps)라는 개념을 고려해보라고 가이드로 제공하긴 합니다. 하지만, 애초에 도메인별로 각각 store를 만들게 유도했다면 더 좋지 않았을까요?

반면 Context는 오른쪽 그림처럼 필요한 경우에 그때그때 context를 만들면 됩니다.

오리지널 Flux 패턴이나 MobX도 마찬가지로 multiple stores를 권장합니다.

#### Lifecycle 관리 문제

위 그림에서 이어서 생각해봅시다. 페이지가 전환되어서 subtree가 unmount된다면 데이터는 어떻게 될까요?

Redux store는 React와 별개로 존재하기 때문에 그 데이터의 lifecycle을 개발자가 직접 관리해줘야 합니다. (혹은 관리하지 않고 방치하면 됩니다.)

Context는 React 컴포넌트 트리 상에 존재하기 때문에 provider 컴포넌트가 unmount되면 함께 사라집니다.

#### Not `state` But `context`

반복해서 말하지만 Context는 state가 아니라 문자 그대로 context입니다. 따라서 이런 것도 가능합니다.

```jsx
import React, { createContext } from 'react';

const defaultValue = { name: 'unknown' };
const SectionContext = createContext(defaultValue);
const SectionProvider = SectionContext.Provider;
const SectionConsumer = SectionContext.Consumer;

const App = () => (
  <div>
    <Header />
    <Content />
  </div>
);

const Header = () => (
  <SectionProvider value={{ name: 'header' }}>
    <header>
      <Link />

      <SectionProvider value={{ name: 'floating bar' }}>
        <aside>
          <Link />
        </aside>
      </SectionProvider>
    </header>
  </SectionProvider>
);

const Content = () => (
  <SectionProvider value={{ name: 'content' }}>
    <article />

    <div>
      <Link />
    </div>
  </SectionProvider>
);

const Link = () => {
  const sendAnalyticsEvent = (sectionName) => {
    /*
     * Link 컴포넌트가 어느 provider 안에 위치하는지에 따라
     * 'unknown' | 'header' | 'floating bar' | 'content' 가 출력됨
     */
    console.log(`Link in ${sectionName} has clicked.`);
  };

  return <SectionConsumer>{({ name }) => <a onClick={() => sendAnalyticsEvent(name)}>click me</a>}</SectionConsumer>;
};
```

Context는 consume할 때 가장 가까운 조상 provider에서 값을 꺼내옵니다. (만약 조상 provider가 없다면 default로 fall back됩니다.) 그래서 context provider를 nesting해서 이렇게 활용할 수도 있습니다.

### React Context의 단점

물론 React Context가 항상 우월한 것은 아닙니다.

#### Wrapper Hell

![Wrapper Hell](/assets/posts/2019-11-12-react-context-from-a-state-management-perspective/wrapper-hell.png)

Context가 제공하는 provider와 consumer는 React 컴포넌트이기 때문에 wrapper hell은 여전히 발생합니다.

```jsx
const App = () => (
  <LocaleProvider>
    <UserProvider>
      <FoobarProvider>
        <Content />
      </FoobarProvider>
    </UserProvider>
  </LocaleProvider>
);
```

`useContext` Hook으로 consumer 갯수는 줄일 수 있지만 provider 중첩은 어쩔 수 없습니다. Redux처럼 single context로 만들면 되겠지만 그럴 바에는 차라리 wrapper hell이 나아 보입니다.

```jsx
const App = () => (
  <LocaleAndUserAndFoobarProvider>
    <Content />
  </LocaleAndUserAndFoobarProvider>
);
```

#### Convention의 부재

라이브러리나 프레임워크의 장점 중 하나는 best practice나 convention, 가이드를 제공한다는 점입니다.

Redux를 예로 들자면 action, reducer, store라는 명확한 개념과 가이드 문서가 수십 페이지나 존재합니다.

하지만 React Context에는 그런 게 없습니다. Context란 아무 거나 담을 수 있는 뻥 뚫린 공간이고, 거기에 무엇을 넣을지는 개발자 마음입니다. 그리고 사용방법에 있어서도 action이나 reducer를 좋아하면 써도 되고 아니면 직접 접근해도 무방합니다.

자유도가 높아서 좋아하는 분도 있겠지만, 저는 이게 협업을 방해하는 요소라고 생각합니다. 동료들 간에 취향이 서로 다르기 때문에 코딩 스타일을 맞추기 위해서 소모적인 논쟁을 하거나, 코드 리뷰가 오래 걸리는 등 결국 생산성을 떨어뜨리게 되기 때문입니다. 앞에서는 Context가 진입 장벽이 낮고 boilerplate가 간결해서 좋다고 얘기했지만 여기서는 오히려 양날의 검으로 작용하는 상황입니다.

> But what IS Context actually?
> In the end, it’s just some data - e.g. a JavaScript object (could also be just a number or string etc) which is shared across component boundaries. You can store any data you want in Context. - [Redux vs React’s Context API](https://academind.com/learn/react/redux-vs-context-api/#the-context-object)

#### 편의 도구의 부재

비슷한 이유로 Redux의 강력한 middleware나 time travel debugging 같은 도구를 사용할 수도 없습니다. 하지만 Redux의 middleware를 도입하는 주된 목적이 async 처리라는 점을 생각해보면 Redux middleware 대신 React의 side effect를 그냥 쓰면 되긴 합니다.

#### React와의 강한 결합도

장점일 수도 있고 단점일 수도 있습니다.

## 결론

React Context는 컴포넌트 간에 데이터를 공유할 수 있게 해주는 React 공식 API입니다. 기존의 HoC나 render props pattern과 배타적이지 않고, 함께 사용했을 때 훨씬 강력해지고 실제로도 그렇게 사용합니다.

React Context는 상태 관리에도 사용할 수 있으며, 그런 측면에서는 Redux나 MobX와 비교될 수는 있습니다. 하지만 관심사를 일부 공유할 뿐, 포지션이 엄연히 다릅니다.

프론트엔드 앱을 만들 때에 상태 관리 도구는 필수적인데, 이때 Redux, MobX, Apollo Client와 함께 React Context도 하나의 옵션이 될 수 있습니다. 앱이 얼마나 복잡한지, 얼마나 규모가 큰지, 동료들의 지식 수준이 어느 정도인지에 따라 선택해서 사용하시면 됩니다. 닭 잡는 데 소 잡는 칼을 쓸 수 없는 것처럼, 상황에 맞는 적절한 도구를 선택하시기를 바랍니다.

{% endraw %}
