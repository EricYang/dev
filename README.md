ROBOT TEST
======

API測試用

# 如何使用
上傳任務檔案(json)就會自動執行，然後回傳任務的結果

#任務格式(json)
```
{
  name:String
  task:Array
    [
      /task item/
      { 
      name:String(require),
      uri:Sting(require),
      method:String(require),
      qs:Object,
      form:Object,
      json:Object,
      target:Object,
      sleep:Number,
      repeat:Object,
      ...
      },
      ...
    ]
}
```
範例一： 登入後再取資料
```
{
    "name":"server mission"(文件名),
        "task":[{"name":"login" (必須的),
                  "uri":"https://api2.karaokecloud.com/user/prologin"(連線位址,必須的),
                  "qs":{"username":"deansoft","password":"deansoft1234","app":"n6MKZWfZC3","key":"1wzy0GntJgwmLpPepZYg5mlQ8lNmkpplWYT6Fgyox6LivBwUAv"}(放query，也就是你GET要附加的params),
                  "method":"GET"(GET or POST or PUT or DELETE,必須的),
                  "target":{"obj":"status","operator":"==","val":"success"}(驗證某一個欄位是否是否是你預期的)
                },
                { "name":"list" (必須的),
                  "uri":"https://api2.karaokecloud.com/apps/list",
                  "qs":{"token":"$$login.data.token"(利用$$取得某task名的物件屬性值),"app_name":"SINGNSHARE"},
                  "method":"GET",
                  "target":{"obj":"data","property":"length","operator":">","val":1}
            
        }
    ]}

```

範例二： rank測試
```
{
    "name":"media-rank",
        "task":[
        {"uri":"http://media.karaokecloud.com/rank/songs.playcount",
            "qs":{"rank_type":"day"},
            "method":"GET",
            "name":"songs.playcount.day",
            "target":{"obj":"info","property":"length","operator":">","val":1}
        },
        {"uri":"http://media.karaokecloud.com/rank/songs.playcount",
            "qs":{"rank_type":"week"},
            "method":"GET",
            "name":"songs.playcount.week",
            "target":{"obj":"info","property":"length","operator":">","val":1}
        },
        {"uri":"http://media.karaokecloud.com/rank/songs.playcount",
            "qs":{"rank_type":"month"},
            "method":"GET",
            "name":"songs.playcount.month",
            "target":{"obj":"info","property":"length","operator":">","val":1}
        },
        ... ...]
}
```

範例二： 排程任務
```
{
    "name":"scan-sns-user",
        "task":[
        {"uri":"http://media.karaokecloud.com:8081/login",
            "form":{"username":"admin","password":"xxxxx","api":"true"},
            "method":"POST",
            "name":"login",
            "sleep":1000(可以先休息1000毫秒再執行下一個任務)
        },
        {"uri":"http://media.karaokecloud.com:8081/ajax/singnshare/covers",
            "qs":{"limit":1000,"token":"$$login.data.token","save_mode":"true"},
            "method":"GET",
            "name":"singnshare.cover",
            "sleep":5000
        },
        {"uri":"http://media.karaokecloud.com:8081/ajax/singnshare/user",
            "qs":{"limit":100,"token":"$$login.data.token","save_mode":"true"},
            "method":"GET",
            "name":"singnshare.user",
            "sleep":5000,
            "repeat":{"field":"page","parent":"qs","start":0,"end":230}(重複執行可以指定某個重複的欄位去遞增任務，在這裏執行page由0到230，所以會增加230個任務去執行)
        }

        ]}
```
#回傳格式(json)
```
{
info:Object(無特別情況為null),
data:{
    xxxx(name of task):{
      head:{
          檔頭傳遞資訊
          },
      body:{
          回應資料
          }
    }

    }
}
```

以下列transcode mission為例：
```
傳遞的json檔案
{
    "name":"transcode",
    "task":[{"uri":"http://54.196.25.114/job/new",
                        "json":{"quality": "240p",
                                "internal_id": "C18156-C",
                                "type": "m4a",
                                "override":"yes"
                        },
                        "method":"POST",
                        "name":"jobNew"
                        }
            ]}
```

```
回應的資料
{
info: null,
data: {
jobNew: {
head: {
uri: "http://54.196.25.114/job/new",
json: {
quality: "240p",
internal_id: "C18156-C",
type: "m4a",
override: "yes"
},
method: "POST",
name: "jobNew",
cost_times: 52,
status: 200
},
error: null,
body: {
id: "3b7802f4-eb00-4b5f-88f3-82cf9cf8c67f",
internal_id: "C18156-C",
override: "yes",
quality: "240p",
type: "m4a"
}
}
}
}
```

