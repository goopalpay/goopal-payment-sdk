# 果仁支付商户服务端 SDK接入手册

![Alt text](1479807057877.png)


![Alt text](./1479871702604.png)


**修订历史**

| 日期 | 版本 | 修改说明 | 作者 |
|:-------- |:--------|:------- |:-------|
| 2016-10-18 | v1.0.0 | 创建文档 | 李宏宗|


-------------------

[TOC]

## 1 文档说明

### 1.1 目标

  本文档对果仁支付平台商户接口的使用进行详细说明，帮助商户熟悉了解果仁支付的交易流程，方便商户快捷、安全的接入果仁支付平台。

### 1.2 阅读对象

本文档的阅读对象是使用果付的商户技术人员，要求熟悉JAVA，PHP，JSP，ASP，ASP.net开发等其中一种开发语言。


###1.3  业务术语
| 术语   | 描述 | 
| :-------- | :--------|
| 请求  | 商户服务端以字符串形式把需要传输的数据发送给果付的过程 |
| 返回  | 果付系统以字符串形式直接把支付处理结果数据返回给商户系统。 |
| 通知  | 服务器异步通知。果付系统根据得到的数据处理完成后，果付的服务器主动发起通知给商户的服务器，同时携带处理完成的支付结果信息反馈给商户服务器。 |
| 查询服务 | 商户服务端通过查询服务可以查询该笔订单的支付结果，并根据结果做相应处理 |


### 1.4 接入流程
#### 1.4.1 商户接入准备
商户服务器需要依赖果付提供的SDK包完成，相关安全规范以及接口的开发。

#### 1.4.2 交易流程示意图

![Alt text](./1479809155337.png)

#### 1.4.3 交易流程
①　用户在商户APP/WEB商城下单，
②　商户服务器接收到下单请求，向果付支付系统下单。
③　果付支付系统验证数据
④　验证通过通知商户服务器，下单成功。
⑤　商户后台通知商户APP唤起果仁宝APP。
⑥　果仁宝APP验证用户的授权状态
⑦　果仁宝APP用户有效则会通过果仁宝向果付系统拉取订单信息
⑧　果付系统会检查用户下单信息的有效性。
⑨　有效订单，则给果仁宝用户展示订单信息。
⑩　用户输入支付密码。
11　果仁宝APP发送支付请求到果付系统，果付系统发送支付请求到银行/第三方
12　返回支付结果。
13　展示支付结果给用户。
14　银行/第三方发送异步通知给果付系统。
15　果付系统发送异步通知给商户服务器。

### 1.5 安全与开发规范

#### 1.5.1签名机制
为保证数据传输过程中的数据真实性、完整性和不可抵赖，我们需要对数据进行签名，在接收响应数据之后进行签名校验。
##### 1.5.1.1请求的签名与验签
首先，对于接口中的请求参数均采用表单形式提交。签名时对于所有以=分隔的参数对(即key，value)以字母顺序进行排序，然后以&作为连接符拼接成待签名串。再使用商户的私钥对待签名串做签名操作（签名时算法选择MD5）。最后，将签名后的签名串放在表单域里和其他表单域一起通过HTTP Post的方式传输给果付系统平台。
当使用MD5进行签名时，将商户的秘钥与待签名串拼接在一起（秘钥在前），并使用相应的charset进行编码，在进行计算MD5摘要。注意charset同样会作为参数出现在表单中。
以商户标准接口为例，接口中的请求参数将包含merchantId，service，version, requestTime, charset, signType, bizConten，sign字段。签名串为
bizContent=biz_content&charset=UTF-8&merchantId=21112345678&requestTime=2016-10-14+14%3A01%3A42&service=order.create& signType=MD5&version=1.0（注意字段之间无空格符）,计算得到的签名为B3DB73CD8CFC3C6E0671FAAAEE1B049（注意签名串为大写）
	当果付系统接收到请求参数时同样会从提交过来的表单中获取所有的参数并进行相同的签名方式, 并将计算得到的签名与表单中签名进行比对。
##### 1.5.1.2 响应的签名与验签
商户标准接口的响应均为json格式的，并且采用统一的格式：sign字段和response字段，sign字段为response的签名，response为具体的响应数据。签名字段为response后面括号内的所有内容(包括括号)。为了避免json反序列化出现的字段顺序不一致问题，response中的所有字段均按照字母顺序进行排序。下面是一个响应的例子，reponse中的字段code，data，msg是有序的。
```
{
  "response": {
    "code": "000000",
    "data": {
      "appId": "GAPP_59F0609B61D0F63E",
      "gopOrderNo": "MN787989390099980289",
    },
    "msg": "成功"
  }
   "sign": "E63D259F3C9B43E972B799997C812EAB"
}
```
#### 1.5.2 订单规范
**商户唯一订单号**：商户订单系统在果付平台使用唯一订单号标识。原则上要求商户订单号唯一标识此笔订单，与自然日无关。
**订单防重**：一个商户号、应用ID、商户订单号可以唯一确定一笔订单。如果商户号、应用ID有相同的订单号，第二次请求订单会被拒绝。
**订单结果查询**： 如果商户订单系统中用此订单唯一标识该笔订单则只需要：商户号+应用ID+商户唯一订单号查询该笔订单状态。

### 1.6 接口接入方式
**请求方式**：HTTPS POST请求,
**参数类型**：application/x-www-form-urlencoded
**请求地址**：https://pay.goopal.com.cn/gateway
## 2 标准商户接口

### 2.1 请求接口格式
标准接口都采用统一的请求格式，请求字段如下

| 名称  | 字段名称 | 数据类型 | 属性 | 备注 | 
| :---  | :-------| :------- |:--------|:--------|
| 商户ID  | merchantId |String|必填|  |
| 服务名称  | service |String|必填|根据不同的服务填写|
| 服务版本号  | version |String|必填|必填|
| 字符集 | charset |String|必填|签名时所使用的字符集|
| 签名类型 | signType|String|必填|MD5|
| 签名 | sign|String|必填|bizContent的签名|
|请求时间 | requestTime|String|必填|格式如2016-01-01 00:00:00|
| 业务数据 | bizContent|Integer|必填|具体的业务请求，json格式的|

### 2.2 接口相应格式
| 名称  | 字段名称 | 数据类型 | 属性 | 备注 | 
| :---  | :------- | :-------|:--------|:--------|
| 响应签名  | sign|String|必填| response的签名，商户可根据验证机制进行验证 |
| 响应数据  | response |String|必填|json格式|
| 状态码  | code|String|必填|返回结果的状态码，参考错误信息表|
| 状态信息 | msg |String|必填|返回结果状态信息，参考错误信息表|
| 返回业务数据 | data |String|必填|具体的业务响应数据，json格式|
请求实例：
POST https://pay.goopal.com.cn/gateway
merchantId=2111234567890832&service=order.create&version=1.0&
requestTime=2016-10-14+14%3A01%3A42&charset=UTF-8&signType=NON_SIGN&sign=B3DB73CD8CFC3C6E0671FAAAEE1B049&
bizContent=specfic_order_create_content

响应实例：
``` 
{
  "response": {
    "code": "000000",
    "data": {
      "appId": "GAPP_59F0609B61D0F63E",
      "gopOrderNo": "MN787989390099980289",
      "h5Url": "",
      "merchantId": "2111234567890832",
      "notifyUrl": "http://test.com/notify.jsp",
      "orderDate": "2016-10-17 20:11:25",
      "orderNo": "MN201610101049225131",
      "pageRetUrl": "http://test.com/redirect.jsp",
      "remark": "remark",
      "singleNo": "MN201610101049225131",
      "source": 1
    },
    "msg": "成功"
  }
   "sign": "E63D259F3C9B43E972B799997C812EAB"
}
```

### 2.3 下单接口
支付请求是交易请求方（如商家）与果付支付平台之间的交易信息通过用户浏览器进行传递的交易，是一种异步的、需要用户参与完成的交易方式。果付支付系统会给商家发送页面通知以及后台通知，商家方也必须实现接收后台通知。
如果是手机端用户需要用户发起下单之后，商户拿果付提供给用户的有效凭证来查询订单，然后发起快捷支付请求。
#### 2.3.1 公共请求字段
| 名称  | 字段名称 | 数据类型 | 属性 | 备注 | 
| :---  | :-------| :-------|:--------|:--------|
| 商户ID  | merchantId|String|必填|   |
| 服务名称  | service|String|必填|固定为order.create|
| 服务版本号  | version|String|必填|固定为1.0|
| 字符集 | charset |String|必填|UTF-8|
| 签名类型 | signType|String|必填|MD5|
| 签名 | sign|String|必填|bizContent的签名|
| 业务数据 | bizContent|String|必填|下单请求，json格式，具体字段参考接口字段|
#### 2.3.2 接口请求字段
| 名称  | 字段名称 | 数据类型 | 属性 | 备注 | 
| :---  | :-------| :-------|:--------|:--------|
| 商户ID  | merchantId|String|必填|   |
| 应用ID  | appId|String|必填|固定为order.create|
| 商户订单号  | orderNo|String|必填|商户唯一的订单号|
| 交易币种 | tradeCurrency |String|必填|交易币种，CNY|
| 结算币种 | settleCurrency |String|必填|期望结算币种， CNY|
| 订单金额 | orderAmount |Long|必填|订单金额，CNY的单位为分|
| 商品名称 | goodsName|String|可选| |
| 商品描述 | goodsDescription |String|可选||
| 订单有效时间 | validityPeriods|Integer|可选|单位为分钟，超过则订单自动过期|
| 下单来源 | source|Integer|必填|1：APP支付 2：H5支付|
| 返回URL | pageRetUrl|String|可选|下单成功商户跳转地址|
| 通知URL | notifyUrl|String|必填|支付成功回调通知地址|
| 备注 | remark|String|可选||
#### 2.3.3 响应接口字段
| 名称  | 字段名称 | 数据类型 | 属性 | 备注 | 
| :---  | :-------| :-------|:--------|:--------|
| 商户ID  | merchantId|String|必填|   |
| 应用ID  | appId|String|必填| |
| 商户订单号  | orderNo|String|必填|商户唯一的订单号|
| 果付订单号 | gopOrderNo|String|必填| |
| 下单时间 | orderDate|String|必填|格式如 2016-09-29 15:24:07|
| 唯一表示码 | singleNo|String|必填| |
| 支付类型 | source|Integer|必填| 1：APP支付 2：H5支付 |
| H5地址 | h5Url|String|可选|当订单来源为H5时指定|
| 返回URL | pageRetUrl|String|可选|下单成功商户跳转地址，与请求中字段一样|
| 通知URL | notifyUrl|String|必填|支付成功回调通知地址， 与请求字段一样|
| 备注 | remark|String|可选||

###2.4 订单查询接口
#### 2.4.1 接口字段
| 名称  | 字段名称 | 数据类型 | 属性 | 备注 | 
| :---  | :-------| :-------|:--------|:--------|
| 商户ID  | merchantId|String|必填|   |
| 应用ID  | appId|String|必填| |
| 商户订单号  | orderNo|String|必填|商户唯一的订单号|

 响应
 
| 名称  | 字段名称 | 数据类型 | 属性 | 备注 | 
| :---  | :-------| :-------|:--------|:--------|
| 商户ID  | merchantId|String|必填|   |
| 应用ID  | appId|String|必填| |
| 商户订单号  | orderNo|String|必填|商户唯一的订单号|
| 果付订单号  | gopOrderNo|String|必填| |
| 交易币种  | tradeCurrency|String|必填|交易币种，CNY|
| 结算币种  | settleCurrency|String|必填|期望结算币种， CNY|
| 订单金额  | settleCurrency|Long|必填|CNY的单位为分|
| 可退款金额  | availableRefundAmount|Long|必填|CNY的单位为分|
| 手续费  | feeAmount|Long|必填|CNY的单位为分|
| 是否有退款  | isRefund|Bool|必填|商户唯一的订单号|
| 下单时间  | orderDate|String|必填|格式如 2016-09-29 15:24:07|
| 订单过期时间  | orderExpireTime|String|必填|订单若超过期限仍未支付则自动取消, 格式如 2016-09-29 15:24:07|
| 商品名称  | goodsName|String|必填||
| 商品描述  | goodsDescription|String|必填||
| 备注  | remark|String|必填||
| 订单来源  | source|Integer|必填|1：APP支付2：H5支付|
| 支付结果  | payResult|String|必填|PENDING：处理中；SUCCESS：支付成功；FAIL：支付失败；EXPIRED:订单过期|

###2.5 退款接口
###2.5.1 接口字段
| 名称  | 字段名称 | 数据类型 | 属性 | 备注 | 
| :---  | :-------| :-------|:--------|:--------|
| 商户ID  | merchantId|String|必填|   |
| 应用ID  | appId|String|必填| |
| 商户退款订单号  | refundOrderNo|String|必填||
| 商户订单号  | orderNo|String|必填|商户唯一的订单号|
| 币种  | currency|String|必填|交易币种，CNY|
| 退款金额  | refundAmount|Long|必填| |
| 备注  | remark|String|可选| |

 响应
 
| 名称  | 字段名称 | 数据类型 | 属性 | 备注 | 
| :---  |:-------|:-------|:--------|:--------|
| 商户ID  | merchantId|String|必填|   |
| 应用ID  | appId|String|必填| |
| 商户退款订单号  | refundOrderNo|String|必填||
| 商户订单号  | orderNo|String|必填|商户唯一的订单号|
| 币种  | currency|String|必填|交易币种，CNY|
| 退款金额  | refundAmount|Long|必填| |
| 果付退款订单号  | gopRefundNo|String|必填| |

###2.6 退款订单查询接口
####2.6.1 接口字段

请求

| 名称  | 字段名称 | 数据类型 | 属性 | 备注 | 
| :---  | :-------| :-------|:--------|:--------|
| 商户ID  | merchantId|String|必填|   |
| 应用ID  | appId|String|必填| |
| 商户退款订单号  | refundOrderNo|String|必填||

  响应

| 名称  | 字段名称 | 数据类型 | 属性 | 备注 | 
| :---  |:-------|:-------|:--------|:--------|
| 商户ID  | merchantId|String|必填|   |
| 应用ID  | appId|String|必填| |
| 商户退款订单号  | refundOrderNo|String|必填||
| 商户订单号  | orderNo|String|必填| 商户唯一的订单号  |
| 币种  | currency|String|必填| CNY|
| 退款金额  | refundAmount|Long|必填|格式如 2016-09-29 15:24:07|
| 退款时间  | refundTime|String|必填|   |
| 果付退款订单号  | gopRefundNo|String|必填| |
| 退款结果  | refundResult|String|必填|PENDING:退款处理中；SUCCESS:退款成功；FAIL:退款失败|
| 备注  | remark|String|必填||
###2.7 商户通知接口
当订单上发生某些事件时，将会通知商户。商户通过设置下单时的notifyUrl字段来指定接收通知的地址。
#### 2.7.1 通知类型
目前主要有两种通知类型：
1）**交易成功的通知**：用户支付成功时发起该通知。
2）**退款成功的通知**：退款成功时发起该通知。商户可以通过两种途径发起退款：通过登录商户后台发起或者通过接口发起。当通过登录商户后台发起时，商户退款订单号由商户后台生成，接收到退款通知时商户需要根据通知内容进行落单并进行相应处理。
#### 2.7.2 通知接口格式
**请求方式**：HTTP POST
**参数类型**：application/x-www-form-urlencoded
####2.7.3 字段名称
所有类型的通知都使用相同格式的数据结构，改数结构中的字段将作为参数以上述形式传递给通知接口。通知接口数据格式如下

| 名称  | 字段名称 | 数据类型 | 属性 | 备注 | 
| :---  |:-------|:-------|:--------|:--------|
| 商户ID  | version|String|必填|   |
| 通知时间  | notifyTime|String|必填|格式如：2016-01-01 12:00:00 |
| 通知标识  | notifyToken|String|必填|用于标识通知的唯一性|
| 通知类型  | notifyType|String|必填| 参考具体的通知|
| 签名类型  | signType|String|必填|MD5|
| 字符集  | charset|String|必填|MD5签名时用的字符集，一般为UTF-8|
| 签名  | sign|String|必填|   |
| 通知内容  | notifyContent|String|必填| 根据通知类型进行解析 |

>**签名机制**：签名机制与前面接口签名机制相同

**实例**：
POST /merchant/notify/url
merchantId=2111234567890832&notifyToken= 5d41402abc4b2a76b9719d911017c592&notifyTime=2016-09-29+13%3A27%3A00&notifyType=REFUND_SUCCESS&signType=MD5 &sign=yoursign&charset=UTF-8&notifyContent=somenotifyconent

**交易成功通知**
通用接口字段

| 名称  | 字段名称 | 数据类型 | 属性 | 备注 | 
| :---  |:-------|:-------|:--------|:--------|
| 通知类型  | notifyType|String|必填| |
| 通知内容 | notifyContent|String|必填| |
| 商户ID  | merchantId|String|必填| |
| 应用ID  | appId|String|必填| |
| 商户订单号  | orderNo|String|必填|商户唯一的订单号|
| 果付订单号  | gopOrderNo|String|必填||
| 交易币种  | tradeCurrency|String|必填| 交易币种，CNY  |
| 订单金额  | orderAmount|Long|必填| 订单金额，CNY的单位为分 |
| 订单日期  | orderDate|String|必填|格式如：2016-01-01 12:00:00|
| 支付成功时间  | paySuccessTime|String|必填| 格式如：2016-01-01 12:00:00|
| 订单来源  | source|Integer|必填|1：APP支付2：H5支付|
| 商品名称  | goodsName|String|可选||
| 商品描述  | goodsDescription|String|可选|   |
| 备注  | remark|String|可选|  |
**退款成功通知**

通用接口字段

| 名称  | 字段名称 | 数据类型 | 属性 | 备注 | 
| :---  |:-------|:-------|:--------|:--------|
| 通知类型  | notifyType|String|必填| |
| 通知内容 | notifyContent|String|必填| |

通用内容字段

| 名称  | 字段名称 | 数据类型 | 属性 | 备注 | 
| :---  | :-------| :-------|:--------|:--------|
| 商户ID  | merchantId|String|必填| |
| 应用ID | appId|String|必填| |
| 商户退款订单号  | refundOrderNo|String|必填| |
| 商户订单号 | orderNo|String|必填| 商户唯一的订单号 |
| 币种  | currency|String|必填| CNY |
| 退款金额 | refundAmount|Long|必填| 货币为CNY时以分为单位 |
| 果付退款订单号  | gopRefundNo|String|必填| |
| 退款成功时间 | refundSuccessTime|String|必填| 格式如：2016-01-01 12:00:00 |
| 备注  | remark|String|可选| |

##3附录
###3.1错误码

| 错误码  | 错误信息 |
| :---  | :-------| 
| 000000  | 成功 |
| 700001  | 参数错误 |
| 800001  | 不支持的服务 |
| 800002  | 不支持的签名类型 |
| 800003  | 商户ID不能为空 |
| 800004  | 商户不存在 |
| 800005  | 商户状态异常 |
| 800006  | 验签失败 |
| 800007  | 无效请求 |
| 800008  | 应用ID不能为空 |
| 800009  | 商户订单号不能为空 |
| 800010  | 交易来源不能为空 |
| 800011  | 无效的交易来源 |
| 800012  | 交易币种不能为空 |
| 800013  | 无效交易币种 |
| 800014  | 结算币种不能为空 |
| 800015  | 无效的结算币种 |
| 800016  | 商品名称不能为空 |
| 800017  | 订单有效期不能为空 |
| 800018  | 无效的订单有效期 |
| 800019  | 订单金额不能为空 |
| 800020  | 无效订单金额 |
| 800021  | 通知URL不能为空 |
| 800022  | 应用不存在 |
| 800023  | 应用状态异常 |
| 800024  | 交易重复 |
| 800025  | 订单不存在 |
| 800026  | 商户退款订单号不能为空 |
| 800027  | 退款金额不能为空 |
| 800028  | 无效退款金额 |
| 800029  | 订单不允许重复退款 |
| 800030  | 该订单不允许退款 |
| 999999  | 系统繁忙, 请稍后再试 |

###3.2对账文件格式
与果仁支付技术对接时获取
###3.3生产环境
与果仁支付技术对接时获取
###3.4 测试环境
与果仁支付技术对接时获取


