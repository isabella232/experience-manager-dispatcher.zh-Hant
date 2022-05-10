---
title: '使用具有多個網域的 Dispatcher '
seo-title: Using Dispatcher with Multiple Domains
description: 瞭解如何使用Dispatcher處理多個Web域中的頁請求。
seo-description: Learn how to use Dispatcher to process page requests in multiple web domains.
uuid: 7342a1c2-fe61-49be-a240-b487d53c7ec1
contentOwner: User
cq-exporttemplate: /etc/contentsync/templates/geometrixx/page/rewrite
products: SG_EXPERIENCEMANAGER/DISPATCHER
topic-tags: dispatcher
content-type: reference
discoiquuid: 40d91d66-c99b-422d-8e61-c0ced23272ef
exl-id: 1470b636-7e60-48cc-8c31-899f8785dafa
source-git-commit: 9d168ab7139e46b0c768fc3bab37245459eca002
workflow-type: tm+mt
source-wordcount: '2965'
ht-degree: 99%

---

# 使用具有多個網域的 Dispatcher {#using-dispatcher-with-multiple-domains}

>[!NOTE]
>
>Dispatcher 版本與 AEM 無關。如果您遵循了嵌入在或CQ文檔中的Dispatcher文檔的連結，則可能已重定向AEM到此頁。

使用Dispatcher處理多個Web域中的頁請求，同時支援以下條件：

* 兩個域的Web內容都儲存在單個存AEM儲庫中。
* Dispatcher快取中的檔案可針對每個域單獨無效。

例如，一家公司發佈兩個品牌的網站：品牌A和品牌B。網站頁面的內容在同一儲存庫工作AEM區中創作並儲存在：

```
/
| - content  
   | - sitea  
       | - content nodes  
   | - siteb  
       | - content nodes
```

頁 `BrandA.com` 儲存在下面 `/content/sitea`。 客戶端請求URL `https://BrandA.com/en.html` 返回的 `/content/sitea/en` 的下界。 同樣，頁 `BrandB.com` 儲存在下面 `/content/siteb`。

使用Dispatcher快取內容時，必須在客戶端HTTP請求中的頁面URL、快取中相應檔案的路徑和儲存庫中相應檔案的路徑之間進行關聯。

## 客戶端請求

當客戶端向Web伺服器發送HTTP請求時，必須將請求頁的URL解析為Dispatcher快取中的內容，並最終解析為儲存庫中的內容。

![](assets/chlimage_1-8.png)

1. 域名系統發現在HTTP請求中為域名註冊的Web伺服器的IP地址。
1. HTTP請求將發送到Web伺服器。
1. HTTP請求將傳遞給Dispatcher。
1. Dispatcher確定快取的檔案是否有效。 如果有效，則快取的檔案將提供給客戶端。
1. 如果快取的檔案無效，Dispatcher會從發佈實例請求新呈AEM現的頁。

## 快取無效

當Dispatcher刷新複製代理請求Dispatcher使快取檔案失效時，儲存庫中內容的路徑必須解析為快取中的內容。

![](assets/chlimage_1-9.png)

1. 在作者實例上激AEM活頁面並將內容複製到發佈實例。
1. Dispatcher Flush Agent調用Dispatcher以使複製內容的快取無效。
1. Dispatcher會觸及一個或多個.stat檔案，使快取檔案無效。

要將Dispatcher用於多個域，需要配AEM置、Dispatcher和Web伺服器。 本頁中介紹的解決方案是一般性的，適用於大多數環境。 由於某些拓撲的複雜AEM性，您的解決方案可能需要進一步的定製配置來解決特定問題。 您可能需要調整示例以滿足您現有的IT基礎架構和管理策略。

## URL映射 {#url-mapping}

要使域URL和內容路徑能夠解析為快取檔案，必須在進程中的某個時刻轉換檔案路徑或頁面URL。 提供了以下常見策略的說明，其中路徑或URL轉換在進程中的不同點發生：

* （推薦）發佈實AEM例使用Sling映射實現資源解析，以實現內部URL重寫規則。 域URL將轉換為內容儲存庫路徑。 請參閱 [重寫AEM傳入URL](#aem-rewrites-incoming-urls)。
* Web伺服器使用內部URL重寫規則，將域URL轉換為快取路徑。 請參閱 [Web伺服器重寫傳入的URL](#the-web-server-rewrites-incoming-urls)。

通常希望為網頁使用短URL。 通常，頁面URL會鏡像包含Web內容的儲存庫資料夾的結構。 但是，URL不會顯示最頂層的儲存庫節點，如 `/content`。 客戶機不一定知道儲存庫的AEM結構。

## 一般要求 {#general-requirements}

您的環境必須實施以下配置以支援Dispatcher使用多個域：

* 每個域的內容都駐留在儲存庫的不同分支中（請參見下面的示例環境）。
* 已在發佈實例上配置Dispatcher Flush復AEM制代理。 (請參閱 [從發佈實例中使Dispatcher快取無效](page-invalidate.md)。)
* 域名系統將域名解析為Web伺服器的IP地址。
* Dispatcher快取鏡像內容儲存庫的目AEM錄結構。 Web伺服器文檔根目錄下的檔案路徑與儲存庫中檔案的路徑相同。

## 提供的示例環境 {#environment-for-the-provided-examples}

所提供的示例解決方案適用於具有以下特徵的環境：

* 作者AEM和發佈實例部署在Linux系統上。
* Apache HTTPD是部署在Linux系統上的Web伺服器。
* Web服AEM務器的內容儲存庫和文檔根目錄使用以下檔案結構(Apache Web伺服器的文檔根目錄為/`usr/lib/apache/httpd-2.4.3/htdocs)`:

   **存放庫**

```
  | - /content  
    | - sitea  
  |    | - content nodes 
    | - siteb  
       | - conent nodes
```

**Web伺服器的文檔根**

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

## 重寫AEM傳入URL {#aem-rewrites-incoming-urls}

使用Sling資源解析映射可以將傳入的URL與內AEM容路徑關聯。 在發佈實例上創AEM建映射，以便將來自Dispatcher的請求解析到儲存庫中的正確內容。

頁面呈現的Dispatcher請求使用從Web伺服器傳遞的URL標識頁面。 當URL包含域名時，Sling映射會將URL解析為內容。 下圖說明了 `branda.com/en.html` 到的URL `/content/sitea/en` 的下界。

![](assets/chlimage_1-10.png)

Dispatcher快取鏡像儲存庫節點結構。 因此，當頁面激活時，導致的對快取頁面無效的請求不需要URL或路徑轉換。

![](assets/chlimage_1-11.png)

## 在Web伺服器上定義虛擬主機 {#define-virtual-hosts-on-the-web-server}

在Web伺服器上定義虛擬主機，以便可以為每個Web域分配不同的文檔根：

* Web伺服器必須為每個Web域定義虛擬域。
* 對於每個域，將文檔根配置為與包含域Web內容的儲存庫中的資料夾一致。
* 每個虛擬域還必須包括與Dispatcher相關的配置，如 [正在安裝Dispatcher](dispatcher-install.md) 的子菜單。

以下示例 `httpd.conf` 檔案為Apache Web伺服器配置兩個虛擬域：

* 伺服器名（與域名相同）是branda.com（第16行）和brandb.com（第30行）。
* 每個虛擬域的文檔根目錄是包含站點頁面的Dispatcher快取中的目錄。 （第17和31行）

使用此配置，Web伺服器在收到請求時執行以下操作 `https://branda.com/en/products.html`:

* 將URL與具有 `ServerName` 共 `branda.com.`

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

請注意，虛擬主機繼承 [DispatcherConfig](dispatcher-install.md#main-pars-67-table-7) 在主伺服器部分中配置的屬性值。 虛擬主機可以包括其自己的DispatcherConfig屬性以覆蓋主伺服器配置。

### 將Dispatcher配置為處理多個域 {#configure-dispatcher-to-handle-multiple-domains}

要支援包含域名及其相應虛擬主機的URL，請定義以下Dispatcher場：

* 為每個虛擬主機配置Dispatcher場。 這些場處理來自Web伺服器的每個域的請求、檢查快取檔案以及從呈現中請求頁面。
* 配置用於使快取內容失效的Dispatcher場，而不管內容屬於哪個域。 此伺服器場處理來自刷新調度程式複製代理的檔案無效請求。

### 為虛擬主機建立Dispatcher場

虛擬主機的場必須具有以下配置，以便將客戶端HTTP請求中的URL解析為Dispatcher快取中的正確檔案：

* 的 `/virtualhosts` 屬性設定為域名。 此屬性使Dispatcher能夠將場與域關聯。
* 的 `/filter` 屬性允許訪問在域名部分之後截斷的請求URL的路徑。 例如， `https://branda.com/en.html` URL，路徑解釋為 `/en.html`，因此篩選器必須允許訪問此路徑。

* 的 `/docroot` 屬性設定為Dispatcher快取中域站點內容的根目錄的路徑。 此路徑用作原始請求中連接的URL的前置詞。 例如， `/usr/lib/apache/httpd-2.4.3/htdocs/sitea` 導致請求 `https://branda.com/en.html` 決定 `/usr/lib/apache/httpd-2.4.3/htdocs/sitea/en.html` 的子菜單。

此外，AEM必須將發佈實例指定為虛擬主機的呈現。 根據需要配置其他場屬性。 以下代碼是branda.com域的縮寫場配置：

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

### 建立快取無效的Dispatcher場

處理對快取檔案無效的請求需要Dispatcher場。 此伺服器場必須能夠訪問每個虛擬主機的docroot目錄中的.stat檔案。

以下屬性配置使Dispatcher能夠從快取中的AEM檔案解析內容儲存庫中的檔案：

* 的 `/docroot` 屬性設定為web伺服器的預設docroot。 通常，這是 `/content` 資料夾。 Linux上Apache的一個示例值是 `/usr/lib/apache/httpd-2.4.3/htdocs`。
* 的 `/filter` 屬性允許訪問下面的檔案 `/content` 的子菜單。

的 `/statfileslevel`屬性必須足夠高，以便在每個虛擬主機的根目錄中建立.stat檔案。 此屬性使每個域的快取單獨失效。 對於示例設定， `/statfileslevel` 值 `2` 在中建立.stat檔案 `*docroot*/content/sitea` 目錄和 `*docroot*/content/siteb` 的子菜單。

此外，必須將發佈實例指定為虛擬主機的呈現。 根據需要配置其他場屬性。 以下代碼是用於使快取失效的場的縮寫配置：

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

### 配置資源解析的Sling映射 {#configure-sling-mapping-for-resource-resolution}

使用Sling映射進行資源解析，以便基於域的URL解析為發佈實例AEM上的內容。 資源映射將傳入的URL從Dispatcher（最初是從客戶端HTTP請求）轉換為內容節點。

要瞭解Sling資源映射，請參見 [資源解析的映射](https://sling.apache.org/site/mappings-for-resource-resolution.html) 中。

通常，下列資源需要映射，但可能需要其他映射：

* 內容頁面的根節點（下面） `/content`)
* 頁面使用的設計節點（下） `/etc/designs`)
* 的 `/libs` 資料夾

在為內容頁建立映射後，要發現其他所需的映射，請使用Web瀏覽器在Web伺服器上開啟頁面。 在發佈實例的error.log檔案中，查找有關未找到的資源的消息。 以下示例消息表示 `/etc/clientlibs` 是必需的：

```shell
01.11.2012 15:59:24.601 *INFO* [10.36.34.243 [1351799964599] GET /etc/clientlibs/foundation/jquery.js HTTP/1.1] org.apache.sling.engine.impl.SlingRequestProcessorImpl service: Resource /content/sitea/etc/clientlibs/foundation/jquery.js not found
```

>[!NOTE]
>
>預設Apache Sling重寫器的連結檢查器轉換器會自動修改頁面中的超連結，以防止連結斷開。 但是，僅當連結目標是HTML或HTM檔案時，才執行連結重寫。 要更新指向其他檔案類型的連結，請建立變壓器元件並將其添加到HTML重寫器管線。

### 資源映射節點示例

下表列出了為branda.com域實現資源映射的節點。 為 `brandb.com` 域，例如 `/etc/map/http/brandb.com`。 在所有情況下，當頁面HTML中的引用在Sling上下文中無法正確解析時，都需要映射。

| 節點路徑 | 類型 | 屬性 |
|--- |--- |--- |
| `/etc/map/http/branda.com` | sling：映射 | 名稱：sling:internalRedirect類型：字串值：/content/sitea |
| `/etc/map/http/branda.com/libs` | sling：映射 | 名稱：sling:internal重定向 <br/>類型：字串 <br/>值：/libs |
| `/etc/map/http/branda.com/etc` | sling：映射 |  |
| `/etc/map/http/branda.com/etc/designs` | sling：映射 | 名稱：sling:internal重定向 <br/>VType:字串 <br/>值：/etc/designs |
| `/etc/map/http/branda.com/etc/clientlibs` | sling：映射 | 名稱：sling:internal重定向 <br/>VType:字串 <br/>值：/etc/clientlibs |

## 配置Dispatcher Flush複製代理 {#configuring-the-dispatcher-flush-replication-agent}

發佈實例上的Dispatcher Flush複製代AEM理必須將無效請求發送到正確的Dispatcher場。 要瞄準場，請使用Dispatcher Flush複製代理的URI屬性（在「傳輸」頁籤上）。 包括 `/virtualhost` 配置用於使快取失效的Dispatcher場的屬性：

`https://*webserver_name*:*port*/*virtual_host*/dispatcher/invalidate.cache`

例如，要使用 `farm_flush` 上例的場，URI為 `https://localhost:80/invalidation_only/dispatcher/invalidate.cache`。

![](assets/chlimage_1-12.png)

## Web伺服器重寫傳入的URL {#the-web-server-rewrites-incoming-urls}

使用Web伺服器的內部URL重寫功能將基於域的URL轉換為Dispatcher快取中的檔案路徑。 例如，客戶端請求 `https://brandA.com/en.html` 翻譯為 `content/sitea/en.html`檔案。

![](assets/chlimage_1-13.png)

Dispatcher快取鏡像儲存庫節點結構。 因此，當頁面激活時，生成的對快取頁面無效的請求不需要URL或路徑轉換。

![](assets/chlimage_1-14.png)

## 在Web伺服器上定義虛擬主機和重寫規則 {#define-virtual-hosts-and-rewrite-rules-on-the-web-server}

在Web伺服器上配置以下方面：

* 為每個Web域定義虛擬主機。
* 對於每個域，將文檔根配置為與包含域Web內容的儲存庫中的資料夾一致。
* 對於每個虛擬域，建立一個URL更名規則，將傳入的URL轉換為快取檔案的路徑。
* 每個虛擬域還必須包括與Dispatcher相關的配置，如 [正在安裝Dispatcher](dispatcher-install.md) 的子菜單。
* 必須將Dispatcher模組配置為使用Web伺服器已重寫的URL。 (請參閱 `DispatcherUseProcessedURL` 物業 [正在安裝Dispatcher](dispatcher-install.md)。)

以下示例httpd.conf檔案為Apache Web伺服器配置了兩個虛擬主機：

* 伺服器名（與域名一致） `brandA.com` （第16行）和 `brandB.com` （第32行）。

* 每個虛擬域的文檔根目錄是包含站點頁面的Dispatcher快取中的目錄。 （第20和33行）
* 每個虛擬域的URL重寫規則是一個規則運算式，用快取中的頁面路徑前置詞所請求頁面的路徑。 （第19和35行）
* 的 `DispatherUseProcessedURL` 屬性設定為 `1`。 （行10）

例如，Web伺服器在收到與 `https://brandA.com/en/products.html` URL:

* 將URL與具有 `ServerName` 共 `brandA.com.`
* 重寫要 `/content/sitea/en/products.html.`
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

### 配置Dispatcher場 {#configure-a-dispatcher-farm}

當Web伺服器重寫URL時， Dispatcher需要根據 [配置Dispatcher](dispatcher-configuration.md)。 支援Web伺服器虛擬主機和URL更名規則需要以下配置：

* 的 `/virtualhosts` 屬性必須包含所有VirtualHost定義的ServerName值。
* 的 `/statfileslevel` 屬性必須足夠高，以便在包含每個域的內容檔案的目錄中建立.stat檔案。

以下示例配置檔案基於該示例 `dispatcher.any` 隨Dispatcher一起安裝的檔案。 需要以下更改才能支援以前版本的Web伺服器配置 `httpd.conf` 檔案：

* 的 `/virtualhosts` 屬性使Dispatcher處理 `brandA.com` 和 `brandB.com` 域。 （第12行）
* 的 `/statfileslevel` 屬性設定為2，以便在包含域Web內容的每個目錄中建立stat檔案（第41行）: `/statfileslevel "2"`

與往常一樣，快取的文檔根與Web伺服器的文檔根（第40行）相同： `/usr/lib/apache/httpd-2.4.3/htdocs`

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
>由於定義了單個Dispatcher場，因此發佈實例上的Dispatcher Flush複製代理AEM不需要特殊配置。

## 重寫到非HTML檔案的連結 {#rewriting-links-to-non-html-files}

要重寫對副檔名為.html或.htm以外的檔案的引用，請建立一個Sling重寫器轉換器元件，並將其添加到預設重寫器管線中。

當資源路徑在Web伺服器上下文中無法正確解析時，重寫引用。 例如，當影像生成元件建立連結(如/content/sitea/en/products.navimage.png)時，需要轉換器。 的頂部導航元件 [如何建立功能齊全的Internet網站](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/the-basics.html) 建立此類連結。

的 [斯林重寫器](https://sling.apache.org/documentation/bundles/output-rewriting-pipelines-org-apache-sling-rewriter.html) 是後處理Sling輸出的模組。 重寫器的SAX流水線實現包括發生器、一個或多個變壓器和串列器：

* **生成器：** 解析Sling輸出流(HTML文檔)並在遇到特定元素類型時生成SAX事件。
* **變壓器：** 偵聽SAX事件，然後修改事件目標(HTML元素)。 重寫器管線包含零個或多個變壓器。 依次執行變壓器，將SAX事件按順序傳遞給下一個變壓器。
* **序列化程式：** 序列化輸出，包括每個變換器的修改。

![](assets/chlimage_1-15.png)

### 預設AEM重寫器管道 {#the-aem-default-rewriter-pipeline}

使AEM用預設的管線重寫器處理文本/html類型的文檔：

* 生成器在遇到a、img、區域、表單、基、連結、指令碼和正文元素時解析HTML文檔並生成SAX事件。 生成器別名為 `htmlparser`。
* 該管道包括以下變壓器： `linkchecker`。 `mobile`。 `mobiledebug`。 `contentsync`。 的 `linkchecker` 變壓器將路徑外部化為引用的HTML或HTM檔案，以防止斷開連結。
* 序列化程式寫入HTML輸出。 序列化程式別名為htmlwriter。

的 `/libs/cq/config/rewriter/default` 節點定義管線。

### 建立變壓器 {#creating-a-transformer}

執行以下任務以建立變壓器元件並在管線中使用：

1. 實施 `org.apache.sling.rewriter.TransformerFactory` 。 此類建立變壓器類的實例。 指定 `transformer.type` 屬性（轉換器別名）並將類配置為OSGi服務元件。
1. 實施 `org.apache.sling.rewriter.Transformer` 。 為了最小化工作量，您可以擴展 `org.apache.cocoon.xml.sax.AbstractSAXPipe` 類。 覆蓋startElement方法以自定義重寫行為。 此方法針對傳遞給變壓器的每個SAX事件調用。
1. 捆綁和部署類。
1. 將配置節點添加AEM到應用程式，以將變壓器添加到管線。

>[!TIP]
>相反，可以將TransformerFactory配置為將變壓器插入定義的每個重寫器。 因此，您不需要配置管道：
>
>* 設定 `pipeline.mode` 屬性 `global`。
>* 設定 `service.ranking` 屬性到正整數。
>* 不包括 `pipeline.type` 屬性。


>[!NOTE]
>
>使用 [多模組](https://helpx.adobe.com/tw/experience-manager/aem-previous-versions.html) 用於建立Maven項目的內容包Maven插件的原型。 POM自動建立和安裝內容包。

以下示例實現了重寫對影像檔案的引用的轉換器。

* MyRewriterTransformerFactory類實例化MyRewriterTransformer對象。 pipeline.type屬性將變壓器別名設定為mytransformer。 要將別名包括在管線中，管道配置節點將此別名包括在變壓器清單中。
* MyRewriterTransformer類覆蓋AbstractSAXTransfer類的startElement方法。 startElement方法為img元素重寫src屬性的值。

示例不可靠，不應在生產環境中使用。

### TransformerFactory實現示例 {#example-transformerfactory-implementation}

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

### 示例變壓器實現 {#example-transformer-implementation}

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

### 將變壓器添加到重寫器管線 {#adding-the-transformer-to-a-rewriter-pipeline}

建立JCR節點，該節點定義使用變壓器的管線。 以下節點定義會建立處理文本/html檔案的管道。 使用默AEM認生成器和HTML分析器。

>[!NOTE]
>
>如果設定了「變壓器」屬性 `pipeline.mode` 至 `global`，您無需配置管道。 的 `global` 模式將變壓器插入所有管線。

### 重寫器配置節點 — XML表示法 {#rewriter-configuration-node-xml-representation}

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

下圖顯示了節點的CRXDE Lite表示：

![](assets/chlimage_1-16.png)
