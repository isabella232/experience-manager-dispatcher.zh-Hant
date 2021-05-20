---
title: Dispatcher 安全性檢查清單
seo-title: Dispatcher 安全性檢查清單
description: 生產前應先完成的安全性檢查清單。
seo-description: 生產前應先完成的安全性檢查清單。
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
source-git-commit: 3a0e237278079a3885e527d7f86989f8ac91e09d
workflow-type: tm+mt
source-wordcount: '653'
ht-degree: 1%

---

# Dispatcher 安全性檢查清單{#the-dispatcher-security-checklist}

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (ims-author-00AF43764F54BE740A490D44@AdobeID)
Last Modified Date: 2015-06-05T05:14:35.365-0400

<p>Food for thought listed on <a href="https://jira.corp.adobe.com/browse/DOC-5649">DOC-5649</a>. To be considered while proof-reading.</p> 
<p> </p>

 -->

Adobe強烈建議您在開始生產前先完成下列檢查清單。

>[!CAUTION]
>
>您也必須先完成AEM版本的安全性檢查清單，才能上線。 請參閱對應的[Adobe Experience Manager檔案](https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/security-checklist.html)。

## 使用最新版本的Dispatcher {#use-the-latest-version-of-dispatcher}

您應安裝適用於您平台的最新可用版本。 您應將Dispatcher例項升級，以使用最新版本，以運用產品和安全性增強功能。 請參閱[安裝Dispatcher](dispatcher-install.md)。

>[!NOTE]
>
>您可以查看Dispatcher記錄檔，以檢查Dispatcher安裝的目前版本。
>
>`[Thu Apr 30 17:30:49 2015] [I] [23171(140735307338496)] Dispatcher initialized (build 4.1.9)`
>
>若要尋找記錄檔，請檢查`httpd.conf`中的Dispatcher設定。

## 限制可刷新快取{#restrict-clients-that-can-flush-your-cache}的客戶端

Adobe建議您[限制可刷新快取的客戶端。](dispatcher-configuration.md#limiting-the-clients-that-can-flush-the-cache)

## 為傳輸層安全啟用HTTPS {#enable-https-for-transport-layer-security}

Adobe建議同時在製作和發佈執行個體上啟用HTTPS傳輸層。

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

## 限制訪問{#restrict-access}

設定Dispatcher時，您應盡可能限制外部存取。 請參閱Dispatcher檔案中的[範例/filter Section](dispatcher-configuration.md#main-pars_184_1_title)。

## 確保拒絕對管理URL的訪問{#make-sure-access-to-administrative-urls-is-denied}

請務必使用篩選器來封鎖外部對任何管理URL（例如Web主控台）的存取。

如需需要封鎖的URL清單，請參閱[測試Dispatcher安全性](dispatcher-configuration.md#testing-dispatcher-security)。

## 使用允許清單，而非封鎖清單{#use-allowlists-instead-of-blocklists}

允許清單是提供訪問控制的更好方法，因為它們本身認為，除非所有訪問請求明確屬於允許清單，否則應拒絕這些請求。 此模型對某些設定階段中可能尚未審核或考慮的新請求提供更嚴格的控制。

## 使用專用系統用戶{#run-dispatcher-with-a-dedicated-system-user}運行Dispatcher

設定Dispatcher時，您應確保Web伺服器是由具有最低權限的專用使用者執行。 建議僅授予Dispatcher快取資料夾的寫入存取權。

此外，IIS用戶需要按如下方式配置其網站：

1. 在網站的物理路徑設定中，選擇&#x200B;**作為特定用戶連接**。
1. 設定使用者。

## 防止拒絕服務(DoS)攻擊{#prevent-denial-of-service-dos-attacks}

拒絕服務(DoS)攻擊是使電腦資源無法供其預定用戶使用的一種嘗試。

在Dispatcher層級，有兩種設定方法可防止DoS攻擊：[](https://docs.adobe.com/content/docs/en/dispatcher.html#/filter (濾鏡))

* 使用mod_rewrite模組（例如[Apache 2.4](https://httpd.apache.org/docs/2.4/mod/mod_rewrite.html)）執行URL驗證（如果URL模式規則不太複雜）。

* 使用[filters](dispatcher-configuration.md#configuring-access-to-conten-tfilter)，防止Dispatcher快取具有假副檔名的URL。\
   例如，變更快取規則，將快取限制為預期的MIME類型，例如：

   * `.html`
   * `.jpg`
   * `.gif`
   * `.swf`
   * `.js`
   * `.doc`
   * `.pdf`
   * `.ppt`

   [限制外部存取](#restrict-access)的範例設定檔案，這包括mime類型的限制。

若要安全地在發佈執行個體上啟用完整功能，請設定篩選器以防止存取下列節點：

* `/etc/`
* `/libs/`

接著，設定篩選器以允許存取下列節點路徑：

* `/etc/designs/*`
* `/etc/clientlibs/*`
* `/etc/segmentation.segment.js`
* `/libs/cq/personalization/components/clickstreamcloud/content/config.json`
* `/libs/wcm/stats/tracker.js`
* `/libs/cq/personalization/*` （JS、CSS和JSON）
* `/libs/cq/security/userinfo.json` （CQ使用者資訊）
* `/libs/granite/security/currentuser.json` (**不得快取資料**)

* `/libs/cq/i18n/*` （內部化）

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (ims-author-00AF43764F54BE740A490D44@AdobeID)
Last Modified Date: 2015-06-26T04:38:17.016-0400

<p>We need to highlight whether a path applies to all versions or specific ones.<br /> </p>

 -->

## 設定Dispatcher以防止CSRF攻擊{#configure-dispatcher-to-prevent-csrf-attacks}

AEM提供[framework](https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/security-checklist.html#verification-steps) ，旨在防止跨網站請求偽造攻擊。 為了正確運用此架構，您必須在Dispatcher中允許列出CSRF代號支援。 您可以透過下列方式執行此作業：

1. 建立允許`/libs/granite/csrf/token.json`路徑的篩選器；
1. 將`CSRF-Token`標題新增至Dispatcher設定的`clientheaders`區段。

## 防止Clickjacking {#prevent-clickjacking}

為防止點按劫持，我們建議您將網站伺服器設定為提供設為`SAMEORIGIN`的`X-FRAME-OPTIONS` HTTP標題。

有關點擊頂升的更多[資訊，請參見OWASP站點](https://www.owasp.org/index.php/Clickjacking)。

## 執行滲透測試{#perform-a-penetration-test}

Adobe強烈建議您在開始生產前，先對AEM基礎架構執行滲透測試。
