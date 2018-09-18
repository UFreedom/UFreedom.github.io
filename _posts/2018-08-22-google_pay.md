---
layout: post
title: Google 内购商品与 Google Billing Library
comments: true
category: Android
tags: [Android]
---


## 一. 应用内商品（内购）

1. 应用内商品有两种类型：

<div align="center">
<img src="http://oak5lii8q.bkt.clouddn.com/google_pay_1.png"  width="406"  height="81.6" />
</div>

- 一次性商品：Google Pay Console 称为 “Managed products”,用户付一笔钱，得到一个商品，一次交易，仅仅扣一笔费用。

- 订阅商品：用户购买一个商品后，需要周期性的进行续费，并持续享受此商品带来的功能。比如：视频VIP，音乐VIP 等

2. 内购配置属性

<div align="center">
<img src="http://oak5lii8q.bkt.clouddn.com/google_pay_2.png"  width="406"  height="81.6" />
</div>

- Production ID:    一个唯一的ID，用来标识一个商品，在 Google Billing Pay Library 被称为 SKU

<div align="center">
<img src="http://oak5lii8q.bkt.clouddn.com/google_pay_3.png"  width="390"  height="238" />
</div>

- Title
- Description
- 状态：有效和无效，初始化时无效。要想编辑的商品上线，需要改为有效状态

<div align="center">
<img src="http://oak5lii8q.bkt.clouddn.com/google_pay_4.png"  width="406"  height="372" />
</div>

- Price：用户为商品支付的金额，Google Play Consol 需要根据一个货币定一个基准价格，Google 会根据每个国家的汇率进行换算


<div align="center">
<img src="http://oak5lii8q.bkt.clouddn.com/google_pay_5.png"  width="394"  height="400" />
</div>

结算周期：这个是订阅商品的价格编辑区域，结算周期共有 周，月，3个月，6个月，年，季度


<div align="center">
<img src="http://oak5lii8q.bkt.clouddn.com/google_pay_6.png"  width="318"  height="104" />
</div>


编辑价格：


<div align="center">
<img src="http://oak5lii8q.bkt.clouddn.com/google_pay_7.png"  width="362"  height="298" />
</div>


一个 APP 可以拥有多个内个商品，只要 Product ID 不一样就行。Google Play Consol 后台提供定价模板，可以方便配置多个相同价格的应用内商品


## 二. Google Billing  Library

Google Pay SDK 使用 IInAppBillingService.aidl  进行通信：，[IInAppBillingService.aidl][1] 定义了和 Google Pay 进行通信的接口:


{% highlight java %}

package com.android.vending.billing;
import android.os.Bundle;

interface IInAppBillingService {

    int isBillingSupported(int apiVersion, String packageName, String type);


    Bundle getSkuDetails(int apiVersion, String packageName, String type, in Bundle skusBundle);


    Bundle getBuyIntent(int apiVersion, String packageName, String sku, String type,
        String developerPayload);


    Bundle getPurchases(int apiVersion, String packageName, String type, String continuationToken);

 
    int consumePurchase(int apiVersion, String packageName, String purchaseToken);

   
    int stub(int apiVersion, String packageName, String type);


    Bundle getBuyIntentToReplaceSkus(int apiVersion, String packageName,
        in List<String> oldSkus, String newSku, String type, String developerPayload);

    Bundle getBuyIntentExtraParams(int apiVersion, String packageName, String sku,
        String type, String developerPayload, in Bundle extraParams);


    Bundle getPurchaseHistory(int apiVersion, String packageName, String type,
        String continuationToken, in Bundle extraParams);

    
    int isBillingSupportedExtraParams(int apiVersion, String packageName, String type,
        in Bundle extraParams);
}

{% endhighlight %}


下面是Client-Server 通信逻辑



<div align="center">
<img src="http://oak5lii8q.bkt.clouddn.com/google_pay_8.png"  width="281"  height="281" />
</div>

  - 早期接入 Google Pay时，并没有官方正式发布的库，需要参考 Google Sample 中的代码，自己进行代码移植和封装
      - 比较考验代码设计功底
      - 设计不当会影响功能稳定性
      - 需要自己处理一些异常的情况
  - [android-inapp-billing-v3][2]] ： 一个开源的库，也是封装 Google AIDL，对外提供的 API  能满足业务需求
  
  - [Google Pay Billing Library][3]: Google 官方出的支付库，2017年6月才发布第一版。目前版本是 1.1
      ○ 提供良好的 API
      ○ 稳定性较好
      ○ 也是目前 Google 强烈建议使用的库


## 三. 如何接入 Google Pay Billing Library


一般接入流程：
  1. 添加依赖
  2. Collecton Google Pay
  3. 查询商品信息
  4. 购买商品
  5. 处理购买回调


**1.添加依赖**

{% highlight java %}

dependencies {
    ...
    implementation 'com.android.billingclient:billing:1.1'
}

{% endhighlight %}

**2.Collection** 

{% highlight java %}

// create new Person
private BillingClient mBillingClient;
...
mBillingClient = BillingClient.newBuilder(mActivity).setListener(this).build();
mBillingClient.startConnection(new BillingClientStateListener() {
    @Override
    public void onBillingSetupFinished(@BillingResponse int billingResponseCode) {
        if (billingResponseCode == BillingResponse.OK) {
            // The billing client is ready. You can query purchases here.
        }
    }
    @Override
    public void onBillingServiceDisconnected() {
        // Try to restart the connection on the next request to
        // Google Play by calling the startConnection() method.
    }
});

{% endhighlight %}

**3.查询商品信息**

List skuList = new ArrayList<> ();
skuList.add("premium_upgrade");
skuList.add("gas");
SkuDetailsParams.Builder params = SkuDetailsParams.newBuilder();
params.setSkusList(skuList).setType(SkuType.INAPP);
mBillingClient.querySkuDetailsAsync(params.build(),
    new SkuDetailsResponseListener() {
        @Override
        public void onSkuDetailsResponse(int responseCode, List skuDetailsList) {
            // Process the result.
        }
    });


<div align="center">
<img src="http://oak5lii8q.bkt.clouddn.com/google_pay_9.png"  width="184"  height="327" />
</div>


**4.购买商品**

{% highlight java %}

BillingFlowParams flowParams = BillingFlowParams.newBuilder()
         .setSku(skuId)
         .setType(SkuType.INAPP) // SkuType.SUB for subscription
         .build();
int responseCode = mBillingClient.launchBillingFlow(flowParams);

{% endhighlight %}


<div align="center">
<img src="http://oak5lii8q.bkt.clouddn.com/google_pay_10.png"  width="240"  height="426" />
</div>

<div align="center">
<img src="http://oak5lii8q.bkt.clouddn.com/google_pay_11.png"  width="240"  height="426" />
</div>

**5.处理购买回调**


{% highlight java %}

@Override
void onPurchasesUpdated(@BillingResponse int responseCode, List purchases) {
    if (responseCode == BillingResponse.OK
            && purchases != null) {
        for (Purchase purchase : purchases) {
            handlePurchase(purchase);
        }
    } else if (responseCode == BillingResponse.USER_CANCELED) {
        // Handle an error caused by a user cancelling the purchase flow.
    } else {
        // Handle any other error codes.
    }
}

{% endhighlight %}

如果是一次性商品，需要调用consume 接口，去消耗商品，要不然下次无法再次购买


**6.验证购买结果**

  - 应用内验证
      - Google Pay 使用私钥将 Response 的数据进行签名
      - 私钥会和每个应用的签名一一对应，也就是说一个应用只对应一个私钥
      - 客户端使用公钥对 Response 进行验证
  - Server Callback 验证
      - 在收到购买成功回调后，根据 Response  向 Server 发起一个 Callback 请求
      - Server 根据购买信息跟 Google Play Server 进行对账
      - 返回 check 结果

## 四. Google Play Billing Library 接入需要注意的问题


1. 一次性商品购买成功后，需要调用 consumeAsync(purchaseToken,onConsumeListener) 进行商品消耗，
     如果不进行消耗，这个商下次购买就会被阻塞，导致无法购买相同的商品
   
2. Server Callback 需要保证 Response 成功，否则会出现丢单情况


##五.Google Pay 沙盒测试

测试支付的时候，需要先在 Google Play进行应用分发。这样参与测试的用户就可以用 Google Play中下载应用了。在实际调试的过程中，我们可能是直接将APK分发给测试，这个前提是必须在 Google Play中发布 Alpha 或者 Beta 版本，且版本号不能低于 Google Play 中的版本。

**测试流程：**

  1. 上传 签名过的  alpha/beta 版本的apk
  2. 配置测试：封闭式测试, 论坛/Google+测试，
      a. 封闭式测试：将需要测试的账号手动添加到 Google Play后台
      b. 论坛,Google+ 测试：需要在后台指定 Google 论坛邮件地址，Google + 社群地址。然后整个论坛或者 Google + 的用户都可以参加测试
  3. 发送测试连接 https://play.google.com/apps/testing/{package-name} ，测试人员访问这个链接，然后确认参加测试
  4.  在Google Play 后台的 “许可测试账号”（非项目设置）里面添加测试的 Google 账号，被添加的账号测试花费的钱是虚拟的


**注意事项：**
  1. 能够进行支付沙盒测试的 APP 必须发布 Alpha 或者 Beta 版本，或者后期的正式版，内部测试版本不能进行支付沙盒测试
  2. APP 首次上传 Alpha 或者Beta，需要 1- 2个小时的更新时间
  3. 上 Alpha 或者 Beta 后，等待 App 进行更新，访问 https://play.google.com/apps/testing/{package-name}  能显示到邀请时，代表可进行支付测试


**参考链接：**

1.https://developer.android.com/google/play/billing/api#managed
2.https://developer.android.com/google/play/billing/billing_library_overview#java
3.https://developer.android.com/google/play/billing/billing_testing
4.http://wiki.midas.qq.com/post/index/2/32/54/0


  [1]: https://github.com/anjlab/android-inapp-billing-v3/blob/6fa7398b6a9d1187d2c0eff9368f0d40a1210e80/library/src/main/aidl/com/android/vending/billing/IInAppBillingService.aidl
  [2]: https://github.com/anjlab/android-inapp-billing-v3
  [3]: https://developer.android.com/google/play/billing/billing_library_overview#java
