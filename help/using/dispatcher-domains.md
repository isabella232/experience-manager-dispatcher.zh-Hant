---
title: '在多個網域中使用 Dispatcher '
seo-title: Using Dispatcher with Multiple Domains
description: 了解如何使用 Dispatcher，以便在多個網域中處理頁面請求。
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
ht-degree: 100%

---

# 在多個網域中使用 Dispatcher {#using-dispatcher-with-multiple-domains}

>[!NOTE]
>
>Dispatcher 版本與 AEM 無關。 如果您依循了內嵌到 AEM 或 CQ 文件中的 Dispatcher 文件的連結，您可能會被重新導向至本頁。

使用 Dispatcher 以便在多個網域中處理頁面請求，同時支援以下情況：

* 這兩個網域的網頁內容都儲存在單一 AEM 存放庫中。
* 可以個別針對每個網域讓 Dispatcher 快取中的檔案失效。

例如，一家公司為其兩個品牌發佈網站：品牌 A 和品牌 B。網站頁面的內容會在 AEM 中編寫，並儲存在相同的存放庫工作區中：

```
/
| - content  
   | - sitea  
       | - content nodes  
   | - siteb  
       | - content nodes
```

`BrandA.com` 的頁面會儲存在 `/content/sitea` 底下。 `https://BrandA.com/en.html` URL 的用戶端請求會傳回 `/content/sitea/en` 節點的轉譯頁面。 同樣地，`BrandB.com` 的頁面會儲存在 `/content/siteb` 底下。

使用 Dispatcher 快取內容時，在用戶端 HTTP 請求中的頁面 URL、快取中對應檔案的路徑及存放庫中對應檔案的路徑之間必須建立關聯。

## 用戶端請求

當用戶端傳送 HTTP 請求給網頁伺服器時，請求的頁面的 URL 必須解析為 Dispatcher 快取中的內容，並在最後解析為存放庫中的內容。

![](assets/chlimage_1-8.png)

1. 網域名稱系統會發現為 HTTP 請求中的網域名稱登錄的網頁伺服器的 IP 位址。
1. HTTP 請求會傳送給網頁伺服器。
1. HTTP 請求會傳遞給 Dispatcher。
1. Dispatcher 會判斷快取檔案是否有效。 如果有效，則會將快取檔案提供給用戶端。
1. 如果快取檔案無效，Dispatcher 會向 AEM 發佈執行個體請求最新轉譯的頁面。

## 快取失效

當 Dispatcher Flush 複寫代理程式請求 Dispatcher 讓快取檔案失效時，存放庫中內容的路徑必須解析為快取中的內容。

![](assets/chlimage_1-9.png)

1. AEM 編寫執行個體上會啟用一個頁面，並將內容複寫到發佈執行個體。
1. Dispatcher Flush 代理程式會呼叫 Dispatcher，好讓複寫內容的快取失效。
1. Dispatcher 會觸及一個或多個 .stat 檔案，好讓快取檔案失效。

若要在多個網域中使用 Dispatcher，您需要設定 AEM、Dispatcher 和您的網頁伺服器。 此頁面上所述的解決方案是通用的，適用於大多數環境。 由於某些 AEM 拓撲很複雜，您的解決方案可能需要進一步的自訂設定才能解決特定問題。 您可能需要改寫範例，以符合您現有的 IT 基礎結構和管理政策。

## URL 對應 {#url-mapping}

若要讓網域 URL 和內容路徑能夠解析為快取檔案，必須在此過程中的某個時間點轉譯檔案路徑或頁面 URL。 我們提供以下常用策略的說明，其中路徑或 URL 的轉譯會發生在此過程中的不同時間點：

* (建議) AEM 發佈執行個體使用 Sling 對應來解析資源，以實作內部 URL 重寫規則。 網域 URL 會解譯成內容存放庫路徑。 請參閱 [AEM 重寫傳入的 URL](#aem-rewrites-incoming-urls)。
* 網頁伺服器會使用內部 URL 重寫規則，將網域 URL 轉譯成快取路徑。 請參閱[網頁伺服器重寫傳入的 URL](#the-web-server-rewrites-incoming-urls)。

通常我們會想要針對網頁使用簡短 URL。 通常頁面 URL 會反映包含網頁內容的存放庫資料夾的結構。 不過，URL 不會揭露最上層的存放庫節點，例如 `/content`。 用戶端不一定會察覺 AEM 存放庫的結構。

## 一般需求 {#general-requirements}

您的環境必須實作以下設定，以支援 Dispatcher 在多個網域中運作：

* 每個網域的內容位在存放庫的不同分支中 (請參閱以下的環境範例)。
* 在 AEM 發佈執行個體上設定 Dispatcher Flush 複寫代理程式。 (請參閱[使發佈執行個體中的 Dispatcher 快取失效](page-invalidate.md)。)
* 網域名稱系統會將網域名稱解析為網頁伺服器的 IP 位址。
* Dispatcher 快取會反映 AEM 內容存放庫的目錄結構。 網頁伺服器的主目錄底下的檔案路徑與存放庫中的檔案路徑相同。

## 提供的範例適用的環境 {#environment-for-the-provided-examples}

提供的範例解決方案適用於具有以下特性的環境：

* 在 Linux 系統上部署 AEM 編寫和發佈執行個體。
* Apache HTTPD 是網頁伺服器，部署在 Linux 系統上。
* AEM 內容存放庫和網頁伺服器的主目錄會使用以下檔案結構 (Apache Web Server 的主目錄為 /`usr/lib/apache/httpd-2.4.3/htdocs)`：

   **存放庫**

```
  | - /content  
    | - sitea  
  |    | - content nodes 
    | - siteb  
       | - conent nodes
```

**網頁伺服器的主目錄**

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

## AEM 重寫傳入的 URL {#aem-rewrites-incoming-urls}

用於解析資源的 Sling 對應可讓您將傳入的 URL 與 AEM 內容路徑建立關聯。 請在 AEM 發佈執行個體上建立對應，好讓來自 Dispatcher 的轉譯器請求可解析成存放庫中的正確內容。

用於頁面轉譯的 Dispatcher 請求會使用從網頁伺服器傳遞的 URL 來識別此頁面。 當 URL 包含網域名稱時，Sling 對應會將 URL 解析成內容。 下圖說明 `branda.com/en.html` URL 對應到 `/content/sitea/en` 節點的情況。

![](assets/chlimage_1-10.png)

Dispatcher 快取會反映存放庫節點的結構。 因此，當發生頁面啟用時，導致快取頁面失效的請求不需要轉譯 URL 或路徑。

![](assets/chlimage_1-11.png)

## 在網頁伺服器上定義虛擬主機 {#define-virtual-hosts-on-the-web-server}

在網頁伺服器上定義虛擬主機，好讓不同主目錄可以指派給每個網域：

* 網頁伺服器必須針對您的每個網域定義虛擬網域。
* 針對每個網域，將主目錄設定為與存放庫中包含網域的網頁內容的資料夾一致。
* 每個虛擬網域也都必須包含 Dispatcher 相關設定，如[安裝 Dispatcher](dispatcher-install.md) 頁面上所述。

以下範例 `httpd.conf` 檔案為 Apache Web Server 設定兩個虛擬網域：

* 伺服器名稱 (與網域名稱一致) 為 branda.com (第 16 行) 和 brandb.com (第 30 行)。
* 每個虛擬網域的主目錄為 Dispatcher 快取中包含網站頁面的目錄。 (第 17 和 31 行)

使用此設定時，網頁伺服器在收到 `https://branda.com/en/products.html` 的請求時會執行以下操作：

* 將 URL 與 `ServerName` 為 `branda.com.` 的虛擬主機建立關聯

* 將 URL 轉送到 Dispatcher。

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

請注意，虛擬主機會繼承在主要伺服器區段中所設定的 [DispatcherConfig](dispatcher-install.md#main-pars-67-table-7) 屬性值。 虛擬主機可包含其自身的 DispatcherConfig 屬性以覆寫主要伺服器設定。

### 設定 Dispatcher 處理多個網域 {#configure-dispatcher-to-handle-multiple-domains}

若要支援包含網域名稱及其對應虛擬主機的 URL，請定義以下 Dispatcher 陣列：

* 為每部虛擬主機設定 Dispatcher 陣列。 這些陣列會針對每個網域處理來自網頁伺服器的請求、檢查快取檔案，並向轉譯器要求頁面。
* 設定用於讓快取內容失效的 Dispatcher 陣列，無論該內容屬於哪個網域。 此陣列會處理來自 Flush Dispatcher 複寫代理程式的檔案失效請求。

### 為虛擬主機建立 Dispatcher 陣列

虛擬主機的陣列必須擁有以下設定，好讓用戶端 HTTP 請求中的 URL 可解析成 Dispatcher 快取中的正確檔案：

* `/virtualhosts` 屬性會設定為網域名稱。 此屬性可讓 Dispatcher 將陣列與網域建立關聯。
* `/filter` 屬性允許存取在網域名稱部分後面被截斷的請求 URL 的路徑。 例如，對於 `https://branda.com/en.html` URL，路徑會解譯成 `/en.html`，好讓篩選條件必須允許對此路徑的存取。

* `/docroot` 屬性會設定為 Dispatcher 快取中網域的網站內容的根目錄的路徑。 此路徑會用作來自原始請求的串連 URL 的前置詞。 例如，`/usr/lib/apache/httpd-2.4.3/htdocs/sitea` 的 docroot 會使得 `https://branda.com/en.html` 的請求解析成 `/usr/lib/apache/httpd-2.4.3/htdocs/sitea/en.html` 檔案。

此外，AEM 發佈執行個體必須指定為虛擬主機的轉譯器。 視需要設定其他陣列屬性。 以下程式碼是 branda.com 網域的簡化陣列設定：

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

### 建立 Dispatcher 陣列讓快取失效

處理讓快取檔案失效的請求時，需要 Dispatcher 陣列。 此陣列必須能夠存取每部虛擬主機的 docroot 中的 .stat 檔案。

以下屬性設定可讓 Dispatcher 從快取中的檔案解析 AEM 內容存放庫中的檔案：

* `/docroot` 屬性會設定為網頁伺服器的預設 docroot。 通常這是 `/content` 資料夾建立所在的目錄。 Linux 上的 Apache 的範例值為 `/usr/lib/apache/httpd-2.4.3/htdocs`。
* `/filter` 屬性允許存取 `/content` 目錄底下的檔案。

`/statfileslevel` 屬性層級必須夠高，才能在每部虛擬主機的根目錄中建立 .stat 檔案。 此屬性可讓每個網域的快取分別失效。 對於範例設定，當 `/statfileslevel` 值為 `2` 時，會在 `*docroot*/content/sitea` 目錄和 `*docroot*/content/siteb` 目錄中建立 .stat 檔案。

此外，發佈執行個體必須指定為虛擬主機的轉譯器。 視需要設定其他陣列屬性。 以下程式碼是用於讓快取失效的陣列的簡化設定：

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

當您啟動網頁伺服器時，Dispatcher 記錄 (偵錯模式下) 會指示所有陣列的初始化：

```shell
Dispatcher initializing (build 4.1.2)
[Fri Nov 02 16:27:18 2012] [D] [24974(140006182991616)] farms[farm_sitea].cache.docroot = /usr/lib/apache/httpd-2.4.3/htdocs/content/sitea
[Fri Nov 02 16:27:18 2012] [D] [24974(140006182991616)] farms[farm_siteb].cache.docroot = /usr/lib/apache/httpd-2.4.3/htdocs/content/siteb
[Fri Nov 02 16:27:18 2012] [D] [24974(140006182991616)] farms[farm_flush].cache.docroot = /usr/lib/apache/httpd-2.4.3/htdocs
[Fri Nov 02 16:27:18 2012] [I] [24974(140006182991616)] Dispatcher initialized (build 4.1.2)
```

### 設定 Sling 對應來解析資源 {#configure-sling-mapping-for-resource-resolution}

使用 Sling 對應來解析資源，好讓網域型 URL 可解析成 AEM 發佈執行個體上的內容。 資源對應會將 Dispatcher 傳入的 URL (原本來自用戶端 HTTP 請求) 解譯成內容節點。

若要了解 Sling 資源對應，請參閱 Sling 文件中的[資源解析的對應](https://sling.apache.org/site/mappings-for-resource-resolution.html)。

通常以下資源需要對應，但可能也需要其他對應：

* 內容頁面的根節點 (在 `/content` 底下)
* 頁面使用的設計節點 (在 `/etc/designs` 底下)
* `/libs` 資料夾

當您為內容頁面建立對應後，若要探索其他必要的對應，請使用網頁瀏覽器在網頁伺服器上開啟頁面。 在發佈執行個體的 error.log 檔案中，找出有關找不到資源的訊息。 以下範例訊息指示需要 `/etc/clientlibs` 的對應：

```shell
01.11.2012 15:59:24.601 *INFO* [10.36.34.243 [1351799964599] GET /etc/clientlibs/foundation/jquery.js HTTP/1.1] org.apache.sling.engine.impl.SlingRequestProcessorImpl service: Resource /content/sitea/etc/clientlibs/foundation/jquery.js not found
```

>[!NOTE]
>
>預設 Apache Sling 重寫程式的 linkchecker 轉換器會自動修改頁面中的超連結，以避免連結失效。 不過，只有當連結目標是 HTML 或 HTM 檔案時，才會執行連結重寫。 若要更新其他檔案類型的連結，請建立轉換器元件，並將其新增到 HTML 重寫程式管道。

### 資源對應節點範例

下表列出為 branda.com 網域實作資源對應的節點。 會針對 `brandb.com` 網域建立類似的節點，例如 `/etc/map/http/brandb.com`。 在所有情況下，當頁面 HTML 中的參照無法在 Sling 的上下文中正確解析時，都需要對應。

| 節點路徑 | 類型 | 屬性 |
|--- |--- |--- |
| `/etc/map/http/branda.com` | sling:Mapping | 名稱：sling:internalRedirect；類型：字串；值：/content/sitea |
| `/etc/map/http/branda.com/libs` | sling：映射 | 名稱：sling:internalRedirect <br/>類型：字串<br/>值：/libs |
| `/etc/map/http/branda.com/etc` | sling：映射 |  |
| `/etc/map/http/branda.com/etc/designs` | sling：映射 | 名稱：sling:internalRedirect <br/>VType：字串<br/>VValue：/etc/designs |
| `/etc/map/http/branda.com/etc/clientlibs` | sling：映射 | 名稱：sling:internalRedirect <br/>VType：字串<br/>VValue：/etc/clientlibs |

## 設定 Dispatcher Flush 複寫代理程式 {#configuring-the-dispatcher-flush-replication-agent}

AEM 發佈執行個體上的 Dispatcher Flush 複寫代理程式必須傳送失效請求給正確的 Dispatcher 陣列。 若要鎖定某個陣列，請使用 Dispatcher Flush 複寫代理程式的 URI 屬性 (在「傳輸」索引標籤上)。 針對設定用於讓快取失效的 Dispatcher 陣列包含 `/virtualhost` 屬性的值：

`https://*webserver_name*:*port*/*virtual_host*/dispatcher/invalidate.cache`

例如，若要使用前一個範例的 `farm_flush` 陣列，URI 會是 `https://localhost:80/invalidation_only/dispatcher/invalidate.cache`。

![](assets/chlimage_1-12.png)

## 網頁伺服器重寫傳入的 URL {#the-web-server-rewrites-incoming-urls}

使用網頁伺服器的內部 URL 重寫功能可將網域型 URL 轉譯成 Dispatcher 快取中的檔案路徑。 例如，對 `https://brandA.com/en.html` 頁面的用戶端請求會轉譯成網頁伺服器的主目錄中的 `content/sitea/en.html` 檔案。

![](assets/chlimage_1-13.png)

Dispatcher 快取會反映存放庫節點的結構。 因此，當發生頁面啟用時，導致快取頁面失效的請求不需要轉譯 URL 或路徑。

![](assets/chlimage_1-14.png)

## 在網頁伺服器上定義虛擬主機並重寫規則 {#define-virtual-hosts-and-rewrite-rules-on-the-web-server}

在網頁伺服器上設定以下方面：

* 為您的每個網域定義虛擬主機。
* 針對每個網域，將主目錄設定為與存放庫中包含網域的網頁內容的資料夾一致。
* 針對每個虛擬網域，建立 URL 重新命名規則，以便將傳入的 URL 轉譯成快取檔案的路徑。
* 每個虛擬網域也都必須包含 Dispatcher 相關設定，如[安裝 Dispatcher](dispatcher-install.md) 頁面上所述。
* 必須將 Dispatcher 模組設定為使用網頁伺服器已重寫的 URL。 (請參閱[安裝 Dispatcher](dispatcher-install.md) 中的 `DispatcherUseProcessedURL` 屬性。)

以下範例 httpd.conf 檔案為 Apache Web Server 設定兩部虛擬主機：

* 伺服器名稱 (與網域名稱一致) 為 `brandA.com` (第 16 行) 和 `brandB.com` (第 32 行)。

* 每個虛擬網域的主目錄為 Dispatcher 快取中包含網站頁面的目錄。 (第 20 和 33 行)
* 每個虛擬網域的 URL 重寫規則都是規則運算式，該運算式使用快取中頁面的路徑當作請求頁面的路徑的前置詞。 (第 19 和 35 行)
* `DispatherUseProcessedURL` 屬性會設定為 `1`。 (第 10 行)

例如，網頁伺服器在收到帶有 `https://brandA.com/en/products.html` URL 的請求時會執行以下操作：

* 將 URL 與 `ServerName` 為 `brandA.com.` 的虛擬主機建立關聯
* 將 URL 重寫為 `/content/sitea/en/products.html.`
* 將 URL 轉送到 Dispatcher。

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

### 設定 Dispatcher 陣列 {#configure-a-dispatcher-farm}

當網頁伺服器重寫 URL 時，Dispatcher 會根據[設定 Dispatcher](dispatcher-configuration.md) 中的內容要求定義單一陣列。 支援網頁伺服器虛擬主機和 URL 重新命名規則需要以下設定：

* 在所有 VirtualHost 定義中，`/virtualhosts` 屬性都必須包含 ServerName 值。
* `/statfileslevel` 屬性層級必須夠高，才能在包含每個網域的內容檔案的目錄中建立 .stat 檔案。

以下設定檔範例是根據 Dispatcher 所安裝的範例 `dispatcher.any` 檔案。 支援先前 `httpd.conf` 檔案的網頁伺服器設定需要以下變更：

* `/virtualhosts` 屬性會讓 Dispatcher 處理對 `brandA.com` 和 `brandB.com` 網域的請求。 (第 12 行)
* `/statfileslevel` 屬性設為 2，所以在包含網域的網頁內容的每個目錄中都會建立 stat 檔案 (第 41 行)：`/statfileslevel "2"`

如往常一樣，快取的主目錄與網頁伺服器的主目錄相同 (第 40 行)：`/usr/lib/apache/httpd-2.4.3/htdocs`

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
>由於定義了單一 Dispatcher 陣列，所以 AEM 發佈執行個體上的 Dispatcher Flush 複寫代理程式不需要特殊設定。

## 重寫非 HTML 檔案的連結 {#rewriting-links-to-non-html-files}

若要重寫對副檔名為 .html 或 .htm 以外的檔案的參照，請建立 Sling 重寫程式轉換器元件，並將其新增到預設重寫程式管道。

當資源路徑無法在網頁伺服器上下文中正確解析時，請重寫參照。 例如，當產生影像的元件建立類似 /content/sitea/en/products.navimage.png 等連結時，需要使用轉換器。 [如何建立完整功能的網站](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/the-basics.html)中的 topnav 元件會建立這種連結。

[Sling 重寫程式](https://sling.apache.org/documentation/bundles/output-rewriting-pipelines-org-apache-sling-rewriter.html)是對 Sling 輸出進行後置處理的模組。 重寫程式的 SAX 管道實作是由產生器、一個或多個轉換器和序列化程式所組成：

* **產生器：**&#x200B;在遇到特定元素類型時剖析 Sling 輸出資料流 (HTML 文件) 並產生 SAX 事件。
* **轉換器：**&#x200B;偵聽 SAX 事件，並在之後修改事件目標 (HTML 元素)。 重寫程式管道包含零個或多個轉換器。 轉換器會依序執行，將 SAX 事件傳遞給序列中的下一個轉換器。
* **序列化程式：**&#x200B;將輸出序列化，包括來自每個轉換器的修改。

![](assets/chlimage_1-15.png)

### AEM 預設重寫程式管道 {#the-aem-default-rewriter-pipeline}

AEM 會使用處理文字/html 類型的文件的預設管道重寫程式：

* 產生器在遇到 a、img、area、form、base、link、script 和 body 元素時會剖析 HTML 文件並產生 SAX 事件。 產生器別名為 `htmlparser`。
* 此管道包含以下轉換器：`linkchecker`、`mobile`、`mobiledebug`、`contentsync`。 `linkchecker` 轉換器會將參照的 HTML 或 HTM 檔案的路徑外部化，以避免連結失效。
* 序列化程式會撰寫 HTML 輸出。 序列化程式別名為 htmlwriter。

`/libs/cq/config/rewriter/default` 節點會定義管道。

### 建立轉換器 {#creating-a-transformer}

執行以下工作來建立轉換器元件，並在管道中使用它：

1. 實作 `org.apache.sling.rewriter.TransformerFactory` 介面。 此類別會建立您的轉換器類別的執行個體。 為 `transformer.type` 屬性 (轉換器別名) 指定值，並將此類別設為 OSGi 服務元件。
1. 實作 `org.apache.sling.rewriter.Transformer` 介面。 若要將工作減到最少，您可以擴充 `org.apache.cocoon.xml.sax.AbstractSAXPipe` 類別。 覆寫 startElement 方法來自訂重寫行為。 傳遞給轉換器的每個 SAX 事件都會呼叫此方法。
1. 組合並部署類別。
1. 在您的 AEM 應用程式中新增設定節點，以將轉換器新增到管道中。

>[!TIP]
>您可以改為將 TransformerFactory 設定為將轉換器插入到每個定義的重寫程式中。 之後您不需要設定管道：
>
>* 將 `pipeline.mode` 屬性設為 `global`。
>* 將 `service.ranking` 屬性設為正整數。
>* 不要包含 `pipeline.type` 屬性。


>[!NOTE]
>
>使用 Content Package Maven 外掛程式的 [multimodule](https://helpx.adobe.com/experience-manager/aem-previous-versions.html) 原型來建立您的 Maven 專案。 POM 會自動建立及安裝內容套件。

以下範例實作的轉換器會重寫影像檔案的參照。

* MyRewriterTransformerFactory 類別會將 MyRewriterTransformer 物件具現化。 pipeline.type 屬性將轉換器別名設為 mytransformer。 為了在管道中包含此別名，管道設定節點會在轉換器清單中包含此別名。
* MyRewriterTransformer 類別會覆寫 AbstractSAXTransformer 類別的 startElement 方法。 startElement 方法會針對 img 元素重寫 src 屬性的值。

這些範例不可靠，不應該用在生產環境中。

### TransformerFactory 實作範例 {#example-transformerfactory-implementation}

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

### 轉換器實作範例 {#example-transformer-implementation}

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

### 將轉換器新增到重寫程式管道 {#adding-the-transformer-to-a-rewriter-pipeline}

建立 JCR 節點，此節點會定義使用您的轉換器的管道。 下列節點定義會建立處理文字/html 檔案的管道。 會使用預設 AEM 產生器及 HTML 的剖析器。

>[!NOTE]
>
>如果您將轉換器屬性 `pipeline.mode` 設為 `global`，您就不需要設定管道。 `global` 模式會將轉換器插入到所有管道中。

### 重寫程式設定節點 - XML 表示方式 {#rewriter-configuration-node-xml-representation}

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

下圖顯示節點的 CRXDE Lite 表示方式：

![](assets/chlimage_1-16.png)
