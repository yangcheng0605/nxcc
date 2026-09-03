

# 开始

## 快速开始
### 初始化你的web服务器，必须使用https访问。

### 获取用户账号

jssdk在登录时，需要使用nxlink账号，也就是下面示例中获取TOKEN接口所需要的账号密码（账号为单点登录），您可以前往[NXLINK](https://nxlink.nxcloud.com/admin/#/register)获取和管理它们。

## SDK使用说明

### SDK使用步骤
1. 导入lib中的 nxwebrtc.js。
2. 定义profile，设置 nxuser,nxpass(话机账号),logLevel,playTone等属性 ，
3. new NxwCall(profile) 创建对象 nxwcall，并基于 nxwcall.myEvents 设置回调方法。 
4. nxwcall会自动启动状态机，在注册成功后，进入 UA_READY 状态，可呼入呼出。
5. 通常需要在回调方法中，针对收到的事件，执行相关的处理。可以调用api执行对应的功能：发起呼叫、接通呼叫、挂断呼叫。

### 示例
#### 1. 引入nxwebrtc.js 
```html
<script src="your/path/nxwebrtc.js"></script>
```
```js
let NxwCall = NXW.default;  //对象的类型声明
let nxwcall = null;         //对象的全局实例，尚未初始化
```

#### 2. 获取TOKEN、话机注册信息

https://nxlink.nxcloud.com//admin/saas_plat/user/login     api请求获取Token (Token不进行任何操作的话5个小时过期，有接口请求就会一直续期)

```
请求体
curl --location --request PUT 'https://nxlink.nxcloud.com/admin/saas_plat/user/login' \
--header 'lang: zh_CN' \
--header 'User-Agent: apifox/1.0.0 (https://www.apifox.cn)' \
--header 'Content-Type: application/json' \
--data-raw '{
	loginMethod: 0， // 0 邮箱方式登录 1 手机号方式登录
	email: "email"， // 邮箱方式登录，手机号登陆：phone： “phone”, 
	password: "password",
    graphVerificationCode: "", // 必填，空字符串
    key: ""   // 必填，空字符串
}'
返回体
{
    "code": 0,
    "message": "",
    "data": {
        "token": "eyJhbGciOiJIUzI1NiJ9.eyJ1SWQiOjEsInV1SWQiOiI2NGMyMjBjNmIxYmMyNjUyM2NjMTIyZmQifQ.uMMrkXVSAXht0AN_KlxN2LHhS9GwlmTYYp61Rm0QXGk",
        "lang": "zh_CN",
        "time_zone_id": 105,
        "time_zone": "UTC+08:00"
    },
    "traceId": "f062fc22055bd86f236ea63032fed48e"
}

```

https://nxlink.nxcloud.com/cc/api/fs/webCall/register    api请求获取注册信息

```
请求体
curl --location --request POST 'https://nxlink.nxcloud.com/cc/api/fs/webCall/register' \
--header 'usertoken: eyJhbGciOiJIUzI1NiJ9.eyJ1SWQiOjEsInV1SWQiOiI2NGMxY2I2Y2IxYmNlYzE0NjM1ZTIyMGUifQ.rYlUFXIqTnP9vCAkkHIU_jGl5SO_oBJq4nzKp8Ivx7g' \
--header 'lang: zh_CN' \
--header 'Authorization;' \
--header 'User-Agent: apifox/1.0.0 (https://www.apifox.cn)' \
--header 'Content-Type: application/json' \
--data-raw '{
    "agentName": "8618344031797"
}'
返回体
{
    "reqId": "41aa5b571b66b33c028c1ef1b1204ecf",
    "code": 0,
    "msg": "请求成功",
    "data": {
        "domain": "domain", // 域名
        "groupNo": "groupNo", // 坐席组
        "sipNum": "sipNum", // 坐席账号
        "url": "wssurl", // wssurl
        "email": "email", // 邮箱
        "tenantNickName": ’tenantNickName‘,// 租户昵称
        "company": ’company‘, // 认证企业
        "tenantName": ’tenantName‘, // 租户
        "roleName": ’roleName‘, // 角色
        "utcDate": "2023/07/27", //utc时间
    }
}

```



#### 3. 定义profile
```js
let profile = {
    nxuser: “xxxxxxxx”,  // 话机信息中的sipNum坐席账号
    nxpass:”xxxxxxx”, // 需md5加密，如：MD5(email + ":" + domain + ":" + utcDate)
    logLevel: "error", 
    playTone: 0xFF, 
    nxtype: 6,
    audioElementId: "remoteAudio", 
    playElementId: "playAudio",
    audioSrcPath: "https://nxcc-sgp-test-1259196162.cos.ap-singapore.myqcloud.com/static/resource/audio",
    domain: `${domain}`,  // 话机信息中的doman域名
    wssurl: `${wssurl}`,  // 话机信息中的wssurl
    ccAgent: `${email}`,  // 话机信息中的email邮箱
    ccToken: `${Token}`,    //  登录接口获取的token'
    ccQueue: `${groupNo}`  // 话机信息中的ccQueue坐席组
  };
```
 - audioElementId与playElementId 是页面的audio组件的id
 - 如果需要自定义playTone请参考<a href='#audiolist'>列表</a>。


#### 4. 初始化并启动对象
把定义的profile作为NxwCall的构造函数的参数，会自动创建nxwcall对象，并且尝试自动执行状态转换，先执行NXAPI 认证，然后连接到wss服务器，然后注册成功后进入UA_READY状态。
```js
function initApp() {  
  if (nxwcall == null) {
    nxwcall = new NxwCall(profile)
    setupEvents(nxwcall);
  }
}
```

#### 5. 编写回调函数

```js
function setupEvents(nxwcall) {
    let e = nxwcall.myEvents;
    console.log("setupEvents e=", e)
	
    // 发起呼叫
    e.on("placeCall", function () {
        console.log("================", "placeCall")        
    });
    
    // 话机注册成功
    e.on("onRegistered", function () {
        console.log("================", "onRegistered")        
    });
    
    // 电话呼入
    e.on("onCallReceived", function (param1) {
        console.log("================", "onCallReceived", param1)
        // 获取呼入的号码 
        const numner = param1.split("@")[0]
    });
    
    // 接听
    e.on("onCallAnswered", function () {
        console.log("================", "onCallAnswered")
        // 获取当前通话秒数
        let callTimer = setInterval(() => {
          const t = nxwcall.talkingTime / 1000;
          const time = Math.floor(t);
          let isTimer = parseInt(time);
          if (isTimer > 0 && isTimer % 180 == 0) {
            changeSipStatu("Available", "In a queue call");
          }
        }, 1000);
    });
    
    // 呼出获取CcallId
    e.on("onAccept", function () {
        console.log("=====获取参数======", "onAccept", nxwcall.lastCcCallId);
    })
    
    // 挂断
    e.on("onCallHangup", function () {
      console.log("=====挂断后的操作======", "onCallHangup");
    })
    
    // 挂断原因映射
    e.on("onReject", function (param1) {
        let code = nxwcall.lastCcCode
        switch (code) {
            case "810":
                message.error(‘话机已欠费’);
                break;
                .... // 更多挂断原因见下面表格
        }
	}）
    
    // wss断开
    e.on("onServerDisconnect", function (param1) {
        console.log("=====获取参数======", "onServerDisconnect", param1);
    })
    
    // wss链接
    e.on("onServerConnect", function (param1) { 
     	console.log("================", "onServerConnect", param1);
    })
    
    // 话机注册失败
    e.on("onUnregistered", function () { 
        console.log("=====获取参数======", "onUnregistered");
        // 注册失败可重新初始化并启动对象
        initApp()
    })
    // 错误集合
    e.on("error", function (param1) {
        console.log("================", "error", param1);
        // 注册错误可重新初始化并启动对象
        initApp();
    });
}

```
| Code | 挂断原因                   |
| ---- | -------------------------- |
| 810  | 话机已欠费                 |
| 811  | 不允许呼叫的国家           |
| 812  | 号码不正确                 |
| 813  | 登录信息已过期，请重新登录 |
| 814  | 呼叫失败（黑名单号码）     |
| 815  | 呼叫失败（呼叫次数限制）   |
| 816  | 当前号码无法使用           |
| 800  | 网络异常                   |
| 801  | 网络异常                   |
| 817  | 今日呼叫体验已上限。       |
| 819  | DID号码呼叫失败。          |

nxwebrtc SDK库封装了多个<a href='#eventlist'>事件通知</a>,可以在相应的事件回调函数中，和业务逻辑互动。

#### 6. 话机状态变更

##### 1. 话机状态切换接口

status、state映射值

   | 页面状态 | status    | state           |
   | -------- | --------- | --------------- |
   | 示闲     | Available | Waiting         |
   | 示忙     | On break  | Idle            |
   | 通话中   | Available | In a queue call |
   | 振铃中   | Available | Receiving       |

   updateTime: 当前时间戳，agentName：当前坐席邮箱（注册接口回调的email）

```
   curl --location --request POST 'https://nxlink.nxcloud.com/cc/fs/ccAgent/status/change' \
   --header 'usertoken: eyJhbGciOiJIUzI1NiJ9.eyJ1SWQiOjkxLCJ1dUlkIjoiNjUwYTUxZTNiMWJjODBhMjg2NzYxNDM1In0.-8Vtg5Hc0g-		  e_1tHmXI_pSFjQuqD6t5YExoZs_cI3t4' \
   --header 'lang: zh_CN' \
   --header 'Authorization;' \
   --header 'User-Agent: apifox/1.0.0 (https://www.apifox.cn)' \
   --header 'Content-Type: application/json' \
   --data-raw '{"agentName":"fang.cheng@nxcloud.com","status":"Available","state":"Waiting","updateTime":1699344035}'
```

##### 2. 不同事件发送不同状态

1. 注册成功发送一次默认状态（示闲或者示忙）

   ```
   e.on("onRegistered", function () {
       const postForm = {
       	"agentName":"fang.cheng@nxcloud.com",
       	"status":"Available",
       	"state":"Waiting",
       	"updateTime":1699344035
       }       
   });
   // 当前示例为默认示闲
   ```

   

2. 振铃中发送一次振铃状态，placeCall和onCallReceived事件都需发送状态

   ```
   e.on("placeCall", function () {
       const postForm = {
       	"agentName":"fang.cheng@nxcloud.com",
       	"status":"Available",
       	"state":"Receiving",
       	"updateTime":1699344035
       }       
   });
   ```

3. 通话中发送一次通话状态

   ```
   e.on("onCallAnswered", function () {
       const postForm = {
       	"agentName":"fang.cheng@nxcloud.com",
       	"status":"Available",
       	"state":"In a queue call",
       	"updateTime":1699344035
       }       
   });
   ```

4. 通话结束发送一次呼入或者呼出前的默认状态

   ```
   e.on("onCallHangup", function () {
       const postForm = {
       	"agentName":"fang.cheng@nxcloud.com",
       	"status":"Available",
       	"state":"Waiting",
       	"updateTime":1699344035
       }       
   });
   // 示例为恢复原先默认示闲状态
   ```

   为保持后台和客户端状态一致，建议五分钟上报请求一次当前坐席示闲或者示忙状态，振铃或者通话中不需要上报

#### 7. 开始测试
在 onRegistered 回调完成之后，才能执行呼出、和处理呼入请求。
本地麦克风测试：
```js
nxwcall.placeCall('9196')
```
远程服务器测试：
```js
nxwcall.placeCall('4444')
```

完成测试后，代表你的电话通道已经就绪。

#### 8. 相关信息接口

1. 通话记录回调

   [](https://www.nxcloud.com/document/nxcc/nxccCdr)

2. 获取外呼显号

   ```
   curl --location --request POST 'https://nxlink.nxcloud.com/cc/api/ccDidNumber/v1/myUsefulNumber' \
   --header 'usertoken: eyJhbGciOiJIUzI1NiJ9.eyJ1SWQiOjkxLCJ1dUlkIjoiNjUwYTUxZTNiMWJjODBhMjg2NzYxNDM1In0.-8Vtg5Hc0g-e_1tHmXI_pSFjQuqD6t5YExoZs_cI3t4' \
   --header 'lang: zh_CN' \
   --header 'Authorization;' \
   --header 'User-Agent: apifox/1.0.0 (https://www.apifox.cn)'
   --header 'Content-Type: application/json' \   
   ```

3. 获取开通国家

   ```
   curl --location --request POST 'https://nxlink.nxcloud.com/cc/api/ccCountry/v1/allForMe' \
   --header 'usertoken: eyJhbGciOiJIUzI1NiJ9.eyJ1SWQiOjkxLCJ1dUlkIjoiNjUwYTUxZTNiMWJjODBhMjg2NzYxNDM1In0.-8Vtg5Hc0g-e_1tHmXI_pSFjQuqD6t5YExoZs_cI3t4' \
   --header 'lang: zh_CN' \
   --header 'Authorization;' \
   --header 'User-Agent: apifox/1.0.0 (https://www.apifox.cn)'
   --header 'Content-Type: application/json' \   
   ```

4. 获取最近通话30条

   ```
   curl --location --request POST 'https://nxlink.nxcloud.com/cc/api/ccCdr/v/recent' \
   --header 'usertoken: eyJhbGciOiJIUzI1NiJ9.eyJ1SWQiOjkxLCJ1dUlkIjoiNjUwYTUxZTNiMWJjODBhMjg2NzYxNDM1In0.-8Vtg5Hc0g-e_1tHmXI_pSFjQuqD6t5YExoZs_cI3t4' \
   --header 'lang: zh_CN' \
   --header 'Authorization;' \
   --header 'User-Agent: apifox/1.0.0 (https://www.apifox.cn)' \
   --header 'Content-Type: application/json' \
   --data-raw '{"direction":null,"answered":null}'   
   ```

5. 断开当前话机

   ```
   nxwcall.disconnect();
   nxwcall = null;
   ```

6. 退出当前登录账号接口

   ```
   curl --location --request PUT 'https://nxlink.nxcloud.com/admin/saas_plat/user/logout' \
   --header 'Authorization: eyJhbGciOiJIUzI1NiJ9.eyJ1SWQiOjkxLCJ1dUlkIjoiNjUwYTUxZTNiMWJjODBhMjg2NzYxNDM1In0.-8Vtg5Hc0g-e_1tHmXI_pSFjQuqD6t5YExoZs_cI3t4' \
   --header 'lang: zh_CN' \
   --header 'User-Agent: apifox/1.0.0 (https://www.apifox.cn)'
   --header 'Content-Type: application/json' \
   ```


## 主要方法
#### 发起呼叫

外呼显号呼出

```js
let hdrs = new Array(
  `X-NXCC-Out-Caller-Number:  ${选择的外呼显号}`,
  // callCountry为当前号码所属国家
  `X-Callee-Country-Code: ${
    callCountry == undefined ? "" : callCountry
  }`
);
nxwcall.placeCall(`${callCountry == undefined ? "" : callCountry}${state.numberText}`, hdrs);
```
随机显号呼出

```js
let hdrs = new Array(、
  // callCountry为当前号码所属国家
  `X-Callee-Country-Code: ${
    callCountry == undefined ? "" : callCountry
  }`
);
nxwcall.placeCall(`${callCountry == undefined ? "" : callCountry}${state.numberText}`, hdrs);
```

#### 接通来电
```js
nxwcall.answerCall()
```

#### 挂断呼叫
```js
nxwcall.hangupCall()  //对已经接通的呼出或呼入的SIP呼叫，本地主动挂断。
```

### 其它方法

#### 注册和注销账号

账号的手工注册和注销，一般使用中不需要此手工操作，本SDK会自动维护状态机，尽力保证已注册状态。

```js

register() // 话机账号的注册。

unregister() // 话机账号的注销。

```

#### 断开wss连接

在创建NxwCall的时候，会尝试自动连接wss服务器，此函数用于主动断开wss连接，如果账号已经注册，会先注销账号。可用用于切换账号或预关闭页面。

```js

disconnect() 

```

#### 播放提示音
```js

声明：play(action: string, filename?: string) 

示例：nxwcall.play('start','my-music.wav') //播放自定义的提示音

     nxwcall.play('end') //停止播放所有提示音
```
- action: start 或 end，启动播放和停止播放。
- filename 音频文件名。

**注意：当前页面在未曾鼠标键盘交互的情况下，页面后台放音会被浏览器自动静音**

#### 发送DTMF按键信息

在呼叫接通的情况下，发送DTMF信息，一般通过INFO消息，也可能通过RTP带内传输。

```js

声明：sendDTMF(tonestr: string)  

示例：nxwcall.sendDTMF('1'); //发生DTMF按键1

```

#### 关闭麦克风,本地静音

```js

muteCall(checked: boolean)

```

#### audio控件音，是否播放对方的声音

```js

silentCall(checked: boolean)

```

#### 设置本地放音音量

设置本地放音音量，参数volume表示最大音量的百分比，可选值为 [0,1]或者(1,100]，

```js

声明：setVolume(volume: number) 

示例：nxwcall.setVolume(0.8); //设置为最大音量的80%

     nxwcall.setVolume(80); //设置为最大音量的80%

```

### 自定义playTone
在发起通话开始，直到通话结束，NXCLOUD一共定义了5种默认提示音，如果你完全不修改提示音的播放机制，设置playTone=0xFF。如果你不想播放任何提示音，设置playTone=0x00。 如果你想完全播放自定义提示音，设置playTone=0x80。
以下是控制播放提示音的几个值：它们是'按位与'的关系：

值|含义
--|:--
0xFF | ALL: 播放全部
0x01 | RINGIN: 允许SDK自动播放振铃提示音
0x02 | RINGOUT: 允许SDK自动播放回铃提示音
0x04 | CONNECTED: 允许SDK自动播放接通提示音
0x08 | HANGUP: 允许SDK自动播放挂断提示音
0x10 | ONLINE: 允许SDK自动播放上线提示音
0x80 | CUSTOM: 允许SDK播放你的自定义语音文件，格式为wav或者mp3，单声道，8kHZ或16kHZ

#### NXCLOUD SDK默认的5个提示音：
通过 profile 的参数audioSrcPath指定提示音的目录

<h2 id='audiolist'></h2>

文件名|用途
--|:--
ringin.wav|振铃提示音
ringout.wav|回铃提示音
connected.wav|接通提示音
hangup.wav|挂断提示音
online.wav|上线提示音


## Nxwebrtc使用说明

### NxwAppConfig 
属性|类型|必选|说明
--|:--|:--|:--
nxuser|string|M|
nxpass|string|M|
nxtype|number|M|NX语音通话生产环境设置为6
audioElementId|string|M|播放对方声音的HTML组件id
playElementId|string|M|播放振铃、回铃、挂掉提示音的audio组件id
logLevel|LogLevel|M|debug:调试，warn:告警，error:错误
playTone|number|O|提示音播放控制
audioSrcPath|string|O|提示音wav文件路径，默认为audio
video|boolean|O|是否启用video，默认false
videoLocalElementId|string|O|本地视频video组件的id
videoRemoteElementId|string|O|远方视频video组件的id
autoAnswer|boolean|O|是否自动接通来电,默认false


### Nxwebrtc的状态
状态|说明|可能触发此状态的事件
--|:--|--
UA_INIT=0|初始状态|
UA_NXAPI|查询NXAPI，根据nxuser/nxpass验证|
UA_CONNECTING|尝试连接NX的wss服务器|
UA_CONNECTED|连接wss服务器成功|onServerConnect
UA_REGISTERING|尝试REGISTER注册中
UA_READY|注册成功，准备完成|onRegistered
UA_CALLING_OUT|外呼状态|onCallCreated
UA_INCOMING|呼入状态|onCallReceived
UA_TALKING|接通状态|onCallAnswered
UA_CALL_ENDING|呼叫结束中|
UA_CALL_END|呼叫完全结束|onCallHangup
UA_DISCONNECTED|从wss服务器断开|onServerDisconnect
UA_ERROR|SDK各种异常事件|error

<h2 id='eventlist'></h2>

### EventEmitter事件通知
Nxwebrtc封装了SIP底层协议栈的呼叫相关的事件，使用EventEmitter对象和业务交互，业务层可以注册回调函数。
Event|参数说明|说明
--|:--|:--
onCallCreated|RemoteNumber|发起呼叫创建成功，主叫模式
onCallAnswered|RemoteNumber|呼叫接通，主叫或被叫
onCallReceived|RemoteNumber|接收到呼叫请求，被叫模式
onCallHangup|RemoteNumber|挂断呼叫，主叫或被叫，对方先挂断
onServerConnect|-|连接到NX的wss服务器
onServerDisconnect|-|断开连接|到NX的wss服务器
onRegistered|-|注册成功
onUnregistered|-|注册失败，或注销成功，
onCallDTMFReceived|tone+":"+duration|DTMF按键信息和持续时间
onMessageReceived|message|MESSAGE消息的文本
error|Msg|异常事件的描述


### Nxwebrtc的属性
可以通过nxwebrtc对象的属性，和业务层的数据关联，在呼叫的CDR回调中有对应的数据。
参数|读写|说明
--|:--|:--
myState|R|只读当前NxwCall的状态机的状态
myOrderId|RW|发起呼叫之前设置此orderId信息
comingOrderId|RW|在呼入时的已经携带的orderId信息


## SDK 兼容性要求
### 浏览器要求
####  PC浏览器（win/linux/macOS）
- 手机浏览器（android/iOS）
- Chrome 56+
- Edge 79+
- Safari 16+
- Firefox 44+
- Opera 43+
- **不支持IE浏览器**

#### 手机浏览器（android/iOS）
- Chrome Android 105+
- Dafari iOS 11+
- UC Android 13.4+
- Android Browser 105+
- Firefox Android 104+
- QQ browser 13.1+
- Baidu Browser 13.18+
- KaiOS Browser 2.5+
