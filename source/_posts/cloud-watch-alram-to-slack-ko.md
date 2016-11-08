title: AWS CloudWatch Alarm 을 slack으로 보내기
author: Cheol Kang
tags:
  - aws
categories:
  - cloud
date: 2016-11-08 18:25:00
---
# AWS CloudWatch Alarm 을 slack으로 보내기

### 흐름

![](http://corporate-tech-blog-wp.s3.amazonaws.com/tech/wp-content/uploads/2016/02/blueprint-flow.png)

이 그림 대로이다. CloudWatch에서 Alarm이 발생되면, SNS로 그걸 알리고, SNS는 그걸 다시 Lambda랑 연결 시켜서 Lambda가 Slack에 통보를 한다.

SNS에 토픽을 만들자

![](https://cloud.githubusercontent.com/assets/8134523/19831259/675ae122-9e40-11e6-9680-e3e1856c0de1.png)

나는 cloudWatch-alarm으로 만들었다.

이제 cloudWatch에 들어가서 Alarm의 Action을 수정해주자.

![](https://cloud.githubusercontent.com/assets/8134523/19831263/aff24e66-9e40-11e6-99cd-0d7e478c214a.png)

Actions을 보면 ALARM이 울릴 때, cloudWath-alarm로 이벤트를 보내준다.

이제 Lambda 을 생성해주자.

![](https://cloud.githubusercontent.com/assets/8134523/19831278/00fa0e52-9e41-11e6-8004-7857ae959a04.png)

Blank Function을 선택해주자. slack blueprint도 있지만 설정이 복잡해 져서 그냥 blank Function으로 했다( 이 경우 보안이 약해진다)

![](https://cloud.githubusercontent.com/assets/8134523/19831284/3e3ab5f0-9e41-11e6-8495-7d68f056f453.png)

이렇게 해주면 SNS로 간 이벤트로 인해서 Lambda가 실행된다. 

그 다음 node code 로

```js
var url = require('url');
var https = require('https');
 
// TODO: !!! Must be changed following Slack WebHook URL !!!
var hookUrl = '<Slack webHookURL>';
 
var processEvent = function(event, context) {
    var message = JSON.parse(event.Records[0].Sns.Message);
 
    // Format Slack posting message
    var text = "<!channel> *" + message.AlarmDescription + "* state is now `" + message.NewStateValue + "`\n" +
               "```" + 
               "reason: " + message.NewStateReason + "\n" +
               "alarm: " + message.AlarmName + "\n" + 
               "time: " + message.StateChangeTime +
               "```"
               ;
 
    var slackMessage = {
        text: text
    };
 
    postMessage(slackMessage, function(response) {
        if (response.statusCode < 400) {
            console.info('Message posted!');
            context.succeed();
        } else if (response.statusCode < 500) {
            console.error("4xx error occured when processing message: " + response.statusCode + " - " + response.statusMessage);
            context.succeed(); // Don't retry when got 4xx cuz its request error
        } else {
            // Retry Lambda func when got 5xx errors
            context.fail("Server error when processing message: " + response.statusCode + " - " + response.statusMessage);
        }
    });
};
 
var postMessage = function(message, callback) {
    var body = JSON.stringify(message);
    var options = url.parse(hookUrl);
    options.method = 'POST';
    options.headers = {
        'Content-Type': 'application/json',
        'Content-Length': Buffer.byteLength(body),
    };
 
    var postReq = https.request(options, function(res) {
        var chunks = [];
        res.setEncoding('utf8');
        res.on('data', function(chunk) {
            return chunks.push(chunk);
        });
        res.on('end', function() {
            var body = chunks.join('');
            if (callback) {
                callback({
                    body: body,
                    statusCode: res.statusCode,
                    statusMessage: res.statusMessage
                });
            }
        });
        return res;
    });
 
    postReq.write(body);
    postReq.end();
};
 
exports.handler = function(event, context) {
    if (hookUrl) {
        processEvent(event, context);
    } else {
        context.fail('Missing Slack Hook URL.');
    }
};
```

이렇게 한다. 여기서 webHookURL은

![](https://cloud.githubusercontent.com/assets/8134523/19831299/c1f2ca4a-9e41-11e6-867e-19277a34dfdb.png)

Slack에서 App & Integrations 로 들어가서 incomingWebHook을 적당히 자신이 원하는 채널에 추가하면 URL이 생성된다.

![](https://cloud.githubusercontent.com/assets/8134523/19831300/c2c830cc-9e41-11e6-9e2e-7b15bcb2f99b.png)

role은 lambda 만 실행할수 있는 권한이면 되니, 미리 만들어 놓은 role에 lambda을 추가하거나, 새로 role을 만들면 된다. 

이러면 완료이다. 나 같은 경우는 dynamoDB의 cloudWatch을 추가했기 때문에 적당히 

```python
while true; do aws dynamodb get-item --table-name testTable --key '{"key": {"S": "abc"}}' 2>&1; done
```

이걸 실행해서 alarm을 울리게 했다.

![](https://cloud.githubusercontent.com/assets/8134523/19831351/bb8109c8-9e42-11e6-93c0-a43df334a8be.png)

문제 없이 잘 되는것 같다. 

보기 불편 하니 이모티콘도 넣어서 만들면 

![](https://cloud.githubusercontent.com/assets/8134523/19831407/ff414bae-9e43-11e6-82e4-a36a723eacc0.png)

이렇게 된다. 이건 lambda의 코드에서 `var text` 부분을

```js
var status = message.NewStateValue;
      if (status === "ALARM") {
          status = ":exclamation: " + status;
      }
      if (status === "OK") {
          status = ":+1: " + status;
      }
var text = "*" +
                status +
                ": " +
                message.AlarmDescription +
                "*" +
                "\n" +
                message.NewStateReason;
```

이렇게 수정하면 된다.

적당히 자기 입맛에 맞게 수정해서 보기 편하게 바꾸면 된다.

나는

```js
var status = message.NewStateValue;
      if (status === "ALARM") {
          status = ":exclamation: " + status;
      }
      if (status === "OK") {
          status = ":+1: " + status;
      }
      var text = "*" +
                status +
                ": " +
                message.AlarmName +
                "*" +
                "\n" +
                "From: " + message.AlarmDescription + "\n" +
                "To: " + message.NewStateValue + "\n" +
                "reason: " + message.NewStateReason + "\n" +
                "time: " + message.StateChangeTime;
```



![](https://cloud.githubusercontent.com/assets/8134523/19831535/01ef7a94-9e47-11e6-80f0-21296603adf5.png)

이런식으로 바꾸었다. Slack text 을 잘 사용한다면 보다 예쁜 Alarm을 볼수 있을 것이다…….