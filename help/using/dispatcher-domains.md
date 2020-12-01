---
title: '使用具有多個網域的 Dispatcher '
seo-title: '使用具有多個網域的 Dispatcher '
description: 瞭解如何使用Dispatcher處理多個Web網域中的頁面請求。
seo-description: 瞭解如何使用Dispatcher處理多個Web網域中的頁面請求。
uuid: 7342a1c2-fe61-49be-a240-b487d53c7ec1
contentOwner: User
cq-exporttemplate: /etc/contentsync/templates/geometrixx/page/rewrite
products: SG_EXPERIENCEMANAGER/DISPATCHER
topic-tags: dispatcher
content-type: reference
discoiquuid: 40d91d66-c99b-422d-8e61-c0ced23272ef
translation-type: tm+mt
source-git-commit: 64d26d802dbc9bb0b6815011a16e24c63a7672aa
workflow-type: tm+mt
source-wordcount: '2983'
ht-degree: 0%

---


# 使用具有多個網域的 Dispatcher {#using-dispatcher-with-multiple-domains}

>[!NOTE]
>
>Dispatcher 版本與 AEM 無關。如果您遵循內嵌於AEM或CQ檔案中之Dispatcher檔案的連結，您可能會被重新導向至本頁面。

使用Dispatcher可處理多個Web網域中的頁面請求，同時支援下列條件：

* 兩個網域的Web內容都儲存在單一AEM儲存庫中。
* Dispatcher快取中的檔案可針對每個域分別失效。

例如，一家公司為其兩個品牌發佈網站：品牌A和品牌B。網站頁面的內容是以AEM編寫，並儲存在相同的儲存庫工作區：

```
/
| - content  
   | - sitea  
       | - content nodes  
   | - siteb  
       | - content nodes
```

`BrandA.com`的頁面儲存在`/content/sitea`下方。 客戶端對URL `https://BrandA.com/en.html`的請求將返回`/content/sitea/en`節點的呈現頁面。 同樣地，`BrandB.com`的頁面儲存在`/content/siteb`下方。

使用Dispatcher快取內容時，必須在用戶端HTTP請求中的頁面URL、快取中對應檔案的路徑，以及儲存庫中對應檔案的路徑之間建立關聯。

## 用戶端要求

當客戶端向Web伺服器發送HTTP請求時，必須將請求頁面的URL解析為Dispatcher快取中的內容，並最終解析為儲存庫中的內容。

![](assets/chlimage_1-8.png)

1. 域名系統發現在HTTP請求中為域名註冊的Web伺服器的IP地址。
1. HTTP要求會傳送至Web伺服器。
1. HTTP請求會傳遞至Dispatcher。
1. Dispatcher確定快取的檔案是否有效。 如果有效，則會將快取的檔案提供給用戶端。
1. 如果快取的檔案無效，Dispatcher會從AEM發佈例項要求新轉譯的頁面。

## 快取失效

當Dispatcher Flush複製代理請求Dispatcher使快取檔案失效時，儲存庫中內容的路徑必須解析為快取中的內容。

![](assets/chlimage_1-9.png)

1. 頁面會在AEM作者例項上啟動，內容會複製至發佈例項。
1. Dispatcher Flush Agent調用Dispatcher以使複製內容的快取失效。
1. Dispatcher會觸及一或多個。stat檔案，使快取檔案無效。

若要搭配多個網域使用Dispatcher，您必須設定AEM、Dispatcher和您的Web伺服器。 本頁所述的解決方案是一般的，適用於大多數環境。 由於某些AEM拓撲的複雜性，您的解決方案可能需要進一步的自訂配置來解決特定問題。 您可能需要調整示例以滿足現有的IT基礎架構和管理策略。

## URL映射{#url-mapping}

若要啟用網域URL和內容路徑以解析為快取檔案，在程式的某個時間點，必須轉換檔案路徑或頁面URL。 提供了以下常見策略的說明，其中路徑或URL轉換在進程中的不同點發生：

* （建議）AEM發佈例項使用Sling對應來解析資源，以實作內部URL重寫規則。 網域URL會轉譯為內容儲存庫路徑。 請參閱[AEM Rewrites Incoming URLs](#aem-rewrites-incoming-urls)。
* Web伺服器使用內部URL重寫規則，將網域URL轉譯為快取路徑。 請參閱[The Web Server Rewrites Incoming URLs](#the-web-server-rewrites-incoming-urls)。

一般而言，最好為網頁使用簡短的URL。 通常，頁面URL會鏡像包含Web內容的儲存庫資料夾的結構。 但是，URL不會顯示最上層的儲存庫節點，如`/content`。 用戶端不一定知道AEM存放庫的結構。

## 一般要求{#general-requirements}

您的環境必須實施以下配置，以支援使用多個域的Dispatcher:

* 每個域的內容駐留在儲存庫的不同分支中（請參見下面的示例環境）。
* Dispatcher Flush複製代理已設定在AEM發佈例項上。 （請參閱[從發佈實例中使Dispatcher Cache失效](page-invalidate.md)）。
* 域名系統將域名解析為Web伺服器的IP地址。
* Dispatcher快取會鏡像AEM內容存放庫的目錄結構。 Web伺服器文檔根目錄下的檔案路徑與儲存庫中檔案的路徑相同。

## 提供的示例環境{#environment-for-the-provided-examples}

提供的示例解決方案適用於具有以下特徵的環境：

* AEM作者和發佈例項部署在Linux系統上。
* Apache HTTPD是部署在Linux系統上的Web伺服器。
* AEM內容存放庫和Web伺服器的檔案根目錄會使用下列檔案結構(Apache Web伺服器的檔案根目錄為/`usr/lib/apache/httpd-2.4.3/htdocs)`:

   **存放庫**

```
  | - /content  
    | - sitea  
  |    | - content nodes 
    | - siteb  
       | - conent nodes
```

**Web伺服器的文檔根目錄**

```
  | - /usr  
    | - lib  
      | - apache  
        | - httpd-2.4.3  
          | - htdocs  
            | - content  
              | - sitea  
                 | - content nodes 
              | - siteb  
                 | - content nodes
```

## AEM重寫傳入的URL {#aem-rewrites-incoming-urls}

資源解析度的Sling對應可讓您將傳入的URL與AEM內容路徑產生關聯。 在AEM發佈例項上建立對應，以便從Dispatcher將請求解析為儲存庫中正確的內容。

頁面演算的Dispatcher要求會使用從Web伺服器傳遞的URL來識別頁面。 當URL包含網域名稱時，Sling映射會將URL解析為內容。 下圖說明`branda.com/en.html` URL與`/content/sitea/en`節點的映射。

![](assets/chlimage_1-10.png)

Dispatcher快取將鏡像儲存庫節點結構。 因此，當頁面啟動時，導致無法編輯快取頁面的請求不需要URL或路徑轉換。

![](assets/chlimage_1-11.png)

## 在Web伺服器上定義虛擬主機{#define-virtual-hosts-on-the-web-server}

定義Web伺服器上的虛擬主機，以便將不同的文檔根目錄分配給每個Web域：

* Web伺服器必須為每個Web域定義虛擬域。
* 對於每個域，請將文檔根配置為與包含域Web內容的儲存庫中的資料夾一致。
* 每個虛擬域還必須包括與Dispatcher相關的配置，如[安裝Dispatcher](dispatcher-install.md)頁中所述。

以下示例`httpd.conf`檔案為Apache Web伺服器配置兩個虛擬域：

* 伺服器名稱（與域名一致）是branda.com（行16）和brandb.com（行30）。
* 每個虛擬域的文檔根目錄是Dispatcher快取中包含站點頁面的目錄。 （第17和31行）

使用此配置時，Web伺服器在收到`https://branda.com/en/products.html`請求時執行以下操作：

* 將URL與具有`branda.com.`的`ServerName`虛擬主機關聯

* 將URL轉發到Dispatcher。

### httpd.conf {#httpd-conf}

```xml
# load the Dispatcher module
LoadModule dispatcher_module modules/mod_dispatcher.so
# configure the Dispatcher module
<IfModule disp_apache2.c>
 DispatcherConfig conf/dispatcher.any
 DispatcherLog    logs/dispatcher.log  
 DispatcherLogLevel 3
 DispatcherNoServerHeader 0 
 DispatcherDeclineRoot 0
 DispatcherUseProcessedURL 0
 DispatcherPassError 0
</IfModule>

# Define virtual host for brandA.com
<VirtualHost *:80>
  ServerName branda.com
  DocumentRoot /usr/lib/apache/httpd-2.4.3/htdocs/content/sitea
   <Directory /usr/lib/apache/httpd-2.4.3/htdocs/content/sitea>
     <IfModule disp_apache2.c>
       SetHandler dispatcher-handler
       ModMimeUsePathInfo On
     </IfModule>
     Options FollowSymLinks
     AllowOverride None
   </Directory>
</VirtualHost>

# define virtual host for brandB.com
<VirtualHost *:80>
  ServerName brandB.com
  DocumentRoot /usr/lib/apache/httpd-2.4.3/htdocs/content/siteb
   <Directory /usr/lib/apache/httpd-2.4.3/htdocs/content/siteb>
     <IfModule disp_apache2.c>
       SetHandler dispatcher-handler
       ModMimeUsePathInfo On
     </IfModule>
     Options FollowSymLinks
     AllowOverride None
   </Directory>
</VirtualHost>

# document root for web server
DocumentRoot "/usr/lib/apache/httpd-2.4.3/htdocs"
```

請注意，虛擬主機繼承在主伺服器部分中配置的[DispatcherConfig](dispatcher-install.md#main-pars-67-table-7)屬性值。 虛擬主機可以包含其自己的DispatcherConfig屬性，以覆蓋主伺服器配置。

### 配置Dispatcher以處理多個域{#configure-dispatcher-to-handle-multiple-domains}

要支援包含域名及其相應虛擬主機的URL，請定義以下Dispatcher場：

* 為每個虛擬主機配置Dispatcher群。 這些場會處理來自網頁伺服器的每個網域要求、檢查快取檔案，以及從轉譯中要求頁面。
* 配置Dispatcher群，該群用於使快取的內容失效，而不管內容屬於哪個域。 此群處理來自刷新Dispatcher複製代理的檔案失效請求。

### 為虛擬主機建立Dispatcher場

虛擬主機的場必須具有以下配置，以便將客戶端HTTP請求中的URL解析為Dispatcher快取中的正確檔案：

* `/virtualhosts`屬性已設定為域名。 此屬性使Dispatcher能夠將群與域關聯。
* `/filter`屬性可讓存取在網域名稱部分之後截斷的請求URL的路徑。 例如，對於`https://branda.com/en.html` URL，路徑會解譯為`/en.html`，因此篩選器必須允許存取此路徑。

* `/docroot`屬性設定為Dispatcher快取中域站點內容的根目錄的路徑。 此路徑會用作原始請求串連URL的首碼。 例如，`/usr/lib/apache/httpd-2.4.3/htdocs/sitea`的docroot會將`https://branda.com/en.html`的要求解析為`/usr/lib/apache/httpd-2.4.3/htdocs/sitea/en.html`檔案。

此外，AEM發佈例項必須指定為虛擬主機的演算。 根據需要配置其他場屬性。 以下代碼是branda.com域的縮寫群配置：

```xml
/farm_sitea  {     
    ...
    /virtualhosts { "branda.com" }
    /renders {
      /rend01  { /hostname "127.0.0.1"  /port "4503" }
    }
    /filter {
      /0001 { /type "deny"  /glob "*" }
      /0023 { /type "allow" /glob "*/en*" }  
      ...
     }
    /cache {
      /docroot "/usr/lib/apache/httpd-2.4.3/htdocs/content/sitea"
      ...
   }
   ...
}
```

### 建立快取失效的Dispatcher群

處理取消快取檔案的請求時，需要Dispatcher群。 此群必須能夠訪問每個虛擬主機的docroot目錄中的。stat檔案。

下列屬性設定可讓Dispatcher從快取中的檔案解析AEM內容存放庫中的檔案：

* `/docroot`屬性設為Web伺服器的預設配置。 通常，此目錄是建立`/content`資料夾的目錄。 在Linux上，Apache的示例值為`/usr/lib/apache/httpd-2.4.3/htdocs`。
* `/filter`屬性允許訪問`/content`目錄下的檔案。

`/statfileslevel`屬性必須足夠高，以便在每個虛擬主機的根目錄中建立。stat檔案。 此屬性使每個域的快取分別失效。 對於示例設定，`2`的`/statfileslevel`值在`*docroot*/content/sitea`目錄和`*docroot*/content/siteb`目錄中建立。stat檔案。

此外，必須將發佈實例指定為虛擬主機的渲染。 根據需要配置其他場屬性。 以下代碼是用於使快取失效的群的縮寫配置：

```xml
/farm_flush {  
    ...
    /virtualhosts   { "invalidation_only" }
    /renders  {
      /rend01  { /hostname "127.0.0.1" /port "4503" }
    }
    /filter   {
      /0001 { /type "deny"  /glob "*" }
      /0023 { /type "allow" /glob "*/content*" } 
      ...
      }
    /cache  {
       /docroot "/usr/lib/apache/httpd-2.4.3/htdocs"
       /statfileslevel "2"
       ...
   }
   ...
}
```

啟動Web伺服器時，Dispatcher日誌（在調試模式下）指示所有場的初始化：

```shell
Dispatcher initializing (build 4.1.2)
[Fri Nov 02 16:27:18 2012] [D] [24974(140006182991616)] farms[farm_sitea].cache.docroot = /usr/lib/apache/httpd-2.4.3/htdocs/content/sitea
[Fri Nov 02 16:27:18 2012] [D] [24974(140006182991616)] farms[farm_siteb].cache.docroot = /usr/lib/apache/httpd-2.4.3/htdocs/content/siteb
[Fri Nov 02 16:27:18 2012] [D] [24974(140006182991616)] farms[farm_flush].cache.docroot = /usr/lib/apache/httpd-2.4.3/htdocs
[Fri Nov 02 16:27:18 2012] [I] [24974(140006182991616)] Dispatcher initialized (build 4.1.2)
```

### 設定資源解析度{#configure-sling-mapping-for-resource-resolution}的Sling對應

使用Sling對應來解析資源，讓網域型URL解析為AEM發佈例項上的內容。 資源映射將Dispatcher（最初是從客戶端HTTP請求）傳入的URL轉換為內容節點。

若要瞭解Sling資源對應，請參閱Sling檔案中的[Mappings for Resource Resolution](https://sling.apache.org/site/mappings-for-resource-resolution.html)。

通常，以下資源需要映射，但可能需要其他映射：

* 內容頁面的根節點（`/content`下）
* 頁面使用的設計節點（`/etc/designs`下）
* `/libs`資料夾

建立內容頁面的對應後，若要探索其他必要的對應，請使用網頁瀏覽器在網頁伺服器上開啟頁面。 在發佈實例的error.log檔案中，找到有關未找到的資源的消息。 以下示例消息表示需要`/etc/clientlibs`的映射：

```shell
01.11.2012 15:59:24.601 *INFO* [10.36.34.243 [1351799964599] GET /etc/clientlibs/foundation/jquery.js HTTP/1.1] org.apache.sling.engine.impl.SlingRequestProcessorImpl service: Resource /content/sitea/etc/clientlibs/foundation/jquery.js not found
```

>[!NOTE]
>
>預設Apache Sling重寫程式的linkchecker變數會自動修改頁面中的超連結，以防止中斷的連結。 但是，只有當連結目標是HTML或HTM檔案時，才會執行連結重寫。 若要更新其他檔案類型的連結，請建立變壓器元件並將它新增至HTML重新寫入器管線。

### 資源映射節點示例

下表列出了為branda.com域實現資源映射的節點。 為`brandb.com`域建立類似節點，如`/etc/map/http/brandb.com`。 在所有情況下，當頁面HTML中的參考無法在Sling內容中正確解析時，都需要映射。

| 節點路徑 | 類型 | 屬性 |
|--- |--- |--- |
| `/etc/map/http/branda.com` | sling:Mapping | 名稱：sling:internalRedirect Type:字串值：/content/sitea |
| `/etc/map/http/branda.com/libs` | sling:Mapping | 名稱：sling:internalRedirect <br/>Type:字串<br/>值：/libs |
| `/etc/map/http/branda.com/etc` | sling:Mapping |  |
| `/etc/map/http/branda.com/etc/designs` | sling:Mapping | 名稱：sling:internalRedirect <br/>VType:字串<br/>值：/etc/designs |
| `/etc/map/http/branda.com/etc/clientlibs` | sling:Mapping | 名稱：sling:internalRedirect <br/>VType:字串<br/>值：/etc/clientlibs |

## 配置Dispatcher Flush複製代理{#configuring-the-dispatcher-flush-replication-agent}

AEM發佈例項上的Dispatcher Flush複製代理必須將失效請求傳送至正確的Dispatcher群。 要定位群，請使用Dispatcher Flush複製代理的URI屬性（在「傳輸」頁籤上）。 為配置為使快取失效的Dispatcher群包含`/virtualhost`屬性的值：

`https://*webserver_name*:*port*/*virtual_host*/dispatcher/invalidate.cache`

例如，若要使用上一個範例的`farm_flush`群，URI為`https://localhost:80/invalidation_only/dispatcher/invalidate.cache`。

![](assets/chlimage_1-12.png)

## Web伺服器重寫傳入的URL {#the-web-server-rewrites-incoming-urls}

使用Web伺服器的內部URL重寫功能，將網域型URL轉譯為Dispatcher快取中的檔案路徑。 例如，將`https://brandA.com/en.html`頁面的客戶機請求轉換為Web伺服器文檔根目錄中的`content/sitea/en.html`檔案。

![](assets/chlimage_1-13.png)

Dispatcher快取將鏡像儲存庫節點結構。 因此，當頁面啟動時，產生的取消驗證快取頁面的要求不需要URL或路徑轉換。

![](assets/chlimage_1-14.png)

## 在Web伺服器{#define-virtual-hosts-and-rewrite-rules-on-the-web-server}上定義虛擬主機和重寫規則

在Web伺服器上配置以下方面：

* 為每個Web網域定義虛擬主機。
* 對於每個域，請將文檔根配置為與包含域Web內容的儲存庫中的資料夾一致。
* 針對每個虛擬網域，建立URL重新命名規則，將傳入的URL轉譯為快取檔案的路徑。
* 每個虛擬域還必須包括與Dispatcher相關的配置，如[安裝Dispatcher](dispatcher-install.md)頁中所述。
* 必須將Dispatcher模組配置為使用Web伺服器已重寫的URL。 （請參閱[安裝Dispatcher](dispatcher-install.md)中的`DispatcherUseProcessedURL`屬性。）

以下示例httpd.conf檔案為Apache Web伺服器配置兩個虛擬主機：

* 伺服器名稱（與域名一致）是`brandA.com`（行16）和`brandB.com`（行32）。

* 每個虛擬域的文檔根目錄是Dispatcher快取中包含站點頁面的目錄。 （第20和33行）
* 每個虛擬網域的URL重寫規則是規則運算式，用於將請求頁面的路徑與快取中頁面的路徑作為前置詞。 （第19和35行）
* `DispatherUseProcessedURL`屬性設為`1`。 （第10行）

例如，Web伺服器在收到具有`https://brandA.com/en/products.html` URL的請求時執行以下操作：

* 將URL與具有`brandA.com.`的`ServerName`虛擬主機關聯
* 將URL重寫為`/content/sitea/en/products.html.`
* 將URL轉發到Dispatcher。

### httpd.conf {#httpd-conf-1}

```xml
# load the Dispatcher module
LoadModule dispatcher_module modules/mod_dispatcher.so
# configure the Dispatcher module
<IfModule disp_apache2.c>
 DispatcherConfig conf/dispatcher.any
 DispatcherLog    logs/dispatcher.log  
 DispatcherLogLevel 3
 DispatcherNoServerHeader 0 
 DispatcherDeclineRoot 0
 DispatcherUseProcessedURL 1
 DispatcherPassError 0
</IfModule>

# Define virtual host for brandA.com
<VirtualHost *:80>
  ServerName branda.com
  DocumentRoot /usr/lib/apache/httpd-2.4.3/htdocs/content/sitea
  RewriteEngine  on
  RewriteRule    ^/(.*)\.html$  /content/sitea/$1.html [PT]
   <Directory /usr/lib/apache/httpd-2.4.3/htdocs/content/sitea>
     <IfModule disp_apache2.c>
       SetHandler dispatcher-handler
       ModMimeUsePathInfo On
     </IfModule>
     Options FollowSymLinks
     AllowOverride None
   </Directory>
</VirtualHost>

# define virtual host for brandB.com
<VirtualHost *:80>
  ServerName brandB.com
  DocumentRoot /usr/lib/apache/httpd-2.4.3/htdocs/content/siteb
  RewriteEngine  on
  RewriteRule    ^/(.*)\.html$  /content/siteb/$1.html [PT]
   <Directory /usr/lib/apache/httpd-2.4.3/htdocs/content/siteb>
     <IfModule disp_apache2.c>
       SetHandler dispatcher-handler
       ModMimeUsePathInfo On
     </IfModule>
     Options FollowSymLinks
     AllowOverride None
   </Directory>
</VirtualHost>

# document root for web server
DocumentRoot "/usr/lib/apache/httpd-2.4.3/htdocs"
```

### 配置Dispatcher Farm {#configure-a-dispatcher-farm}

當Web伺服器重寫URL時，Dispatcher需要根據[ Configuring Dispatcher](dispatcher-configuration.md)定義的單個群。 支援Web伺服器虛擬主機和URL更名規則需要以下配置：

* `/virtualhosts`屬性必須包含所有VirtualHost定義的ServerName值。
* `/statfileslevel`屬性必須足夠高，才能在包含每個域內容檔案的目錄中建立。stat檔案。

以下示例配置檔案基於隨Dispatcher一起安裝的示例`dispatcher.any`檔案。 要支援前一個`httpd.conf`檔案的Web伺服器配置，需要進行以下更改：

* `/virtualhosts`屬性使Dispatcher處理`brandA.com`和`brandB.com`域的請求。 （第12行）
* `/statfileslevel`屬性設定為2，以便在包含域Web內容的每個目錄中建立stat檔案（第41行）:`/statfileslevel "2"`

和往常一樣，快取的檔案根目錄與網頁伺服器的檔案根目錄相同（第40行）:`/usr/lib/apache/httpd-2.4.3/htdocs`

### `dispatcher.any` {#dispatcher-any}

```xml
/name "testDispatcher"
/farms
  {
  /dispfarm0
    {  
    /clientheaders
      {
      "*"
      }      
    /virtualhosts
      {
      "brandA.com" "brandB.com"
      }
    /renders
      {
      /rend01    {  /hostname "127.0.0.1"   /port "4503"  }
      }
    /filter
      {
      /0001 { /type "deny"  /glob "*" }
      /0023 { /type "allow" /glob "*/content*" }  # disable this rule to allow mapped content only
      /0041 { /type "allow" /glob "* *.css *"   }  # enable css
      /0042 { /type "allow" /glob "* *.gif *"   }  # enable gifs
      /0043 { /type "allow" /glob "* *.ico *"   }  # enable icos
      /0044 { /type "allow" /glob "* *.js *"    }  # enable javascript
      /0045 { /type "allow" /glob "* *.png *"   }  # enable png
      /0046 { /type "allow" /glob "* *.swf *"   }  # enable flash
      /0061 { /type "allow" /glob "POST /content/[.]*.form.html" }  # allow POSTs to form selectors under content
      /0062 { /type "allow" /glob "* /libs/cq/personalization/*"  }  # enable personalization
      /0081 { /type "deny"  /glob "GET *.infinity.json*" }
      /0082 { /type "deny"  /glob "GET *.tidy.json*"     }
      /0083 { /type "deny"  /glob "GET *.sysview.xml*"   }
      /0084 { /type "deny"  /glob "GET *.docview.json*"  }
      /0085 { /type "deny"  /glob "GET *.docview.xml*"  }      
      /0086 { /type "deny"  /glob "GET *.*[0-9].json*" }
      /0090 { /type "deny"  /glob "* *.query.json*" }
      }
    /cache
      {
      /docroot "/usr/lib/apache/httpd-2.4.3/htdocs"
      /statfileslevel "2"
      /allowAuthorized "0"
      /rules
        {
        /0000  { /glob "*"     /type "allow"  }
        }
      /invalidate
        {
        /0000  {   /glob "*" /type "deny"  }
        /0001 {  /glob "*.html" /type "allow"  }
        }
      /allowedClients
        {
        }     
      }
    /statistics
      {
      /categories
        {
        /html  { /glob "*.html" }
        /others  {  /glob "*"  }
        }
      }
    }
  }
```

>[!NOTE]
>
>由於已定義單一Dispatcher群，因此AEM發佈例項上的Dispatcher Flush複製代理不需要特殊組態。

## 重寫非HTML檔案的連結{#rewriting-links-to-non-html-files}

若要重寫副檔名為。html或。htm以外檔案的參照，請建立Sling rewriter變形器元件，並將它新增至預設的重寫器管線。

當資源路徑在Web伺服器上下文中無法正確解析時，重寫引用。 例如，當影像產生元件建立連結(例如/content/sitea/en/products.navimage.png)時，需要變形器。 [如何建立功能完備的網際網路網站](https://helpx.adobe.com/experience-manager/6-3/sites/developing/using/the-basics.html)的topnav元件會建立此類連結。

[Sling rewriter](https://sling.apache.org/documentation/bundles/output-rewriting-pipelines-org-apache-sling-rewriter.html)是後處理Sling輸出的模組。 重寫器的SAX流水線實現包括發生器、一個或多個變壓器和串列器：

* **Generator：剖析** Sling輸出串流（HTML檔案），並在遇到特定元素類型時產生SAX事件。
* **變形器：** 監聽SAX事件，然後修改事件目標（HTML元素）。重寫器管線包含零個或多個變壓器。 互感器依序執行，將SAX事件傳遞到序列中的下一個互感器。
* **串列化** 器：串列化輸出，包括每個變壓器的修改。

![](assets/chlimage_1-15.png)

### AEM Default Rewriter Pipeline {#the-aem-default-rewriter-pipeline}

AEM使用預設的管線重寫程式來處理文字/html類型的檔案：

* 產生器會剖析HTML檔案，並在遇到a、img、區域、表單、基本、連結、指令碼和內文元素時產生SAX事件。 生成器別名為`htmlparser`。
* 該管道包括以下變壓器：`linkchecker`、`mobile`、`mobiledebug`、`contentsync`。 `linkchecker`轉換器將路徑外部化為參考的HTML或HTM檔案，以防止連結中斷。
* 序列化程式會寫入HTML輸出。 序列化器別名是htmlwriter。

`/libs/cq/config/rewriter/default`節點定義管線。

### 建立變壓器{#creating-a-transformer}

執行以下任務以建立變壓器元件並在管線中使用它：

1. 實施`org.apache.sling.rewriter.TransformerFactory`介面。 此類建立變壓器類的實例。 指定`transformer.type`屬性（變壓器別名）的值，並將類配置為OSGi服務元件。
1. 實施`org.apache.sling.rewriter.Transformer`介面。 要最小化工作量，可以擴展`org.apache.cocoon.xml.sax.AbstractSAXPipe`類。 覆寫startElement方法以自訂重寫行為。 對每個傳遞給變壓器的SAX事件調用此方法。
1. 捆綁和部署類。
1. 將設定節點新增至AEM應用程式，將變壓器新增至管線。

>[!TIP]
>您可以改為將TransformerFactory配置為將變壓器插入定義的每個重寫器。 因此，您不需要配置管線：
>
>* 將`pipeline.mode`屬性設為`global`。
>* 將`service.ranking`屬性設為正整數。
>* 請勿包含`pipeline.type`屬性。


>[!NOTE]
>
>使用Content Package Maven Plugin的[multimodule](https://helpx.adobe.com/experience-manager/aem-previous-versions.html)原型建立您的Maven專案。 POM會自動建立和安裝內容包。

下列範例實作可重寫影像檔案參照的變形器。

* MyRewriterTransformerFactory類實例化MyRewriterTransformer對象。 pipeline.type屬性將變壓器別名設定為mytransformer。 為了將別名包括在流水線中，流水線配置節點將該別名包括在變壓器清單中。
* MyRewriterTransformer類覆蓋AbstractSAXTransfer類的startElement方法。 startElement方法會重寫img元素的src屬性值。

這些範例不健全，不應用於生產環境。

### TransformerFactory實施示例{#example-transformerfactory-implementation}

```java
package com.adobe.example;

import org.apache.felix.scr.annotations.Component;
import org.apache.felix.scr.annotations.Service;
import org.apache.felix.scr.annotations.Property;

import org.apache.sling.rewriter.Transformer;
import org.apache.sling.rewriter.TransformerFactory;

@Component
@Service
public class MyRewriterTransformerFactory implements TransformerFactory {
    /* Define the alias */
    @Property(value="mytransformer")
    static final String PIPELINE_TYPE ="pipeline.type";
 
    public Transformer createTransformer() {
        
        return new MyRewriterTransformer ();
    }
}
```

### 變壓器實施示例{#example-transformer-implementation}

```java
package com.adobe.example;

import java.io.IOException;

import org.apache.cocoon.xml.sax.AbstractSAXPipe;

import org.apache.sling.api.SlingHttpServletRequest;
import org.apache.sling.rewriter.ProcessingComponentConfiguration;
import org.apache.sling.rewriter.ProcessingContext;
import org.apache.sling.rewriter.Transformer;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import org.xml.sax.Attributes;
import org.xml.sax.SAXException;
import org.xml.sax.helpers.AttributesImpl;

import javax.servlet.http.HttpServletRequest;

public class MyRewriterTransformer extends AbstractSAXPipe implements Transformer {

 private static final Logger log = LoggerFactory.getLogger(MyRewriterTransformer.class);
 private SlingHttpServletRequest httpRequest; 
 /* The element and attribute to act on  */
 private static final String ATT_NAME = new String("src");
 private static final String EL_NAME = new String("img");

 public MyRewriterTransformer () {
 }
 public void dispose() {
 }
 public void init(ProcessingContext context, ProcessingComponentConfiguration config) throws IOException {
  this.httpRequest = context.getRequest();
  log.debug("Transforming request {}.", httpRequest.getRequestURI());
 }
 @Override
 public void startElement (String nsUri, String localname, String qname, Attributes atts) throws SAXException {
  /* copy the element attributes */
  AttributesImpl linkAtts = new AttributesImpl(atts); 
  /* Only interested in EL_NAME elements */
  if(EL_NAME.equalsIgnoreCase(localname)){

   /* iterate through the attributes of the element and act only on ATT_NAME attributes */
   for (int i=0; i < linkAtts.getLength(); i++) {
    if (ATT_NAME.equalsIgnoreCase(linkAtts.getLocalName(i))) {
     String path_in_link = linkAtts.getValue(i);

     /* use the resource resolver of the http request to reverse-resolve the path  */
     String mappedPath = httpRequest.getResourceResolver().map(httpRequest, path_in_link);

     log.info("Tranformed {} to {}.", path_in_link,mappedPath);

     /* update the attribute value */
     linkAtts.setValue(i,mappedPath);
    }
   }

  }
        /* return updated attributes to super and continue with the transformer chain */
 super.startElement(nsUri, localname, qname, linkAtts);
 }
}
```

### 將變壓器添加到重寫器管線{#adding-the-transformer-to-a-rewriter-pipeline}

建立JCR節點，該節點定義使用變壓器的管線。 下列節點定義會建立處理文字/html檔案的管線。 使用HTML的預設AEM產生器和剖析器。

>[!NOTE]
>
>如果將「變壓器」屬性`pipeline.mode`設定為`global`，則無需配置管線。 `global`模式將變壓器插入所有管道。

### 重寫器配置節點- XML表示法{#rewriter-configuration-node-xml-representation}

```xml
<?xml version="1.0" encoding="UTF-8"?>
<jcr:root xmlns:jcr="https://www.jcp.org/jcr/1.0" xmlns:nt="https://www.jcp.org/jcr/nt/1.0"
    jcr:primaryType="nt:unstructured"
    contentTypes="[text/html]"
    enabled="{Boolean}true"
    generatorType="htmlparser"
    order="5"
    serializerType="htmlwriter"
    transformerTypes="[mytransformer]">
</jcr:root>
```

下圖顯示節點的CRXDE Lite表示法：

![](assets/chlimage_1-16.png)
