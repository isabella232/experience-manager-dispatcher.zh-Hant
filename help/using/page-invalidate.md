---
title: 使 AEM 快取的頁面失效
seo-title: 使來自AdobeAEM的快取頁面失效
description: 了解如何設定Dispatcher與AEM之間的互動，以確保有效的快取管理。
seo-description: 了解如何設定AdobeAEM Dispatcher與AEM之間的互動，以確保有效管理快取。
uuid: 66533299-55c0-4864-9beb-77e281af9359
cmgrlastmodified: 01.11.2007 08 22 29 [aheimoz]
pageversionid: 1193211344162
template: /apps/docs/templates/contentpage
contentOwner: User
products: SG_EXPERIENCEMANAGER/DISPATCHER
topic-tags: dispatcher
content-type: reference
discoiquuid: 79cd94be-a6bc-4d34-bfe9-393b4107925c
exl-id: 90eb6a78-e867-456d-b1cf-f62f49c91851
source-git-commit: 3a0e237278079a3885e527d7f86989f8ac91e09d
workflow-type: tm+mt
source-wordcount: '1427'
ht-degree: 0%

---

# 使 AEM 快取的頁面失效 {#invalidating-cached-pages-from-aem}

搭配AEM使用Dispatcher時，必須設定互動以確保有效管理快取。 視您的環境而定，設定也可以提高效能。

## 設定AEM使用者帳戶{#setting-up-aem-user-accounts}

預設`admin`用戶帳戶用於驗證預設安裝的複製代理。 您應建立專用的使用者帳戶以與復寫代理搭配使用。

如需詳細資訊，請參閱AEM安全性檢查清單的[設定復寫和傳輸使用者](https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/security-checklist.html#VerificationSteps)區段。

## 使編寫環境{#invalidating-dispatcher-cache-from-the-authoring-environment}中的Dispatcher快取失效

AEM製作例項上的復寫代理會在頁面發佈時，傳送快取無效請求給Dispatcher。 請求會使Dispatcher在發佈新內容時，最終重新整理快取中的檔案。

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

請依照下列步驟，在AEM製作例項上設定復寫代理，以在頁面啟動時使Dispatcher快取失效：

1. 開啟AEM工具主控台。(`https://localhost:4502/miscadmin#/etc`)
1. 在作者上的工具/復寫/代理下開啟所需的復寫代理。 您可以使用預設安裝的Dispatcher排清代理程式。
1. 按一下「編輯」，然後在「設定」標籤中確定已選取&#x200B;**Enabled**。

1. （選用）若要啟用別名或虛名路徑無效請求，請選取「別名更新&#x200B;**」選項。**
1. 在「傳輸」標籤上，輸入存取Dispatcher所需的URI。\
   如果您使用標準Dispatcher排清代理，則可能需要更新主機名稱和埠；例如， https://&lt;*dispatcherHost*>:&lt;*portApache*>/dispatcher/invalidate.cache

   **注意：** 對於Dispatcher排清代理，只有在使用基於路徑的虛擬主機條目來區分伺服器陣列時，才會使用URI屬性。使用此欄位可以將伺服器陣列定位為無效。 例如，場#1的虛擬主機為`www.mysite.com/path1/*` ，場#2的虛擬主機為`www.mysite.com/path2/*`。 您可以使用`/path1/invalidate.cache`的URL來定位第一個伺服器陣列，使用`/path2/invalidate.cache`來定位第二個伺服器陣列。 如需詳細資訊，請參閱[將Dispatcher與多個網域搭配使用](dispatcher-domains.md)。

1. 視需要設定其他參數。
1. 按一下「確定」以激活代理。

或者，您也可以從[AEM觸控式UI](https://helpx.adobe.com/experience-manager/6-2/sites/deploying/using/replication.html#ConfiguringaDispatcherFlushagent)存取及設定Dispatcher排清代理程式。

如需如何啟用虛名URL存取權限的其他詳細資訊，請參閱[啟用虛名URL存取權](dispatcher-configuration.md#enabling-access-to-vanity-urls-vanity-urls)。

>[!NOTE]
>
>排清調度程式快取的代理不必具有用戶名和密碼，但如果配置了，則將使用基本身份驗證發送這些代理。

此方法有兩個潛在問題：

* Dispatcher必須可從製作例項存取。 如果您的網路（例如防火牆）被配置為在兩個之間的訪問受到限制，則情況可能並非如此。

* 發佈和快取失效同時發生。 視時間而定，使用者可能會在頁面從快取中移除後，以及新頁面發佈前請求頁面。 AEM現在會傳回舊頁面，而Dispatcher會再次快取該頁面。 這對大型網站而言更為重要。

## 從發佈執行個體{#invalidating-dispatcher-cache-from-a-publishing-instance}使Dispatcher快取失效

在某些情況下，可將快取管理從製作環境傳輸至發佈執行個體，以提升效能。 接著，接收到已發佈頁面時，會將快取失效請求傳送至Dispatcher的發佈環境(而非AEM製作環境)。

此類情況包括：

<!-- 

Comment Type: draft

<p>Cache invalidation requests for a page are also generated for any aliases or vanity URLs that are configured in the page properties. </p>

 -->

* 防止Dispatcher與發佈例項之間可能的計時衝突（請參閱從製作環境](#invalidating-dispatcher-cache-from-the-authoring-environment)中使Dispatcher快取失效）。[
* 此系統包含多個駐留在高效能伺服器上的發佈執行個體，以及僅一個製作執行個體。

>[!NOTE]
>
>有經驗的AEM管理員應決定使用此方法。

Dispatcher排清是由在發佈執行個體上運作的復寫代理所控制。 但是，配置是在創作環境中進行的，然後通過激活代理來轉移：

1. 開啟AEM工具主控台。
1. 在發佈時在工具/復寫/代理下開啟所需的復寫代理。 您可以使用預設安裝的Dispatcher排清代理程式。
1. 按一下「編輯」，然後在「設定」標籤中確定已選取&#x200B;**Enabled**。
1. （選用）若要啟用別名或虛名路徑無效請求，請選取「別名更新&#x200B;**」選項。**
1. 在「傳輸」標籤上，輸入存取Dispatcher所需的URI。\
   如果您使用標準Dispatcher排清代理，則可能需要更新主機名稱和埠；例如， `http://<dispatcherHost>:<portApache>/dispatcher/invalidate.cache`

   **注意：** 對於Dispatcher排清代理，只有在使用基於路徑的虛擬主機條目來區分伺服器陣列時，才會使用URI屬性。使用此欄位可以將伺服器陣列定位為無效。 例如，場#1的虛擬主機為`www.mysite.com/path1/*` ，場#2的虛擬主機為`www.mysite.com/path2/*`。 您可以使用`/path1/invalidate.cache`的URL來定位第一個伺服器陣列，使用`/path2/invalidate.cache`來定位第二個伺服器陣列。 如需詳細資訊，請參閱[將Dispatcher與多個網域搭配使用](dispatcher-domains.md)。

1. 視需要設定其他參數。
1. 對受影響的每個發佈例項重複。

設定後，當您從作者啟動頁面並發佈時，此代理會起始標準復寫。 記錄檔包含類似下列範例的訊息，指出來自您發佈伺服器的請求：

1. `<publishserver> 13:29:47 127.0.0.1 POST /dispatcher/invalidate.cache 200`

## 手動使Dispatcher快取{#manually-invalidating-the-dispatcher-cache}失效

若要在不啟動頁面的情況下使Dispatcher快取無效（或排清），您可以向Dispatcher發出HTTP請求。 例如，您可以建立AEM應用程式，讓管理員或其他應用程式排清快取。

HTTP要求會使Dispatcher從快取中刪除特定檔案。 接著，Dispatcher會以新副本重新整理快取（選擇性）。

### 刪除快取檔案{#delete-cached-files}

發出HTTP要求，導致Dispatcher從快取中刪除檔案。 Dispatcher只有在收到頁面的用戶端請求時，才會再次快取檔案。 以此方式刪除快取檔案對於不太可能同時收到相同頁面請求的網站來說是適當的。

HTTP要求的格式如下：

```xml
POST /dispatcher/invalidate.cache HTTP/1.1  
CQ-Action: Activate  
CQ-Handle: path-pattern
Content-Length: 0
```

Dispatcher會刷新（刪除）名稱與`CQ-Handler`標頭值相符的快取檔案和資料夾。 例如，`/content/geomtrixx-outdoors/en`的`CQ-Handle`符合下列項目：

* `geometrixx-outdoors`目錄中名為`en`的所有檔案（任何副檔名的檔案）

* en目錄下名為「 `_jcr_content`」的任何目錄（如果存在，則包含頁面子節點的快取呈現）

Dispatcher快取中的所有其他檔案（或最高達特定層級，視`/statfileslevel`設定而定）會透過接觸`.stat`檔案而失效。 此檔案的上次修改日期與快取文檔的上次修改日期進行比較，如果`.stat`檔案較新，則重新獲取文檔。 有關詳細資訊，請參閱[按資料夾級別使檔案失效](dispatcher-configuration.md#main-pars_title_26)。

通過發送附加的標頭`CQ-Action-Scope: ResourceOnly`，可以防止失效（即.stat檔案的觸摸）。 這可用來排清特定資源，而不使快取的其他部分失效，例如動態建立且需要與快取無關的定期排清的JSON資料（例如，代表從協力廠商系統取得的資料，以顯示新聞、股票代號等）。

### 刪除和重新快取檔案{#delete-and-recache-files}

發出HTTP要求，導致Dispatcher刪除快取檔案，並立即擷取和重新快取檔案。 當網站可能收到同一頁面的同時用戶端請求時，請刪除檔案並立即重新快取。 立即重新快取可確保Dispatcher只擷取和快取頁面一次，而非同時處理的每個用戶端請求一次。

**注意：** 只應在發佈執行個體上執行刪除和重新擷取檔案。從製作例項執行時，當嘗試重新快取資源在發佈前發生時，就會發生競爭條件。

HTTP要求的格式如下：

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

要立即重新快取的頁面路徑會列在訊息內文的不同行上。 `CQ-Handle`值是讓頁面無法重新快取的頁面路徑。 （請參閱[Cache](dispatcher-configuration.md#main-pars_146_44_0010)配置項的`/statfileslevel`參數。） 下列範例HTTP要求訊息會刪除並重新呼叫`/content/geometrixx-outdoors/en.html page`:

```xml
POST /dispatcher/invalidate.cache HTTP/1.1  
CQ-Action: Activate  
Content-Type: text/plain   
CQ-Handle: /content/geometrixx-outdoors/en/men.html  
Content-Length: 36

/content/geometrixx-outdoors/en.html
```

### 排清Servlet {#example-flush-servlet}示例

下列程式碼會實作Servlet，此Servlet會傳送無效請求給Dispatcher。 Servlet接收包含`handle`和`page`參數的請求消息。 這些參數分別提供`CQ-Handle`標題的值和要重新快取的頁面路徑。 Servlet會使用值來建構Dispatcher的HTTP要求。

將Servlet部署至發佈例項時，下列URL會使Dispatcher刪除/content/geometrixx-outdoors/en.html頁面，然後快取新副本。

`10.36.79.223:4503/bin/flushcache/html?page=/content/geometrixx-outdoors/en.html&handle=/content/geometrixx-outdoors/en/men.html`

>[!NOTE]
>
>此範例Servlet不安全，僅示範如何使用HTTP Post要求訊息。 您的解決方案應能安全存取servlet。


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
