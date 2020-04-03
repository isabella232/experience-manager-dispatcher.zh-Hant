---
title: 使 AEM 快取的頁面失效
seo-title: 從Adobe AEM使快取頁面無效
description: 瞭解如何設定Dispatcher與AEM之間的互動，以確保有效的快取管理。
seo-description: 瞭解如何設定Adobe AEM Dispatcher與AEM之間的互動，以確保有效的快取管理。
uuid: 66533299-55c0-4864-9beb-77e281af9359
cmgrlastmodified: 01.11.2007 08 22 29 [aheimoz]
pageversionid: 1193211344162
template: /apps/docs/templates/contentpage
contentOwner: User
products: SG_EXPERIENCEMANAGER/DISPATCHER
topic-tags: dispatcher
content-type: reference
discoiquuid: 79cd94be-a6bc-4d34-bfe9-393b4107925c
translation-type: tm+mt
source-git-commit: 85497651ce29c8564da4b52c60819a48b776af7b

---


# 使 AEM 快取的頁面失效 {#invalidating-cached-pages-from-aem}

搭配使用Dispatcher與AEM時，必須設定互動以確保有效的快取管理。 視您的環境而定，此配置還可以提高效能。

## 設定AEM使用者帳戶 {#setting-up-aem-user-accounts}

預設用 `admin` 戶帳戶用於驗證預設安裝的複製代理。 您應建立專用用戶帳戶以用於複製代理。

如需詳細資訊，請參 [閱「AEM安全性檢查清單」的「設定複製與傳輸使用者](https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/security-checklist.html#VerificationSteps) 」區段。

## 從編寫環境中使Dispatcher Cache失效 {#invalidating-dispatcher-cache-from-the-authoring-environment}

AEM作者例項上的複製代理會在發佈頁面時，將快取失效要求傳送給Dispatcher。 請求會使Dispatcher最終在發佈新內容時刷新快取中的檔案。

<!-- 

Comment Type: draft

<p>Cache invalidation requests for a page are also generated for any aliases or vanity URLs that are configured in the page properties. </p>

 -->

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (ims-author-0436B4A35714BFF67F000101@AdobeID)
Last Modified Date: 2017-05-25T10:37:23.679-0400

<p>Hiding this information due to <a href="https://jira.corp.adobe.com/browse/CQDOC-5805">CQDOC-5805</a>.</p>

 -->

請依照下列程式，在AEM作者例項上設定複製代理，以便在頁面啟動時使Dispatcher快取失效：

1. 開啟AEM Tools主控台。(`https://localhost:4502/miscadmin#/etc`)
1. 在Tools/replication/Agents on author下開啟所需的複製代理。 您可以使用預設安裝的Dispatcher Flush代理。
1. 按一下「編輯」，然後在「設定」標籤中確定已選 **取「啟** 用」。

1. （可選）要啟用別名或虛名路徑失效請求，請選擇「別 **名更新** 」選項。
1. 在「傳輸」頁籤上，輸入訪問Dispatcher所需的URI。\
   如果您使用標準Dispatcher Flush代理，則可能需要更新主機名和埠；例如，https://&lt;*dispatcherHost*>:&lt;*portApache*>/dispatcher/invalidate.cache

   **注意：** 對於Dispatcher Flush代理，僅當使用基於路徑的虛擬主機條目來區分場時，才使用URI屬性。 使用此欄位可以定位要失效的場。 例如，場#1的虛擬主機為，場#2 `www.mysite.com/path1/*` 的虛擬主機為 `www.mysite.com/path2/*`。 您可以使用的URL來 `/path1/invalidate.cache` 定位第一個群體， `/path2/invalidate.cache` 以及定位第二個群體。 如需詳細資訊，請參 [閱使用Dispatcher搭配多個網域](dispatcher-domains.md)。

1. 視需要設定其他參數。
1. 按一下「確定」激活代理。

或者，您也可以從 [AEM Touch UI存取和設定Dispatcher Flush代理](https://helpx.adobe.com/experience-manager/6-2/sites/deploying/using/replication.html#ConfiguringaDispatcherFlushagent)。

如需如何啟用虛名URL存取權的詳細資訊，請參 [閱啟用虛名URL存取權](dispatcher-configuration.md#enabling-access-to-vanity-urls-vanity-urls)。

>[!NOTE]
>
>用於刷新調度器快取的代理不必具有用戶名和口令，但如果配置了這些名稱和口令，則會使用基本身份驗證發送。

這種方法存在兩個潛在問題：

* 必須從編寫實例訪問Dispatcher。 如果您的網路（例如防火牆）已設定成限制兩者之間的存取，則可能不會。

* 發佈和快取失效同時發生。 視時間而定，使用者可能會在從快取中移除頁面後，以及新頁面發佈前，請求頁面。 AEM現在會傳回舊頁面，而Dispatcher會再次快取它。 對於大型網站而言，這更是個問題。

## 從發佈實例中使Dispatcher Cache失效 {#invalidating-dispatcher-cache-from-a-publishing-instance}

在某些情況下，將快取管理從製作環境傳輸至發佈執行個體，可提升效能。 接著，當收到發佈頁面時，會將快取失效要求傳送給Dispatcher的發佈環境（而非AEM製作環境）。

此類情況包括：

<!-- 

Comment Type: draft

<p>Cache invalidation requests for a page are also generated for any aliases or vanity URLs that are configured in the page properties. </p>

 -->

* 防止Dispatcher和發佈實例之間可能發生的定時衝突(請參 [閱「編寫環境」中的Invalidating Dispatchercache](#invalidating-dispatcher-cache-from-the-authoring-environment))。
* 此系統包含數個駐留在高效能伺服器上的發佈執行個體，以及僅一個編寫執行個體。

>[!NOTE]
>
>使用此方法的決定應由經驗豐富的AEM管理員決定。

調度器刷新由運行在發佈實例上的複製代理控制。 不過，此配置是在編寫環境中進行的，然後通過激活代理進行傳輸：

1. 開啟AEM Tools主控台。
1. 在發佈時，在「工具／複製／代理」下開啟所需的複製代理。 您可以使用預設安裝的Dispatcher Flush代理。
1. 按一下「編輯」，然後在「設定」標籤中確定已選 **取「啟** 用」。
1. （可選）要啟用別名或虛名路徑失效請求，請選擇「別 **名更新** 」選項。
1. 在「傳輸」頁籤上，輸入訪問Dispatcher所需的URI。\
   如果您使用標準Dispatcher Flush代理，則可能需要更新主機名和埠；例如， `http://<dispatcherHost>:<portApache>/dispatcher/invalidate.cache`

   **注意：** 對於Dispatcher Flush代理，僅當使用基於路徑的虛擬主機條目來區分場時，才使用URI屬性。 使用此欄位可以定位要失效的場。 例如，場#1的虛擬主機為，場#2 `www.mysite.com/path1/*` 的虛擬主機為 `www.mysite.com/path2/*`。 您可以使用的URL來 `/path1/invalidate.cache` 定位第一個群體， `/path2/invalidate.cache` 以及定位第二個群體。 如需詳細資訊，請參 [閱使用Dispatcher搭配多個網域](dispatcher-domains.md)。

1. 視需要設定其他參數。
1. 對每個受影響的發佈例項重複此步驟。

配置後，當您從作者啟動頁面以進行發佈時，此代理會啟動標準複製。 記錄檔包含指示來自您發佈伺服器的請求的訊息，類似下列範例：

1. `<publishserver> 13:29:47 127.0.0.1 POST /dispatcher/invalidate.cache 200`

## 手動使Dispatcher快取失效 {#manually-invalidating-the-dispatcher-cache}

要使Dispatcher快取無效（或刷新）而不激活頁，可以向調度程式發出HTTP請求。 例如，您可以建立AEM應用程式，讓管理員或其他應用程式清除快取。

HTTP請求會使Dispatcher從快取中刪除特定檔案。 （可選）Dispatcher然後使用新副本刷新快取。

### 刪除快取檔案 {#delete-cached-files}

發出HTTP請求，使Dispatcher從快取中刪除檔案。 Dispatcher僅在收到頁面的客戶端請求時才會再次快取檔案。 以此方式刪除快取檔案，對於不太可能同時收到相同頁面要求的網站而言是適當的。

HTTP請求有下列格式：

```xml
POST /dispatcher/invalidate.cache HTTP/1.1  
CQ-Action: Activate  
CQ-Handle: path-pattern
Content-Length: 0
```

Dispatcher會刷新（刪除）名稱與標題值匹配的快取檔案和 `CQ-Handler` 資料夾。 例如，有一個 `CQ-Handle` 項 `/content/geomtrixx-outdoors/en` 目符合下列項目：

* 目錄中命名的所有檔案(任何副 `en` 檔名) `geometrixx-outdoors`

* en目錄下的任 `_jcr_content`何名為&quot;&quot;的目錄（如果存在，則包含頁面子節點的快取轉譯）

通過觸摸檔案，調度器快取中的所有其他檔案(或最高到特定級別，取決於 `/statfileslevel` 設定)都將失效 `.stat` 。 此檔案的上次修改日期與快取文檔的上次修改日期進行比較，如果檔案較新，則重新提 `.stat` 取文檔。 有關詳 [細資訊，請參閱按資料夾級別使檔案失效](dispatcher-configuration.md#main-pars_title_26) 。

傳送額外的標題可防止失效（即。stat檔案的觸碰） `CQ-Action-Scope: ResourceOnly`。 這可用於刷新特定資源而不使快取的其他部分失效，例如動態建立並需要定期刷新的JSON資料（例如，表示從第三方系統獲取的資料以顯示新聞、股票行情等）。

### 刪除和重新快取檔案 {#delete-and-recache-files}

發出HTTP請求，使Dispatcher刪除快取的檔案，並立即檢索和重新快取檔案。 當網站可能同時收到同一頁面的用戶端要求時，請刪除檔案並立即重新快取。 立即重新快取可確保Dispatcher只檢索和快取頁面一次，而不是對每個同步客戶端請求一次。

**注意：** 刪除和重新快取檔案應僅在發佈實例上執行。 從作者實例執行時，當重新快取資源的嘗試在發佈之前發生時，就會發生競爭條件。

HTTP請求有下列格式：

```xml
POST /dispatcher/invalidate.cache HTTP/1.1  
CQ-Action: Activate   
`Content-Type: text/plain  
CQ-Handle: path-pattern  
Content-Length: numchars in bodypage_path0

page_path1 
...  
page_pathn
```

要立即重新快取的頁面路徑會列在消息正文的單獨行上。 值是使 `CQ-Handle` 要重新快取的頁面無效的頁面路徑。 (請參 `/statfileslevel` 閱快取 [設定項](dispatcher-configuration.md#main-pars_146_44_0010) 。)下列範例HTTP要求訊息會刪除並重新宣告 `/content/geometrixx-outdoors/en.html page`:

```xml
POST /dispatcher/invalidate.cache HTTP/1.1  
CQ-Action: Activate  
Content-Type: text/plain   
CQ-Handle: /content/geometrixx-outdoors/en/men.html  
Content-Length: 36

/content/geometrixx-outdoors/en.html
```

### 刷新servlet示例 {#example-flush-servlet}

以下代碼實現了向Dispatcher發送無效請求的servlet。 Servlet接收包含和參數的請 `handle` 求消 `page` 息。 這些參數分別提供 `CQ-Handle` 要重新快取的頁首和路徑的值。 Servlet使用這些值來構建Dispatcher的HTTP請求。

將servlet部署到發佈實例時，以下URL會使Dispatcher刪除/content/geometrixx-outdoors/en.html頁，然後快取新副本。

`10.36.79.223:4503/bin/flushcache/html?page=/content/geometrixx-outdoors/en.html&handle=/content/geometrixx-outdoors/en/men.html`

>[!NOTE]
>
>此示例servlet不安全，僅演示了HTTP Post請求消息的使用。 您的解決方案應確保對servlet的訪問安全。


```java
package com.adobe.example;

import org.apache.felix.scr.annotations.Component;
import org.apache.felix.scr.annotations.Service;
import org.apache.felix.scr.annotations.Property;

import org.apache.sling.api.SlingHttpServletRequest;
import org.apache.sling.api.SlingHttpServletResponse;
import org.apache.sling.api.servlets.SlingSafeMethodsServlet;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import org.apache.commons.httpclient.*;
import org.apache.commons.httpclient.methods.PostMethod;
import org.apache.commons.httpclient.methods.StringRequestEntity;

@Component(metatype=true)
@Service
public class Flushcache extends SlingSafeMethodsServlet {

 @Property(value="/bin/flushcache")
 static final String SERVLET_PATH="sling.servlet.paths";

 private Logger logger = LoggerFactory.getLogger(this.getClass());

 public void doGet(SlingHttpServletRequest request, SlingHttpServletResponse response) {
  try{ 
      //retrieve the request parameters
      String handle = request.getParameter("handle");
      String page = request.getParameter("page");

      //hard-coding connection properties is a bad practice, but is done here to simplify the example
      String server = "localhost"; 
      String uri = "/dispatcher/invalidate.cache";

      HttpClient client = new HttpClient();

      PostMethod post = new PostMethod("https://"+host+uri);
      post.setRequestHeader("CQ-Action", "Activate");
      post.setRequestHeader("CQ-Handle",handle);
   
      StringRequestEntity body = new StringRequestEntity(page,null,null);
      post.setRequestEntity(body);
      post.setRequestHeader("Content-length", String.valueOf(body.getContentLength()));
      client.executeMethod(post);
      post.releaseConnection();
      //log the results
      logger.info("result: " + post.getResponseBodyAsString());
      }
  }catch(Exception e){
      logger.error("Flushcache servlet exception: " + e.getMessage());
  }
 }
}
```

