title: React Native 에서 만나는 Firebase Error
author: Cheol Kang
tags:
  - react-native
  - korean
categories:
  - react-native
date: 2016-10-29 18:11:00
---

### [Firebase] does not exist

Firebase 가 존재 하지 않아서 생기는 에러이다. Firebase initialize을 통해서 Firebase 을 만들어주자.

### [Firebase] already exist

이미 Firebase가 정의 되었는데, 또 다시 정의하는 경우다. 보통 2가지 경우가 있다.

#### Firebase을 파일마다 정의하는 경우.

Firebase을 이미 정의 했는데 다른 파일에서도 정의하는 경우이다. 이 경우에는 Firebase을 정의하는 파일을 1개 만들고 거기서 다른 파일들은 그 파일을 import 하는 방법으로 하면 된다.

[이 글](http://stackoverflow.com/questions/37337080/initialize-firebase-references-in-two-separate-files-in-the-new-api)을 참조하면 된다.

#### React Native 에서 reload할 때 Firebase 정의를 다시 하는 경우

결국 React Native 에서 앱을 실행한다면 index 을 거쳐서 앱을 실행하게 되는데 그 과정에 Firebase을 정의할 것이다. 저 같은 경우는 index.js -> setup.js -> app.js -> navigator.js 을 통해서 앱이 실행되는데 app.js의 constructor에서 Firebase을 정의 하도록 했는데 에러가 발생했다. 에러는 Android에서 앱을 종료하고 다시 킬때 발생되었다. 안드로이드에서 뒤로가기 버튼을 통해서 앱을 종료하면 앱이 메모리 상에 남아 있다. 또 오랫동안 앱을 사용하지 않을경우에도 앱을 다시 킬 경우 reload되는 경우가 있는데 여기서 문제가 발생한다. React Native에서 앱을 Sleep을 하는 경우는 아마 모든 component을 unMount하는 것 같다.  하지만 Firebase 는 component가 unMount 된다고 해도 남아 있기 때문에 앱을 다시 리로드할때, Firebase을 다시 정의하게 된다. 만약 완전히 앱을 종료햇다면 이 에러가 발생되지 않는다.

길게 적었지만 해결 방법은 간단하다. 
```js
    firebase.initializeApp(firebaseConfig);
```
```js
    if (firebase.apps.length === 0) {
          firebase.initializeApp(firebaseConfig);
        }
```
위의 코드를 아래 처럼 바꾸면 된다. 이미 firebase가 실행되지 않았을 때만 정의를 한다. 
