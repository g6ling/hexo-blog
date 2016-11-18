title: Promise 실행순서 이해하기
author: Cheol Kang
tags:
  - tech
  - es6
  - korean
categories:
  - tech
date: 2016-11-18 17:51:00
---
여러 글에서 `Promise`의 사용법 등은 많은 글이 있지만 `Promise`가 중복되었을 때의 `chain`에 대해서 잘 정리한 글이 없어서 정리해봅니다. 일단 밑의 코드부터 살펴보죠.(`Promise`에 대한 개념은 많은 다른 글들을 참조해주세요)


{% gist g6ling/bbc08b8f217d50e72e4873dab056ed3f %}

을 보면 왜 이런 결과가 되는 걸까요?

이걸 알기 위해서는 일단 자바스크립트가 비동기를 어떻게 실행하는지를 먼저 알아야 합니다. 밑에 코드부터 살펴보도록 하죠

```js
setTimeout(function() {
	console.log(2)
},0)
console.log(1)
// result
// 1
// 2
```
단순히 생각하면 0초뒤에 2가 출력되어야 해서 2,1 순서의 결과가 나올거 같지만 다릅니다. 왜 그럴까요? 알기 위해서는 JS가 어떤식으로 비동기를 처리하는지 알아야 합니다.

JS의 실행 방식을 간단하게 말하면 
> 1. JS코드를 실행. 만약 이벤트 발생 코드(비동기코드)가 있다면 stack에 넣음.
> 2. 1번 다 실행이 되었다면, stack을 돌면서 이번프레임에 발생한 이벤트를 찾음.
> 3. 2에서 찾은 이벤트에 알맞은 코드를 실행. 이 과정에서 다시 이벤트 발생 코드들이 있다면 1처럼 stack에 넣음
> 4. event stack이 빌때까지 2-3을 실행

입니다. 1은 JS 코드를 절차지향적으로 실행하게 되죠. 이 과정에서 만약 `setTimeout`과 같은 비동기 코드가 있다면 stack에 차곡차곡 쌓게 됩니다.
그 다음 2는 stack에서 이번프레임에 발생한 이벤트를 찾는건데 예를 들어 `setTimeout 0 `같은 이벤트라면 1의과정이 끝나고 바로 발생한 이벤트가 되겠지요. 3은 그 코드를 실행합니다. 이걸 event stack이 빌때까지 계속 실행하게 됩니다. 

이제 위의 코드를 다시 보도록 하죠. 그렇다면 일단 1을 실행하기 때문에 `setTimeout`은 스택에 들어가게 됩니다. 그리고 `console.log(1)`을 실행하겠죠. 그리고 2로 넘어가서 `setTimeout 0 `이기 때문에 바로 `setTimeout`안의 코드가 실행되고 `console.log(2)`가 실행됩니다. 

이제 좀더 복잡한 예제를 보도록 하죠.
```js
setTimeout(function() {
	console.log(2)
	setTimeout(function() {
		console.log(4)
	})
},0)
setTimeout(function() {
	console.log(3)
},0)
console.log(1)
// result
// 1
// 2
// 3
// 4
```
일단 `setTimeout`2개가 stack에 들어가고 `console.log(1)`이 실행 됩니다.
그 후에 먼저 stack에 들어간 위의 `setTimeout`이 실행되고 `console.log(2)`가 출력 안의 `setTimeout`이 event stack에 들어가게 됩니다. (이 setTimeout은 다음 프레임에 실행이 되게 됩니다) 이제 밑의 `setTimeout`을 실행합니다. `console.log(3)`가 실행. 4로 가서 아직 eventStack이 남아있으니 다음프레임으로 넘어갑니다. 다음프레임에서 아까넣었던 `setTimeout`이 이벤트를 발생하고 실행하게 되면 `console.log(4)`을 실행합니다. 그래서 결과는 `1,2,3,4`가 됩니다.

그렇다면 다시 이 코드를 보도록하죠.
{% gist g6ling/bbc08b8f217d50e72e4873dab056ed3f %}

여기서 
```js
	_promise2(true)
	.then(function (text) {
		console.log(text);
	})
	console.log(text);
```

이 코드는 단순히 `_promise2` 을 event stack에 넣게만 됩니다. 그리고 `console.log`을 실행하게 되고 이 코드 부분 자체가 끝나게 됩니다. 그렇기 때문에 바로 뒤에 있는 
```js
.then(function() {
	console.log('ok3')
});
```
가 실행되고 3초뒤에 `_promise2`가 이벤트를 발생하게 됩니다. 결과는 `ok1, ok3, ok2`가 됩니다.

그렇다면 `ok1,ok2,ok3`을 출력하기 위해서는 어떻게 하면 될까요? 

{% gist g6ling/ecf2a15cd1afa8ab10047274bc4c914d %}

이렇게 `return promise2`을 한다면 결과는 `ok1,ok2,ok3`가 됩니다.

```js
_promise1(true)
.then(function (text) {
	console.log(text);
	return _promise2(true)
            .then(function (text) {
                console.log(text);
            }).then(function() {
                console.log('ok3')
            });
});
```
위의 코드와 밑의 코드는 같은 결과를 보여줍니다.  물론 
```js
_promise1(true)
.then(function (text) {
	console.log(text);
	return _promise2(true)
            .then(function (text) {
                console.log(text);
            });
}).then(function() {
	console.log('ok3')
});
```
도 같은 결과 입니다. 대충 이해가 되셧나요?


{% gist g6ling/0c312b7cfb50a70d83d01bf9b7e4c025 %}
는 params에 따라 `return`을 바꿔줌으로써 실행되는 flow을 다르게 합니다. 
프레임별로 설명을 해보도록 하죠.
 - 1프레임: promise1 코드를 실행합니다. setTimeout이 있군요. 이벤트 스택에 넣습니다.
 - 2프레임: 아무 이벤트도 없군요. 다음프레임으로 갑시다.(이번에는 `setTiemout 0`가 아니기 때문에 아무 이벤트가 없습니다. 이벤트는 `setTimeout 100`인 프레임에서 생기게 됩니다.
 - `setTimeout 100`이벤트가 일어나기 전까지의 프레임: 2프레임과 같이 아무것도 안하고 이벤트를 기다립니다.
 - `setTimeout 100`이벤트가 일어난 프레임: `_promise1`안에 있는 `setTimeout`코드가 실행이 됩니다. `console.log(ok1)`을 실행합니다. `resolve(true)`을 실행합니다. `resolve(true)`에 의해 `_promise1`뒤에 있는 `then`코드가 실행됩니다.정확히는 `promise`에의해서 `then`뒤의 함수가 인자로 들어가고 그걸 `resolve`라는 이름으로 쓰는거죠. 이제 `param`을 비교합니다. `param`이 `true`라면 `_promise2`을 리턴하는 군요. `_promise2`을 실행합니다. 다시 `setTimeout 100`이 있군요. 이벤트 스택에 넣읍시다. 여기까지가 한 프레임 입니다.
 - 그 다음 `setTimeout 100`이벤트가 일어나기 전까지의 프레임: 아무것도 안하고 이벤트를 기다립니다.
 - 그 다음`setTimeout 100`이벤트가 일어난 프레임: `_promise2` 안에 있는 `setTimeout 100`안에 있는 `console.log(ok2)`을 실행합니다. 그 다음에 `resolve`을 실행하는데 여기서 resolve는 `function() {console.log('ok3')}` 가 되겠지요. `console.log('ok3')`이 실행됩니다. event stack을 봅니다. 비었네요. 프로그램을 종료합니다.
 
 만약 `param`이 `false`라면 곧바로 2번째 `then`이 실행되겠죠.
 
 근데 여기서 의문점이 1개 생길수 있습니다.
 

> 그건 `true`는 `Promise`객체가 아닌데 어떻게 `then`이 실행되는거지? 

정답은 
>If a lambda passed to then returns a Promise, then will return an equivalent Promise. Generally, then will return Promise.resolve(<value returned by whichever handler was called>) - [https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/then#Chaining](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/then#Chaining)

`then`은 기본적으로 `return`을 `Promise`로 변환을 해줍니다. 만약 `return`이 없다면 `Promsie.resolve(null)`이 되겠지요.

{% gist g6ling/96980fb868f4aacee554d9eb7a642532 %}

이 코드가 몇번의 프레임에 실행이 될지 생각을 해보고, 왜 결과가 이렇게 됬을지 고민해 보세요. 여기까지 이해하셧다면 `Promise`의 실행순서를 완벽히 이해하셨을 겁니다.