title: Node.js 로 JWT(JsonWebToken) 방식 RESTful api login 구현
author: Cheol Kang
tags:
  - nodejs
  - korean
categories:
  - nodejs
date: 2016-09-17 18:11:00
---
##### 배경?
일단 JWT가 무엇인지 부터 알아보자. JWT는 JsonWebToken 의 줄임말로 그전에는 사용자의 정보를 사용자(client)측 에서 저장하는 경우가 많았다. 이 경우에는 사용자가 쿠키 변조 등으로 쉽게 정보를 바꿀 수 있기 때문에 보안에 문제가 많았다. 또한 최근의 흐름인 RESTful api 와는 맞지 않는 경향이 많았다. 전통적인 방법으로는 mobile 등의 여러가지 플랫폼에 대한 접속이 지원하기 힘들었고 이러한 문제점등을 해결하기 위해서 나온 방식이다. 최근 많은 회사들이 OAuth 을 지원하는데 대부분 JWT를 사용한 방식이다.(물론 더 많이 복잡한 보안 기술이 들어가겠지만 기본 개념은 같다). 

##### JWT란
단순하게 생각해서 사용자(client)에게 사용자의 정보를 주지 않고 단순히 token 이라는 정보만을 사용자 에게 준다. 그리고 사용자가 권한을 사용하고 싶을때는 자신의 token 만을 서버에게 넘겨주고, 서버측에서 자신이 DB에 저장한 token과 받은 tokend을 비교한 뒤에 사용자 정보를 보고 그에 알맞은 권한을 가지고 있는지 판단하는 것이다.

간단한 예를 들어서 이야기를 해보자. 어느 회원제 호텔이 있다고 해보자. 이 호텔은 식당도 있고 수영장도 있고 헬스장도 있다. 하지만 스위트룸이상 묵고 있는 사람들만이 스카이 바를 이용할 수 있다고 해보자. 그러면 처음에 호텔에 들어갈때 자신의 정보를 말하고 라운지에서 이름과 자신이 묵고있는 룸번호, 전화번호 등 여러 정보가 써져있는 종이를 한장 받는다. 그리고 수영장에서 이 종이를 보여주면 들어갈수 있게 해준다. 그런데 만약 일반 객실에 묵고 있는 사람이 스카이바를 사용하고 싶어진다. 그래서 자신이 가지고 있는 종이에 룸번호를 지우고 스위트룸번호를 적는다. 그 뒤 스카이바를 가고 스카이바에서는 아무 것도 모르는 상태에서 여러분을 입장시킬 것이다. 또한 누군가가 그 종이를 가로챈다면 수많은 개인정보가 빠져 나갈 것이다. 이게 전통적인 방식이다.

그러면 토큰방식을 보자. 호텔에 들어갈때 한번만 자신의 정보를 말하면 라운지에서 번호가 써져있는 동전(token)을 1개 준다. 그 뒤에 자신이 어디를 가던지 그 토큰만 보여주면 되는 것이다. 직원은 그 동전을 번호를 검색하고 그 사람이 누구인지 알게 된다. 자신이 동전을 변형시켜서 스카이바를 가려고 해도 동전의 번호는 암호화되어있기 때문에 어떠한 번호가 스위트룸의 번호인지 알수가 없기 때문에 변조가 불가능하고 누군가가 동전을 가로채도 이사람이 누구인지 어떠한 정보도 얻을 수가 없다. 

![](https://cloud.githubusercontent.com/assets/8134523/18607411/6985c706-7d06-11e6-817c-771883ea7e52.png)

대략 이런식의 구조이다. 

1. 로그인 정보를 Authorization Server에 넘긴다. 
2. Authorization Server는 DB와 값을 비교
3. DB는 token을 반환해준다.
4. client는 token을 받는다
5. client는 token과 자신이 하고싶은 행동(api call)을 보낸다
6. api server는 token을 DB에서 비교
7. user의 정보를 반환받는다
8. user의 정보가 api call의 권한을 가지고 있다면 실행을 해주고 결과를 반환해준다.

이러한 구조를 가지고 있다. 

##### 실제 구현
그러면 이걸 node와 angular로 구현해보자. 

일단 [Token-Based Authentication With AngularJS & NodeJS
](http://code.tutsplus.com/tutorials/token-based-authentication-with-angularjs-nodejs--cms-22543)을 보면서 해보았다. 안에 깃헙도 있으니 하면 그대로 clone해서 보면 되지만 그러면 의미가 없으니 살짝 바꿔서 express 제너레이터로 만든 서버에서 만들기로 하자(어차피 똑같다.)

express generator가 없으면 설치하자. 터미널에서  `npm install -g express generator` 그 뒤에 `express token-based-auth` 을 하면 express 의 스켈레톤이 만들어 질것이다.

app.js에 mongoose을 사용위해
```js
var mongoose = require('mongoose');
mongoose.connect('mongodb:localhost/token-based-auth');
```
을 추가. mongoose가 무엇인지 잘 모르는 사람은 검색해주길 바란다. mongoDB을 좀 더 쓰기 쉽게 만드는 모듈이라고 생각하면 편하다.

app.js에서 `app.use('/',routes);`위의 부분에
```js
app.use(function(req, res, next){
	res.setHeader('Access-Control-Allow-Origin', '*');
    res.setHeader('Access-Control-Allow-Methods', 'GET, POST');
	res.setHeader('Access-Control-Allow-Headers', 'X-Requested-With,content-type, Authorization');
```
을 추가. 이건 CORS을 회피하기 위해서 추가한다. cors란 쉽게 이야기 하면 같은 도메인을 가지고 있는 서버에서의 접속을 인정해준다는 것이다. aaa.com 이라는 도메인을 가지고 있는 서버가 bbb.com이라는 서버로 ajax요청을 할때 bbb.com 의 서버는 aaa.com이 ajax하는 것을 허용해줄것인가 안해줄 것인가를 설정할수 있는 것이다. 우리는 지금 node 서버를 만들고, 이건 api 서버 이기 때문에 다시 angular client 서버를 만들고 사용할 것이다.(물론 현재 하는 express 서버에 angular을 넣어도 되지만 backend와 frontend을 분리함으로써 얻는 이득이 크다) 위에서 부터 읽으면 
`Access-Control-Origin, * `의 부분은 모든 ip주소에서부터 오는 요청을 전부다 허락해 주겠다는 것이다. 만약 특정한 주소만을 허락해 주고 싶다면 이부분을 *가 아니라 특정주소를 넣으면 된다.
그리고 `access-control-method, get,post`는 get요청과 post요청을 허락해 준다는 것이다. put과 delete도 허락해 주고 싶다면 추가하면 된다. `header`는 어떤 헤더속성을 허락해주겠냐는 것이다. 뒤의 angular 부분을 보면 좀 더 이해하길 편할 것이다.

model이라는 폴더를 만들고 그안에 user.js 라는 파일을 만들자. 
```js
var mongoose = require('mongoose');
var Schema = mongoose.Schema;

var UserSchema = new Schema({
	email: String,
    password: String,
    token: String,
 });
 
 module.exports = mongoose.model('user',UserSchema);
```
유저 스키마를 만든다. 간단하게 email, password, token만을 저장하겠다.
그 다음 `route/user.js` 로 가자
```js
var express = require('express');
var router = express.Router();

var User = require('./../model/user.js');
var jwt = require('jsonwebtoken');
var jwt_secret = 'secret';

router.get('/', function(req, res, next){
	res.send('respond with a resource');
});

router.post('/authenticate', function(req, res){
	User.findOne({email: req.body.email, password: req.body.password}, function(err, user){
		if(err) {
			res.json({
				type: false,
				data: 'Error occured' + err
			});
		} else {
			if (user) {
				res.json({
					type: true,
					data: user,
					token: user.token,
				});
			} else {
				res.json({
					type: false,
					data: 'Incorrect email/password'
				})
			}
		}
	});
});
```
이렇게 추가. jwt는 `npm install jsonwebtoken`을 하자.

`authenticate의 post요청`(로그인요청)을 살펴보면 단순하게 email과 password을 비교 DB에 같은것이 있다면 반환, 없다면 error을 반환한다. 이때 token에 user.token을 반환하는부분을 잘 살피자.

그 다음은 
![]({{ site.baseurl }}/images/-----2016-02-17----2-09-37.png)
signin(회원가입)부분을 보면
들어온 정보를 email 부분을 보고 user을 찾아서 같은 ID을 이미 가지고 있다면 이미가지고 있다는 에러는 반환, 없다면 새롭게 user을 만든다. 그리고 user을 만들고 저장한뒤 callback으로 이 유저의 정보에 대해서 userd의 토큰을 생성한다. 이때 위에서 지정한 jet_secretd을 사용한다. 그 다음 저장. 유저의 token을 반환해준다.

```js
router.get('/me', ensureAuthorized, function(req, res){
	User.findOne({token: req.token}, function(err, user){
		if(err) {
			res.json({
				type: false,
				data: 'Error occured: ' + err,
			});
		} else {
			console.log('me: ' + user);
			res.json({
				type: true,
				data: suer,
			});
		}
	});
});

function ensureAuthorized(req, res, next) {
	var bearerToken;
	var bearerHeader = req.headers["authorization"];
	if (typeof bearerHeader !== 'undefined') {
		var bearer = bearerHeader.split(' ');
		bearerToken = bearer[1];
		req.token = bearerToken;
		next();
	} else {
		res.send(403);
	}
}
module.exports = router;
```
이제 get요청을 처리해보자. 우리는 이 요청을 토대로 api call을 구현할수 있다. 자신이 어떠한 api을 만들어도 이걸 바탕으로 잘 바꾸면 된다. 요청이 들어오면 일단 ensureAuthorized을 middleware 로써 사용한다. 즉 요청이 들어오면 ensureAuthorized가 실행된다. ensureAuthorized는 next라는 함수를 인자로 받고 그 next라는 함수가 me 에서 뒷부분에 적어진 callback 함수이다. 잘 모르겟으면 node.js의 콜백에 대해 검색해 보자. 

이 함수에 대해 알아보자. 이 함수에서 하는 일은 request에서 authorization이라는 헤더를 찾아서 `.`을 기준으로 그 헤더를 나눈뒤 2번째에 있는 값을 request.token이라는 값에 넣는것이다. 그래서 그뒤의 next callback함수에서는 req.token을 이용하여서 user을 찾을 수 있었다. 


이걸 알기 위해서 좀더 jwt의 구조에 대해 알아보자.
내가 링크한 블로그에 있는 글을 번역한 것이다.<br>
###### 번역글
JWT는 JSON Web Token을 가르키고, 그것은 authorization 헤더안에서 사용되는 토큰포맷이다. 이 토큰은 너가 2 시스템을 안전하게 커뮤니케이션을 디자인할때 도움을 준다. 이 튜토리얼의 목적을 위해 jwt을 "bearer token"이라고 고쳐부르자. bearer token은 3부분: header, payload, signature로 구성되어 잇다.

헤더는 토큰타입과 암호화방법을 저장하는 토큰의 한 부분이고 이것 또한 base-64로 암호화 되어있다.

payload는 정보를 포함하고 있다. 어떤 종류의 데이터라도(예를 들어 유저 정보, 상품정보 등) base-64암호화된 형태로 payload 안에 넣을 수 있다.

signature(서명) 은 header, payload, 그리고 secret key의 조합으로 이루어져 있다. secret key 는 반드시 server-side에 안전하게 저장 되어야 한다.

![](https://cms-assets.tutsplus.com/uploads/users/487/posts/22543/image/jwt-schema-png.png)

bearer token generator을 만들필요 없이 이미 여러 언어에서 존재하는 것을 찾을수 잇을 것이다.

* NodeJS: [http://github.com/auth0/node-jsonwebtoken](http://github.com/auth0/node-jsonwebtoken)<br>
* php : [http://github.com/firebase/php-jwt](http://github.com/firebase/php-jwt)
* Java: [http://github.com/auth0/java-jwt](http://github.com/auth0/java-jwt)
* Ruby : [http://github.com/progrium/ruby-jwt](	http://github.com/progrium/ruby-jwt)
* .NET	: [http://github.com/AzureAD/azure-activedirectory-identitymodel-extensions-for-dotnet
](http://github.com/AzureAD/azure-activedirectory-identitymodel-extensions-for-dotnet
)
* Python : [http://github.com/progrium/pyjwt/
](http://github.com/progrium/pyjwt/)
###### 여기까지 번역글 입니다. 

대략 번역글을 읽었으면 저 함수가 무엇을 의미 하실지 알 것이라고 생각됩니다.
그뒤 callback함수는 token으로 DB에서 user을 찾고 user정보를 넘겨 줍니다. 만약 게시판에 글을 쓰는 내용이라면 req.body에 데이터가 있을거고 그 데이터와 token으로 찾은 user정보로 그 user가 req.body의 내용으로 글을 쓰는 것도 할 수 있겠죠.


front-end는 [https://github.com/cubuzoa/token-based-auth-frontend](https://github.com/cubuzoa/token-based-auth-frontend) 여기를 클론하시면 됩니다.

각 부분들이 잘 모듈화 되어있는데요, service.js에서 return과 baseUrl 부분을 제외하고는 지우셔도 상관없습니다. 아마 예전에 작성자가 만들어논 부분같네요. 그리고 혹시 redirect뒤에 localstorage가 날라가는 사람들이 있다면 redirect하는 부분을 setTimeout을 100정도 줘서 하면 됩니다. (저도 이러한 현상을 겪었습니다. 정확한 이유는 모르나 아마 localStorage에 저장하는데 시간이 어느정도 걸리는데 그전에 redirect해버려서 저장이 안되는것 같다고 저는 예상합니다만 잘은 모릅니다.) 이렇게 하시면 api call로 우리는 로그인을 구현할수있습니다. 이걸 모바일에 적용하기는 매우 쉽습니다. post로 요청. 토큰을 json으로 받음. 그걸 localStorage에 저장. 현재 로그인되었는지 안되었는지를 bool로 저장한뒤, token을 저장해 놓으면 로그인 되어있다면 token을 참조, api call을 사용. 세션등에 비해서 모바일에서 쉽게 할 수 있습니다. 
*현재 password의 암호화를 하지 않았습니다. 패스워드의 암호화는 bcrypt-nodejs등을 이용하여 할 수 있습니다. 나중에 이 부분을 추가 하도록 하겠습니다.*