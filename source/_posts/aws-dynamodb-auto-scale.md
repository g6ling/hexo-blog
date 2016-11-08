title: AWS DynamoDB을 Lambda로 Auto Scaling 하기
author: Cheol Kang
tags:
  - aws
categories:
  - cloud
date: 2016-11-08 18:25:00
---
# AWS DynamoDB을 Lambda로 Auto Scaling 하기

DynamoDB는 언제든지 scale-in/out 이 가능한 좋은 DB인데 자체적으로 auto scaling을 지원하지는 않습니다.

하지만 CloudWatch와 Lambda로 AutoScaling 비슷한 거 정도는 구현할 수 있습니다.

대충 흐름은 이런 느낌입니다

![](https://cloud.githubusercontent.com/assets/8134523/19892198/20a6ef16-a087-11e6-8893-dd3fa38f6cb2.png)

일단 DynamoDB에서 일정이상의 사용률이 된다면( cosumed Provision이 provied Provision 보다 더 커진다면) Cloud Watch의 알람이 울리게 됩니다. 이 알람은 SNS로 전송되고 SNS는 Alram 이벤트를 받는다면 그 이벤트를 Lambda로 넘겨주면서 실행하게 됩니다. 그렇다면 Lambda는 Alaram이 울린 DynamoDB의 용량(provied Provision)을 증가시켜줍니다. 

## 구성하기

일단 적당히 `scaleDynamoTest`라는 이름을 가진 DynamoDB을 생성합니다.

`Auto_Scale_DynamoDB_SNS`라는 SNS Topic을 생성합니다.

cloudWatch에서는 아마 자동으로 이름이 `scaleDynamoTest-ReadCapacityUnitsLimit-BasicAlarm` , `scaleDynamoTest-WriteCapacityUnitsLimit-BasicAlarm` 인 alarm이 만들어져 있을겁니다.

![](https://cloud.githubusercontent.com/assets/8134523/19892458/56d056da-a088-11e6-95e1-48ed365ea5c7.png)

`scaleDynamoTest-ReadCapacityUnitsLimit-BasicAlarm`의 Alarm이 좀더 울리기 쉽게 20이상일때 alram이 되도록 바꾸고, Action을 만들어놓은 SNS Topic인 `Auto_Scale_DynamoDB_SNS` 으로 변경해 줍시다. 이제 DynamoDB에서 사용량이 20이 넘는다면 알람이 울리고 그 알람은 SNS로 넘겨집니다. 

이제 Lambda에 들어가서 새로운 `Function`을 만듭시다. 일단 `Blank Function`으로 만들어봅시다. 그 다음 Trigger을 SNS의 `Auto_Scale_DynamoDB_SNS` Topic 으로 설정합시다.

![](https://cloud.githubusercontent.com/assets/8134523/19892590/f17528be-a088-11e6-8f61-4e3210487823.png)

function 의 코드는

```js
exports.handler = (event, context, callback) => {
    console.log(event);
    // TODO implement
    callback(null, 'Hello from Lambda');
};
```

이렇게 해서 일단 어떤 event가 넘겨지는지 봅시다. Function의 이름은 `Auto-Dynamo-Function`정도로 합시다. Role 부분은 IAM에 들어가서 DynamoDB, CloudWatch, Lambda에 액세스 할수 있는 권한을 주고 그걸 사용합시다(이 부분은 잘 모르시면 검색해 주세요)

이제 CloudWatch Alarm을 울려봅시다.

터미널에 

```python
while true; do aws dynamodb get-item --table-name scaleDynamoTest --key '{"key": {"S": "testKey"}}' 2>&1; done
```



을 실행시키면 계속 Read을 하게 됩니다. 물론 그전에 적당히 Item을 넣어줍시다.(또한 aws-cli가 설치 되어 있어야 합니다 )

![](https://cloud.githubusercontent.com/assets/8134523/19893876/20bd4e44-a08e-11e6-8375-f25a0f12166b.png)

대충 이런 구조군요. Sns가 어떤 Object인지 알기 위해서 코드를

```json
exports.handler = (event, context, callback) => {
    console.log(event.Records[0].Sns);
    // TODO implement
    callback(null, 'Hello from Lambda');
};
```

로 바꿔서 다시 결과를 보면

```js
{ 
	Type: 'Notification', 
	MessageId: '87a2280f-e75e-5083-ae39-789d4a546070', 
	TopicArn: 'arn:aws:sns:ap-northeast-1:772235007445:Auto_Scale_DynamoDB_SNS', 
	Subject: 'ALARM: "scaleDynamoTest-ReadCapacityUnitsLimit-BasicAlarm" in Asia Pacific - Tokyo', 
	Message: '{
		"AlarmName":"scaleDynamoTest-ReadCapacityUnitsLimit-BasicAlarm",
		"AlarmDescription":null,
		"AWSAccountId":"772235007445",
		"NewStateValue":"ALARM",
		"NewStateReason":"Threshold Crossed: 1 datapoint (26.5) was greater than or equal to the threshold (5.0).",
		"StateChangeTime":"2016-11-01T14:46:34.006+0000",
		"Region":"Asia Pacific - Tokyo",
		"OldStateValue":"INSUFFICIENT_DATA",
		"Trigger":{
			"MetricName":"ConsumedReadCapacityUnits",
			"Namespace":"AWS/DynamoDB",
			"Statistic":"SUM",
			"Unit":null,
			"Dimensions":[
					{
						"name":"TableName",
						"value":"scaleDynamoTest"
					}
				],
			"Period":60,
			"EvaluationPeriods":1,
			"ComparisonOperator":"GreaterThanOrEqualToThreshold",
			"Threshold":5.0
		}
	}', 
	Timestamp: '2016-11-01T14:46:34.132Z', 
	SignatureVersion: '1', 
	Signature: '****', 
	SigningCertUrl: '****', 
	UnsubscribeUrl: '****',
	MessageAttributes: {} 
}
```

대충 이런구조군요. 

`message.Trigger.Dimensions[0].value`가 TableName 이군요. 이 정보로 다시 DynamoDB의 정보를 가져오고 현재 Capacity을 알 수 있을거고, 그 값을 이용해서 새로운 Capacity로 업그레이드 할 수 있을거 같군요.



이제 적당히 코드를 적어봅시다.

```js
var AWS = require('aws-sdk');
 
var increaseReadPercentage = 20;
var increaseWritePercentage = 20;
var readAlarmThreshold = 80;
var writeAlarmThreshold = 80;
 
var region = {
  'APAC - Tokyo': 'ap-northeast-1',
  '...': '...'
};
 
exports.handler = function(event, context, callback) {
    console.log(event);
  var message = JSON.parse(event.Records[0].Sns.Message) ;
    if (message.NewStateValue =! 'ALARM') {
    context.callbackWaitsForEmptyEventLoop = false;
    callback(null, message);
  }
 
  AWS.config.update({ region: region[message.Region]});
  // Waterfall을 통해서 동기적인 작업을 합니다.
  var dynamodb = new AWS.DynamoDB();
  // dynamoDBTableName을 가져옵시다.
  var dynamodbTable = message.Trigger.Dimensions[0].value;
  dynamodb.describeTable({TableName: dynamodbTable}, function(err, tableInfo) {
    // dynamodbTable이라는 이름을 가진 table 정보를 가져옵니다.
    if (err) {
      context.callbackWaitsForEmptyEventLoop = false;
      callback(err);
    }
    else {
      var params = {
        TableName: dynamodbTable
      }
   
      var currentThroughput = tableInfo.Table.ProvisionedThroughput;
      // Alarm이 ReadCapacity에 의해서 생겼다면, ReadCapacity을 업그레이드 해줍시다.
      if (message.Trigger.MetricName == 'ConsumedReadCapacityUnits') {
        params.ProvisionedThroughput = {
          ReadCapacityUnits: Math.ceil(currentThroughput.ReadCapacityUnits * (100 + increaseReadPercentage)/100),
          WriteCapacityUnits: currentThroughput.WriteCapacityUnits
        };
      }
      // 마찬가지로 WriteCapacity에 의해서 생겼다면, WriteCapacity을 업그레이드 해줍시다.
      else if (message.Trigger.MetricName == 'ConsumedWriteCapacityUnits') {
        params.ProvisionedThroughput = {
          ReadCapacityUnits: currentThroughput.ReadCapacityUnits,
          WriteCapacityUnits: Math.ceil(currentThroughput.WriteCapacityUnits * (100 + increaseWritePercentage)/100)
        };
      }
      else {
        context.callbackWaitsForEmptyEventLoop = false;
        callback(err);
      }
      // 테이블을 업데이트 합시다.
      dynamodb.updateTable(params, function(err, response) {
        if (err) {
          context.callbackWaitsForEmptyEventLoop = false;
          callback(err);
        }
        else {
          console.log('modify provisioned throughput:', currentThroughput, params);
          context.callbackWaitsForEmptyEventLoop = false;
          callback(null, params.ProvisionedThroughput);
        }
      });
    }
  });
}
```

이제 다시 이벤트를 발생시켜 보면

![](https://cloud.githubusercontent.com/assets/8134523/19896967/85ddf3b8-a099-11e6-86dd-f71bbe146360.png)

어느정도 시차가 있는 거 같지만 잘 작동이 되는거 같군요. 시차는 알람이 바로 울리는게 아니기 때문에 그렇거 같군요. CloudWatch 에서 설정을 바꾸면 좀 더 빨리 알람이 울릴것 같아요. 지금 문제는 

![](https://cloud.githubusercontent.com/assets/8134523/19897112/2c4640ca-a09a-11e6-991a-f6d8cfecb6bd.png)

지속적으로 계속 Capacity가 증가 됩니다. 이유는 DB의 Capacity는 커졌지만 CloudWatch의 알람조건이 바뀌지는 않았기 때문이죠. 이제 DB을 수정한 뒤 cloudWatch도 수정하도록 해봅시다.

```js
var AWS = require('aws-sdk');
var async = require('async');
 
var increaseReadPercentage = 50;
var increaseWritePercentage = 50;
var readAlarmThreshold = 50;
var writeAlarmThreshold = 50;
 
var region = {
  'APAC - Tokyo': 'ap-northeast-1',
  '...': '...'
};
 
exports.handler = function(event, context, callback) {
  
  var message = JSON.parse(event.Records[0].Sns.Message);
  if (message.NewStateValue =! 'ALARM') {
    context.callbackWaitsForEmptyEventLoop = false;
    callback(null, message);
  }
 
  AWS.config.update({ region: region[message.Region]});
  // Waterfall을 통해서 동기적인 작업을 합니다.
  var dynamodb = new AWS.DynamoDB();
  // dynamoDBTableName을 가져옵시다.
  var dynamodbTable = message.Trigger.Dimensions[0].value;
  async.waterfall(
    [
      // 일단 DynamoDB 을 새로운 Capacity을 갖도록 업데이트를 해줍시다.
      function (cb) {
        var dynamodb = new AWS.DynamoDB();
        // dynamoDBTableName을 가져옵시다.
        var dynamodbTable = message.Trigger.Dimensions[0].value;
        dynamodb.describeTable({TableName: dynamodbTable}, function(err, tableInfo) {
          // dynamodbTable이라는 이름을 가진 table 정보를 가져옵니다.
          if (err) cb(err);
          else {
            var params = {
              TableName: dynamodbTable
            }
 
            var currentThroughput = tableInfo.Table.ProvisionedThroughput;
            // Alarm이 ReadCapacity에 의해서 생겼다면, ReadCapacity을 업그레이드 해줍시다.
            if (message.Trigger.MetricName == 'ConsumedReadCapacityUnits') {
              params.ProvisionedThroughput = {
                ReadCapacityUnits: Math.ceil(currentThroughput.ReadCapacityUnits * (100 + increaseReadPercentage)/100),
                WriteCapacityUnits: currentThroughput.WriteCapacityUnits
              };
            }
            // 마찬가지로 WriteCapacity에 의해서 생겼다면, WriteCapacity을 업그레이드 해줍시다.
            else if (message.Trigger.MetricName == 'ConsumedWriteCapacityUnits') {
              params.ProvisionedThroughput = {
                ReadCapacityUnits: currentThroughput.ReadCapacityUnits,
                WriteCapacityUnits: Math.ceil(currentThroughput.WriteCapacityUnits * (100 + increaseWritePercentage)/100)
              };
            }
            else {
              cb(message);
            }
            // 테이블을 업데이트 합시다.
            dynamodb.updateTable(params, function(err, response) {
              if (err) cb(err);
              else {
                console.log('modify provisioned throughput:', currentThroughput, params);
                cb(null, params.ProvisionedThroughput);
              }
            });
          }
        });
      },
      // CloudWatch의 알람도 업데이트 해주어야 합니다. 새로운 Capacity을 가졌기 때문이죠.
      function (newThroughput, cb) {
        var cloudwatch = new AWS.CloudWatch();
        cloudwatch.describeAlarms({AlarmNames: [message.AlarmName]}, function(err, alarms) {
          if (err) cb(err);
          else {
            var currentAlarm = alarms.MetricAlarms[0];
            var params = {
              AlarmName: currentAlarm.AlarmName,
              ComparisonOperator: currentAlarm.ComparisonOperator,
              EvaluationPeriods: currentAlarm.EvaluationPeriods,
              MetricName: currentAlarm.MetricName,
              Namespace: currentAlarm.Namespace,
              Period: currentAlarm.Period,
              Statistic: currentAlarm.Statistic,
              ActionsEnabled: currentAlarm.ActionsEnabled,
              AlarmActions: currentAlarm.AlarmActions,
              AlarmDescription: currentAlarm.AlarmDescription,
              Dimensions: currentAlarm.Dimensions,
              InsufficientDataActions: currentAlarm.InsufficientDataActions,
              OKActions: currentAlarm.OKActions,
              Unit: currentAlarm.Unit,
            }
            if (message.Trigger.MetricName == 'ConsumedReadCapacityUnits') {
              params.Threshold = newThroughput.ReadCapacityUnits * params.Period * readAlarmThreshold / 100;
            }
            else {
              params.Threshold = newThroughput.WriteCapacityUnits * params.Period * writeAlarmThreshold / 100;
            }
            cloudwatch.putMetricAlarm(params, function(err, response) {
              if (err) cb(err);
              else {
                console.log('modify alarm:', currentAlarm, params);
                cb(null, response);
              }
            });
          }
        });
      }
    ],
    function (err, result) {
      context.callbackWaitsForEmptyEventLoop = false; 
      if (err) {callback(err);}
      else {callback(null, result);}
    }
  );
}
```

이런 코드를 작성해 줍시다. 이걸 그대로 넣으면 모듈을 찾을수 없다는 에러가 나옵니다. library을 사용할 경우에는 library 까지 업로드를 해야하는것 같습니다.

일단 폴더를 1개 만듭시다. 저는 `autoDynamic` 이라는 이름으로 폴더를 만들고 폴더안에 `index.js`라는 파일을 생성하고 위에 코드를 넣읍시다. 그 다음, `npm init` 명령어를 통해서`package.json`을  생성합시다. 그 뒤, `npm i async —save` 을 통해서 모듈을 설치해 줍시다. 이제 이 폴더를 압축합시다. 그 압축시킨 파일을 upload 해주시면 됩니다. 이제  Configuration의 Handler의 값을 `압축파일명.handler` 로 변경해주세요.

![](https://cloud.githubusercontent.com/assets/8134523/19901745/84d058dc-a0ab-11e6-8076-e5bbf7f14160.png)

저는 파일명이 `autoDynamic` 이였기 때문에 `autoDynamic.handler`로 수정하였습니다.

이제 다시 테스트를 해보면 

![](https://cloud.githubusercontent.com/assets/8134523/19901815/c3e6aff8-a0ab-11e6-9a29-e964c8fee230.png)

이렇게 Capacity가 증가하면, 

![](https://cloud.githubusercontent.com/assets/8134523/19901728/7350245c-a0ab-11e6-9038-d54fcebe8c4f.png)

이게 (사실은 1 minutes로 수정하였지만 사진을 찍지 못해서 이걸로 대체 합니다. ㅠㅠ)

![](https://cloud.githubusercontent.com/assets/8134523/19901734/7657c06a-a0ab-11e6-81aa-b87a301320f5.png)

이렇게 Capacity가 1분에 150 이상일 경우 알람이 울리도록 바뀌었습니다. `1초당5개의 처리 * 60초 * 50%`의 결과 150이 정해졌네요. 만약 여기서 150이상의 요청이 들어온다면 알람이 울리고 `현재 Capacity (5) * 50%` 인 `2.5`의 올림, 즉 3만큼의 Capacity가 증가하게 되고, `8*60*50%` 즉 240의 알람으로 바뀌게 되겠네요. 

하지만 이 경우 문제점이 몇개 생깁니다. 

1. 크기를 키우는 것은 되는데, 크기를 줄이는 것이 안된다.
2. 테이블마다 사용량이 급격히 증가하는 테이블이 있을수도 있고, 사용량이 거의 증가되지 않는 테이블이 있을수 있기 때문에 테이블마다 세부 설정을 할 수 있게 해야한다.

정도 일텐데, 2번은 설정파일을 만들고, 그 설정파일을 참조해서 테이블 Capacity을 조정하는 방법으로 됩니다. 하지만 1번은 조금 생각을 해보아야 하군요. 

### 첫번째 방법

일정 Capacity 이하를 사용할때 Alarm 을 울리도록 하는 이벤트를 추가해서, 이 알람이 울린다면 Capacity을 줄인다.

### 두번째 방법

일정주기마다 테이블의 정보를 가져와서, 테이블의 Capacity가 일정 이하라면 Capacity을 줄인다.

결국 2가지 방법의 차이는 어떤 이벤트로 Capacity을 줄이는 작업을 할 것이냐 의 차이 이겟네요.

첫번째는 Alarm, 두번째는 일정주기(CloudWatch Schedule) 입니다.

이 포스트 에서는 두번째 방법으로 하겠습니다. 

이유는 

1. 첫번째 방법은 위에 있는 코드만 약간 수정하고, 알람만 추가한다면 할 수 있습니다.
2. 이게 큰 이유 인데요, DynamoDB는 스케일 다운을 하루 4번으로 제한합니다. 

하루 4번 밖에 쓸 수 없기 때문에, 알람이 울릴때 마다 스케일 다운을 한다면, 정말로 필요할때 못쓸수도 있기 때문이죠. 또한 만약 사용자가 언제 몰릴지 안다면 Scale Down은 스케쥴로 하는게 더 옳은 방법 같습니다. (Schedule 이벤트는 일정 시간마다로도 설정이 가능하고, 특정요일 특정 시간등 여러가지 조건을 만들수 있습니다)



그렇다면 일단 적당히 Lambda Function을 만들어 보죠.

스케일 다운의 흐름은 

![](https://cloud.githubusercontent.com/assets/8134523/20055024/ac0e419a-a522-11e6-9256-b2c5d1553cf6.png)

대충 이런 느낌이겠네요. 이제 실제로 만들어 보도록 하죠.

![](https://cloud.githubusercontent.com/assets/8134523/19902539/86beb078-a0ae-11e6-91a9-9efc01b404bb.png)

저는 이번에 5분마다 Event을 발생시키도록 만들었습니다. 

이제 우리는 5분마다 무엇을 하면 될까요? cloudWatch에 있는 dynamoDB의 log을 읽어서 현재의  `consumed ReadCapacity` 와 `consumed WriteCapacity` 을 구하고 그 값이현재 `provided`의 값보다 일정수준 이하로 작다면 새로 DynamoDB 의 Capacity을 조정해 주고, cloudWatch의 값을 조정해 주면 됩니다.

이걸 다 짤려고 하니 복잡해서 라이브러리를 찾아보니 좋은 라이브러리가 있더군요. [https://github.com/rockeee/dynamic-dynamodb-lambda](https://github.com/rockeee/dynamic-dynamodb-lambda) 이걸 사용합시다. 

안에는 config.js 파일이 있습니다. 이부분만 수정해 줘서 올리면 간단하게 됩니다. 

git clone 을 한뒤, config.js 파일을 수정하고, zip으로 압축해서 올립시다. 

![](https://cloud.githubusercontent.com/assets/8134523/20061016/440ac482-a541-11e6-942e-c4fe49ca8847.png)

흐음 근데 5분에 1번씩 하다보니 스케일 다운 하루 제한을 금방 채우게 되는군요. 1시간에 1번으로 실행을 하도록 바꿉시다.

스케일 업은 Alarm 통해서 하도록 하고, 스케일 다운은 스케쥴을 통해서 하도록 합시다. 

![](https://cloud.githubusercontent.com/assets/8134523/20084987/31ec8be2-a5a9-11e6-9543-7d4a54e7f990.png)

잘 되는 것 같군요. 

코드는 [https://github.com/g6ling/auto_dynamoDB_scale](https://github.com/g6ling/auto_dynamoDB_scale) 에 올렸습니다. 알람을 1분 간격으로 울리게 했더니, 알람이 연속적으로 계속 울려서 너무 많이 scale up 이 됬습니다. 적당히 5분 15분 간격정도로 하게 합시다. scaleDown 은 상황에 알맞은 시간에 할 수 있도록 만들면 됩니다. 만약 모바일 게임 이라면 아침 출근이 끝난 10시 점심시간이 끝난 2시 새벽 2시 이런 시간에 scaleDown을 하게 한다면 좀 더 효율적인 autoScale이 되겠지요. 