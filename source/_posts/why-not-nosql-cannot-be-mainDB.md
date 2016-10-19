title: noSql의 mainDB로써의 한계
author: Cheol Kang
tags:
  - tech
  - korean
  - architecture
categories:
  - tech
date: 2016-10-20 04:50:00
---


이번에 프로젝트를 진행하면서 처음에는 DynamoDB, Google cloud Datastore 을 메인 DB로 사용하려고 하였으나, 한계를 느끼고 MariaDB로 돌아갔다.

### 턱없이 부족한 기능들

nosql 은 단순히 key-value 스토어를 넘어선 좀더 많은 기능을 가지고 있지만 RDB의 수많은 기능들을 사용할수 없다는 게 단점이다. transaction, join 등 약간 조금이라도 DB적인 고급 기능들은 기본적으로 제공하지 않고, 억지로 만들수 있지만 결국 여러가지 문제가 발생하게 된다.

### DynamoDB의 문제

DynamoDB는 range-key, sort-key 방식으로 key-value 스토어를 만든다. range-key로 hash-key을 만들고 그 데이터들 안에서 sort-key을 사용하는건데, 문제점은 range-key는 한번에 1개의 프로비전밖에 사용할수 없다는 것이다. DynamoDB는 프로비전이라는 단위로 과금을 하는데 한번에 몇개의 요청까지 처리할수 있냐가 기준이다. 예를 들어 프로비전 5단위을 사용한다면 1번에 5개의 요청까지 처리를 할 수가 있고, 더 많은 요청이 가게 된다면 큐에 담는게 아니라 요청을 무시한다. 근데 1번에 1개의 range-key만 처리 할수 없다. 예를들어 post라는 range-key에 아무리 많은 요청을 하고, 프로비전 처리량이 높아도, 결국 1개의 range-key에서 처리할수 있는 처리량은 한정되어 있다. 그래서 DynamoDB의 설명서에서도 최대한 중복된 range-key을 가지지 않도록, 최대한 primary 하게 만드라고 설명이 되어있고, 혹시라도 처리량이 1개의 range-key에 몰린다면, 이런저런 방법을 통해서 range-key을 분산시키라고 한다. 

또한 복잡한 index을 만들수가 없다. 만약 RDB로 어느 유저가 어떠한 게시판에 쓴 글을 최근순서대로 보여주고 싶다면 

    SELECT * FROM POSTS WHERE UserId='abc' AND Board='food' ORDER By Created DESC

이렇게 간단하게 할 수 있는걸, DynamoDB에서는 못한다 . 물론 range-key을 userId로 하고, sort-key을 created로 만들고 filter을 잘 준다면 어찌어찌 가능하다. 그런데 여기서 조금이라도 더 복잡해 진다면 DynamoDB로 만들 수 없다. 

### Google Cloud Datastore 라는 대안

하지만 Google Cloud Datastore는 complex Index 라는 것을 지원한다. 만약 복잡한 query을 사용하고 싶다면, complex Index을 생성하고 query을 사용하면 된다. transaction도 물론 지원한다. Google Cloud Datastore는 Nosql 중에서 가장 RDB에 가까운 기능을 수행 할 수 있도록 해준다.

### Google Cloud Datastore 단점

이건 비단 Datastore 뿐만 아니라 DynamoDB에도 해당되는 말인데, RDB의 기능을 따라하기 위해서는 보다 많은 비용과 시간이 들어간다. RDB에서 당연히 되는걸 Nosql에서는 자기가 직접 하나한 구현을 해야 한다. 또한 query마다 따로 index을 1개씩 만들어서 사용해야 하기 때문에 간단히 1줄로 끝날 일을 몇시간 단위로 해야하니 시간은 오래 걸린다. 

그렇다면 비용은 왜 더 비쌀까? 일반적으로 Cloud Platform 에서 제공하는 Nosql은 RDB에 비해서 훨씬 합리적으로 비용을 부과하고(예를 들어 1GB만 저장햇다면 그만큼만 비용을 낸다) , 훨씬 더 싸다. 하지만 실제로 RDB같은 기능을 원한다면 비용은 RDB보다 많이 필요 할 수도 있다. query마다 index을 만들어야 하기 때문에 Index의 용량이 무지막지 하게 커지게 된다. 거기에 Complex Index는 훨씬더 많은 용량을 사용하기 때문에 실제로 저장되는 데이터의 몇십배가 되는 용량이 Index로 사용되는 경우가 발생하게 된다. 

또한 RDB에서는 한번에 많은 조건을 통해서 비교가 가능하고, Join또한 가능하지만 Nosql에서는 구현을 한다고 해도, 결국 DB에서 하는게 아니라, Serer에서 해야 하기 때문에 DB와 Server가 더욱많은 통신을 해야하고, 또한 Server의 리소스를 많이 쓰기 때문에 Nosql을 싸다. 라고 말하기는 힘들다. 

### 그렇다면 Nosql은 사용하지 않아야 하나?

아마 이러한 특성들 때문에 대부분의 기업들 에서는 Nosql을 Log저장소 정도로만 생각하는듯 하다. 하지만 그러기에는 Nosql이 아깝다.

1. Nosql은  RDB보다 빠르고 RDB보다 싸며, RDB보다 자유롭다.   
2. Nosql 은 RDB을 대신할 수 없다. 

이 2가지를 기억하고 자신이 지금 어떤 DB가 필요한지를 생각하면, 아무 무슨 DB가 자신에게 더 효율적인지 알게 될 것이다.

micro architecture와 nosql, rdb을 조화롭게 잘 사용한다면, 보다 효율적으로 리소스를 사용하고, 장애발생에도 더 좋은 대처를 할 수 있을 것 이라고 생각된다. 
