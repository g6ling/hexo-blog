title: firestack을 사용할때 android error 처리
author: Cheol Kang
tags:
  - react-native
  - error
categories:
  - react-native
date: 2016-09-25 03:04:00
---
## Android에서 Firebase [DEFAULT] error가 날때
그냥 맘편하게 `google-service.json`을 추가한다. firestack에서는 안써도 된다고 하는데 auth에서는 써야하는것 같다. 그러기 위해서 `app.gradle` 맨 밑에 `apply plugin: 'com.google.gms.google-services'` 을 추가. `project.gradle` 의 `dependencies`에 `classpath 'com.google.gms:google-services:3.0.0'`을 추가. firebase console에 들어가서 `google-service.json`을 다운 추가.