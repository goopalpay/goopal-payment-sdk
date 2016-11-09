# 果仁支付Android SDK接入指南（v1.0.0）

## 1. 订单支付
> 为商户提供果仁支付功能，唤起果仁宝APP支付页面或H5页面。

**方法名称：**    ` goopalPayTask.goopalPay`

**方法原型：**
``` java
GoopalPayTask goopalPayTask = new GoopalPayTask(activity); 
HashMap<String,String> params=new HashMap<String, String>();
goopalPay.goopalPay(params);
```
<br></br>
**参数说明：**

| 参数名      | 参数类型 | 参数说明  |
| :-------- | --------:| :-- |
| params  | HashMap<String,String>| 订单支付的参数，详见下面params列表 |


<br></br>
**params列表：** 

| 名称  | 说明 | 类型  |备注 |
| :-------- | :--------| :-- | :-- |
| merchantId|商户ID| String 
| appId| 商户应用ID| String 
| orderNo| 商户订单号| String 
| gopOrderNo| 果付订单号| String 
| orderDate| 下单时间| String 
| singleNo| 唯一标识码| String 
| source| 支付类型| String |取下单时返回的source字段值|
| h5Url| H5收银台URL| String |取下单时返回的h5Url字段值|

`说明：以上参数不能为null`

<br></br>
**参数示例：**
```java
params.put("merchantId ","2112345713904416");
params.put("appId","GAPP_22F1F63307809F76");
params.put("orderNo ","MN20160718042759177");
params.put("gopOrderNo ","201607184275923400024");
params.put("orderDate ","20160718042759");
params.put("singleNo","MN20160718042759177");
params.put("source","1");
params.put("h5Url", " ");
goopalPay.goopalPay(params);
```
<br></br>
## 2. 集成流程
### 2.1 导入开发资源
第一步：将goopalpay-sdk-v1.jar包放入商户应用工程的libs目录下。
![Alt text](https://github.com/smilce/Interview-topic/blob/master/images/android1.png)

第二步：在Android studio 中右键点击该jar文件选择 add as Library…导入工程目录
![Alt text](https://github.com/smilce/Interview-topic/blob/master/images/Android2.png)

<br></br>
### 2.2 修改Manifest
在商户应用工程的AndroidManifest.xml文件里面添加声明：
``` java
<activity
    android:name="com.goopalpay.sdk.GoopalH5PayActivity"
    android:exported="false"
    android:screenOrientation="behind"
    android:windowSoftInputMode="adjustResize|stateHidden" >
</activity>
![Alt text](./1478578803787.png)
```

和权限声明：
``` java
<uses-permission android:name="android.permission.INTERNET"/>
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
```
<br></br>
### 2.3 添加混淆规则
Android studio中在工程的混淆规则文件中添加
```java
-keep class com.goopalpay.sdk.** {*;}
```
