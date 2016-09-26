title: 'MongooseでやるGeospatial, GeoJson処理'
author: Cheol Kang
tags:
  - mongoDB
  - japan
categories:
  - tech
date: 2016-09-17 18:10:00
---
ちなみにgeoJsonが何かを言います。jsonの形である位置情報ですが[geoJSON ホムページ](http://geojson.org/)やグーグルすると出ます。
```
{
  "type": "Feature",
  "geometry": {
    "type": "Point",
    "coordinates": [125.6, 10.1]
  },
  "properties": {
    "name": "Dinagat Islands"
  }
}
```
だいたいこんな形をセーブされるjsonです。もっと多い例を見たいとすると[geojson.io](http://geojson.io/#map=19/34.79164/135.44078)で入れて色々やるとすぐ分かれると思います。
この情報をmongooseでセーブしましょ。
```
var mongoose = require('mongoose');

var chatRoomSchema = mongoose.Schema({
    created: Date,
    name: String,
    roomId: String,
    chats: Array,
    location : {
        type : {
            type: String,
            default: 'Point'
        },
        coordinates: [Number]
    }
});

chatRoomSchema.index({ location : '2dsphere'});

module.exports = chatRoomSchema;
```
これは僕が作ったサンプルmongoose　Schemaです。見ると作った日、名前、id, chats, locationの情報がありますがここでlocationはobjectだし、これはまたtypeとcoordinatesがあります。そしてindexはlocationを基づいて2dsphere(球面を２dで表現）するとを表す。これを実際にどう使うか見ます。
![]({{ site.baseurl }}/images/Screen-Shot-2016-02-29-at---3-57-43.png)
chatroomsでget要請がくるとパラメーターをlongitude(緯度),latitude(経度)で受けます。そのあとそれらをcoordsっていう配列を作ります。そのあとfindをするけど

```
location: {
                $near : {
                    $geometry : {
                        type: "Point",
                        coordinates : coords
                    },
                    $maxDistance : 1000
                }
            }
```
この条件でfindをします。もっと多い条件についてわかりたっかたら
[MongoDB Near Document](https://docs.mongodb.org/manual/reference/operator/query/near/)を見るとわかります。 $near operatorは 2dsphere indexを必要するのでSchemaを作る時設定しなければならないです。上の条件はcoordsから1000m以内の点だけ検索することです。 $minDistanceを使うと一定距離以上の点だけも検索ができます。
このポストは説明が足りないのでしたのリンクたちを読むことを欲しいです。（短いので読んだ方がいいです）

* [MongoDB GeoJSON Object](https://docs.mongodb.org/manual/reference/geojson/)<br>
* [MongoDB Near Document](https://docs.mongodb.org/manual/reference/operator/query/near/)<br>
* [geoJSON ホムページ](http://geojson.org/)<br>
* [参考したブログのポスト](http://blog.robertonodi.me/how-to-use-geospatial-indexing-in-mongodb-using-express-and-mongoose/)