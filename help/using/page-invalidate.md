---
title: 使 AEM 中的快取頁面失效
seo-title: Invalidating Cached Pages From Adobe AEM
description: 了解如何設定 Dispatcher 與 AEM 之間的互動，以確保有效的快取管理。
seo-description: Learn how to configure the interaction between Adobe AEM Dispatcher and AEM to ensure effective cache management.
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
source-git-commit: 25f8569bdeb6b675038bea02637900e9d0fc1f27
workflow-type: tm+mt
source-wordcount: '1404'
ht-degree: 99%

---

# 使 AEM 中的快取頁面失效 {#invalidating-cached-pages-from-aem}

在搭配 AEM 使用 Dispatcher 時，必須設定互動以確保有效的快取管理。 根據您的環境，此設定也可提高效能。

## 設定 AEM 使用者帳戶 {#setting-up-aem-user-accounts}

系統會使用預設 `admin` 使用者帳戶來驗證預設情況下安裝的複寫代理程式。 您應該建立專用使用者帳戶以搭配複寫代理程式使用。

如需詳細資訊，請參閱 AEM 安全性檢查清單的[設定複寫和傳輸使用者](https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/security-checklist.html#VerificationSteps)一節。

## 使編寫環境中的 Dispatcher 快取失效 {#invalidating-dispatcher-cache-from-the-authoring-environment}

發佈頁面時，AEM 編寫執行個體上的複寫代理程式會傳送快取失效請求給 Dispatcher。 此請求最終會導致 Dispatcher 在發佈新內容時重新整理快取中的檔案。

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

請使用以下程序在 AEM 編寫執行個體上設定複寫代理程式，以便在啟用頁面時讓 Dispatcher 快取失效：

1. 開啟 AEM 工具主控台。 (`https://localhost:4502/miscadmin#/etc`)
1. 開啟編寫執行個體上的 Tools/replication/Agents 底下所需的複寫代理程式。 您可以使用預設情況下安裝的 Dispatcher Flush 代理程式。
1. 按一下「編輯」，然後在「設定」索引標籤中確定選取了&#x200B;**已啟用**。

1. (選擇性) 若要啟用別名或虛名路徑失效請求，請選取&#x200B;**別名更新**&#x200B;選項。
1. 在「傳輸」索引標籤上，輸入存取 Dispatcher 所需的 URI。\
   如果您使用標準 Dispatcher Flush 代理程式，您可能需要更新主機名稱和連接埠；例如 https://&lt;*dispatcherHost*>:&lt;*portApache*>/dispatcher/invalidate.cache

   **注意：**&#x200B;對於 Dispatcher Flush 代理程式，只有當您使用以路徑為根據的虛擬主機項目來區分陣列時，才會使用 URI 屬性。 您會使用此欄位來鎖定要失效的陣列。 例如，陣列 #1 的虛擬主機為 `www.mysite.com/path1/*`，而陣列 #2 的虛擬主機為 `www.mysite.com/path2/*`。 您可以使用 URL `/path1/invalidate.cache` 鎖定第一個陣列，並使用 `/path2/invalidate.cache` 鎖定第二個陣列。 如需詳細資訊，請參閱[在多個網域中使用 Dispatcher](dispatcher-domains.md)。

1. 視需要設定其他參數。
1. 按一下「確定」，啟用代理程式。

或者，您也可以從 [AEM Touch UI](https://experienceleague.adobe.com/docs/experience-manager-65/deploying/configuring/replication.html#configuring-a-dispatcher-flush-agent) 存取及設定 Dispatcher Flush 代理程式。

如需如何啟用對虛名 URL 之存取權的詳細資訊，請參閱[啟用對虛名 URL 的存取權](dispatcher-configuration.md#enabling-access-to-vanity-urls-vanity-urls)。

>[!NOTE]
>
>清除 Dispatcher 快取的代理程式不必有使用者名稱和密碼，但如果有設定的話，將透過基本驗證來傳送。

這個做法可能有兩個問題：

* 必須可以從編寫執行個體聯繫 Dispatcher。 如果您的網路 (例如防火牆) 已設定為限制兩者之間的存取，就可能不是這種情況。

* 發佈及快取失效會在同一時間發生。 根據時間的不同，使用者可能會在頁面從快取中移除之後以及新頁面發佈之前要求該頁面。 AEM 現在會傳回舊頁面，而 Dispatcher 會再次快取該頁面。 這比較是大型網站的問題。

## 使發佈執行個體中的 Dispatcher 快取失效 {#invalidating-dispatcher-cache-from-a-publishing-instance}

在某些情況下，可以從編寫環境將快取管理轉移到發佈執行個體來獲得效能的提升。 然後在收到已發佈的頁面時，會由發佈環境 (而不是 AEM 編寫環境) 傳送快取失效請求給 Dispatcher。

這類情況包括：

<!-- 

Comment Type: draft

<p>Cache invalidation requests for a page are also generated for any aliases or vanity URLs that are configured in the page properties. </p>

 -->

* 避免 Dispatcher 與發佈執行個體之間可能發生的時間衝突 (請參閱[使編寫環境中的 Dispatcher 快取失效](#invalidating-dispatcher-cache-from-the-authoring-environment))。
* 此系統包括位於高效能伺服器上的幾個發佈執行個體，而且只有一個編寫執行個體。

>[!NOTE]
>
>應該由經驗豐富的 AEM 管理員做出使用此方法的決定。

Dispatcher 清除作業是由發佈執行個體上運作的複寫代理程式所控制。 不過，設定是在編寫環境上進行，然後透過啟用代理程式進行傳輸：

1. 開啟 AEM 工具主控台。
1. 開啟發佈執行個體上的 Tools/replication/Agents 底下所需的複寫代理程式。 您可以使用預設情況下安裝的 Dispatcher Flush 代理程式。
1. 按一下「編輯」，然後在「設定」索引標籤中確定選取了&#x200B;**已啟用**。
1. (選擇性) 若要啟用別名或虛名路徑失效請求，請選取&#x200B;**別名更新**&#x200B;選項。
1. 在「傳輸」索引標籤上，輸入存取 Dispatcher 所需的 URI。\
   如果您使用標準 Dispatcher Flush 代理程式，您可能需要更新主機名稱和連接埠；例如 `http://<dispatcherHost>:<portApache>/dispatcher/invalidate.cache`

   **注意：**&#x200B;對於 Dispatcher Flush 代理程式，只有當您使用以路徑為根據的虛擬主機項目來區分陣列時，才會使用 URI 屬性。 您會使用此欄位來鎖定要失效的陣列。 例如，陣列 #1 的虛擬主機為 `www.mysite.com/path1/*`，而陣列 #2 的虛擬主機為 `www.mysite.com/path2/*`。 您可以使用 URL `/path1/invalidate.cache` 鎖定第一個陣列，並使用 `/path2/invalidate.cache` 鎖定第二個陣列。 如需詳細資訊，請參閱[在多個網域中使用 Dispatcher](dispatcher-domains.md)。

1. 視需要設定其他參數。
1. 針對每個受影響的發佈執行個體重複此程序。

在設定後，當您啟用編寫環境中的頁面進行發佈時，此代理程式會起始標準複寫。 記錄中包含的訊息會指示來自您的發佈伺服器的請求，類似於以下範例：

1. `<publishserver> 13:29:47 127.0.0.1 POST /dispatcher/invalidate.cache 200`

## 手動讓 Dispatcher 快取失效 {#manually-invalidating-the-dispatcher-cache}

若要讓 Dispatcher 快取失效 (或將其清除) 而不啟用頁面，您可以發出 HTTP 請求給 Dispatcher。 例如，您可以建立 AEM 應用程式，好讓管理員或其他應用程式可以清除快取。

HTTP 請求會讓 Dispatcher 從快取中刪除特定檔案。 然後 Dispatcher 會選擇性地以新複本重新整理快取。

### 刪除快取檔案 {#delete-cached-files}

發出 HTTP 請求讓 Dispatcher 從快取中刪除檔案。 Dispatcher 只有在收到頁面的用戶端請求時才會再次快取檔案。 以這種方式刪除快取檔案適用於不太可能接收相同頁面的同時請求的網站。

HTTP 請求具有以下形式：

```xml
POST /dispatcher/invalidate.cache HTTP/1.1  
CQ-Action: Activate  
CQ-Handle: path-pattern
Content-Length: 0
```

Dispatcher 會清除 (刪除) 名稱符合 `CQ-Handler` 標頭值的快取檔案和資料夾。 例如，`/content/geomtrixx-outdoors/en` 的 `CQ-Handle` 符合以下項目：

* `geometrixx-outdoors` 目錄中名為 `en` 的所有檔案 (副檔名不限)

* en 目錄底下名為「`_jcr_content`」的任何目錄 (如果存在的話，會包含頁面的子節點的快取呈現)

接觸 `.stat` 檔案會使得 Dispatcher 快取中的其他所有檔案 (或高至特定層級，視 `/statfileslevel` 設定而定) 失效。 將這個檔案的最後修改日期與快取文件的最後修改日期做比較，並在 `.stat` 檔案比較新的時候重新提取該文件。 如需詳細資訊，請參閱[依照資料夾層級讓檔案失效](dispatcher-configuration.md#main-pars_title_26)。

您可以藉由傳送其他標頭 `CQ-Action-Scope: ResourceOnly` 來避免失效 (也就是接觸 .stat 檔案)。 這可用來清除特定資源，而不會讓快取的其他部分 (例如動態建立的 JSON 資料) 失效，並且需要獨立於快取之外的定期清除 (例如，表示從第三方系統取得的資料以顯示新聞、股票行情等)。

### 刪除並重新快取檔案 {#delete-and-recache-files}

發出 HTTP 請求讓 Dispatcher 刪除快取檔案，然後立即擷取並重新快取檔案。 當網站可能接收相同頁面的同時用戶端請求時，刪除並立即重新快取檔案。 立即重新快取可確保 Dispatcher 只會擷取及快取頁面一次，而不是針對每個同時用戶端請求擷取及快取頁面一次。

**注意：**&#x200B;刪除及重新快取檔案應該只能在發佈執行個體上執行。 從編寫執行個體執行時，如果在發佈資源前嘗試重新快取資源，則會發生競爭情況。

HTTP 請求具有以下形式：

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

立即重新快取的頁面路徑會列在訊息本文中的個別字行上。 `CQ-Handle` 的值是讓重新快取的頁面失效的頁面的路徑。 (請參閱[快取](dispatcher-configuration.md#main-pars_146_44_0010)設定項目的 `/statfileslevel` 參數。) 以下範例 HTTP 請求訊息會刪除並重新聯繫 `/content/geometrixx-outdoors/en.html page`：

```xml
POST /dispatcher/invalidate.cache HTTP/1.1  
CQ-Action: Activate  
Content-Type: text/plain   
CQ-Handle: /content/geometrixx-outdoors/en/men.html  
Content-Length: 36

/content/geometrixx-outdoors/en.html
```

### 清除 servlet 範例 {#example-flush-servlet}

以下程式碼會實作一個 servlet，以便將失效請求傳送給 Dispatcher。 此 servlet 會收到包含 `handle` 和 `page` 參數的請求訊息。 這兩個參數分別會提供 `CQ-Handle` 標頭的值以及要重新快取的頁面的路徑。 此 servlet 會使用這些值來為 Dispatcher 建構 HTTP 請求。

將此 servlet 部署到發佈執行個體時，以下 URL 會使 Dispatcher 刪除 /content/geometrixx-outdoors/en.html 頁面然後快取新的複本。

`10.36.79.223:4503/bin/flushcache/html?page=/content/geometrixx-outdoors/en.html&handle=/content/geometrixx-outdoors/en/men.html`

>[!NOTE]
>
>此範例 servlet 並不安全，僅用來示範 HTTP Post 請求訊息的使用。 您的解決方案應該要保護對此 servlet 的存取權。

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
