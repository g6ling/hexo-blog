title: AWS dynamoDB 와 google datastore 비교
author: Cheol Kang
tags:
  - aws
  - google-cloud
categories:
  - cloud
date: 2016-09-17 18:25:00
---
# dynamoDB 와 datastore
아마 현재 가장 유명한 cloud NoSQL 서비스 들입니다. 이 2개에 대해서 잘 모르시거나, NoSQL이 무엇인지 잘 모르시다면 일단 검색하신후 읽으시길 권합니다.

# 같은거 아니에요?
그냥 NoSQL 이다. cloud다 이것만 같고, 나머지는 다 다른것 같습니다. 일단 요금체계가 다르고, 설계 방법이 달라지게 됩니다.(제 경험상으로는)

# 근데 일반적인 관계형 DB을 안쓰고 이걸 왜 써요?
이건 다른 많은 글을 보셔도 아시겠지만, 엄청나게 귀찮은 작업이 줄어듭니다. (단 그만큼 초반에 설계를 할때 귀찮습니다.)
그냥 DB를 정말로 데이터를 넣고 빼는 용도로만 생각을 하고, 제가 딱히 DB 설정들을 하나도 안해도 됩니다.(한번 만들어 놓으면 DB가 문제 생길일이 거의 없다고 보시면 됩니다.)
예를들어 scale의 문제, 백업의 문제, 레플리카의 문제등, 아무것도 걱정할 필요 없이 그냥 데이터를 넣었다가 뺏다가 하면 됩니다. 또한 속도를 언제나 보장해 주기 때문에
DB성능 에 대해서도 무시하셔도 됩니다.

# 그러면 무조건 관계형 DB보다 좋은건가요?
그러면 다 NoSQL을 쓸텐데 그건 아니죠. 일단 가장 큰 문제로 관계형 DB에서 당연히 되야지 라고 생각되는게 안됩니다. 항목들의 join, 트랜젝션, 단순한 정렬까지 좀 안되는게 많습니다.
그리고 dynamoDB의 경우에는 일정처리량을 넘어가면 요청자체를 씹는 경우도 생기기 때문에 안정성이 엄청 떨어집니다. NoSQL을 주력으로 쓰는 회사들도 결제정보등 안정성이 최우선인 작업을 관계형 DB을 쓴다고 합니다. 일관된 처리속도 + 자유자재의 scale 을 위해서 희생되는 부분이 매우 큽니다. 

# DynamoDB 와 datastore의 가격
일단 둘의 요금체계가 매우 다릅니다. DynamoDB의 경우에는 provision이라는 단위가 있어서 1초에 몇번의 작업을 할수있는지가 요금 단위입니다. 
이걸 1시간 단위로 요금을 받기 때문에 일반적인 클라우드 호스팅의 가격 개념입니다. 이 provision은 늘리고 줄이는게 가능하기 때문에
데이터가 몰릴경우에는 provision을 올리고, 트래픽이 줄어들면 provision을 내리는 작업등을 통해서 필요한 만큼만 사용하여 가격을 줄일수 있습니다.

그에비해 datastore의 경우에는 매우 심플합니다. 10만 요청 읽기당 6센트, 10만요청 쓰기당 18센트. 끝

단순계산으로는 dynamoDB의 경우에는 하루에 0.5 달러도 안되는 가격에 100만요청, 읽기쓰기가 가능하고, datastore은 2달러가 넘어갑니다.
최적화를 완벽하게 할경우네는 사실 dynamoDB가 더 쌉니다만, 그게 힘들경우에는 사실 datastore가 훨씬 싼것 같습니다.
dynamoDB을 사용할 경우엔 그 provision을 넘어가는 처리가 생길경우에는 요청을 무시하기도 하기 때문에 언제나 그거보다 여유있게 만들어야 하고,
최적화를 할려면 결국 다른 워커서버를 통해서 요청을 넘겨야 하는등 생각보다 귀찮은 작업이 많이생기고 여기서 생기는 비용문제도 있습니다.

# DynamoDB 와 datastore의 성능
사실 둘다 성능차이는 없다고 보시면 됩니다. 일관된 속도를 둘다 보장해 주기 때문에 유의미한 속도차이가 있다고는 보기 어려울듯 합니다. 
다만 DynamoDB의 경우에는 이러한 속도를 보장해 주기 위해서 여러가지 제한이 있습니다.([http://docs.aws.amazon.com/ko_kr/amazondynamodb/latest/developerguide/GuidelinesForTables.html](http://docs.aws.amazon.com/ko_kr/amazondynamodb/latest/developerguide/GuidelinesForTables.html) 이글을 참조.)
결국 이렇게 만들게 아니면 속도를 보장 못함이기 때문에 DB를 설계할때 생각보다 애로사항이 꽃핍니다.

간단한 예로 여러주제가 있는 게시판은 만들기 엄청 힘듭니다. 간단히 생각하면 hash key을 주제, range key로 생성날짜로 하면 될 거 같은데 이러면 또 1개의 hashkey에 너무 많은 작업이 몰려서 안됩니다. 그렇다고 생성날짜로만 해서 글을 쓰고 나중에 필터로 주제를 거른다고 하면 인기없는 게시판 글 볼려면 트래픽이 많이 생기고, 여러 문제가 있습니다.

그에 반해 datastore은 제약사항이 있긴 있는데 훨씬 상황이 좋습니다. 
[https://cloud.google.com/datastore/docs/concepts/queries?hl=ko#restrictions_on_queries](https://cloud.google.com/datastore/docs/concepts/queries?hl=ko#restrictions_on_queries) 성능적인게 아니라 그냥 안되거든요. 무조건 이렇게 해야만 실행이 됩니다. 

# 기능차이
기능 자체는 datastore가 훨씬 좋습니다. 트랜젝션도 지원하고(dynamoDB도 지원합니다만 꽤 제한적이에요), index도 생각보다 한계가 많이 없고(dynamoDB는 index 갯수까지 한계가 있어서 힘듭니다.) dynamoDB에서 되는건 datastore에서도 다 된다고 보시면 됩니다.

중요한건 dynamoDB는 그나마 자료가 있는데 datastore는 그냥 자료가 없습니다. 기능이 있는데 쓰는 방법을 몰라요. 

# 가장 큰 차이
dynamoDB는 AWS , datastore는 google cloud Platform 입니다. 구글 App Engine쓰면서 dynamoDB쓸수도 있고, datastore 쓰면서 amazon ec2 쓸수도 있습니다. 하지만 역시 특별한 경우가 아니라면 각각의 플랫폼에 알맞게 쓰는 경우가 대부분 일 것이라고 생각됩니다. 

# 결론
진짜 DB에 대해서 별 고민 없이 쓰고 싶다 -> google datastore
정말로 많은 데이터를 싸게 저장하고 사용하고 싶다 -> dynamoDB 일까요?
