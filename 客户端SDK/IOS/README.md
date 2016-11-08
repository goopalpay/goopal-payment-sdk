# 果仁支付iOS端SDK接入指南（v1.0.0）
> 注：本文为果仁支付iOS SDK的新手使用教程，只涉及教授SDK的使用方法，默认读者已经熟悉XCode开发工具的基本使用方法，以及具有一定的编程知识基础等。Xcode 版本要求7.3以上，iOS8.0以上。

<br></br>
## 1. 获取AppID
>商户与果仁宝签订合同、通过公司认证后，果仁支付开通商户账号。
>商户可通过登录`果仁支付商户平台`，创建应用后获取AppID。
<br></br>
`注：合作邮箱goopal.service@goopal.com `

<br></br>
## 2. 导入 SDK
>SDK文件包括以下三个：
>`goopalpay-sdk-v1.a`, `GoopalPayApi.h` , `GoopalPayObject.h`
<br></br>
请前往[资源中心](https://mer.goopal.com.cn/#/doc/guide)下载

<br></br>
SDK导入到您的项目中，步骤如下：
* 1. 解压 iOS SDK 压缩文件；
* 2. 添加 `goopalpay-sdk-v1.a` 、`GoopalPayApi.h` 、`GoopalPayObject.h` 添加到您的 iOS 工程中；

`注：记得勾选 "Copy items if needed"`

<br></br>
## 3. 集成与使用
### 3.1导入SDK
将SDK文件中包含的 `goopalpay-sdk-v1.a`，`GoopalPayApi.h`，`GoopalPayObject.h` 三个文件添加到你所建的工程中（如下图所示，建立了一个名为`textPay` 的工程，并把以上三个文件添加到`testPay`文件夹下）。
![Alt text](https://github.com/smilce/Interview-topic/blob/master/images/ios1.png)
<br></br>
在你需要使用的控制器导入头文件`GoopalPayApi.h`。

<br></br>
### 3.2使用goopalpay-sdk-v1.a
> 在文件导入和编译成功以后，开始使用果仁支付。在你需要的控制器中，导入头文件#import "GoopalPayApi.h"。在获取商品信息后，传入参数，参数格式必须严格要求。


**参数示例：**

```
GoopalRequest * reqs = [[GoopalRequest alloc]init];
reqs.gopOrderNo = data[@"gopOrderNo"];
reqs.orderNo= data[@"orderNo"];
reqs.merchantId = data[@"merchantId"]
reqs.singleNo= data[@"singleNo"];
reqs.appId= data[@"appId"];
reqs.h5Url = data[@"h5Url"];
reqs.orderDate= data[@"orderDate"]; 
reqs.source = data[@"source"];
```

**参数内容如下：（所有参数不能为nil）**

| 名称  | 说明 | 类型  |备注|
| :-------- | :--------| :-- |:--|
| gopOrderNo|果付订单号| String ||
| orderNo| 商户订单号| String ||
| merchantId| 商户ID| String ||
| singleNo| 唯一标识码| String ||
| appId | 应用程序的ID | String ||
| source| 支付方式| String |1表示APP，2表示H5|
| h5Url| H5收银台URL| String |下单时返回的h5Url字段值|
| orderDate| 下单时间| String ||

<br></br>
### 3.3调起支付
> 将以下参数传入，调用如下函数即可唤起果仁宝APP或H5页面完成支付:
```
[GoopalPayApi sendRequst:reqs];
```

<br></br>
## 4.注意事项

1. ios9以后，苹果公司只支持https请求，请在info.plist文件中设置白名单,如下图
![Alt text](https://github.com/smilce/Interview-topic/blob/master/images/ios2.png)

2. 一定要将下面的URL Types设定成自己的Schemes.
![Alt text](https://github.com/smilce/Interview-topic/blob/master/images/ios3.png)

