title: Mysql 을 NodeJS에서 사용할때 글자 깨짐
author: Cheol Kang
tags:
  - nodejs
  - db
categories:
  - node
date: 2016-12-29 17:43:00
---
nodeJS에서 Mysql 을 사용하는데, 분명 서버를 통해서 저장을 하고 읽을 때는 글자가 잘 출력이 되는데, 이상하게 DB 에서 직접 보는 경우에는 글자가 깨져서 저장되는 경우가 생겼다. 분명 DB의 모든설정을 utf8로 바꾸었음에도 불구하고, 계속 글자가 깨졋는데, 이유는 단순하게 node 와 mysql 의 연결 설정이 utf8이 아니었기 때문이다. 연결설정 문제였기 때문에 DB 상에 이상하게 저장이 되도, 서버가 읽고 쓰기를 할 경우에는 정상작동을 했었다. 해결방법은 단순하게

```javascript
const mariaDB = new Client({
    host: process.env.RDS_HOST,
    user: process.env.RDS_USER,
    password: process.env.RDS_PASSWORD,
    db: process.env.RDS_DB,
    charset  : 'utf8' // 이 부분을 추가하면 된다.
});
```