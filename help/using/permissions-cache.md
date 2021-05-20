---
title: 快取安全內容
seo-title: 快取AEM Dispatcher中的安全內容
description: 了解Dispatcher中對權限敏感的快取運作方式。
seo-description: 了解AEM Dispatcher中對權限敏感的快取運作方式。
uuid: abfed68a-2efe-45f6-bdf7-2284931629d6
contentOwner: User
products: SG_EXPERIENCEMANAGER/DISPATCHER
topic-tags: dispatcher
content-type: reference
discoiquuid: 4f9b2bc8-a309-47bc-b70d-a1c0da78d464
exl-id: 3d8d8204-7e0d-44ad-b41b-6fec2689c6a6
source-git-commit: 3a0e237278079a3885e527d7f86989f8ac91e09d
workflow-type: tm+mt
source-wordcount: '762'
ht-degree: 0%

---

# 快取安全內容 {#caching-secured-content}

對權限敏感的快取可讓您快取受保護的頁面。 傳送快取頁面之前，Dispatcher會檢查使用者的頁面存取權限。

Dispatcher包含AuthChecker模組，可實作對權限敏感的快取。 當模組啟動時，呈現會呼叫AEM servlet以對請求的內容執行使用者驗證和授權。 Servlet響應確定內容是否傳遞到Web瀏覽器。

由於驗證和授權方法是AEM部署專屬的，因此需要建立servlet。

>[!NOTE]
>
>使用`deny`篩選器來強制實施一攬子安全限制。 對已設定為允許存取使用者或群組子集的頁面，使用對權限敏感的快取。

下列圖表說明當網頁瀏覽器要求使用權限相關快取時，所發生事件的順序。

## 已快取頁面，且已授權用戶{#page-is-cached-and-user-is-authorized}

![](assets/chlimage_1.png)

1. Dispatcher會判斷請求的內容是快取且有效。
1. Dispatcher傳送請求訊息至呈現。 HEAD區段包含瀏覽器請求中的所有標題行。
1. 轉譯會呼叫授權者以執行安全性檢查並回應Dispatcher。 回應訊息包含HTTP狀態代碼200，以指出使用者已獲授權。
1. Dispatcher會傳送回應訊息給瀏覽器，該訊息包含轉譯回應的標題行和內文中的快取內容。

## 未快取頁面，且用戶已獲得{#page-is-not-cached-and-user-is-authorized}授權

![](assets/chlimage_1-1.png)

1. Dispatcher會判斷內容未快取或需要更新。
1. Dispatcher會將原始請求轉送至轉譯。
1. 呈現會呼叫授權程式servlet以執行安全性檢查。 授權使用者後，呈現會在回應訊息的正文中包含呈現的頁面。
1. Dispatcher將回應轉送至瀏覽器。 Dispatcher會將轉譯之回應訊息的內文新增至快取。

## 未授權用戶{#user-is-not-authorized}

![](assets/chlimage_1-2.png)

1. Dispatcher會檢查快取。
1. Dispatcher會將請求訊息傳送至呈現，其中包含瀏覽器請求中的所有標題行。
1. 轉譯會呼叫授權者servlet以執行安全檢查，但檢查失敗，且轉譯會將原始請求轉送至Dispatcher。

## 實施對權限敏感的快取{#implementing-permission-sensitive-caching}

若要實作需要權限的快取，請執行下列工作：

* 開發執行驗證和授權的Servlet
* 設定Dispatcher

>[!NOTE]
>
>通常，安全資源儲存在單獨的資料夾中，而不是不安全的檔案。 例如/content/secure/


## 建立授權servlet {#create-the-authorization-servlet}

建立並部署Servlet，以執行請求Web內容的用戶的驗證和授權。 Servlet可以使用任何驗證和授權方法，如AEM用戶帳戶和儲存庫ACL或LDAP查找服務。 您將servlet部署至AEM執行個體，Dispatcher將其用作轉譯。

所有使用者都必須能存取Servlet。 因此，您的servlet應擴展`org.apache.sling.api.servlets.SlingSafeMethodsServlet`類，該類提供對系統的只讀訪問。

Servlet只會從轉譯接收HEAD請求，因此您只需要實作`doHead`方法。

呈現包含請求資源的URI，作為HTTP請求的參數。 例如，授權servlet是透過`/bin/permissioncheck`存取。 若要在/content/geometrixx-outdoors/en.html頁面上執行安全檢查，轉譯會在HTTP要求中包含下列URL:

`/bin/permissioncheck?uri=/content/geometrixx-outdoors/en.html`

Servlet響應消息必須包含以下HTTP狀態代碼：

* 200:已傳遞驗證和授權。

下列範例servlet從HTTP要求中取得所請求資源的URL。 該代碼使用Felix SCR `Property`注釋將`sling.servlet.paths`屬性的值設定為/bin/permissioncheck。 在`doHead`方法中，servlet獲取會話對象，並使用`checkPermission`方法確定相應的響應代碼。

>[!NOTE]
>
>必須在Sling Servlet Resolver(org.apache.sling.servlets.resolver.SlingServletResolver)服務中啟用sling.servlet.paths屬性的值。

### Servlet {#example-servlet}示例

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

## 設定Dispatcher以進行需要權限的快取{#configure-dispatcher-for-permission-sensitive-caching}

dispatcher.any檔案的auth_checker區段會控制對權限敏感的快取行為。 auth_checker區段包含下列子區段：

* `url`:執行安全檢 `sling.servlet.paths` 查的Servlet屬性的值。

* `filter`:用於指定對權限敏感的快取應用到的資料夾的篩選器。通常， `deny`篩選器會套用至所有資料夾，而`allow`篩選器會套用至安全資料夾。

* `headers`:指定授權servlet在回應中包含的HTTP標題。

Dispatcher啟動時，Dispatcher記錄檔會包含下列除錯層級的訊息：

`AuthChecker: initialized with URL 'configured_url'.`

以下範例auth_checker區段會將Dispatcher設定為使用上一個主題的servlet。 篩選器區段會導致權限檢查僅對安全HTML資源執行。

### 配置示例{#example-configuration}

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
