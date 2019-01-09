---
categories:
  - Tooling
  - Babel
tags:
  - Babel
  - JavaScript
  - Tooling
  - Translation
date: 2019-01-02T00:00:00+09:00
last_modified_at: 2019-01-03T01:28:00+09:00

title: babel-plugin-macros로 무설정 코드 변환
description: babel-plugin-macros가 무엇인지, 이것이 어떻게 생산성을 올려주는지 설명합니다.
excerpt: babel-plugin-macros가 무엇인지, 이것이 어떻게 생산성을 올려주는지 설명합니다.
---

{% raw %}

<p class="notice--info" markdown="1">
이 글은 [Kent C. Dodds](https://twitter.com/kentcdodds)의 [Zero-config code transformation with babel-plugin-macros
](https://babeljs.io/blog/2017/09/11/zero-config-with-babel-macros)를 번역한 글입니다.
</p>

Babel의 출발점은 최신 ECMAScript 스펙으로 작성된 코드를 구형 환경에서 실행될 수 있도록 해주는 트랜스파일러였습니다. 그러나 이제 Babel은 그 이상이 되었습니다. [Tom Dale](https://twitter.com/tomdale)과 저는 ["Compilers are the New Frameworks"](https://tomdale.net/2017/09/compilers-are-the-new-frameworks/)에 대해 전적으로 동의합니다. 라이브러리와 프레임워크를 위한 컴파일 타임 최적화에 대한 것들이 점점 많아지고 있습니다. 언어의 문법을 확장하는 것뿐만 아니라, 간단하지만 다른 방식으로는 실현하기 어려운 코드 변환(code transformations)까지 말입니다.

제가 컴파일러 플러그인을 좋아하는 가장 큰 이유는, 사용자 경험과 개발자 경험을 동시에 개선할 수 있기 때문입니다(["How writing custom Babel & ESLint plugins can increase productivity & improve user experience"](https://medium.com/@kentcdodds/how-writing-custom-babel-and-eslint-plugins-can-increase-your-productivity-and-improve-user-fd6dd8076e26)를 더 읽어보세요).

하지만 저는 Babel 플러그인이 몇 가지 문제를 가지고 있다고 생각합니다.

1. 어떤 프로젝트의 코드를 읽을 때, 코드를 변환하는 플러그인이 존재하는지 파악하지 못할 수도 있습니다.
2. (`.babelrc` 이나 webpack 설정 파일(configuration) 같이) 전역적이거나 코드 바깥에서 설정되어야 합니다.
3. (Babel의 AST를 통해) 모든 Babel 플러그인은 동시에 실행되기 때문에, 매우 혼란스럽게 충돌할 수 있습니다.

만약 코드에서 직접 Babel 플러그인을 import해서 사용한다면 이 문제들을 해결할 수 있습니다. 이것은 변환이 더 명시적이고, 설정 파일에 추가할 필요도 없으며, 플러그인을 import한 순서대로 적용된다는 것을 뜻합니다. 멋지지 않나요!?!?

## [`babel-plugin-macros`](https://github.com/kentcdodds/babel-plugin-macros) 소개 🎣

그거 아세요? 이미 이런 도구가 존재한다는 것을요! `babel-plugin-macros`는 우리가 얘기한 것을 정확하게 구현해주는 새로운 Babel 플러그인입니다. 이것은 코드 변환에 대한 "새로운" 접근 방식입니다. 이것은 무설정(zero-config)이고, import 가능한 코드 변환입니다. 이것은 [create-react-app 이슈](https://github.com/facebookincubator/create-react-app/issues/2730)에서 [Sunil Pai](https://twitter.com/threepointone)가 제안한 [아이디어](https://github.com/threepointone/babel-macros)에서 영감을 받았습니다.

그래서 그게 어떻게 생겼냐고요? 놀라지 마세요! 지금 당장 사용할 수 있는 `babel-plugin-macros` 패키지가 이미 몇 가지 나와 있습니다!

여기 실제 사례가 있습니다. [Next.js](https://github.com/zeit/next.js)로 만들어진 [범용 애플리케이션](https://github.com/kentcdodds/glamorous-website)에서 SVG 파일을 인라인으로 변환하기 위해 [`preval.macro`](https://github.com/kentcdodds/preval.macro)를 사용하고 있습니다.

```javascript
// search.js
// this file runs in the browser
import preval from 'preval.macro';
import glamorous from 'glamorous';

const base64SearchSVG = preval.require('./search-svg');
// this will be transpiled to something like:
// const base65SearchSVG = 'PD94bWwgdmVyc2lv...etc...')

const SearchBox = glamorous.input('algolia_searchbox', (props) => ({
  backgroundImage: `url("data:image/svg+xml;base64,${base64SearchSVG}")`,
  // ...
}));

// search-svg.js
// this file runs at build-time only
// because it's required using preval.require function, which is a macro!
const fs = require('fs');
const path = require('path');

const svgPath = path.join(__dirname, 'svgs/search.svg');
const svgString = fs.readFileSync(svgPath, 'utf8');
const base64String = new Buffer(svgString).toString('base64');

module.exports = base64String;
```

어떤 점이 멋지냐고요? 음, 원래 방식대로 구현한다면 위에서 말한 문제들이 그대로 재현될 것입니다.

1. 코드에 `import preval from 'preval.macro'` 같은 라인이 없기 때문에 덜 명시적(less explicit)입니다.
2. Babel 설정 파일에 `babel-plugin-preval`을 추가해야 합니다.
3. `preval`을 전역변수로 허용하도록 ESLint 설정 파일을 업데이트 해야 합니다.
4. 만약 `babel-plugin-preval`을 잘못 설정한다면, `Uncaught ReferenceError: preval is not defined` 같은 알아보기 힘든 **런타임** 에러가 발생합니다.

`babel-plugin-macros`와 `preval.macro`를 함께 사용한다면, 이 모든 문제가 해결됩니다.

1. 코드에 import가 존재하므로 명시적입니다.
2. `babel-plugin-macros`를 설정 파일에 추가하긴 해야 하지만 딱 한 번이면 됩니다. 그럼 어디에서든지 매크로를 사용할 수 있습니다(심지어는 지역 매크로도).
3. 명시적으로 사용하기 때문에, ESLint 설정 파일을 업데이트할 필요가 없습니다.
4. 만약 `babel-plugin-macros`를 잘못 설정한다면, [훨씬 친절한 **컴파일 타임** 에러 메시지](https://github.com/kentcdodds/babel-plugin-macros/blob/f7c9881ee22b19b3c53c93711af6a42895ba1c71/src/__tests__/__snapshots__/index.js.snap#L100)가 출력되는데, 구체적으로 무엇이 문제인지를 알려주고 관련 문서까지 제공합니다.

**그게 무슨 뜻이냐고요? 한마디로 말해서, `babel-plugin-macros`는 Babel 변환(transforms)을 만들거나 사용하는 더 간단한 방법입니다.**

[이미 공개된 `babel-plugin-macros`](https://www.npmjs.com/browse/keyword/babel-plugin-macros)가 몇 가지 있습니다. [`preval.macro`](https://github.com/kentcdodds/preval.macro), [`codegen.macro`](https://github.com/kentcdodds/codegen.macro), [`idx.macro`](https://github.com/dralletje/idx.macro), [`emotion/macro`](https://github.com/emotion-js/emotion/blob/master/docs/babel.md#babel-macros), [`tagged-translations/macro`](https://github.com/vinhlh/tagged-translations#via-babel-macros), [`babel-plugin-console/scope.macro`](https://github.com/mattphillips/babel-plugin-console#macros) 등이 있고, [`glamor`도 곧 지원할 예정 🔜](https://github.com/threepointone/glamor/pull/312)입니다.

### 다른 예제

`babel-plugin-macros`는 Babel 플러그인 -- 문법 관련 플러그인 제외 -- 을 설정 없이 사용할 수 있게 해줍니다. 이미 존재하는 수많은 Babel 플러그인들이 매크로로 구현될 수 있습니다. 여기 [`babel-plugin-console`](https://github.com/mattphillips/babel-plugin-console)이라는 다른 예제가 있습니다. 이것은 [macro 버전](https://github.com/mattphillips/babel-plugin-console/blob/master/README.md#macros)도 함께 제공하고 있습니다.

```javascript
import scope from 'babel-plugin-console/scope.macro';

function add100(a) {
  const oneHundred = 100;
  scope('Add 100 to another number');
  return add(a, oneHundred);
}

function add(a, b) {
  return a + b;
}
```

이제, 이 코드가 실행될 때 `scope` 함수는 멋진 일을 합니다.

**Browser:**

![Browser console scoping add100](https://github.com/mattphillips/babel-plugin-console/raw/53536cba919d5be49d4f66d957769c07ca7a4207/assets/add100-chrome.gif)

**Node:**

<img alt="Node console scoping add100" src="https://github.com/mattphillips/babel-plugin-console/raw/53536cba919d5be49d4f66d957769c07ca7a4207/assets/add100-node.png" width="372">

멋지지 않나요? 그냥 다른 의존성 모듈을 사용하듯이 매크로를 사용하면 됩니다. 위에서 말한 모든 장점은 덤으로 따라옵니다.

## 결론

지금까지 얘기한 것은 `babel-plugin-macros`가 할 수 있는 일의 빙산에 일각에 불과합니다. 저는 이것이 [create-react-app](https://github.com/facebookincubator/create-react-app/issues/2730)에 추가되어서, `create-react-app`을 무설정으로 훨씬 강력하게 사용할 수 있게 되기를 바랍니다. 그리고 점점 더 많은 Babel 플러그인들이 `macro`를 제공하고 있어서 정말 기쁩니다. 앞으로 각자가 프로젝트의 니즈에 맞는 매크로를 만들어가는 모습을 볼 수 있기를 기대합니다.

**매크로를 만드는 것은 일반적인 Babel 플러그인을 만드는 것보다 훨씬 쉽습니다**. 하지만 AST와 Babel 자체에 대한 지식이 더 필요합니다. 만약 이런 게 처음이라면, 여기에 당신을 위한 [몇](https://kentcdodds.com/talks/#writing-custom-babel-and-eslint-plugins-with-asts) [가지](https://github.com/thejameskyle/babel-handbook) [글](https://kentcdodds.com/workshops/#code-transformation-and-linting)이 있습니다. 😀

행운을 빕니다! 👋

추신. 언어에서 매크로는 전혀 새로운 컨셉이 아니며, 오래전부터 존재했습니다. 사실 [JavaScript용 매크로](http://sweetjs.org/)도 이미 존재하고, 심지어는 [Babel 플러그인으로 구현된 것](https://github.com/codemix/babel-plugin-macros)도 있습니다. 하지만 `babel-plugin-macros`의 접근법은 조금 다릅니다. 매크로는 주로 언어에 새로운 문법을 추가하기 위해 사용되지만, `babel-plugin-macros`의 목적은 그런 게 아닙니다. `babel-plugin-macros`는 코드 변환에 관한 것입니다.

{% endraw %}
