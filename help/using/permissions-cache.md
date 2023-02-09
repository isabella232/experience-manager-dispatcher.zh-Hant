---
title: 快取安全內容
seo-title: Caching Secured Content in AEM Dispatcher
description: 了解權限敏感型快取如何在 Dispatcher 中運作。
seo-description: Learn how permission-sensitive caching works in AEM Dispatcher.
uuid: abfed68a-2efe-45f6-bdf7-2284931629d6
contentOwner: User
products: SG_EXPERIENCEMANAGER/DISPATCHER
topic-tags: dispatcher
content-type: reference
discoiquuid: 4f9b2bc8-a309-47bc-b70d-a1c0da78d464
exl-id: 3d8d8204-7e0d-44ad-b41b-6fec2689c6a6
source-git-commit: 31eaa42b17838d97cacd5c535e04be01a3eb6807
workflow-type: ht
source-wordcount: '918'
ht-degree: 100%

---

# 快取安全內容 {#caching-secured-content}

權限敏感型快取可讓您快取安全頁面。 Dispatcher 在傳遞快取頁面之前會檢查使用者對頁面的存取權限。

Dispatcher 包含的 AuthChecker 模組會實作權限敏感型快取。 在啟用此模組時，Dispatcher 會呼叫 AEM servlet 來針對請求的內容執行使用者驗證和授權。 此 servlet 回應會決定是否從快取中將內容傳遞給網頁瀏覽器。

由於驗證和授權的方法為 AEM 部署所特有，所以您需要建立此 servlet。

>[!NOTE]
>
>使用 `deny` 篩選條件可強制施行安全限制。 請將權限敏感型快取用於設定為允許存取一部分使用者或群組的頁面。

下圖說明當網頁瀏覽器請求使用了權限敏感型快取的頁面時，發生的事件順序。

## 快取頁面並授權使用者 {#page-is-cached-and-user-is-authorized}

![](assets/chlimage_1.png)

1. Dispatcher 判斷請求的內容已快取且有效。
1. Dispatcher 傳送請求訊息給轉譯器。 HEAD 區段包含來自瀏覽器請求的所有標題行。
1. 轉譯器呼叫 Auth Checker servlet 來執行安全性檢查，並回應 Dispatcher。 回應訊息包含 HTTP 狀態代碼 200，表示已授權使用者。
1. Dispatcher 傳送回應訊息給瀏覽器，該訊息包含來自轉譯器回應的標題行以及內文中的快取內容。

## 不快取頁面，但會授權使用者 {#page-is-not-cached-and-user-is-authorized}

![](assets/chlimage_1-1.png)

1. Dispatcher 判斷內容未快取或需要更新。
1. Dispatcher 將原始請求轉送給轉譯器。
1. 轉譯器呼叫 AEM 授權程式 servlet (這不是 Dispatcher AuthChcker servlet) 來執行安全性檢查。 當使用者獲得授權時，轉譯器會將轉譯的頁面納入回應訊息的內文中。
1. Dispatcher 將回應轉送給瀏覽器。 Dispatcher 將轉譯器的回應訊息內文新增到快取中。

## 使用者未獲得授權 {#user-is-not-authorized}

![](assets/chlimage_1-2.png)

1. Dispatcher 檢查快取。
1. Dispatcher 傳送請求訊息給轉譯器，該訊息包含來自瀏覽器的請求的所有標題行。
1. 轉譯器呼叫 Auth Checker servlet 來執行安全性檢查，結果檢查失敗，然後轉譯器將原始請求轉送給 Dispatcher。
1. Dispatcher 將原始請求轉送給轉譯器。
1. 轉譯器呼叫 AEM 授權程式 servlet (這不是 Dispatcher AuthChcker servlet) 來執行安全性檢查。 當使用者獲得授權時，轉譯器會將轉譯的頁面納入回應訊息的內文中。
1. Dispatcher 將回應轉送給瀏覽器。 Dispatcher 將轉譯器的回應訊息內文新增到快取中。

## 實作權限敏感型快取 {#implementing-permission-sensitive-caching}

若要實作權限敏感型快取，請執行以下工作：

* 開發一個 servlet 來執行驗證和授權
* 設定 Dispatcher

>[!NOTE]
>
>通常安全資源會儲存在與不安全檔案不同的資料夾中。 例如，/content/secure/

>[!NOTE]
>
>當 Dispatcher 前面有 CDN (或任何其他快取) 時，您應該據此設定快取標頭，使 CDN 不至於快取私人內容。例如：`Header always set Cache-Control private`。
>對於 AEM as a Cloud Service，請參閱[快取](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/content/implementing/content-delivery/caching.html)頁面，了解更多有關如何設定私人快取標頭的詳細資訊。

## 建立 Auth Checker servlet {#create-the-auth-checker-servlet}

建立並部署一個 servlet 來針對請求網頁內容的使用者執行驗證和授權工作。 此 servlet 可使用任何驗證和授權方法，例如 AEM 使用者帳戶和存放庫 ACL 或是 LDAP 查詢服務。 您將此 servlet 部署到 Dispatcher 當作轉譯器使用的 AEM 執行個體。

此 servlet 必須可供所有使用者存取。 因此，您的 servlet 應該擴充 `org.apache.sling.api.servlets.SlingSafeMethodsServlet` 類別，以提供對系統的唯讀存取權。

此 servlet 只會從轉譯器接收 HEAD 請求，所以您只需要實作 `doHead` 方法。

轉譯器包含請求的資源的 URI 以當作 HTTP 請求的參數。 例如，授權 servlet 是透過 `/bin/permissioncheck` 進行存取。 為了在 /content/geometrixx-outdoors/en.html 頁面上執行安全性檢查，轉譯器在 HTTP 請求中包含以下 URL：

`/bin/permissioncheck?uri=/content/geometrixx-outdoors/en.html`

servlet 回應訊息必須包含以下 HTTP 狀態代碼：

* 200：已通過驗證和授權。

以下範例 servlet 會從 HTTP 請求中取得請求的資源的 URL。 程式碼會使用 Felix SCR `Property` 註解，將 `sling.servlet.paths` 屬性的值設為 /bin/permissioncheck。 在 `doHead` 方法中，此 servlet 取得工作階段物件，並使用 `checkPermission` 方法來判斷合適的回應代碼。

>[!NOTE]
>
>sling.servlet.paths 屬性的值必須在 Sling Servlet Resolver (org.apache.sling.servlets.resolver.SlingServletResolver) 服務中啟用。

### 範例 servlet {#example-servlet}

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

import javax.jcr.Session;

@Component(metatype=false)
@Service
public class AuthcheckerServlet extends SlingSafeMethodsServlet {
 
    @Property(value="/bin/permissioncheck")
    static final String SERVLET_PATH="sling.servlet.paths";
    
    private Logger logger = LoggerFactory.getLogger(this.getClass());
    
    public void doHead(SlingHttpServletRequest request, SlingHttpServletResponse response) {
     try{ 
      //retrieve the requested URL
      String uri = request.getParameter("uri");
      //obtain the session from the request
      Session session = request.getResourceResolver().adaptTo(javax.jcr.Session.class);     
      //perform the permissions check
      try {
       session.checkPermission(uri, Session.ACTION_READ);
       logger.info("authchecker says OK");
       response.setStatus(SlingHttpServletResponse.SC_OK);
      } catch(Exception e) {
       logger.info("authchecker says READ access DENIED!");
       response.setStatus(SlingHttpServletResponse.SC_FORBIDDEN);
      }
     }catch(Exception e){
      logger.error("authchecker servlet exception: " + e.getMessage());
     }
    }
}
```

## 設定 Dispatcher 使用權限敏感型快取 {#configure-dispatcher-for-permission-sensitive-caching}

>[!NOTE]
>
>如果您的要求允許快取經驗證的文件，請將 /cache 部分下的 /allowAuthorized 屬性設定為 `/allowAuthorized 1`。有關詳細資訊，請參閱[使用驗證時快取](/help/using/dispatcher-configuration.md)。

dispatcher.any 檔案的 auth_checker 區段會控制權限敏感型快取的行為。 auth_checker 區段包含以下子區段：

* `url`：執行安全性檢查的 servlet 的 `sling.servlet.paths` 屬性值。

* `filter`：指定要將權限敏感型快取套用到哪些資料夾的篩選條件。 通常 `deny` 篩選條件會套用到所有資料夾，而 `allow` 篩選條件則套用到安全資料夾。

* `headers`：指定授權 servlet 包含在回應中的 HTTP 標題。

當 Dispatcher 啟動時，Dispatcher 記錄檔會包含以下偵錯層級的訊息：

`AuthChecker: initialized with URL 'configured_url'.`

以下範例的 auth_checker 區段會設定 Dispatcher 使用前一個主題中的 servlet。 篩選區段會讓權限檢查工作僅針對安全 HTML 資源來執行。

### 設定範例 {#example-configuration}

```xml
/auth_checker
  {
  # request is sent to this URL with '?uri=<page>' appended
  /url "/bin/permissioncheck"
      
  # only the requested pages matching the filter section below are checked,
  # all other pages get delivered unchecked
  /filter
    {
    /0000
      {
      /glob "*"
      /type "deny"
      }
    /0001
      {
      /glob "/content/secure/*.html"
      /type "allow"
      }
    }
  # any header line returned from the auth_checker's HEAD request matching
  # the section below will be returned as well
  /headers
    {
    /0000
      {
      /glob "*"
      /type "deny"
      }
    /0001
      {
      /glob "Set-Cookie:*"
      /type "allow"
      }
    }
  }
```
