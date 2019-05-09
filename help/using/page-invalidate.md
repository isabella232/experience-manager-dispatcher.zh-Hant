---
title: 從AEM停用快取頁面
seo-title: 從Adobe AEM停用快取頁面
description: 瞭解如何設定Dispatcher和AEM之間的互動，以確保快取管理。
seo-description: 瞭解如何設定Adobe AEM Dispatcher和AEM之間的互動，以確保快取管理。
uuid: 66533299-55c0-4864-9Beb-77e281af9359
cmgrlastmodified: 01.11.2007082929[ahheimoz]
pageversionid: '1193211344162'
template: /apps/docs/templates/contentpage
contentOwner: 使用者
products: SG_ PERIENCENCENAGER/ADDER
topic-tags: dispatcher
content-type: 引用
discoiquuid: 79cd94be-a6 bc-4d34-bfe9-393b4107925 c
translation-type: tm+mt
source-git-commit: 76cffbfb616cd5601aed36b7076f67a2faf3ed3b

---


# 從AEM停用快取頁面 {#invalidating-cached-pages-from-aem}

搭配使用Dispatcher與AEM時，必須設定互動，以確保快取管理。視您的環境而定，此組態也可以提高效能。

## 設定AEM使用者帳戶 {#setting-up-aem-user-accounts}

預設 `admin` 使用者帳戶可用來驗證預設安裝的複製代理程式。您應建立專用的使用者帳戶，以便與複製代理程式搭配使用。 [](https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/security-checklist.html#VerificationSteps)

如需詳細資訊，請參閱AEM Security Checklist [的「Configure Replication and Transport Users](https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/security-checklist.html#VerificationSteps) 」區段。

## 從編寫環境中停用Dispatcher快取 {#invalidating-dispatcher-cache-from-the-authoring-environment}

AEM作者實例上的複製代理程式會在發佈頁面時傳送快取失效請求至Dispatcher。當發佈新內容時，此請求會使Dispatcher最終重新整理快取中的檔案。

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

使用下列程序，在AEM作者實例上設定複製代理程式，以便在啓用頁面時使Dispatcher快取失效：

1. 開啓AEM Tools主控台。`https://localhost:4502/miscadmin#/etc`()
1. 在作者的「工具/複製/代理程式」下方開啓必要的複製代理程式。您可以使用預設安裝的Dispatcher刷新代理程式。
1. 按一下「編輯」，然後在「設定」標籤中確定已選取 **「已啓用** 」。

1. (選擇性)若要啓用別名或虛名路徑無效請求，請選取 **別名更新** 選項。
1. 在「傳輸」標籤上，輸入存取Dispatcher所需的URI。\
   如果您使用標準Dispatcher刷新代理程式，可能需要更新主機名稱和連接埠；例如，https://&lt;*DispatcherHost*&gt;：&lt;*portAPache*&gt;/dispatcher/alignate. cache

   **注意：** 對於Dispatcher Cloud代理，URI屬性僅在您使用路徑架構的虛擬化主機項目來區分農場時使用。您使用此欄位來定位農場使其失效。例如，農場#為虛擬主機， `www.mysite.com/path1/*` 農場#則為虛擬主機 `www.mysite.com/path2/*`。您可以使用URL `/path1/invalidate.cache` 來定位第一個農場， `/path2/invalidate.cache` 並定位第二個農場。如需詳細資訊，請參閱 [「搭配多個網域使用Dispatcher](dispatcher-domains.md)」。

1. 視需要設定其他參數。
1. 按一下「確定」啓用代理程式。

或者，您也可以從AEM Touch UI存取及設定Dispatcher [Seap代理](https://helpx.adobe.com/experience-manager/6-2/sites/deploying/using/replication.html#ConfiguringaDispatcherFlushagent)。

如需如何啓用虛名URL存取權的其他詳細資訊，請參閱 [啓用對虛名URL](dispatcher-configuration.md#enabling-access-to-vanity-urls-vanity-urls)的存取。

>[!NOTE]
>
>Flooning Dispatcher快取的代理程式不需要使用者名稱和密碼，但設定後，將會隨基本驗證傳送。

此方法有兩個潛在問題：

* Dispatcher必須可從編寫例項中存取。如果您的網路(例如防火牆)設定為限制兩者之間的存取，則可能不是個案。

* 出版物和快取失效同時發生。視時間而定，使用者只會在從快取移除頁面，以及在新頁面發佈之前請求頁面。AEM現在會傳回舊頁面，而Dispatcher會再次快取。這是大型網站的問題。

## 從發佈例項停用Dispatcher快取 {#invalidating-dispatcher-cache-from-a-publishing-instance}

在某些情況下，可以將快取管理從編寫環境傳輸至發佈例項來進行效能提升。然後會是發佈頁面時，傳送快取失效要求至Dispatcher的發佈環境(而非AEM編寫環境)。

這些情況包括：

<!-- 

Comment Type: draft

<p>Cache invalidation requests for a page are also generated for any aliases or vanity URLs that are configured in the page properties. </p>

 -->

* 避免Dispatcher和發佈例項之間的時間衝突(請參閱 [「編寫環境](#invalidating-dispatcher-cache-from-the-authoring-environment)」中的「無效的Dispatcher快取」)。
* 此系統包含數個位於高效能伺服器上的發佈實例，僅包含一個編寫例項。

>[!NOTE]
>
>使用此方法的決定應由經驗豐富的AEM管理員進行。

dispatcher刷新是由在發佈例項上操作的複製代理所控制。不過，設定是在編寫環境上進行，然後透過啓動代理程式進行轉讓：

1. 開啓AEM Tools主控台。
1. 在發佈時，在「工具/複製/代理程式」下方開啓必要的複製代理程式。您可以使用預設安裝的Dispatcher刷新代理程式。
1. 按一下「編輯」，然後在「設定」標籤中確定已選取 **「已啓用** 」。
1. (選擇性)若要啓用別名或虛名路徑無效請求，請選取 **別名更新** 選項。
1. 在「傳輸」標籤上，輸入存取Dispatcher所需的URI。\
   如果您使用標準Dispatcher刷新代理程式，可能需要更新主機名稱和連接埠；例如， `http://<dispatcherHost>:<portApache>/dispatcher/invalidate.cache`

   **注意：** 對於Dispatcher Cloud代理，URI屬性僅在您使用路徑架構的虛擬化主機項目來區分農場時使用。您使用此欄位來定位農場使其失效。例如，農場#為虛擬主機， `www.mysite.com/path1/*` 農場#則為虛擬主機 `www.mysite.com/path2/*`。您可以使用URL `/path1/invalidate.cache` 來定位第一個農場， `/path2/invalidate.cache` 並定位第二個農場。如需詳細資訊，請參閱 [「搭配多個網域使用Dispatcher](dispatcher-domains.md)」。

1. 視需要設定其他參數。
1. 針對每個受影響的發佈例項進行重復。

設定後，當您從作者啓動頁面以發佈時，此代理程式會起始標準複製。The log include messages indicates requests comes from your publish server(類似下列範例：

1. `<publishserver> 13:29:47 127.0.0.1 POST /dispatcher/invalidate.cache 200`

## 手動使Dispatcher快取失效 {#manually-invalidating-the-dispatcher-cache}

若要停用(或清除) Dispatcher快取，無須啓用頁面，您可以對傳送程式發出HTTP要求。例如，您可以建立AEM應用程式，讓管理員或其他應用程式可以清除快取。

HTTP要求會使Dispatcher刪除快取中的特定檔案。或者，Dispatcher會使用新復本重新整理快取。

### 刪除快取的檔案 {#delete-cached-files}

發出HTTP要求，使Dispatcher從快取中刪除檔案。Dispatcher只有在收到頁面的用戶端要求時，才會再次快取檔案。刪除此方式的快取檔案適合不可能同時收到相同頁面要求的網站。

HTTP要求具有下列表格：

```xml
POST /dispatcher/invalidate.cache HTTP/1.1  
CQ-Action: Activate  
CQ-Handle: path-pattern
Content-Length: 0
```

Dispatcher會將名稱符合 `CQ-Handler` 標題值的快取檔案和資料夾刪除(刪除)。例如，a `CQ-Handle` 與 `/content/geomtrixx-outdoors/en` 下列項目相符：

* 目錄中命名 `en` 的所有檔案(副 `geometrixx-outdoors` 檔名)

* 位於en目錄下方的任何名為「 `_jcr_content`」的目錄(如果存在，則包含頁面的子節點的快取)

dispatcher快取中的所有其他檔案(或最高等級的特定層級 `/statfileslevel` )都會透過觸控 `.stat` 檔案失效。此檔案的最後修改日期會與快取文件的最後一個修改日期進行比較，如果 `.stat` 檔案較新，則會重新擷取文件。如需詳細資訊，請參閱 [「資料夾](dispatcher-configuration.md#main-pars_title_26) 層級無效」。

傳送額外的標題 `CQ-Action-Scope: ResourceOnly`時，無法將失效(亦即touch. stat檔案)加以防止。這可用來清除特定資源，而不會使快取的其他部分失效，例如動態建立的JSON資料，而且需要定期刷新快取(例如，代表從第三方系統取得的資料，以顯示新聞、股票行情等)。

### 刪除和重新ache檔案 {#delete-and-recache-files}

發出HTTP要求，使Dispatcher刪除快取的檔案，並立即擷取並重新切入檔案。當網站可能同時收到相同頁面的用戶端要求時，刪除並立即重新快取檔案。立即重新傳送可確保Dispatcher只擷取一次並快取頁面一次，而不是針對同時每個用戶端請求一次。

**注意：** 刪除和重新存取檔案只能在發佈例項上執行。從作者執行個體執行時，會在嘗試重新呼叫資源之前發生比賽條件。

HTTP要求具有下列表格：

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

頁面路徑會立即出現在訊息內文的個別行上。頁面的 `CQ-Handle` 值會使頁面失效，使頁面失效。(請參閱 `/statfileslevel`[快取](dispatcher-configuration.md#main-pars_146_44_0010) 設定項目的參數)。以下範例HTTP請求訊息會刪除並重新解密 `/content/geometrixx-outdoors/en.html page`：

```xml
POST /dispatcher/invalidate.cache HTTP/1.1  
CQ-Action: Activate  
Content-Type: text/plain   
CQ-Handle: /content/geometrixx-outdoors/en/men.html  
Content-Length: 36

/content/geometrixx-outdoors/en.html
```

### 範例刷新servlet {#example-flush-servlet}

下列程式碼會實作可傳送無效要求給Dispatcher的servlet。servlet會接收包含 `handle` 和 `page` 參數的請求訊息。這些參數會分別提供 `CQ-Handle` 頁首的值和頁面路徑。servlet使用值來建構Dispatcher的HTTP要求。

將servlet部署至發佈例項時，下列URL會導致Dispatcher刪除/content/geometrixx-outdoors/en.html頁面，然後快取新的副本。

`10.36.79.223:4503/bin/flushcache/html?page=/content/geometrixx-outdoors/en.html&handle=/content/geometrixx-outdoors/en/men.html`

>[!NOTE]
>
>此範例servlet不安全，僅顯示使用HTTP貼文要求訊息。您的解決方案應安全地存取servlet。


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

