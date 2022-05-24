---
title: Dispatcher 安全性檢查清單
seo-title: The Dispatcher Security Checklist
description: 在進入生產階段前應該完成的安全性檢查清單。
seo-description: A security checklist that should be completed before going on production.
uuid: 7bfa3202-03f6-48e9-8d2e-2a40e137ecbe
contentOwner: User
products: SG_EXPERIENCEMANAGER/DISPATCHER
topic-tags: dispatcher
content-type: reference
discoiquuid: fbfafa55-c029-4ed7-ab3e-1bebfde18248
jcr-lastmodifiedby: remove-legacypath-6-1
index: y
internal: n
snippet: y
exl-id: 49009810-b5bf-41fd-b544-19dd0c06b013
source-git-commit: a21b527700b3a5c3a11eadaefc729e754ba2b37b
workflow-type: tm+mt
source-wordcount: '638'
ht-degree: 100%

---

# Dispatcher 安全性檢查清單{#the-dispatcher-security-checklist}

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (ims-author-00AF43764F54BE740A490D44@AdobeID)
Last Modified Date: 2015-06-05T05:14:35.365-0400

<p>Food for thought listed on <a href="https://jira.corp.adobe.com/browse/DOC-5649">DOC-5649</a>. To be considered while proof-reading.</p> 
<p> </p>

 -->

Adobe 極力建議您在進入生產階段前完成以下檢查清單。

>[!CAUTION]
>
>您也必須在上線之前完成您的 AEM 版本的安全性檢查清單。 請參閱相應的 [Adobe Experience Manager 文件](https://helpx.adobe.com/experience-manager/6-5/sites/administering/using/security-checklist.html)。

## 使用最新版的 Dispatcher {#use-the-latest-version-of-dispatcher}

您應該安裝您的平台適用的最新可用版本。 您應該升級 Dispatcher 執行個體來使用最新版，以充分利用產品和安全性增強功能。 請參閱[安裝 Dispatcher](dispatcher-install.md)。

>[!NOTE]
>
>您可以查看 Dispatcher 記錄檔來檢查 Dispatcher 安裝的最新版本。
>
>`[Thu Apr 30 17:30:49 2015] [I] [23171(140735307338496)] Dispatcher initialized (build 4.1.9)`
>
>若要尋找記錄檔，請在 `httpd.conf` 中檢查 Dispatcher 設定。

## 限制可以清除您的快取的用戶端 {#restrict-clients-that-can-flush-your-cache}

Adobe 建議您最好[限制可以清除您的快取的用戶端。](dispatcher-configuration.md#limiting-the-clients-that-can-flush-the-cache)

## 啟用 HTTPS 以提供傳輸層安全性 {#enable-https-for-transport-layer-security}

Adobe 建議您在編寫和發佈執行個體上啟用 HTTPS 傳輸層安全性。

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

當您設定 Dispatcher 時，應該盡可能限制外部存取。 請參閱 Dispatcher 文件中的[範例 /filter 區段](dispatcher-configuration.md#main-pars_184_1_title)。

## 確保對管理 URL 的存取權已被拒絕 {#make-sure-access-to-administrative-urls-is-denied}

務必使用篩選條件來封鎖對任何管理 URL (例如網頁主控台) 的外部存取。

如需需要封鎖的 URL 清單，請參閱[測試 Dispatcher 安全性](dispatcher-configuration.md#testing-dispatcher-security)。

## 使用允許清單而非封鎖清單 {#use-allowlists-instead-of-blocklists}

允許清單是提供存取控制的較佳方式，因為其原本就會假定所有存取請求都應該被拒絕，除非這些請求被明確列為允許清單的一部分。 此模型對可能尚未審查或在特定設定階段被列入考量的新請求提供較嚴格的控管。

## 透過專用系統使用者執行 Dispatcher {#run-dispatcher-with-a-dedicated-system-user}

在設定 Dispatcher 時，您應該確保網頁伺服器是由具備最低權限的專用使用者所執行。 建議您最好只授與對 Dispatcher 快取資料夾的寫入權限。

另外，IIS 使用者需要依照如下的方式設定其網站：

1. 在您網站的實體路徑設定中，請選取&#x200B;**以特定使用者身分連線**。
1. 設定使用者。

## 避免阻斷服務 (DoS) 攻擊 {#prevent-denial-of-service-dos-attacks}

阻斷服務 (DoS) 攻擊指的是嘗試讓電腦資源無法提供給其目標使用者使用。

在 Dispatcher 層級，有兩個設定方法可避免 DoS 攻擊：[](https://docs.adobe.com/content/docs/en/dispatcher.html#/filter (篩選條件))

* 使用 mod_rewrite 模組 (例如 [Apache 2.4](https://httpd.apache.org/docs/2.4/mod/mod_rewrite.html)) 執行 URL 驗證 (如果 URL 模式規則不是太複雜)。

* 使用[篩選條件](dispatcher-configuration.md#configuring-access-to-conten-tfilter)避免 Dispatcher 快取帶有虛假副檔名的 URL。\
   例如，變更快取規則，將快取限制為預期的 mime 類型，例如：

   * `.html`
   * `.jpg`
   * `.gif`
   * `.swf`
   * `.js`
   * `.doc`
   * `.pdf`
   * `.ppt`

   可以看到用來[限制外部存取](#restrict-access)的設定檔範例，這包括 mime 類型的限制。

若要安全地在發佈執行個體上啟用完整功能，請設定篩選條件來避免存取以下節點：

* `/etc/`
* `/libs/`

然後設定篩選條件來允許存取以下節點路徑：

* `/etc/designs/*`
* `/etc/clientlibs/*`
* `/etc/segmentation.segment.js`
* `/libs/cq/personalization/components/clickstreamcloud/content/config.json`
* `/libs/wcm/stats/tracker.js`
* `/libs/cq/personalization/*` (JS、CSS 和 JSON)
* `/libs/cq/security/userinfo.json` (CQ 使用者資訊)
* `/libs/granite/security/currentuser.json` (**資料不得快取**)

* `/libs/cq/i18n/*` (內部化)

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (ims-author-00AF43764F54BE740A490D44@AdobeID)
Last Modified Date: 2015-06-26T04:38:17.016-0400

<p>We need to highlight whether a path applies to all versions or specific ones.<br /> </p>

 -->

## 設定 Dispatcher 以避免 CSRF 攻擊 {#configure-dispatcher-to-prevent-csrf-attacks}

AEM 提供了旨在避免跨網站請求偽造攻擊的[架構](https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/security-checklist.html#verification-steps)。 為了適當地利用此架構，您在 Dispatcher 中需要允許清單 CSRF 權杖支援。 您可以透過以下方式達成此目的：

1. 建立篩選條件以允許 `/libs/granite/csrf/token.json` 路徑；
1. 將 `CSRF-Token` 標頭新增到 Dispatcher 設定的 `clientheaders` 區段。

## 避免點擊劫持 {#prevent-clickjacking}

若要避免點擊劫持，建議您設定網頁伺服器，以提供設為 `SAMEORIGIN` 的 `X-FRAME-OPTIONS` HTTP 標頭。

如需[點擊劫持的詳細資訊，請查看 OWASP 網站](https://www.owasp.org/index.php/Clickjacking)。

## 執行滲透測試 {#perform-a-penetration-test}

Adobe 極力建議您在進入生產階段前執行 AEM 基礎結構的滲透測試。
