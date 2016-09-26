title: google datastoreで compositeIndexを 作る
author: Cheol Kang
tags:
  - google-cloud
  - japan
  - ''
categories:
  - cloud
date: 2016-09-17 19:08:00
---
## 僕が作った時思ったより難しくて片付けします。

一応基本的にgoogle datastoreは自動でindexを作ります。でもこのindexはただ１個のpropertyだけに対してindexしますので

もし２つのpropertyに対してindexしたいとき、例えば) author=私、sort=最近　とかができないです。

僕の場合にはnodeJSでgoogle datastoreを使ったですが、ただ preCondition Errorとでて、このエラーが indexの問題かはわからなかったです。

（結局ウェブで同じqueryをして、このとき、　composite indexがありません　とでてindexの問題だとわかりました。）	

また例題をみるとcomposite indexが必要な例題が普通にあって別にこの時はいらんかと思ったけど例題もindexを設定しないとできないです。

[google datastore 例題](https://cloud.google.com/datastore/docs/concepts/queries?hl=ko) 全ての問題がここから.

別にcomposite indexが必要です。はどこにもありません。

composite indexを作ろうとすると index.yaml　が必要ですと書いてるけど、　これはどこにあるか　探してもわからないし、

index.yamlを datastoreに　アップロードしましょう。と書いてるけどどうやってアップロードするかもわからない。		

[https://cloud.google.com/sdk/gcloud/reference/preview/datastore/create-indexes](https://cloud.google.com/sdk/gcloud/reference/preview/datastore/create-indexes?hl=ko) 結局さがしました


もし　さっきの　author=私、sort=最近　これをやりたい時は

일단
- kind: post
  properties:
  - name: author
    direction: desc 
  - name: created
    direction: desc 

こうやるとできます。

多分もっと複雑なindexも十分可能と思いますが、indexの大きさがちょっと大きすぎると思います。僕の場合にはindexの大きさ　＞＞　dataの大きさ　になりました。
もし実際これを使う時は基本的につくられるindexは全部けして、自分が使うものだけつくってつかう方がいいとおもいます。	
