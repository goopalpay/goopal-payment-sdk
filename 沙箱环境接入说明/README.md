# 商户沙箱环境接入指南

<br>
商户接入果仁支付时，可先在沙箱环境做调试，<br>

具体步骤为:<br>

**1.商户入驻** <br>
  登录http://mer.sandbox.treespaper.com/ 获取商户编号和app_id<br>
 （注：登录账号密码请联系果仁支付相关人员获得）<br>

**2.下载SDK** <br>
  1）移动支付产品：需下载服务端sdk和客户端sdk<br>
  2）手机网站支付产品：只下载服务端sdk<br>

**3.服务端接口地址配置** <br>
  商户交易接口：http://pay.sandbox.treespaper.com/gateway<br>
 （其他与正式环境的内容相同，无需修改）
 
**4.验证流程**<br>
  1）下载并安装Android版果仁宝app，即 app-guorenbao-sandbox.apk，如果不安装此app，选择果仁支付后会跳转至果付H5支付页面；<br>
  2）按签署的果付产品，接入相应的服务端/客户端SDK；<br>
  3）下单验证：调用下单接口，完成支付功能；<br>
  4）退款验证：通过接口/商户平台发起退款。<br>
