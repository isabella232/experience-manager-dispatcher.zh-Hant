---
title: 快取保全內容
seo-title: 在AEM Dispatcher中快取保全內容
description: 瞭解權限感應快取如何在Dispatcher中運作。
seo-description: 瞭解權限感應快取如何在AEM Dispatcher中運作。
uuid: abed68a-2efe-45f6-bdf7-2284931629d6
contentOwner: 使用者
products: SG_ PERIENCENCENAGER/ADDER
topic-tags: dispatcher
content-type: 引用
discoiquuid: 4f9b2bc8-a309-47bc-b70 d-a1 c0 da78 d464
translation-type: tm+mt
source-git-commit: 8dd56f8b90331f0da43852e25893bc6f3e606a97

---


# 快取保全內容 {#caching-secured-content}

權限感應快取可讓您快取安全頁面。Dispatcher會檢查使用者在傳送快取頁面之前的存取權限。

Dispatcher包含可實施權限感應快取的authCheker模組。啓動模組時，演算會呼叫AEM Servlet，以執行要求的內容的使用者驗證和授權。servlet回應會決定內容是否傳送至網頁瀏覽器。

由於驗證和授權方法是AEM部署的專屬方式，所以必須建立servlet。

>[!NOTE]
>
>使用 `deny` 篩選器來強制執行外掛程式的安全性限制。針對設定為允許存取使用者或群組子集的頁面，使用權限感應快取。

下列圖表說明當網頁瀏覽器請求使用權限感應快取的頁面時，發生的事件順序。

## 已快取頁面並授權使用者 {#page-is-cached-and-user-is-authorized}

![](assets/chlimage_1.png)

1. Dispatcher會判斷要求的內容快取和有效。
1. Dispatcher會傳送要求訊息給演算。HEAD區段包含瀏覽器請求中的所有標題行。
1. 演算程式會呼叫授權者執行安全性檢查並回應Dispatcher。回應訊息包含200個HTTP狀態碼，指出使用者已獲得授權。
1. Dispatcher會傳送回應訊息給瀏覽器，其中包含來自轉譯回應和內文快取內容的標題行。

## 未快取頁面，且使用者已獲得授權 {#page-is-not-cached-and-user-is-authorized}

![](assets/chlimage_1-1.png)

1. Dispatcher會判斷內容未快取或需要更新。
1. Dispatcher會將原始要求轉送至演算。
1. 演算會呼叫authorizer servlet以執行安全檢查。當使用者獲得授權時，演算會包含回應訊息內文中的轉譯頁面。
1. Dispatcher會將回應轉送至瀏覽器。Dispatcher會將轉譯回應訊息的內文新增至快取。

## 未授權使用者 {#user-is-not-authorized}

![](assets/chlimage_1-2.png)

1. Dispatcher會檢查快取。
1. Dispatcher會傳送要求訊息給演算，其中包含瀏覽器要求中所有標題行。
1. 演算會呼叫authorizer servlet，以執行安全性檢查失敗，並將原始請求轉送至Dispatcher。

## 實作權限感應快取 {#implementing-permission-sensitive-caching}

若要實施權限感應快取，請執行下列任務：

* 開發執行驗證和授權的servlet
* 設定Dispatcher

>[!NOTE]
>
>通常，安全資源會儲存在不同的資料夾中，而非不安全的檔案中。例如，/content/secure/


## 建立授權Servlet {#create-the-authorization-servlet}

建立並部署Servlet，可執行要求網頁內容的使用者驗證與授權。servlet可使用任何驗證和授權方法，例如AEM使用者帳戶和儲存庫ACLs或LDAP查閱服務。您將servlet部署至Dispatcher做為演算用途的AEM實例。

所有使用者都必須存取servlet。因此，您的Servlet應該延伸 `org.apache.sling.api.servlets.SlingSafeMethodsServlet` 類別，此類別提供唯讀存取權。

servlet只會傳送轉譯的HEAD請求，因此您只需要實作 `doHead` 方法。

演算包括要求資源的URI，做為HTTP要求的參數。例如，授權Servlet可透過下列方式存取 `/bin/permissioncheck`。若要對/content/geometrixx-outdoors/en.html頁面執行保全檢查，演算會在HTTP要求中包含下列URL：

`/bin/permissioncheck?uri=/content/geometrixx-outdoors/en.html`

servlet回應訊息必須包含下列HTTP狀態代碼：

* 200：通過驗證和授權。

下列範例servlet從HTTP請求取得請求資源的URL。程式碼使用Felix SCR `Property` 備注，將 `sling.servlet.paths` 屬性值設定為/bin/permissioncheck。在 `doHead` 此方法中，servlet會取得工作階段物件，並使用 `checkPermission` 方法判斷適當的回應代碼。

>[!NOTE]
>
>sling. servlet. path屬性的值必須在Sling Servlet Solution(org. apache. sling. servlets. resollist. slingServerResolution)服務中啓用。

### servlet範例 {#example-servlet}

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

## 設定Dispatcher以進行權限區分式快取 {#configure-dispatcher-for-permission-sensitive-caching}

dispatcher的auth_ checker區段。任何檔案都可控制權限感應快取的行為。auth_ checker區段包含下列子區域：

* `url`：執行安全性檢查之servlet `sling.servlet.paths` 屬性的值。

* `filter`：指定套用權限感應快取之資料夾的篩選器。`deny` 通常，篩選器會套用至所有資料夾， `allow` 而篩選器會套用至安全資料夾。

* `headers`：指定授權Servlet在回應中包含的HTTP標題。

當Dispatcher開始時，Dispatcher記錄檔包含下列除錯層級訊息：

`AuthChecker: initialized with URL 'configured_url'.`

下列範例auth_ checker區段會設定Dispatcher使用prevoius主題的servlet。篩選區段會讓權限檢查只對安全的HTML資源執行。

### 範例組態 {#example-configuration}

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

