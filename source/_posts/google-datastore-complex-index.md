title: google datastore에서 compositeIndex(복합색인) 만들기
author: Cheol Kang
tags:
  - google-cloud
  - korean
categories:
  - cloud
date: 2016-09-17 19:05:00
---

일단 기본적으로 google datastore은 자동으로 index(색인)을 만듭니다. 하지만 이 색인은 단순히 1개의 속성에 대해서만 index의 기능을 발휘하기 때문에

만약 2개의 속성에 대해서 색인을 사용할 경우 예를들어) 작성자 = 나, 순서 = 만들어진 순서대로, 로 할 경우 2개의 속성에 대해서 indexing을 하기 때문에

일반적인 index로는 에러가 생깁니다. 저 같은 경우는 node로 사용하고 있었는데, 단순히 preCondition Error 라고 뜨면서 어떤에러인지 정확하게 보여주지 않아서

복합색인의 문제라고 인식하는데도 시간이 걸렸습니다. ㅠㅠ.(결국 브라우저에서 쿼리를 했는데 복합색인이 없어서 안됩니다. 라고 떠서 알게되었습니다)

근데 또 예제를 보면 복합색인이 필요한 예제가 복합색인 없이 잘 실행되기 때문에 필요없다고 생각했는데 그건 또 아니더군요.

[google datastore 예제](https://cloud.google.com/datastore/docs/concepts/queries?hl=ko) 모든 문제의 근원.

딱히 composite index가 필요합니다 라는 글이 어디에도 없습니다. 그냥 예제만 달랑.

그래서 또 만들려고 하면 index.yaml이 필요하다고 뜨는데 이건 또 어디에 있나 이걸 수정해야 하나라고 찾았는데 어디에도 없습니다.

그냥 index.yaml을 만들고 그걸 업로드 하라는데 datastore에 어떻게 업로드 하는지 모르겠는데 

[https://cloud.google.com/sdk/gcloud/reference/preview/datastore/create-indexes?hl=ko](https://cloud.google.com/sdk/gcloud/reference/preview/datastore/create-indexes?hl=ko) 여기에 있습니다.

그냥 이거 보고 따라하면 됩니다.


만약 kind가 post에 author 속성과, created 속성을 가지고 있다면

일단
- kind: post
  properties:
  - name: author
    direction: desc # 이건 오림차순이든 내림차순이든 상관없습니다. 어차피 정렬이 아니라 같은것만 찾아내는 것 이니까요
  - name: created
    direction: desc # 이건 내림차순을 하면 최신순대로, 오름차순으로 하면 옛날순서대로 나오는데, 그 순서를 못바꿉니다.(일단 datastore 내에서는 서버측에서 그걸 역으로 한다면 문제 없습니다만, 귀찮고, 트래픽이 훨씬 많이 생기니)

대략 이렇게 하면 됩니다. 

아마 훨씬 복잡한 index도 가능할 것 같은데, index크기가 너무 커서 문제입니다. 데이터크기 << 인덱스 크기가 됩니다. 정말 꼭 사용해야할 index만 사용하고 나머지는 index을 절대로 안하는 형태로 해야할것 같습니다.
