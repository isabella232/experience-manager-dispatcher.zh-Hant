---
title: Dispatcher 安全性檢查清單
seo-title: Dispatcher 安全性檢查清單
description: 在開始生產之前應先完成的安全檢查清單。
seo-description: 在開始生產之前應先完成的安全檢查清單。
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
translation-type: tm+mt
source-git-commit: 7889c025fb8fb29e6f11ea01c5248470556d3160
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

Adobe強烈建議您在開始生產之前先完成下列檢查清單。

>[!CAUTION]
>
>您也必須先完成AEM版本的安全性檢查清單，才能上線。 請參閱對應的[Adobe Experience Manager檔案](https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/security-checklist.html)。

## 使用Dispatcher的最新版本{#use-the-latest-version-of-dispatcher}

您應安裝適用於您平台的最新可用版本。 您應升級您的Dispatcher實例，以使用最新版本，以利用產品和安全性增強功能。 請參閱[安裝Dispatcher](dispatcher-install.md)。

>[!NOTE]
>
>您可以查看調度程式日誌檔案來檢查當前版本的調度程式安裝。
>
>`[Thu Apr 30 17:30:49 2015] [I] [23171(140735307338496)] Dispatcher initialized (build 4.1.9)`
>
>要查找日誌檔案，請檢查`httpd.conf`中的調度程式配置。

## 限制可刷新快取的客戶端{#restrict-clients-that-can-flush-your-cache}

Adobe建議您[限制可清除快取的用戶端。](dispatcher-configuration.md#limiting-the-clients-that-can-flush-the-cache)

## 啟用HTTPS以確保傳輸層安全{#enable-https-for-transport-layer-security}

Adobe建議在作者和發佈例項上啟用HTTPS傳輸層。

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

在配置Dispatcher時，應盡可能限制外部訪問。 請參閱Dispatcher文檔中的[示例/filter Section](dispatcher-configuration.md#main-pars_184_1_title)。

## 確保管理URL的存取權被拒絕{#make-sure-access-to-administrative-urls-is-denied}

請確定您使用篩選器來封鎖任何管理URL（例如Web主控台）的外部存取。

有關需要阻止的URL清單，請參見[Testing Dispatcher Security](dispatcher-configuration.md#testing-dispatcher-security)。

## 使用Allowlists代替塊清單{#use-allowlists-instead-of-blocklists}

允許清單是提供訪問控制的更好方法，因為它們本身假定，除非所有訪問請求明確地屬於允許清單，否則應拒絕所有訪問請求。 此模型對某些配置階段可能尚未審查或考慮的新請求提供更嚴格的控制。

## 使用專用系統用戶{#run-dispatcher-with-a-dedicated-system-user}運行Dispatcher

配置Dispatcher時，您應確保Web伺服器由具有最低權限的專用用戶運行。 建議僅授予調度程式快取資料夾的寫訪問權限。

此外，IIS使用者需要依下列方式設定其網站：

1. 在網站的物理路徑設定中，選擇&#x200B;**以特定用戶身份連接**。
1. 設定使用者。

## 防止拒絕服務(DoS)攻擊{#prevent-denial-of-service-dos-attacks}

拒絕服務(DoS)攻擊是企圖使電腦資源對其預定用戶不可用。

在調度器級別，有兩種配置方法可防止DoS攻擊：[](https://docs.adobe.com/content/docs/en/dispatcher.html#/filter (濾鏡))

* 使用mod_rewrite模組（例如[Apache 2.4](https://httpd.apache.org/docs/2.4/mod/mod_rewrite.html)）執行URL驗證（如果URL模式規則不太複雜）。

* 使用[filters](dispatcher-configuration.md#configuring-access-to-conten-tfilter)防止分派器快取具有假擴充名稱的URL。\
   例如，變更快取規則，將快取限制在預期的MIME類型，例如：

   * `.html`
   * `.jpg`
   * `.gif`
   * `.swf`
   * `.js`
   * `.doc`
   * `.pdf`
   * `.ppt`

   例如，[限制外部訪問](#restrict-access)的配置檔案，這包括MIME類型的限制。

若要安全地在發佈例項上啟用完整功能，請設定篩選器以防止存取下列節點：

* `/etc/`
* `/libs/`

然後，配置篩選器以允許訪問以下節點路徑：

* `/etc/designs/*`
* `/etc/clientlibs/*`
* `/etc/segmentation.segment.js`
* `/libs/cq/personalization/components/clickstreamcloud/content/config.json`
* `/libs/wcm/stats/tracker.js`
* `/libs/cq/personalization/*` （JS、CSS和JSON）
* `/libs/cq/security/userinfo.json` （CQ使用者資訊）
* `/libs/granite/security/currentuser.json` (**資料不得快取**)

* `/libs/cq/i18n/*` （內部化）

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (ims-author-00AF43764F54BE740A490D44@AdobeID)
Last Modified Date: 2015-06-26T04:38:17.016-0400

<p>We need to highlight whether a path applies to all versions or specific ones.<br /> </p>

 -->

## 配置Dispatcher以防止CSRF攻擊{#configure-dispatcher-to-prevent-csrf-attacks}

AEM提供[framework](https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/security-checklist.html#verification-steps)，旨在防止跨網站偽造要求攻擊。 為了正確使用此框架，您需要允許在調度器中列出CSRF Token支援。 您可以透過下列方式執行此動作：

1. 建立允許`/libs/granite/csrf/token.json`路徑的篩選；
1. 將`CSRF-Token`標題添加到Dispatcher配置的`clientheaders`部分。

## 防止點按劫持{#prevent-clickjacking}

為避免點按劫持，建議您設定您的網站伺服器，以提供設為`SAMEORIGIN`的`X-FRAME-OPTIONS` HTTP標題。

有關點按劫持的更多[資訊，請參閱OWASP站點](https://www.owasp.org/index.php/Clickjacking)。

## 執行滲透測試{#perform-a-penetration-test}

Adobe強烈建議您在開始生產之前，先對AEM基礎架構執行滲透率測試。

