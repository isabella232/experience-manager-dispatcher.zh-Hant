---
title: 快取安全內容
seo-title: Caching Secured Content in AEM Dispatcher
description: 瞭解權限敏感型快取在Dispatcher中的工作原理。
seo-description: Learn how permission-sensitive caching works in AEM Dispatcher.
uuid: abfed68a-2efe-45f6-bdf7-2284931629d6
contentOwner: User
products: SG_EXPERIENCEMANAGER/DISPATCHER
topic-tags: dispatcher
content-type: reference
discoiquuid: 4f9b2bc8-a309-47bc-b70d-a1c0da78d464
exl-id: 3d8d8204-7e0d-44ad-b41b-6fec2689c6a6
source-git-commit: 753f9fc35968996ee83d5947585fd52f2b981632
workflow-type: tm+mt
source-wordcount: '827'
ht-degree: 0%

---

# 快取安全內容 {#caching-secured-content}

對權限敏感的快取使您能夠快取受保護的頁面。 Dispatcher在傳遞快取頁之前檢查用戶對頁的訪問權限。

Dispatcher包括AuthChecker模組，用於實現對權限敏感的快取。 當該模組被激活時，Dispatcher調用AEMservlet對所請求的內容執行用戶驗證和授權。 Servlet響應確定內容是否從快取傳送到Web瀏覽器。

由於身份驗證和授權方法是特定於部AEM署的，因此需要建立servlet。

>[!NOTE]
>
>使用 `deny` 用於強制實施一攬子安全限制的篩選器。 對配置為允許訪問用戶或組子集的頁使用權限敏感快取。

下圖說明了當Web瀏覽器請求使用權限敏感快取的頁面時發生的事件的順序。

## 頁面已快取且用戶已授權 {#page-is-cached-and-user-is-authorized}

![](assets/chlimage_1.png)

1. Dispatcher確定請求的內容已快取且有效。
1. 調度程式向呈現器發送請求消息。 HEAD部分包括瀏覽器請求中的所有標題行。
1. 呈現器調用身份驗證檢查器servlet以執行安全檢查並響應Dispatcher。 響應消息包括HTTP狀態代碼200，以指示用戶已授權。
1. 調度程式向瀏覽器發送響應消息，該瀏覽器由呈現響應的標題行和主體中的快取內容組成。

## 頁面未快取且用戶已授權 {#page-is-not-cached-and-user-is-authorized}

![](assets/chlimage_1-1.png)

1. Dispatcher確定內容未快取或需要更新。
1. 調度程式將原始請求轉發到呈現器。
1. 呈現器調AEM用授權程式servlet（這不是Dispatcher AuthChcker Servlet）以執行安全檢查。 當用戶被授權時，呈現器將呈現的頁面包括在響應消息的正文中。
1. 調度程式將響應轉發到瀏覽器。 Dispatcher將呈現的響應消息的正文添加到快取中。

## 用戶未授權 {#user-is-not-authorized}

![](assets/chlimage_1-2.png)

1. 調度程式檢查快取。
1. Dispatcher向呈現器發送請求消息，該呈現器包含瀏覽器請求中的所有標題行。
1. 呈現器調用身份驗證檢查器servlet執行失敗的安全檢查，並且呈現器將原始請求轉發到Dispatcher。
1. 調度程式將原始請求轉發到呈現器。
1. 呈現器調AEM用授權程式servlet（這不是Dispatcher AuthChcker Servlet）以執行安全檢查。 當用戶被授權時，呈現器將呈現的頁面包括在響應消息的正文中。
1. 調度程式將響應轉發到瀏覽器。 Dispatcher將呈現的響應消息的正文添加到快取中。


## 實現對權限敏感的快取 {#implementing-permission-sensitive-caching}

要實現對權限敏感的快取，請執行以下任務：

* 開發執行身份驗證和授權的Servlet
* 配置Dispatcher

>[!NOTE]
>
>通常，安全資源儲存在一個單獨的資料夾中，而不是不安全的檔案中。 例如，/content/secure/

## 建立身份驗證檢查器Servlet {#create-the-auth-checker-servlet}

建立並部署一個Servlet，該Servlet執行請求Web內容的用戶的驗證和授權。 Servlet可以使用任何驗證和授權方法，AEM如用戶帳戶和儲存庫ACL或LDAP查找服務。 將Servlet部署到DispatcherAEM用作呈現的實例。

Servlet必須可供所有用戶訪問。 因此，您的Servlet應擴展 `org.apache.sling.api.servlets.SlingSafeMethodsServlet` 類，它提供對系統的只讀訪問。

Servlet只接收來自呈現器的HEAD請求，因此您只需要實現 `doHead` 的雙曲餘切值。

呈現器包括作為HTTP請求參數的所請求資源的URI。 例如，通過 `/bin/permissioncheck`。 要在/content/geometrixx-outdoors/en.html頁面上執行安全檢查，呈現器在HTTP請求中包含以下URL:

`/bin/permissioncheck?uri=/content/geometrixx-outdoors/en.html`

Servlet響應消息必須包含以下HTTP狀態代碼：

* 200:已通過身份驗證和授權。

以下示例servlet從HTTP請求獲取所請求資源的URL。 代碼使用Felix SCR `Property` 注釋以設定 `sling.servlet.paths` 屬性。 在 `doHead` 方法，servlet獲取會話對象並使用 `checkPermission` 確定相應響應代碼的方法。

>[!NOTE]
>
>必須在Sling Servlet解析器(org.apache.sling.servlet.resolver.SlingServletResolver)服務中啟用sling.servlet.paths屬性的值。

### Servlet示例 {#example-servlet}

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

## 配置Dispatcher以進行權限敏感快取 {#configure-dispatcher-for-permission-sensitive-caching}

dispatcher.any檔案的auth_checker部分控制對權限敏感的快取的行為。 auth_checker部分包含以下子部分：

* `url`:的值 `sling.servlet.paths` 執行安全檢查的servlet的屬性。

* `filter`:指定對權限敏感的快取應用到的資料夾的篩選器。 通常， `deny` 篩選器應用於所有資料夾， `allow` 篩選器將應用於安全資料夾。

* `headers`:指定授權servlet在響應中包括的HTTP標頭。

啟動Dispatcher時，Dispatcher日誌檔案包含以下調試級別消息：

`AuthChecker: initialized with URL 'configured_url'.`

下面的auth_checker示例部分將Dispatcher配置為使用前一個主題的servlet。 篩選器部分使權限檢查僅對安全HTML資源執行。

### 配置示例 {#example-configuration}

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
