---
title: uni-app开发之unipush推送
date: 2020-04-13 17:23:53 
categories: Code
tags:
  - app
  - uniApp

id: "js-unipush"
recommend: true
---

### 摘要

:::note
  本文详细介绍使用uni-app进行推送的步骤，包括开通UniPush服务、配置推送参数、自定义基座、测试推送，以及服务端推送代码示例。适用于希望了解uni-app推送功能的开发者。
:::

### 堆&栈

:::note
  最近需要开发app，经过多方对比，最终选择了使用dcloud的uni-app，一套代码可以编译七个平台，最主要就是相比于apicloud来说支持本地打包，减少了很多可能会带来的限制。 也就想着，它最大的诟病是说社区不活跃，可是社区氛围也是需要大家一块积极贡献的，那么我们团队加进来也给他增加一点活跃度。
  做app免不了会需要推送，从服务器主动给客户端发信息联系，这是客户端app相对来说非常大的一个优势。
  刚刚把uni-app在安卓端的推送走通，此处记录一下经验以方便后来者参考。
:::

### 开通UniPush推送服务
:::note
  UniPush仅支持uni-app类型项目，其它类型项目暂不支持 点此查看如何开通UniPush推送服务
:::

:::note
  在Hbuilder中修改mainfest.json文件配置
:::
:::picture
![uni-app开发之unipush推送](https://wp-cdn.4ce.cn/v2/VVAL1zd.png)
:::

:::note
勾选DCloud UniPush，然后点击配置按钮会进入uniPush的后端配置，需要先认证，认证通过后，可以在我创建的应用列表中查看自己当前的应用。如果有多个项目，此处可以根据列表中的App_id和本地的appid关联查看 https://dev.dcloud.net.cn/app/index?type=0 
:::

![fa8725a5e187bce66701bf228c816238.png](https://wp-cdn.4ce.cn/v2/exk3nOu.png)



:::note
点击当前项目，进入对应项目的管理页面，展开Unipush菜单，选择修改应用信息，填写自己想要指定的包名及根据本地证书生成的签名信息（如果要使用官方的证书，那就按提示填写官方签名信息） 
:::

![bf5ed12973e0e21c37ef992207728890.png](https://wp-cdn.4ce.cn/v2/pxXO9LA.png)

:::md
配置完可进入Unipush的“推送通知”测试页面 
:::

![d779e0ed555977023cc969c831c936f1.png](https://wp-cdn.4ce.cn/v2/01GhL09.png)

### 自定义基座

:::md
要想实现推送功能，需要先自定义基座(因为推送也是需要依赖包名等信息的，官方说过，推送必须要用自定义基座才行)
:::

![5a6010b4ce22cf3dcd2dd32a1cb87e87.png](https://wp-cdn.4ce.cn/v2/Zk0oIdW.png)

:::md
选择自定义基座，然后会有弹窗提示
:::

![2927c028d680fb4fd60b439f6f3c9c56.png](https://wp-cdn.4ce.cn/v2/osAscnX.png)

:::md
这里可以选择使用公用证书，也可以使用自己之前已经生成的自有证书签名，填写必要的信息，然后选择打包；此处要注意这里填写的包名要和上面unipush管理页面中配置的一样。

填写完成后，点击打包按钮，等待。完成后会在控制台提示
:::

![c1354a0f5c9edbeb35864b4aafe3d682.png](https://wp-cdn.4ce.cn/v2/EzLWXqw.png)


:::md
我们按照提示，选择自己之前定义的基座即可。这个时候再点击运行到手机或模拟器，然后在后台测试推送即可 
:::

![2dd2c75c7a1950706d53f0fa4e22a764.png](https://wp-cdn.4ce.cn/v2/oDr21M7.png)



### 测试推送

:::note
进入unipush推送通知的页面创建推送，填写通知标题和通知内容，此处如果有客户端之前运行成功，那么预计人数会不是0人，如果是0人的话确定按钮会无法点击，因为无人可推；即使在目标选择中根据客户端输出的clientid选择了特定用户，手机端也是无法推送的。此处如果不行，就再去检查包名配置和基座选择是否有问题。 
:::

![708db0cfadc850d55fd1dd2198cd4ee4.png](https://wp-cdn.4ce.cn/v2/3Vhqhs8.png)


:::md
因为这个系统就是个推给dcloud的私有部署，所以如果有疑问的方面也可以到个推官方注册一个账号，下载demo试试，增加一点成就感。并且后续的开发sdk、文档等都可以根据个推的做。
:::

### 服务端推送快速入门

:::md
我们在本地建一个maven项目，在pom.xml的dependencies中增加一个推送sdk的依赖包配置
:::


```js
<dependency>
    <groupId>com.gexin.platform</groupId>
    <artifactId>gexin-rp-sdk-http</artifactId>
    <version>4.1.0.1</version>
</dependency>

如果下载有问题，可以在project的下层添加一个库位置

 
<repositories>
    <repository>
        <id>getui-nexus</id>
        <url>http://mvn.gt.igexin.com/nexus/content/repositories/releases/</url>
    </repository>
 </repositories>

然后建测试类如下，把其中的appid、appkey、masterSecret换成自己的，url不用换； 经过测试推测，虽然在unipush中有私有部署系统，但是推送的消息队列还是在个推云。大概是OEM

 
import com.gexin.rp.sdk.base.IPushResult;
import com.gexin.rp.sdk.base.impl.AppMessage;
import com.gexin.rp.sdk.http.IGtPush;
import com.gexin.rp.sdk.template.LinkTemplate;
 
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;
 
 
public class AppPush {
 
    //定义常量, appId、appKey、masterSecret 采用本文档 "第二步 获取访问凭证 "中获得的应用配置
    private static String appId = "你的appid";
    private static String appKey = "你的appkey";
    private static String masterSecret = "你的masterSecret";
    private static String url = "http://sdk.open.api.igexin.com/apiex.htm";
 
    public static void main(String[] args) throws IOException {
 
        IGtPush push = new IGtPush(url, appKey, masterSecret);
 
        // 定义"点击链接打开通知模板"，并设置标题、内容、链接
        LinkTemplate template = new LinkTemplate();
        template.setAppId(appId);
        template.setAppkey(appKey);
        template.setTitle("你好你好");
        template.setText("哦，好吧");
        template.setUrl("http://getui.com");
 
        List<String> appIds = new ArrayList<String>();
        appIds.add(appId);
 
        // 定义"AppMessage"类型消息对象，设置消息内容模板、发送的目标App列表、是否支持离线发送、以及离线消息有效期(单位毫秒)
        AppMessage message = new AppMessage();
        message.setData(template);
        message.setAppIdList(appIds);
        message.setOffline(true);
        message.setOfflineExpireTime(1000 * 600);
 
        IPushResult ret = push.pushMessageToApp(message);
        System.out.println(ret.getResponse().toString());
    }
}

```
### 透传消息

考虑到推送通知只在android端可以使用，为了以后ios推送少改，所以此处我们也使用透传消息推送；又知透传消息的格式当满足

```json
t.setTransmissionContent("{title:\"标题\",content:\"内容\",payload:\"自定义数据\"}"); 
```
时，会在安卓端自动转为通知消息，所以我们此处就按这种方式推送透传消息。 其中payload的值既可以是字符串，又可以是对象；这里的示例数据我写的是字符串，但实际使用时可以修改格式来符合自身业务逻辑的处理，比如

```json
{{title:"标题",content:"内容",payload:{type:1,fId:1}}
```


对于代码的修改主要是要改动消息模板，即将上面的LinkTemplate改成TransmissionTemplate， 代码如下：

```java
public static void main(String[] args) throws IOException {
 
        IGtPush push = new IGtPush(url, appKey, masterSecret);
 
        TransmissionTemplate t = new TransmissionTemplate();
        t.setAppId(appId);
        t.setAppkey(appKey);
        t.setTransmissionContent("{title:\"标题\",content:\"内容\",payload:\"自定义数据\"}");
        t.setTransmissionType(1);
        
        // 定义"点击链接打开通知模板"，并设置标题、内容、链接
//        LinkTemplate template = new LinkTemplate();
//        template.setAppId(appId);
//        template.setAppkey(appKey);
//        template.setTitle("你好你好");
//        template.setText("哦，好吧");
//        template.setLogoUrl("");
//        template.setUrl("http://getui.com");
 
        List<String> appIds = new ArrayList<String>();
        appIds.add(appId);
 
        // 定义"AppMessage"类型消息对象，设置消息内容模板、发送的目标App列表、是否支持离线发送、以及离线消息有效期(单位毫秒)
        AppMessage message = new AppMessage();
        message.setData(t);
        message.setAppIdList(appIds);
        message.setOffline(true);
        message.setOfflineExpireTime(1000 * 600);
 
        IPushResult ret = push.pushMessageToApp(message);
        System.out.println(ret.getResponse().toString());
}
```

### 本地编译
作为专业的客户端开发人员，最好还是在本地有完整的编译环境。这也是我们在dcloud和apicloud之间选择当前技术栈的原因。

为了简单起见，可以把官方提供的实例项目HBuilder-Hello先在本地打包，然后用官方实例的壳，将里面h5的一些生成文件替换。此处操作流程如下：

1. 生成本地打包App资源
![ff0ea2b18fd6a8ef4134283844ec1421.png](https://wp-cdn.4ce.cn/v2/L588Tse.png)

2. 将生成的资源放进实例项目app\src\main\assets\apps文件夹下，将之前的Hello H5删掉即可， 然后将此路径下的dcloud_control.xml中的appid的值改为apps下一层即www上一层文件夹的名称 

![cd87a2619ec23e0234703eeb04e22e5f.png](https://wp-cdn.4ce.cn/v2/H1ti1rY.png)

:::md
此时已经可以打包成app并且正常打开，但是还不能推送 
:::
:::md
  修改src/main/java文件夹下的包名为自己之前填写的包名，修改src/main下AndroidManifest.xml中package包名，修改AndroidManifest.xml中的推送参数配置 
:::

![080f82120ae256773d2efefeceb1388f.png](https://wp-cdn.4ce.cn/v2/KzNDgpX.png)
![799b2dd127daa89c00f7ec06e2e51b7b.png](https://wp-cdn.4ce.cn/v2/GG9aUtK.png)

:::md
 然后debug或打包即可。
:::
