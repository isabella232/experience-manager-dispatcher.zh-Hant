---
title: Dispatcher Security Checklist
seo-title: Dispatcher Security Checklist
description: 執行前應先完成的安全性檢查清單。
seo-description: 執行前應先完成的安全性檢查清單。
uuid: 7bfa3202-03f6-48e9-1d2e-2a40e137ecobe
contentOwner: 使用者
products: SG_ PERIENCENCENAGER/ADDER
topic-tags: dispatcher
content-type: 引用
discoiquuid: fbfafa55-c029-4eed7-ab3 e-1babfde18248
jcr-lastmodifiedby: remove-leacacypath-6-1
index: y
internal: n
snippet: y
translation-type: tm+mt
source-git-commit: f35c79b487454059062aca6a7c989d5ab2afaf7b

---


# Dispatcher Security Checklist{#the-dispatcher-security-checklist}

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (ims-author-00AF43764F54BE740A490D44@AdobeID)
Last Modified Date: 2015-06-05T05:14:35.365-0400

<p>Food for thought listed on <a href="https://jira.corp.adobe.com/browse/DOC-5649">DOC-5649</a>. To be considered while proof-reading.</p> 
<p> </p>

 -->

dispatcher做為前端系統，為您的Adobe Experience Manager基礎架構提供額外的安全性層。Adobe強烈建議您在進行生產前，先完成下列檢查清單。

>[!CAUTION]
>
>您也必須先完成AEM版本的安全性檢查清單，才能上線。請參閱對應的 [Adobe Experience Manager文件](https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/security-checklist.html)。

## 使用最新版Dispatcher {#use-the-latest-version-of-dispatcher}

您應安裝適用於平台的最新可用版本。您應升級Dispatcher實例，以使用最新版本來運用產品和安全性增強功能。請參閱 [安裝Dispatcher](dispatcher-install.md)。

>[!NOTE]
>
>您可以查看dispatcher記錄檔，檢查最新版本的dispatcher安裝。
>
>`[Thu Apr 30 17:30:49 2015] [I] [23171(140735307338496)] Dispatcher initialized (build 4.1.9)`
>
>若要尋找記錄檔，請檢查您 `httpd.conf`的dispatcher組態。

## 限制可清除快取的客戶 {#restrict-clients-that-can-flush-your-cache}

Adobe建議 [您限制可清除快取的用戶端。](dispatcher-configuration.md#limiting-the-clients-that-can-flush-the-cache)

## 啓用HTTPS以提供傳輸層安全性 {#enable-https-for-transport-layer-security}

Adobe建議在作者和發佈例項上啓用HTTPS傳輸層。

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (ims-author-00AF43764F54BE740A490D44@AdobeID)
Last Modified Date: 2015-06-26T04:41:28.841-0400

<p>Recommended to have SSL termination, front end SSL.</p> 
<p>Question is do we want to have SSL communication between dispatcher and AEM instances (publish and/or author).</p> 
<p>We might want to have two items:</p> 
<ul> 
 <li>MUST HTTPS clients -&gt; dispatcher / load balancer</li> 
 <li>NICE load balancer -&gt; dispatcher<br /> </li> 
 <li>NICE dispatcher -&gt; instances if sensitive information such as credit cards / or infrastructure requirements such as DMZ</li> 
</ul>

 -->

## 限制存取 {#restrict-access}

設定Dispatcher時，您應盡可能限制外部存取。請參閱 [Dispatcher文件中的範例/filter章節](dispatcher-configuration.md#main-pars_184_1_title) 。

## 確定存取管理URL遭拒 {#make-sure-access-to-administrative-urls-is-denied}

確定您使用篩選器封鎖任何管理URL(例如Web主控台)的外部存取。

如需需要封鎖的URL清單，請參閱 [測試Dispatcher安全性](dispatcher-configuration.md#testing-dispatcher-security) 。

## 使用白名單取代黑名單 {#use-whitelists-instead-of-blacklists}

白名單是提供存取控制的較好方式，因為他們繼承了存取權，因此他們假設所有存取要求都應該被拒絕，除非明確的白名單部分。此模型對於在特定配置階段尚未審查或考量的新請求，提供更嚴格的控制權。

## 使用專用系統使用者執行Dispatcher {#run-dispatcher-with-a-dedicated-system-user}

設定Dispatcher時，應確保Web伺服器由具有最少權限的專用使用者執行。建議只將寫入操作授與dispatcher快取資料夾。

Additionnaly，IIS使用者需要設定其網站如下：

1. 在網站的實體路徑設定中，選取 **「Connect」作為特定使用者**。
1. 設定使用者。

## 防止拒絕服務(DoS)攻擊 {#prevent-denial-of-service-dos-attacks}

拒絕服務(DoS)攻擊試圖讓其預定使用者無法使用電腦資源。

在dispatcher層級，有兩種設定以防止DoS攻擊的方法： [](https://docs.adobe.com/content/docs/en/dispatcher.html#/filter (濾鏡))

* 使用mod_ rewrite模組(例如 [Apache2.2](https://httpd.apache.org/docs/2.2/mod/mod_rewrite.html))來執行URL驗證(如果URL模式規則不是太複雜)。

* 使用 [篩選條件，防止傳送程式使用惡意擴充功能快取URL](dispatcher-configuration.md#configuring-access-to-conten-tfilter)。\
   例如，變更快取規則以限制快取至預期的MIME類型，例如：

   * `.html`
   * `.jpg`
   * `.gif`
   * `.swf`
   * `.js`
   * `.doc`
   * `.pdf`
   * `.ppt`
   可看到設定檔案 [以限制外部存取](#restrict-access)，這包括MIME類型的限制。

若要安全啓用發佈例項的完整功能，請設定篩選器以防止存取下列節點：

* `/etc/`
* `/libs/`

接著，設定篩選器以允許存取下列節點路徑：

* `/etc/designs/*`
* `/etc/clientlibs/*`
* `/etc/segmentation.segment.js`
* `/libs/cq/personalization/components/clickstreamcloud/content/config.json`
* `/libs/wcm/stats/tracker.js`
* `/libs/cq/personalization/*` (JS、CSS和JSON)
* `/libs/cq/security/userinfo.json` (CQ使用者資訊)
* `/libs/granite/security/currentuser.json` (**資料不得快取**)

* `/libs/cq/i18n/*` (Internation)

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (ims-author-00AF43764F54BE740A490D44@AdobeID)
Last Modified Date: 2015-06-26T04:38:17.016-0400

<p>We need to highlight whether a path applies to all versions or specific ones.<br /> </p>

 -->

## 設定Dispatcher以防止CSRF攻擊 {#configure-dispatcher-to-prevent-csrf-attacks}

AEM提供一個 [架構](https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/security-checklist.html#verification-steps) ，旨在防止跨網站偽造要求偽造攻擊。若要正確使用此架構，您必須在傳送程式中建立安全清單的CSRF代號支援。You can do this by：

1. 建立篩選器以允許 `/libs/granite/csrf/token.json` 路徑；
1. 將 `CSRF-Token` 標題新增至Dispatcher組態 `clientheaders` 的區段。

## 防止Clickjacking {#prevent-clickjacking}

若要防止clickjacking，建議您設定網路伺服器，提供 `X-FRAME-OPTIONS` HTTP標題設 `SAMEORIGIN`為。

有關clickjacking的詳細 [資訊，請參閱OASP網站](https://www.owasp.org/index.php/Clickjacking)。

## 執行滲透效果測試 {#perform-a-penetration-test}

Adobe強烈建議您先執行AEM基礎架構的滲透測試，然後進行生產。

