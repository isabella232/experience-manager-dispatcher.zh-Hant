---
title: 設定 Dispatcher
description: 了解如何設定Dispatcher。
exl-id: 91159de3-4ccb-43d3-899f-9806265ff132
source-git-commit: 9ad35121bde90916a0376b33853e190b382ce5cd
workflow-type: tm+mt
source-wordcount: '8528'
ht-degree: 2%

---

# 設定 Dispatcher {#configuring-dispatcher}

>[!NOTE]
>
>Dispatcher 版本與 AEM 無關。如果您依循連結至 Dispatcher 文件，且該連結內嵌於舊版 AEM 的文件中，您可能會被重新導向至本頁。

以下各節說明如何設定Dispatcher的各個層面。

## 支援IPv4和IPv6 {#support-for-ipv-and-ipv}

AEM和Dispatcher的所有元素都可安裝在IPv4和IPv6網路中。 請參閱 [IPV4和IPV6](https://experienceleague.adobe.com/docs/experience-manager-65/deploying/introduction/technical-requirements.html?lang=en#ipv-and-ipv).

## Dispatcher組態檔 {#dispatcher-configuration-files}

依預設，Dispatcher設定會儲存在 `dispatcher.any` 文本檔案，但您可以在安裝期間更改此檔案的名稱和位置。

設定檔案包含一系列單值或多值屬性，可控制Dispatcher的行為：

* 屬性名稱的前置詞為正斜線 `/`.
* 多值屬性使用大括弧括住子項 `{ }`.

範例設定的結構如下：

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

您可以包含對設定有貢獻的其他檔案：

* 如果您的設定檔案很大，您可以將其分割為數個較小的檔案（更容易管理），然後納入這些檔案。
* 包含自動生成的檔案。

例如，要在/farms配置中包含檔案myFarm.any ，請使用以下代碼：

```xml
/farms
  {
  $include "myFarm.any"
  }
```

使用星號(`*`)作為萬用字元，以指定要包含的檔案範圍。

例如，若 `farm_1.any` 到 `farm_5.any` 包含從1到5的伺服器陣列的配置，您可以包括如下內容：

```xml
/farms
  {
  $include "farm_*.any"
  }
```

## 使用環境變數 {#using-environment-variables}

您可以在dispatcher.any檔案的字串值屬性中使用環境變數，而非硬式編碼值。 若要納入環境變數的值，請使用格式 `${variable_name}`.

例如，如果dispatcher.any檔案位於與快取目錄相同的目錄中，則 [杜克羅](#specifying-the-cache-directory) 屬性可供使用：

```xml
/docroot "${PWD}/cache"
```

再舉一例，如果您建立的環境變數名為 `PUBLISH_IP` 儲存AEM發佈執行個體的主機名稱，以下是 [/renders](#defining-page-renderers-renders) 屬性可供使用：

```xml
/renders {
  /0001 {
    /hostname "${PUBLISH_IP}"
    /port "8443"
  }
}
```

## 命名Dispatcher例項 {#naming-the-dispatcher-instance-name}

使用 `/name` 屬性，以指定唯一名稱以識別您的Dispatcher例項。 此 `/name` 屬性是配置結構中的頂級屬性。

## 定義伺服器陣列 {#defining-farms-farms}

此 `/farms` 屬性會定義一或多組Dispatcher行為，其中每組行為都與不同的網站或URL相關聯。 此 `/farms` 屬性可包含單一農場或多個農場：

* 當您希望Dispatcher以相同方式處理所有網頁或網站時，請使用單一伺服器陣列。
* 當您網站的不同區域或不同網站需要不同的Dispatcher行為時，請建立多個伺服器陣列。

此 `/farms` 屬性是配置結構中的頂級屬性。 若要定義伺服器陣列，請新增子屬性至 `/farms` 屬性。 使用屬性名稱，可唯一識別Dispatcher例項內的伺服器陣列。

此 `/farmname` 屬性是多值的，且包含定義Dispatcher行為的其他屬性：

* 伺服器陣列套用之頁面的URL。
* 用於轉譯檔案的一或多個服務URL(通常為AEM發佈例項)。
* 用於負載平衡多個文檔呈現器的統計資訊。
* 其他幾種行為，例如要快取的檔案和位置。

值可以包含任何英數字元(a-z、0-9)。 以下示例顯示了兩個名為的場的骨架定義 `/daycom` 和 `/docsdaycom`:

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
>如果使用多個呈現場，則從下到上計算該清單。 這在定義 [虛擬主機](#identifying-virtual-hosts-virtualhosts) 的URL。

每個伺服器陣列屬性都可包含下列子屬性：

| 屬性名稱 | 說明 |
|--- |--- |
| [/homepage](#specify-a-default-page-iis-only-homepage) | 預設首頁（可選）（僅限IIS） |
| [/clientheaders](#specifying-the-http-headers-to-pass-through-clientheaders) | 要傳遞的用戶端HTTP要求的標題。 |
| [/virtualhosts](#identifying-virtual-hosts-virtualhosts) | 此場的虛擬主機。 |
| [/sessionmanagement](#enabling-secure-sessions-sessionmanagement) | 支援工作階段管理和驗證。 |
| [/renders](#defining-page-renderers-renders) | 提供轉譯頁面的伺服器(通常為AEM發佈例項)。 |
| [/filter](#configuring-access-to-content-filter) | 定義Dispatcher啟用存取的URL。 |
| [/vanity_urls](#enabling-access-to-vanity-urls-vanity-urls) | 設定虛名URL的存取權。 |
| [/propagateSyncPost](#forwarding-syndication-requests-propagatesyndpost) | 支援轉發聯合請求。 |
| [/cache](#configuring-the-dispatcher-cache-cache) | 配置快取行為。 |
| [/statistics](#configuring-load-balancing-statistics) | 定義負載平衡計算的統計類別。 |
| [/stickyConnectionsFor](#identifying-a-sticky-connection-folder-stickyconnectionsfor) | 包含黏著檔案的資料夾。 |
| [/health_check](#specifying-a-health-check-page) | 用於確定伺服器可用性的URL。 |
| [/retryDelay](#specifying-the-page-retry-delay) | 重試失敗連接之前的延遲。 |
| [/unavailableDepantim](#reflecting-server-unavailability-in-dispatcher-statistics) | 影響負載平衡計算統計資料的處罰。 |
| [/failover](#using-the-failover-mechanism) | 當原始請求失敗時，將請求重新傳送至不同的轉譯。 |
| [/auth_checker](permissions-cache.md) | 如需需要權限的快取，請參閱 [快取安全內容](permissions-cache.md). |

## 指定預設頁（僅限IIS） — /homepage {#specify-a-default-page-iis-only-homepage}

>[!CAUTION]
>
>此 `/homepage`參數（僅限IIS）不再有效。 請改為使用 [IIS URL重寫模組](https://docs.microsoft.com/en-us/iis/extensions/url-rewrite-module/using-the-url-rewrite-module).
>
>如果您使用Apache，則應使用 `mod_rewrite` 模組。 如需相關資訊，請參閱Apache網站檔案 `mod_rewrite` (例如， [Apache 2.4](https://httpd.apache.org/docs/current/mod/mod_rewrite.html))。 使用時 `mod_rewrite`，建議使用標幟 **[&#39;passthrough|PT&#39;（傳遞至下一個處理常式）](https://helpx.adobe.com/dispatcher/kb/DispatcherModReWrite.html)** 強制重寫引擎將 `uri` 內部欄位 `request_rec` 結構至 `filename` 欄位。

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

## 指定要傳遞的HTTP標題 {#specifying-the-http-headers-to-pass-through-clientheaders}

此 `/clientheaders` 屬性會定義Dispatcher從用戶端HTTP要求傳遞至轉譯器(AEM例項)的HTTP標題清單。

依預設，Dispatcher會將標準HTTP標題轉送至AEM例項。 在某些情況下，您可能會想要轉送其他標題，或移除特定標題：

* 新增AEM例項在HTTP要求中預期的標題，例如自訂標題。
* 移除僅與Web伺服器相關的標題，例如驗證標題。

如果您要自訂要傳遞的標題集，您必須指定完整的標題清單，包括通常預設會包含的標題。

例如，處理發佈例項之頁面啟動請求的Dispatcher例項需要 `PATH` 標題 `/clientheaders` 區段。 此 `PATH` 標題可讓復寫代理與dispatcher之間通訊。

下列程式碼是 `/clientheaders`:

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

## 識別虛擬主機 {#identifying-virtual-hosts-virtualhosts}

此 `/virtualhosts` 屬性會定義Dispatcher為此伺服器陣列接受的所有主機名/URI組合清單。 您可以使用星號(`*`)字元作為萬用字元。 /的值 `virtualhosts` 屬性會使用下列格式：

```xml
[scheme]host[uri][*]
```

* `scheme`:（選用） `https://` 或 `https://.`
* `host`:主機的名稱或IP地址以及埠號（如有必要）。 (請參閱 [https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.23](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.23))
* `uri`:（選用）資源的路徑。

以下示例配置處理myCompany的.com和.ch域以及mySubDivision的所有域的請求：

```xml
   /virtualhosts
    {
    "www.myCompany.com"
    "www.myCompany.ch"
    "www.mySubDivison.*"
    }
```

以下配置句柄 *all* 要求：

```xml
   /virtualhosts
    {
    "*"
    }
```

### 解析虛擬主機 {#resolving-the-virtual-host}

當Dispatcher收到HTTP或HTTPS要求時，會找到最符合 `host,` `uri`，和 `scheme` 請求的標題。 Dispatcher會評估 `virtualhosts` 屬性順序如下：

* Dispatcher從最低的伺服器陣列開始，並在dispatcher.any檔案中向上推進。
* 對於每個伺服器陣列，Dispatcher會從 `virtualhosts` 屬性，並向下顯示值清單。

Dispatcher會以下列方式找出最符合的虛擬主機值：

* 第一個遇到的虛擬主機，與 `host`, `scheme`，和 `uri` ，則會使用。
* 若否 `virtualhosts` 值 `scheme` 和 `uri` 兩者都匹配的部件 `scheme` 和 `uri` ，即第一個遇到的符合 `host` ，則會使用。
* 若否 `virtualhosts` 值的主機部分與請求的主機匹配，該主機部分使用最頂端伺服器陣列的最頂端虛擬主機。

因此，您應將預設虛擬主機放在 `virtualhosts` 最頂端的農場 `dispatcher.any` 檔案。

### 虛擬主機解析示例 {#example-virtual-host-resolution}

下列範例代表 `dispatcher.any` 定義兩個Dispatcher伺服器陣列的檔案，每個伺服器陣列定義 `virtualhosts` 屬性。

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

使用此範例，下表顯示為指定HTTP請求解析的虛擬主機：

| 請求URL | 已解析的虛擬主機 |
|---|---|
| `https://www.mycompany.com/products/gloves.html` | `www.mycompany.com/products/` |
| `https://www.mycompany.com/about.html` | `www.mycompany.com` |

## 啟用安全會話 — /sessionmanagement {#enabling-secure-sessions-sessionmanagement}

>[!CAUTION]
>
>`/allowAuthorized` **必須** 設為 `"0"` 在 `/cache` 區段來啟用此功能。

建立安全工作階段以存取轉譯伺服器陣列，讓使用者需要登入才能存取伺服器陣列中的任何頁面。 登入後，使用者可以存取伺服器陣列中的頁面。 請參閱 [建立封閉的使用者群組](https://experienceleague.adobe.com/docs/experience-manager-65/administering/security/cug.html?lang=en#creating-the-user-group-to-be-used) 以了解如何搭配CUG使用此功能。 另請參閱Dispatcher [安全性檢查清單](/help/using/security-checklist.md) 才能上線。

此 `/sessionmanagement` 屬性是的子屬性 `/farms`.

>[!CAUTION]
>
>如果網站的區段使用不同的存取需求，您需要定義多個伺服器陣列。

**/sessionmanagement** 有幾個子參數：

**/directory** （強制）

儲存會話資訊的目錄。 如果目錄不存在，則會建立該目錄。

>[!CAUTION]
>
> 配置目錄子參數時 **不** 指向根資料夾(`/directory "/"`)，因為這會造成嚴重問題。 應始終指定儲存會話資訊的資料夾的路徑。 例如：

```xml
/sessionmanagement
  {
  /directory "/usr/local/apache/.sessions"
  }
```

**/encode** （可選）

工作階段資訊的編碼方式。 使用 `md5` 使用md5算法進行加密，或 `hex` 十六進位編碼。 如果您加密會話資料，則具有檔案系統訪問權限的用戶無法讀取會話內容。 預設為 `md5`.

**/header** （可選）

儲存授權資訊的HTTP標題或Cookie的名稱。 如果將資訊儲存在http標題中，請使用 `HTTP:<header-name>`. 若要將資訊儲存在Cookie中，請使用 `Cookie:<header-name>`. 如果您未指定值 `HTTP:authorization` 中所有規則的URL區段。

**/timeout** （可選）

上次使用後，工作階段逾時的秒數。 若未指定 `"800"` 已使用，因此工作階段會在使用者最後一個要求後13分鐘多一點逾時。

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

## 定義頁面轉譯器 {#defining-page-renderers-renders}

/renders屬性定義Dispatcher傳送請求以轉譯檔案的URL。 下列範例 `/renders` 區段可識別單一AEM例項以進行轉譯：

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

下列範例/renders小節識別與Dispatcher在相同電腦上執行的AEM例項：

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

下列範例/renders小節會在兩個AEM例項之間平均分發呈現請求：

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

指定存取AEM例項的連線逾時（以毫秒為單位）。 預設為 `"0"`，導致Dispatcher無限期等候。

**/receiveTimeout**

指定允許回應花費的時間（毫秒）。 預設為 `"600000"`，導致Dispatcher等候10分鐘。 設定 `"0"` 完全消除逾時。

如果剖析回應標題時逾時，會傳回504（錯誤閘道）的HTTP狀態。 如果在讀取回應內文時逾時，Dispatcher會將不完整的回應傳回給用戶端，但刪除可能已寫入的任何快取檔案。

**/ipv4**

指定Dispatcher是否使用 `getaddrinfo` 函式（適用於IPv6）或 `gethostbyname` 函式（適用於IPv4），以取得轉譯的IP位址。 值0表示原因 `getaddrinfo` 來使用。 值 `1` 原因 `gethostbyname` 來使用。 預設值為 `0`.

此 `getaddrinfo` 函式會傳回IP位址清單。 Dispatcher會反覆計算地址清單，直到它建立TCP/IP連線為止。 因此， `ipv4` 當呈現主機名稱與多個IP位址和主機相關聯時，屬性很重要，以回應 `getaddrinfo` 函式中，會傳回IP位址清單，這些位址會一律以相同順序顯示。 在此情況下，您應使用 `gethostbyname` 函式，讓Dispatcher連線的IP位址隨機化。

Amazon Elastic Load Balancing(ELB)是一項服務，可使用可能相同順序的IP位址清單回應getaddrinfo。

**/secure**

若 `/secure` 屬性的值為 `"1"` Dispatcher使用HTTPS來與AEM例項通訊。 如需其他詳細資訊，另請參閱 [將Dispatcher設定為使用SSL](dispatcher-ssl.md#configuring-dispatcher-to-use-ssl).

**/always-resolve**

使用Dispatcher版本 **4.1.6**，您可以設定 `/always-resolve` 屬性如下：

* 設為時 `"1"` 它會解析每個請求的主機名稱（Dispatcher永遠不會快取任何IP位址）。 由於需要額外呼叫來取得每個請求的主機資訊，因此可能會對效能造成輕微影響。
* 如果未設定屬性，預設會快取IP位址。

此外，此屬性也可用於防止您遇到動態IP解析問題，如下列範例所示：

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

## 設定內容存取權 {#configuring-access-to-content-filter}

使用 `/filter` 區段來指定Dispatcher接受的HTTP請求。 所有其他請求都會以404錯誤碼（找不到頁面）傳回至Web伺服器。 若否 `/filter` 區段存在，則會接受所有要求。

**注意：** 請求 [statfile](#naming-the-statfile) 一律拒絕。

>[!CAUTION]
>
>請參閱 [Dispatcher安全性檢查清單](security-checklist.md) 以供限制使用Dispatcher存取時參考。 此外，請閱讀 [AEM安全性檢查清單](https://experienceleague.adobe.com/docs/experience-manager-65/administering/security/security-checklist.html?lang=en#security) 有關AEM安裝的其他安全詳細資訊。

此 `/filter` 小節包含一系列規則，這些規則會根據HTTP要求的要求行部分的模式拒絕或允許存取內容。 您應將允許清單策略用於 `/filter` 小節：

* 首先，拒絕訪問所有內容。
* 視需要允許存取內容。

>[!NOTE]
>
>建議只要篩選規則中有任何變更，就清除快取。

### 定義篩選器 {#defining-a-filter}

中的每個項目 `/filter` 區段包含與請求行或整個請求行的特定元素相符的類型和模式。 每個篩選器可包含下列項目：

* **類型**:此 `/type` 指示是否允許或拒絕對符合模式的請求的訪問。 值可以是 `allow` 或 `deny`.

* **請求行的元素：** 包括 `/method`, `/url`, `/query`，或 `/protocol` 以及根據HTTP要求之要求行部分的這些特定部分來篩選要求的模式。 偏好的篩選方法是依請求行的元素進行篩選（而非整個請求行）。

* **請求行的進階元素：** 從Dispatcher 4.2.0開始，有四個新的篩選元素可供使用。 這些新元素包括 `/path`, `/selectors`, `/extension` 和 `/suffix` 分別為5個。 包含一或多個這些項目以進一步控制URL模式。

>[!NOTE]
>
>如需這些元素所參考之請求行的哪一部分的詳細資訊，請參閱 [Sling URL分解](https://sling.apache.org/documentation/the-sling-engine/url-decomposition.html) 維客頁面。

* **全域屬性**:此 `/glob` 屬性可用來比對HTTP要求的整個要求行。

>[!CAUTION]
>
>Dispatcher不建議使用全域篩選。 因此，您應避免在 `/filter` 章節，因為這可能會導致安全問題。 因此，不必：
>
>`/glob "* *.css *"`
>
>您應使用
>
>`/url "*.css"`

#### HTTP要求的要求行部分 {#the-request-line-part-of-http-requests}

HTTP/1.1定義 [request-line](https://www.w3.org/Protocols/rfc2616/rfc2616-sec5.html) 如下所示：

`Method Request-URI HTTP-Version<CRLF>`

此 `<CRLF>` 字元代表歸位字元，後面接著行摘要。 以下範例是當客戶要求WKND網站的美文 — 英文頁面時收到的要求行：

`GET /content/wknd/us/en.html HTTP.1.1<CRLF>`

您的模式必須考慮要求行和 `<CRLF>` 字元。

#### 雙引號與單引號 {#double-quotes-vs-single-quotes}

建立篩選規則時，請使用雙引號 `"pattern"` 簡單模式。 如果您使用Dispatcher 4.2.0或更新版本，而您的模式包含規則運算式，則必須包含規則運算式模式 `'(pattern1|pattern2)'` 單引號內。

#### 規則運算式 {#regular-expressions}

在4.2.0以後的Dispatcher版本中，您可以在篩選模式中加入POSIX延伸規則運算式。

#### 疑難排解篩選器 {#troubleshooting-filters}

如果您的篩選器未以預期的方式觸發，請啟用 [追蹤記錄](#trace-logging) 在dispatcher上，以便查看是哪一個篩選器正在攔截請求。

#### 範例篩選：全部拒絕 {#example-filter-deny-all}

下列範例篩選器區段會使Dispatcher拒絕所有檔案的請求。 您應拒絕訪問所有檔案，然後允許訪問特定區域。

```xml
  /0001  { /glob "*" /type "deny" }
```

請求明確拒絕的區域會導致傳回404錯誤碼（找不到頁面）。

#### 範例篩選：拒絕訪問特定區域 {#example-filter-deny-access-to-specific-areas}

篩選器也可讓您拒絕存取各種元素，例如ASP頁面和發佈執行個體內的敏感區域。 下列篩選器拒絕存取ASP頁面：

```xml
/0002  { /type "deny" /url "*.asp"  }
```

#### 範例篩選：啟用POST請求 {#example-filter-enable-post-requests}

下列範例篩選器允許使用POST方法提交表單資料：

```xml
/filter {
    /0001  { /glob "*" /type "deny" }
    /0002 { /type "allow" /method "POST" /url "/content/[.]*.form.html" }
}
```

#### 範例篩選：允許存取工作流程主控台 {#example-filter-allow-access-to-the-workflow-console}

下列範例顯示用於拒絕外部存取工作流程控制台的篩選器：

```xml
/filter {
    /0001  { /glob "*" /type "deny" }
    /0002  {  /type "allow"  /url "/libs/cq/workflow/content/console*"  }
}
```

如果您的發佈例項使用Web應用程式內容（例如發佈），則這也可以新增至您的篩選器定義。

```xml
/0003   { /type "deny"  /url "/publish/libs/cq/workflow/content/console/archive*"  }
```

如果您仍需要存取限制區域內的單一頁面，則可允許存取這些頁面。 例如，若要允許存取工作流程控制台中的封存索引標籤，請新增下列區段：

```xml
/0004  { /type "allow"  /url "/libs/cq/workflow/content/console/archive*"   }
```

>[!NOTE]
>
>當多個篩選模式套用至請求時，套用的最後一個篩選模式即有效。

#### 範例篩選：使用規則運算式 {#example-filter-using-regular-expressions}

此篩選器會使用規則運算式，在非公用內容目錄中啟用擴充功能，定義於此處的單引號之間：

```xml
/005  {  /type "allow" /extension '(css|gif|ico|js|png|swf|jpe?g)' }
```

#### 範例篩選：篩選請求URL的其他元素 {#example-filter-filter-additional-elements-of-a-request-url}

以下是封鎖內容從 `/content` 路徑及其子樹，使用路徑的篩選器、選取器和擴充功能：

```xml
/006 {
        /type "deny"
        /path "/content/*"
        /selectors '(feed|rss|pages|languages|blueprint|infinity|tidy)'
        /extension '(json|xml|html)'
        }
```

### 範例/filter區段 {#example-filter-section}

在設定Dispatcher時，您應盡可能限制外部存取。 下列範例將外部訪客的存取權限降至最低：

* `/content`
* 其他內容，例如設計和用戶端資料庫；例如：

   * `/etc/designs/default*`
   * `/etc/designs/mydesign*`

建立篩選器後， [測試頁面存取](#testing-dispatcher-security) 以確保AEM例項安全。

以下 `/filter` 區段 `dispatcher.any` 檔案可作為 [Dispatcher設定檔。](#dispatcher-configuration-files)

此範例以隨Dispatcher提供的預設設定檔案為基礎，並打算以範例形式用於生產環境。 前置詞為的項目 `#` 已停用（留言），若您決定啟用其中任一項(透過將 `#` )，因為這可能會對安全性造成影響。

您應拒絕存取所有項目，然後允許存取特定（有限）元素：

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
>與Apache搭配使用時，請根據Dispatcher模組的DispatcherUseProcessedURL屬性來設計您的篩選器URL模式。 (請參閱 [Apache Web Server — 為Dispatcher配置Apache Web Server](dispatcher-install.md##apache-web-server-configure-apache-web-server-for-dispatcher).)

>[!NOTE]
>
>篩選器 `0030` 和 `0031` 關於Dynamic Media適用於AEM 6.0及更新版本。

如果您選擇延伸存取權，請考慮下列建議：

* 外部存取 `/admin` 應該永遠 *完全* 如果您使用CQ 5.4版或舊版，則會停用。

* 允許存取 `/libs`. 應允許個別存取。
* 拒絕對復寫配置的訪問，因此無法看到：

   * `/etc/replication.xml*`
   * `/etc/replication.infinity.json*`

* 拒絕訪問Google小工具反向代理：

   * `/libs/opensocial/proxy*`

視您的安裝而定，下可能會有其他資源 `/libs`, `/apps` 或者在其他地方，這必須是可以利用的。 您可以使用 `access.log` 檔案，作為確定外部訪問的資源的一種方法。

>[!CAUTION]
>
>對控制台和目錄的訪問可能會對生產環境帶來安全風險。 除非您有明確的理由，否則應保持停用狀態（注釋掉）。

>[!CAUTION]
>
>如果您 [在發佈環境中使用報表](https://experienceleague.adobe.com/docs/experience-manager-65/administering/operations/reporting.html?lang=en#using-reports-in-a-publish-environment) 您應將Dispatcher設定為拒絕 `/etc/reports` 外部訪客。

### 限制查詢字串 {#restricting-query-strings}

自Dispatcher 4.1.5版起，請使用 `/filter` 區段來限制查詢字串。 It is highly recommended to explicitly allow query strings and exclude generic allowance through `allow` filter elements.

單一項目可以 `glob` 或是某種組合 `method`, `url`, `query`，和 `version`，但不能兩者兼而有之。 下列範例允許 `a=*` 查詢字串，並拒絕解析為的URL的所有其他查詢字串 `/etc` 節點：

```xml
/filter {
 /0001 { /type "deny" /method "POST" /url "/etc/*" }
 /0002 { /type "allow" /method "GET" /url "/etc/*" /query "a=*" }
}
```

>[!NOTE]
>
>如果規則包含 `/query`，則只會比對包含查詢字串的要求，並符合提供的查詢模式。
>
>在上例中，如果 `/etc` 也應允許沒有查詢字串的規則，則需要下列規則：

```xml
/filter {  
>/0001 { /type "deny" /method “*" /url "/path/*" }  
>/0002 { /type "allow" /method "GET" /url "/path/*" }  
>/0003 { /type “deny" /method "GET" /url "/path/*" /query "*" }  
>/0004 { /type "allow" /method "GET" /url "/path/*" /query "a=*" }  
}  
```

### 測試Dispatcher安全性 {#testing-dispatcher-security}

Dispatcher篩選器應會封鎖AEM發佈例項上對下列頁面和指令碼的存取權。 使用網頁瀏覽器嘗試以網站訪客會的方式開啟下列頁面，並驗證是否傳回程式碼404。 如果獲得任何其他結果，請調整您的篩選。

請注意，您應會看到 `/content/add_valid_page.html?debug=layout`.

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

在終端或命令提示符中發出以下命令，以確定是否啟用匿名寫入訪問。 您不應將資料寫入節點。

`curl -X POST "https://anonymous:anonymous@hostname:port/content/usergenerated/mytestnode"`

在終端機或命令提示字元中發出下列命令，以嘗試使Dispatcher快取失效，並確保您收到程式碼404回應：

`curl -H "CQ-Handle: /content" -H "CQ-Path: /content" https://yourhostname/dispatcher/invalidate.cache`

## 啟用虛名URL的存取 {#enabling-access-to-vanity-urls-vanity-urls}

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (sbroders@adobe.com)
Last Modified Date: 2015-03-25T14:23:05.185-0400

<p style="font-family: tahoma, arial, helvetica, sans-serif; font-size: 12px;">For https://jira.corp.adobe.com/browse/DOC-4812</p> 
<p style="font-family: tahoma, arial, helvetica, sans-serif; font-size: 12px;">The "com.adobe.granite.dispatcher.vanityurl.content" package needs to be made public before publishing this contnet.</p>
 -->

設定Dispatcher以啟用對您AEM頁面所設定之虛名URL的存取。

啟用虛名URL的存取時，Dispatcher會定期呼叫呈現例項上執行的服務，以取得虛名URL清單。 Dispatcher將此清單儲存在本機檔案中。 若因 `/filter` 區段，則Dispatcher會諮詢虛名URL的清單。 如果拒絕的URL在清單上，Dispatcher會允許存取虛名URL。

若要啟用虛名URL的存取，請新增 `/vanity_urls` 區段 `/farms` 區段，類似下列範例：

```xml
 /vanity_urls {
      /url "/libs/granite/dispatcher/content/vanityUrls.html"
      /file "/tmp/vanity_urls"
      /delay 300
 }
```

此 `/vanity_urls` 一節包含下列屬性：

* `/url`:演算執行個體上執行之虛名URL服務的路徑。 此屬性的值必須是 `"/libs/granite/dispatcher/content/vanityUrls.html"`.

* `/file`:本機檔案的路徑，Dispatcher會在此儲存虛名URL清單。 請確定Dispatcher對此檔案具有寫入存取權。
* `/delay`:（秒）呼叫虛名URL服務之間的時間。

>[!NOTE]
>
>如果您的轉譯是AEM的例項，您必須安裝 [來自Software Distribution的VanityURLS-Components套件](https://experience.adobe.com/#/downloads/content/software-distribution/en/aem.html?package=/content/software-distribution/en/details.html/content/dam/aem/public/adobe/packages/granite/vanityurls-components) 啟用虛名URL服務。 (請參閱 [Software Distribution](https://experienceleague.adobe.com/docs/experience-manager-65/administering/contentmanagement/package-manager.html?lang=en#software-distribution) 如需更多詳細資訊。)

請依照下列程式啟用虛名URL的存取權。

1. 如果您的轉譯服務是AEM例項，請安裝 `com.adobe.granite.dispatcher.vanityurl.content` 封裝（請參閱上述附註）。
1. 針對您為AEM或CQ頁面設定的每個虛名URL，請確定 [`/filter`](#configuring-access-to-content-filter) 設定拒絕URL。 如有必要，請新增拒絕URL的篩選器。
1. 新增 `/vanity_urls` 下文 `/farms`.
1. 重新啟動Apache Web伺服器。

## 轉發聯合請求 — /propagateSyndPost {#forwarding-syndication-requests-propagatesyndpost}

整合請求通常僅針對Dispatcher，因此依預設，這些請求不會傳送至轉譯器(例如AEM例項)。

如有必要，請設定 `/propagateSyndPost` 屬性 `"1"` 將整合請求轉發到Dispatcher。 若已設定，您必須確定篩選器區段中不會拒絕POST要求。

## 設定Dispatcher快取 — /cache {#configuring-the-dispatcher-cache-cache}

此 `/cache` 一節控制Dispatcher快取文檔的方式。 設定數個子屬性以實作快取策略：

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

快取區段的範例如下所示：

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
>若為需要權限的快取，請閱讀 [快取安全內容](permissions-cache.md).

### 指定快取目錄 {#specifying-the-cache-directory}

此 `/docroot` 屬性標識儲存快取檔案的目錄。

>[!NOTE]
>
>值必須與Web伺服器的檔案根路徑完全相同，這樣Dispatcher和Web伺服器就能處理相同的檔案。\
>Web伺服器在使用Dispatcher快取檔案時負責傳送正確的狀態代碼，因此也請務必尋找它。

如果使用多個伺服器陣列，則每個伺服器陣列必須使用不同的文檔根。

### 命名Statfile {#naming-the-statfile}

此 `/statfile` 屬性標識要用作statfile的檔案。 Dispatcher使用此檔案來註冊最新內容更新的時間。 statfile可以是Web伺服器上的任何檔案。

statfile沒有內容。 內容更新時，Dispatcher會更新時間戳記。 預設的statfile的名稱為 `.stat` 並儲存在docroot中。 Dispatcher會封鎖對statfile的存取。

>[!NOTE]
>
>若 `/statfileslevel` 已設定，則Dispatcher會忽略 `/statfile` 屬性與使用 `.stat` 作為名稱。

### 發生錯誤時提供過時文檔 {#serving-stale-documents-when-errors-occur}

此 `/serveStaleOnError` 屬性可控制轉譯伺服器傳回錯誤時，Dispatcher是否會傳回失效的檔案。 依預設，當接觸到statfile且讓快取內容失效時，Dispatcher會在下次請求快取內容時刪除該內容。

若 `/serveStaleOnError` 設為 `"1"`，除非轉譯伺服器傳回成功回應，否則Dispatcher不會從快取中刪除失效的內容。 來自AEM的5xx回應或連線逾時會導致Dispatcher提供過時內容，並以111的和HTTP狀態回應（重新驗證失敗）。

### 使用驗證時快取 {#caching-when-authentication-is-used}

此 `/allowAuthorized` 屬性會控制是否快取包含下列任何驗證資訊的請求：

* 此 `authorization` 標題
* 名為的Cookie `authorization`
* 名為的Cookie `login-token`

預設情況下，不會快取包含此身份驗證資訊的請求，因為當快取文檔返回到客戶端時，不會執行身份驗證。 This configuration prevents Dispatcher from serving cached documents to users who do not have the necessary rights.

However, if your requirements permit the caching of authenticated documents, set `/allowAuthorized` to one:

`/allowAuthorized "1"`

>[!NOTE]
>
>To enable session management (using the `/sessionmanagement` property), the `/allowAuthorized` property must be set to `"0"`.

### 指定要快取的文檔 {#specifying-the-documents-to-cache}

The `/rules` property controls which documents are cached according to the document path. 無論 `/rules` 屬性，則Dispatcher在下列情況下不會快取檔案：

* 如果請求URI包含問號(`?`)。
   * This usually indicates a dynamic page, such as a search result that does not need to be cached.
* 缺少檔案副檔名。
   * Web 伺服器需要副檔名來判斷文件類型 (MIME 類型)。
* 驗證標頭已設定 (這可以設定)。
* 如果AEM例項使用下列標題回應：

   * `no-cache`
   * `no-store`
   * `must-revalidate`

>[!NOTE]
>
>GET 或 HEAD (用於 HTTP 標頭) 方法可讓 Dispatcher 快取。如需回應標頭快取的詳細資訊，請參閱 [快取HTTP回應標題](#caching-http-response-headers) 區段。

中的每個項目 `/rules` 屬性包含 [`glob`](#designing-patterns-for-glob-properties) 模式和類型：

* 此 `glob` 模式用於匹配文檔的路徑。
* 類型指示是否快取與 `glob` 模式。 此值可以是允許（快取文檔）或拒絕（始終呈現文檔）。

如果您沒有動態頁面（除了上述規則已排除的頁面以外），您可以設定Dispatcher來快取所有內容。 此區段的規則區段如下：

```xml
/rules
  {
    /0000  {  /glob "*"   /type "allow" }
  }
```

如需全域屬性的相關資訊，請參閱 [設計全局屬性的模式](#designing-patterns-for-glob-properties).

如果您的頁面中有某些區段是動態的（例如新聞應用程式），或是封閉的使用者群組內，您可以定義例外：

>[!NOTE]
>
>不得快取封閉的使用者群組，因為系統未檢查快取頁面的使用者權限。

```xml
/rules
  {
   /0000  { /glob "*" /type "allow" }
   /0001  { /glob "/en/news/*" /type "deny" }
   /0002  { /glob "*/private/*" /type "deny"  }
  }
```

**壓縮**

在Apache Web伺服器上，您可以壓縮快取的文檔。 如果客戶端請求壓縮，則Apache可以以壓縮格式返回文檔。 透過啟用Apache模組，自動完成壓縮 `mod_deflate`，例如：

```xml
AddOutputFilterByType DEFLATE text/plain
```

預設情況下，模組會與Apache 2.x一起安裝。

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

### 按資料夾級別使檔案失效 {#invalidating-files-by-folder-level}

使用 `/statfileslevel` 根據快取檔案的路徑使其無效的屬性：

* Dispatcher建立 `.stat`檔案從資料夾到您指定的級別。 資料夾為0級。
* 若觸及 `.stat` 檔案。 此 `.stat` 將檔案的上次修改日期與快取文檔的上次修改日期進行比較。 如果 `.stat` 檔案較新。

* 若某個層級的檔案失效， **all** `.stat` 檔案 **to** 已失效檔案或已配置的 `statsfilevel` （以較小者為準）會被觸碰。

   * 例如：如果您設定 `statfileslevel` 屬性設為6，而檔案在第5層時每次失效 `.stat` 檔案從docroot到5將被觸摸。 在此範例中，如果檔案在7級失效，則每次。 `stat` 檔案從docroot移動到6(因為 `/statfileslevel = "6"`)。

僅資源 **沿著路** 會影響已失效的檔案。 請考量下列範例：網站使用結構 `/content/myWebsite/xx/.` 如果您設定 `statfileslevel` as 3,a `.stat`檔案的建立方式如下：

* `docroot`
* `/content`
* `/content/myWebsite`
* `/content/myWebsite/*xx*`

若 `/content/myWebsite/xx` 失效，然後每 `.stat` 檔案從docroot到 `/content/myWebsite/xx`被觸摸。 這隻適用於 `/content/myWebsite/xx` 而不是 `/content/myWebsite/yy` 或 `/content/anotherWebSite`.

>[!NOTE]
>
>傳送其他標題可防止失效 `CQ-Action-Scope:ResourceOnly`. 這可用來排清特定資源，而不會使快取的其他部分失效。 請參閱 [本頁](https://adobe-consulting-services.github.io/acs-aem-commons/features/dispatcher-flush-rules/index.html) 和 [手動使Dispatcher快取失效](https://experienceleague.adobe.com/docs/experience-manager-dispatcher/using/configuring/page-invalidate.html?lang=en#configuring) 以取得其他詳細資訊。

>[!NOTE]
>
>如果您指定 `/statfileslevel` 屬性， `/statfile` 會忽略屬性。

### 自動使快取檔案失效 {#automatically-invalidating-cached-files}

此 `/invalidate` 屬性會定義內容更新時自動失效的檔案。

在自動失效的情況下，Dispatcher不會在內容更新後刪除快取檔案，但會在下次請求時檢查其有效性。 快取中未自動失效的檔案會保留在快取中，直到內容更新明確刪除為止。

自動失效通常用於HTML頁面。 HTML頁面通常包含其他頁面的連結，因此難以判斷內容更新是否會影響頁面。 若要確保內容更新時所有相關頁面都失效，請自動使所有HTML頁面失效。 下列設定會使所有HTML頁面失效：

```xml
  /invalidate
  {
   /0000  { /glob "*" /type "deny" }
   /0001  { /glob "*.html" /type "allow" }
  }
```

如需全域屬性的相關資訊，請參閱 [設計全局屬性的模式](#designing-patterns-for-glob-properties).

此設定會在 `/content/wknd/us/en` 已啟用：

* 所有模式為en的檔案。*會從 `/content/wknd/us` 檔案夾。
* 此 `/content/wknd/us/en./_jcr_content` 檔案夾。
* 符合 `/invalidate` 不會立即刪除設定。 下次請求發生時，會刪除這些檔案。 在我們的範例中 `/content/wknd.html` 不會刪除，則會在 `/content/wknd.html` 已請求。

如果您提供自動產生的PDF和ZIP檔案以供下載，您也可能必須自動使這些檔案無效。 此設定範例如下所示：

```xml
/invalidate
  {
   /0000 { /glob "*" /type "deny" }
   /0001 { /glob "*.html" /type "allow" }
   /0002 { /glob "*.zip" /type "allow" }
   /0003 { /glob "*.pdf" /type "allow" }
  }
```

AEM與Adobe Analytics的整合可在 `analytics.sitecatalyst.js` 檔案。 範例 `dispatcher.any` 隨Dispatcher提供的檔案包含此檔案的下列無效規則：

```xml
{
   /glob "*/analytics.sitecatalyst.js"  /type "allow"
}
```

### 使用自訂失效指令碼 {#using-custom-invalidation-scripts}

此 `/invalidateHandler` 屬性可讓您定義指令碼，該指令碼會針對Dispatcher收到的每個無效請求進行呼叫。

會使用下列引數呼叫：

* Handle — 已失效的內容路徑
* Action - The replication Action (e.g. Activate, Deactivate)
* Action Scope - The replication Action&#39;s Scope (empty, unless a header of `CQ-Action-Scope: ResourceOnly` is sent, see [Invalidating Cached Pages from AEM](page-invalidate.md) for details)

This can be used to cover a number of different use cases, such as invalidating other application specific caches, or to handle cases where the externalized URL of a page and its place in the docroot does not match the content path.

以下範例指令碼記錄對檔案的每個無效請求。

```xml
/invalidateHandler "/opt/dispatcher/scripts/invalidate.sh"
```

#### 範例失效處理程式指令碼 {#sample-invalidation-handler-script}

```shell
#!/bin/bash

printf "%-15s: %s %s" $1 $2 $3>> /opt/dispatcher/logs/invalidate.log
```

### Limiting the Clients That Can Flush the Cache {#limiting-the-clients-that-can-flush-the-cache}

The `/allowedClients` property defines specific clients that are allowed to flush the cache. 全域模式會與IP相符。

下列範例：

1. 拒絕訪問任何客戶端
1. 顯式允許訪問localhost

```xml
/allowedClients
  {
   /0001 { /glob "*.*.*.*"  /type "deny" }
   /0002 { /glob "127.0.0.1" /type "allow" }
  }
```

如需全域屬性的相關資訊，請參閱 [設計全局屬性的模式](#designing-patterns-for-glob-properties).

>[!CAUTION]
>
>建議您定義 `/allowedClients`.
>
>若未執行此操作，任何用戶端都可發出呼叫以清除快取；如果重複執行此操作，可能會嚴重影響網站效能。

### 忽略URL參數 {#ignoring-url-parameters}

此 `ignoreUrlParams` 區段會定義在判斷頁面是快取還是從快取傳送時，會忽略哪些URL參數：

* 當要求URL包含全部忽略的參數時，系統會快取頁面。
* 當要求URL包含一或多個未忽略的參數時，系統不會快取頁面。

忽略頁面的參數時，系統會在首次要求頁面時快取頁面。 快取頁面會提供頁面的後續要求，不論要求中的參數值為何。

若要指定要忽略的參數，請將全域規則新增至 `ignoreUrlParams` 屬性：

* 若要忽略參數，請建立允許參數的全域屬性。
* 若要防止快取頁面，請建立拒絕參數的全域屬性。

下列範例會使Dispatcher忽略 `q` 參數，以便快取包含q參數的要求URL:

```xml
/ignoreUrlParams
{
    /0001 { /glob "*" /type "deny" }
    /0002 { /glob "q" /type "allow" }
}
```

使用範例 `ignoreUrlParams` 值，則下列HTTP要求會導致頁面被快取，因為 `q` 參數會被忽略：

```xml
GET /mypage.html?q=5
```

使用範例 `ignoreUrlParams` 值，則下列HTTP要求會使頁面 **not** 因為 `p` 參數未忽略：

```xml
GET /mypage.html?q=5&p=4
```

如需全域屬性的相關資訊，請參閱 [設計全局屬性的模式](#designing-patterns-for-glob-properties).

### 快取HTTP回應標題 {#caching-http-response-headers}

>[!NOTE]
>
>此功能可搭配版本使用 **4.1.11** 的URL。

此 `/headers` 屬性可讓您定義Dispatcher將要快取的HTTP標題類型。 在對未快取資源的第一個請求中，與其中一個已設定值相符的所有標題（請參閱下方的設定範例）會儲存在快取檔案旁的個別檔案中。 在對快取資源的後續請求中，儲存的標題會新增至回應。

Presented below is a sample from the default configuration:

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
>Also be aware that file globbing characters are not allowed. 如需詳細資訊，請參閱 [設計全局屬性的模式](#designing-patterns-for-glob-properties).

>[!NOTE]
>
>If you need Dispatcher to store and deliver ETag response headers from AEM, do the following:
>
>* 在 `/cache/headers`區段。
>* 新增下列項目 [Apache指令](https://httpd.apache.org/docs/2.4/mod/core.html#fileetag) 在「Dispatcher相關」區段中：

>
>```xml
>FileETag none
>```

### Dispatcher Cache File Permissions {#dispatcher-cache-file-permissions}

此 `mode` 屬性指定將哪些檔案權限應用於快取中的新目錄和檔案。 This setting is restricted by the `umask` of the calling process. 它是從以下一個或多個值的總和中構建的八位數：

* `0400` 允許由所有者讀取。
* `0200` Allow write by owner.
* `0100` 允許所有者在目錄中搜索。
* `0040` 允許按組成員讀取。
* `0020` 允許按組成員寫入。
* `0010` 允許組成員在目錄中搜索。
* `0004` 允許他人閱讀。
* `0002` 允許他人寫。
* `0001` 允許其他人在目錄中搜尋。

預設值為 `0755` 允許所有者讀取、寫入或搜索，組和其他人讀取或搜索。

### 節流.stat檔案接觸 {#throttling-stat-file-touching}

使用預設 `/invalidate` 屬性，每次啟動都會有效讓所有 `.html` 檔案(當其路徑符合 `/invalidate` 區段)。 在具有相當流量的網站上，多次後續啟動會增加後端的cpu負載。 在這種情況下，最好&quot;節制&quot; `.stat` 檔案接觸，讓網站保持回應。 您可以使用 `/gracePeriod` 屬性。

此 `/gracePeriod` 屬性會定義上次啟動後，過時、自動失效的資源仍可從快取中提供的秒數。 該屬性可用於設定中，否則批次啟動會重複使整個快取失效。 建議的值為2秒。

如需其他詳細資訊，請一併閱讀 `/invalidate` 和 `/statfileslevel`區段。

### 配置基於時間的快取失效 — /enableTTL {#configuring-time-based-cache-invalidation-enablettl}

若已設定，則 `/enableTTL` 屬性會評估來自後端的回應標題，以及這些標題是否包含 `Cache-Control` 最大年齡或 `Expires` 日期，則建立快取檔案旁的輔助空檔案，修改時間等於到期日。 在修改時間之後請求快取檔案時，會自動從後端重新請求該檔案。

>[!NOTE]
>
>此功能在版本中提供 **4.1.11** 或更新版本的Dispatcher。

## 配置負載平衡 — /statistics {#configuring-load-balancing-statistics}

此 `/statistics` 區段會定義Dispatcher對每個轉譯的回應進行評分的檔案類別。 Dispatcher uses the scores to determine which render to send a request.

Each category that you create defines a glob pattern. Dispatcher會比較所請求內容的URI與這些模式，以判斷所請求內容的類別：

* 類別的順序決定了它們與URI的比較順序。
* The first category pattern that matches the URI is the category of the file. 不再評估類別模式。

Dispatcher最多支援8個統計資料類別。 如果您定義了8個以上的類別，則只會使用前8個類別。

**渲染選擇**

每次Dispatcher需要呈現的頁面時，都會使用下列演算法來選取呈現：

1. If the request contains the render name in a `renderid` cookie, Dispatcher uses that render.
1. 如果請求包含否 `renderid` Cookie,Dispatcher會比較轉譯統計資料：

   1. Dispatcher會決定請求URI的類別。
   1. Dispatcher會判斷哪個轉譯具有該類別最低的回應分數，並選取該轉譯。

1. 如果尚未選取任何呈現，請在清單中使用第一個呈現。

呈現類別的分數會以先前的回應時間，以及Dispatcher嘗試的先前失敗和成功連線為基礎。 對於每次嘗試，所請求URI類別的分數都會更新。

>[!NOTE]
>
>如果不使用負載平衡，則可忽略此部分。

### 定義統計類別 {#defining-statistics-categories}

為要保留用於選擇渲染的統計資訊的每種類型的文檔定義類別。 此 `/statistics` 區段包含 `/categories` 區段。 若要定義類別，請在 `/categories` 區段，其格式如下：

`/name { /glob "pattern"}`

類別 `name` 必須是農場特有的。 此 `pattern` 在 [設計全局屬性的模式](#designing-patterns-for-glob-properties) 區段。

為了確定URI的類別，Dispatcher會將URI與每個類別模式進行比較，直到找到相符項目為止。 Dispatcher從清單中的第一個類別開始，並依順序繼續。 因此，請先放置具有更具體模式的類別。

例如，Dispatcher是預設 `dispatcher.any` 檔案會定義HTML類別和其他類別。 HTML類別較具體，因此會先顯示：

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

下列範例也包含搜尋頁面的類別：

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

### 在Dispatcher統計資料中反映伺服器不可用性 {#reflecting-server-unavailability-in-dispatcher-statistics}

此 `/unavailablePenalty` 屬性設定當與呈現的連接失敗時應用到呈現統計資訊的時間（以秒的十分之一為單位）。 Dispatcher會將時間新增至符合請求URI的統計資料類別。

例如，當無法建立到指定主機名/埠的TCP/IP連接時，將應用懲罰，原因可能是AEM未運行（且未偵聽）或因為與網路相關的問題。

此 `/unavailablePenalty` 屬性是的直接子項 `/farm` 節( `/statistics` 區段)。

若否 `/unavailablePenalty` 屬性存在，值為 `"1"` 中所有規則的URL區段。

```xml
/unavailablePenalty "1"
```

## 識別黏著連線資料夾 — /stickyConnectionsFor {#identifying-a-sticky-connection-folder-stickyconnectionsfor}

The `/stickyConnectionsFor` property defines one folder that contains sticky documents; this will be accessed using the URL. Dispatcher sends all requests, from a single user, that are in this folder to the same render instance. 黏著連線可確保所有檔案都有工作階段資料且一致。 此機制會使用 `renderid` cookie。

下列範例定義與/products資料夾的黏著連線：

```xml
/stickyConnectionsFor "/products"
```

When a page is composed of content from several content nodes, include the `/paths` property that lists the paths to the content. For example, a page contains content from `/content/image`, `/content/video`, and `/var/files/pdfs`. 下列設定會針對頁面上的所有內容啟用黏著連線：

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

When sticky connections are enabled, the dispatcher module sets the `renderid` cookie. 此Cookie沒有 `httponly` 標幟，應新增此標幟以增強安全性。 You can do this by setting the `httpOnly` property in the `/stickyConnections` node of a `dispatcher.any` configuration file. 屬性的值( `0` 或 `1`)定義 `renderid` cookie具有 `HttpOnly` 屬性附加。 預設值為 `0`，表示不會新增屬性。

如需 `httponly` 標幟，閱讀 [本頁](https://www.owasp.org/index.php/HttpOnly).

### secure {#secure}

嚴格連線啟用時，Dispatcher模組會設定 `renderid` cookie。 此Cookie沒有 `secure` 標幟，應新增此標幟以增強安全性。 您可以透過設定 `secure` 屬性 `/stickyConnections` 節點 `dispatcher.any` 設定檔。 屬性的值( `0` 或 `1`)定義 `renderid` cookie具有 `secure` 屬性附加。 預設值為 `0`，表示將新增屬性 **if** 傳入的請求是安全的。 如果值設為 `1`，則無論傳入的要求是否安全，都會新增安全標幟。

## 處理渲染連接錯誤 {#handling-render-connection-errors}

當轉譯伺服器傳回500錯誤或無法使用時，設定Dispatcher行為。

### 指定運行狀況檢查頁 {#specifying-a-health-check-page}

使用 `/health_check` 屬性，以指定當500狀態代碼發生時要檢查的URL。 如果此頁還返回500狀態代碼，則該實例被視為不可用，並且可配置的時間懲罰( `/unavailablePenalty`)會在重試前套用至轉譯。

```xml
/health_check
  {
  # Page gets contacted when an instance returns a 500
  /url "/health_check.html"
  }
```

### 指定頁面重試延遲 {#specifying-the-page-retry-delay}

此 `/retryDelay` 屬性會設定Dispatcher在伺服器陣列轉譯的連線嘗試回合之間等待的時間（以秒為單位）。 對於每個回合，Dispatcher嘗試連線至轉譯的次數上限為伺服器陣列中的轉譯次數。

Dispatcher使用 `"1"` if `/retryDelay` 未明確定義。 預設值在大多數情況下都適用。

```xml
/retryDelay "1"
```

### 設定重試次數 {#configuring-the-number-of-retries}

此 `/numberOfRetries` 屬性會設定Dispatcher在轉譯時執行的連線嘗試次數上限。 如果在此次重試次數之後，Dispatcher無法成功連線至轉譯，Dispatcher會傳回失敗的回應。

對於每個回合，Dispatcher嘗試連線至轉譯的次數上限為伺服器陣列中的轉譯次數。 因此，Dispatcher嘗試連線的次數上限為( `/numberOfRetries`)x（轉譯次數）。

如果未明確定義值，則預設值為 `5`.

```xml
/numberOfRetries "5"
```

### 使用故障轉移機制 {#using-the-failover-mechanism}

在原始請求失敗時，啟用Dispatcher伺服器陣列上的故障轉移機制，將請求重新傳送至不同的轉譯。 When failover is enabled, Dispatcher has the following behavior:

* When a request to a render returns HTTP status 503 (UNAVAILABLE), Dispatcher sends the request to a different render.
* 當呈現請求傳回HTTP狀態50x（503除外）時，Dispatcher會傳送針對 `health_check` 屬性。
   * 如果健康狀況檢查傳回500(INTERNAL_SERVER_ERROR),Dispatcher會將原始請求傳送至不同的轉譯。
   * 如果健康檢查傳回HTTP狀態200,Dispatcher會將初始HTTP 500錯誤傳回給用戶端。

要啟用故障轉移，請將以下行添加到場（或網站）中：

```xml
/failover "1"
```

>[!NOTE]
>
>若要重試包含內文的HTTP請求，Dispatcher會傳送 `Expect: 100-continue` 在假設實際內容之前，請求標題至轉譯。 含CQSE的CQ 5.5接著會以100（繼續）或錯誤碼立即回覆。 Other servlet containers should support this as well.

## 忽略中斷錯誤 — /ignoreEINTR {#ignoring-interruption-errors-ignoreeintr}

>[!CAUTION]
>
>通常不需要此選項。 您只需在看到下列記錄訊息時使用：
>
>`Error while reading response: Interrupted system call`

任何面向檔案系統的系統調用都可能中斷 `EINTR` 如果系統調用的對象位於通過NFS訪問的遠程系統上。 這些系統調用是否可能超時或中斷取決於本地電腦上安裝基礎檔案系統的方式。

使用 `/ignoreEINTR` 參數（若您的執行個體具有此類設定，且記錄檔包含下列訊息）:

`Error while reading response: Interrupted system call`

在內部，Dispatcher會使用可表示為的回圈，從遠端伺服器(即AEM)讀取回應：

```text
while (response not finished) {  
read more data  
}
```

當 `EINTR` 發生於 `read more data`」部分，是在收到任何資料之前接收信號的結果。

要忽略此類中斷，可將以下參數添加到 `dispatcher.any` （之前） `/farms`):

`/ignoreEINTR "1"`

設定 `/ignoreEINTR` to `"1"` 會使Dispatcher繼續嘗試讀取資料，直到讀取完整回應為止。 預設值為 `0` 並停用選項。

## 設計全局屬性的模式 {#designing-patterns-for-glob-properties}

Dispatcher設定檔案中的數個區段使用 `glob` 屬性作為用戶端請求的選擇標準。 的值 `glob` 屬性是Dispatcher與請求的一個方面（例如所請求資源的路徑或用戶端的IP位址）相比較的模式。 例如， `/filter` 區段使用 `glob` 模式，用於識別Dispatcher作用或拒絕的頁面路徑。

此 `glob` 值可包含萬用字元和英數字元來定義模式。

| 萬用字元 | 說明 | 範例 |
|--- |--- |--- |
| `*` | 匹配字串中任何字元的零個或多個連續例項。 匹配的最終字元由以下任一情況決定： <br/>字串中的字元與模式中的下一個字元匹配，而模式字元具有以下特徵：<br/><ul><li>不是*</li><li>不是？</li><li>常值字元（包括空格）或字元類。</li><li>模式的結尾已到。</li></ul>在字元類別中，字元會以字面方式解譯。 | `*/geo*` 符合下方的任何頁面 `/content/geometrixx` 節點和 `/content/geometrixx-outdoors` 節點。 下列HTTP要求符合全域模式： <br/><ul><li>`"GET /content/geometrixx/en.html"`</li><li>`"GET /content/geometrixx-outdoors/en.html"` </li></ul><br/> `*outdoors/*` <br/>符合下方的任何頁面 `/content/geometrixx-outdoors` 節點。 例如，下列HTTP要求符合全域模式： <br/><ul><li>`"GET /content/geometrixx-outdoors/en.html"`</li></ul> |
| `?` | 符合任何單一字元。 使用外部字元類。 在字元類別中，會以字面方式解譯此字元。 | `*outdoors/??/*`<br/> 相符項目：geometrixx-outdoors網站中任何語言的頁面。 例如，下列HTTP要求符合全域模式： <br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul><br/>下列請求與全域模式不符： <br/><ul><li>&quot;GET/content/geometrixx-outdoors/en.html&quot;</li></ul> |
| `[ and ]` | 取消字元類的開頭和結尾的標籤。 字元類別可包含一或多個字元範圍和單一字元。<br/>如果目標字元符合字元類別中的任何字元，或在定義的範圍內，則會發生符合。<br/>如果未包括右括弧，則圖案不會產生匹配項。 | `*[o]men.html*`<br/> 符合下列HTTP要求：<br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li></ul><br/>不符合下列HTTP要求：<br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul><br/> `*[o/]men.html*` <br/>符合下列HTTP要求： <br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul> |
| `-` | 表示字元範圍。 用於字元類。  在字元類之外，該字元被字面地解釋。 | `*[m-p]men.html*` 符合下列HTTP要求： <br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li></ul>不符合下列HTTP要求：<br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul> |
| `!` | 否定後面的字元或字元類。 僅用於否定字元類中的字元和字元範圍。 等同於 `^ wildcard`. <br/>在字元類之外，該字元被字面地解釋。 | `*[!o]men.html*`<br/> 符合下列HTTP要求： <br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul><br/>不符合下列HTTP要求： <br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li></ul><br/>`*[!o!/]men.html*`<br/> 不符合下列HTTP要求：<br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"` 或 `"GET /content/geometrixx-outdoors/en/men. html"`</li></ul> |
| `^` | 否定後面的字元或字元範圍。 僅用於消除字元類別中的字元和字元範圍。 等同於 `!` 萬用字元。 <br/>在字元類之外，該字元被字面地解釋。 | 的範例 `!` 萬用字元，替換 `!` 示例模式中的字元 `^` 字元。 |


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

* Dispatcher記錄檔的位置。
* 記錄層級。

如需詳細資訊，請參閱Web伺服器檔案和Dispatcher例項的自述檔案。

**Apache旋轉/管道記錄檔**

若使用 **Apache** Web伺服器可以對旋轉和/或管道日誌使用標準功能。 例如，使用管道日誌：

`DispatcherLog "| /usr/apache/bin/rotatelogs logs/dispatcher.log%Y%m%d 604800"`

這會自動旋轉：

* dispatcher日誌檔案；具有副檔名中的時間戳記(`logs/dispatcher.log%Y%m%d`)。
* 每週(60 x 60 x 24 x 7 = 604800秒)。

請參閱Log Rotation和Finued Logs上的Apache Web伺服器文檔；例如 [Apache 2.4](https://httpd.apache.org/docs/2.4/logs.html).

>[!NOTE]
>
>安裝時，預設記錄層級為高（即層級3 =除錯），讓Dispatcher記錄所有錯誤和警告。 這在初期非常有用。
>
>不過，這需要額外的資源，因此當Dispatcher正常運作時 *根據你的要求*，您可以（應該）降低記錄層級。

### 追蹤記錄 {#trace-logging}

在Dispatcher的其他增強功能中，4.2.0版也導入了追蹤記錄功能。

這比偵錯記錄檔的級別高，會在記錄檔中顯示其他資訊。 它會新增下列項目的記錄：

* 轉送標題的值；
* 正針對特定動作套用的規則。

您可以將日誌級別設定為 `4` 在Web伺服器中。

以下是已啟用追蹤的記錄範例：

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

請求符合封鎖規則的檔案時記錄的事件：

```xml
[Thu Mar 03 14:42:45 2016] [T] [11831] 'GET /content.infinity.json HTTP/1.1' was blocked because of /0082
```

## 確認基本操作 {#confirming-basic-operation}

若要確認Web伺服器、Dispatcher和AEM例項的基本操作和互動，您可以使用下列步驟：

1. 設定 `loglevel` to `3`.

1. 啟動Web伺服器；這也會啟動Dispatcher。
1. 啟動AEM例項。
1. 檢查您的Web伺服器和Dispatcher的記錄檔和錯誤檔案。
   * 根據您的Web伺服器，您應該會看到下列訊息：
      * `[Thu May 30 05:16:36 2002] [notice] Apache/2.0.50 (Unix) configured`與
      * `[Fri Jan 19 17:22:16 2001] [I] [19096] Dispatcher initialized (build XXXX)`

1. 通過Web伺服器瀏覽網站。 確認內容已視需要顯示。\
   例如，在AEM在連接埠上執行的本機安裝上 `4502` 和Web伺服器 `80` 使用下列兩種方式存取網站主控台：
   * `https://localhost:4502/libs/wcm/core/content/siteadmin.html`
   * `https://localhost:80/libs/wcm/core/content/siteadmin.html`
   * 結果應相同。 使用相同機制確認對其他頁面的存取。

1. 檢查快取目錄是否已填入。
1. 啟動頁面以檢查快取是否正確清除。
1. 如果一切正常運作，您可以減少 `loglevel` to `0`.

## 使用多個 Dispatcher {#using-multiple-dispatchers}

您可以透過複雜的設定來使用多個 Dispatcher。例如您可以使用：

* 一個 Dispatcher 在內部網路上發佈網站
* 另一個 Dispatcher 位於不同的位址下，並具有不同的安全設定，以便在網際網路上發佈相同的內容。

在這種情況下，請確定每個請求只通過一個 Dispatcher。Dispatcher 不會處理來自其他 Dispatcher 的請求。因此，請確定兩個 Dispatcher 都直接存取 AEM 網站。

## 除錯 {#debugging}

新增標題時 `X-Dispatcher-Info` 對於請求，Dispatcher會回答是否已快取、從快取中傳回目標，還是完全無法快取。 回應標題 `X-Cache-Info` 以可讀的形式包含此資訊。 您可以使用這些回應標題來偵錯與Dispatcher快取的回應有關的問題。

此功能預設不會啟用，因此為回應標題的順序 `X-Cache-Info` 要包括，場必須包含以下條目：

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

此外， `X-Dispatcher-Info` 標題不需要值，但若您使用 `curl` 在測試中，您必須提供值才能傳送標題，例如：

```xml
curl -v -H "X-Dispatcher-Info: true" https://localhost/content/wknd/us/en.html
```

以下是包含回應標題的清單， `X-Dispatcher-Info` 將返回：

* **快取**\
   目標檔案包含在快取中，且Dispatcher已判斷傳送該檔案是有效的。
* **快取**\
   快取中未包含目標檔案，且Dispatcher已判斷對輸出進行快取並傳送該檔案有效。
* **快取：stat檔案更新**
目標檔案包含在快取中，但是，最近的統計檔案會使該檔案失效。 Dispatcher將刪除目標檔案，從輸出中重新建立該檔案並進行傳送。
* **無法快取：無文檔根**
伺服器場的配置不包含文檔根（配置元素） 
`cache.docroot`)。
* **無法快取：快取檔案路徑太長**\
   目標檔案（文檔根檔案和URL檔案的串連）超過了系統上可能最長的檔案名。
* **無法快取：臨時檔案路徑太長**\
   臨時檔案名模板超過了系統上可能的最長檔案名。 Dispatcher會先建立暫時檔案，然後再實際建立或覆寫快取的檔案。 臨時檔案名是包含字元的目標檔案名 `_YYYYXXXXXX` 附加於其中， `Y` 和 `X` 將會取代以建立唯一名稱。
* **無法快取：要求URL沒有副檔名**\
   請求URL沒有副檔名，或有路徑在副檔名後面，例如： `/test.html/a/path`.
* **無法快取：要求不是GET或HEAD**
HTTP方法既非GET，也非HEAD。 Dispatcher假設輸出包含不應進行快取的動態資料。
* **無法快取：包含查詢字串的要求**\
   要求包含查詢字串。 Dispatcher假設輸出取決於指定的查詢字串，因此不會進行快取。
* **無法快取：會話管理器未驗證**\
   伺服器場的快取由會話管理器管理(配置包含 `sessionmanagement` 節點)，而請求中未包含適當的驗證資訊。
* **無法快取：請求包含授權**\
   不允許該場快取輸出( `allowAuthorized 0`)，且要求包含驗證資訊。
* **無法快取：target是目錄**\
   目標檔案是目錄。 這可能會指出某些概念錯誤，例如，如果請求 `/test.html/a/file.ext` 會先出現且包含可快取的輸出，則dispatcher將無法快取後續請求的輸出，以便 `/test.html`.
* **無法快取：要求URL的尾端斜線**\
   請求URL有尾斜線。
* **無法快取：請求URL不在快取規則中**\
   伺服器陣列的快取規則會明確拒絕快取某些要求URL的輸出。
* **無法快取：授權檢查程式拒絕訪問**\
   伺服器場的授權檢查器拒絕訪問快取的檔案。
* **無法快取：會話無效**
伺服器場的快取由會話管理器管理(配置包含 `sessionmanagement` 節點)，且使用者的工作階段不再有效。
* **無法快取：回應包含`no_cache`**
遠程伺服器返回 
`Dispatcher: no_cache` 標題，禁止dispatcher快取輸出。
* **無法快取：回應內容長度為零**
回應的內容長度為零；dispatcher不會建立零長度的檔案。
