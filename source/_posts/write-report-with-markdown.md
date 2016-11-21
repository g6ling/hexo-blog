title: markdown으로 레포트 쓰기
author: Cheol Kang
tags: []
categories:
  - tech
date: 2016-11-21 04:25:00
---
markdown으로 레포트를 쓰다가 겪은 경험

# 왜 markdown으로 썻나요?
그러게요. 원래 평소에는 word로 썻는데 word가 사진좀 들어가고, 장수가 한 20장만 넘어가도 맨날 버벅버벅 거리던 기억이 있길래
> markdown으로 쓰면 가볍겠지.

라는 생각으로 일단 했습니다. 사실 처음에는 markdown으로는 형태 좀 잡고, 글만 쓸려고 했는데 어느순간 표도 넣고, 이미지까지 넣다보니 word로 처음부터 하기 귀찮아서 이렇게 했는데 더 오래 걸렸습니다. 

# [Typora](http://www.typora.io)

제가 매우 좋아하는 markdown 에디터 입니다. 일반적인 markdown 에디터는 왼쪽에는 text, 오른쪽에는 preview 가 있는 형태인데, typora는 preview + text 느낌입니다. 일반적인 에디터 같은 모습이죠. 거기에 image drag&drop도 가능하고, Latex 도 지원, export 도 꽤나 많은 종류로 되서 전부터 애용하고 하고 있던 markdown editor 입니다. 물론 앞으로도 쭈욱 애용할겁니다. theme도 좋아요. 

## 문제점
### 이미지 크기조절
레포트를 쓰는데 이미지 크기를 조절 할 수가 없습니다.
### 가운데 정렬
이미지나, 글들의 가운데 정렬이 안됩니다.

해결을 할려고 해도 editor 단에서 안되서 여러가지 방법을 찾습니다.

### html 로 export, html을 수정하자
일단 시도는 했는데, 오늘 한번만 레포트 쓰고 말것도 아니고 한 몇분 하다가 이건 아니지 하면서 때려쳤습니다.


## markdown editor을 찾아서
markdown image resize 라는 키워드 구글링을 열심히 하다 보니, Mou 라는 에디터에서는 이미지 resize을 할수 있다고 합니다. 근데 Mou는 sieera에서는 실행이 안된다고 합니다.

또 찾습니다. haroopad 을 쓰면 된다고 합니다. 컴에 깔려있엇는데 미리 써볼걸 그랫어요....

실행을 합니다. haroopad 문서를 잘 찾아보니, `![](url "title" "style")` 이라는 것이 있군요. style에 html style 을 넣어보니 됩니다. 
> WOW

희망이 보입니다. style에 `height: 300px, float: left;'을 넣어 봅시다.

됩니다. 사진을 옆으로도 놓을수 있게 됫어요. 감동

이제 글을 가운데 정렬 시켜봅시다. 

안됩니다. 아무리 찾아도 지원하지를 않습니다.

## 새로운 방법
markdown preview란 무엇일까? markdown에 대해 알아 봅니다. 

기본적으로 preview는 markdown을 html로 변환시켜서 보여줍니다.

markdown에 html tag을 넣어봅니다.

html tag가 preview에서는 html tag로 인식해서 보여줍니다.

> 이거군

`<img src="" style="" />`
을 사용해서 이미지를 적당히 바꿔줍시다.
글 가운데 정렬은 `<p style=""></p>`을 써서 바꿉니다. 후. 거의 다 쓸만해진것 같습니다. 

이제 표만 가운데 정렬을 하면 됩니다. 단순하게 `div`로 감싸서 스타일을 줄려고 했는데, `div`로 감싸면 markdown이 그 부분을 변환을 안합니다.

이미 표는 markdown으로 만들어서 html로 바꾸기가 매우매우 귀찮습니다......

다시 고민....

`style`태그 쓰면 어떻게 처리를 하지? 문서 맨위에 써봅니다.

preview에 잘 적용이 됩니다. 근데 table이 항상 `width: 100%`로 설정이 되어있습니다. table마다 크기가 다른데 style안에서 구분을 줄 수가 없습니다. ( markdown이여서 id나 class 아무것도 지정이 안됨)

chrome 개발자 도구로 스타일을 바꾸어 봅니다.
```
table {
	width: 0 !important;
	margin: auto !important;
	overflow: visible !important;
}
```
이렇게 하니깐 됩니다. 가운데 정렬 이 됩니다. 휴우. `!important`는 css 우선순위 때문에 사용하였습니다.

결국 html + markdown 처럼 됬네요. 출력할려면 pdf 파일이 있어야 되니깐 일단 html 로 변환하고 브라우저로 html을 연다음에 출력하면 완벽.

haroopad 의 경우에는 html로 출력을 할 경우에 `footer`태그가 붙습니다. 영 보기가 안좋아서 개발자 도구로 지웁니다. 그다음 출력. 그냥 혹시 몰라서 haroopad에서 바로 pdf 로 출력을 해도 결과가 같습니다. 아마 pdf 출력도 `html로 변환 -> html을 pdf로 변환` 과정을 하는 것 같습니다.

맨위에
```
<style>
table {
	width: 0 !important;
	margin: auto !important;
	overflow: visible !important;
}
thead {
	background: lightgray !important;
}
img {
	display: block !important;
    margin: auto !important;
}
p.description {
	text-align: center !important;
}
</style>
```

을 넣읍시다. 이제 레포트 쓸때 맨위에 저것만 붙여넣고 markdown으로 열심히 쓰면 됩니다.


# 결론

귀찮게 word로 수식 안넣어도 됩니다. 필요한 부분만 Latex 감동.

귀찮게 간단한 workflow 안그려도 됩니다. `mermaid` 사용합시다. (물론 복잡하면 draw.io 같은거 사용하겟지만)

귀찮게 전체 스타일 한개한개 바꿀 필요가 없습니다. 저에겐 style태그가 잇으니까요.

의도치 않게 알게된 markdown preview 구현방법... typora에서는 html 태그가 안먹던데, 역시 preview 형식이 아니다 보니 html 을 쓰는게 아니라 다른 방법을 써서 구현해서 그런듯 하다. 

처음 글 쓰면서 구조 잡을때는 typora 쓰고, 나중에 이미지 넣고, 스타일 맞출때는 haroopad나 atom 쓰면 된다. (atom 좀 길어지니깐 엄청 느려진다. 부가기능은 많은데 너무 느려지는게 흠)

