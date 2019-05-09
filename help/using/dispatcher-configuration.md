---
title: 設定Dispatcher
seo-title: 設定Dispatcher
description: 瞭解如何設定Dispatcher。
seo-description: 瞭解如何設定Dispatcher。
uuid: 253ef0f7-2491-4cec-ab22-97439df29 fd6
cmgrlastmodified: 01.11.2007082929[ahheimoz]
pageversionid: '1193211344162'
topic-tags: dispatcher
content-type: 引用
discoiquuid: affee8e-bb34-42a-7a5 e-b7 d0 e848391 a
translation-type: tm+mt
source-git-commit: bd8fff69a9c8a32eade60c68fc75c3aa411582af

---


# 設定Dispatcher{#configuring-dispatcher}

>[!NOTE]
>
>Dispatcher版本與AEM獨立。如果您關注Dispatcher文件的連結(內嵌於舊版AEM的文件中)，可能會重新導向至此頁面。

以下章節說明如何設定Dispatcher的各個方面。

## 支援IPv和IPv6 {#support-for-ipv-and-ipv}

AEM和Dispatcher的所有元素都可安裝在IPv和IPv網路中。請參閱 [IPv和IPv6](https://helpx.adobe.com/experience-manager/6-3/sites/deploying/using/technical-requirements.html#AdditionalPlatformNotes)。

## Dispatcher組態檔 {#dispatcher-configuration-files}

根據預設，Dispatcher組態會儲存在 `dispatcher.any` 文字檔案中，不過您可以在安裝期間變更此檔案的名稱和位置。

設定檔案包含一系列可控制Dispatcher行為的單一值或多值屬性：

* 屬性名稱前面加上斜線 `/`。
* 多值屬性會使用括號括住子項項目 `{ }`。

範例組態的結構如下：

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

您可以包含其他對設定貢獻的檔案：

* 如果您的組態檔案很大，可以將它分割為幾個較小的檔案(更容易管理)，然後包含這些檔案。
* 包含自動產生的檔案。

例如，若要加入檔案MyFARE，/farms設定中的任何項目都會使用下列程式碼：

```xml
/farms
  {
  $include "myFarm.any"
  }
```

使用星號(「*」)做為萬用字元，指定要包含的檔案範圍。

例如，如果檔案 `farm_1.any` 包含 `farm_5.any` 一到五個農場的設定，您可以將其納入如下：

```xml
/farms
  {
  $include "farm_*.any"
  }
```

## 使用環境變數 {#using-environment-variables}

您可以在dispatcher的字串值屬性中使用環境變數，而不是硬式編碼值。若要包含環境變數的值，請使用格式 `${variable_name}`。

例如，如果dispatcher與快取目錄位於相同目錄中，則可使用 [docroot](dispatcher-configuration.md#main-pars-title-30) 屬性的下列值：

```xml
/docroot "${PWD}/cache"
```

另例如，如果您建立名為 `PUBLISH_IP` 儲存AEM發佈例項主機名稱的環境變數，則可使用下列 [/renders](dispatcher-configuration.md#main-pars-127-25-0008) 屬性的組態：

```xml
/renders {
  /0001 {
    /hostname "${PUBLISH_IP}"
    /port "8443"
  }
}
```

## 命名Dispatcher例項 {#naming-the-dispatcher-instance-name}

使用 `/name` 屬性來指定唯一名稱，以識別您的Dispatcher例項。`/name` 屬性是組態結構中的頂層屬性。

## Defining農場 {#defining-farms-farms}

`/farms` 屬性定義一或多組Dispatcher行為，每一組都與不同網站或URL相關聯。`/farms` 屬性可以包含單一農場或多個農場：

* 當您想要Dispatcher處理您的所有網頁或網站時，請使用單一農場。
* 當網站或不同網站的不同區域需要不同的Dispatcher行為時，建立多個農場。

`/farms` 屬性是組態結構中的頂層屬性。若要定義農場，請新增子屬性至 `/farms` 屬性。使用屬性名稱，可唯一識別Dispatcher例項中的農場。

`/*farmname*` 屬性為多值，並包含其他定義Dispatcher行為的屬性：

* 農場所套用之頁面的URL。
* 用於轉換文件的一或多個服務URL(通常是AEM發佈例項)。
* 用於負載平衡多個文件轉譯器的統計資料。
* 其他數個行為，例如快取和何處。

值可以包含任何字母數字(a-z、0-9)字元。The following example shows the skeleton definition for two farms named `/daycom` and `/docsdaycom`:

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
>如果您使用多個轉譯農場，則會以下拉式評估清單。這在定義網站 [的虛擬主機](dispatcher-configuration.md#main-pars-117-15-0006) 時特別有用。

每個農場屬性都可以包含下列子屬性：

| 屬性名稱 | 說明 |
|--- |--- |
| [/homepage](#specify-a-default-page-iis-only-homepage) | 預設首頁(選用)(僅限IIS) |
| [/clientheaders](#specifying-the-http-headers-to-pass-through-clientheaders) | 來自用戶端HTTP要求的標題以通過。 |
| [/virtualhosts](#identifying-virtual-hosts-virtual-hosts) | 此農場的虛擬主機。 |
| [/sessionmanagement](#enabling-secure-sessions-session-management) | 支援工作階段管理和驗證。 |
| [/renders](#defining-page-renderers-renders) | 提供轉譯頁面(通常是AEM發佈例項)的伺服器。 |
| [/filter](#configuring-access-to-content-filter) | 定義Dispatcher允許存取的URL。 |
| [/vanity_urls](#enabling-access-to-vanity-urls-vanity-urls) | 設定存取虛名URL。 |
| [/propagateSyndPost](#forwarding-syndication-requests-propagate-syndpost) | 支援轉送請求的轉送。 |
| [/cache](#configuring-the-dispatcher-cache-cache) | 設定快取行為。 |
| [/statistics](#configuring-load-balancing-statistics) | 定義負載平衡計算的統計類別。 |
| [/stickyConnectionsFor](#identifying-a-sticky-connection-folder-sticky-connections-for) | 包含自黏文件的資料夾。 |
| [/health_check](#specifying-a-health-check-page) | 用來判斷伺服器可用性的URL。 |
| [/retryDelay](#specifying-the-page-retry-delay) | 重試失敗連線之前的延遲。 |
| [/unavailablePenalty](#reflecting-server-unavailability-in-dispatcher-statistics) | 影響負載平衡計算統計資料的懲罰。 |
| [/failover](#using-the-fail-over-mechanism) | 當原始請求失敗時，重新傳送請求至不同的轉譯。 |

## 指定預設頁面(IIS)-/homepage {#specify-a-default-page-iis-only-homepage}

>[!CAUTION]
>
>`/homepage`參數(僅IIS)不再運作。您應改用 [IIS URL重寫模組](https://docs.microsoft.com/en-us/iis/extensions/url-rewrite-module/using-the-url-rewrite-module)。
>
>如果您使用Apache，則應使用 `mod_rewrite` 模組。如需有關Apache2.4的 `mod_rewrite`[資訊，請參閱Apache網站文件](https://httpd.apache.org/docs/current/mod/mod_rewrite.html)。使用 `mod_rewrite`時，建議您使用標幟** [&#39;pasthrough| PT&#39;(傳遞至下一個處理常式)](https://helpx.adobe.com/dispatcher/kb/DispatcherModReWrite.html)**強制重寫引擎將 `uri` 內部 `request_rec` 結構的欄位設定為 `filename` 欄位的值。

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

## 指定HTTP標題要通過 {#specifying-the-http-headers-to-pass-through-clientheaders}

`/clientheaders` 屬性定義Dispatcher從用戶端HTTP請求傳遞至轉譯器(AEM實例)的HTTP標題清單。

依預設，Dispatcher會將標準HTTP標題轉送至AEM實例。在某些情況下，您可能想要轉寄其他標題，或移除特定標題：

* 新增標題(例如自訂標題)，您的AEM實例會在HTTP要求中預期。
* 移除僅與Web伺服器相關的標題(例如驗證標題)。

如果您自訂要通過的標題集，您必須指定完整的標題清單，包括通常包含在內的標題清單。

例如，處理發佈例項的頁面啓動要求的Dispatcher例項需要區段中的 `PATH` 標題 `/clientheaders` 。`PATH` 標題可啓用複製代理程式與發送器之間的通訊。

下列程式碼為下列項目的範例設定 `/clientheaders`：

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

`/virtualhosts` 此屬性定義Dispatcher接受此農場的所有主機名稱/URI組合清單。您可以使用星號(「*」)字元作為萬用字元。/ `virtualhosts` 屬性的值使用下列格式：

```xml
[scheme]host[uri][*]
```

* `scheme`: (Optional) Either `https://` or `https://.`
* `host`：主機的名稱或IP位址，以及必要的連接埠號碼。(請參閱「 [https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.23)](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.23)
* `uri`：(可選)資源的路徑。

下列範例設定控制MyCompany.com和. ch網域的請求，以及MySubDivision的所有網域：

```xml
   /virtualhosts
    {
    "www.myCompany.com"
    "www.myCompany.ch"
    "www.mySubDivison.*"
    }
```

下列設定會處理 *所有* 請求：

```xml
   /virtualhosts
    {
    "*"
    }
```

### 解決虛擬主機 {#resolving-the-virtual-host}

當Dispatcher收到HTTP或HTTPS要求時，會找出最符合要求的 `host,``uri`虛擬主機值， `scheme` 以及要求標題。Dispatcher會依下列順序評估 `virtualhosts` 屬性中的值：

* Dispatcher會從dispatcher. any檔案中的最低農場和進度開始。
* 對於每個農場，Dispatcher會從 `virtualhosts` 屬性中最上方的值開始，並向下瀏覽值清單。

Dispatcher以下列方式找出最佳符合的虛擬主機值：

* 第一個符合此要求的虛擬主機，與 `host`要求 `scheme`的所有 `uri` 三個相符。
* 如果 `virtualhosts` 沒有任何值和 `scheme``uri` 與請求 `scheme` 相符的 `uri` 部分，則會使用符合請求的 `host` 第一個虛擬主機。
* 如果沒有 `virtualhosts` 任何值的主機部分符合要求，則會使用最上方農場的最上方虛擬主機。

因此，您應將預設虛擬主機置於傳送caper. any檔案最上方農場 `virtualhosts` 的屬性上方。

### 虛擬主機解析度範例 {#example-virtual-host-resolution}

下列範例代表來自dispatcher的程式片段。任何定義兩個Dispatcher農場的檔案，而且每個農場定義 `virtualhosts` 一個屬性。

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

下表顯示針對指定HTTP請求解決的虛擬主機：

| 請求URL | 已解決虛擬主機 |
|---|---|
| `https://www.mycompany.com/products/gloves.html` | `www.mycompany.com/products/*;` |
| `https://www.mycompany.com/about.html` | `www.mycompany.com` |

## 啓用安全會話-/sessionmanagement {#enabling-secure-sessions-sessionmanagement}

>[!CAUTION]
>
>`/allowAuthorized`**必須** 設定 `"0"` 在 `/cache` 區段中，才能啓用此功能。

建立安全作業以存取演算農場，讓使用者必須登入才能存取農場中的任何頁面。登入後，使用者可以存取農場中的所有頁面。如需有關搭配使用此功能的詳細資訊，請參閱 [建立封閉使用者群組](https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/cug.html#CreatingTheUserGroupToBeUsed) 。

`/sessionmanagement` 屬性是一個子屬性 `/farms`。

>[!CAUTION]
>
>如果網站的區段使用不同的存取要求，您必須定義多個農場。

**/sessionmanagement** 有數個子參數：

**/directory** (強制)

儲存工作階段資訊的目錄。如果目錄不存在，則會建立此目錄。

**/encode** (選用)

作業資訊的編碼方式。使用md演算法對加密使用md演算法或十六進位編碼的「十六進位」。如果您加密作業資料，存取檔案系統的使用者無法讀取工作階段內容。預設值為「md5」。

**/header** (選用)

儲存授權資訊的HTTP標題或Cookie的名稱。如果您將資訊儲存在http標題中，請使用 `HTTP:<*header-name*>`。若要將資訊儲存在Cookie中，請使用 `Cookie:<header-name>`。如果您未指定值 `HTTP:authorization` ，則會使用此值。

**/timeout** (選用)

秒數的秒數，直到作業逾時為止。如果未指定「800」，則會在使用者最後一個要求超過13分鐘後逾時。

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

## 定義頁面演算程式 {#defining-page-renderers-renders}

/renders屬性定義Dispatcher傳送請求以轉譯文件的URL。下列範例 `/renders` 區段識別單一AEM實例用於演算：

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

下列範例/renders區段識別與Dispatcher在相同電腦上執行的AEM實例：

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

下列範例/renders區段在兩個AEM實例之間均分轉譯請求：

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

### 演算選項 {#renders-options}

**/timeout**

指定存取AEM實例以毫秒為單位的連線逾時。預設值為「0」，造成Dispatcher無限期等候。

**/receiveTimeout**

指定允許回應的毫秒數。預設值為「6000」，導致Dispatcher等候10分鐘。「0」設定會完全消除逾時。\
如果在剖析回應標題時到達逾時，則會傳回HTTP狀態504(不當閘道)。如果在讀取回應主體時到達逾時，Dispatcher會傳回對用戶端不完整的回應，但刪除任何可能寫入的快取檔案。

**/ipv4**

指定Dispatcher使用 `getaddrinfo` 函數(for IPv6)或 `gethostbyname` 函數(for IPv4)來取得演算的IP位址。會使用的值 `getaddrinfo` 為0。值會被使用 `gethostbyname` 。預設值為0。

getaddrinfo函數會傳回IP位址清單。Dispatcher會重復位址清單，直到它建立TCP/IP連線為止。因此，當演算主機名稱與getdrinfo函數相關聯時，會傳回一份一律位於相同順序的IP位址清單，因此piv屬性很重要，當演算主機名稱與之關聯時。在此情況下，您應使用getoshybiname函數，以便Dispatcher連接的IP位址是randomized的。

Amazon Elastic Load Balancing(ELB)是一種回應getdrinfo的服務，可使用相同的IP位址清單來回應getdrinfo。

**/secure**

如果 `/secure` 屬性的值為「1」，則Dispatcher會使用HTTPS與AEM實例通訊。如需詳細資訊，請參閱 [「設定Dispatcher使用SSL](dispatcher-ssl.md#configuring-dispatcher-to-use-ssl)」。

**/always-resolve**

有了Dispatcher **4.1.6**版，您可以設定 `/always-resolve` 屬性如下：

* 設為「1」時，它將會解決每個要求上的主機名稱(Dispatcher絕不會快取任何IP位址)。由於取得每個要求的主機資訊所需的額外呼叫，可能會造成輕微效能影響。
* 如果未設定屬性，預設會快取IP位址。

此外，此屬性可用於動態IP解析度問題，如下列範例所示：

```xml
/rend {
  /0001 {
     /hostname "host-name-here"
     /port "4502"
     /ipv4 "1"
     /always-resolve "1"
     }
  }
```

## 設定內容存取權 {#configuring-access-to-content-filter}

使用 `/filter` 區段來指定Dispatcher接受的HTTP請求。所有其他請求都會傳回至含有404錯誤代碼(找不到頁面)的Web伺服器。如果 `/filter` 沒有區段，則會接受所有請求。

**注意：** 一律拒絕 [對statfile](dispatcher-configuration.md#main-pars-title-28) 的請求。

>[!CAUTION]
>
>請參閱 [Dispatcher Security檢查清單](security-checklist.md) ，以瞭解使用Dispatcher限制存取時的詳細考量。此外，請參閱 [AEM安全性Chocklist](https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/security-checklist.html) ，以取得有關您AEM安裝的其他安全性詳細資訊。

/filter區段包含一系列規則，可根據HTTP要求的請求列部分中的模式拒絕或允許存取內容。您應為您的/filter區段使用一個模糊策略：

* 首先，拒絕存取所有內容。
* 允許存取內容。

### 定義篩選 {#defining-a-filter}

`/filter` 區段中的每個項目都包含一個類型和一個符合請求列特定元素或整個請求行的模式。每個篩選器可包含下列項目：

* **類型**：指出 `/type` 是否允許或拒絕符合模式的請求。值可以是 `allow` 或 `deny`。

* **請求列的元素：** 根據 `/method`HTTP要求的請求行部分的特定部分，包括 `/url`、 `/query`或 `/protocol` 建立篩選請求的模式。篩選請求行元素(而非整個請求行)是偏好篩選方法。

* **glob屬性**： `/glob` 此屬性可用來符合HTTP要求的整個要求列。

如需/glob屬性的詳細資訊，請參閱 [「設計glob屬性的圖樣](#designing-patterns-for-glob-properties)」。在/glob屬性中使用萬用字元字元的規則也會套用至請求行的對應元素的模式。

>[!NOTE]
>
>在Dispatcher4.2.0版中，已新增多項篩選組態和記錄功能的增強功能：
>
>* [支援POXX規則運算式](dispatcher-configuration.md#main-pars-title-1996763852)
>* [支援篩選請求URL的其他元素](dispatcher-configuration.md#main-pars-title-694578373)
>* [追蹤記錄](dispatcher-configuration.md#main-pars-title-1950006642)
>



#### HTTP請求的請求行部分 {#the-request-line-part-of-http-requests}

HTTP/1.1定義 [要求列](https://www.w3.org/Protocols/rfc2616/rfc2616-sec5.html) 如下：

*方法要求-URI HTTP-Version*&lt; CRLF&gt;

&lt; CRLF&gt;字元會重新產生歸位，再加上行饋送。以下範例為當用戶端請求Geometrixx-UTouts網站頁面時的請求列：

GET/content/geometrixx-outdoors/en.htmlHTTP.1.1&lt; CRLF&gt;

您的模式必須考量請求列和&lt; CRLF&gt;字元中的空格字元。

#### 篩選範例：拒絕全部 {#example-filter-deny-all}

下列範例篩選區段會造成Dispatcher拒絕對所有檔案的請求。您應拒絕存取所有檔案，然後允許存取特定區域。

```xml
  /0001  { /glob "*" /type "deny" }
```

要求明確拒絕區域會傳回404錯誤代碼(找不到頁面)。

#### 篩選範例：拒絕特定區域的運作方式 {#example-filter-deny-acess-to-specific-areas}

篩選器也允許您拒絕對不同元素(例如ASP頁面和發佈例項中的敏感區域)的存取。下列篩選條件可讓您存取ASP頁面：

```xml
/0002  { /type "deny" /url "*.asp"  }
```

#### 篩選範例：啓用POST請求 {#example-filter-enable-post-requests}

下列範例篩選可讓您依POST方法提交表單資料：

```xml
/filter {
    /0001  { /glob "*" /type "deny" }
    /0002 { /type "allow" /method "POST" /url "/content/[.]*.form.html" }
}
```

#### 篩選範例：允許存取工作流程控制台 {#example-filter-allow-access-to-the-workflow-console}

下列範例顯示用來拒絕外部存取工作流程控制台的篩選器：

```xml
/filter {
    /0001  { /glob "*" /type "deny" }
    /0002  {  /type "allow"  /url "/libs/cq/workflow/content/console*"  }
}
```

如果您的發佈例項使用網頁應用程式內容(例如發佈)，這也可以新增至您的篩選器定義。

```xml
/0003   { /type "deny"  /url "/publish/libs/cq/workflow/content/console/archive*"  }
```

如果您仍需要存取受限制區域內的單一頁面，您可以允許存取這些頁面。例如，若要允許存取工作流程控制台中的「封存」標籤，請新增下一節：

```xml
/0004  { /type "allow"  /url "/libs/cq/workflow/content/console/archive*"   }
```

>[!NOTE]
>
>當套用多個篩選器模式時，套用的最後一個篩選模式有效。

#### 篩選範例：使用規則運算式 {#example-filter-using-regular-expressions}

此篩選器可使用規則運算式在非公開的內容目錄中啓用擴充功能，並在此處定義單引號：

```xml
/005  {  /type "allow" /extension '(css|gif|ico|js|png|swf|jpe?g)' }
```

#### 篩選範例：篩選請求URL的其他元素 {#example-filter-filter-additional-elements-of-a-request-url}

dispatcher4.2.0引進的增強功能之一，就是篩選請求URL的其他元素。引進的新元素包括：

* 路徑
* 選擇器
* 擴充功能
* 字尾

您可以將相同名稱的屬性新增至篩選規則來設定： `/path``/selectors``/extension``/suffix` 和分別。

以下是一個規則範例，可使用路徑、選擇器和擴充功能的篩選器封鎖 `/content` 路徑及其子樹狀結構中的內容抓取：

```xml
/006 {
        /type "deny"
        /path "/content/*"
        /selectors '(feed|rss|pages|languages|blueprint|infinity|tidy)'
        /extension '(json|xml|html)'
        }
```

### /filter區段範例 {#example-filter-section}

設定Dispatcher時，您應盡可能限制外部存取。下列範例為外部訪客提供最少的存取權：

* `/content`
* 不同的內容，例如設計和用戶端程式庫；例如：

   * `/etc/designs/default*`
   * `/etc/designs/mydesign*`

建立篩選器後 [，測試頁面存取權](dispatcher-configuration.md#main-pars-title-19) 以確保AEM實例是安全的。

Dispatcher的下列/filter區段。任何檔案都可作為 [Dispatcher組態](dispatcher-configuration.md) 檔的基礎。

此範例是以Dispatcher提供的預設組態檔案為基礎，並作為在生產環境中使用的範例。預先加上#的項目會停用(註解)，因此如果您決定啓用任一個項目(只要將#上的#移除)，這可能會造成安全性影響。

您應拒絕存取所有項目，然後允許存取特定(有限)元素：

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
>與Apache搭配使用時，請根據Dispatcher模組的dispatcheruseProcessurel屬性來設計您的篩選器URL圖樣。(請參閱 [Apache Web Server-設定您的Apache Web Server for Dispatcher)](dispatcher-install.md#main-pars-55-35-1022)。

>[!NOTE]
>
>有關動態媒體的篩選0030和0031適用於AEM6.0及更高版本。

如果您選擇延長存取權，請考慮下列建議：

* 如果您使用CQ5.4版或更早版本，外部存取權限 `/admin`*應一律* 完全停用。

* 允許存取檔案時必須小心謹慎 `/libs`。存取權應以個人為準。
* 拒絕對複製設定的存取，使其無法查看：

   * `/etc/replication.xml*`
   * `/etc/replication.infinity.json*`

* 拒絕存取Google Gadget反向proxy：

   * `/libs/opensocial/proxy*`

視您的安裝而定，必須有其他資源可 `/libs``/apps` 供使用。您可以將 `access.log` 檔案當做一種判斷外部存取資源的方法。

>[!CAUTION]
>
>存取主機和目錄可能會對生產環境造成安全性風險。除非您有明確的猜測，否則它們應保持停用(留言)。

>[!CAUTION]
>
>如果您 [在發佈環境](https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/reporting.html#UsingReportsinaPublishEnvironment) 中使用報表，則應設定Dispatcher拒絕外部訪客 `/etc/reports` 的存取權。

### 限制查詢字串 {#restricting-query-strings}

由於Dispatcher版本4.1.5，請使用 `/filter` 區段來限制查詢字串。強烈建議您透過 `allow` 篩選元素明確允許查詢字串並排除一般津貼。

單一項目可以具有 *glob* 或部分 *方法*、*URL*、*查詢* 和 *版本，* 但不能同時組合。下列範例允許 `a=*` 查詢字串，並為解析為 `/etc` 節點的URL提供所有其他的查詢字串：

```xml
/filter {
 /0001 { /type "deny" /method "POST" /url "/etc/*" }
 /0002 { /type "allow" /method "GET" /url "/etc/*" /query "a=*" }
}
```

>[!NOTE]
>
>如果規則包含a `/query`，則只會匹配包含查詢字串並符合提供的查詢模式的請求。
>
>在上述範例中， `/etc` 如果請求沒有查詢字串，則需要下列規則：


```xml
/filter {  
>/0001 { /type "deny" /method “*" /url "/path/*" }  
>/0002 { /type "allow" /method "GET" /url "/path/*" }  
>/0003 { /type “deny" /method "GET" /url "/path/*" /query "*" }  
>/0004 { /type "allow" /method "GET" /url "/path/*" /query "a=*" }  
}  
```

### 測試Dispatcher安全性 {#testing-dispatcher-security}

Dispatcher篩選器應封鎖AEM發佈例項上的下列頁面和指令碼存取權。使用網頁瀏覽器嘗試開啓網站訪客的下列頁面，並確認是否傳回程式碼404。如果取得其他結果，請調整您的篩選器。

請注意，您應該會看到/content/add_valid_page.html的正常頁面演算嗎？debug=版面。


* /admin
* /system/console
* /dav/crx.default
* /crx
* /bin/crxde/logs
* /jcr：system/jcr：VersionStorage. json
* /_jcr_system/_jcr_versionStorage.json
* /libs/wcm/core/content/siteadmin.html
* /libs/collab/core/content/admin.html
* /libs/cq/ui/content/dumplibs.html
* /var/linkchecker.html
* /etc/linkchecker.html
* /home/users/a/admin/profile.json
* /home/users/a/admin/profile.xml
* /libs/cq/core/content/login.json
* /content/../libs/foundation/components/text/text.jsp
* /content/.{.}/libs/foundation/components/text/text/text. jsp
* /apps/sling/config/org.apache.felix.webconsole.internal.servlet.OsgiManager.config/jcr%3acontent/jcr%3adata
* /libs/foundation/components/primary/cq/workflow/components/participants/json.GET.servlet
* /content.pages.json
* /content.languages.json
* /content.blueprint.json
* /content.-1. json
* /content.10.json
* /content.infinity.json
* /content.tidy.json
* /content.tidy。-blubber. json
* /content/dam.tidy。-100. json
* /content/content/geometrixx.sitemap.txt
* /content/add_valid_page.query.json？陳述式=//*
* /content/add_valid_page.qu%65ry. js%6fn？陳述式=//*
* /content/add_valid_page.query.json？陳述式=//*[@ transportPassword]/(@ TransportPassword%20|%20@transportUri%20|%20@transportUser)
* /content/add_valid_path_to_a_page/_jcr_content.json
* /content/add_valid_path_to_a_page/jcr：content. json
* /content/add_valid_path_to_a_page/_jcr_content.feed
* /content/add_valid_path_to_a_page/jcr：content. feed
* /content/add_valid_path_to_a_page/pagename。_ jcr_ content. peet
* /content/add_valid_path_to_a_page/pagename.jcr：content. feed
* /content/add_valid_path_to_a_page/pagename.docview.xml
* /content/add_valid_path_to_a_page/pagename.docview.json
* /content/add_valid_path_to_a_page/pagename.sysview.xml
* /etc.xml
* /content.feed.xml
* /content.rss.xml
* /content.feed.html
* /content/add_valid_page.html？debug=版面配置
* /projects
* /tagging
* /etc/replication.html
* /etc/cloudservices.html
* /歡迎

在終端機或命令提示中執行下列命令，以判斷是否啓用匿名寫入存取權。您應該無法將資料寫入節點。

`curl -X POST "https://anonymous:anonymous@hostname:port/content/usergenerated/mytestnode"`

在終端機或命令提示中執行下列命令，嘗試使Dispatcher快取失效，並確保您獲得程式碼404回應：

`curl -H "CQ-Handle: /content" -H "CQ-Path: /content" https://yourhostname/dispatcher/invalidate.cache`

## 啓用存取虛名URL {#enabling-access-to-vanity-urls-vanity-urls}

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (sbroders@adobe.com)
Last Modified Date: 2015-03-25T14:23:05.185-0400

<p style="font-family: tahoma, arial, helvetica, sans-serif; font-size: 12px;">For https://jira.corp.adobe.com/browse/DOC-4812</p> 
<p style="font-family: tahoma, arial, helvetica, sans-serif; font-size: 12px;">The com.adobe.granite.dispatcher.vanityurl.content package needs to be made public before publishing this contnet.</p>

 -->

設定Dispatcher以啓用對您CQ或AEM頁面設定之虛名URL的存取。

啓用虛名URL存取權時，Dispatcher會定期呼叫執行於演算例項上的服務，以取得虛名URL清單。Dispatcher會將此清單儲存在本機檔案中。由於 `/filter` 區段中的篩選條件拒絕頁面的請求，Dispatcher會諮詢虛名URL清單。如果拒絕的URL位於清單中，Dispatcher允許存取虛名URL。

若要啓用虛名URL的存取權，請將 `/vanity_urls` 區段新增 `/farms` 至區段，類似下列範例：

```xml
 /vanity_urls {
      /url "/libs/granite/dispatcher/content/vanityUrls.html"
      /file "/tmp/vanity_urls"
      /delay 300
 }
```

`/vanity_urls` 區段包含下列屬性：

* `/url`：在演算例項上執行的虛名URL服務路徑。此屬性的值必須 `"/libs/granite/dispatcher/content/vanityUrls.html"`為。

* `/file`：Locatcher儲存虛名URL清單的本機檔案路徑。請確定Dispatcher具有對此檔案的寫入存取權。
* `/delay`：(秒)呼叫虛名URL服務之間的時間。

>[!NOTE]
>
>如果您的演算是AEM實例，您必須安裝 [VanityURL-Components](https://www.adobeaemcloud.com/content/marketplace/marketplaceProxy.html?packagePath=/content/companies/public/adobe/packages/cq600/component/vanityurls-components) 套件以安裝虛名URL服務。(請參閱 [登入套件共用](https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/package-manager.html#SigningIntoPackageShare))。

使用下列程序可讓存取虛名URL。

1. 如果您的演算服務是AEM實例，請在發佈例項上安裝com. adobe. granite. vanitcher. vanityurl. content套件(請參閱上述附註)。
1. 對於您已為AEM或CQ頁面設定的每個虛名URL，請確定 ` [/filter](dispatcher-configuration.md#main-pars_134_32_0009)` 設定已拒絕URL。如有必要，請新增拒絕URL的篩選器。
1. 新增 `/vanity_urls` 區段 `/farms`。
1. 重新啓動Apache網頁伺服器。

## 轉送教學請求-/propagateSyndPost {#forwarding-syndication-requests-propagatesyndpost}

教學請求通常僅適用於Dispatcher，因此，依預設，它們不會傳送至轉譯器(例如AEM實例)。

如有必要，請將「/propagateSyndPost」屬性設為「1」，然後轉送至「Dispatcher」。如果設定，您必須確定篩選區段中不拒絕POST請求。

## 設定Dispatcher快取-/cache {#configuring-the-dispatcher-cache-cache}

`/cache` 此區段控制Dispatcher如何快取文件。設定數個子屬性來實施您的快取策略：


* /docroot
* /statfile
* /serveStaleOnError
* /allowAuthorized
* /多項規則
* /statfileslevel
* /invalidate
* /invalidateHandler
* /allowedClients
* /ignoreUrlParams
* /headers
* /mode
* /gracePeriod


快取區段的外觀可能如下所示：

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
>對於權限感應快取，請閱讀 [快取安全內容](permissions-cache.md)。

### 指定快取目錄 {#specifying-the-cache-directory}

`/docroot` 屬性會識別儲存快取檔案的目錄。

>[!NOTE]
>
>此值必須與Web伺服器的文件根目錄完全相同，以便Dispatcher和Web伺服器處理相同的檔案。\
>當使用dispatcher快取檔案時，Web伺服器會負責提供正確的狀態代碼，因此也就很重要。

如果您使用多個農場，每個農場都必須使用不同的文件根目錄。

### 命名統計資料 {#naming-the-statfile}

`/statfile` 屬性會識別檔案以用作statfile。Dispatcher會使用此檔案來註冊最新內容更新的時間。statfile可以是Web伺服器上的任何檔案。

statfile沒有內容。更新內容時，Dispatcher會更新時間戳記。預設的statfile會命名為. stat並儲存在docroot中。Dispatcher會封鎖對statfile的存取。

>[!NOTE]
>
>`/statfileslevel` 如果已設定，則Dispatcher會忽略 `/statfile` 屬性並使用. stat作為名稱。

### 發生錯誤時提供過時文件 {#serving-stale-documents-when-errors-occur}

`/serveStaleOnError` 當演算伺服器傳回錯誤時，屬性會控制Dispatcher是否傳回失效文件。依預設，當中繼檔案被觸控並失效快取內容時，Dispatcher會在下次請求時刪除快取內容。

如果 `/serveStaleOnError` 設為「1」，Dispatcher不會刪除快取中的無效內容，除非演算伺服器傳回成功回應。來自AEM或連線逾時的xx回應會造成Dispatcher提供過時內容，並回應與HTTP狀態114(重新驗證失敗)。

### 使用驗證時快取 {#caching-when-authentication-is-used}

`/allowAuthorized` 屬性會控制是否快取包含下列任何驗證資訊的請求：

* `authorization` 標題。
* 名為 `authorization`Cookie的Cookie。
* 名為 `login-token`Cookie的Cookie。

根據預設，不會快取包含此驗證資訊的請求，因為當已快取的文件傳回用戶端時，不會執行驗證。此設定可防止Dispatcher將快取的文件提供給沒有必要權限的使用者。

不過，如果您的需求允許快取已驗證文件，請將「/allowAuthorized」設為一：

`/allowAuthorized "1"`

>[!NOTE]
>
>若要啓用工作階段管理(使用 `/sessionmanagement` 屬性)， `/allowAuthorized` 必須將屬性設 `"0"`為。

### 指定文件至快取 {#specifying-the-documents-to-cache}

`/rules` 屬性會控制根據文件路徑快取的文件。不論/rules屬性為何，Dispatcher都不會在下列情況下快取文件：

* 如果請求URI包含問號(」？」)。\
   這通常表示動態頁面，例如不需要快取的搜尋結果。
* 副檔名遺失。\
   網頁伺服器需要擴充功能來判斷文件類型(MIME類型)。
* 已設定驗證標題(可設定)
* 如果AEM實例回應下列標題：

   * `no-cache`
   * `no-store`
   * `must-revalidate`

>[!NOTE]
>
>DET或HEAD(適用於HTTP標題)方法可由Dispatcher使用。如需回應標題快取的其他資訊，請參閱 [快取HTTP回應標題](dispatcher-configuration.md#caching-http-response-headers) 區段。

/rules屬性中的每個項目都包含 [glob](#designing-patterns-for-glob-properties) 模式和類型：

* glob圖樣可用來比對文件的路徑。
* 類型會指出要快取符合glob模式的文件。值可以是允許(快取文件)或拒絕(一律顯示文件)。

如果您沒有動態頁面(除了上述規則已排除的頁面)，您也可以設定Dispatcher快取所有內容。The rules section for this look as follows：

```xml
/rules
  { 
    /0000  {  /glob "*"   /type "allow" }
  }
```

如需glob屬性的詳細資訊，請參閱 [「設計glob屬性的圖樣](#designing-patterns-for-glob-properties)」。

如果您的頁面有一些區段是動態的(例如新聞應用程式)或在封閉的使用者群組內，則可以定義例外：

>[!NOTE]
>
>無法快取已關閉的使用者群組，因為未檢查快取頁面的使用者權限。

```xml
/rules
  {
   /0000  { /glob "*" /type "allow" }
   /0001  { /glob "/en/news/*" /type "deny" }
   /0002  { /glob "*/private/*" /type "deny"  }   
  }
```

**壓縮**

在Apache Web伺服器上，您可以壓縮快取的文件。壓縮可讓Apache依照用戶端要求，以壓縮形式傳回文件。啓用Apache模組會自動完成壓縮 `mod_deflate`，例如：

```xml
AddOutputFilterByType DEFLATE text/plain
```

此模組預設為Apache2.x安裝。

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

### 依資料夾層級停用檔案 {#invalidating-files-by-folder-level}

使用 `/statfileslevel` 屬性，根據其路徑使快取的檔案失效：

* Dispatcher會從docroot資料夾建立 `.stat`每個資料夾中的檔案至您所指定的層級。docroot資料夾為level0。
* 觸控檔案會使檔案失效 `.stat` 。`.stat` 檔案的最後修改日期會與快取文件的最後修改日期進行比較。`.stat` 如果檔案較新，則會重新擷取文件。

* 當位於某個層級的檔案失效時，會 **將所有**`.stat` 檔案從docroot **移至** 失效檔案的層級，或是將其 `statsfilevel` 設定為較小的檔案。

   * 例如：如果您將 `statfileslevel` 屬性設為個，且某個檔案在層級失效，則將會觸控docroot到5的每 `.stat` 個檔案。繼續使用此範例，如果某個檔案在層級上失效。`stat` file from docroot to6will be touch(from `/statfileslevel = "6"`).

只有沿著路徑**的資源**會受到影響。請考慮下列範例：網站使用結構 `/content/myWebsite/xx/.` 如果您設定 `statfileslevel` 為3， `.stat`則會建立檔案：

* `docroot`
* `/content`
* `/content/myWebsite`
* `/content/myWebsite/*xx*`

When a file in `/content/myWebsite/xx` is invalidated then every `.stat` file from docroot down to `/content/myWebsite/xx`is touched. 這僅適用於範例 `/content/myWebsite/xx``/content/myWebsite/yy` 或 `/content/anotherWebSite`不適用。

>[!NOTE]
>
>傳送額外標題 `CQ-Action-Scope:ResourceOnly`可防止停用。這可用來清除特定資源，而不會使快取的其他部分失效。如需詳細資訊，請參閱 [此頁面](https://adobe-consulting-services.github.io/acs-aem-commons/features/dispatcher-flush-rules/index.html) 並 [手動使Dispatcher快取](https://helpx.adobe.com/experience-manager/dispatcher/using/page-invalidate.html) 失效。

>[!NOTE]
>
>如果您指定 `/statfileslevel` 屬性的值， `/statfile` 則會忽略屬性。

### 自動失效快取檔案 {#automatically-invalidating-cached-files}

`/invalidate` 屬性定義當內容更新時自動失效的文件。

自動失效後，Dispatcher不會在內容更新後刪除快取檔案，但在下次要求時檢查其有效性。快取中未自動失效的文件將保留在快取中，直到內容更新明確刪除它們為止。

自動失效通常用於HTML頁面。HTML頁面通常包含其他頁面的連結，因此很難判斷內容更新是否會影響頁面。為了確保更新內容時所有相關頁面都會失效，請自動使所有HTML頁面失效。下列設定會使所有HTML頁面失效：

```xml
  /invalidate
  {
   /0000  { /glob "*" /type "deny" }
   /0001  { /glob "*.html" /type "allow" }
  }
```

如需glob屬性的詳細資訊，請參閱 [「設計glob屬性的圖樣](#designing-patterns-for-glob-properties)」。

啓用/content/geometrixx/en時，此設定會造成下列活動：

* 所有檔案都具有圖樣en。*會從/content/geometrixx/資料夾中移除。
* /content/geometrixx/en/_jcr_content資料夾隨即移除。
* 所有符合/invalidate組態的其他檔案都不會立即刪除。當下一個請求發生時，就會刪除這些檔案。在我們的範例/content/geometrixx.html未刪除的情況下，請求/content/geometrixx.html時將會刪除它。

如果您提供自動產生的PDF和ZIP檔案供下載，您可能也必須自動使這些檔案失效。設定範例如下所示：

```xml
/invalidate
  {
   /0000 { /glob "*" /type "deny" }
   /0001 { /glob "*.html" /type "allow" }
   /0002 { /glob "*.zip" /type "allow" }
   /0003 { /glob "*.pdf" /type "allow" }
  }
```

AEM與Adobe Analytics整合，可在您的網站中的分析. siteCatalyst. js檔案中傳送組態資料。Example dispatcher。任何隨Dispatcher提供的檔案包含下列檔案的下列無效規則：

```xml
{
   /glob "*/analytics.sitecatalyst.js"  /type "allow"
}
```

### 使用自訂無效指令碼 {#using-custom-invalidation-scripts}

Social屬性可讓您定義由Dispatcher接收的每個無效要求所呼叫的指令碼。

系統會呼叫下列引數：

* 控點\
   無效的內容路徑
* 動作\
   複製動作(例如啓用、停用)
* 動作範圍\
   複製動作的範圍(除非傳送的標題是空的， `CQ-Action-Scope: ResourceOnly` 請參閱 [AEM中的停用快取頁面，以](page-invalidate.md) 瞭解詳細資訊)

這可用於涵蓋許多不同的使用案例，例如無效的其他應用程式快取，或處理頁面的外部URL及其位置與內容路徑不符的案例。

下方範例指令檔會記錄每個無效要求至檔案。

```xml
/invalidateHandler "/opt/dispatcher/scripts/invalidate.sh"
```

#### 範例無效處理常式指令碼 {#sample-invalidation-handler-script}

```shell
#!/bin/bash

printf "%-15s: %s %s" $1 $2 $3>> /opt/dispatcher/logs/invalidate.log
```

### 限制可清除快取的客戶 {#limiting-the-clients-that-can-flush-the-cache}

Social屬性定義允許清除快取的特定用戶端。全域圖樣會與IP相符。

下列範例：

1. 拒絕存取任何用戶端
1. 明確允許存取localhost

```xml
/allowedClients
  {
   /0001 { /glob "*.*.*.*"  /type "deny" }
   /0002 { /glob "127.0.0.1" /type "allow" }
  }
```

如需glob屬性的詳細資訊，請參閱 [「設計glob屬性的圖樣](#designing-patterns-for-glob-properties)」。

>[!CAUTION]
>
>建議您定義Social。
>
>如果尚未完成，任何用戶端都可以發出清除快取的呼叫；如果重復完成，則會嚴重影響網站效能。

### 忽略URL參數 {#ignoring-url-parameters}

`ignoreUrlParams` 此區段定義了在判斷頁面快取或從快取傳送時，哪些URL參數被忽略：

* 當請求URL包含所有已忽略的參數時，會快取頁面。
* 當請求URL包含一或多個未被忽略的參數時，不會快取頁面。

當頁面的參數被忽略時，第一次請求頁面時會快取頁面。無論請求中的參數值為何，都會對頁面的後續請求提供快取頁面。

若要指定哪些參數被忽略，請新增glob規則至 `ignoreUrlParams` 屬性：

* 若要忽略參數，請建立允許參數的glob屬性。
* 若要防止快取頁面，請建立拒絕參數的glob屬性。

下列範例會讓Dispatcher忽略「q」參數，因此會快取包含q參數的請求URL：

```xml
/ignoreUrlParams
{
    /0001 { /glob "*" /type "deny" }
    /0002 { /glob "q" /type "allow" }
}
```

使用範例 `ignoreUrlParams` 值，下列HTTP請求會使頁面快取，因為 `q` 參數被忽略：

```xml
GET /mypage.html?q=5
```

使用範例 `ignoreUrlParams` 值，下列HTTP請求會導致頁面 **不** 被快取，因為 `p` 參數不被忽略：

```xml
GET /mypage.html?q=5&p=4
```

如需glob屬性的詳細資訊，請參閱 [「設計glob屬性的圖樣](#designing-patterns-for-glob-properties)」。

### 快取HTTP回應標題 {#caching-http-response-headers}

>[!NOTE]
>
>此功能可與Dispatcher4.1.11版 **** 搭配使用。

`/headers` 此屬性可讓您定義要由Dispatcher快取的HTTP標題類型。在第一個對快取資源的請求中，所有符合已設定值之一的標題(請參閱以下設定範例)會儲存在快取檔案旁邊的個別檔案中。在後續請求快取資源時，儲存的標題會新增至回應。

以下為預設組態的範例：

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
>此外，請注意，不允許檔案全域字元。如需詳細資訊，請參閱 [「設計glob屬性的圖樣](#designing-patterns-for-glob-properties)」。

>[!NOTE]
>
>如果您需要Dispatcher來儲存並提供AEM的eTag回應標題，請執行下列動作：
>
>* 在 `/cache/headers`區段中新增標題名稱。
>* 在Dispatcher相關區段中新增下列 [Apache指令](https://httpd.apache.org/docs/2.4/mod/core.html#fileetag) ：
>



```xml
FileETag none
```

### Dispatcher快取檔案權限 {#dispatcher-cache-file-permissions}

`mode` 該屬性指定了哪些檔案權限套用至快取中的新目錄和檔案。此設定受到呼叫 `umask` 程序的限制。它是以下列一或多個值之總和建構的八位數數字：

* 0400允許擁有者讀取。
* 0200允許擁有者撰寫。
* 0100允許擁有者在目錄中搜尋。
* 0040允許群組成員讀取。
* 0020允許群組成員編寫。
* 0010允許群組成員在目錄中搜尋。
* 0004允許他人讀取。
* 0002允許他人編寫。
* 0001允許其他人在目錄中搜尋。

預設值為0755，可讓擁有者讀取、寫入或搜尋，以及群組及其他人讀取或搜尋。

### Rootling. stat檔案觸控 {#throttling-stat-file-touching}

使用預設 `/invalidate` 屬性，每個啓動都會有效地使所有 `.html` 檔案失效(當其路徑符合 `/invalidate` 區段時)。在流量相當大的網站上，後續啓動會增加後端的CPU負載。在這種情況下，您需要「節流」 `.stat` 檔案觸控，以維持網站的回應性。您可以使用 `/gracePeriod` 屬性來完成此動作。

`/gracePeriod` 屬性會定義過時的秒數，自動失敗的資源在最後一次啓動後仍可從快取中提供。您可以在設定中使用屬性，在此設定中，一批次啓動會重復失效整個快取。建議的值為秒。

如需詳細資訊，請參閱上述 `/invalidate` 章節和 `/statfileslevel`章節。

## 設定以時間為基礎的快取失效-/enableTTL {#configuring-time-based-cache-invalidation-enablettl}

如果設定， `enableTTL` 屬性會從後端評估回應標題，如果它們包含 `Cache-Control` 最大年齡或 `Expires` 日期，則會建立快取檔案旁邊的空白檔案，修改時間等於過期日期。當已快取的檔案經過請求後，會自動從後端請求修改時間。

您可以將此行新增至 `dispatcher.any` 檔案，以啓用此功能：

```xml
/enableTTL "1"
```

>[!NOTE]
>
>此功能可與Dispatcher4.1.11版 **** 搭配使用。

## 設定負載平衡-/statistics {#configuring-load-balancing-statistics}

`/statistics` 本節定義Dispatcher為每個演算的回應速度評分的檔案類別。Dispatcher會使用分數來決定要傳送請求的轉譯。

您建立的每個類別都會定義glob模式。Dispatcher會比較要求內容的URI，以判斷所請求內容的類別：

* 類別順序會決定與URI比較的順序。
* 符合URI的第一個類別模式是檔案的類別。不會評估類別模式。

Dispatcher最多支援個統計類別。如果定義超過個類別，則只會使用前個類別。

**演算選取範圍**

每次Dispatcher需要轉譯頁面時，它會使用下列演算法來選取演算：

1. 如果請求包含 `renderid` Cookie中的轉譯名稱，Dispatcher會使用該演算。
1. 如果請求不包含 `renderid` Cookie，則Dispatcher會比較演算統計資料：

   1. Dispatcher會判斷請求URI的目錄。
   1. Dispatcher會判斷哪個演算具有該類別的最低回應分數，並選取該演算。

1. 如果尚未選取演算，請使用清單中的第一個演算。

轉譯類別的分數是根據先前的回應時間，以及Dispatcher嘗試的先前失敗和成功連線。每次嘗試時，都會更新請求之URI類別的分數。

>[!NOTE]
>
>如果您不使用負載平衡，可以忽略此區段。

### 定義統計類別 {#defining-statistics-categories}

為您要保留演算選取範圍統計資料的每個文件類型定義類別。/statistics區段包含/categories區段。若要定義類別，請在具有下列格式的/categories區段下方新增一行：

`/name { /glob "pattern"}`

類別 `name` 必須是農場唯一的唯一類別。「 `pattern`[設計glob屬性](#designing-patterns-for-glob-properties) 」區段中的說明說明。

若要判斷URI的類別，Dispatcher會將URI與每個類別模式進行比較，直到找到相符項目為止。Dispatcher從清單和目錄中的第一個類別開始。因此，請先將類別放置在更具體的模式下。

例如，Dispatcher預設的dispatcher。任何檔案都會定義HTML類別和其他類別。HTML類別更具體，因此會先顯示：

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

### 反映Dispatcher統計資料中的伺服器不可用 {#reflecting-server-unavailability-in-dispatcher-statistics}

`/unavailablePenalty` 屬性會設定當連線至演算失敗時，套用至演算統計資料的時間(1/10)。Dispatcher會將時間新增至符合要求URI的統計類別。

例如，當AEM未執行(且不會監聽)或因網路相關問題而無法建立指定主機名稱/連接埠時，就會套用罰金。

`/unavailablePenalty` 屬性是 `/farm` 區段的直接子項( `/statistics` 區段的子系)。

如果 `/unavailablePenalty` 沒有屬性，則會使用「1」值。

```xml
/unavailablePenalty "1"
```

## 識別自黏連線資料夾-/stickyConnectionsFor {#identifying-a-sticky-connection-folder-stickyconnectionsfor}

`/stickyConnectionsFor` 屬性定義一個包含嚴格文件的檔案夾；這會使用URL存取。Dispatcher會將所有請求從單一使用者傳送至相同的轉譯例項。黏著連線可確保所有文件的作業資料呈現和一致。此機制會使用 `renderid` Cookie。

下列範例定義/products檔案夾的嚴格連線：

```xml
/stickyConnectionsFor "/products"
```

當頁面由數個內容節點的內容組成時，請包含列出內容路徑的 `/paths` 屬性。例如，頁面包含來自 `/content/image`、 `/content/video`和 `/var/files/pdfs`.下列組態可對頁面上的所有內容啓用嚴格連線：

```xml
/stickyConnections {
  /paths {
    "/content/image"
    "/content/video"
    "/var/files/pdfs"
  }
}
```

### HTTPOnly {#httponly}

啓用自黏連線時，dispatcher模組會設定 `renderid` Cookie。此Cookie沒有 `httponly` 標幟，因此應新增此標幟以加強安全性。您可以在設定 `httpOnly` 檔案的 `/stickyConnections` 節點中設定 `dispatcher.any` 屬性。屬性的值(或1)會定義 `renderid` Cookie是否已附加 `HttpOnly` 屬性。預設值為0，表示不會新增屬性。

如需 `httponly` 標幟的詳細資訊，請閱讀 [此頁面](https://www.owasp.org/index.php/HttpOnly)。

### secure {#secure}

啓用自黏連線時，dispatcher模組會設定 `renderid` Cookie。此Cookie沒有 **安全** 標幟，因此應新增此標幟以加強安全性。您可以在設定 `secure` 檔案的 `/stickyConnections` 節點中設定 `dispatcher.any` 屬性。屬性的值(或1)會定義 `renderid` Cookie是否已附加 `secure` 屬性。預設值為0，表示將新增屬性(若**傳入的要求安全)。如果值設為1，則無論傳入的要求是否安全，都會新增安全標幟。

## 處理演算連線錯誤 {#handling-render-connection-errors}

當演算伺服器傳回500錯誤或無法使用時，設定Dispatcher行為。

### 指定Health Check Page {#specifying-a-health-check-page}

使用 `/health_check` 屬性，指定當500個狀態代碼發生時所勾選的URL。如果此頁面也傳回500個狀態代碼，則會將此例項視為無法使用，且可設定的時間罰( `/unavailablePenalty`)會在重試前套用至演算。

```xml
/health_check
  {
  # Page gets contacted when an instance returns a 500
  /url "/health_check.html"
  }
```

### 指定頁面重試延遲 {#specifying-the-page-retry-delay}

/ `retryDelay` 屬性會設定Dispatcher在與農場renders一起嘗試回合之間的時間(以秒為單位)。在每個回合中，Dispatcher嘗試連線到演算的次數上限是農場中轉譯的次數。

Dispatcher會使用未 `"1"``/retryDelay` 明確定義的值。在大多數情況下，預設值是適當的。

```xml
/retryDelay "1"
```

### 設定重試次數 {#configuring-the-number-of-retries}

`/numberOfRetries` 該屬性會設定Dispatcher對轉譯執行的連線嘗試次數上限。如果Dispatcher在重試後無法成功連線到演算，Dispatcher會傳回失敗的回應。

在每個回合中，Dispatcher嘗試連線到演算的次數上限是農場中轉譯的次數。因此，Dispatcher嘗試連線的次數上限為( `/numberOfRetries`) x(轉譯次數)。

如果未明確定義值，預設值為 **5**。

```xml
/numberOfRetries "5"
```

### 使用容錯機制 {#using-the-failover-mechanism}

啓用Dispatcher農場上的容錯機制，在原始請求失敗時重新傳送請求至不同的轉譯。啓用容錯時，Dispatcher有下列行為：

* 當對演算的請求傳回HTTP狀態503(無法使用)時，Dispatcher會傳送要求給不同的演算。
* 當對轉譯的請求傳回HTTP狀態50x(以外的其他)時，Dispatcher會傳送對 `health_check` 該屬性設定之頁面的請求。

   * 如果health check傳回500(INTERNAL_ SERVER_ ERROR)，則Dispatcher會傳送原始請求給不同的演算。
   * 如果healthcheck傳回HTTP狀態200，Dispatcher會傳回初始HTTP500錯誤給用戶端。

若要啓用容錯，請將下列行新增至農場(或網站)：

```xml
/failover "1" 
```

>[!NOTE]
>
>若要重試包含主體的HTTP請求，Dispatcher會先傳送 `Expect: 100-continue` 請求標題至演算，再將實際內容遮住。CQ5.5與CQSE搭配使用，然後以100(READED)或錯誤碼立即回答。其他servlet容器也應該支援此功能。

## 忽略中斷錯誤-/ignoreEINTR {#ignoring-interruption-errors-ignoreeintr}

>[!CAUTION]
>
>通常不需要此選項。當您看到下列記錄訊息時，才需要使用此項目：
>
>`Error while reading response: Interrupted system call`

如果系統呼叫的物件位於透過NFS存取的遠端系統上，任何檔案系統導向的系統呼叫都可能中斷 `EINTR` 。這些系統呼叫是否可逾時或中斷，是根據底層檔案系統在本機電腦上的掛載方式。

如果您的例項具有此設定，則使用Social參數，記錄檔包含下列訊息：

`Error while reading response: Interrupted system call`

在內部，Dispatcher會使用可呈現為：

`while (response not finished) {  
read more data  
}`

此類訊息可在「 `EINTR``read more data`」區段發生時產生，且是由接收到任何資料前接收的訊號而產生。

若要忽略此類中斷，您可以將下列參數新增至 `dispatcher.any` (之前 `/farms`)：

`/ignoreEINTR "1"`

設定 `/ignoreEINTR` ，使 `"1"` Dispatcher繼續嘗試讀取資料，直到讀取完整回應為止。預設值為0並停用選項。

## 設計glob屬性的圖樣 {#designing-patterns-for-glob-properties}

Dispatcher配置檔案中的數個區段使用 `glob` 屬性作為用戶端請求的選擇標準。glob屬性的值是Dispatcher與請求方面的比較，例如請求資源的路徑，或用戶端的IP位址。例如 `/filter` ，區段中的項目使用glob模式來識別Dispatcher在或拒絕的頁面路徑。

glob值可以包含萬用字元和英數字元，以定義模式。

| 萬用字元字元 | 說明 | 範例 |
|--- |--- |--- |
| `*` | 相符項目：字串中任何字元的零或更多連續例項。符合的最終字元由下列其中一種情況決定： <br/>字串中的字元符合模式中的下一個字元，圖樣字元具有下列特性：<br/><ul><li>不是*</li><li>不是嗎？</li><li>常值字元(包括空格)或字元類別。</li><li>隨即到達圖樣結尾。</li></ul>在字元類別中，字元是由字元解譯。 | `*/geo*` 相符項目： `/content/geometrixx` 節點與 `/content/geometrixx-outdoors` 節點下方的任何頁面。下列HTTP請求符合glob模式： <br/><ul><li>`"GET /content/geometrixx/en.html"`</li><li>`"GET /content/geometrixx-outdoors/en.html"` </li></ul><br/> `*outdoors/*`<br/>相符項目： `/content/geometrixx-outdoors` 節點下方的任何頁面。例如，下列HTTP請求符合glob模式： <br/><ul><li>`"GET /content/geometrixx-outdoors/en.html"`</li></ul> |
| `?` | 比對任何單一字元。使用外部字元類別。在字元類別中，此字元是由字詞解譯。 | `*outdoors/??/*`<br/> 符合幾何xx戶外網站中任何語言的頁面。例如，下列HTTP請求符合glob模式： <br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul><br/>下列請求不符合glob模式： <br/><ul><li>「GET/content/geometrixx-outdoors/en.html」</li></ul> |
| `[ and ]` | 標記字元類別的開始和結束。字元類別可包含一個或多個字元範圍和單一字元。<br/>如果目標字元符合字元類別中的任何字元或在定義的範圍內，則會發生相符項目。<br/>如果未包含封閉括號，則圖樣不會產生相符項目。 | `*[o]men.html*`<br/> 符合下列HTTP要求：<br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li></ul><br/>不符合下列HTTP要求：<br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul><br/> `*[o/]men.html*`<br/>符合下列HTTP要求： <br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul> |
| `-` | 表示各種字元範圍。用於字元類別。除了字元類別之外，此字元也是由字元解譯。 | `*[m-p]men.html*` 符合下列HTTP要求： <br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li></ul>不符合下列HTTP要求：<br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul> |
| `!` | 否定追隨的字元或字元類別。僅用於否定字元類別內的字元和字元範圍。相當 `^ wildcard`於<br/>除了字元類別之外，此字元也是由字元解譯。 | `*[!o]men.html*`<br/> 符合下列HTTP要求： <br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul><br/>不符合下列HTTP要求： <br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li></ul><br/>`*[!o!/]men.html*`<br/> 不符合下列HTTP要求：<br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"` 或 `"GET /content/geometrixx-outdoors/en/men. html"`</li></ul> |
| `^` | 否定追隨的字元或字元範圍。僅用於否定字元類別內的字元和字元範圍。等同於 `!` 萬用字元。<br/>除了字元類別之外，此字元也是由字元解譯。 | `!` 套用萬用字元的範例，將 `!` 字元替換為 `^` 字元圖樣中的字元。 |


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

## Login {#logging}

在Web伺服器組態中，您可以設定：

* Dispatcher記錄檔的位置。
* 記錄層級。

如需詳細資訊，請參閱網頁伺服器文件和Dispatcher實例的讀我檔案。

**Apache旋轉/動態記錄檔**

如果使用 **Apache** Web伺服器，您可以使用旋轉和/或傳送的記錄檔的標準功能。例如，使用piped記錄檔：

`DispatcherLog "| /usr/apache/bin/rotatelogs logs/dispatcher.log%Y%m%d 604800"`

這會自動旋轉：

* dispatcher記錄檔；副檔名中的時間戳記(logs/dispatcher. log%Y%m%d)。
* (60x60x24x=604800秒)。

請參閱「記錄旋轉和標頭記錄檔」的Apache網頁伺服器文件；例如 [Apache2.2](https://httpd.apache.org/docs/2.2/logs.html)。

>[!NOTE]
>
>安裝時，預設記錄層級很高(即Level3=除錯)，因此Dispatcher會記錄所有錯誤和警告。這在初始階段非常有用。
>
>不過，這需要額外資源，因此當Dispatcher根據 *您的需求*平順地運作時，您可以(應)降低記錄層級。

### 追蹤記錄 {#trace-logging}

在Dispatcher的其他增強功能中，4.2.0版也引進了「追蹤記錄」。

這比除錯記錄更上層樓，在記錄檔中顯示其他資訊。它會新增記錄：

* 轉寄標題的值；
* 用於特定動作的規則。

您可以在網站伺服器中設定記錄層， `4` 以啓用追蹤記錄。

以下是啓用追蹤的記錄檔範例：

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

當請求符合封鎖規則的檔案時，事件會記錄：

```xml
[Thu Mar 03 14:42:45 2016] [T] [11831] 'GET /content.infinity.json HTTP/1.1' was blocked because of /0082
```

## 確認基本操作 {#confirming-basic-operation}

若要確認web server、Dispatcher和AEM實例的基本操作與互動，您可以執行下列步驟：

1. 設定 `loglevel` 為 `3`。

1. 啓動網頁伺服器；這也會啓動Dispatcher。
1. 啓動AEM實例。
1. 檢查您的網站伺服器和Dispatcher的記錄檔和錯誤檔案。\
   視您的Web伺服器而定，您應該會看見訊息，例如：\
   `[Thu May 30 05:16:36 2002] [notice] Apache/2.0.50 (Unix) configured`\
   和:\
   `[Fri Jan 19 17:22:16 2001] [I] [19096] Dispatcher initialized (build XXXX)`

1. 透過網頁伺服器瀏覽網站。確認內容已可視需要顯示。\
   例如，在本機安裝上AEM會在連接埠 `4502` 和網站伺服器上執行，並使用兩者來 `80` 存取網站主控台：\
   ` https://localhost:4502/libs/wcm/core/content/siteadmin.html  
https://localhost:80/libs/wcm/core/content/siteadmin.html  
`結果應相同。確認使用相同機制的其他頁面存取權。

1. 檢查快取目錄是否已填滿。
1. 啓動頁面以檢查快取是否正被清除。
1. 如果一切都正常運作，您可以降低 `loglevel``0`工作量。

## 使用多個Dispatchers {#using-multiple-dispatchers}

在複雜的設定中，您可以使用多個Dispatchers。例如，您可以使用：

* 一個Dispatcher在內部網路發佈網站
* 另一個Dispatcher，位於不同的位址，並具有不同的安全性設定，可在網際網路上發佈相同的內容。

在這種情況下，請確定每個請求只執行一個Dispatcher。Dispatcher不處理來自其他Dispatcher的請求。因此，請確定這兩個Dispatcher都直接存取AEM網站。

## 除錯 {#debugging}

將標題 `X-Dispatcher-Info` 新增至請求時，Dispatcher會回答目標是快取、從快取傳回還是完全不可快取。回應標題 `X-Cache-Info` 包含可讀表單中的資訊。您可以使用這些回應標題來除錯Dispatcher快取的回應問題。

此功能預設未啓用，因此，若要納入回應標題 `X-Cache-Info` ，農場必須包含下列項目：

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

`X-Dispatcher-Info` 此外，標題不需要值，但如果您用於 `curl` 測試，必須提供值才能傳送標題，例如：

```xml
curl -v -H "X-Dispatcher-Info: true" https://localhost/content/we-retail/us/en.html
```

以下是包含回應標題 `X-Dispatcher-Info` 的清單，會傳回：

* **快取**\
   目標檔案包含在快取中，而傳送程式已判斷為提供此檔案。
* **快取**\
   快取中不包含目標檔案，而傳送程式已判斷快取輸出並提供它。
* **快取：stat檔案太新，**定位檔案包含在快取中，不過，它會被較新的statter檔案無效。dispatcher會刪除目標檔案，從輸出重新建立它並提供它。
* **不適用：無文件根目錄**該農場的設定不包含文件根目錄(設定元素 `cache.docroot`)。
* **不適用：快取檔案路徑過長**\
   目標檔案-文件根目錄與URL檔案的串連-超過系統上最長的可能檔案名稱。
* **不適用：暫存檔案路徑過長**\
   暫存檔案名稱範本超過系統上最長的可能檔案名稱。在實際建立或覆寫快取檔案之前，發送器會先建立暫存檔案。暫存檔案名稱是目標檔案名稱，加上 `_YYYYXXXXXX` 附加至其上的字元，並將取代 `Y` 為 `X` 建立唯一名稱。
* **不適用：請求URL沒有副檔名**\
   請求URL沒有副檔名，或會遵循檔案副檔名，例如： `/test.html/a/path`。
* **不適用：請求不是GET或HEAD**
HTTP方法不是GET或HEAD。傳送程式假設輸出包含不應快取的動態資料。
* **不適用：請求包含查詢字串**\
   請求包含查詢字串。傳送程式假設輸出取決於給定的查詢字串，因此不會快取。
* **不適用：作業管理員未驗證**\
   農場快取由作業管理員負責(設定包含 `sessionmanagement` 節點)，且請求未包含適當的驗證資訊。
* **不適用：請求包含授權**\
   農場不允許快取輸出( `allowAuthorized 0`)，且請求包含驗證資訊。
* **不適用：target is a directory**\
   目標檔案為目錄。這可能會指出一些概念錯誤，其中URL和部分子URL都包含可快取的輸出，例如，如果要求先 `/test.html/a/file.ext` 傳入並包含可快取輸出，則傳送程式將無法快取後續請求的輸出 `/test.html`。
* **不適用：請求URL具有尾隨斜線**\
   請求URL具有結尾斜線。
* **不適用：請求URL不在快取規則中**\
   農場的快取規則明確拒絕快取某些請求URL的輸出。
* **不適用：授權檢查程式拒絕存取**\
   農場的授權檢查程式拒絕存取快取檔案。
* **不適用：作業不有效**，農場的快取由作業管理員管理(設定包含 `sessionmanagement` 節點)，且使用者的作業不會再有效。
* **不適用：回應包含`no_cache `**遠端伺服器傳回 `Dispatcher: no_cache` 標題，禁止傳送程式快取輸出。
* **不適用：回應內容長度為零**，回應的內容長度為零；dispatcher不會建立零長度檔案。
