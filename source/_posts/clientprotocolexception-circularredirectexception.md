title: ClientProtocolException / CircularRedirectException
id: 149
categories:
  - Blog
date: 2014-12-01 08:37:26
tags:
---

## ClientProtocolException / CircularRedirectException

![redirect](/images/redirect.png)


CircularRedirectException 循环重定向
其实并没有真的发生循环重定向

三个请求顺序如下:

https://stage.ajinga.com/v1/client-jobs/2680

-->

http://stage.ajinga.com/m/client-jobs/2680/

-->

https://stage.ajinga.com/m/client-jobs/2680/

由于第2个和第3个请求 地址完全一致,
httpclient错误的判定其为 循环重定向.

当然实际上 第一次 redirect就该把请求重定向到 https://stage.ajinga.com/m/client-jobs/2680/

但是server错误的返回了
http://stage.ajinga.com/m/client-jobs/2680/

由于server不支持非ssl方式连接,自动触发了重定向,又将其重定向至https://stage.ajinga.com/m/client-jobs/2680/

逻辑上也没大问题,就是多了一次请求的时间.

通过设置参数

    httpClient.getHttpClient().getParams().setParameter(ClientPNames.ALLOW_CIRCULAR_REDIRECTS, true);

可以ignore 循环重定向问题.

但是本质上 server第一次就应当返回正确的https方式的redirect地址.