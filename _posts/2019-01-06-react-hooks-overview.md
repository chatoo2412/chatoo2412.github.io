---
categories:
  - JavaScript
  - React
tags:
  - JavaScript
  - Library
  - React
  - Translation
date: 2019-01-06T21:00:00+09:00
last_modified_at: 2019-01-09T23:44:00+09:00

title: React 공식 문서 번역 - Hooks 개요
description: React Hooks이 무엇인지, 어떻게 사용하는지 간략하게 소개합니다.
excerpt: React Hooks이 무엇인지, 어떻게 사용하는지 간략하게 소개합니다.
---

{% raw %}

<p class="notice--info" markdown="1">
이 글은 React 공식 문서 [Hooks at a Glance](https://reactjs.org/docs/hooks-overview.html)를 번역한 글입니다.
</p>

<p class="notice--success">
*Hooks*는 클래스를 작성하지 않고도 state와 다른 React의 기능들을 사용할 수 있게 해주는 새로운 기능입니다. React v16.7.0-alpha부터 사용할 수 있습니다.
</p>

Hooks는 [하위 호환성](https://reactjs.org/docs/hooks-intro.html#no-breaking-changes)을 가지고 있습니다. 이 문서는 React에 경험이 있는 사용자를 대상으로 Hooks에 대해 간략히 소개합니다.

이 문서는 빠르게 진행됩니다. 혼란스러운 경우에는 다음과 같은 박스를 참고하세요.

<section class="notice--warning">
  <h4>자세한 설명</h4>
  <p markdown="1">React에 Hooks를 도입하는 이유를 알고 싶다면 [Motivation](https://reactjs.org/docs/hooks-intro.html#motivation) 파트를 읽어보세요.</p>
</section>

**↑↑↑ 각 섹션 마지막에는 이런 박스가 있습니다.** 자세한 설명을 보시려면 링크를 따라가시면 됩니다.

## 📌 State Hook

여기 버튼을 클릭하면 값이 증가하는 간단한 카운터 예제가 있습니다.

```jsx
import { useState } from 'react';

function Example() {
  // Declare a new state variable, which we'll call "count"
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>Click me</button>
    </div>
  );
}
```

여기서 `useState`가 바로 _Hook_ 입니다(이게 무슨 의미인지는 앞으로 알아보겠습니다). Hook을 호출해 함수 컴포넌트(function component) 안에 state를 추가했습니다. 이 state는 컴포넌트가 재렌더링 되어도 그대로 유지될 것입니다. `useState`는 _현재의_ state 값과 이 값을 업데이트하는 함수를 쌍으로 제공합니다. 우리는 이 함수를 이벤트 핸들러나 다른 곳에서 호출할 수 있습니다. 이것은 클래스의 `this.setState`와 거의 유사하지만, 이전 state에 새 state를 merge하지 않는다는 차이점이 있습니다. (`useState`와 `this.state`를 비교하는 예시가 [Using the State Hook](https://reactjs.org/docs/hooks-state.html) 문서에 있으니 한번 보세요.)

`useState`는 인자로 초기값을 받습니다. 카운터는 0부터 시작하기 때문에 위 예제에서는 초기값으로 `0`을 넣어준 것입니다. `this.state`와는 달리 `setState` Hook의 state는 객체일 필요가 없습니다 -- 물론 원한다면 그렇게도 가능하지만요. 이 초기값은 첫 번째 렌더링에만 딱 한번 사용됩니다.

#### 복수의 state 변수 선언하기

하나의 컴포넌트 내에서 State Hook을 여러 개 사용할 수도 있습니다.

```jsx
function ExampleWithManyStates() {
  // Declare multiple state variables!
  const [age, setAge] = useState(42);
  const [fruit, setFruit] = useState('banana');
  const [todos, setTodos] = useState([{ text: 'Learn Hooks' }]);
  // ...
}
```

[배열 구조 분해(destructuring)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment#Array_destructuring) 문법은 `useState`로 호출될 state 변수들을 다른 변수명으로 할당할 수 있게 해줍니다. 이 변수명은 `useState` API와 관련이 없습니다. 대신에 React는 매 렌더링 때마다 `useState`를 사용한 순서대로 실행할 것입니다. 왜 이렇게 동작하는지는 나중에 살펴보겠습니다.

#### 근데 Hook이 뭔가요?

Hooks는 함수 컴포넌트에서 React state와 생명주기 기능(lifecycle features)을 "연동(hook into)"할 수 있게 해주는 함수입니다. Hooks는 클래스 안에서는 동작하지 않습니다 -- 대신 클래스 없이 React를 사용할 수 있게 해주는 것입니다. (하지만 이미 짜놓은 컴포넌트를 모조리 재작성하는 것은 [권장하지 않습니다](https://reactjs.org/docs/hooks-intro.html#gradual-adoption-strategy). 대신 새로 작성하는 컴포넌트부터는 Hooks를 이용하시면 됩니다.)

React는 `useState` 같은 내장 Hooks를 몇 가지 제공합니다. 컴포넌트 간에 stateful 로직을 재사용하기 위해서 custom Hooks를 만드는 것도 물론 가능합니다. 일단 내장 Hooks를 먼저 보겠습니다.

<section class="notice--warning">
  <h4>자세한 설명</h4>
  <p markdown="1">State Hook에 대해서는 독립된 문서 [Using the State Hook](https://reactjs.org/docs/hooks-state.html)에서 더 알아보세요.</p>
</section>

## ⚡️ Effect Hook

React 컴포넌트 안에서 데이터 가져오거나 구독하고, DOM을 직접 조작하는 작업을 이전에도 종종 해보셨을 것입니다. 우리는 이런 모든 동작을 "side effects"(또는 짧게 "effects")라고 합니다. 왜냐하면 이것은 다른 컴포넌트에 영향을 줄 수도 있고, 렌더링 과정으로는 구현할 수 없는 작업이기 때문입니다.

Effect Hook, 즉 `useEffect`는 함수 컴포넌트 내에서 이런 side effects를 수행할 수 있게 해줍니다. React 클래스의 `componentDidMount` 나 `componentDidUpdate`, `componentWillUnmount`와 같은 목적으로 제공되지만, 하나의 API로 통합된 것입니다. (`useEffect`와 이 세 가지 메소드를 비교하는 예시가 [Using the Effect Hook](https://reactjs.org/docs/hooks-effect.html) 문서에 있습니다.)

예를 들어, 이 예제는 React가 DOM을 업데이트한 뒤에 페이지의 title을 바꾸는 컴포넌트입니다.

```jsx
import { useState, useEffect } from 'react';

function Example() {
  const [count, setCount] = useState(0);

  // Similar to componentDidMount and componentDidUpdate:
  useEffect(() => {
    // Update the document title using the browser API
    document.title = `You clicked ${count} times`;
  });

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>Click me</button>
    </div>
  );
}
```

`useEffect`를 사용하면, React는 DOM을 바꾼 뒤에 "effect" 함수를 실행할 것입니다. Effects는 컴포넌트 안에 선언되어있기 때문에 props와 state에 접근할 수 있습니다. 기본적으로 React는 매 렌더링 이후에 effects를 실행합니다 -- 첫 번째 렌더링도 _포함해서요_. (클래스 생명주기(lifecycle)와 비교해서 어떻게 다른지는 [Using the Effect Hook](https://reactjs.org/docs/hooks-effect.html) 문서에서 더 다루고 있습니다.)

Effect를 "해제"할 필요가 있다면, 해제하는 함수를 return해주면 됩니다. 이는 선택적입니다(optional). 예를 들어, 이 컴포넌트는 친구의 온라인 여부를 구독하는 effect를 사용했고, 구독을 해지함으로써 정리(clean up)해줍니다.

```jsx
import { useState, useEffect } from 'react';

function FriendStatus(props) {
  const [isOnline, setIsOnline] = useState(null);

  function handleStatusChange(status) {
    setIsOnline(status.isOnline);
  }

  useEffect(() => {
    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);

    return () => {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });

  if (isOnline === null) {
    return 'Loading...';
  }
  return isOnline ? 'Online' : 'Offline';
}
```

이 예제에서, 컴포넌트가 unmount될 때 React는 `ChatAPI`에서 구독을 해지할 것입니다. 또한 재 렌더링이 일어나 effect를 재실행하기 전에도 마찬가지로 구독을 해지합니다. (만약 원한다면 `props.friend.id`가 바뀌지 않았을 때 [재구독을 건너뛰도록 설정](https://reactjs.org/docs/hooks-effect.html#tip-optimizing-performance-by-skipping-effects)할 수 있습니다.)

`useState`와 마찬가지로, 컴포넌트 내에서 여러 개의 effect를 사용할 수 있습니다.

```jsx
function FriendStatusWithCounter(props) {
  const [count, setCount] = useState(0);
  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });

  const [isOnline, setIsOnline] = useState(null);
  useEffect(() => {
    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });

  function handleStatusChange(status) {
    setIsOnline(status.isOnline);
  }
  // ...
```

Hooks을 사용하면 -- 구독을 추가하고 제거하는 로직 같은 -- 서로 관련있는 코드들을 한군데에 모아서 작성할 수 있습니다. 반면 클래스 컴포넌트에서는 생명주기 메소드(lifecycle methods) 각각에 쪼개서 넣어야만 했습니다. <span class="small">(역자 주: [Sunil Pai](https://twitter.com/threepointone)의 [글과 그림](https://twitter.com/threepointone/status/1056594421079261185)을 보면 차이를 한눈에 볼 수 있습니다.)</span>

<section class="notice--warning">
  <h4>자세한 설명</h4>
  <p markdown="1">`useEffect`에 대해서는 독립된 문서 [Using the Effect Hook](https://reactjs.org/docs/hooks-effect.html)에서 더 알아보세요.</p>
</section>

## ✌️ Hooks 사용 규칙

Hooks는 그냥 JavaScript 함수이지만, 두 가지 규칙을 준수해야 합니다.

- **최상위(at the top level)**에서만 Hooks를 호출해야 합니다. 반복문, 조건문, 중첩된 함수 내에서 Hooks를 실행하지 마세요.
- **React 함수 컴포넌트** 내에서만 Hooks를 호출해야 합니다. 일반 JavaScript 함수에서는 Hooks를 호출해서는 안됩니다. (Hooks를 호출할 수 있는 곳이 딱 한 군데 더 있습니다 -- 바로 직접 작성한 custom Hooks 내입니다. 이것에 대해서는 나중에 알아보겠습니다.)

이 규칙들을 강제하기 위해서 [linter plugin](https://www.npmjs.com/package/eslint-plugin-react-hooks)을 제공하고 있습니다. 이 규칙들이 제약이 심하고 혼란스럽다고 처음에는 느낄 수 있습니다. 하지만 이것은 Hooks가 제대로 동작하기 위해서는 필수적인 조건입니다.

<section class="notice--warning">
  <h4>자세한 설명</h4>
  <p markdown="1">이 규칙들에 대해서는 독립된 문서 [Rules of Hooks](https://reactjs.org/docs/hooks-rules.html)에서 더 알아보세요.</p>
</section>

## 💡 나만의 Hooks 만들기

개발을 하다보면 가끔 stateful 로직을 컴포넌트 간에 재사용하고싶은 경우가 생깁니다. 이 문제를 해결하기 위한 전통적인 방법이 두 가지 있었는데, [higher-order components](https://reactjs.org/docs/higher-order-components.html)와 [render props](https://reactjs.org/docs/render-props.html)가 바로 그것입니다. Custom Hooks는 이들 둘과는 달리 컴포넌트 트리에 새 컴포넌트를 추가하지 않고도 이것을 가능하게 해줍니다. <span class="small">(역자 주: [wrapper hell](https://medium.com/@jackyef/react-hooks-why-we-should-embrace-it-86e408663ad6#bfce)이라고 불리는 그것을 말합니다.)

친구의 온라인 여부를 구독하기 위해서 `useState`와 `useEffect` Hooks를 사용한 `FriendStatus` 컴포넌트 예제를 다시 한번 보겠습니다. 이 로직을 다른 컴포넌트에서도 재사용하고 싶다고 가정을 해봅시다.

먼저, 이 로직을 `useFriendStatus`라는 custom Hook으로 뽑아냅니다.

```js
import { useState, useEffect } from 'react';

function useFriendStatus(friendID) {
  const [isOnline, setIsOnline] = useState(null);

  function handleStatusChange(status) {
    setIsOnline(status.isOnline);
  }

  useEffect(() => {
    ChatAPI.subscribeToFriendStatus(friendID, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(friendID, handleStatusChange);
    };
  });

  return isOnline;
}
```

이 Hook은 `friendID`를 인자로 받아서 친구의 온라인 여부를 return해줍니다.

이제 우리는 이것을 여러 컴포넌트에서 사용할 수 있습니다.

```jsx
function FriendStatus(props) {
  const isOnline = useFriendStatus(props.friend.id);

  if (isOnline === null) {
    return 'Loading...';
  }
  return isOnline ? 'Online' : 'Offline';
}
```

```jsx
function FriendListItem(props) {
  const isOnline = useFriendStatus(props.friend.id);

  return <li style={{ color: isOnline ? 'green' : 'black' }}>{props.friend.name}</li>;
}
```

각 컴포넌트의 state는 완전히 독립적입니다. Hooks는 state 그 자체가 아니라, *stateful logic*을 재사용하는 방법입니다. 실제로 각각의 Hook *호출*은 완전히 독립된 state를 가집니다 -- 그래서 심지어는 한 컴포넌트 안에서 같은 custom Hook을 두 번 쓸 수도 있습니다.

Custom Hooks는 기능이라기보다는 컨벤션(convention)에 가깝습니다. 이름이 "`use`"로 시작하고, 안에서 다른 Hooks를 호출한다면 그 함수를 custom Hook이라고 부를 수 있습니다. `useSomething`이라는 네이밍 컨벤션은 linter 플러그인이 Hooks를 인식하고 버그를 찾을 수 있게 해줍니다.

폼 핸들링, 애니메이션, 선언적 구독(declarative subscriptions), 타이머 등 많은 경우에 custom Hooks를 사용할 수 있습니다. 우리는 React 커뮤니티에서 어떤 custom Hooks가 만들어질지 정말 기대됩니다.

<section class="notice--warning">
  <h4>자세한 설명</h4>
  <p markdown="1">Custom Hooks에 대해서는 독립된 문서 [Building Your Own Hooks](https://reactjs.org/docs/hooks-custom.html)에서 더 알아보세요.</p>
</section>

## 🔌 다른 내장 Hooks

보편적이지는 않지만 유용하다고 느낄만한 내장 Hooks가 몇 가지 더 있습니다. 예를 들어, [`useContext`](https://reactjs.org/docs/hooks-reference.html#usecontext)는 컴포넌트를 중첩하지 않고도 React context를 구독할 수 있게 해줍니다.

```jsx
function Example() {
  const locale = useContext(LocaleContext);
  const theme = useContext(ThemeContext);
  // ...
}
```

그리고 [`useReducer`](https://reactjs.org/docs/hooks-reference.html#usereducer)는 복잡한 컴포넌트들의 state를 reducer로 관리할 수 있게 해줍니다.

```jsx
function Todos() {
  const [todos, dispatch] = useReducer(todosReducer);
  // ...
```

<section class="notice--warning">
  <h4>자세한 설명</h4>
  <p markdown="1">모든 내장 Hooks에 대해서는 독립된 문서 [Hooks API Reference](https://reactjs.org/docs/hooks-reference.html)에서 더 알아보세요.</p>
</section>

## 다음 단계

휴, 정말 순식간이었죠! 만약 잘 이해가 안되는 부분이 있거나 좀 더 깊이 공부하고 싶으면 [State Hook](https://reactjs.org/docs/hooks-state.html)부터 시작해서 다른 문서들을 읽어보시기 바랍니다.

또한 [Hooks API reference](https://reactjs.org/docs/hooks-reference.html)와 [Hooks FAQ](https://reactjs.org/docs/hooks-faq.html)도 참고하시기 바랍니다.

마지막으로, 우리가 _왜_ Hooks를 추가하는지 그 이유와, -- 앱을 재작성하지 않고도 -- 클래스와 함께 Hooks를 사용하는 방법을 설명한 [소개 페이지](https://reactjs.org/docs/hooks-intro.html)도 놓치지 마세요.

{% endraw %}
