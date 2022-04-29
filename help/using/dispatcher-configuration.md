---
title: 設定 Dispatcher
description: 瞭解如何配置Dispatcher。
exl-id: 91159de3-4ccb-43d3-899f-9806265ff132
source-git-commit: deb232be3c4c5e3d11d13cbccb282409d90b93bb
workflow-type: tm+mt
source-wordcount: '8528'
ht-degree: 98%

---

# 設定 Dispatcher {#configuring-dispatcher}

>[!NOTE]
>
>Dispatcher 版本與 AEM 無關。如果您依循連結至 Dispatcher 文件，且該連結內嵌於舊版 AEM 的文件中，您可能會被重新導向至本頁。

以下各節介紹如何配置Dispatcher的各個方面。

## 支援IPv4和IPv6 {#support-for-ipv-and-ipv}

Dispatcher和AEMIpv4的所有元素都可以安裝在IPv4和IPv6網路中。 請參閱 [IPV4和IPV6](https://experienceleague.adobe.com/docs/experience-manager-65/deploying/introduction/technical-requirements.html?lang=en#ipv-and-ipv)。

## Dispatcher配置檔案 {#dispatcher-configuration-files}

預設情況下， Dispatcher配置儲存在 `dispatcher.any` 文本檔案，但您可以在安裝期間更改此檔案的名稱和位置。

配置檔案包含一系列單值或多值屬性，這些屬性控制Dispatcher的行為：

* 屬性名稱前置詞為正斜槓 `/`。
* 多值屬性使用大括弧將子項括起來 `{ }`。

示例配置的結構如下：

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

您可以包括其他有助於配置的檔案：

* 如果配置檔案很大，可以將其拆分為多個較小的檔案（更易於管理），則包括這些檔案。
* 包含自動生成的檔案。

例如，要在/farms配置中包含檔案myFarm.any，請使用以下代碼：

```xml
/farms
  {
  $include "myFarm.any"
  }
```

使用星號(`*`)作為通配符，以指定要包括的檔案範圍。

例如，如果 `farm_1.any` 到 `farm_5.any` 包含一到五個農場的配置，您可以包括以下內容：

```xml
/farms
  {
  $include "farm_*.any"
  }
```

## 使用環境變數 {#using-environment-variables}

您可以在dispatcher.any檔案中的字串值屬性中使用環境變數，而不是硬編碼值。 要包括環境變數的值，請使用 `${variable_name}`。

例如，如果dispatcher.any檔案與快取目錄位於同一目錄中，則 [多克羅](#specifying-the-cache-directory) 屬性可用：

```xml
/docroot "${PWD}/cache"
```

作為另一個示例，如果建立名為 `PUBLISH_IP` 儲存發佈實例的AEM主機名， [/renders](#defining-page-renderers-renders) 屬性可用：

```xml
/renders {
  /0001 {
    /hostname "${PUBLISH_IP}"
    /port "8443"
  }
}
```

## 命名Dispatcher實例 {#naming-the-dispatcher-instance-name}

使用 `/name` 屬性，以指定唯一名稱以標識Dispatcher實例。 的 `/name` 屬性是配置結構中的頂級屬性。

## 定義場 {#defining-farms-farms}

的 `/farms` 屬性定義一個或多個Dispatcher行為集，其中每個集與不同的網站或URL關聯。 的 `/farms` 該屬性可以包括單個場或多個場：

* 如果希望Dispatcher以相同方式處理所有網頁或網站，請使用單個伺服器場。
* 當您的網站或不同網站的不同區域需要不同的Dispatcher行為時，建立多個場。

的 `/farms` 屬性是配置結構中的頂級屬性。 要定義場，請向 `/farms` 屬性。 使用唯一標識Dispatcher實例中的場的屬性名稱。

的 `/farmname` 屬性是多值的，並包含定義Dispatcher行為的其他屬性：

* 伺服器場應用的頁面的URL。
* 用於呈現文檔的一個或多個服務URL(AEM通常為發佈實例)。
* 用於負載平衡多個單據呈現器的統計資訊。
* 其他幾種行為，如要快取的檔案和位置。

該值可以包含任何字母數字(a-z, 0-9)字元。 以下示例顯示了名為的兩個場的骨架定義 `/daycom` 和 `/docsdaycom`:

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
>如果使用多個呈現場，則將從下到上計算清單。 在定義 [虛擬主機](#identifying-virtual-hosts-virtualhosts) 你的網站。

每個場屬性都可以包含以下子屬性：

| 屬性名稱 | 說明 |
|--- |--- |
| [/首頁](#specify-a-default-page-iis-only-homepage) | 預設首頁（可選）（僅限IIS） |
| [/clieader](#specifying-the-http-headers-to-pass-through-clientheaders) | 要傳遞的客戶端HTTP請求的標頭。 |
| [/virtualhosts](#identifying-virtual-hosts-virtualhosts) | 此場的虛擬主機。 |
| [/會話管理](#enabling-secure-sessions-sessionmanagement) | 支援會話管理和身份驗證。 |
| [/renders](#defining-page-renderers-renders) | 提供呈現頁面(通常為發AEM布實例)的伺服器。 |
| [/篩選器](#configuring-access-to-content-filter) | 定義Dispatcher啟用訪問的URL。 |
| [/vanity_urls](#enabling-access-to-vanity-urls-vanity-urls) | 配置對虛擬URL的訪問。 |
| [/propagateSyndPost](#forwarding-syndication-requests-propagatesyndpost) | 支援轉發聯合請求。 |
| [/快取](#configuring-the-dispatcher-cache-cache) | 配置快取行為。 |
| [/統計資訊](#configuring-load-balancing-statistics) | 定義負載平衡計算的統計類別。 |
| [/stickyConnectionsFor](#identifying-a-sticky-connection-folder-stickyconnectionsfor) | 包含粘滯文檔的資料夾。 |
| [/health_check](#specifying-a-health-check-page) | 用於確定伺服器可用性的URL。 |
| [/retryDelay](#specifying-the-page-retry-delay) | 重試失敗連接之前的延遲。 |
| [/unavailable懲罰](#reflecting-server-unavailability-in-dispatcher-statistics) | 影響負載平衡計算統計的懲罰。 |
| [/failover](#using-the-failover-mechanism) | 當原始請求失敗時，將請求重新發送到不同的呈現。 |
| [/auth_checker](permissions-cache.md) | 有關權限敏感的快取，請參見 [快取安全內容](permissions-cache.md)。 |

## 指定預設頁（僅限IIS） — /homepage {#specify-a-default-page-iis-only-homepage}

>[!CAUTION]
>
>的 `/homepage`參數（僅限IIS）不再工作。 相反，您應使用 [IIS URL重寫模組](https://docs.microsoft.com/en-us/iis/extensions/url-rewrite-module/using-the-url-rewrite-module)。
>
>如果使用Apache，應使用 `mod_rewrite` 中。 有關資訊，請參閱Apache網站文檔 `mod_rewrite` (例如， [Apache 2.4](https://httpd.apache.org/docs/current/mod/mod_rewrite.html))。 使用時 `mod_rewrite`，建議使用該標誌 **[「passthrough|PT」（傳遞到下一個處理程式）](https://helpx.adobe.com/dispatcher/kb/DispatcherModReWrite.html)** 強制重寫引擎設定 `uri` 內部 `request_rec` 結構到 `filename` 的子菜單。

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

## 指定要傳遞的HTTP標頭 {#specifying-the-http-headers-to-pass-through-clientheaders}

的 `/clientheaders` 屬性定義Dispatcher從客戶端HTTP請求傳遞給呈現器（實例）的HTTP標AEM頭清單。

預設情況下，Dispatcher將標準HTTP標頭轉發AEM到實例。 在某些情況下，您可能希望轉發其他標頭或刪除特定標頭：

* 添加實例在HTTP請求中需AEM要的標頭（如自定義標頭）。
* 刪除僅與Web伺服器相關的標頭（如驗證標頭）。

如果自定義要傳遞的標頭集，則必須指定詳盡的標頭清單，包括通常預設包含的標頭。

例如，處理發佈實例的頁面激活請求的Dispatcher實例需要 `PATH` 標題 `/clientheaders` 的子菜單。 的 `PATH` 標頭允許複製代理和調度程式之間通信。

以下代碼是 `/clientheaders`:

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
  "if-match"
  "if-none-match"
  "if-range"
  "if-unmodified-since"
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

## 標識虛擬主機 {#identifying-virtual-hosts-virtualhosts}

的 `/virtualhosts` 屬性定義Dispatcher為此場接受的所有主機名/URI組合的清單。 您可以使用星號(`*`)字元作為通配符。 /的值 `virtualhosts` 屬性使用以下格式：

```xml
[scheme]host[uri][*]
```

* `scheme`:（可選） `https://` 或 `https://.`
* `host`:主機的名稱或IP地址，以及埠號（如果需要）。 (請參閱 [https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.23](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.23))
* `uri`:（可選）資源路徑。

以下示例配置處理myCompany的.com和.ch域以及mySubDivision的所有域的請求：

```xml
   /virtualhosts
    {
    "www.myCompany.com"
    "www.myCompany.ch"
    "www.mySubDivison.*"
    }
```

以下配置句柄 *全部* 請求：

```xml
   /virtualhosts
    {
    "*"
    }
```

### 解析虛擬主機 {#resolving-the-virtual-host}

當Dispatcher收到HTTP或HTTPS請求時，它會找到與 `host,` `uri`, `scheme` 請求的標題。 Dispatcher計算 `virtualhosts` 屬性：

* Dispatcher從最低的場開始，在dispatcher.any檔案中向上推進。
* 對於每個伺服器場， Dispatcher以中的最頂值開頭 `virtualhosts` 屬性，並向下推進值清單。

Dispatcher通過以下方式查找最佳匹配的虛擬主機值：

* 第一個遇到的與所有三個 `host`，也請參見Wiki頁。 `scheme`的 `uri` 的下界。
* 否 `virtualhosts` 值 `scheme` 和 `uri` 兩者都匹配的部件 `scheme` 和 `uri` 請求的第一個與 `host` 的下界。
* 否 `virtualhosts` 值的主機部分與請求的主機匹配，使用最頂層場的最頂層虛擬主機。

因此，您應將預設虛擬主機放在 `virtualhosts` 你最頂端農場的房產 `dispatcher.any` 的子菜單。

### 虛擬主機解析示例 {#example-virtual-host-resolution}

以下示例表示來自 `dispatcher.any` 定義兩個Dispatcher場的檔案，每個場定義一個 `virtualhosts` 屬性。

```xml
/farms
  {
  /myProducts
    {
    /virtualhosts
      {
      "www.mycompany.com"
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
      "www.mycompany.com/products/*"
      }
    /renders
      {
      /hostname "server2.myCompany.com"
      /port "80"
      }
    }
  }
```

使用此示例，下表顯示了為給定HTTP請求解析的虛擬主機：

| 請求URL | 已解析的虛擬主機 |
|---|---|
| `https://www.mycompany.com/products/gloves.html` | `www.mycompany.com/products/` |
| `https://www.mycompany.com/about.html` | `www.mycompany.com` |

## 啟用安全會話 — /會話管理 {#enabling-secure-sessions-sessionmanagement}

>[!CAUTION]
>
>`/allowAuthorized` **必須** 設定 `"0"` 的 `/cache` 的子菜單。

建立安全會話以訪問呈現場，以便用戶需要登錄才能訪問場中的任何頁。 登錄後，用戶可以訪問伺服器場中的頁面。 請參閱 [建立關閉的用戶組](https://experienceleague.adobe.com/docs/experience-manager-65/administering/security/cug.html?lang=en#creating-the-user-group-to-be-used) 有關將此功能與CUG一起使用的資訊。 另請參見Dispatcher [安全核對表](/help/using/security-checklist.md) 才能活下去。

的 `/sessionmanagement` 屬性是的子屬性 `/farms`。

>[!CAUTION]
>
>如果網站的各個部分使用不同的訪問要求，則需要定義多個場。

**/會話管理** 有幾個子參數：

**/目錄** （強制）

儲存會話資訊的目錄。 如果目錄不存在，則建立該目錄。

>[!CAUTION]
>
> 配置目錄子參數時 **不** 指向根資料夾(`/directory "/"`)，因為它會造成嚴重問題。 應始終指定儲存會話資訊的資料夾的路徑。 例如：

```xml
/sessionmanagement
  {
  /directory "/usr/local/apache/.sessions"
  }
```

**/encode** （可選）

如何編碼會話資訊。 使用 `md5` 使用md5算法進行加密，或 `hex` 十六進位編碼。 如果加密會話資料，則具有檔案系統訪問權限的用戶無法讀取會話內容。 預設值為 `md5`。

**/header** （可選）

儲存授權資訊的HTTP標頭或Cookie的名稱。 如果將資訊儲存在http標頭中，請使用 `HTTP:<header-name>`。 要將資訊儲存在Cookie中，請使用 `Cookie:<header-name>`。 如果未指定值 `HTTP:authorization` 的子菜單。

**/timeout** （可選）

上次使用會話後會話超時的秒數。 如果未指定 `"800"` ，因此會話在用戶上次請求後13分鐘多一點超時。

配置示例如下所示：

```xml
/sessionmanagement
  {
  /directory "/usr/local/apache/.sessions"
  /encode "md5"
  /header "HTTP:authorization"
  /timeout "800"
  }
```

## 定義頁面呈現器 {#defining-page-renderers-renders}

/renders屬性定義Dispatcher向其發送請求以呈現文檔的URL。 以下示例 `/renders` 部分標識用於呈AEM現的單個實例：

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

以下示例/renders節標識在與調度程式AEM相同的電腦上運行的實例：

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

以下示例/renders節在兩個實例之間平均分配呈現AEM請求：

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

### 呈現選項 {#renders-options}

**/timeout**

指定訪問實例的連AEM接超時（毫秒）。 預設值為 `"0"`，導致調度程式無限期等待。

**/receiveTimeout**

指定允許響應所花費的時間（以毫秒為單位）。 預設值為 `"600000"`，導致Dispatcher等待10分鐘。 設定 `"0"` 完全消除超時。

如果分析響應標頭時達到超時，則返回HTTP狀態504（錯誤網關）。 如果在讀取響應正文時達到超時，則Dispatcher將將不完整的響應返回給客戶端，但刪除可能已寫入的任何快取檔案。

**/ipv4**

指定Dispatcher是否使用 `getaddrinfo` 函式（對於IPv6）或 `gethostbyname` 函式（對於IPv4），以獲取呈現器的IP地址。 0的原因值 `getaddrinfo` 的下界。 值 `1` 原因 `gethostbyname` 的下界。 預設值為 `0`.

的 `getaddrinfo` 函式返回IP地址清單。 Dispatcher迭代地址清單，直到它建立TCP/IP連接。 因此， `ipv4` 當render主機名與多個IP地址和主機關聯時，屬性非常重要， `getaddrinfo` 函式，返回始終按相同順序排列的IP地址清單。 在這種情況下，您應使用 `gethostbyname` 使Dispatcher連接的IP地址是隨機的。

Amazon彈性負載平衡(ELB)是一種服務，它以可能相同順序的IP地址清單來響應getaddrinfo。

**/安全**

如果 `/secure` 屬性值為 `"1"` Dispatcher使用HTTPS與實例AEM通信。 有關其他詳細資訊，另請參閱 [將Dispatcher配置為使用SSL](dispatcher-ssl.md#configuring-dispatcher-to-use-ssl)。

**/always resolve**

使用Dispatcher版本 **4.1.6**，您可以配置 `/always-resolve` 如下：

* 設定為時 `"1"` 它將解析每個請求的主機名（Dispatcher永遠不會快取任何IP地址）。 由於獲取每個請求的主機資訊所需的額外呼叫，可能會對效能產生輕微影響。
* 如果未設定屬性，則預設情況下將快取IP地址。

此外，在遇到動態IP解決問題時，可使用此屬性，如以下示例所示：

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

## 配置對內容的訪問 {#configuring-access-to-content-filter}

使用 `/filter` 指定Dispatcher接受的HTTP請求。 所有其它請求都會以404錯誤代碼（找不到頁面）發回Web伺服器。 否 `/filter` 部分存在，所有請求都被接受。

**注：** 請求 [statfile](#naming-the-statfile) 總是被拒絕。

>[!CAUTION]
>
>查看 [調度程式安全核對表](security-checklist.md) 以瞭解限制使用Dispatcher的訪問時的進一步注意事項。 另外，閱讀 [安AEM全核對表](https://experienceleague.adobe.com/docs/experience-manager-65/administering/security/security-checklist.html?lang=en#security) 有關安裝的其他安全詳AEM細資訊。

的 `/filter` 部分由一系列規則組成，這些規則根據HTTP請求請求請求行部分的模式拒絕或允許訪問內容。 您應使用允許清單策略 `/filter` 部分：

* 首先，拒絕訪問所有內容。
* 允許根據需要訪問內容。

>[!NOTE]
>
>建議在篩選規則發生任何更改時清除快取。

### 定義篩選器 {#defining-a-filter}

中的每個項 `/filter` 部分包括與請求行或整個請求行的特定元素匹配的類型和模式。 每個篩選器可包含以下項：

* **類型**:的 `/type` 指示是否允許或拒絕對與模式匹配的請求的訪問。 值可以是 `allow` 或 `deny`。

* **請求行的要素：** 包括 `/method`。 `/url`。 `/query`或 `/protocol` 以及根據HTTP請求的請求行部分的這些特定部分過濾請求的模式。 在請求行的元素（而不是整個請求行）上過濾是首選的過濾方法。

* **請求行的高級要素：** 從Dispatcher 4.2.0開始，有四個新的篩選器元素可供使用。 這些新元素 `/path`。 `/selectors`。 `/extension` 和 `/suffix` 分別進行。 包括一個或多個這些項目以進一步控制URL模式。

>[!NOTE]
>
>有關請求行中每個元素所參照的部分的詳細資訊，請參見 [Sling URL分解](https://sling.apache.org/documentation/the-sling-engine/url-decomposition.html) 維客頁面。

* **格洛布財產**:的 `/glob` 屬性用於與HTTP請求的整個請求行匹配。

>[!CAUTION]
>
>在Dispatcher中，不建議使用glob篩選。 因此，應避免在 `/filter` 因為可能會導致安全問題。 所以，不是：
>
>`/glob "* *.css *"`
>
>你應該
>
>`/url "*.css"`

#### HTTP請求的請求行部分 {#the-request-line-part-of-http-requests}

HTTP/1.1定義 [請求行](https://www.w3.org/Protocols/rfc2616/rfc2616-sec5.html) 如下：

`Method Request-URI HTTP-Version<CRLF>`

的 `<CRLF>` 字元表示回車符，後跟換行符。 以下示例是當客戶端請求WKND站點的「美文」頁時收到的請求行：

`GET /content/wknd/us/en.html HTTP.1.1<CRLF>`

您的模式必須考慮請求行中的空格字元和 `<CRLF>` 字元。

#### 雙引號與單引號 {#double-quotes-vs-single-quotes}

建立篩選器規則時，請使用雙引號 `"pattern"` 簡單圖案。 如果使用Dispatcher 4.2.0或更高版本，並且您的模式包含規則運算式，則必須將regex模式括起來 `'(pattern1|pattern2)'` 在單引號內。

#### 規則運算式 {#regular-expressions}

在4.2.0以後的Dispatcher版本中，可以在篩選器模式中包括POSIX擴展規則運算式。

#### 疑難解答篩選器 {#troubleshooting-filters}

如果篩選器未按預期方式觸發，請啟用 [跟蹤記錄](#trace-logging) 在dispatcher上，這樣您就可以看到哪個篩選器正在攔截請求。

#### 示例篩選器：全部拒絕 {#example-filter-deny-all}

以下示例篩選器部分使Dispatcher拒絕所有檔案的請求。 您應拒絕訪問所有檔案，然後允許訪問特定區域。

```xml
  /0001  { /glob "*" /type "deny" }
```

對明確拒絕區域的請求將導致返回404錯誤代碼（找不到頁面）。

#### 示例篩選器：拒絕訪問特定區域 {#example-filter-deny-access-to-specific-areas}

篩選器還允許您拒絕訪問發佈實例中的各種元素，例如ASP頁和敏感區域。 以下篩選器拒絕訪問ASP頁：

```xml
/0002  { /type "deny" /url "*.asp"  }
```

#### 示例篩選器：啟用POST請求 {#example-filter-enable-post-requests}

下面的示例篩選器允許通過POST方法提交表單資料：

```xml
/filter {
    /0001  { /glob "*" /type "deny" }
    /0002 { /type "allow" /method "POST" /url "/content/[.]*.form.html" }
}
```

#### 示例篩選器：允許訪問工作流控制台 {#example-filter-allow-access-to-the-workflow-console}

以下示例顯示了用於拒絕對工作流控制台的外部訪問的篩選器：

```xml
/filter {
    /0001  { /glob "*" /type "deny" }
    /0002  {  /type "allow"  /url "/libs/cq/workflow/content/console*"  }
}
```

如果發佈實例使用Web應用程式上下文（例如發佈），也可以將此內容添加到篩選器定義中。

```xml
/0003   { /type "deny"  /url "/publish/libs/cq/workflow/content/console/archive*"  }
```

如果您仍需要訪問受限區域內的單個頁面，則可以允許訪問這些頁面。 例如，要允許訪問工作流控制台中的「存檔」頁籤，請添加以下部分：

```xml
/0004  { /type "allow"  /url "/libs/cq/workflow/content/console/archive*"   }
```

>[!NOTE]
>
>當多個篩選器模式應用於請求時，應用的最後一個篩選器模式是有效的。

#### 示例篩選器：使用規則運算式 {#example-filter-using-regular-expressions}

此篩選器使用規則運算式在非公共內容目錄中啟用擴展，該表達式在單引號之間定義：

```xml
/005  {  /type "allow" /extension '(css|gif|ico|js|png|swf|jpe?g)' }
```

#### 示例篩選器：篩選請求URL的附加元素 {#example-filter-filter-additional-elements-of-a-request-url}

下面是阻止內容從 `/content` 路徑及其子樹，使用路徑、選擇器和擴展的篩選器：

```xml
/006 {
        /type "deny"
        /path "/content/*"
        /selectors '(feed|rss|pages|languages|blueprint|infinity|tidy)'
        /extension '(json|xml|html)'
        }
```

### 示例/filter節 {#example-filter-section}

配置Dispatcher時，應盡可能限制外部訪問。 以下示例為外部訪問者提供了最小的訪問權限：

* `/content`
* 設計、客戶端庫等雜項內容；例如：

   * `/etc/designs/default*`
   * `/etc/designs/mydesign*`

建立篩選器後， [test頁訪問](#testing-dispatcher-security) 確保實AEM例安全。

以下 `/filter` 的下界 `dispatcher.any` 檔案可用作 [調度程式配置檔案。](#dispatcher-configuration-files)

此示例基於隨Dispatcher提供的預設配置檔案，並打算作為示例在生產環境中使用。 以前置詞的項目 `#` 停用（注釋掉），如果您決定激活其中任何一項(通過刪除 `#` 在該線上)，因為這會對安全產生影響。

您應拒絕訪問所有內容，然後允許訪問特定（受限）元素：

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
      /0001 { /type "deny" /glob "*" }

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
>與Apache一起使用時，根據Dispatcher模組的DispatcherUseProcessedURL屬性設計篩選器URL模式。 (請參閱 [Apache Web Server — 為Dispatcher配置Apache Web Server](dispatcher-install.md##apache-web-server-configure-apache-web-server-for-dispatcher)。)

>[!NOTE]
>
>篩選器 `0030` 和 `0031` Dynamic Media適AEM用於6.0及更高版本。

如果您確實選擇擴展訪問權限，請考慮以下建議：

* 外部訪問 `/admin` 應該永遠 *完全* 已禁用。

* 允許訪問中的檔案時必須小心 `/libs`。 應允許個人訪問。
* 拒絕對複製配置的訪問，因此無法看到：

   * `/etc/replication.xml*`
   * `/etc/replication.infinity.json*`

* 拒絕訪問Google小工具反向代理：

   * `/libs/opensocial/proxy*`

根據您的安裝，可能會有其他資源 `/libs`。 `/apps` 或者別的地方，必須提供。 您可以使用 `access.log` 檔案，作為確定正在外部訪問的資源的一種方法。

>[!CAUTION]
>
>對控制台和目錄的訪問可能會給生產環境帶來安全風險。 除非您有明確的理由，否則應保持停用（注釋掉）。

>[!CAUTION]
>
>如果 [在發佈環境中使用報表](https://experienceleague.adobe.com/docs/experience-manager-65/administering/operations/reporting.html?lang=en#using-reports-in-a-publish-environment) 您應將Dispatcher配置為拒絕訪問 `/etc/reports` 外部訪問者。

### 限制查詢字串 {#restricting-query-strings}

自Dispatcher版本4.1.5起，請使用 `/filter` 部分以限制查詢字串。 強烈建議通過顯式允許查詢字串和排除泛型允許 `allow` 篩選元素。

單個條目可以 `glob` 或者某些組合 `method`。 `url`。 `query`, `version`但不是兩者。 下面的示例允許 `a=*` 查詢字串，並拒絕解析為 `/etc` 節點：

```xml
/filter {
 /0001 { /type "deny" /method "POST" /url "/etc/*" }
 /0002 { /type "allow" /method "GET" /url "/etc/*" /query "a=*" }
}
```

>[!NOTE]
>
>如果規則包含 `/query`，它將僅匹配包含查詢字串的請求，並匹配提供的查詢模式。
>
>在上例中，如果請求 `/etc` 不允許使用查詢字串，則需要以下規則：

```xml
/filter {  
>/0001 { /type "deny" /method “*" /url "/path/*" }  
>/0002 { /type "allow" /method "GET" /url "/path/*" }  
>/0003 { /type “deny" /method "GET" /url "/path/*" /query "*" }  
>/0004 { /type "allow" /method "GET" /url "/path/*" /query "a=*" }  
}  
```

### 測試Dispatcher安全性 {#testing-dispatcher-security}

Dispatcher篩選器應阻止對發佈實例上的以下頁和腳AEM本的訪問。 使用Web瀏覽器嘗試開啟以下頁面，因為站點訪問者會開啟並驗證是否返回代碼403。 如果獲得了任何其他結果，請調整濾鏡。

請注意，您應看到的普通頁面呈現 `/content/add_valid_page.html?debug=layout`。

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

在終端或命令提示符下發出以下命令，以確定是否啟用了匿名寫訪問。 您不應該將資料寫入節點。

`curl -X POST "https://anonymous:anonymous@hostname:port/content/usergenerated/mytestnode"`

在終端或命令提示符下發出以下命令，嘗試使Dispatcher快取失效，並確保收到代碼404響應：

`curl -H "CQ-Handle: /content" -H "CQ-Path: /content" https://yourhostname/dispatcher/invalidate.cache`

## 啟用對虛榮URL的訪問 {#enabling-access-to-vanity-urls-vanity-urls}

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (sbroders@adobe.com)
Last Modified Date: 2015-03-25T14:23:05.185-0400

<p style="font-family: tahoma, arial, helvetica, sans-serif; font-size: 12px;">For https://jira.corp.adobe.com/browse/DOC-4812</p> 
<p style="font-family: tahoma, arial, helvetica, sans-serif; font-size: 12px;">The "com.adobe.granite.dispatcher.vanityurl.content" package needs to be made public before publishing this contnet.</p>
 -->

配置Dispatcher以啟用對為您的頁面配置的虛AEM擬URL的訪問。

啟用對虛擬URL的訪問後，Dispatcher會定期調用在呈現實例上運行的服務，以獲取虛擬URL清單。 調度程式將此清單儲存在本地檔案中。 當頁面請求因 `/filter` 的子目錄中的子目錄。 如果清單中包含拒絕的URL，則Dispatcher允許訪問虛擬URL。

要啟用對虛擬URL的訪問，請添加 `/vanity_urls` 的 `/farms` 部分，與以下示例類似：

```xml
 /vanity_urls {
      /url "/libs/granite/dispatcher/content/vanityUrls.html"
      /file "/tmp/vanity_urls"
      /delay 300
 }
```

的 `/vanity_urls` 部分包含以下屬性：

* `/url`:在呈現實例上運行的虛擬URL服務的路徑。 此屬性的值必須為 `"/libs/granite/dispatcher/content/vanityUrls.html"`。

* `/file`:Dispatcher儲存虛擬URL清單的本地檔案的路徑。 確保Dispatcher對此檔案具有寫訪問權限。
* `/delay`:（秒）調用虛擬URL服務之間的時間。

>[!NOTE]
>
>如果呈現為實例，AEM則必須安裝 [軟體分發中的VanityURLS-Components包](https://experience.adobe.com/#/downloads/content/software-distribution/en/aem.html?package=/content/software-distribution/en/details.html/content/dam/aem/public/adobe/packages/granite/vanityurls-components) 啟用虛榮URL服務。 (請參閱 [軟體分發](https://experienceleague.adobe.com/docs/experience-manager-65/administering/contentmanagement/package-manager.html?lang=en#software-distribution) )

請按下列步驟啟用對虛擬URL的訪問。

1. 如果您的呈現服務是實AEM例，請安裝 `com.adobe.granite.dispatcher.vanityurl.content` 檔案包（請參閱上面的說明）。
1. 對於為或CQ頁配置的AEM每個虛擬URL，請確保 [`/filter`](#configuring-access-to-content-filter) 配置拒絕URL。 如有必要，請添加拒絕URL的篩選器。
1. 添加 `/vanity_urls` 下面 `/farms`。
1. 重新啟動Apache Web伺服器。

## 轉發聯合請求 — /propagateSyndPost {#forwarding-syndication-requests-propagatesyndpost}

聯合請求通常僅用於Dispatcher，因此預設情況下不會將它們發送到呈現器(例如AEM實例)。

如有必要，請設定 `/propagateSyndPost` 屬性 `"1"` 將聯合請求轉發到Dispatcher。 如果設定，則必須確保篩選器部分不拒絕POST請求。

## 配置Dispatcher快取 — /cache {#configuring-the-dispatcher-cache-cache}

的 `/cache` 節控制Dispatcher如何快取文檔。 配置幾個子屬性以實施快取策略：

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

快取節示例如下所示：

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
>對於權限敏感快取，請讀取 [快取安全內容](permissions-cache.md)。

### 指定快取目錄 {#specifying-the-cache-directory}

的 `/docroot` 屬性標識儲存快取檔案的目錄。

>[!NOTE]
>
>該值必須與Web伺服器的文檔根路徑完全相同，以便Dispatcher和Web伺服器處理相同的檔案。\
>Web伺服器負責在使用調度程式快取檔案時提供正確的狀態代碼，因此它也能找到它也很重要。

如果使用多個場，則每個場必須使用不同的文檔根。

### 命名Statfile {#naming-the-statfile}

的 `/statfile` 屬性標識要用作statfile的檔案。 Dispatcher使用此檔案註冊最近內容更新的時間。 statfile可以是Web伺服器上的任何檔案。

statfile沒有內容。 更新內容後， Dispatcher將更新時間戳。 預設的statfile名為 `.stat` 並儲存在docroot中。 調度程式阻止對statfile的訪問。

>[!NOTE]
>
>如果 `/statfileslevel` 已配置，但Dispatcher忽略 `/statfile` 屬性和使用 `.stat` 的下界。

### 出錯時提供過時文檔 {#serving-stale-documents-when-errors-occur}

的 `/serveStaleOnError` 屬性控制呈現伺服器返回錯誤時Dispatcher是否返回無效文檔。 預設情況下，當觸碰靜態檔案並使快取內容無效時，Dispatcher會在下次請求快取內容時刪除它。

如果 `/serveStaleOnError` 設定為 `"1"`，除非呈現伺服器返回成功響應，否則Dispatcher不會從快取中刪除無效內容。 來自或連接超時AEM的5xx響應導致Dispatcher為過時內容提供服務，並使用HTTP狀態111（重新驗證失敗）進行響應。

### 使用身份驗證時快取 {#caching-when-authentication-is-used}

的 `/allowAuthorized` 屬性控制是否快取包含以下任何驗證資訊的請求：

* 的 `authorization` 標題
* 名為 `authorization`
* 名為 `login-token`

預設情況下，不快取包含此驗證資訊的請求，因為當快取文檔返回給客戶端時不執行驗證。 此配置可阻止Dispatcher向沒有必要權限的用戶提供快取的文檔。

但是，如果您的要求允許快取已驗證的文檔，請設定 `/allowAuthorized` 至一：

`/allowAuthorized "1"`

>[!NOTE]
>
>啟用會話管理(使用 `/sessionmanagement` 屬性), `/allowAuthorized` 屬性必須設定為 `"0"`。

### 指定要快取的文檔 {#specifying-the-documents-to-cache}

的 `/rules` 屬性控制根據文檔路徑快取哪些文檔。 無論 `/rules` 屬性，Dispatcher從不在以下情況下快取文檔：

* 如果請求URI包含問號(`?`)。
   * 這通常表示動態頁，例如不需要快取的搜索結果。
* 缺少檔案副檔名。
   * Web 伺服器需要副檔名來判斷文件類型 (MIME 類型)。
* 驗證標頭已設定 (這可以設定)。
* 如果實AEM例使用以下標題進行響應：

   * `no-cache`
   * `no-store`
   * `must-revalidate`

>[!NOTE]
>
>GET 或 HEAD (用於 HTTP 標頭) 方法可讓 Dispatcher 快取。有關響應頭快取的其他資訊，請參見 [快取HTTP響應標頭](#caching-http-response-headers) 的子菜單。

中的每個項 `/rules` 屬性包括 [`glob`](#designing-patterns-for-glob-properties) 模式和類型：

* 的 `glob` 模式用於匹配文檔的路徑。
* 類型指示是否快取與 `glob` 的上界。 該值可以是允許（快取文檔）或拒絕（始終呈現文檔）。

如果沒有動態頁（超出上述規則已排除的頁），則可以配置Dispatcher以快取所有內容。 此項的規則部分如下所示：

```xml
/rules
  {
    /0000  {  /glob "*"   /type "allow" }
  }
```

有關全局屬性的資訊，請參見 [設計全局屬性的模式](#designing-patterns-for-glob-properties)。

如果頁面中有一些部分是動態的（例如，新聞應用程式）或在關閉的用戶組內，則可以定義例外：

>[!NOTE]
>
>不得快取關閉的用戶組，因為沒有為快取的頁面檢查用戶權限。

```xml
/rules
  {
   /0000  { /glob "*" /type "allow" }
   /0001  { /glob "/en/news/*" /type "deny" }
   /0002  { /glob "*/private/*" /type "deny"  }
  }
```

**壓縮**

在Apache Web伺服器上，可以壓縮快取的文檔。 壓縮允許Apache在客戶端請求時以壓縮形式返回文檔。 通過啟用Apache模組自動執行壓縮 `mod_deflate`，例如：

```xml
AddOutputFilterByType DEFLATE text/plain
```

預設情況下，Apache 2.x將安裝該模組。

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

### 按資料夾級別取消驗證檔案 {#invalidating-files-by-folder-level}

使用 `/statfileslevel` 根據快取檔案的路徑使其失效的屬性：

* 調度程式建立 `.stat`從docroot資料夾到您指定的級別的每個資料夾中的檔案。 docroot資料夾為0級。
* 通過觸碰 `.stat` 的子菜單。 的 `.stat` 將檔案的上次修改日期與快取文檔的上次修改日期進行比較。 如果 `.stat` 檔案更新。

* 如果某個級別的檔案無效，則 **全部** `.stat` docroot的檔案 **至** 無效檔案或已配置的 `statsfilevel` （以較小者為準）將被觸碰。

   * 例如：的 `statfileslevel` 屬性為6，檔案在5級後每 `.stat` 將觸碰docroot到5的檔案。 繼續此示例，如果檔案在7級失效，則每次。 `stat` docroot到6的檔案將被觸碰(因為 `/statfileslevel = "6"`)。

僅資源 **沿著小路** 影響無效檔案。 請考慮以下示例：網站使用結構 `/content/myWebsite/xx/.` 如果設定 `statfileslevel` 作為3,a `.stat`檔案建立方式如下：

* `docroot`
* `/content`
* `/content/myWebsite`
* `/content/myWebsite/*xx*`

檔案在 `/content/myWebsite/xx` 失效後 `.stat` 檔案從docroot到 `/content/myWebsite/xx`被觸碰。 這隻適用於 `/content/myWebsite/xx` 不是 `/content/myWebsite/yy` 或 `/content/anotherWebSite`。

>[!NOTE]
>
>通過發送附加報頭可以防止無效 `CQ-Action-Scope:ResourceOnly`。 這可用於刷新特定資源而不使快取的其他部分失效。 請參閱 [此頁](https://adobe-consulting-services.github.io/acs-aem-commons/features/dispatcher-flush-rules/index.html) 和 [手動使Dispatcher快取無效](https://experienceleague.adobe.com/docs/experience-manager-dispatcher/using/configuring/page-invalidate.html?lang=en#configuring) 的雙曲餘切值。

>[!NOTE]
>
>如果為 `/statfileslevel` 屬性， `/statfile` 屬性被忽略。

### 自動使快取的檔案無效 {#automatically-invalidating-cached-files}

的 `/invalidate` 屬性定義內容更新時自動失效的文檔。

如果自動失效，則Dispatcher不會在內容更新後刪除快取檔案，但會在下次請求時檢查其有效性。 快取中未自動失效的文檔將保留在快取中，直到內容更新明確刪除它們。

自動失效通常用於HTML頁。 HTML頁面通常包含指向其他頁面的連結，因此很難確定內容更新是否影響頁面。 要確保在更新內容時所有相關頁面都無效，請自動使所有HTML頁面失效。 以下配置使所有HTML頁失效：

```xml
  /invalidate
  {
   /0000  { /glob "*" /type "deny" }
   /0001  { /glob "*.html" /type "allow" }
  }
```

有關全局屬性的資訊，請參見 [設計全局屬性的模式](#designing-patterns-for-glob-properties)。

此配置在 `/content/wknd/us/en` 已激活：

* 所有模式為en的檔案。*從 `/content/wknd/us` 的子菜單。
* 的 `/content/wknd/us/en./_jcr_content` 資料夾。
* 所有與 `/invalidate` 未立即刪除配置。 下次請求時，這些檔案將被刪除。 在我們的例子中 `/content/wknd.html` 未刪除，在 `/content/wknd.html` 請求。

如果您提供自動生成的PDF和ZIP檔案供下載，則可能還必須自動使這些檔案失效。 配置示例如下所示：

```xml
/invalidate
  {
   /0000 { /glob "*" /type "deny" }
   /0001 { /glob "*.html" /type "allow" }
   /0002 { /glob "*.zip" /type "allow" }
   /0003 { /glob "*.pdf" /type "allow" }
  }
```

與AEMAdobe Analytics的整合 `analytics.sitecatalyst.js` 檔案。 示例 `dispatcher.any` 隨Dispatcher提供的檔案包含此檔案的以下無效規則：

```xml
{
   /glob "*/analytics.sitecatalyst.js"  /type "allow"
}
```

### 使用自定義失效指令碼 {#using-custom-invalidation-scripts}

的 `/invalidateHandler` 屬性允許您定義指令碼，該指令碼針對Dispatcher接收的每個無效請求調用。

調用時使用以下參數：

* Handle — 無效的內容路徑
* 操作 — 複製操作（例如激活、停用）
* 操作範圍 — 複製操作的範圍(空，除非 `CQ-Action-Scope: ResourceOnly` 已發送，請參閱 [從中取消驗證快取頁AEM面](page-invalidate.md) 有關詳細資訊)

這可用於涵蓋許多不同的使用情形，如使其他應用程式特定的快取無效，或處理頁面的外部化URL及其在域中的位置與內容路徑不匹配的情況。

下面的示例指令碼記錄每個無效請求到檔案。

```xml
/invalidateHandler "/opt/dispatcher/scripts/invalidate.sh"
```

#### 示例無效處理程式指令碼 {#sample-invalidation-handler-script}

```shell
#!/bin/bash

printf "%-15s: %s %s" $1 $2 $3>> /opt/dispatcher/logs/invalidate.log
```

### 限制可刷新快取的客戶端 {#limiting-the-clients-that-can-flush-the-cache}

的 `/allowedClients` 屬性定義允許刷新快取的特定客戶端。 所述環形模式與所述IP匹配。

以下示例：

1. 拒絕訪問任何客戶端
1. 明確允許訪問localhost

```xml
/allowedClients
  {
   /0001 { /glob "*.*.*.*"  /type "deny" }
   /0002 { /glob "127.0.0.1" /type "allow" }
  }
```

有關全局屬性的資訊，請參見 [設計全局屬性的模式](#designing-patterns-for-glob-properties)。

>[!CAUTION]
>
>建議您定義 `/allowedClients`。
>
>如果不執行此操作，任何客戶端都可發出調用以清除快取；如果反複執行此操作，可能會嚴重影響站點效能。

### 忽略URL參數 {#ignoring-url-parameters}

的 `ignoreUrlParams` 部分定義在確定是快取頁面還是從快取中傳遞頁面時忽略哪些URL參數：

* 當請求URL包含所有忽略的參數時，將快取該頁。
* 當請求URL包含一個或多個未忽略的參數時，不快取該頁。

當忽略某頁的參數時，在首次請求該頁時快取該頁。 對該頁的後續請求被提供給快取的頁，而不管該請求中的參數的值如何。

要指定忽略哪些參數，請將全局規則添加到 `ignoreUrlParams` 屬性：

* 要忽略參數，請建立允許該參數的glob屬性。
* 要防止對頁面進行快取，請建立一個拒絕該參數的glob屬性。

以下示例導致Dispatcher忽略 `q` 參數，以便快取包含q參數的請求URL:

```xml
/ignoreUrlParams
{
    /0001 { /glob "*" /type "deny" }
    /0002 { /glob "q" /type "allow" }
}
```

使用示例 `ignoreUrlParams` 值，以下HTTP請求導致頁面被快取，因為 `q` 參數被忽略：

```xml
GET /mypage.html?q=5
```

使用示例 `ignoreUrlParams` 值，以下HTTP請求將頁面 **不** 因為 `p` 未忽略參數：

```xml
GET /mypage.html?q=5&p=4
```

有關全局屬性的資訊，請參見 [設計全局屬性的模式](#designing-patterns-for-glob-properties)。

### 快取HTTP響應標頭 {#caching-http-response-headers}

>[!NOTE]
>
>此功能可隨版本一起使用 **4.1.11** 調度員。

的 `/headers` 屬性允許您定義Dispatcher將要快取的HTTP標頭類型。 在對未快取資源的第一個請求中，與配置值之一匹配的所有標頭（請參閱下面的配置示例）都儲存在快取檔案旁邊的單獨檔案中。 在對快取資源的後續請求中，儲存的報頭被添加到響應中。

下面是預設配置的示例：

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
>另請注意，不允許使用檔案全局字元。 有關詳細資訊，請參閱 [設計全局屬性的模式](#designing-patterns-for-glob-properties)。

>[!NOTE]
>
>如果需要Dispatcher儲存和傳遞來自的ETag響應標頭AEM，請執行以下操作：
>
>* 在 `/cache/headers`的子菜單。
>* 添加以下內容 [Apache指令](https://httpd.apache.org/docs/2.4/mod/core.html#fileetag) 在「與Dispatcher相關」部分：
>
>```xml
>FileETag none
>```

### Dispatcher快取檔案權限 {#dispatcher-cache-file-permissions}

的 `mode` 屬性指定哪些檔案權限應用於快取中的新目錄和檔案。 此設定受 `umask` 調用過程。 它是由下列一個或多個值之和構成的八進位數：

* `0400` 允許所有者讀取。
* `0200` 允許由所有者寫入。
* `0100` 允許所有者在目錄中搜索。
* `0040` 允許按組成員讀取。
* `0020` 允許按組成員寫入。
* `0010` 允許組成員在目錄中搜索。
* `0004` 允許其他人閱讀。
* `0002` 允許其他人寫入。
* `0001` 允許其他人在目錄中搜索。

預設值為 `0755` 允許所有者讀取、寫入或搜索，組和其他人讀取或搜索。

### 限制.stat檔案觸摸 {#throttling-stat-file-touching}

使用預設值 `/invalidate` 屬性，每次激活都有效地使所有 `.html` 檔案(當其路徑與 `/invalidate` )的正平方根。 在流量可觀的網站上，多次後續激活會增加後端的cpu負載。 在這種情況下，最好&quot;節制&quot; `.stat` 檔案觸碰，使網站保持響應。 您可以使用 `/gracePeriod` 屬性。

的 `/gracePeriod` 屬性定義在上次激活後仍然可以從快取中提供失效的自動失效資源的秒數。 該屬性可用於設定中，否則一批激活將重複使整個快取無效。 建議的值為2秒。

有關其他詳細資訊，請閱讀 `/invalidate` 和 `/statfileslevel`的上界。

### 配置基於時間的快取無效 — /enableTTL {#configuring-time-based-cache-invalidation-enablettl}

如果設定， `/enableTTL` 屬性將評估來自後端的響應標頭，如果它們包含 `Cache-Control` 最大年齡或 `Expires` 日期，建立快取檔案旁邊的輔助空檔案，修改時間等於到期日期。 在修改時間之後請求快取檔案時，會自動從後端重新請求該檔案。

>[!NOTE]
>
>此功能在版本中可用 **4.1.11** 或更晚的調度員。

## 配置負載平衡 — /statistics {#configuring-load-balancing-statistics}

的 `/statistics` 部分定義Dispatcher對每個呈現的響應性進行評分的檔案類別。 Dispatcher使用分數來確定發送請求的呈現方式。

您建立的每個類別都定義全局模式。 Dispatcher將請求內容的URI與這些模式進行比較以確定請求內容的類別：

* 類別的順序確定它們與URI的比較順序。
* 與URI匹配的第一個類別模式是檔案的類別。 不再計算類別模式。

Dispatcher最多支援8個統計類別。 如果定義了8個以上的類別，則僅使用前8個類別。

**呈現選擇**

每次Dispatcher需要呈現頁時，它都會使用以下算法來選擇呈現：

1. 如果請求包含的呈現名稱 `renderid` Cookie,Dispatcher使用該呈現器。
1. 如果請求不包括 `renderid` Cookie,Dispatcher比較呈現統計資訊：

   1. 調度程式確定請求URI的類別。
   1. 調度程式確定哪個呈現具有該類別的最低響應得分，並選擇該呈現。

1. 如果尚未選擇渲染，請使用清單中的第一個渲染。

呈現類別的分數基於以前的響應時間，以及Dispatcher嘗試的以前失敗和成功的連接。 對於每次嘗試，都會更新請求URI類別的分數。

>[!NOTE]
>
>如果不使用負載平衡，則可以忽略此部分。

### 定義統計類別 {#defining-statistics-categories}

為要保留用於呈現選擇的統計資訊的每種類型的文檔定義類別。 的 `/statistics` 節包含 `/categories` 的子菜單。 要定義類別，請在 `/categories` 的子菜單。

`/name { /glob "pattern"}`

類別 `name` 必須是場所特有的。 的 `pattern` 在 [設計全局屬性的模式](#designing-patterns-for-glob-properties) 的子菜單。

要確定URI的類別，Dispatcher會將URI與每個類別模式進行比較，直到找到匹配項。 Dispatcher從清單中的第一個類別開始，並按順序繼續。 因此，首先將具有更具體模式的類別放置。

例如，Dispatcher是 `dispatcher.any` 檔案定義HTML類別和其他類別。 HTML類別更具體，因此它首先出現：

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

以下示例還包括搜索頁的類別：

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

### 反映調度器統計資訊中伺服器不可用性 {#reflecting-server-unavailability-in-dispatcher-statistics}

的 `/unavailablePenalty` 屬性設定在與呈現器的連接失敗時應用於呈現統計資訊的時間（以秒的十分之一為單位）。 Dispatcher將時間添加到與請求的URI匹配的統計類別。

例如，當無法建立到指定主機名/埠的TCP/IP連接時，會應用該懲罰，原因是AEM未運行（且未偵聽），或者是網路相關問題。

的 `/unavailablePenalty` 屬性是 `/farm` 節( `/statistics` )的正平方根。

否 `/unavailablePenalty` 屬性存在，值 `"1"` 的子菜單。

```xml
/unavailablePenalty "1"
```

## 標識粘滯連接資料夾 — /stickyConnectionsFor {#identifying-a-sticky-connection-folder-stickyconnectionsfor}

的 `/stickyConnectionsFor` 屬性定義一個包含粘滯文檔的資料夾；將使用URL訪問此內容。 Dispatcher將此資料夾中的所有請求從單個用戶發送到同一呈現實例。 粘滯連接可確保所有文檔的會話資料都存在且一致。 此機制使用 `renderid` 餅乾。

以下示例定義到/products資料夾的粘滯連接：

```xml
/stickyConnectionsFor "/products"
```

當頁面由來自多個內容節點的內容組成時，包括 `/paths` 列出內容路徑的屬性。 例如，頁面包含的內容來自 `/content/image`。 `/content/video`, `/var/files/pdfs`。 以下配置為頁面上的所有內容啟用粘滯連接：

```xml
/stickyConnections {
  /paths {
    "/content/image"
    "/content/video"
    "/var/files/pdfs"
  }
}
```

### 僅http {#httponly}

啟用粘滯連接後，調度器模組將 `renderid` 餅乾。 此Cookie沒有 `httponly` 為了加強安全，應添加標誌。 可以通過設定 `httpOnly` 屬性 `/stickyConnections` 節點 `dispatcher.any` 配置檔案。 屬性的值( `0` 或 `1`)定義 `renderid` 餅乾 `HttpOnly` 屬性附加。 預設值為 `0`，表示不添加屬性。

有關 `httponly` 標誌，讀取 [此頁](https://www.owasp.org/index.php/HttpOnly)。

### 安全 {#secure}

啟用粘滯連接後，調度器模組將 `renderid` 餅乾。 此Cookie沒有 `secure` 為了加強安全，應添加標誌。 可以通過設定 `secure` 屬性 `/stickyConnections` 節點 `dispatcher.any` 配置檔案。 屬性的值( `0` 或 `1`)定義 `renderid` 餅乾 `secure` 屬性附加。 預設值為 `0`，表示將添加屬性 **如果** 傳入請求是安全的。 如果值設定為 `1`，則無論傳入請求是否安全，都會添加安全標誌。

## 處理呈現連接錯誤 {#handling-render-connection-errors}

當呈現伺服器返回500錯誤或不可用時，配置Dispatcher行為。

### 指定運行狀況檢查頁 {#specifying-a-health-check-page}

使用 `/health_check` 屬性，指定在出現500狀態代碼時檢查的URL。 如果此頁還返回500狀態代碼，則該實例被視為不可用，並且可配置的時間懲罰( `/unavailablePenalty`)應用於呈現，然後重試。

```xml
/health_check
  {
  # Page gets contacted when an instance returns a 500
  /url "/health_check.html"
  }
```

### 指定頁重試延遲 {#specifying-the-page-retry-delay}

的 `/retryDelay` 屬性設定Dispatcher在使用場呈現的連接嘗試輪次之間等待的時間（秒）。 對於每輪，Dispatcher嘗試連接到呈現器的最大次數是場中呈現的次數。

Dispatcher使用的值 `"1"` 如果 `/retryDelay` 未明確定義。 預設值在大多數情況下都適用。

```xml
/retryDelay "1"
```

### 配置重試次數 {#configuring-the-number-of-retries}

的 `/numberOfRetries` 屬性設定Dispatcher在呈現時執行的連接嘗試的最大輪數。 如果在此次重試次數後Dispatcher無法成功連接到呈現器，則Dispatcher返回失敗的響應。

對於每輪，Dispatcher嘗試連接到呈現器的最大次數是場中呈現的次數。 因此， Dispatcher嘗試連接的最大次數為( `/numberOfRetries`)x（呈現數）。

如果未顯式定義值，則預設值為 `5`。

```xml
/numberOfRetries "5"
```

### 使用故障轉移機制 {#using-the-failover-mechanism}

啟用Dispatcher伺服器場上的故障切換機制，以便在原始請求失敗時將請求重新發送到不同的呈現形式。 啟用故障轉移後，Dispatcher具有以下行為：

* 當對呈現器的請求返回HTTP狀態503（不可用）時，Dispatcher會將該請求發送到其他呈現器。
* 當對呈現器的請求返回HTTP狀態50x（除503外）時，Dispatcher會發送為 `health_check` 屬性。
   * 如果運行狀況檢查返回500(INTERNAL_SERVER_ERROR)，則Dispatcher會將原始請求發送到其他呈現器。
   * 如果健康檢查返回HTTP狀態200，則Dispatcher會將初始HTTP 500錯誤返回給客戶端。

要啟用故障轉移，請將以下行添加到場（或網站）:

```xml
/failover "1"
```

>[!NOTE]
>
>要重試包含正文的HTTP請求，Dispatcher將發送 `Expect: 100-continue` 在假離線實際內容之前，請求標題到render。 CQ 5.5（帶CQSE），然後立即回答100（繼續）或錯誤代碼。 其他Servlet容器也應支援此功能。

## 忽略中斷錯誤 — /ignoreEINTR {#ignoring-interruption-errors-ignoreeintr}

>[!CAUTION]
>
>通常不需要此選項。 您只需在看到以下日誌消息時使用此選項：
>
>`Error while reading response: Interrupted system call`

任何面向檔案系統的系統調用都可以中斷 `EINTR` 如果系統調用的對象位於通過NFS訪問的遠程系統上。 這些系統調用是否可以超時或被中斷取決於基礎檔案系統如何裝載到本地電腦上。

使用 `/ignoreEINTR` 參數（如果實例具有此類配置，且日誌包含以下消息）:

`Error while reading response: Interrupted system call`

在內部， Dispatcher使用可以表示為AEM:

```text
while (response not finished) {  
read more data  
}
```

當 `EINTR` 出現在「」中 `read more data`在接收任何資料之前，接收信號導致。

要忽略此類中斷，可將以下參數添加到 `dispatcher.any` 之前 `/farms`):

`/ignoreEINTR "1"`

設定 `/ignoreEINTR` 至 `"1"` 導致Dispatcher在讀取完整響應之前繼續嘗試讀取資料。 預設值為 `0` 並取消激活選項。

## 設計全局屬性的模式 {#designing-patterns-for-glob-properties}

Dispatcher配置檔案中使用的幾個部分 `glob` 屬性作為客戶端請求的選擇標準。 值 `glob` 屬性是Dispatcher與請求的一個方面進行比較的模式，如所請求資源的路徑或客戶端的IP地址。 例如， `/filter` 節使用 `glob` 模式，用於標識Dispatcher所執行或拒絕的頁的路徑。

的 `glob` 值可以包括通配符和字母數字字元來定義模式。

| 通配符 | 說明 | 示例 |
|--- |--- |--- |
| `*` | 匹配字串中任意字元的零個或多個連續實例。 匹配的最終字元由以下任一情況確定： <br/>字串中的字元與模式中的下一個字元匹配，並且模式字元具有以下特徵：<br/><ul><li>不是*</li><li>不是？</li><li>文字字元（包括空格）或字元類。</li><li>到達圖案的末尾。</li></ul>在字元類中，字元是字面解釋的。 | `*/geo*` 匹配下面的任何頁 `/content/geometrixx` 和 `/content/geometrixx-outdoors` 的下界。 以下HTTP請求與glob模式匹配： <br/><ul><li>`"GET /content/geometrixx/en.html"`</li><li>`"GET /content/geometrixx-outdoors/en.html"` </li></ul><br/> `*outdoors/*` <br/>匹配下面的任何頁 `/content/geometrixx-outdoors` 的下界。 例如，以下HTTP請求與glob模式匹配： <br/><ul><li>`"GET /content/geometrixx-outdoors/en.html"`</li></ul> |
| `?` | 匹配任何單個字元。 使用外部字元類。 在字元類中，該字元是字面解釋的。 | `*outdoors/??/*`<br/> 與geometrixx-outdoors站點中任何語言的頁面匹配。 例如，以下HTTP請求與glob模式匹配： <br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul><br/>以下請求與全局模式不匹配： <br/><ul><li>&quot;GET/content/geometrixx-outdoors/en.html&quot;</li></ul> |
| `[ and ]` | 對字元類的開頭和結尾進行定義。 字元類可以包括一個或多個字元範圍和單個字元。<br/>如果目標字元與字元類中或定義範圍內的任何字元匹配，則出現匹配。<br/>如果未包括右括弧，則陣列不產生匹配項。 | `*[o]men.html*`<br/> 匹配以下HTTP請求：<br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li></ul><br/>與以下HTTP請求不匹配：<br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul><br/> `*[o/]men.html*` <br/>匹配以下HTTP請求： <br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul> |
| `-` | 表示字元範圍。 用於字元類。  在字元類之外，該字元將按字面方式進行解釋。 | `*[m-p]men.html*` 匹配以下HTTP請求： <br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li></ul>與以下HTTP請求不匹配：<br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul> |
| `!` | 否定後面的字元或字元類。 僅用於否定字元類內的字元和字元範圍。 相當於 `^ wildcard`。 <br/>在字元類之外，該字元將按字面方式進行解釋。 | `*[!o]men.html*`<br/> 匹配以下HTTP請求： <br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul><br/>與以下HTTP請求不匹配： <br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li></ul><br/>`*[!o!/]men.html*`<br/> 與以下HTTP請求不匹配：<br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"` 或 `"GET /content/geometrixx-outdoors/en/men. html"`</li></ul> |
| `^` | 否定後面的字元或字元範圍。 用於僅否定字元類內的字元和字元範圍。 相當於 `!` 通配符。 <br/>在字元類之外，該字元將按字面方式進行解釋。 | 的示例 `!` 應用通配符，替換 `!` 示例模式中的字元 `^` 字元。 |


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

在Web伺服器配置中，可以設定：

* Dispatcher日誌檔案的位置。
* 日誌級別。

有關詳細資訊，請參閱Web伺服器文檔和Dispatcher實例的自述檔案。

**Apache旋轉/管道日誌**

如果使用 **阿帕奇** Web伺服器可以使用標準功能進行旋轉和/或管道日誌。 例如，使用管道日誌：

`DispatcherLog "| /usr/apache/bin/rotatelogs logs/dispatcher.log%Y%m%d 604800"`

這將自動旋轉：

* 調度程式日誌檔案；帶有副檔名中的時間戳(`logs/dispatcher.log%Y%m%d`)。
* 每週（60 x 60 x 24 x 7 = 604800秒）。

請參閱日誌輪替和管道日誌上的Apache Web伺服器文檔；例如 [Apache 2.4](https://httpd.apache.org/docs/2.4/logs.html)。

>[!NOTE]
>
>安裝後，預設日誌級別為高（即級別3 =調試），因此Dispatcher將記錄所有錯誤和警告。 這在初期非常有用。
>
>但是，這需要額外的資源，因此當Dispatcher工作正常時 *根據你的要求*，可以（應）降低日誌級別。

### 跟蹤記錄 {#trace-logging}

除了Dispatcher的其他增強功能外， 4.2.0版還引入了跟蹤日誌記錄。

此級別高於調試日誌記錄，顯示日誌中的其他資訊。 它為：

* 轉發報頭的值；
* 正應用於某個操作的規則。

通過將日誌級別設定為 `4` 在Web伺服器中。

下面是啟用跟蹤的日誌示例：

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

請求與阻止規則匹配的檔案時記錄的事件：

```xml
[Thu Mar 03 14:42:45 2016] [T] [11831] 'GET /content.infinity.json HTTP/1.1' was blocked because of /0082
```

## 確認基本工序 {#confirming-basic-operation}

要確認Web伺服器、Dispatcher和實例的基本操作和交互AEM，可以執行以下步驟：

1. 設定 `loglevel` 至 `3`。

1. 啟動Web伺服器；這也會啟動Dispatcher。
1. 啟動實AEM例。
1. 檢查Web伺服器和Dispatcher的日誌和錯誤檔案。
   * 根據Web伺服器的不同，您應看到以下消息：
      * `[Thu May 30 05:16:36 2002] [notice] Apache/2.0.50 (Unix) configured`與
      * `[Fri Jan 19 17:22:16 2001] [I] [19096] Dispatcher initialized (build XXXX)`

1. 通過Web伺服器瀏覽網站。 確認內容是否按要求顯示。\
   例如，在本地安裝中，AEM在埠上運行 `4502` 和Web伺服器 `80` 使用以下兩種方式訪問網站控制台：
   * `https://localhost:4502/libs/wcm/core/content/siteadmin.html`
   * `https://localhost:80/libs/wcm/core/content/siteadmin.html`
   * 結果應該一致。 使用相同機制確認對其他頁面的訪問。

1. 檢查是否正在填充快取目錄。
1. 激活頁面以檢查快取是否正確刷新。
1. 如果一切正常，可以 `loglevel` 至 `0`。

## 使用多個 Dispatcher {#using-multiple-dispatchers}

您可以透過複雜的設定來使用多個 Dispatcher。例如您可以使用：

* 一個 Dispatcher 在內部網路上發佈網站
* 另一個 Dispatcher 位於不同的位址下，並具有不同的安全設定，以便在網際網路上發佈相同的內容。

在這種情況下，請確定每個請求只通過一個 Dispatcher。Dispatcher 不會處理來自其他 Dispatcher 的請求。因此，請確定兩個 Dispatcher 都直接存取 AEM 網站。

## 調試 {#debugging}

添加標題時 `X-Dispatcher-Info` 對於請求， Dispatcher將回答目標是否已快取、從快取中返回還是根本不可快取。 響應標頭 `X-Cache-Info` 以可讀的形式包含此資訊。 您可以使用這些響應標頭來調試與由Dispatcher快取的響應相關的問題。

預設情況下未啟用此功能，因此為響應標題的順序 `X-Cache-Info` 要包括，場必須包含以下條目：

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

另外， `X-Dispatcher-Info` 標頭不需要值，但如果使用 `curl` 要進行測試，必須提供值才能發送題頭，例如：

```xml
curl -v -H "X-Dispatcher-Info: true" https://localhost/content/wknd/us/en.html
```

下面是包含響應標頭的清單 `X-Dispatcher-Info` 將返回：

* **快取**\
   目標檔案包含在快取中，並且調度程式已確定傳送該檔案是有效的。
* **快取**\
   目標檔案未包含在快取中，並且調度程式已確定快取輸出並傳送輸出是有效的。
* **快取：stat檔案更新**
目標檔案包含在快取中，但是，最近的stat檔案會使其失效。 調度程式將刪除目標檔案，從輸出中重新建立並傳送它。
* **無法快取：無文檔根**
伺服器場的配置不包含文檔根（配置元素） 
`cache.docroot`).
* **無法快取：快取檔案路徑太長**\
   目標檔案（文檔根檔案和URL檔案的串聯）超過了系統上可能的最長檔案名。
* **無法快取：臨時檔案路徑太長**\
   臨時檔案名模板超出了系統上可能的最長檔案名。 調度程式首先建立臨時檔案，然後才實際建立或覆蓋快取檔案。 臨時檔案名是包含字元的目標檔案名 `_YYYYXXXXXX` 附在上面， `Y` 和 `X` 將被替換以建立唯一名稱。
* **無法快取：請求URL沒有擴展**\
   請求URL沒有副檔名，或者檔案副檔名後面有路徑，例如： `/test.html/a/path`。
* **無法快取：請求不是GET或HEAD**
HTTP方法既不是GET，也不是HEAD。 調度程式假定輸出將包含不應快取的動態資料。
* **無法快取：請求包含查詢字串**\
   該請求包含查詢字串。 調度程式假定輸出取決於給定的查詢字串，因此不進行快取。
* **無法快取：會話管理器未驗證**\
   伺服器場的快取由會話管理器管理(配置包含 `sessionmanagement` 節點)，並且請求不包含相應的驗證資訊。
* **無法快取：請求包含授權**\
   不允許場快取輸出( `allowAuthorized 0`)，並且請求包含身份驗證資訊。
* **無法快取：目標是目錄**\
   目標檔案是目錄。 這可能指出某些概念性錯誤，其中URL和某些子URL都包含可快取的輸出，例如，如果請求 `/test.html/a/file.ext` 首先出現並包含可快取的輸出，調度程式將無法快取後續請求的輸出到 `/test.html`。
* **無法快取：請求URL具有尾斜線**\
   請求URL具有尾斜槓。
* **無法快取：請求URL不在快取規則中**\
   場的快取規則明確拒絕快取某些請求URL的輸出。
* **無法快取：授權檢查器拒絕訪問**\
   伺服器場的授權檢查器拒絕訪問快取的檔案。
* **無法快取：會話無效**
伺服器場的快取由會話管理器管理(配置包含 `sessionmanagement` 節點)，並且用戶的會話無效或不再有效。
* **無法快取：響應包含`no_cache`**
遠程伺服器返回 
`Dispatcher: no_cache` 頭，禁止調度程式快取輸出。
* **無法快取：響應內容長度為零**
響應內容長度為零；調度程式將不會建立零長度檔案。
