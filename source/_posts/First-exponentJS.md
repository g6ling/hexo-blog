title: exponentjs(react-native)의 감상
author: Cheol Kang
tags:
  - react-native
  - korean
categories:
  - react-native
date: 2016-10-12 17:47:00
---
# exponentjs(react-native)의 감상

exponent란 react-native로 만들어진 `프레임워크`이다. [홈페이지](https://getexponent.com/) 로 들어가면 동영상도 있고, 자세한 설명이 있으니 궁금하면 들어가서 읽어보자.


대충 설명하면 말 그대로 react-native 로 만들어진 `프레임워크`인데, 굉장히 편한 기능들이 많이 있다. 일단 ![](https://docs.getexponent.com/_images/xde-signin-success.png)

exponent xde 라는 툴로 관리를 하는데 여기서 프로젝트를 만들고 전체적인 로그들도 볼수 있으며, publish 을 통해 codepush 같은 실시간 앱 업데이트를 할 수 있다. 또한 sendlink라는 것을 통해서 개발중인 앱을 여러 사람들에게 테스트를 하는것도 간단한데, 그건 좀 있다 설명하도록 하겠다. 

일단 xde에서 프로젝트를 만들면 

![](https://cloud.githubusercontent.com/assets/8134523/19301693/b3e4a8fa-909b-11e6-9c9c-2082365aa441.png)

이렇게 폴더가 생성되는데 React-Native 로 프로젝트를 생성했을때와 꽤나 다르다. 일단 ios, android라는 폴더가 없고, 또한 index.ios.js, index.android.js 가 아니라 main.js 한개로 통일되어 있다. ios, android폴더가 없다는 것이 exponent의 가장 큰 특징이다. 일단 main.js 1개만 있는데 ios.js 나 android.js을 아예 지원을 하는지 안하는지가 좀 궁금하긴 한데 아마 지원을 할 것 같다. (베이스가 RN이여서) 하지만 exponent 에서는 1개의 js파일로 양쪽모두에서 사용하는 것을 지향하는 듯 하다. 

그리고 exp.json 이라는 파일이 있는데 이게 중요한 부분이다. 

```json
{
  "name": "exponenttest",
  "description": "An empty new project",
  "slug": "exponenttest",
  "sdkVersion": "10.0.0",
  "version": "1.0.0",
  "orientation": "portrait",
  "primaryColor": "#cccccc",
  "iconUrl": "https://s3.amazonaws.com/exp-brand-assets/ExponentEmptyManifest_192.png",
  "notification": {
    "iconUrl": "https://s3.amazonaws.com/exp-us-standard/placeholder-push-icon-blue-circle.png",
    "color": "#000000"
  },
  "loading": {
    "iconUrl": "https://s3.amazonaws.com/exp-brand-assets/ExponentEmptyManifest_192.png",
    "hideExponentText": false
  },
  "packagerOpts": {
    "assetExts": ["ttf", "mp4"]
  }
}

```

대략 이런 구성이 되어 있는데, 여기서 앱의 이름, 설명, icon, notification, loading 화면등 여러가지 설정을 할 수 있다. 원래의 RN이라면 이러한 설정은 ios 의 xcode프로젝트나, android 의 gradle, xml 파일 등을 수정하여서 설정을 해야 하지만 exponent 에서는 이걸 매우매우 쉽게 할 수 있게 해준다. 여기서 설정을 하면 exponent가 자동으로 ios, android 파일을 수정하여서 실제 빌드 할 때는 우리가 원하는 어플이 만들어 진다. 이 파일에 대해 더 많은 설명이 궁금하면 [설정파일 설명](https://docs.getexponent.com/versions/v10.0.0/guides/configuration.html) 을 참조하도록 하자.

이것 외에도 꽤나 지원하는 기능이 막강하여서 FaceBook Login등 귀찮은 설정이 필요한 기능들도 바로 쓸수 있으며, 그 외에 RN에서 기본으로 지원하지 않는 svg, custom font, imagePicker, AppLoading, GPS, Contacts 등 전부다 꿀 같은 기능들을 바로 제공을 해주어서 외부 라이브러리를 사용할 때 필요한 설정들을 하나도 안건드려도 된다. [여기](https://docs.getexponent.com/versions/v10.0.0/sdk/index.html) 을 보면 지원하는 많은 기능들을 볼 수 있다. 

이런 기능들이 전부다 쉽게 가능한 이유는 ios/ android 폴더가 없기 때문이다. exponent에서 자동으로 생성하기 때문에 정말로 JS파일만 건드리면 왠만한 어플을 만들 수 있다. 하지만 단점은 JS파일만 건드릴수 있다는 거다. 

이게 장점이자 단점인데, 만약 커스텀 라이브러리를 사용하고 싶다던가, native code을 수정하고 싶다고 해도 할 수가 없다. React Native의 최대 장점인 native Code와의 연동이 자유롭다의 장점이 사라지고, 쉽고 쓰기편한 React Native가 된 느낌이다. exponent의 말을 들어보면 

> You can do more in just JavaScript than you realize--and in fact, we recommend doing as much in JS as possible, since it will immediately work across both platforms and is less fragile--so you shouldn't need to include UI widgets that are written as native modules (there are almost always good pure JavaScript alternatives that are better).

라고 하신다. 

### exponent 을 쓰면 좋을 상황

사실 안 좋을 상황을 제외하면 다 좋을 것 같다. 처음부터 들어있는 기능이 너무 막강해서 아마 대부분의 어플의 경우에는 문제없이 다 해결될 거 라고 보인다. 문제는 exponent에서 react-native로 다시 migration이 어느정도 되냐 인데, js파일 자체는 대부분 공유 할 수 있으니, 라이브러리만 잘 추가 한다면 나중에라도 충분히 migration이 가능하기 때문에 처음의 빠른 개발을 더욱 빠르게 할 때는 좋아 보인다. 

### 안좋을 상황 

사실 뻔하긴 한데, native 코드를 바꾸는 라이브러리을 사용해야할 경우이다. (모든 라이브러리가 안되는게 아니라 단순히 js만 사용하는 라이브러리면 문제 없다). 

### 성능

RN이랑 똑같다. 오히려 RN에서 여러가지 라이브러리를 덕지덕지 하는거 보다, exponent 팀에서 만든 API을 사용하는게 더 성능이 좋다고 생각된다.

exponent 팀이 말하는 장점

- **Support for iOS and Android**

  You can use apps written in Exponent on both iOS and Android right out of the box. You don't need to go through a separate build process for each one. Just open any Exponent app in the Exponent Developer app from the App Store on either iOS or Android (or in a simulator or emulator on your computer).

- **Push Notifications**

  Push notifications work right out of the box across both iOS and Android, using a single, unified API. You don't have to set up APNS and GCM/FCM or configure ZeroPush or anything like that. We think we've made this as easy as it can be right now

- **Facebook Login**

  This can take a long time to get set up properly yourself, but you should be able to get it working in 10 minutes or less on Exponent.

- **Instant Updating**

  All Exponent apps can be updated in seconds by just clicking Publish in XDE. You don't have to set anything up; it just works this way. If you aren't using Exponent, you'd either use Microsoft Code Push or roll your own solution for this problem

- **Asset Management**

  Images, videos, fonts, etc. are all distributed dynamically over the Internet with Exponent. This means they work with instant updating and can be changed on the fly. The asset management system built-in to Exponent takes care of uploading all the assets in your repo to a CDN so they'll load quickly for anyone.

  Without Exponent, the normal thing to do is to bundle your assets into your app which means you can't change them. Or you'd have to manage putting your assets on a CDN or similar yourself.

- **Easier Updating To New React Native Releases**

  We do new releases of Exponent every few weeks. You can stay on an old version of React Native if you like, or upgrade to a new one, without worrying about rebuilding your app binary. You can worry about upgrading the JavaScript on your own time.

위에서 말하는 기능 1개만 구현 할려고 해도 처음에는 하루가 넘게 걸리는 설정이다. 정말로 개발 효율을 엄청나게 올려주기는 하는데, 반대로 말하면 전부다 다른 라이브러리로 구현이 가능하다는 말이 된다.  그리고 어떻게 보면 가장큰 문제점 이기도 한데, exponent 구동 방식자체가 좀 이상하다. 별다른 설정없이 전부다 구현 할 수 있다는 건 결국 어딘가에서 많은 설정을 했다는 건데 exponent로 만들어진 어플을 설치하면 exponent 어플, 내가 설치한 어플, 총 2개의 어플이 설치된다. 

작동방식이 exponent 어플 위에서 내가 만든 js코드가 실행되는 느낌이다. 그래서 사용자는 exponent 라는 이유모를 어플이 핸드폰 안에 있어야 되는 상황이다. 이게 아마 내가 exponent을 사용하지 않을 가장 큰 이유 되시겠다. 

### exponent 가 앞으로 어떻게 될까?

처음 만졌을 때는 React Native 의 미래버전 0.0.1 버전 정도 느낌? 근데 react-native가 이렇게 되는 느낌보다는 react-native link가 훨씬 많은 것들이 가능해서, 여러 라이브러리를 쉽게 설정이 가능하게 되는게 훨씬 빠를것 같다. 지금 당장 또 못쓸정도의 물건은 아니여서, 회사가 아니라, 개인적으로 취미로 어플을 만드는 정도에는 꽤나 쓸만한 물건같다. 또한 react-native 로 알파버전 어플을 만들거나, react-native로 처음 개발을 시작할 사람에게도 추천할 정도 일것 같다. 