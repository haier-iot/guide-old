#  介绍

## 简介
	
>  海尔账号的注册登录服务由海尔集团690用户中心提供。	


## 注册成为开发者  

>  请联系用户中心相关人员，获取开发者权限。

## 申请应用

>  联系相关人员，申请应用通过后，获得测试环境和正式环境的client_id与client_secret，前者为应用Id，后者为应用密钥。测试环境和正式环境的client_id与client_secret不会相同。  



## 前提

1.接口中需要的client_id和client_secret由用户中心授权下发，根据接入应用的不同进行不同的授权。<br/>
2.文档提供的接口为海尔品牌接入接口，卡萨帝接口及GE品牌接口，分别随品牌不同而区分不同调用域名。<br/>
3.文档提供的接口大多受到access_token保护，接口分为应用级token保护和个人级token保护：应用级
保护大多发生于登录动作之前，token由以下获取应用级保护的access_token接口获取；个人级token保护大多发生于登录动作之后，token
由登录等动作接口返回。受保护的接口在被调用时，调用方需要在HTTP请求的Header中新增Authorization,
值为Bearer[access_token]，后面接口将直接备注受应用级还是设备级保护，请调用方自行甄别。<br/>
4.以下所有接口的正常Response下的HTTP Status Code为200，后无特殊情况不再说明。<br/>