title: react native 구조짜기
author: Cheol Kang
tags:
  - react-native
  - korean
  - architecture
categories:
  - react-native
date: 2016-09-17 17:21:00
---
## 1. Atomic Design
저 같은 경우에는 atomic Design을 채용하였습니다. atomic design에 대한 간단한 설명은 [여기](http://bradfrost.com/blog/post/atomic-web-design/) 을 참조해 주시길 바랍니다. atomic design은 원래 react component의 구조를 짤때 사용하는 디자인 패턴입니다. `atom => molecule => organism => template => page` 순으로 구조를 짜아 올리게 됩니다. 하지만 저같은 저런식으로 구조를 짤 경우 훨씬더 `react component` 들의 재활용성이 높아지고 또한 코드 가독성도 높아집니다. 하지만 template 나 page는 web의 환경에 맞춰진 구조 입니다. web은 헤더나, 사이드바는 그대로 있고 나머지 부분만 바뀌는 경우가 많기 때문에 template, page 구조를 사용하지만 mobile은 대부분의 경우 화면전체가 바뀌는 경우가 많기 때문이죠. 그래서 저는 `atomic design`을 약간 변형하여서 `atom => molecule => organism => screen` 구조를 채용했습니다. 또한 web과 다른 환경때문에 이 구조에서 screen은 매우 특별한 역할을 가지게 됩니다. 밑에서 계속 설명을 하겠습니다.

## 2. redux-persist
일반적으로 `react`는 항상 `redux`와 궁합이 좋습니다. 이건 `react-native`에서도 예외가 아니죠. 하지만 native환경은 역시 web과 큰 차이점을 갖습니다. 일반적인 경우 web은 대부분의 데이터는 서버에 있고 local에 데이터를 거의 저장을 하지 않습니다. 하지만 native에서는 local에 데이터를 저장하는것은 매우 흔한경우입니다. 또한 사용자가 앱을 실행했을때는 전에 앱을 중단했던 지점부터 다시 이어서 사용하는 경우가 많습니다. 그렇기 때문에 현재의 앱의 상태를 항상 어딘가에 저장을 해야 하고 그 방법으로써 `redux-persist`을 사용합니다.

## 3. react-native-router-flux
저 같은 경우는 navigation을 위해서 [react-native-router-flux](https://github.com/aksonov/react-native-router-flux)을 사용했습니다. react-native-router-flux 는 매우 사용하기 쉽고 복잡하게 screen이동이 얽혀 있을때도 매우 알아 보기 쉽습니다. 또한 수많은 부가 기능들은 더욱 navigation을 쉽게 만들어 줍니다.

## 4. redux-persist + atomic design + react-native-router-flux
일반적인 web의 redux는 `component`의 `state`을 전혀 사용하지 않고 항상 `redux`의 `state`만을 사용하는 경우가 많습니다. 하지만 우리는 native라는 환경 때문에 `redux-persist` 을 사용합니다. redux-persist을 사용할 경우 빈번한 `action`은 속도 저하가 발생할 수 있습니다. 그렇기 때문에 저는 `component`의 `state`와 `redux`의 state 둘다 사용하도록 하였습니다. 하지만 이렇게 될 경우 각각의 state가 어디에서 오는 state인지 알기 더욱 어렵게 되기 때문에 몇가지 규칙을 만들었습니다. 
1. component의 state는 항상 screen만 갖는다.(에외. animation을 위한 component state는 예외로 한다)
2. redux의 state, action도 항상 screen만 갖는다.
3. component의 state는 빈번하게 바뀌는 값, 지워져도 되는 값에 대해 사용을 합니다. (예를 들어 유저의 textInput 입력값등은 자주 바뀌고 save Button등에 어떠한 의미있는 데이터가 되기 전까지는 지워져도 상관이 없을것 같습니다. 저 같은 경우는 user입력이 있는 대부분의 form은 `component state`을 이용해서 처리했습니다.
4. `redux state`는 저장해야하는 user의 data을 위해 사용했습니다. (예를들어 app의 설정부분, user의 세션, user가 자주 사용하는 데이터등)
5. screen 이외의 component에서 data가 필요하다면 screen에서 props을 통해 받는다. 
6. react-native-router-flux의 action 또한 screen에서만 사용한다.

이렇게 함으로써 우리는 screen component 들만 봐도 앱의 전체적인 흐름을 쉽게 캐치 할수 있습니다. 하지만 이 구조는 
1. screen component가 너무 비대해짐.
2. screen component만 읽어도 구조를 알수 있지만 screen component 을 읽는게 힘듬.

이 됩니다. 이걸 해결하기 위해서는 screen에서는 자잘한 작업을 하지 않고 각 작업들을 잘 분리 하여 screen에서는 그것들을 조합하는 형태로만 하는게 좋습니다.
예를들어 network통신은 api라는 폴더를 따로 만들어서 대부분의 처리는 여기서 하고 screen을 이걸 사용하거나, 각각의 데이터들도 class화 시켜서 data의 처리는 여기서 하게 만드는 등. 작업이 필요하게 됩니다.

```js
    + src // js 파일
        + actions // redux에서 사용하는 action들이 들어 있습니다
        + api // network 통신은 여기에서 합니다
        + asset // 사진등 local 파일이 들어갑니다
        + components // 실제 react component 들이 들어갑니다. atomic Design을 참조 하시길 바랍니다
            + atoms
            + molecules
            + organisms
            + screens 
        + env // firebase 나 network통신의 기본 설정 등이 들어있습니다.
        + reducers // redux에서 사용하는 reducers가 들어 있습니다.
        + schools // 학교들에 관한 정보가 들어있습니다. 현재는 crawl에 대한 부분이 들어있습니다.
        + store // redux의 store 정의 합니다.
        + structure // data의 class을 정의합니다.
        + style
            - size.js // 기본적인 size을 정의합니다. fontSize등은 통일성을 위해서 여기에서 가져옵니다.
            - color.js // component 에서 사용하는 색에 대해 정의 합니다. 모든 색은 여기에서 가져옵니다.
        + utils // 자주 쓰는 함수, 여러곳 에서 사용되는 함수등을 정의합니다.
        - app.js // 2번째로 실행되는 파일입니다. 필요한 설정이 있을경우 추가합니다.
        - navigator.js // 3번째로 실행되는 파일입니다. 전체적인 route가 들어 있습니다.
        - setup.js // 가장 먼저 실행되는 파일입니다. reducer의 store등을 정의합니다.
```
저같은 경우는 이런식으로 구조를 짜서 만들고 있습니다. 

다음편에서는 실제로 이 구조를 어떻게 사용하는지에 대해 보여드리겠습니다.