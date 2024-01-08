---
title: 設定 Dispatcher
description: 了解如何設定 Dispatcher。了解對 IPv4 和 IPv6 的支援、設定檔案、環境變數、為執行個體命名、定義陣列、識別虛擬主機等。
exl-id: 91159de3-4ccb-43d3-899f-9806265ff132
source-git-commit: 410346694a134c0f32a24de905623655f15269b4
workflow-type: tm+mt
source-wordcount: '8857'
ht-degree: 99%

---

# 設定 Dispatcher {#configuring-dispatcher}

>[!NOTE]
>
>Dispatcher 版本與 AEM 無關。如果您依循了內嵌到舊版 AEM 文件中的 Dispatcher 文件的連結，您可能會被重新導向至本頁。

以下幾節將說明如何設定 Dispatcher 的各個層面。

## 對 IPv4 和 IPv6 的支援 {#support-for-ipv-and-ipv}

AEM 和 Dispatcher 的所有元素都可以安裝在 IPv4 和 IPv6 網路上。請參閱 [IPV4 和 IPV6](https://experienceleague.adobe.com/docs/experience-manager-65/deploying/introduction/technical-requirements.html?lang=zh-Hant#ipv-and-ipv)。

## Dispatcher 設定檔案 {#dispatcher-configuration-files}

根據預設，Dispatcher 設定會儲存在 `dispatcher.any` 文字檔中，但是您在安裝期間可以變更此檔案的名稱和位置。

此設定檔案包含一系列的單一值或多值屬性，這些屬性用來控制 Dispatcher 的行為：

* 屬性名稱的前置詞為正斜線 `/`。
* 多值屬性會使用大括號 `{ }` 括住子項目。

範例設定的結構如下所示：

```xml
# name of the dispatcher
/name "internet-server"

# each farm configures a set off (loadbalanced) renders
/farms
 {
  # first farm entry (label is not important, just for your convenience)
   /website
     {  
     /clientheaders
       {
       # List of headers that are passed on
       }
     /virtualhosts
       {
       # List of URLs for this Web site
       }
     /sessionmanagement
       {
       # settings for user authentification
       }
     /renders
       {
       # List of AEM instances that render the documents
       }
     /filter
       {
       # List of filters
       }
     /vanity_urls
       {
       # List of vanity URLs
       }
     /cache
       {
       # Cache configuration
       /rules
         {
         # List of cachable documents
         }
       /invalidate
         {
         # List of auto-invalidated documents
         }
       }
     /statistics
       {
       /categories
         {
         # The document categories that are used for load balancing estimates
         }
       }
     /stickyConnectionsFor "/myFolder"
     /health_check
       {
       # Page gets contacted when an instance returns a 500
       }
     /retryDelay "1"
     /numberOfRetries "5"
     /unavailablePenalty "1"
     /failover "1"
     }
 }
```

您可以包含有助於設定的其他檔案：

* 如果設定檔案很大，您可以將其分割成幾個較小的檔案 (以便於管理) 並包含當中各個檔案。
* 包含自動產生的檔案。

例如，若要在 /farms 設定中包含 myFarm.any 檔案，請使用以下程式碼：

```xml
/farms
  {
  $include "myFarm.any"
  }
```

要指定要包含的檔案範圍，請使用星號 (`*`) 作為萬用字元。

例如，如果檔案 `farm_1.any` 到 `farm_5.any` 包含陣列一到五的設定，您可依照以下方式來包含這些陣列：

```xml
/farms
  {
  $include "farm_*.any"
  }
```

## 使用環境變數 {#using-environment-variables}

您可以在 dispatcher.any 檔案中的字串-值屬性內使用環境變數，而不是以硬式編碼方式編寫這些值。若要包含環境變數的值，請使用 `${variable_name}` 格式。

例如，如果 dispatcher.any 檔案位於與快取目錄相同的目錄中，則可以針對 [docroot](#specifying-the-cache-directory) 屬性使用下列值：

```xml
/docroot "${PWD}/cache"
```

還有另一個範例，如果您建立名為 `PUBLISH_IP` 的環境變數來儲存 AEM Publish 執行個體的主機名稱，則可以使用以下的 [/renders](#defining-page-renderers-renders) 屬性設定：

```xml
/renders {
  /0001 {
    /hostname "${PUBLISH_IP}"
    /port "8443"
  }
}
```

## 為 Dispatcher 執行個體命名 {#naming-the-dispatcher-instance-name}

使用 `/name` 屬性可指定唯一名稱來識別您的 Dispatcher 執行個體。`/name` 屬性是設定結構中的最上層屬性。

## 定義陣列 {#defining-farms-farms}

`/farms` 屬性可定義一組或多組 Dispatcher 行為，每一組行為會與不同網站或 URL 有關聯。`/farms` 屬性可以包含單一陣列或多個陣列：

* 當您希望 Dispatcher 以相同方式處理您的所有網頁或網站時，請使用單一陣列。
* 當您網站的不同區域或是不同網站需要不同的 Dispatcher 行為時，請建立多個陣列。

`/farms` 屬性是設定結構中的最上層屬性。若要定義陣列，請在 `/farms` 屬性中新增子屬性。使用在 Dispatcher 執行個體中可唯一識別陣列的屬性名稱。

`/farmname` 屬性是多值屬性，而且包含可定義 Dispatcher 行為的其他屬性：

* 陣列套用到的頁面的 URL。
* 用於轉譯文件的一或多個服務 URL (通常是 AEM Publish 執行個體的 URL)。
* 用於讓多個文件轉譯器負載平衡的統計資料。
* 其他幾個行為，例如要快取哪些檔案以及在何處快取。

值可包含任何英數 (a-z、0-9) 字元。 以下範例說明名為 `/daycom` 和 `/docsdaycom` 的兩個陣列的骨架定義：

```xml
#name of dispatcher
/name "day sites"

#farms section defines a list of farms or sites
/farms
{
   /daycom
   {
       ...
   }
   /docdaycom
   {
      ...
   }
}
```

>[!NOTE]
>
>如果您使用一個以上的轉譯陣列，則會從下到上評估此清單。此流程在為您的網站定義[虛擬主機](#identifying-virtual-hosts-virtualhosts)時具有相關性。

每個陣列屬性都可包含以下子屬性：

| 屬性名稱 | 說明 |
|--- |--- |
| [/homepage](#specify-a-default-page-iis-only-homepage) | 預設首頁 (選擇性)(僅限 IIS) |
| [/clientheaders](#specifying-the-http-headers-to-pass-through-clientheaders) | 要傳遞的用戶端 HTTP 請求中的標題。 |
| [/virtualhosts](#identifying-virtual-hosts-virtualhosts) | 此陣列的虛擬主機。 |
| [/sessionmanagement](#enabling-secure-sessions-sessionmanagement) | 對工作階段管理和驗證的支援。 |
| [/renders](#defining-page-renderers-renders) | 提供轉譯的頁面的伺服器 (通常是 AEM Publish 執行個體)。 |
| [/filter](#configuring-access-to-content-filter) | 定義 Dispatcher 允許存取的 URL。 |
| [/vanity_urls](#enabling-access-to-vanity-urls-vanity-urls) | 設定對虛名 URL 的存取權。 |
| [/propagateSyndPost](#forwarding-syndication-requests-propagatesyndpost) | 對轉送整合請求的支援。 |
| [/cache](#configuring-the-dispatcher-cache-cache) | 設定快取行為。 |
| [/statistics](#configuring-load-balancing-statistics) | 為負載平衡計算定義統計類別。 |
| [/stickyConnectionsFor](#identifying-a-sticky-connection-folder-stickyconnectionsfor) | 包含黏性文件的資料夾。 |
| [/health_check](#specifying-a-health-check-page) | 用來判斷伺服器是否可用的 URL。 |
| [/retryDelay](#specifying-the-page-retry-delay) | 重試失敗的連線之前的延遲時間。 |
| [/unavailablePenalty](#reflecting-server-unavailability-in-dispatcher-statistics) | 影響負載平衡計算之統計資料的懲罰。 |
| [/failover](#using-the-failover-mechanism) | 當原始請求失敗時重新傳送請求給不同轉譯器。 |
| [/auth_checker](permissions-cache.md) | 如需了解權限敏感型快取，請參閱[快取安全內容](permissions-cache.md)。 |

## 指定預設頁面 (僅限 IIS) - /homepage {#specify-a-default-page-iis-only-homepage}

>[!CAUTION]
>
>`/homepage` 參數 (僅限 IIS) 不再有效。您應該改用 [IIS URL Rewrite 模組](https://learn.microsoft.com/en-us/iis/extensions/url-rewrite-module/using-the-url-rewrite-module)。
>
>如果您使用 Apache，則應該使用 `mod_rewrite` 模組。請參閱 Apache 網站文件以取得 `mod_rewrite` 的相關資訊 (例如 [Apache 2.4](https://httpd.apache.org/docs/current/mod/mod_rewrite.html))。在使用 `mod_rewrite` 時，建議最好使用標幟 &#39;passthrough|PT&#39; (傳遞給下一個處理常式)，以強制重寫引擎將內部 `uri` 結構的 `request_rec` 欄位設定為 `filename` 欄位的值。

<!-- 

Comment Type: draft

<p>The optional /homepage parameter specifies the page that Dispatcher returns when a client requests an undeterminable page or file.</p> 
<p>Typically this situation occurs when a user specifies an URL for which neither IIS or AEM provides an automatic redirection target. For example, if the AEM render instance is shut down after the content is cached, the content redirect URL is unavailable.</p> 
<p>The following example configuration displays the <span class="code">index.html</span> page in such circumstances:</p>

 -->

<!-- 

Comment Type: draft

<codeblock gutter="true" class="syntax xml">
  /homepage&nbsp;"/index.html" 
</codeblock>

 -->

<!-- 

Comment Type: draft

<p>The <span class="code">/homepage</span> section is located inside the <span class="code">/farms</span> section, for example:<br /> </p>

 -->

<!-- 

Comment Type: draft

<codeblock gutter="true" class="syntax xml">
  #name&nbsp;of&nbsp;dispatcher!!discoiqbr!!/name&nbsp;"day&nbsp;sites"!!discoiqbr!!!!discoiqbr!!#farms&nbsp;section&nbsp;defines&nbsp;a&nbsp;list&nbsp;of&nbsp;farms&nbsp;or&nbsp;sites!!discoiqbr!!/farms!!discoiqbr!!{!!discoiqbr!!&nbsp;&nbsp;&nbsp;/daycom!!discoiqbr!!&nbsp;&nbsp;&nbsp;{!!discoiqbr!!&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;/homepage&nbsp;"/index.html"!!discoiqbr!!&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;...!!discoiqbr!!&nbsp;&nbsp;&nbsp;}!!discoiqbr!!&nbsp;&nbsp;&nbsp;/docdaycom!!discoiqbr!!&nbsp;&nbsp;&nbsp;{!!discoiqbr!!&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;...!!discoiqbr!!&nbsp;&nbsp;&nbsp;}!!discoiqbr!!} 
</codeblock>

 -->

## 指定要傳遞的 HTTP 標題 {#specifying-the-http-headers-to-pass-through-clientheaders}

`/clientheaders` 屬性會定義 Dispatcher 從用戶端 HTTP 請求傳送給轉譯器 (AEM 執行個體) 的 HTTP 標題清單。

Dispatcher 預設會將標準 HTTP 標題轉送給 AEM 執行個體。在某些情況下，您可能會想要轉送其他標題或移除特定標題：

* 在 HTTP 請求中新增您的 AEM 執行個體所期望的標題，例如自訂標題。
* 移除僅與網頁伺服器有關的標頭，例如驗證標頭。

如果您自訂要傳遞的標頭組合，則必須指定詳盡的標頭清單，包括通常在預設情況下包含的該些標頭。

例如，為發佈執行個體處理頁面啟用請求的 Dispatcher 執行個體需要 `/clientheaders` 區段中的 `PATH` 標題。`PATH` 標頭會啟用複寫代理程式與 Dispatcher 之間的通訊。

以下程式碼為 `/clientheaders` 的設定範例：

```shell
/clientheaders
  {
  "CSRF-Token"
  "X-Forwarded-Proto"
  "referer"
  "user-agent"
  "authorization"
  "from"
  "content-type"
  "content-length"
  "accept-charset"
  "accept-encoding"
  "accept-language"
  "accept"
  "host"
  "max-forwards"
  "proxy-authorization"
  "proxy-connection"
  "range"
  "cookie"
  "cq-action"
  "cq-handle"
  "handle"
  "action"
  "cqstats"
  "depth"
  "translate"
  "expires"
  "date"
  "dav"
  "ms-author-via"
  "if"
  "lock-token"
  "x-expected-entity-length"
  "destination"
  "PATH"
  }
```

## 識別虛擬主機 {#identifying-virtual-hosts-virtualhosts}

`/virtualhosts` 屬性會定義 Dispatcher 在此陣列中接受的所有主機名稱/URI 組合的清單。您可以使用星號 (`*`) 當作萬用字元。/ `virtualhosts` 屬性的值會使用以下格式：

```xml
[scheme]host[uri][*]
```

* `scheme`：(選擇性) `https://` 或 `https://.`
* `host`：主機電腦的名稱或 IP 位址，以及連接埠號碼 (如有需要)。(請參閱 [https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.23](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.23))
* `uri`：(選擇性) 資源的路徑。

以下範例設定會處理 myCompany 的 `.com` 和 `.ch` 網域以及 mySubDivision 的所有網域的請求：

```xml
   /virtualhosts
    {
    "www.myCompany.com"
    "www.myCompany.ch"
    "www.mySubDivison.*"
    }
```

以下設定會處理&#x200B;*所有*&#x200B;請求：

```xml
   /virtualhosts
    {
    "*"
    }
```

### 解析虛擬主機 {#resolving-the-virtual-host}

當 Dispatcher 收到 HTTP 或 HTTPS 請求時，它會尋找最符合該請求的 `host,` `uri` 和 `scheme` 標題的虛擬主機值。Dispatcher 會依照以下順序評估 `virtualhosts` 屬性中的值：

* Dispatcher 會從最低的陣列開始，然後在 dispatcher.any 檔案中往上進行。
* 對於每個陣列，Dispatcher 都會從 `virtualhosts` 屬性中的最高值開始，然後沿著值清單往下進行。

Dispatcher 會依照以下順序尋找最符合的虛擬主機值：

* 將會使用遇到的第一個與請求的所有三個 `host`、`scheme` 和 `uri` 相符的虛擬主機。
* 如果沒有任何 `virtualhosts` 值具有 `scheme` 和 `uri` 部分，同時符合請求的 `scheme` 和 `uri`，則會使用遇到的第一個與請求的 `host` 相符的虛擬主機。
* 如果沒有任何 `virtualhosts` 值具有符合請求的主機的主機部分，則會使用最上層陣列的最上層虛擬主機。

因此，您應該將預設虛擬主機放在 `dispatcher.any` 檔案的最上層陣列中的 `virtualhosts` 屬性的最上方。

### 範例虛擬主機解析 {#example-virtual-host-resolution}

以下範例代表 `dispatcher.any` 檔案中的一個片段，該片段定義兩個 Dispatcher 陣列，每個陣列各定義一個 `virtualhosts` 屬性。

```xml
/farms
  {
  /myProducts
    {
    /virtualhosts
      {
      "www.mycompany.com/products/*"
      }
    /renders
      {
      /hostname "server1.myCompany.com"
      /port "80"
      }
    }
  /myCompany
    {
    /virtualhosts
      {
      "www.mycompany.com"
      }
    /renders
      {
      /hostname "server2.myCompany.com"
      /port "80"
      }
    }
  }
```

根據此範例，下表顯示為特定 HTTP 請求解析的虛擬主機：

| 請求 URL | 解析的虛擬主機 |
|---|---|
| `https://www.mycompany.com/products/gloves.html` | `www.mycompany.com/products/` |
| `https://www.mycompany.com/about.html` | `www.mycompany.com` |

## 啟用安全工作階段 - /sessionmanagement {#enabling-secure-sessions-sessionmanagement}

>[!CAUTION]
>
>`/allowAuthorized` 設定為 `"0"` (在 `/cache` 區段) 以啟用此功能。 如[使用驗證時快取](#caching-when-authentication-is-used)區段中詳細說明的，當您設定包含驗證資訊的 `/allowAuthorized 0 ` 請求時，**不會**&#x200B;快取。如需權限敏感型快取，請參閱[快取安全內容](https://experienceleague.adobe.com/docs/experience-manager-dispatcher/using/configuring/permissions-cache.html?lang=zh-Hant)頁面。

建立安全工作階段以存取轉譯器陣列，讓使用者必須登入才能存取陣列中的任何頁面。 使用者在登入後，就可以存取陣列中的頁面。如需搭配 CUG 使用此功能的相關資訊，請參閱[建立封閉式使用者群組](https://experienceleague.adobe.com/docs/experience-manager-65/administering/security/cug.html?lang=zh-Hant#creating-the-user-group-to-be-used)。此外，也請在上線前參考 Dispatcher [安全性檢查清單](/help/using/security-checklist.md)。

`/sessionmanagement` 屬性是 `/farms` 的子屬性。

>[!CAUTION]
>
>如果您網站的區段使用不同存取要求，您必須定義多個陣列。

**/sessionmanagement** 有幾個子參數：

**/directory** (必要)

儲存工作階段資訊的目錄。如果此目錄不存在，則會建立它。

>[!CAUTION]
>
> 在設定目錄子參數時，**請勿**&#x200B;指向根資料夾 (`/directory "/"`)，因為這樣可能會導致嚴重的問題。 務必指定儲存工作階段資訊的資料夾的路徑。 例如：

```xml
/sessionmanagement
  {
  /directory "/usr/local/apache/.sessions"
  }
```

**/encode** (選擇性)

工作階段資訊的編碼方式。使用 `md5` 可透過 md5 演算法進行加密，或是使用 `hex` 進行十六進位加密。如果您為工作階段資料加密，可存取檔案系統的使用者將無法讀取工作階段內容。預設為 `md5`。

**/header** (選擇性)

儲存授權資訊的 HTTP 標題或 Cookie 的名稱。如果您將此資訊儲存在 http 標題中，請使用 `HTTP:<header-name>`。若要將此資訊儲存在 Cookie 中，請使用 `Cookie:<header-name>`。如果您未指定值，則會使用 `HTTP:authorization`。

**/timeout** (選擇性)

工作階段在上次使用之後，進入逾時之前的秒數。如果未指定，則會使用 `"800"`，所以工作階段的逾時時間為使用者的最後一次請求後的 13 分鐘多一點。

範例設定如下所示：

```xml
/sessionmanagement
  {
  /directory "/usr/local/apache/.sessions"
  /encode "md5"
  /header "HTTP:authorization"
  /timeout "800"
  }
```

## 定義頁面轉譯器 {#defining-page-renderers-renders}

/renders 屬性會定義 URL，Dispatcher 會傳送請求到該 URL 以轉譯文件。以下範例 `/renders` 區段會識別用於轉譯的單一 AEM 執行個體：

```xml
/renders
  {
    /myRenderer
      {
      # hostname or IP of the renderer
      /hostname "aem.myCompany.com"
      # port of the renderer
      /port "4503"
      # connection timeout in milliseconds, "0" (default) waits indefinitely
      /timeout "0"
      }
  }
```

以下範例 /renders 區段會識別在相同電腦上當作 Dispatcher 執行的 AEM 執行個體：

```xml
/renders
  {
    /myRenderer
     {
     /hostname "127.0.0.1"
     /port "4503"
     }
  }
```

以下範例 /renders 區段會將轉譯請求平均分配到兩個 AEM 執行個體：

```xml
/renders
  {
    /myFirstRenderer
      {
      /hostname "aem.myCompany.com"
      /port "4503"
      }
    /mySecondRenderer
      {
      /hostname "127.0.0.1"
      /port "4503"
      }
  }
```

### 轉譯選項 {#renders-options}

**/timeout**

指定存取 AEM 執行個體的連線逾時值 (以毫秒為單位)。預設為 `"0"`，此值會讓 Dispatcher 無限期等待。

**/receiveTimeout**

指定允許接收回應的時間 (以毫秒為單位)。預設為 `"600000"`，此值會讓 Dispatcher 等待 10 分鐘。設定為 `"0"` 可免除逾時。

如果在剖析回應標頭時到達逾時時間，則會傳回 HTTP 狀態 504 (閘道錯誤)。如果在讀取回應本文時到達逾時時間，Dispatcher 會將不完整的回應傳回給用戶端。 它也會刪除任何可能已寫入的快取檔案。

**/ipv4**

指定 Dispatcher 是要使用 `getaddrinfo` 函數 (適用於 IPv6) 還是 `gethostbyname` 函數 (適用於 IPv4) 來取得轉譯器的 IP 位址。0 的值會使用 `getaddrinfo`。`1` 的值會使用 `gethostbyname`。預設值為 `0`。

`getaddrinfo` 函數會傳回 IP 位址清單。Dispatcher 會逐一查看此位址清單，直到建立 TCP/IP 連線為止。因此，當轉譯器主機名稱與多個 IP 位址相關聯，而主機在回應 `getaddrinfo` 函數時傳回始終具有相同順序的 IP 位址清單時，`ipv4` 屬性會很重要。在此情況下，您應該使用 `gethostbyname` 函數，好讓 Dispatcher 連線的 IP 位址是隨機的。

Amazon Elastic Load Balancing (ELB) 服務可使用可能具有相同順序的 IP 位址清單來回應 getaddrinfo。

**/secure**

如果 `/secure` 屬性的值為 `"1"`，則 Dispatcher 會使用 HTTPS 來與 AEM 執行個體通訊。 如需其他詳細資訊，請參閱[設定 Dispatcher 使用 SSL](dispatcher-ssl.md#configuring-dispatcher-to-use-ssl)。

**/always-resolve**

使用 Dispatcher 版本 **4.1.6** 時，您可以依照以下方式設定 `/always-resolve` 屬性：

* 設為 `"1"` 時，它會解析每個請求的主機名稱 (Dispatcher 絕對不會快取任何 IP 位址)。 由於獲得每個請求的主機資訊需要額外的呼叫，所以可能會對效能造成輕微的影響。
* 如果未設定此屬性，預設會快取 IP 位址。

此外，如果您遇到動態 IP 解析問題，也可以使用此屬性，如以下範例所示：

```xml
/renders {
  /0001 {
     /hostname "host-name-here"
     /port "4502"
     /ipv4 "1"
     /always-resolve "1"
     }
  }
```

## 設定對內容的存取權 {#configuring-access-to-content-filter}

使用 `/filter` 區段可指定 Dispatcher 接受的 HTTP 請求。所有其他請求都會傳回網頁伺服器，並傳回 404 錯誤碼 (找不到頁面)。如果 `/filter` 區段不存在，則會接受所有請求。

**注意：**&#x200B;對 [statfile](#naming-the-statfile) 的請求一律會遭到拒絕。

>[!CAUTION]
>
>請參閱 [Dispatcher 安全性檢查清單](security-checklist.md)，了解使用 Dispatcher 限制存取時的更多考量事項。此外，也請閱讀 [AEM 安全性檢查清單](https://experienceleague.adobe.com/docs/experience-manager-65/administering/security/security-checklist.html?lang=zh-Hant#security)，了解有關 AEM 安裝的更多安全性細節。

`/filter` 區段是由一系列規則所組成，這些規則會根據 HTTP 請求的請求行部分中的模式來拒絕或允許對內容的存取。 針對 `/filter` 區段使用允許清單策略：

* 首先，拒絕存取所有內容。
* 視需要允許存取內容。

>[!NOTE]
>
>在篩選規則發生任何變化時清除快取。

### 定義篩選條件 {#defining-a-filter}

`/filter` 區段中的每個項目都包含與請求行的特定元素或完整請求行相符的類型和模式。每個篩選條件都可包含以下項目：

* **類型**：`/type` 會指示允許還是拒絕存取符合此模式的請求。該值可以是 `allow` 或 `deny`。

* **請求行的元素：**&#x200B;包含 `/method`、`/url`、`/query` 或 `/protocol`，以及根據這些特定部分或 HTTP 請求的請求行部分來篩選請求的模式。依請求行的元素 (而不是整個請求行) 進行篩選是首選的篩選方法。

* **請求行的進階元素：**&#x200B;從 Dispatcher 4.2.0 開始，有四個新的篩選元素可供使用。這些新元素分別為 `/path`、`/selectors`、`/extension` 和 `/suffix`。 包含這些項目中的一個或多個可進一步控制 URL 模式。

>[!NOTE]
>
>如需有關這些元素各自參照請求行的哪個部分的詳細資訊，請參閱 [Sling URL 分解](https://sling.apache.org/documentation/the-sling-engine/url-decomposition.html) wiki 頁面。

* **glob 屬性**：`/glob` 屬性是用來比對 HTTP 請求的整個請求行。

>[!CAUTION]
>
>在 Dispatcher 中使用 glob 進行篩選的功能已過時。因此，您應該避免在 `/filter` 區段中使用 glob，因為它可能會導致安全性問題。所以，請不要：
>
>`/glob "* *.css *"`
>
>使用
>
>`/url "*.css"`

#### HTTP 請求的請求行部分 {#the-request-line-part-of-http-requests}

HTTP/1.1 會依據以下方式定義[請求行](https://www.w3.org/Protocols/rfc2616/rfc2616-sec5.html)：

`Method Request-URI HTTP-Version<CRLF>`

`<CRLF>` 字元代表歸位字元及後面跟著的換行字元。以下範例是當用戶端請求 WKND 網站的英文版頁面時所收到的請求行：

`GET /content/wknd/us/en.html HTTP.1.1<CRLF>`

您的模式必須考量請求行中的空格字元和 `<CRLF>` 字元。

#### 雙引號與單引號的比較 {#double-quotes-vs-single-quotes}

在建立篩選規則時，請針對簡單模式使用雙引號 `"pattern"`。如果您使用 Dispatcher 4.2.0 或更新版本，而且您的模式包含規則運算式，則必須用單引號括住規則運算式模式 `'(pattern1|pattern2)'`。

#### 規則運算式 {#regular-expressions}

在 Dispatcher 4.2.0 之後的版本中，您可以在篩選模式中包含 POSIX 擴充型規則運算式。

#### 篩選的疑難排解 {#troubleshooting-filters}

如果篩選器並未如您預期的方式觸發，請對 Dispatcher 啟用[追蹤記錄](#trace-logging)，以便可以查看哪個篩選條件正在攔截請求。

#### 範例篩選條件：全部拒絕 {#example-filter-deny-all}

以下範例篩選區段會讓 Dispatcher 拒絕所有檔案的請求。拒絕存取所有檔案，然後允許存取特定區域。

```xml
/0001  { /type "deny" /url "*"  }
```

針對被明確拒絕的區域的請求會傳回 404 錯誤碼 (找不到頁面)。

#### 範例篩選條件：拒絕存取特定區域 {#example-filter-deny-access-to-specific-areas}

篩選器也可讓您拒絕存取各種元素 (例如 ASP 頁面) 以及發佈執行個體中的敏感區域。 以下篩選條件會拒絕存取 ASP 頁面：

```xml
/0002  { /type "deny" /url "*.asp"  }
```

#### 範例篩選條件：啟用 POST 請求 {#example-filter-enable-post-requests}

以下範例篩選條件可透過 POST 方法提交表單資料：

```xml
/filter {
    /0001  { /glob "*" /type "deny" }
    /0002 { /type "allow" /method "POST" /url "/content/[.]*.form.html" }
}
```

#### 範例篩選條件：允許存取 Workflow Console {#example-filter-allow-access-to-the-workflow-console}

以下範例顯示用來允許外部使用者存取 Workflow console 的篩選條件：

```xml
/filter {
    /0001  { /glob "*" /type "deny" }
    /0002  {  /type "allow"  /url "/libs/cq/workflow/content/console*"  }
}
```

如果您的發佈執行個體使用網站應用程式環境 (例如發佈)，您也可以將其新增到您的篩選定義中。

```xml
/0003   { /type "deny"  /url "/publish/libs/cq/workflow/content/console/archive*"  }
```

如果您必須存取受限區域中的單次頁面，您可以允許存取這些頁面。 例如，若要允許存取 Workflow console 內的 Archive 索引標籤，請新增以下區段：

```xml
/0004  { /type "allow"  /url "/libs/cq/workflow/content/console/archive*"   }
```

>[!NOTE]
>
>將多個篩選模式套用到一個請求時，最後一個套用的篩選模式會生效。

#### 範例篩選條件：使用規則運算式 {#example-filter-using-regular-expressions}

此篩選條件使用規則運算式，在非公開內容目錄中啟用副檔名 (在這裡是定義在單引號之間)：

```xml
/005  {  /type "allow" /extension '(css|gif|ico|js|png|swf|jpe?g)' }
```

#### 範例篩選條件：篩選請求 URL 的其他元素 {#example-filter-filter-additional-elements-of-a-request-url}

底下規則範例使用路徑、選擇器和副檔名的篩選條件來封鎖擷取自 `/content` 路徑及其子樹狀結構的內容：

```xml
/006 {
        /type "deny"
        /path "/content/*"
        /selectors '(feed|rss|pages|languages|blueprint|infinity|tidy|sysview|docview|query|jcr:content|_jcr_content|search|childrenlist|ext|assets|assetsearch|[0-9-]+)'
        /extension '(json|xml|html|feed))'
        }
```

### 範例 /filter 區段 {#example-filter-section}

當您設定 Dispatcher 時，應該盡可能限制外部存取。 以下範例為外部訪客提供最低限度的存取權：

* `/content`
* 像是設計和用戶端資料庫等其他內容。例如：

   * `/etc/designs/default*`
   * `/etc/designs/mydesign*`

在建立篩選條件後，請[測試頁面存取權](#testing-dispatcher-security)以確保您的 AEM 執行個體安全無虞。

`dispatcher.any` 檔案的以下 `/filter` 區段可當作您的 [Dispatcher 設定檔案](#dispatcher-configuration-files)的基礎。

以下範例是根據 Dispatcher 所提供的預設設定檔案，可當作生產環境中的範例使用。前置詞為 `#` 的項目已停用 (標記為註解)。 如果您決定啟用其中任何項目 (藉由移除該行上的 `#`)，則應該要謹慎。 這麼做可能會影響安全性。

拒絕存取所有內容，然後允許存取特定 (受限) 元素：

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (ims-author-00AF43764F54BE740A490D44@AdobeID)
Last Modified Date: 2015-06-26T04:32:37.986-0400

<p>We should mention the config files that are shipped with the dispatcher distribution and only give a few examples here. This aims to avoid confusion and reduce content maintenance.<br /> </p>

 -->

```xml
  /filter
      {
      # Deny everything first and then allow specific entries
      /0001  { /type "deny" /url "*"  }

      # Open consoles
#     /0011 { /type "allow" /url "/admin/*"  }  # allow servlet engine admin
#     /0012 { /type "allow" /url "/crx/*"    }  # allow content repository
#     /0013 { /type "allow" /url "/system/*" }  # allow OSGi console

      # Allow non-public content directories
#     /0021 { /type "allow" /url "/apps/*"   }  # allow apps access
#     /0022 { /type "allow" /url "/bin/*"    }
      /0023 { /type "allow" /url "/content*" }  # disable this rule to allow mapped content only

#     /0024 { /type "allow" /url "/libs/*"   }
#     /0025 { /type "deny"  /url "/libs/shindig/proxy*" } # if you enable /libs close access to proxy

#     /0026 { /type "allow" /url "/home/*"   }
#     /0027 { /type "allow" /url "/tmp/*"    }
#     /0028 { /type "allow" /url "/var/*"    }

      # Enable extensions in non-public content directories, using a regular expression
      /0041
        {
        /type "allow"
        /extension '(css|gif|ico|js|png|swf|jpe?g)'
        }

      # Enable features
      /0062 { /type "allow" /url "/libs/cq/personalization/*"  }  # enable personalization

      # Deny content grabbing, on all accessible pages, using regular expressions
      /0081
        {
        /type "deny"
        /selectors '((sys|doc)view|query|[0-9-]+)'
        /extension '(json|xml)'
        }
      # Deny content grabbing for /content and its subtree
      /0082
        {
        /type "deny"
        /path "/content/*"
        /selectors '(feed|rss|pages|languages|blueprint|infinity|tidy)'
        /extension '(json|xml|html)'
        }

#     /0087 { /type "allow" /method "GET" /extension 'json' "*.1.json" }  # allow one-level json requests
}
```

>[!NOTE]
>
>在搭配 Apache 使用時，請根據 Dispatcher 模組的 DispatcherUseProcessedURL 屬性來設計您的篩選條件 URL 模式。(請參閱 [Apache Web Server - 為 Dispatcher 設定 Apache Web Server](dispatcher-install.md##apache-web-server-configure-apache-web-server-for-dispatcher)。)

<!----
>[!NOTE]
>
>Filters `0030` and `0031` regarding Dynamic Media are applicable to AEM 6.0 and higher. -->

如果您選擇擴充存取權，請考量以下建議事項：

* 如果您使用 CQ 版本 5.4 或較舊的版本，請停用 `/admin` 的外部存取權。

* 在允許存取 `/libs` 中的檔案時，必須要謹慎。應根據個人情況來允許存取。
* 拒絕存取複寫設定，使其無法被人看見：

   * `/etc/replication.xml*`
   * `/etc/replication.infinity.json*`

* 拒絕存取 Google Gadgets 反向 Proxy：

   * `/libs/opensocial/proxy*`

根據您的安裝情況，在 `/libs`、`/apps` 底下或其他地方可能有其他必須提供的資源。您可以使用 `access.log` 檔案當作判斷是否有人在外部存取資源的方式之一。

>[!CAUTION]
>
>存取主控台和目錄可能會給生產環境帶來安全性風險。除非您有明確的理由，否則應該將其保持停用狀態 (標記為註解)。

>[!CAUTION]
>
>如果您[正在發佈環境中使用報表](https://experienceleague.adobe.com/docs/experience-manager-65/administering/operations/reporting.html?lang=zh-Hant#using-reports-in-a-publish-environment)，您應該設定 Dispatcher 以拒絕外部訪客存取 `/etc/reports`。

### 限制查詢字串 {#restricting-query-strings}

從 Dispatcher 版本 4.1.5 開始，請使用 `/filter` 區段來限制查詢字串。極力建議您明確允許查詢字串，並透過 `allow` 篩選元素來排除一般性允許。

單一項目可以是 `glob` 或是 `method`、`url`、`query` 和 `version` 的某個組合，但兩者不能同時存在。以下範例針對解析成 `/etc` 節點的 URL，允許 `a=*` 查詢字串但拒絕所有其他查詢字串：

```xml
/filter {
 /0001 { /type "deny" /method "POST" /url "/etc/*" }
 /0002 { /type "allow" /method "GET" /url "/etc/*" /query "a=*" }
}
```

>[!NOTE]
>
>如果規則包含 `/query`，它只會比對包含查詢字串的請求，並比對提供的查詢模式。
>
>在上述範例中，如果也應該允許對 `/etc` 的沒有查詢字串的請求，則需要以下規則：
>

```xml
/filter {  
>/0001 { /type "deny" /method "*" /url "/path/*" }  
>/0002 { /type "allow" /method "GET" /url "/path/*" }  
>/0003 { /type "deny" /method "GET" /url "/path/*" /query "*" }  
>/0004 { /type "allow" /method "GET" /url "/path/*" /query "a=*" }  
}  
```

### 測試 Dispatcher 安全性 {#testing-dispatcher-security}

Dispatcher 篩選條件應該在 AEM Publish 執行個體上封鎖對以下頁面和指令碼的存取。使用網頁瀏覽器，嘗試以網站訪客的身分開啟以下頁面，並驗證是否傳回錯誤碼 404。如果獲得其他任何結果，請調整篩選條件。

您應該會看到 `/content/add_valid_page.html?debug=layout` 的正常頁面呈現。

* `/admin`
* `/system/console`
* `/dav/crx.default`
* `/crx`
* `/bin/crxde/logs`
* `/jcr:system/jcr:versionStorage.json`
* `/_jcr_system/_jcr_versionStorage.json`
* `/libs/wcm/core/content/siteadmin.html`
* `/libs/collab/core/content/admin.html`
* `/libs/cq/ui/content/dumplibs.html`
* `/var/linkchecker.html`
* `/etc/linkchecker.html`
* `/home/users/a/admin/profile.json`
* `/home/users/a/admin/profile.xml`
* `/libs/cq/core/content/login.json`
* `/content/../libs/foundation/components/text/text.jsp`
* `/content/.{.}/libs/foundation/components/text/text.jsp`
* `/apps/sling/config/org.apache.felix.webconsole.internal.servlet.OsgiManager.config/jcr%3acontent/jcr%3adata`
* `/libs/foundation/components/primary/cq/workflow/components/participants/json.GET.servlet`
* `/content.pages.json`
* `/content.languages.json`
* `/content.blueprint.json`
* `/content.-1.json`
* `/content.10.json`
* `/content.infinity.json`
* `/content.tidy.json`
* `/content.tidy.-1.blubber.json`
* `/content/dam.tidy.-100.json`
* `/content/content/geometrixx.sitemap.txt `
* `/content/add_valid_page.query.json?statement=//*`
* `/content/add_valid_page.qu%65ry.js%6Fn?statement=//*`
* `/content/add_valid_page.query.json?statement=//*[@transportPassword]/(@transportPassword%20|%20@transportUri%20|%20@transportUser)`
* `/content/add_valid_path_to_a_page/_jcr_content.json`
* `/content/add_valid_path_to_a_page/jcr:content.json`
* `/content/add_valid_path_to_a_page/_jcr_content.feed`
* `/content/add_valid_path_to_a_page/jcr:content.feed`
* `/content/add_valid_path_to_a_page/pagename._jcr_content.feed`
* `/content/add_valid_path_to_a_page/pagename.jcr:content.feed`
* `/content/add_valid_path_to_a_page/pagename.docview.xml`
* `/content/add_valid_path_to_a_page/pagename.docview.json`
* `/content/add_valid_path_to_a_page/pagename.sysview.xml`
* `/etc.xml`
* `/content.feed.xml`
* `/content.rss.xml`
* `/content.feed.html`
* `/content/add_valid_page.html?debug=layout`
* `/projects`
* `/tagging`
* `/etc/replication.html`
* `/etc/cloudservices.html`
* `/welcome`

若要確定是否已啟用匿名寫入權限，請在終端機或命令提示字元中發出以下命令。 您應該無法將資料寫入節點。

`curl -X POST "https://anonymous:anonymous@hostname:port/content/usergenerated/mytestnode"`

若要嘗試讓 Dispatcher 快取失效，並確定您收到了代碼 403 回應，請在終端機或命令提示字元中發出以下命令：

`curl -H "CQ-Handle: /content" -H "CQ-Path: /content" https://yourhostname/dispatcher/invalidate.cache`

## 啟用對虛名 URL 的存取權 {#enabling-access-to-vanity-urls-vanity-urls}

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (sbroders@adobe.com)
Last Modified Date: 2015-03-25T14:23:05.185-0400

<p style="font-family: tahoma, arial, helvetica, sans-serif; font-size: 12px;">For https://jira.corp.adobe.com/browse/DOC-4812</p> 
<p style="font-family: tahoma, arial, helvetica, sans-serif; font-size: 12px;">The "com.adobe.granite.dispatcher.vanityurl.content" package needs to be made public before publishing this contnet.</p>
 -->

設定 Dispatcher 以啟用對虛名 URL 的存取權，這些是為您的 AEM 頁面設定的虛名 URL。

在啟用對虛名 URL 的存取權時，Dispatcher 會定期呼叫在轉譯器執行個體上執行的服務來取得虛名 URL 清單。Dispatcher 會將此清單儲存在本機檔案中。當由於 `/filter` 區段中的篩選條件而拒絕頁面的請求時，Dispatcher 會查詢虛名 URL 清單。如果被拒絕的 URL 在此清單上，Dispatcher 會允許存取虛名 URL。

若要啟用對虛名 URL 的存取權，請在 `/farms` 區段中新增 `/vanity_urls` 區段，類似於以下範例：

```xml
 /vanity_urls {
      /url "/libs/granite/dispatcher/content/vanityUrls.html"
      /file "/tmp/vanity_urls"
      /delay 300
 }
```

`/vanity_urls` 區段包含以下屬性：

* `/url`：在轉譯器執行個體上執行的虛名 URL 服務的路徑。這個屬性的值必須是 `"/libs/granite/dispatcher/content/vanityUrls.html"`。

* `/file`：Dispatcher 用來儲存虛名 URL 清單的本機檔案的路徑。請確定 Dispatcher 具有此檔案的寫入權限。
* `/delay`：(秒) 對虛名 URL 服務的兩次呼叫之間的時間。

>[!NOTE]
>
>如果您的轉譯器是 AEM 的執行個體，您必須[從軟體散發安裝 VanityURLS-Components 套件](https://experience.adobe.com/#/downloads/content/software-distribution/en/aem.html?package=/content/software-distribution/en/details.html/content/dam/aem/public/adobe/packages/granite/vanityurls-components)以啟用虛名 URL 服務。 (如需了解詳情，請參閱[軟體散發](https://experienceleague.adobe.com/docs/experience-manager-65/administering/contentmanagement/package-manager.html?lang=zh-Hant#software-distribution)。)

使用以下程序來啟用對虛名 URL 的存取權。

1. 如果您的轉譯服務為 AEM 執行個體，請在發佈執行個體上安裝 `com.adobe.granite.dispatcher.vanityurl.content` 套件 (請參閱上面的註解)。
1. 對於您為 AEM 或 CQ 頁面設定的每個虛名 URL，請確定 [`/filter`](#configuring-access-to-content-filter) 設定會拒絕該 URL。必要時，請新增拒絕該 URL 的篩選條件。
1. 在 `/farms` 底下新增 `/vanity_urls` 區段。
1. 重新啟動 Apache Web Server。

## 轉送整合請求 - /propagateSyndPost {#forwarding-syndication-requests-propagatesyndpost}

整合請求僅適用於 Dispatcher，所以在預設情況下不會傳送給轉譯器 (例如 AEM 執行個體)。

必要時，請將 `/propagateSyndPost` 屬性設定為 `"1"` 以將整合請求轉送給 Dispatcher。如果有設定此屬性，您必須確定篩選區段中不會拒絕 POST 請求。

## 設定 Dispatcher 快取 - /cache {#configuring-the-dispatcher-cache-cache}

`/cache` 區段會控制 Dispatcher 如何快取文件。設定幾個子屬性以實施您的快取策略：

* `/docroot`
* `/statfile`
* `/serveStaleOnError`
* `/allowAuthorized`
* `/rules`
* `/statfileslevel`
* `/invalidate`
* `/invalidateHandler`
* `/allowedClients`
* `/ignoreUrlParams`
* `/headers`
* `/mode`
* `/gracePeriod`
* `/enableTTL`

快取區段範例可能如下所示：

```xml
/cache
  {
  /docroot "/opt/dispatcher/cache"
  /statfile  "/tmp/dispatcher-website.stat"
  /allowAuthorized "0"

  /rules
    {
    # List of files that are cached
    }

  /invalidate
    {
    # List of files that are auto-invalidated
    }
  }
  
```

>[!NOTE]
>
>如需了解權限敏感型快取，請閱讀[快取安全內容](permissions-cache.md)。

### 指定快取目錄 {#specifying-the-cache-directory}

`/docroot` 屬性可識別快取檔案儲存所在的目錄。

>[!NOTE]
>
>該值必須與網頁伺服器的主目錄的路徑完全相同，以便 Dispatcher 和網頁伺服器可處理相同檔案。\
>網頁伺服器負責在使用 Dispatcher 快取檔案時傳遞正確的狀態代碼，這就是為什麼能夠找到該檔案非常重要的原因。

如果您使用多個陣列，每個陣列都必須使用不同的主目錄。

### 為 Statfile 命名 {#naming-the-statfile}

`/statfile` 屬性可識別要當作 statfile 使用的檔案。Dispatcher 會使用這個檔案來登錄最近一次內容更新的時間。statfile 可以是網頁伺服器上的任何檔案。

statfile 沒有任何內容。當更新內容時，Dispatcher 會更新時間戳記。預設 statfile 命名為 `.stat`，並儲存在 docroot 中。Dispatcher 會封鎖對 statfile 的存取權。

>[!NOTE]
>
>如果設定了 `/statfileslevel`，Dispatcher 會忽略 `/statfile` 屬性並使用 `.stat` 當作名稱。

### 在發生錯誤時提供過時文件 {#serving-stale-documents-when-errors-occur}

`/serveStaleOnError` 屬性可控制當轉譯伺服器傳回錯誤時，Dispatcher 是否會傳回失效的文件。根據預設，在接觸了 statfile 並讓快取內容失效時，Dispatcher 在下次請求該內容時會刪除快取的內容。

如果 `/serveStaleOnError` 設定為 `"1"`，則 Dispatcher 不會從快取中刪除失效的內容，除非轉譯伺服器傳回成功的回應。來自 AEM 的 5xx 回應或連線逾時會導致 Dispatcher 提供過時的內容及 HTTP 狀態 111 (重新驗證失敗) 的回應。

### 在使用驗證時快取 {#caching-when-authentication-is-used}

`/allowAuthorized` 屬性可控制是否快取包含以下任何驗證資訊的請求：

* `authorization` 標題
* 名為 `authorization` 的 Cookie
* 名為 `login-token` 的 Cookie

預設不會快取包含此驗證資訊的請求，因為將快取的文件傳回用戶端時並不會執行驗證。此設定可避免 Dispatcher 提供快取的文件給沒有必要權限的使用者。

不過，如果您的要求允許快取經過驗證的文件，請將 `/allowAuthorized` 設為一：

`/allowAuthorized "1"`

>[!NOTE]
>
>若要啟用工作階段管理 (使用 `/sessionmanagement` 屬性)，`/allowAuthorized` 屬性必須設為 `"0"`。

### 指定要快取的文件 {#specifying-the-documents-to-cache}

`/rules` 屬性可控制根據文件路徑快取哪些文件。無論 `/rules` 屬性為何，Dispatcher 在以下情況下絕對不會快取文件：

* 請求 URI 包含問號 (`?`)。
   * 表示這是不需要快取的動態頁面，例如搜尋結果。
* 缺少副檔名。
   * 網頁伺服器需要副檔名來判斷文件類型 (MIME 類型)。
* 驗證標頭已設定 (可設定)。
* 如果 AEM 執行個體提供以下標題當作回應：

   * `no-cache`
   * `no-store`
   * `must-revalidate`

>[!NOTE]
>
>Dispatcher 可快取 GET 或 HEAD (用於 HTTP 標題) 方法。如需有關回應標頭快取的其他資訊，請參閱[快取 HTTP 回應標頭](#caching-http-response-headers)一節。

`/rules` 屬性中的每個項目都包含 [`glob`](#designing-patterns-for-glob-properties) 模式和一個類型：

* `glob` 模式是用來比對文件的路徑。
* 類型可指示是否快取符合 `glob` 模式的文件。該值可以是 `allow` (可快取文件) 或 `deny` (一律轉譯文件)。

如果您沒有動態頁面 (在上述規則已排除的頁面之外)，您可以設定 Dispatcher 快取所有內容。 規則區段如下所示：

```xml
/rules
  {
    /0000  {  /glob "*"   /type "allow" }
  }
```

如需 glob 屬性的相關資訊，請參閱[為 glob 屬性設計模式](#designing-patterns-for-glob-properties)。

如果頁面中的某些區段是動態的 (例如新聞應用程式) 或是在封閉式使用者群組內，您可以定義例外情況：

>[!NOTE]
>
>請勿快取封閉式使用者群組，因為系統不會檢查快取頁面的使用者權限。

```xml
/rules
  {
   /0000  { /glob "*" /type "allow" }
   /0001  { /glob "/en/news/*" /type "deny" }
   /0002  { /glob "*/private/*" /type "deny"  }
  }
```

**壓縮**

在 Apache Web Server 上，您可以壓縮快取文件。 如果用戶端要求的話，壓縮可讓 Apache 傳回壓縮格式的文件。啟用 Apache 模組 `mod_deflate` 即可自動進行壓縮，例如：

```xml
AddOutputFilterByType DEFLATE text/plain
```

Apache 2.x 預設會安裝此模組。

<!-- 

Comment Type: draft

<note type="note"> 
 <p>Depending on the Apache web server version you can enable <span class="code">gzip</span> compression as follows:</p> 
 <ul> 
  <li>For Apache 1.3 you can enable the <span class="code">mod_gzip </span>module. The module must be downloaded and installed.</li> 
  <li>For Apache 2.x you can enable the <span class="code">mod_deflate</span> module. The module is installed by default with Apache 2.x.<br /> </li> 
 </ul> 
 <p> </p> 
</note>

 -->

<!-- 

Comment Type: draft

<p>The following rule caches all documents in compressed form; Apache can return either the uncompressed or the compressed form to the client:</p>

 -->

<!-- 

Comment Type: draft

<codeblock gutter="true" class="syntax xml">
  /rules!!discoiqbr!!&nbsp;&nbsp;{!!discoiqbr!!&nbsp;&nbsp;&nbsp;/rulelabel&nbsp;&nbsp;{&nbsp;&nbsp;/glob&nbsp;"*"&nbsp;/type&nbsp;"allow"&nbsp;&nbsp;/compress&nbsp;"gzip"&nbsp;}!!discoiqbr!!&nbsp;&nbsp;} 
</codeblock>

 -->

<!-- 

Comment Type: remark
Last Modified By: Silviu Raiman (raiman)
Last Modified Date: 2017-11-13T09:23:24.326-0500

<p>Hidden the <span class="code">mod_gzip</span> content as requested in CQDOC-11124.</p>

 -->

### 依照資料夾層級讓檔案失效 {#invalidating-files-by-folder-level}

使用 `/statfileslevel` 屬性可根據快取檔案的路徑來讓這些檔案失效：

* Dispatcher 在從 docroot 資料夾到您指定的層級的每個資料夾中建立 `.stat` 檔案。docroot 資料夾為第 0 層。
* 接觸 `.stat` 檔案即可讓檔案失效。`.stat` 檔案的最後修改日期會與快取文件的最後修改日期做比較。如果 `.stat` 檔案較新，則會重新提取該文件。

* 當某個層級的檔案失效時，會觸及&#x200B;**所有** `.stat` 檔案，從 docroot **到**&#x200B;失效檔案的層級或設定的 `statsfilevel` (以較小者為準)。

   * 例如：如果您將 `statfileslevel` 屬性設為 6，而在第 5 層的某個檔案失效，則會觸及從 docroot 到第 5 層的每個 `.stat` 檔案。 繼續這個範例，如果位在第 7 層的某個檔案失效，則會觸及從 docroot 到第 6 層的每個 `stat` 檔案 (從 `/statfileslevel = "6"` 開始)。

只有失效檔案的&#x200B;**路徑上**&#x200B;的資源會受到影響。考量以下範例：網站使用此結構：`/content/myWebsite/xx/.` 如果您將 `statfileslevel` 設為 3，則會建立 `.stat` 檔案，如下所示：

* `docroot`
* `/content`
* `/content/myWebsite`
* `/content/myWebsite/*xx*`

當 `/content/myWebsite/xx` 中的某個檔案失效時，會觸及每個 `.stat` 檔案，從 docroot 往下到 `/content/myWebsite/xx`。 只有 `/content/myWebsite/xx` 才會是這種情況，而不是 `/content/myWebsite/yy` 或 `/content/anotherWebSite` 等。

>[!NOTE]
>
>可藉由傳送另一個標頭 `CQ-Action-Scope:ResourceOnly` 來避免失效。 此方法可用來清除特定資源，而不會讓快取的其他部分失效。 如需其他詳細資訊，請參閱[此頁面](https://adobe-consulting-services.github.io/acs-aem-commons/features/dispatcher-flush-rules/index.html)和[手動讓 Dispatcher 快取失效](https://experienceleague.adobe.com/docs/experience-manager-dispatcher/using/configuring/page-invalidate.html?lang=zh-Hant#configuring)。

>[!NOTE]
>
>如果您為 `/statfileslevel` 屬性指定了值，則會忽略 `/statfile` 屬性。

### 自動讓快取檔案失效 {#automatically-invalidating-cached-files}

`/invalidate` 屬性會定義在內容更新時自動失效的文件。

使用自動失效時，Dispatcher 並不會在內容更新後刪除快取檔案，而是在下次請求快取檔案時檢查其是否有效。快取中未自動失效的文件會留在快取中，直到內容更新明確將其刪除為止。

自動失效通常用於 HTML 頁面。HTML 頁面經常包含其他頁面的連結，所以很難判斷內容更新是否會影響頁面。為了確保在內容更新時所有相關頁面都會失效，請讓所有 HTML 頁面自動失效。以下設定會讓所有 HTML 頁面失效：

```xml
  /invalidate
  {
   /0000  { /glob "*" /type "deny" }
   /0001  { /glob "*.html" /type "allow" }
  }
```

如需 glob 屬性的相關資訊，請參閱[為 glob 屬性設計模式](#designing-patterns-for-glob-properties)。

在啟用 `/content/wknd/us/en` 時，此設定會產生以下活動：

* 所有帶有 en.* 模式的檔案都會從 `/content/wknd/us` 資料夾中移除。
* `/content/wknd/us/en./_jcr_content` 資料夾會被移除。
* 所有其他符合 `/invalidate` 設定的檔案都不會被立刻刪除。當下次請求發生時，就會刪除這些檔案。在範例中，`/content/wknd.html` 不會被刪除，但是在請求 `/content/wknd.html` 時會將其刪除。

如果您提供自動產生的 PDF 和 ZIP 檔案供人下載，您也可能必須讓這些檔案自動失效。 設定範例如下所示：

```xml
/invalidate
  {
   /0000 { /glob "*" /type "deny" }
   /0001 { /glob "*.html" /type "allow" }
   /0002 { /glob "*.zip" /type "allow" }
   /0003 { /glob "*.pdf" /type "allow" }
  }
```

AEM 與 Adobe Analytics 的整合可在您網站的`analytics.sitecatalyst.js` 檔案中提供設定資料。Dispatcher 提供的範例 `dispatcher.any` 檔案包含此檔案適用的以下失效規則：

```xml
{
   /glob "*/analytics.sitecatalyst.js"  /type "allow"
}
```

### 使用自訂失效指令碼 {#using-custom-invalidation-scripts}

`/invalidateHandler` 屬性可讓您定義指令碼，Dispatcher 收到的每個失效請求都會呼叫此指令碼。

呼叫此指令碼時會使用以下引數：

* 控制代碼 - 失效的內容路徑
* 動作 - 複寫動作 (例如啟用、停用)
* 行動範圍 - 複寫行動的範圍 (除非傳送了 `CQ-Action-Scope: ResourceOnly` 的標題，否則為空白，詳情請參閱[使 AEM 中的快取頁面失效](page-invalidate.md))

此方法可用於涵蓋多個不同的使用案例。 例如，讓其他應用程式特有的快取失效，或是處理頁面的外部化 URL 及其在 docroot 中的位置與內容路徑不符的情況。

底下範例指令碼會將每個失效請求記錄到檔案中。

```xml
/invalidateHandler "/opt/dispatcher/scripts/invalidate.sh"
```

#### 失效處理常式指令碼範例 {#sample-invalidation-handler-script}

```shell
#!/bin/bash

printf "%-15s: %s %s" $1 $2 $3>> /opt/dispatcher/logs/invalidate.log
```

### 限制可以清除快取的用戶端 {#limiting-the-clients-that-can-flush-the-cache}

`/allowedClients` 屬性會定義允許清除快取的特定用戶端。系統會將萬用字元模式與 IP 進行比對。

下列範例：

1. 拒絕存取任何用戶端
1. 明確地允許存取 localhost

```xml
/allowedClients
  {
   /0001 { /glob "*.*.*.*"  /type "deny" }
   /0002 { /glob "127.0.0.1" /type "allow" }
  }
```

如需 glob 屬性的相關資訊，請參閱[為 glob 屬性設計模式](#designing-patterns-for-glob-properties)。

>[!CAUTION]
>
>建議您最好定義 `/allowedClients`。
>
>如果沒有完成，任何用戶端都可以發出清除快取的呼叫。 如果重複執行，可能會嚴重影響網站績效。

### 忽略 URL 參數 {#ignoring-url-parameters}

`ignoreUrlParams` 區段會定義在判斷要快取頁面還是從快取中傳遞頁面時要忽略哪些 URL 參數：

* 當請求 URL 包含所有被忽略的參數時，將會快取頁面。
* 當請求 URL 包含一個或多個未被忽略的參數時，將不會快取頁面。

忽略頁面的某個參數時，將會在第一次請求該頁面時快取該頁面。無論請求中的參數值為何，都將為該頁面的後續請求提供快取頁面。

>[!NOTE]
>
>建議您以白名單方式配置 `ignoreUrlParams` 設定。這樣一來，所有查詢參數都將被忽略，並且只有已知或預期的查詢參數可以免除 (「已拒絕」) 被忽略。如需更多資訊和範例，請參閱[此頁面](https://github.com/adobe/aem-dispatcher-optimizer-tool/blob/main/docs/Rules.md#dot---the-dispatcher-publish-farm-cache-should-have-its-ignoreurlparams-rules-configured-in-an-allow-list-manner)。

若要指定哪些參數會被忽略，請在 `ignoreUrlParams` 屬性中新增 glob 規則：

* 雖然請求包含 URL 參數，但若要快取頁面，請建立一個允許該參數 (被忽略) 的 glob 屬性。
* 若要防止頁面快取，請建立一個拒絕參數的 glob 屬性 (將被忽略)。

>[!NOTE]
>
>設定 glob 屬性時，應該配合查詢參數名稱來設定。 例如，如果您想忽略來自以下 URL `http://example.com/path/test.html?p1=test&p2=v2` 的 &quot;p1&quot; 參數，則 glob 屬性應為：
> `/0002 { /glob "p1" /type "allow" }`

下列範例會造成 Dispatcher 忽略所有參數，`nocache` 參數除外。如此一來，Dispatcher 永遠不會快取包含 `nocache` 參數的要求 URL：

```xml
/ignoreUrlParams
{
    # ignore-all-url-parameters-by-dispatcher-and-requests-are-cached
    /0001 { /glob "*" /type "allow" }
    # allow-the-url-parameter-nocache-to-bypass-dispatcher-on-every-request
    /0002 { /glob "nocache" /type "deny" }
}
```

在上述 `ignoreUrlParams` 配置範例的上下文中，以下 HTTP 要求會導致頁面被快取，因為 `willbecached` 參數被忽略：

```xml
GET /mypage.html?willbecached=true
```

在上述 `ignoreUrlParams` 配置範例的上下文中，以下 HTTP 要求會導致頁面&#x200B;**不會**&#x200B;被快取，因為`nocache`參數不會被忽略：

```xml
GET /mypage.html?nocache=true
GET /mypage.html?nocache=true&willbecached=true
```

如需 glob 屬性的相關資訊，請參閱[為 glob 屬性設計模式](#designing-patterns-for-glob-properties)。

### 快取 HTTP 回應標頭 {#caching-http-response-headers}

>[!NOTE]
>
>Dispatcher 版本 **4.1.11** 可使用此功能。

`/headers` 屬性可讓您定義將由 Dispatcher 快取的 HTTP 標題類型。初次請求未快取的資源時，符合其中一個設定值 (請參閱底下的設定範例) 的所有標題都會儲存在快取檔案旁邊的另一個檔案中。後續請求快取的資源時，儲存的標題會新增到回應中。

下面顯示的是預設設定中的範例：

```xml
/cache {
  ...
  /headers {
    "Cache-Control"
    "Content-Disposition"
    "Content-Type"
    "Expires"
    "Last-Modified"
    "X-Content-Type-Options"
    "Last-Modified"
  }
}
```

>[!NOTE]
>
>不允許使用檔案萬用字元。 如需詳細資訊，請參閱[為 glob 屬性設計模式](#designing-patterns-for-glob-properties)。

>[!NOTE]
>
>如果您需要 Dispatcher 儲存並傳遞 AEM 中的 ETag 回應標頭，請執行以下操作：
>
>* 在 `/cache/headers` 區段中新增標題名稱。
>* 在 Dispatcher 相關區段中新增以下 [Apache 指示詞](https://httpd.apache.org/docs/2.4/mod/core.html#fileetag)：
>
>```xml
>FileETag none
>```

### Dispatcher 快取檔案權限 {#dispatcher-cache-file-permissions}

`mode` 屬性會指定哪些檔案權限會套用到快取中的新目錄和檔案。此設定受限於呼叫程序的 `umask`。這是一個八進位數字，由下列一個或多個值的總和所構成：

* `0400` 允許所有者讀取。
* `0200` 允許所有者寫入。
* `0100` 允許所有者在目錄中搜尋。
* `0040` 允許群組成員讀取。
* `0020` 允許群組成員寫入。
* `0010` 允許群組成員在目錄中搜尋。
* `0004` 允許其他人讀取。
* `0002` 允許其他人寫入。
* `0001` 允許其他人在目錄中搜尋。

預設值為 `0755`，允許所有者讀取、寫入或搜尋，並允許群組成員和其他人讀取或搜尋。

### 限制 .stat 檔案的接觸 {#throttling-stat-file-touching}

使用預設 `/invalidate` 屬性時，每次啟用都會有效地讓所有 `.html` 檔案失效 (當其路徑符合 `/invalidate` 區段時)。在流量很大的網站上，後續的多次啟用會增加後端的 CPU 負載。在這種情況下，最好「限制」`.stat` 檔案的接觸，以維持網站的回應能力。 您可以使用 `/gracePeriod` 屬性來完成此動作。

`/gracePeriod` 屬性會定義在最後一次啟用後，仍然可以從快取中提供舊的、已自動失效的資源的秒數。 此屬性適用於以下情況的設定：如果不這樣設定，有一批啟用會反覆地讓整個快取失效。建議值為 2 秒。

如需其他詳細資訊，請閱讀上面的 `/invalidate` 和 `/statfileslevel` 區段。

### 設定以時間為基礎的快取失效 - /enableTTL {#configuring-time-based-cache-invalidation-enablettl}

以時間為基礎的快取失效取決於 `/enableTTL` 屬性以及來自 HTTP 標準的一般到期標頭是否存在。 如果您將屬性設為1 (`/enableTTL "1"`)，則會評估來自後端的回應標頭。 如果標題包含 `Cache-Control`， `max-age` 或 `Expires` 日期，在快取檔案旁邊建立一個空的輔助檔案，其修改時間等於到期日。 在修改時間過後請求快取檔案時，將會自動從後端重新請求該檔案。

在 Dispatcher 4.3.5 之前，TTL 失效邏輯僅根據設定的 TTL 值。 使用 Dispatcher 4.3.5，設定的 TTL **和** Dispatcher 快取失效規則都會列入考量。 因此，對於快取的檔案：

1. 如果 `/enableTTL` 設定為 1，會檢查檔案是否過期。如果根據設定的 TTL 確定檔案已過期，則不執行其他檢查，並從後端重新請求快取的檔案。
2. 如果檔案未過期或 `/enableTTL` 未設定，則會套用標準快取失效規則，就像 [/statfileslevel](#invalidating-files-by-folder-level) 和 [/invalidate](#automatically-invalidating-cached-files) 所設定的規則。 此流程表示 Dispatcher 可以使 TTL 尚未過期的檔案失效。

此新實作支援的使用案例是檔案有較長的 TTL (例如，在 CDN 上)，但即使 TTL 尚未過期檔案仍可能失效。它有利於內容處於最新狀態，而不是 Dispatcher 上的快取命中率。

相反地，如果您&#x200B;**僅**&#x200B;需要套用到檔案的到期邏輯，則將 `/enableTTL` 設定為 1，並從標準快取失效機制中排除該檔案。 例如，您可以：

* 若要忽略該檔案，請在快取區段設定[失效規則](#automatically-invalidating-cached-files)。 在下面的程式碼片段中，所有結尾是 `.example.html` 的檔案都會忽略，並且僅當超過設定的 TTL 時才會過期。

```xml
  /invalidate
  {
   /0000  { /glob "*" /type "deny" }
   /0001  { /glob "*.html" /type "allow" }
   /0002  { /glob "*.example.html" /type "deny" }
  }
```

* 設計內容結構時將 [/statfilelevel](#invalidating-files-by-folder-level) 設定較高，如此檔案不會自動失效。

這麼做確保不會使用 `.stat` 檔案失效，只有 TTL 到期對指定的檔案有效。

>[!NOTE]
>
>請記住將 `/enableTTL` 設定為 1 只會在 Dispatcher 端啟用 TTL 快取。因此，其他檔案 (見上文) 包含的 TTL 資訊不會提供給要求來自 Dispatcher 之此檔案類型的任何其他使用者代理。如果你想要提供快取標頭給下游系統 (如 CDN 或瀏覽器)，你應該相應地設定 `/cache/headers` 區段。

>[!NOTE]
>
>Dispatcher 版本 **4.1.11** 或更新版本中可使用此功能。

## 設定負載平衡 - /statistics {#configuring-load-balancing-statistics}

`/statistics` 區段會定義檔案類別，Dispatcher 針對這類檔案為每個轉譯器的回應能力評分。Dispatcher 會使用這些評分來決定哪個轉譯器要傳送請求。

您建立的每個類別都會定義 glob 模式。Dispatcher 會將請求內容的 URI 與這些模式做比較，以判斷請求內容的類別：

* 類別的順序會決定將其與 URI 比較的順序。
* 第一個符合 URI 的類別模式為該檔案的類別。不再評估其他類別模式。

Dispatcher 最多可支援 8 個統計類別。 如果您定義 8 個以上的類別，只會使用前 8 個類別。

**轉譯器選擇**

每當 Dispatcher 需要轉譯頁面時，它都會使用以下演算法來選取轉譯器：

1. 如果請求在 `renderid` Cookie 中包含轉譯器名稱，Dispatcher 會使用該轉譯器。
1. 如果請求未包含 `renderid` Cookie，Dispatcher 會比較轉譯器統計資料：

   1. Dispatcher 會判斷請求 URI 的類別。
   1. Dispatcher 會判斷哪一個轉譯器在該類別中的回應分數最低，並選取該轉譯器。

1. 如果尚未選取轉譯器，則會使用清單中的第一個轉譯器。

轉譯器類別的分數是根據之前的回應時間，以及之前 Dispatcher 嘗試建立的失敗和成功的連線。 對於每次嘗試，都會更新請求的 URI 的類別分數。

>[!NOTE]
>
>如果您未使用負載平衡，可以省略此區段。

### 定義統計資料類別 {#defining-statistics-categories}

針對您想要保留統計資料以便選擇轉譯器的每個文件類型定義一個類別。`/statistics` 區段包含 `/categories` 區段。若要定義類別，請在 `/categories` 區段底下新增具有以下格式的一行：

`/name { /glob "pattern"}`

類別 `name` 對該陣列而言必須是獨一無二的。[為 glob 屬性設計模式](#designing-patterns-for-glob-properties)一節中有說明 `pattern`。

為了判斷 URI 的類別，Dispatcher 會將 URI 與每個類別模式做比較，直到找到符合項目為止。Dispatcher 會從清單中的第一個類別開始，然後依順序繼續進行。 所以，請將具有更具體模式的類別放在前面。

例如，Dispatcher 預設 `dispatcher.any` 檔案會定義 HTML 類別以及一個其他類別。HTML 類別比較具體，所以會先出現：

```xml
/statistics
  {
  /categories
    {
      /html { /glob "*.html" }
      /others  { /glob "*" }
    }
  }
```

以下範例也包含搜尋頁面的類別：

```xml
/statistics
  {
  /categories
    {
      /search { /glob "*search.html" }
      /html { /glob "*.html" }
      /others  { /glob "*" }
    }
  }
```

### 在 Dispatcher 統計資料中反映伺服器無法使用 {#reflecting-server-unavailability-in-dispatcher-statistics}

`/unavailablePenalty` 屬性會設定當與轉譯器的連線失敗時，套用到轉譯器統計資料的時間 (以十分之一秒為單位)。Dispatcher 會將此時間新增到符合請求的 URI 的統計資料類別。

例如，當無法建立與指定的主機名稱/連接埠的 TCP/IP 連線時就會套用懲罰，這是因為 AEM 未執行 (也未偵聽) 或是網路相關問題所造成。

`/unavailablePenalty` 屬性是 `/farm` 區段的直接子項 (`/statistics` 區段的同層級)。

如果沒有 `/unavailablePenalty` 屬性存在，則會使用 `"1"` 的值。

```xml
/unavailablePenalty "1"
```

## 識別黏性連線資料夾 - /stickyConnectionsFor {#identifying-a-sticky-connection-folder-stickyconnectionsfor}

`/stickyConnectionsFor` 屬性會定義一個包含黏性文件的資料夾。 此屬性是使用 URL 存取。 Dispatcher 會將來自單一使用者的所有請求 (都在此資料夾中) 傳送給相同轉譯器執行個體。 黏性連線可確保工作階段資料存在，且在所有文件中都是一致的。此機制會使用 `renderid` Cookie。

以下範例定義與 /products 資料夾的黏性連線：

```xml
/stickyConnectionsFor "/products"
```

當頁面是由數個內容節點中的內容所組成時，請包含列出內容路徑的 `/paths` 屬性。例如，某個頁面包含 `/content/image`、`/content/video` 和 `/var/files/pdfs` 中的內容。以下設定會針對該頁面上的所有內容啟用黏性連線：

```xml
/stickyConnections {
  /paths {
    "/content/image"
    "/content/video"
    "/var/files/pdfs"
  }
}
```

### httpOnly {#httponly}

當啟用黏性連線時，Dispatcher 模組會設定 `renderid` Cookie。 此 Cookie 沒有 `httponly` 標幟，應該要新增此標幟以增強安全性。 您新增 `httponly` 標幟的方式是設定 `httpOnly` 屬性，位於 `/stickyConnections` 節點中 (`dispatcher.any` 設定檔案)。 該屬性的值 (`0` 或 `1`) 會定義 `renderid` Cookie 是否已附加 `HttpOnly` 屬性。預設值為 `0`，這表示不會新增此屬性。

如需有關 `httponly` 標幟的其他資訊，請閱讀[此頁面](https://www.owasp.org/index.php/HttpOnly)。

### secure {#secure}

當啟用黏性連線時，Dispatcher 模組會設定 `renderid` Cookie。 此 Cookie 沒有 `secure` 標幟，應該要新增此標幟以增強安全性。 您新增 `secure` 標幟，設定 `secure` 屬性，位於 `/stickyConnections` 節點中 (`dispatcher.any` 設定檔案)。 該屬性的值 (`0` 或 `1`) 會定義 `renderid` Cookie 是否已附加 `secure` 屬性。預設值為 `0`，這表示&#x200B;**如果**&#x200B;傳入的請求是安全的，會新增此屬性。 如果此值設定為 `1`，則無論傳入的請求是否安全，都會新增 secure 標幟。

## 處理轉譯器連線錯誤 {#handling-render-connection-errors}

設定當轉譯器伺服器傳回 500 錯誤或無法使用時的 Dispatcher 行為。

### 指定健康情況檢查頁面 {#specifying-a-health-check-page}

使用 `/health_check` 屬性可指定在出現 500 狀態代碼時檢查的 URL。如果此頁面也傳回 500 狀態代碼，則會將執行個體視為無法使用，並在重試之前將可設定的時間懲罰 ( `/unavailablePenalty`) 套用到轉譯器。

```xml
/health_check
  {
  # Page gets contacted when an instance returns a 500
  /url "/health_check.html"
  }
```

### 指定頁面重試延遲 {#specifying-the-page-retry-delay}

`/retryDelay` 屬性會設定 Dispatcher 嘗試與陣列轉譯器建立的連線輪次之間等待的時間 (以秒為單位)。針對每一輪，Dispatcher 嘗試與轉譯器建立連線的次數上限為陣列中的轉譯器數目。

如果未明確定義 `/retryDelay`，Dispatcher 會使用 `"1"` 的值。預設值一般都適用。

```xml
/retryDelay "1"
```

### 設定重試次數 {#configuring-the-number-of-retries}

`/numberOfRetries` 屬性會設定 Dispatcher 嘗試與轉譯器建立的連線輪次的數目上限。如果 Dispatcher 在超過此重試次數後還是無法成功連線到轉譯器，Dispatcher 會傳回失敗的回應。

針對每一輪，Dispatcher 嘗試與轉譯器建立連線的次數上限為陣列中的轉譯器數目。因此，Dispatcher 嘗試建立連線的次數上限為 ( `/numberOfRetries`) x (轉譯器數目)。

如果未明確定義此值，預設值為 `5`。

```xml
/numberOfRetries "5"
```

### 使用容錯移轉機制 {#using-the-failover-mechanism}

若要在原始請求失敗時重新傳送請求給不同的轉譯器，請在您的 Dispatcher 陣列上啟用容錯移轉機制。 當啟用容錯移轉時，Dispatcher 具有以下行為：

* 當傳送給轉譯器的請求傳回 HTTP 狀態 503 (無法使用) 時，Dispatcher 會將該請求傳送給不同的轉譯器。
* 當傳送給轉譯器的請求傳回 HTTP 狀態 50x (而不是 503) 時，Dispatcher 會為已設定 `health_check` 屬性的頁面傳送請求。
   * 如果健康情況檢查傳回 500 (INTERNAL_SERVER_ERROR)，Dispatcher 會將原始請求傳送給不同的轉譯器。
   * 如果健康情況檢查傳回 HTTP 狀態 200，Dispatcher 會將最初的 HTTP 500 錯誤傳回給用戶端。

若要啟用容錯移轉，請在陣列 (或網站) 中新增下面這一行：

```xml
/failover "1"
```

>[!NOTE]
>
>為了重試包含本文的 HTTP 請求，Dispatcher 會傳送 `Expect: 100-continue` 請求標題給轉譯器，然後進行實際內容的多工緩衝處理。然後含 CQSE 的 CQ 5.5 會立即傳送 100 (繼續) 或錯誤碼當作回應。其他 servlet 容器也支援此功能。

## 忽略中斷錯誤 - /ignoreEINTR {#ignoring-interruption-errors-ignoreeintr}

>[!CAUTION]
>
>不需要這個選項。只有當您看到以下記錄訊息時才使用此選項：
>
>`Error while reading response: Interrupted system call`

如果系統呼叫的物件位在透過 NFS 的方式存取的遠端系統上，則任何檔案系統導向的系統呼叫都可以被中斷 `EINTR`。 這些系統呼叫可以逾時還是被中斷，取決於基礎檔案系統如何在本機電腦上裝載。

如果您的執行個體有這類設定，而且記錄中包含以下訊息，請使用 `/ignoreEINTR` 參數：

`Error while reading response: Interrupted system call`

在內部，Dispatcher 會使用可以表示如下的迴圈來讀取來自遠端伺服器 (亦即 AEM) 的回應：

```text
while (response not finished) {  
read more data  
}
```

當「`read more data`」區段中出現 `EINTR`，而且是因為在收到任何資料之前收到訊號所導致，則會產生這類訊息。

若要忽略這類中斷，可以將下面參數新增到 `dispatcher.any` (在 `/farms` 前面)：

`/ignoreEINTR "1"`

將 `/ignoreEINTR` 設為 `"1"` 會讓 Dispatcher 繼續嘗試讀取資料，直到讀取完整回應為止。預設值為 `0`，且會停用此選項。

## 為 glob 屬性設計模式 {#designing-patterns-for-glob-properties}

Dispatcher 設定檔案中有幾個區段會使用 `glob` 屬性當作用戶端請求的選擇標準。`glob` 屬性的值是 Dispatcher 與請求的某個層面進行比較的模式，例如請求的資源的路徑或是用戶端的 IP 位址。例如，`/filter` 區段中的項目會使用 `glob` 模式來識別 Dispatcher 採取行動或拒絕的頁面的路徑。

`glob` 值可包含萬用字元和英數字元來定義模式。

| 萬用字元 | 說明 | 範例 |
|--- |--- |--- |
| `*` | 符合字串中任何字元的零個或多個連續執行個體。相符項目的最後一個字元是由以下情況的任何一個所決定：<br/>字串中的某個字元符合模式中的下一個字元，且模式字元具有以下特性：<br/><ul><li>不是 *</li><li>不是 ?</li><li>常值字元 (包括空格) 或字元類別。</li><li>已達模式的結尾。</li></ul>在字元類別內，依照字面上的意義解譯字元。 | `*/geo*` 符合在 `/content/geometrixx` 節點和 `/content/geometrixx-outdoors` 節點底下的任何頁面。以下 HTTP 請求符合 glob 模式：<br/><ul><li>`"GET /content/geometrixx/en.html"`</li><li>`"GET /content/geometrixx-outdoors/en.html"` </li></ul><br/> `*outdoors/*` <br/>符合在 `/content/geometrixx-outdoors` 節點底下的任何頁面。例如，以下 HTTP 請求符合 glob 模式：<br/><ul><li>`"GET /content/geometrixx-outdoors/en.html"`</li></ul> |
| `?` | 符合任何單一字元。使用外部字元類別。在字元類別內，依照字面上的意義解譯此字元。 | `*outdoors/??/*`<br/> 符合 geometrixx-outdoors 網站中任何語言的頁面。例如，以下 HTTP 請求符合 glob 模式：<br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul><br/>以下請求不符合 glob 模式：<br/><ul><li>&quot;GET /content/geometrixx-outdoors/en.html&quot;</li></ul> |
| `[ and ]` | 取消標記字元類別的開頭和結尾。字元類別可包含一個或多個字元範圍及單一字元。<br/>如果目標字元符合字元類別中或定義的範圍內的任何字元，就會有符合項目。<br/>如果未包含右中括號，則模式不會產生任何符合項目。 | `*[o]men.html*`<br/> 符合下列 HTTP 請求：<br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li></ul><br/>不符合下列 HTTP 請求：<br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul><br/> `*[o/]men.html*` <br/>符合下列 HTTP 請求：<br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul> |
| `-` | 代表一連串的字元。用於字元類別中。在字元類別外，依照字面上的意義解譯此字元。 | `*[m-p]men.html*` 符合下列 HTTP 請求：<br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li></ul>不符合下列 HTTP 請求：<br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul> |
| `!` | 讓緊接在後面的字元或字元類別無效。僅用於讓字元類別內的字元或字元範圍無效。相當於 `^ wildcard`。<br/>在字元類別外，依照字面上的意義解譯此字元。 | `*[!o]men.html*`<br/> 符合下列 HTTP 請求：<br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul><br/>不符合下列 HTTP 請求：<br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li></ul><br/>`*[!o!/]men.html*`<br/> 不符合下列 HTTP 請求：<br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"` 或 `"GET /content/geometrixx-outdoors/en/men. html"`</li></ul> |
| `^` | 讓緊接在後面的字元或字元範圍無效。僅用於讓字元類別內的字元或字元範圍無效。相當於 `!` 萬用字元。<br/>在字元類別外，依照字面上的意義解譯此字元。 | 適用 `!` 萬用字元的範例，將範例模式中的 `!` 字元取代為 `^` 字元。 |


<!--- need to troubleshoot table

The following table describes the wildcard characters.

<table border="1" cellpadding="1" cellspacing="0" width="100%"> 
 <tbody> 
  <tr> 
   <th>Wildcard character</th> 
   <th>Description</th> 
   <th>Examples</th> 
  </tr> 
  <tr> 
   <td>*</td> 
   <td><p>Matches zero or more contiguous instances of any character in the string. The final character of the match is determined by either of the following situations:</p> 
    <ul> 
     <li>A character in the string matches the next character in the pattern, and the pattern character has the following characteristics: 
      <ul> 
       <li>Not a *</li> 
       <li>Not a ?</li> 
       <li>A literal character (including a space) or a character class.</li> 
      </ul> </li> 
     <li>The end of the pattern is reached.</li> 
    </ul> <p>Inside a character class, the character is interpreted literally.</p> </td> 
   <td><p>*/geo*</p> <p>Matches any page below the /content/geometrixx node and the /content/geometrixx-outdoors node. The following HTTP requests match the glob pattern:</p> 
    <ul> 
     <li>"GET /content/geometrixx/en.html"</li> 
     <li>"GET /content/geometrixx-outdoors/en.html" </li> 
    </ul> <p>*outdoors/*</p> <p>Matches any page below the /content/geometrixx-outdoors node. For example, the following HTTP request matches the glob pattern:</p> 
    <ul> 
     <li>"GET /content/geometrixx-outdoors/en.html"</li> 
    </ul> </td> 
  </tr> 
  <tr> 
   <td>?</td> 
   <td><p>Matches any single character. Use outside character classes.</p> <p>Inside a character class, this character is interpreted literally.</p> </td> 
   <td><p>*outdoors/??/*</p> <p>Matches the pages for any language in the geometrixx-outdoors site. For example, the following HTTP request matches the glob pattern:</p> 
    <ul> 
     <li>"GET /content/geometrixx-outdoors/en/men.html"</li> 
    </ul> <p>The following request does not match the glob pattern:</p> 
    <ul> 
     <li>"GET /content/geometrixx-outdoors/en.html" </li> 
    </ul> </td> 
  </tr> 
  <tr> 
   <td>[ and ]</td> 
   <td><p>Demarks the beginning and end of a character class.</p> <p>Character classes can include one or more character ranges and single characters.</p> <p>A match occurs if the target character matches any of the characters in the character class, or within a defined range.</p> <p>If the closing bracket is not included, the pattern produces no matches.</p> <p></p> <p></p> <p></p> </td> 
   <td><p>*[o]men.html*</p> <p>Matches the following HTTP request:</p> 
    <ul> 
     <li>"GET /content/geometrixx-outdoors/en/women.html"</li> 
    </ul> <p>Does not match the following HTTP request:</p> 
    <ul> 
     <li>"GET /content/geometrixx-outdoors/en/men.html" </li> 
    </ul> <p>*[o/]men.html*</p> <p>Matches the following HTTP requests: </p> 
    <ul> 
     <li>"GET /content/geometrixx-outdoors/en/women.html" </li> 
     <li> "GET /content/geometrixx-outdoors/en/men.html" </li> 
    </ul> <p></p> </td> 
  </tr> 
  <tr> 
   <td>-</td> 
   <td><p>Denotes a range of characters. For use in character classes.<br /> </p> <p>Outside of a character class, this character is interpreted literally.</p> <p></p> </td> 
   <td><p>*[m-p]men.html*</p> <p>Matches the following HTTP request:</p> 
    <ul> 
     <li>"GET /content/geometrixx-outdoors/en/women.html"</li> 
    </ul> <p>Does not match the following HTTP request:</p> 
    <ul> 
     <li>"GET /content/geometrixx-outdoors/en/men.html"</li> 
    </ul> <p></p> </td> 
  </tr> 
  <tr> 
   <td>!</td> 
   <td><p>Negates the character or character class that follows. Use only for negating characters and character ranges inside character classes. Equivalent to the ^ wildcard.</p> <p>Outside of a character class, this character is interpreted literally.</p> <p></p> </td> 
   <td><p>*[!o]men.html*</p> <p>Matches the following HTTP request: </p> 
    <ul> 
     <li>"GET /content/geometrixx-outdoors/en/men.html"</li> 
    </ul> <p>Does not match the following HTTP request</p> 
    <ul> 
     <li>"GET /content/geometrixx-outdoors/en/women.html" </li> 
    </ul> <p>*[!o!/]men.html*</p> <p>Does not match the following HTTP request:</p> 
    <ul> 
     <li>"GET /content/geometrixx-outdoors/en/women.html" or "GET /content/geometrixx-outdoors/en/men. html" </li> 
    </ul> </td> 
  </tr> 
  <tr> 
   <td>^</td> 
   <td><p>Negates the character or character range that follows. Use for negating only characters and character ranges inside character classes. Equivalent to the ! wildcard character.</p> <p>Outside of a character class, this charcter is interpreted literally.</p> </td> 
   <td>The examples for the ! wildcard character apply, substituting the ! characters in the example patterns with ^ characters.</td> 
  </tr> 
 </tbody> 
</table>
-->

## 記錄 {#logging}

在網頁伺服器設定中，您可以設定：

* Dispatcher 記錄檔的位置。
* 記錄層級。

如需詳細資訊，請參閱您的 Dispatcher 執行個體的網頁伺服器文件和讀我檔案。

**Apache 輪換/管道傳輸記錄**

如果您使用 **Apache** 網頁伺服器，可以將標準功能用於輪換記錄或管道傳輸記錄，或兩者。 例如，使用管道傳輸記錄：

`DispatcherLog "| /usr/apache/bin/rotatelogs logs/dispatcher.log%Y%m%d 604800"`

此功能自動輪換：

* Dispatcher 記錄檔；副檔名中有時間戳記 (`logs/dispatcher.log%Y%m%d`)。
* 每週 (60 x 60 x 24 x 7 = 604800 秒)。

請參閱 Apache 網頁伺服器文件中有關記錄輪換和管道傳輸記錄的內容。 例如，[Apache 2.4](https://httpd.apache.org/docs/2.4/logs.html)。

>[!NOTE]
>
>安裝後，預設記錄層級為高 (亦即層級 3 = 偵錯)，所以 Dispatcher 會記錄所有錯誤和警告。 此層級在初始階段非常有用。
>
>然而，此等層級需要額外的資源。 當 Dispatcher *根據您的需求*&#x200B;順利運作時，您可以降低記錄層級。

### 追蹤記錄 {#trace-logging}

在 Dispatcher 的其他增強功能中，版本 4.2.0 也引進了追蹤記錄。

此功能層級高於偵錯記錄，會在記錄中顯示額外的資訊。 它會新增以下項目的記錄：

* 轉送的標題的值；
* 正套用到某個動作的規則。

若要啟用追蹤記錄，請在網頁伺服器中將記錄層級設定為 `4`。

底下是已啟用追蹤的記錄範例：

```xml
[Thu Mar 03 16:05:38 2016] [T] [17183] request.headers[Host] = "localhost:8443"
[Thu Mar 03 16:05:38 2016] [T] [17183] request.headers[User-Agent] = "curl/7.43.0"
[Thu Mar 03 16:05:38 2016] [T] [17183] request.headers[Accept] = "*/*"
[Thu Mar 03 16:05:38 2016] [T] [17183] request.headers[X-Forwarded-SSL-Client-Cert] = "(null)"
[Thu Mar 03 16:05:38 2016] [T] [17183] request.headers[Via] = "1.1 localhost:8443 (dispatcher)"
[Thu Mar 03 16:05:38 2016] [T] [17183] request.headers[X-Forwarded-For] = "::1"
[Thu Mar 03 16:05:38 2016] [T] [17183] request.headers[X-Forwarded-SSL] = "on"
[Thu Mar 03 16:05:38 2016] [T] [17183] request.headers[X-Forwarded-SSL-Cipher] = "DHE-RSA-AES256-SHA"
[Thu Mar 03 16:05:38 2016] [T] [17183] request.headers[X-Forwarded-SSL-Session-ID] = "ba931f5e4925c2dde572d766fdd436375e15a0fd24577b91f4a4d51232a934ae"
[Thu Mar 03 16:05:38 2016] [T] [17183] request.headers[X-Forwarded-Port] = "8443"
[Thu Mar 03 16:05:38 2016] [T] [17183] request.headers[Server-Agent] = "Communique-Dispatcher"
```

在請求符合封鎖規則的檔案時記錄了一個事件：

```xml
[Thu Mar 03 14:42:45 2016] [T] [11831] 'GET /content.infinity.json HTTP/1.1' was blocked because of /0082
```

## 確認基本操作 {#confirming-basic-operation}

若要確認網頁伺服器、Dispatcher 和 AEM 執行個體的基本操作和互動，您可以使用以下步驟：

1. 將 `loglevel` 設為 `3`。

1. 啟動網頁伺服器。 這麼做也會啟動 Dispatcher。
1. 啟動 AEM 執行個體。
1. 檢查網頁伺服器和 Dispatcher 的記錄檔和錯誤檔。
   * 根據您使用的網頁伺服器，您應該會看到類似以下的訊息：
      * `[Thu May 30 05:16:36 2002] [notice] Apache/2.0.50 (Unix) configured` 和
      * `[Fri Jan 19 17:22:16 2001] [I] [19096] Dispatcher initialized (build XXXX)`

1. 透過網頁伺服器隨意瀏覽網站。確認顯示的內容符合您的要求。\
   例如，在 AEM 於連接埠 `4502` 上執行且網頁伺服器於 `80` 上執行的本機安裝中，使用以下兩者來存取網站主控台：
   * `https://localhost:4502/libs/wcm/core/content/siteadmin.html`
   * `https://localhost:80/libs/wcm/core/content/siteadmin.html`
   * 結果應該是相同的。使用相同機制確認對其他頁面的存取權。

1. 檢查是否有填入快取目錄。
1. 若要檢查是否已正確清除快取，請啟用一個頁面。
1. 如果一切都正常運作，您可以將 `loglevel` 降低為 `0`。

## 使用多個 Dispatcher {#using-multiple-dispatchers}

您可以透過複雜的設定來使用多個 Dispatcher。例如您可以使用：

* 一個 Dispatcher 在內部網路上發佈網站
* 另一個 Dispatcher 位於不同的位址下，並具有不同的安全設定，以便在網際網路上發佈相同的內容。

在這種情況下，請確定每個請求只通過一個 Dispatcher。Dispatcher 不會處理來自其他 Dispatcher 的請求。因此，請確定兩個 Dispatcher 都直接存取 AEM 網站。

## 偵錯 {#debugging}

將標頭 `X-Dispatcher-Info` 新增到請求時，Dispatcher 會回應是快取了目標、從快取中返回還是完全不可快取。 回應標頭 `X-Cache-Info` 會在可讀的表單中包含此資訊。您可以使用這些回應標頭來偵錯 Dispatcher 快取回應的相關問題。

預設不會啟用此功能，所以若要包含回應標頭 `X-Cache-Info`，陣列必須包含以下項目：

```xml
/info "1"
```

例如，

```xml
/farm
{
    /mywebsite
    {
        # Include X-Cache-Info response header if X-Dispatcher-Info is in request header
        /info "1"
    }
}
```

此外，`X-Dispatcher-Info` 標頭不需要值，但如果您使用 `curl` 進行測試，就必須提供值以便傳送標頭，例如：

```xml
curl -v -H "X-Dispatcher-Info: true" https://localhost/content/wknd/us/en.html
```

底下是包含 `X-Dispatcher-Info` 會傳回的回應標頭的清單：

* **cached**\
  目標檔案包含在快取中，而且 Dispatcher 已判定傳遞它是有效的。
* **caching**\
  目標檔案未包含在快取中，而且 Dispatcher 已判定快取輸出並傳遞它是有效的。
* **caching: stat file is more recent**
目標檔案包含在快取中，但是較新版的統計檔案使其失效。Dispatcher 會刪除目標檔案、從輸出中重建檔案並傳遞它。
* **not cacheable: no document root**
陣列的設定不包含主目錄 (設定元素 `cache.docroot`)。
* **not cacheable: cache file path too long**\
  目標檔案 (主目錄和 URL 檔案的串連) 超出系統上可行的最長檔案名稱。
* **not cacheable: temporary file path too long**\
  暫存檔案名稱範本超出系統上可行的最長檔案名稱。Dispatcher 會在實際建立或覆寫快取檔案之前先建立暫存檔案。 暫存檔案名稱是有附加 `_YYYYXXXXXX` 字元的目標檔案名稱，其中會取代 `Y` 和 `X` 以建立唯一名稱。
* **not cacheable: request URL has no extension**\
  請求 URL 沒有副檔名，或是副檔名後面有路徑，例如：`/test.html/a/path`。
* **not cacheable: request wasn&#39;t a GET or HEAD**
HTTP 方法不是 GET 或 HEAD。Dispatcher 假設輸出會包含不應該快取的動態資料。
* **not cacheable: request contained a query string**\
  請求包含查詢字串。Dispatcher 假設輸出取決於提供的查詢字串，所以不會快取。
* **not cacheable: session manager didn&#39;t authenticate**\
  陣列的快取是由工作階段管理員所控管 (設定包含 `sessionmanagement` 節點)，而且請求未包含適當的驗證資訊。
* **not cacheable: request contains authorization**\
  不允許陣列快取輸出 ( `allowAuthorized 0`) 而且請求包含驗證資訊。
* **not cacheable: target is a directory**\
  目標檔案是目錄。此位置可能指向一些概念錯誤，其中 URL 和某些子 URL 都包含可快取的輸出。 例如，如果對 `/test.html/a/file.ext` 的請求先出現並且包含可快取的輸出，則 Dispatcher 無法快取對 `/test.html` 的後續請求的輸出。
* **not cacheable: request URL has a trailing slash**\
  請求 URL 後面有斜線。
* **not cacheable: request URL not in cache rules**\
  陣列的快取規則明確拒絕快取部分請求 URL 的輸出。
* **not cacheable: authorization checker denied access**\
  陣列的授權檢查程式拒絕存取快取檔案。
* **not cacheable: session not valid**
陣列的快取是由工作階段管理員所控管 (設定包含 `sessionmanagement` 節點)，而且使用者的工作階段無效或不再有效。
* **not cacheable： response contains`no_cache`**
遠端伺服器傳回 `Dispatcher: no_cache` 標頭，禁止Dispatcher快取輸出。
* **not cacheable: response content length is zero**
回應的內容長度為零；Dispatcher 不會建立長度為零的檔案。
