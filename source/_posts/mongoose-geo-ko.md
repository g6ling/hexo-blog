title: 'Mongoose로 Geospatial, GeoJson처리 (가까운 점 찾기)'
author: Cheol Kang
tags:
  - mongoDB
  - korean
categories:
  - tech
date: 2016-09-17 18:09:00
---
일단 geoJson이 무엇인지 알아야 한다. json형태로 만든 위치 정보인데 [geoJSON 공식 홈페이지](http://geojson.org/)나 google검색을 하면 바로 나온다. 
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
대략 이런형태로 저장되는 json 이다. 더욱 많은 예를 보고 싶다면 [geojson.io](http://geojson.io/#map=19/34.79164/135.44078)을 들어가서 이것저것 해보면 알 수 있을 것이다. 
그러면 이 정보를 일단 mongoose에 저장을 해보자. 
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
이건 내가 만든 샘플 mongoose Schema이다. 대충보면 만든날짜, 이름 ID, chats, location의 정보가 있는데 여기서 location은 object이고 이건 다시 type과, coordinates가 있다. 그리고 index는 location 기준으로 2dsphere(구면을 2d로 표현)한다는 것을 명시한다. 이제 이것을 어떻게 실제로 이용하는지 보도록 한다.
![]({{ site.baseurl }}/images/Screen-Shot-2016-02-29-at---3-57-43.png)
일단 chatrooms로 get요청이 온다면 parameter을 longitude(경도), latitude(위도)로 받는다. 그다음 longitude와 latitude가 있다면 그것으로 coords라는 배열을 만든다. 그뒤 find을 하는데 
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
이 조건으로 find을 한다. 더욱 많은 조건을 넣고 싶다면 [MongoDB Near Document](https://docs.mongodb.org/manual/reference/operator/query/near/)을 참조하면 될 것 이다. $near operator는 2dsphere index을 필요로 하기 때문에 꼭 앞에서 지정을 해 주어야 한다. 위의 조건은 지정한 coords로 부터 1000m 이내의 점 만을 검색한다는 것이다. $minDistance를 이용하면 일정거리 이상의 점만을 검색 할 수 있다. 
블로그의 글은 설명이 부족하니 위의 링크들을 들어가서 읽기 바란다.(매우짧다)

* [MongoDB GeoJSON Object](https://docs.mongodb.org/manual/reference/geojson/)<br>
* [MongoDB Near Document](https://docs.mongodb.org/manual/reference/operator/query/near/)<br>
* [geoJSON 공식 홈페이지](http://geojson.org/)<br>
* [참고 블로그 글](http://blog.robertonodi.me/how-to-use-geospatial-indexing-in-mongodb-using-express-and-mongoose/)