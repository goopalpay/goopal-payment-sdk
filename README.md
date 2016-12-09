# 果仁支付商户服务端 SDK接入手册-v1.0.0

## 1 文档说明

### 1.1 目标

> 本文档对果仁支付平台(以下简称：果付)商户接口的使用进行详细说明，帮助商户方便、快捷和安全的接入。

### 1.2 阅读对象

> 本文档的阅读对象是使用果付的商户技术人员，要求熟悉JAVA，PHP，JSP，ASP，ASP.net开发等其中一种开发语言。


###1.3  业务术语

| 术语   | 描述 | 
| :-------- | :--------|
| 请求  | 商户服务端以字符串形式把需要传输的数据发送给果付的过程 |
| 返回  | 果付系统以字符串形式直接把支付处理结果数据返回给商户系统。 |
| 通知  | 服务器异步通知。果付系统根据得到的数据处理完成后，果付的服务器主动发起通知给商户的服务器，同时携带处理完成的支付结果信息反馈给商户服务器。 |
| 查询服务 | 商户服务端通过查询服务可以查询该笔订单的支付结果，并根据结果做相应处理 |


<br>
### 1.4 接入

#### 1.4.1 接入准备

> 商户需要下载相应的SDK，完成相关安全规范以及接口的开发。

#### 1.4.2 交易流程示意图

![交易流程图](https://github.com/goopalpay/goopal-payment-sdk/blob/master/images/tradepayflow.png)


#### 1.4.3 交易流程
1) 用户在商户APP/H5选择果仁支付<br>
2) 商户后台向果付后台发起下单请求<br>
3) 果付后台返回订单信息<br>
4) 商户后台返回订单信息到商户APP/H5，并展示给用户<br>
5) 商户APP/H5跳转到果仁宝APP/H5<br>
6) 果仁宝APP/H5发起预支付请求<br>
7) 果付后台返回支付信息<br>
8) 果仁宝APP/H5发起支付请求<br>
9) 果付后台返回支付结果<br>
10) 果付后台异步通知商户后台订单结果

<br>

### 1.5 安全与开发规范

#### 1.5.1订单规范
> **商户唯一订单号**：商户订单系统在果付平台使用唯一订单号标识。原则上要求商户订单号唯一标识此笔订单，与自然日无关。<br>
**订单防重**：一个商户号、应用ID、商户订单号可以唯一确定一笔订单。如果商户号、应用ID有相同的订单号，第二次请求订单会被拒绝。<br>
**订单结果查询**： 如果商户订单系统中用此订单唯一标识该笔订单则只需要：商户号+应用ID+商户唯一订单号查询该笔订单状态。<br>

#### 1.5.2签名机制
为保证数据传输过程中的数据真实性、完整性和不可抵赖，需要对数据进行签名，在接收响应数据之后进行签名校验。<br>

##### MD5签名
> 请求签名：<br>
1）接口中的请求参数均采用表单形式提交;<br>
2）签名时对于所有以`=`分隔的参数对(即key，value)以字母顺序进行排序，并以`&`作为连接符拼接成待签名串;<br>
3）用商户的私钥对待签名串做签名操作，签名时算法选择MD5。具体做法为将商户的秘钥与待签名串拼接在一起（秘钥在前)，并使用相应的`charset`进行编码，计算编码后的数据`MD5`摘要作为签名;<br>
4）将签名后的签名串放在表单域里和其他表单域一起通过`HTTP Post`的方式传输给果付系统平台。<br>

示例：<br>
请求参数包含`merchantId`，`service`，`version`, `requestTime`, `charset`, `signType`, `bizConten`，`sign`字段。签名串为<br>

``` 
bizContent=biz_content&charset=UTF-8&merchantId=21112345678&
requestTime=2016-10-14+14%3A01%3A42&service=order.create& 
signType=MD5&version=1.0

```

**注意:**字段之间无空格符<br>
计算得到的签名为`B3DB73CD8CFC3C6E0671FAAAEE1B049`（注意签名串为大写）<br>

请求签名配置(java示例)：<br>

``` java
// 使用时只需要配置签名类型和私钥即可
SignScheme signScheme = new SignScheme(
        SignType.MD5, // 签名类型
        "db87b447cf84bf2a2eaeac5cb587852c"); // 秘钥

```

<br>
> 响应签名：<br>
1）商户标准接口的响应均为`json格式`的，采用统一的格式：`sign`字段和`response`字段;<br>
2）签名字段为`response`后面括号内的所有内容(包括括号)。为了避免`json`反序列化出现的字段顺序不一致问题，`response`中的所有字段均按照字母顺序进行排序。<br>

响应示例：`reponse`中的字段`code`，`data`，`msg`是有序的。

``` java
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


<br>

## 2 标准商户接口
> **请求方式**：`HTTPS POST`<br>
**参数类型**：`application/x-www-form-urlencoded`<br>
**请求地址**：https://pay.guorenbao.com/gateway<br>

<br>

### 2.1 公共请求参数

标准接口都采用统一的请求格式，格式如下：

| 名称  | 字段名称 | 数据类型 | 属性 | 备注 | 
| :---  |:-------|:-------|:--------|:--------|
| 商户ID  | merchantId |String|必填|   |
| 服务名称  | service |String|必填|根据不同的服务填写|
| 服务版本号  | version |String|必填|必填|
| 字符集 | charset |String|必填|签名时所使用的字符集|
| 签名类型 | signType|String|必填|MD5|
| 签名 | sign|String|必填|bizContent的签名|
| 请求时间 | requestTime|String|必填|格式如2016-01-01 00:00:00|
| 业务数据 | bizContent|Integer|必填|具体的业务请求，json格式的|
<br>

### 2.2 公共响应参数

| 名称  | 字段名称 | 数据类型 | 属性 | 备注 | 
| :---  |:-------|:-------|:--------|:--------|
| 响应签名  | sign|String|必填| response的签名，商户可根据验证机制进行验证 |
| 响应数据  | response |String|必填|json格式|
| 状态码  | code|String|必填|返回结果的状态码，参考错误信息表|
| 状态信息 | msg |String|必填|返回结果状态信息，参考错误信息表|
| 返回业务数据 | data |String|必填|具体的业务响应，json格式|
<br>

### 2.3 order.create（下单接口）
> 商户通过该接口进行交易的创建下单

#### 2.3.1 公共参数说明
| 名称  | 字段名称 | 数据类型 | 属性 | 备注 | 
| :---  |:-------|:-------|:--------|:--------|
| 服务名称  | service |String|必填|order.create|
#### 2.3.2 请求参数

| 名称  | 字段名称 | 数据类型 | 属性 | 备注 | 
| :---  |:-------|:-------|:--------|:--------|
| 商户ID  | merchantId|String|必填|   |
| 应用ID  | appId|String|必填||
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

#### 2.3.3 响应参数

| 名称  | 字段名称 | 数据类型 | 属性 | 备注 | 
| :---  | :-------|:-------|:--------|:--------|
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


<br>
HTTP请求源码：

```http
POST https://pay.guorenbao.com/gateway
requestTime=2016-12-06+11%3A31%3A30&charset=UTF-8&merchantId=2111234567890832&
service=order.create&bizContent=%7B%22appId%22%3A%22GAPP_59F0609B61D0F63E%22%2C
%22goodsDescription%22%3A%22%E6%B5%8B%E8%AF%95%E5%95%86%E5%93%81%E6%8F%8F%E8%BF
%B0%22%2C%22goodsName%22%3A%22%E6%B5%8B%E8%AF%95%E5%95%86%E5%93%81%E5%90%8D
%22%2C%22merchantId%22%3A%222111234567890832%22%2C%22notifyUrl%22%3A%22http%3A
%2F%2Ftest.com%2Fredirect.jsp%22%2C%22orderAmount%22%3A10000%2C%22orderNo%22%3A
%22NO20160929144922599%22%2C%22pageRetUrl%22%3A%22http%3A%2F%2Ftest.com
%2Fnotify.jsp%22%2C%22remark%22%3A%22%E4%B8%8B%E5%8D%95%E5%A4%87%E6%B3%A8%22%2C
%22settleCurrency%22%3A%22CNY%22%2C%22source%22%3A1%2C%22tradeCurrency%22%3A
%22CNY%22%2C%22validityPeriods%22%3A10%7D&sign=685B1E63EED570D4887FFB7105A477F3&
signType=MD5&version=1.0
```

java请求：

```java
SignScheme signScheme = new SignScheme(
            SignType.MD5, // 签名类型
            "db87b447cf84bf2a2eaeac5cb587852c"); // 秘钥

    GopPayClient client = new DefaultGopPayClient(
            "https://pay.guorenbao.com/gateway", // 请求地址
            "2111234567890452", // 商户号
            signScheme); // 签名方案
    GopPayOrderCreateRequest request = new GopPayOrderCreateRequest();
    request.setMerchantId("2111234567890832");
    request.setAppId("GAPP_59F0609B61D0F63E");
    request.setOrderNo("NO20160929144922599");
    request.setTradeCurrency("CNY");
    request.setSettleCurrency("CNY");
    request.setOrderAmount(10000L); // 订单金额 单位为分,100元
    request.setGoodsName("测试商品名");
    request.setGoodsDescription("测试商品描述");
    request.setValidityPeriods(10); // 过期时间 单位为分钟, 10分钟后订单过期
    request.setSource(1); // 1 app支付 2 H5 支付
    request.setPageRetUrl("http://test.com/redirect.jsp");
    request.setNotifyUrl("http://test.com/notify.jsp");
    request.setRemark("下单备注");
    GopPayOrderCreateResponse response = client.execute(request);
    System.out.println(response);
```

响应示例(成功)：

```json
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
      "remark": "下单备注",
      "singleNo": "MN201610101049225131",
      "source": 1
    },
    "msg": "成功"
  }
   "sign": "E63D259F3C9B43E972B799997C812EAB"
}
```


响应实例(失败)：

```json
{"response":{
  "code":"800006",
  "msg":"验签失败"
  },
"sign":"E63D259F3C9B43E972B799997C812EAB"}
```

详细失败信息请参照：[附录-错误码](#codes)

<br>
###2.4 order.query（订单查询接口）
>该接口主要提供订单信息查询

#### 2.4.1 公共参数说明
| 名称  | 字段名称 | 数据类型 | 属性 | 备注 | 
| :---  |:-------|:-------|:--------|:--------|
| 服务名称  | service |String|必填|order.query|

#### 2.4.2 请求参数

| 名称  | 字段名称 | 数据类型 | 属性 | 备注 | 
| :---  |:-------|:-------|:--------|:--------|
| 商户ID  | merchantId|String|必填|   |
| 应用ID  | appId|String|必填| |
| 商户订单号  | orderNo|String|必填|商户唯一的订单号|

#### 2.4.3 响应参数

| 名称  | 字段名称 | 数据类型 | 属性 | 备注 | 
| :---  | :-------|:-------|:--------|:--------|
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


<br>
HTTP请求源码：

```http
POST https://pay.guorenbao.com/gateway
requestTime=2016-12-06+13%3A42%3A50&charset=UTF-8&merchantId=2111234567890452&
service=order.query&bizContent=%7B%22appId%22%3A%22GAPP_BB049EC805CD32E8%22%2C
%22merchantId%22%3A%222111234567890452%22%2C%22orderNo%22%3A
%22MN20160929144922599%22%7D&sign=852B07352D60666E8BEDD13E8F653D16&
signType=MD5&version=1.0
```

java请求：

```java
SignScheme signScheme = new SignScheme(
        SignType.MD5, // 签名类型
        "db87b447cf84bf2a2eaeac5cb587852c"); // 秘钥

    GopPayClient client = new DefaultGopPayClient(
        "https://pay.guorenbao.com/gateway", // 请求地址
        "2111234567890452", // 商户号
        signScheme); // 签名方案

    GopPayOrderQueryRequest request = new GopPayOrderQueryRequest();
    request.setMerchantId("2111234567890452");
    request.setAppId("GAPP_BB049EC805CD32E8");
    request.setOrderNo("MN20160929144922599");
    GopPayOrderQueryResponse response = client.execute(request);
    System.out.println(response);
```

响应示例(成功)：

```json
{"response":{
   "code":"000000",
   "data":{
      "appId":"GAPP_BB049EC805CD32E8",
      "availableRefundAmount":100,
      "feeAmount":2,
      "goodsDescription":"",
      "goodsName":"test-goods",
      "gopOrderNo":"MN781394105676570625",
      "isRefund":false,
      "merchantId":"2111234567890452",
      "orderAmount":100,
      "orderDate":"2016-09-29 15:24:07",
      "orderExpireTime":"2016-09-29 15:29:07",
      "orderNo":"MN20160929144922599",
      "payResult":"EXPIRED",
      "remark":"test",
      "settleCurrency":"CNY",
      "source":1,
      "tradeCurrency":"CNY"},
  "msg":"成功"},
"sign":"3F6CF7921334BE0708314E8A9F525F35"}
```

<br>

###2.5 order.refund（退款接口）
>商户通过该接口交易成功的订单进行退款申请

#### 2.5.1 公共参数说明
| 名称  | 字段名称 | 数据类型 | 属性 | 备注 | 
| :---  |:-------|:-------|:--------|:--------|
| 服务名称  | service |String|必填|order.refund|

####2.5.2 请求参数

| 名称  | 字段名称 | 数据类型 | 属性 | 备注 | 
| :---  | :-------| :-------|:--------|:--------|
| 商户ID  | merchantId|String|必填|   |
| 应用ID  | appId|String|必填| |
| 商户退款订单号  | refundOrderNo|String|必填||
| 商户订单号  | orderNo|String|必填|商户唯一的订单号|
| 币种  | currency|String|必填|交易币种，CNY|
| 退款金额  | refundAmount|Long|必填| |
| 备注  | remark|String|可选| |

####2.5.3 响应参数

| 名称  | 字段名称 | 数据类型 | 属性 | 备注 | 
| :---  |:-------|:-------|:--------|:--------|
| 商户ID  | merchantId|String|必填|   |
| 应用ID  | appId|String|必填| |
| 商户退款订单号  | refundOrderNo|String|必填||
| 商户订单号  | orderNo|String|必填|商户唯一的订单号|
| 币种  | currency|String|必填|交易币种，CNY|
| 退款金额  | refundAmount|Long|必填| |
| 果付退款订单号  | gopRefundNo|String|必填| |

HTTP请求源码：

```http
POST https://pay.guorenbao.com/gateway
requestTime=2016-12-08+09%3A40%3A58&charset=UTF-8&merchantId=2111234567890832
&service=order.refund&bizContent=%7B%22appId%22%3A%22GAPP_59F0609B61D0F63E%22%
2C%22currency%22%3A%22CNY%22%2C%22merchantId%22%3A%222111234567890832%22%2C%22
orderNo%22%3A%22MK796956059575103489%22%2C%22refundAmount%22%3A1000%2C%22
refundOrderNo%22%3A%22RNO20160929144922599%22%2C%22remark%22%3A%22%E9%80%80%E6%
AC%BE%E5%A4%87%E6%B3%A8%22%7D&sign=3CEA88BF09D17171440C03ED2E73972F&
signType=MD5&version=1.0
```

java请求：

```java
   SignScheme signScheme = new SignScheme(
        SignType.MD5, // 签名类型
        "b0651984a70b5ffc3e2befad9387852c"); // 秘钥

   GopPayClient client = new DefaultGopPayClient(
        "https://pay.guorenbao.com/gateway", // 请求地址
        "2111234567890812", // 商户号
        signScheme); // 签名方案
   GopPayOrderRefundRequest request = new GopPayOrderRefundRequest();
   request.setMerchantId("2111234567890812");
   request.setAppId("GAPP_59F0609B61D0F63E");
   request.setRefundOrderNo("RNO20160929144922599");
   request.setOrderNo("MK796956059575103489");
   request.setCurrency("CNY");
   request.setRefundAmount(1000L); // 订单金额 单位为分,10元
   request.setRemark("退款备注");
   GopPayOrderRefundResponse response = client.execute(request);
   System.out.println(response);
```

响应示例(成功)：

```json
{"response":{
   "code":"000000",
   "data":{
      "appId":"GAPP_59F0609B61D0F63E",
      "currency":"CNY",
      "gopRefundNo":"RF806674862122840065",
      "merchantId":"2111234567890832",
      "orderNo":"MK796956059575103489",
      "refundAmount":1000,
      "refundOrderNo":"RNO20160929144922599"},
   "msg":"成功"},
"sign":"BF456A8EA61341E3C84AE96CA1869600"}
```

<br>

###2.6 refund.query（退款订单查询接口）
>该接口主要提供退款订单信息查询

#### 2.6.1 公共参数说明
| 名称  | 字段名称 | 数据类型 | 属性 | 备注 | 
| :---  |:-------|:-------|:--------|:--------|
| 服务名称  | service |String|必填|order.refund|
#### 2.6.2 请求参数

| 名称  | 字段名称 | 数据类型 | 属性 | 备注 | 
| :---  |:-------|:-------|:--------|:--------|
| 商户ID  | merchantId|String|必填|   |
| 应用ID  | appId|String|必填| |
| 商户退款订单号  | refundOrderNo|String|必填||

#### 2.6.3 响应参数

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

<br>
HTTP请求源码：

```http
POST https://pay.guorenbao.com/gateway
requestTime=2016-12-08+10%3A08%3A58&charset=UTF-8&merchantId=2111234567890832&
service=refund.query&bizContent=%7B%22appId%22%3A%22GAPP_59F0609B61D0F63E%22%2C
%22merchantId%22%3A%222111234567890832%22%2C%22refundOrderNo%22%3A%22RNO20160929
144922599%22%7D&sign=7FA90CCA02E28DE43033435D96BE72ED&signType=MD5&version=1.0
```

java请求：

```java
 SignScheme signScheme = new SignScheme(
        SignType.MD5, // 签名类型
        "b0651984a70b5ffc3e2befad93c380d3"); // 秘钥

  GopPayClient client = new DefaultGopPayClient(
        "http://172.16.33.8:9012/pay-trade/gateway", // 请求地址
        "2111234567890832", // 商户号
        signScheme); // 签名方案
  GopPayRefundQueryRequest request = new GopPayRefundQueryRequest();
  request.setMerchantId("2111234567890832");
  request.setAppId("GAPP_59F0609B61D0F63E");
  request.setRefundOrderNo("RNO20160929144922599");
  GopPayRefundQueryResponse response = client.execute(request);
  System.out.println(response);

```

响应示例(成功)：

```json
{"response":{
   "code":"000000",
   "data":{
      "appId":"GAPP_59F0609B61D0F63E",
      "currency":"CNY",
      "gopRefundNo":"RF806674862122840065",
      "merchantId":"2111234567890832",
      "orderNo":"MK796956059575103489",
      "refundAmount":980,
      "refundOrderNo":"RNO20160929144922599",
      "refundResult":"SUCCESS",
      "refundTime":"2016-12-08 09:40:48",
      "remark":"退款备注"},
   "msg":"成功"},
 "sign":"3C9B18BBAD4517988EC95D896AE696A7"}
```

<br>

###2.7 商户通知接口
> 当订单上发生某些事件时，将会通知商户。商户通过设置下单时的`notifyUrl`字段来指定接收通知的地址。<br>
> 目前主要有两种通知类型：**交易成功通知**，**退款成功通知**
>
**请求方式**：`HTTP POST`<br>
**参数类型**：`application/x-www-form-urlencoded`<br>

####2.7.1 公共参数
> 所有类型的通知都使用相同格式的数据结构，改数结构中的字段将作为参数以上述形式传递给通知接口。通知接口数据格式如下：

| 名称  | 字段名称 | 数据类型 | 属性 | 备注 | 
| :---  |:-------|:-------|:--------|:--------|
| 商户ID  | version|String|必填|   |
| 通知时间  | notifyTime|String|必填|格式如：2016-01-01 12:00:00 |
| 通知标识  | notifyToken|String|必填|用于标识通知的唯一性|
| 签名类型  | signType|String|必填|MD5|
| 字符集  | charset|String|必填|MD5签名时用的字符集，一般为UTF-8|
| 签名  | sign|String|必填|   |
| 通知类型  | notifyType|String|必填| 参考具体的通知|
| 通知内容  | notifyContent|String|必填| 根据通知类型进行解析 |

**签名机制**：签名机制与前面接口签名机制相同

#### 2.7.2 交易成功通知
>用户支付成功时发起该通知

公共参数说明

| 名称  | 字段名称 | 数据类型 | 属性 | 备注 | 
| :---  |:-------|:-------|:--------|:--------|
| 通知类型  | notifyType|String|必填|TRADE_SUCCESS|
| 通知内容 | notifyContent|String|必填|参考通知内容字段|

通知内容字段

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

**实例**：

```
POST /merchant/notify/url
merchantId=2111234567890832&notifyToken=5d41402abc4b2a76b9719d911017c592&notify
Time=2016-09-29+13%3A27%3A00&notifyType=TRADE_SUCCESS&signType=MD5
&sign=yoursign&charset=UTF-8&notifyContent=somenotifyconent
```


#### 2.7.3 退款成功通知
>商户可以通过两种途径发起退款：通过登录商户后台发起或者通过接口发起，退款成功时发起该通知。
>**注意:**当通过登录商户后台发起时，商户退款订单号由商户后台生成

公共参数说明

| 名称  | 字段名称 | 数据类型 | 属性 | 备注 | 
| :---  |:-------|:-------|:--------|:--------|
| 通知类型  | notifyType|String|必填|REFUND_SUCCESS |
| 通知内容 | notifyContent|String|必填|参考通知内容字段|

通知内容字段

| 名称  | 字段名称 | 数据类型 | 属性 | 备注 | 
| :---  | :-------|:-------|:--------|:--------|
| 商户ID  | merchantId|String|必填| |
| 应用ID | appId|String|必填| |
| 商户退款订单号  | refundOrderNo|String|必填| |
| 商户订单号 | orderNo|String|必填| 商户唯一的订单号 |
| 币种  | currency|String|必填| CNY |
| 退款金额 | refundAmount|Long|必填| 货币为CNY时以分为单位 |
| 果付退款订单号  | gopRefundNo|String|必填| |
| 退款成功时间 | refundSuccessTime|String|必填| 格式如：2016-01-01 12:00:00 |
| 备注  | remark|String|可选| |

**实例**：

``` 
POST /merchant/notify/url
merchantId=2111234567890832&notifyToken=5d41402abc4b2a76b9719d911017c592&notify
Time=2016-09-29+13%3A27%3A00&notifyType=REFUND_SUCCESS&signType=MD5&sign=yoursign
&charset=UTF-8&notifyContent=somenotifyconent
```

<br>

##3附录
###3.1错误码

<span id="codes">错误信息列表</span>

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

<br>

###3.2对账文件格式
与果仁支付技术对接时获取