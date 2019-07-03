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
source-git-commit: a997d2296e80d182232677af06a2f4ab5a14bfd5

---


# Configuring Dispatcher{#configuring-dispatcher}

>[!NOTE]
>
>Dispatcher版本與AEM獨立。如果您關注Dispatcher文件的連結(內嵌於舊版AEM的文件中)，可能會重新導向至此頁面。

以下章節說明如何設定Dispatcher的各個方面。

## Support for IPv4 and IPv6 {#support-for-ipv-and-ipv}

AEM和Dispatcher的所有元素都可安裝在IPv和IPv網路中。See [IPV4 and IPV6](https://helpx.adobe.com/experience-manager/6-3/sites/deploying/using/technical-requirements.html#AdditionalPlatformNotes).

## Dispatcher Configuration Files {#dispatcher-configuration-files}

By default the Dispatcher configuration is stored in the `dispatcher.any` text file, though you can change the name and location of this file during installation.

設定檔案包含一系列可控制Dispatcher行為的單一值或多值屬性：

* Property names are prefixed with a forward slash `/`.
* Multi-valued properties enclose child items using braces `{ }`.

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

For example, if the files `farm_1.any` through to `farm_5.any` contain the configuration of farms one to five, you can include them as follows:

```xml
/farms
  {
  $include "farm_*.any"
  }
```

## Using Environment Variables {#using-environment-variables}

您可以在dispatcher的字串值屬性中使用環境變數，而不是硬式編碼值。To include the value of an environment variable, use the format `${variable_name}`.

For example, if the dispatcher.any file is located in the same directory as the cache directory, the following value for the [docroot](dispatcher-configuration.md#main-pars-title-30) property can be used:

```xml
/docroot "${PWD}/cache"
```

As another example, if you create an environment variable named `PUBLISH_IP` that stores the hostname of the AEM publish instance, the following configuration of the [/renders](dispatcher-configuration.md#main-pars-127-25-0008) property can be used:

```xml
/renders {
  /0001 {
    /hostname "${PUBLISH_IP}"
    /port "8443"
  }
}
```

## Naming the Dispatcher Instance {#naming-the-dispatcher-instance-name}

Use the `/name` property to specify a unique name to identify your Dispatcher instance. `/name` 屬性是組態結構中的頂層屬性。

## Defining Farms {#defining-farms-farms}

`/farms` 屬性定義一或多組Dispatcher行為，每一組都與不同網站或URL相關聯。`/farms` 屬性可以包含單一農場或多個農場：

* 當您想要Dispatcher處理您的所有網頁或網站時，請使用單一農場。
* 當網站或不同網站的不同區域需要不同的Dispatcher行為時，建立多個農場。

`/farms` 屬性是組態結構中的頂層屬性。To define a farm, add a child property to the `/farms` property. 使用屬性名稱，可唯一識別Dispatcher例項中的農場。

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
>如果您使用多個轉譯農場，則會以下拉式評估清單。This is particularly relevant when defining [Virtual Hosts](dispatcher-configuration.md#main-pars-117-15-0006) for your websites.

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

## Specify a Default Page (IIS Only) - /homepage {#specify-a-default-page-iis-only-homepage}

>[!CAUTION]
>
>`/homepage`參數(僅IIS)不再運作。Instead, you should use the [IIS URL Rewrite Module](https://docs.microsoft.com/en-us/iis/extensions/url-rewrite-module/using-the-url-rewrite-module).
>
>If you are using Apache, you should use the `mod_rewrite` module. See the Apache web site documentation for information about `mod_rewrite` (for example, [Apache 2.4](https://httpd.apache.org/docs/current/mod/mod_rewrite.html)). When using `mod_rewrite`, it is advisable to use the flag ** [&#39;passthrough|PT&#39; (pass through to next handler)](https://helpx.adobe.com/dispatcher/kb/DispatcherModReWrite.html)** to force the rewrite engine to set the `uri` field of the internal `request_rec` structure to the value of the `filename` field.

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

## Specifying the HTTP Headers to Pass Through {#specifying-the-http-headers-to-pass-through-clientheaders}

`/clientheaders` 屬性定義Dispatcher從用戶端HTTP請求傳遞至轉譯器(AEM實例)的HTTP標題清單。

依預設，Dispatcher會將標準HTTP標題轉送至AEM實例。在某些情況下，您可能想要轉寄其他標題，或移除特定標題：

* 新增標題(例如自訂標題)，您的AEM實例會在HTTP要求中預期。
* 移除僅與Web伺服器相關的標題(例如驗證標題)。

如果您自訂要通過的標題集，您必須指定完整的標題清單，包括通常包含在內的標題清單。

For example, a Dispatcher instance that handles page activation requests for publish instances requires the `PATH` header in the `/clientheaders` section. `PATH` 標題可啓用複製代理程式與發送器之間的通訊。

The following code is an example configuration for `/clientheaders`:

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

## Identifying Virtual Hosts {#identifying-virtual-hosts-virtualhosts}

`/virtualhosts` 此屬性定義Dispatcher接受此農場的所有主機名稱/URI組合清單。您可以使用星號(「*」)字元作為萬用字元。Values for the / `virtualhosts` property use the following format:

```xml
[scheme]host[uri][*]
```

* `scheme`：(選用)或 `https://``https://.`
* `host`：主機的名稱或IP位址，以及必要的連接埠號碼。(See [https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.23](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.23))
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

The following configuration handles *all* requests:

```xml
   /virtualhosts
    {
    "*"
    }
```

### Resolving the Virtual Host {#resolving-the-virtual-host}

When Dispatcher receives an HTTP or HTTPS request, it finds the virtual host value that best-matches the `host,` `uri`, and `scheme` headers of the request. Dispatcher evaluates the values in the `virtualhosts` properties in the following order:

* Dispatcher會從dispatcher. any檔案中的最低農場和進度開始。
* For each farm, Dispatcher begins with the topmost value in the `virtualhosts` property and progresses down the list of values.

Dispatcher以下列方式找出最佳符合的虛擬主機值：

* The first-encountered virtual host that matches all three of the `host`, the `scheme`, and the `uri` of the request is used.
* If no `virtualhosts` values has `scheme` and `uri` parts that both match the `scheme` and `uri` of the request, the first-encountered virtual host that matches the `host` of the request is used.
* If no `virtualhosts` values have a host part that matches the host of the request, the topmost virtual host of the topmost farm is used.

Therefore, you should place your default virtual host at the top of the `virtualhosts` property in the topmost farm of your dispatcher.any file.

### Example Virtual Host Resolution {#example-virtual-host-resolution}

The following example represents a snippet from a dispatcher.any file that defines two Dispatcher farms, and each farm defines a `virtualhosts` property.

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

## Enabling Secure Sessions - /sessionmanagement {#enabling-secure-sessions-sessionmanagement}

>[!CAUTION]
>
>`/allowAuthorized`**必須** 設定 `"0"` 在 `/cache` 區段中，才能啓用此功能。

建立安全作業以存取演算農場，讓使用者必須登入才能存取農場中的任何頁面。登入後，使用者可以存取農場中的頁面。See [Creating a Closed User Group](https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/cug.html#CreatingTheUserGroupToBeUsed) for information about using this feature with CUGs. Also, see the Dispatcher [Security Checklist](/help/using/security-checklist.md) before going live.

`/sessionmanagement` 屬性是一個子屬性 `/farms`。

>[!CAUTION]
>
>如果網站的區段使用不同的存取要求，您必須定義多個農場。

**/sessionmanagement** 有數個子參數：

**/directory** (強制)

儲存工作階段資訊的目錄。如果目錄不存在，則會建立此目錄。

>[!CAUTION]
>
> When configuring the directory sub-parameter **do not** point to the root folder (`/directory "/"`) as it can cause serious problems. 您應永遠指定儲存工作階段資訊的資料夾路徑。例如：

```xml
/sessionmanagement 
  { 
  /directory "/usr/local/apache/.sessions"
  }
```

**/encode** (選用)

作業資訊的編碼方式。使用md演算法對加密使用md演算法或十六進位編碼的「十六進位」。如果您加密作業資料，存取檔案系統的使用者無法讀取工作階段內容。預設值為「md5」。

**/header** (選用)

儲存授權資訊的HTTP標題或Cookie的名稱。If you store the information in the http header, use `HTTP:<*header-name*>`. To store the information in a cookie, use `Cookie:<header-name>`. If you do not specify a value `HTTP:authorization` is used.

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

## Defining Page Renderers {#defining-page-renderers-renders}

/renders屬性定義Dispatcher傳送請求以轉譯文件的URL。The following example `/renders` section identifies a single AEM instance for rendering:

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

### Renders options {#renders-options}

**/timeout**

指定存取AEM實例以毫秒為單位的連線逾時。預設值為「0」，造成Dispatcher無限期等候。

**/receiveTimeout**

指定允許回應的毫秒數。預設值為「6000」，導致Dispatcher等候10分鐘。「0」設定會完全消除逾時。\
如果在剖析回應標題時到達逾時，則會傳回HTTP狀態504(不當閘道)。如果在讀取回應主體時到達逾時，Dispatcher會傳回對用戶端不完整的回應，但刪除任何可能寫入的快取檔案。

**/ipv4**

Specifies whether Dispatcher uses the `getaddrinfo` function (for IPv6) or the `gethostbyname` function (for IPv4) for obtaining the IP address of the render. A value of 0 causes `getaddrinfo` to be used. A value of 1 causes `gethostbyname` to be used. 預設值為0。

getaddrinfo函數會傳回IP位址清單。Dispatcher會重復位址清單，直到它建立TCP/IP連線為止。因此，當演算主機名稱與getdrinfo函數相關聯時，會傳回一份一律位於相同順序的IP位址清單，因此piv屬性很重要，當演算主機名稱與之關聯時。在此情況下，您應使用getoshybiname函數，以便Dispatcher連接的IP位址是randomized的。

Amazon Elastic Load Balancing(ELB)是一種回應getdrinfo的服務，可使用相同的IP位址清單來回應getdrinfo。

**/secure**

If the `/secure` property has a value of &quot;1&quot; Dispatcher uses HTTPS to communicate with the AEM instance. For additional details, also see [Configuring Dispatcher to Use SSL](dispatcher-ssl.md#configuring-dispatcher-to-use-ssl).

**/always-resolve**

With Dispatcher version **4.1.6**, you can configure the `/always-resolve` property as follows:

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

## Configuring Access to Content {#configuring-access-to-content-filter}

Use the `/filter` section to specify the HTTP requests that Dispatcher accepts. 所有其他請求都會傳回至含有404錯誤代碼(找不到頁面)的Web伺服器。If no `/filter` section exists, all requests are accepted.

**注意：** 一律拒絕 [對statfile](dispatcher-configuration.md#main-pars-title-28) 的請求。

>[!CAUTION]
>
>See the [Dispatcher Security Checklist](security-checklist.md) for further considerations when restricting access using Dispatcher. Also, read the [AEM Security Cheklist](https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/security-checklist.html) for additional security details regarding your AEM installation.

/filter區段包含一系列規則，可根據HTTP要求的請求列部分中的模式拒絕或允許存取內容。您應為您的/filter區段使用一個模糊策略：

* 首先，拒絕存取所有內容。
* 允許存取內容。

### Defining a Filter {#defining-a-filter}

`/filter` 區段中的每個項目都包含一個類型和一個符合請求列特定元素或整個請求行的模式。每個篩選器可包含下列項目：

* **類型**：指出 `/type` 是否允許或拒絕符合模式的請求。The value can be either `allow` or `deny`.

* **請求列的元素：** 根據 `/method`HTTP要求的請求行部分的特定部分，包括 `/url`、 `/query`或 `/protocol` 建立篩選請求的模式。篩選請求行元素(而非整個請求行)是偏好篩選方法。

* **請求行的進階元素：** 從Dispatcher4.2.0開始，有個新的篩選元素可供使用。These new elements are `/path`, `/selectors`, `/extension` and `/suffix` respectively. 納入一或多個這些項目，以進一步控制URL模式。

>[!NOTE]
>
>For more information about what part of the request line each of these elements references, see the [Sling URL Decomposition](https://sling.apache.org/documentation/the-sling-engine/url-decomposition.html) wiki page.

* **glob屬性**： `/glob` 此屬性可用來符合HTTP要求的整個要求列。

>[!CAUTION]
>
>在Dispatcher中停用使用globs的篩選。As such, you should avoid using globs in the `/filter` sections since it may lead to security issues. 因此，不是：

`/glob "* *.css *"`

您應使用

`/url "*.css"`

#### The request-line Part of HTTP Requests {#the-request-line-part-of-http-requests}

HTTP/1.1 defines the [request-line](https://www.w3.org/Protocols/rfc2616/rfc2616-sec5.html) as follows:

*方法要求-URI HTTP-Version*&lt; CRLF&gt;

&lt; CRLF&gt;字元會重新產生歸位，再加上行饋送。以下範例為當用戶端請求Geometrixx-UTouts網站頁面時的請求列：

GET/content/geometrixx-outdoors/en.htmlHTTP.1.1&lt; CRLF&gt;

您的模式必須考量請求列和&lt; CRLF&gt;字元中的空格字元。

#### Double-quotes vs Single-quotes {#double-quotes-vs-single-quotes}

When creating your filter rules, use double quotation marks `"pattern"` for simple patterns. If you are using Dispatcher 4.2.0 or later and your pattern includes a regular expression, you must enclose the regex pattern `'(pattern1|pattern2)'` within single quotation marks.

#### Regular Expressions {#regular-expressions}

在Dispatcher4.2.0之後，您可以在篩選模式中加入POSX Extended規則運算式。

#### Troubleshooting Filters {#troubleshooting-filters}

If your filters are not triggering in the way you would expect, enable [Trace Logging](#trace-logging) on dispatcher so you can see which filter is intercepting the request.

#### Example Filter: Deny All {#example-filter-deny-all}

下列範例篩選區段會造成Dispatcher拒絕對所有檔案的請求。您應拒絕存取所有檔案，然後允許存取特定區域。

```xml
  /0001  { /glob "*" /type "deny" }
```

要求明確拒絕區域會傳回404錯誤代碼(找不到頁面)。

#### Example Filter: Deny Acess to Specific Areas {#example-filter-deny-acess-to-specific-areas}

篩選器也允許您拒絕對不同元素(例如ASP頁面和發佈例項中的敏感區域)的存取。下列篩選條件可讓您存取ASP頁面：

```xml
/0002  { /type "deny" /url "*.asp"  }
```

#### Example Filter: Enable POST Requests {#example-filter-enable-post-requests}

下列範例篩選可讓您依POST方法提交表單資料：

```xml
/filter {
    /0001  { /glob "*" /type "deny" }
    /0002 { /type "allow" /method "POST" /url "/content/[.]*.form.html" }
}
```

#### Example Filter: Allow Access to the Workflow Console {#example-filter-allow-access-to-the-workflow-console}

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

#### Example filter: Using Regular Expressions {#example-filter-using-regular-expressions}

此篩選器可使用規則運算式在非公開的內容目錄中啓用擴充功能，並在此處定義單引號：

```xml
/005  {  /type "allow" /extension '(css|gif|ico|js|png|swf|jpe?g)' }
```

#### Example filter: Filter Additional Elements of a Request URL {#example-filter-filter-additional-elements-of-a-request-url}

Below is a rule example that blocks content grabbing from the `/content` path and its subtree, using filters for path, selectors and extensions:

```xml
/006 {
        /type "deny"
        /path "/content/*"
        /selectors '(feed|rss|pages|languages|blueprint|infinity|tidy)'
        /extension '(json|xml|html)'
        }
```

### Example /filter section {#example-filter-section}

設定Dispatcher時，您應盡可能限制外部存取。下列範例為外部訪客提供最少的存取權：

* `/content`
* 不同的內容，例如設計和用戶端程式庫；例如：

   * `/etc/designs/default*`
   * `/etc/designs/mydesign*`

After you create filters, [test page access](dispatcher-configuration.md#main-pars-title-19) to ensure your AEM instance is secure.

The following /filter section of the dispatcher.any file can be used as a basis in your [Dispatcher configuration](dispatcher-configuration.md) file.

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
>與Apache搭配使用時，請根據Dispatcher模組的dispatcheruseProcessurel屬性來設計您的篩選器URL圖樣。(See [Apache Web Server - Configure your Apache Web Server for Dispatcher](dispatcher-install.md#main-pars-55-35-1022).)

>[!NOTE]
>
>有關動態媒體的篩選0030和0031適用於AEM6.0及更高版本。

如果您選擇延長存取權，請考慮下列建議：

* External access to `/admin` should always be *completely* disabled if you are using CQ version 5.4 or an earlier version.

* Care must be taken when allowing access to files in `/libs`. 存取權應以個人為準。
* 拒絕對複製設定的存取，使其無法查看：

   * `/etc/replication.xml*`
   * `/etc/replication.infinity.json*`

* 拒絕存取Google Gadget反向proxy：

   * `/libs/opensocial/proxy*`

Depending on your installation, there might be additional resources under `/libs`, `/apps` or elsewhere, that must be made available. You can use the `access.log` file as one method of determining resources that are being accessed externally.

>[!CAUTION]
>
>存取主機和目錄可能會對生產環境造成安全性風險。除非您有明確的猜測，否則它們應保持停用(留言)。

>[!CAUTION]
>
>If you are [using reports in a publish environment](https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/reporting.html#UsingReportsinaPublishEnvironment) you should configure Dispatcher to deny access to `/etc/reports` for external visitors.

### Restricting Query Strings {#restricting-query-strings}

Since Dispatcher version 4.1.5, use the `/filter` section to restrict query strings. It is highly recommended to explicitly allow query strings and exclude generic allowance through `allow` filter elements.

A single entry can have either *glob* or some combination of *method*,*url*,*query* and *version* but not both. The following example allows the `a=*` query string and denies all other query strings for URLs that resolve to the `/etc` node:

```xml
/filter {
 /0001 { /type "deny" /method "POST" /url "/etc/*" }
 /0002 { /type "allow" /method "GET" /url "/etc/*" /query "a=*" }
}
```

>[!NOTE]
>
>If a rule contains a `/query`, it will only match requests that contain a query string and match the provided query pattern.
>
>In above example, if requests to `/etc` that have no query string should be allowed as well, the following rules would be required:


```xml
/filter {  
>/0001 { /type "deny" /method “*" /url "/path/*" }  
>/0002 { /type "allow" /method "GET" /url "/path/*" }  
>/0003 { /type “deny" /method "GET" /url "/path/*" /query "*" }  
>/0004 { /type "allow" /method "GET" /url "/path/*" /query "a=*" }  
}  
```

### Testing Dispatcher Security {#testing-dispatcher-security}

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
* /content/add_valid_page.query.json?statement=//*[@transportPassword]/(@transportPassword%20|%20@transportUri%20|%20@transportUser)
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

## Enabling Access to Vanity URLs {#enabling-access-to-vanity-urls-vanity-urls}

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (sbroders@adobe.com)
Last Modified Date: 2015-03-25T14:23:05.185-0400

<p style="font-family: tahoma, arial, helvetica, sans-serif; font-size: 12px;">For https://jira.corp.adobe.com/browse/DOC-4812</p> 
<p style="font-family: tahoma, arial, helvetica, sans-serif; font-size: 12px;">The "com.adobe.granite.dispatcher.vanityurl.content" package needs to be made public before publishing this contnet.</p>
 -->

設定Dispatcher以啓用對您CQ或AEM頁面設定之虛名URL的存取。

啓用虛名URL存取權時，Dispatcher會定期呼叫執行於演算例項上的服務，以取得虛名URL清單。Dispatcher會將此清單儲存在本機檔案中。When a request for a page is denied due to a filter in the `/filter` section, Dispatcher consults the list of vanity URLs. 如果拒絕的URL位於清單中，Dispatcher允許存取虛名URL。

To enable access to vanity URLs, add a `/vanity_urls` section to the `/farms` section, similar to the following example:

```xml
 /vanity_urls {
      /url "/libs/granite/dispatcher/content/vanityUrls.html"
      /file "/tmp/vanity_urls"
      /delay 300
 }
```

`/vanity_urls` 區段包含下列屬性：

* `/url`：在演算例項上執行的虛名URL服務路徑。The value of this property must be `"/libs/granite/dispatcher/content/vanityUrls.html"`.

* `/file`：Locatcher儲存虛名URL清單的本機檔案路徑。請確定Dispatcher具有對此檔案的寫入存取權。
* `/delay`：(秒)呼叫虛名URL服務之間的時間。

>[!NOTE]
>
>If your render is an instance of AEM you must install the [VanityURLS-Components](https://www.adobeaemcloud.com/content/marketplace/marketplaceProxy.html?packagePath=/content/companies/public/adobe/packages/cq600/component/vanityurls-components) package to install the vanity URL service. (See [Signing In to Package Share](https://helpx.adobe.com/experience-manager/6-3/sites/administering/using/package-manager.html#SigningIntoPackageShare).)

使用下列程序可讓存取虛名URL。

1. 如果您的演算服務是AEM實例，請在發佈例項上安裝com. adobe. granite. vanitcher. vanityurl. content套件(請參閱上述附註)。
1. For each vanity URL that you have configured for an AEM or CQ page, ensure that the ` [/filter](dispatcher-configuration.md#main-pars_134_32_0009)` configuration denies the URL. 如有必要，請新增拒絕URL的篩選器。
1. Add the `/vanity_urls` section below `/farms`.
1. 重新啓動Apache網頁伺服器。

## Forwarding Syndication Requests - /propagateSyndPost {#forwarding-syndication-requests-propagatesyndpost}

教學請求通常僅適用於Dispatcher，因此，依預設，它們不會傳送至轉譯器(例如AEM實例)。

如有必要，請將「/propagateSyndPost」屬性設為「1」，然後轉送至「Dispatcher」。如果設定，您必須確定篩選區段中不拒絕POST請求。

## Configuring the Dispatcher Cache - /cache {#configuring-the-dispatcher-cache-cache}

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
>For permission-sensitive caching, read [Caching Secured Content](permissions-cache.md).

### Specifying the Cache Directory {#specifying-the-cache-directory}

`/docroot` 屬性會識別儲存快取檔案的目錄。

>[!NOTE]
>
>此值必須與Web伺服器的文件根目錄完全相同，以便Dispatcher和Web伺服器處理相同的檔案。\
>當使用dispatcher快取檔案時，Web伺服器會負責提供正確的狀態代碼，因此也就很重要。

如果您使用多個農場，每個農場都必須使用不同的文件根目錄。

### Naming the Statfile {#naming-the-statfile}

`/statfile` 屬性會識別檔案以用作statfile。Dispatcher會使用此檔案來註冊最新內容更新的時間。statfile可以是Web伺服器上的任何檔案。

statfile沒有內容。更新內容時，Dispatcher會更新時間戳記。預設的statfile會命名為. stat並儲存在docroot中。Dispatcher會封鎖對statfile的存取。

>[!NOTE]
>
>`/statfileslevel` 如果已設定，則Dispatcher會忽略 `/statfile` 屬性並使用. stat作為名稱。

### Serving Stale Documents When Errors Occur {#serving-stale-documents-when-errors-occur}

`/serveStaleOnError` 當演算伺服器傳回錯誤時，屬性會控制Dispatcher是否傳回失效文件。依預設，當中繼檔案被觸控並失效快取內容時，Dispatcher會在下次請求時刪除快取內容。

If `/serveStaleOnError` is set to &quot;1&quot;, Dispatcher does not delete invalidated content from the cache unless the render server returns a successful response. 來自AEM或連線逾時的xx回應會造成Dispatcher提供過時內容，並回應與HTTP狀態114(重新驗證失敗)。

### Caching When Authentication is Used {#caching-when-authentication-is-used}

`/allowAuthorized` 屬性會控制是否快取包含下列任何驗證資訊的請求：

* `authorization` 標題。
* A cookie named `authorization`.
* A cookie named `login-token`.

根據預設，不會快取包含此驗證資訊的請求，因為當已快取的文件傳回用戶端時，不會執行驗證。此設定可防止Dispatcher將快取的文件提供給沒有必要權限的使用者。

不過，如果您的需求允許快取已驗證文件，請將「/allowAuthorized」設為一：

`/allowAuthorized "1"`

>[!NOTE]
>
>To enable session management (using the `/sessionmanagement` property), the `/allowAuthorized` property must be set to `"0"`.

### Specifying the Documents to Cache {#specifying-the-documents-to-cache}

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
>DET或HEAD(適用於HTTP標題)方法可由Dispatcher使用。For additional information on response header caching, see the [Caching HTTP Response Headers](dispatcher-configuration.md#caching-http-response-headers) section.

Each item in the /rules property includes a [glob](#designing-patterns-for-glob-properties) pattern and a type:

* glob圖樣可用來比對文件的路徑。
* 類型會指出要快取符合glob模式的文件。值可以是允許(快取文件)或拒絕(一律顯示文件)。

如果您沒有動態頁面(除了上述規則已排除的頁面)，您也可以設定Dispatcher快取所有內容。The rules section for this look as follows：

```xml
/rules
  { 
    /0000  {  /glob "*"   /type "allow" }
  }
```

For information about glob properties, see [Designing Patterns for glob Properties](#designing-patterns-for-glob-properties).

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

在Apache Web伺服器上，您可以壓縮快取的文件。壓縮可讓Apache依照用戶端要求，以壓縮形式傳回文件。Compression is done automatically by enabling the Apache module `mod_deflate`, for example:

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

### Invalidating Files by Folder Level {#invalidating-files-by-folder-level}

Use the `/statfileslevel` property to invalidate cached files according to their path:

* Dispatcher creates `.stat`files in each folder from the docroot folder to the level that you specify. docroot資料夾為level0。
* Files are invalidated by touching the `.stat` file. `.stat` 檔案的最後修改日期會與快取文件的最後修改日期進行比較。`.stat` 如果檔案較新，則會重新擷取文件。

* When a file located at a certain level is invalidated then **all** `.stat` files from the docroot **to** the level of the invalidated file or the configured `statsfilevel` (whichever is smaller) will be touched.

   * For example: if you set the `statfileslevel` property to 6 and a file is invalidated at level 5 then every `.stat` file from docroot to 5 will be touched. 繼續使用此範例，如果某個檔案在層級上失效。`stat` file from docroot to6will be touch(from `/statfileslevel = "6"`).

只有沿著路徑**的資源**會受到影響。Consider the following example: a website uses the structure `/content/myWebsite/xx/.` If you set `statfileslevel` as 3, a `.stat`file is created as follows:

* `docroot`
* `/content`
* `/content/myWebsite`
* `/content/myWebsite/*xx*`

When a file in `/content/myWebsite/xx` is invalidated then every `.stat` file from docroot down to `/content/myWebsite/xx`is touched. This would be the case only for `/content/myWebsite/xx` and not for example `/content/myWebsite/yy` or `/content/anotherWebSite`.

>[!NOTE]
>
>Invalidation can be prevented by sending an additional Header `CQ-Action-Scope:ResourceOnly`. 這可用來清除特定資源，而不會使快取的其他部分失效。See [this page](https://adobe-consulting-services.github.io/acs-aem-commons/features/dispatcher-flush-rules/index.html) and [Manually Invalidating the Dispatcher Cache](https://helpx.adobe.com/experience-manager/dispatcher/using/page-invalidate.html) for additional details.

>[!NOTE]
>
>If you specify a value for the `/statfileslevel` property, the `/statfile` property is ignored.

### Automatically Invalidating Cached Files {#automatically-invalidating-cached-files}

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

For information about glob properties, see [Designing Patterns for glob Properties](#designing-patterns-for-glob-properties).

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

### Using custom invalidation scripts {#using-custom-invalidation-scripts}

Social屬性可讓您定義由Dispatcher接收的每個無效要求所呼叫的指令碼。

系統會呼叫下列引數：

* 控點\
   無效的內容路徑
* 動作\
   複製動作(例如啓用、停用)
* 動作範圍\
   The replication Action&#39;s Scope (empty, unless a header of `CQ-Action-Scope: ResourceOnly` is sent, see [Invalidating Cached Pages from AEM](page-invalidate.md) for details)

這可用於涵蓋許多不同的使用案例，例如無效的其他應用程式快取，或處理頁面的外部URL及其位置與內容路徑不符的案例。

下方範例指令檔會記錄每個無效要求至檔案。

```xml
/invalidateHandler "/opt/dispatcher/scripts/invalidate.sh"
```

#### sample invalidation handler script {#sample-invalidation-handler-script}

```shell
#!/bin/bash

printf "%-15s: %s %s" $1 $2 $3>> /opt/dispatcher/logs/invalidate.log
```

### Limiting the Clients That Can Flush the Cache {#limiting-the-clients-that-can-flush-the-cache}

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

For information about glob properties, see [Designing Patterns for glob Properties](#designing-patterns-for-glob-properties).

>[!CAUTION]
>
>建議您定義Social。
>
>如果尚未完成，任何用戶端都可以發出清除快取的呼叫；如果重復完成，則會嚴重影響網站效能。

### Ignoring URL Parameters {#ignoring-url-parameters}

`ignoreUrlParams` 此區段定義了在判斷頁面快取或從快取傳送時，哪些URL參數被忽略：

* 當請求URL包含所有已忽略的參數時，會快取頁面。
* 當請求URL包含一或多個未被忽略的參數時，不會快取頁面。

當頁面的參數被忽略時，第一次請求頁面時會快取頁面。無論請求中的參數值為何，都會對頁面的後續請求提供快取頁面。

To specify which parameters are ignored, add glob rules to the `ignoreUrlParams` property:

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

Using the example `ignoreUrlParams` value, the following HTTP request causes the page to be cached because the `q` parameter is ignored:

```xml
GET /mypage.html?q=5
```

Using the example `ignoreUrlParams` value, the following HTTP request causes the page to **not** be cached because the `p` parameter is not ignored:

```xml
GET /mypage.html?q=5&p=4
```

For information about glob properties, see [Designing Patterns for glob Properties](#designing-patterns-for-glob-properties).

### Caching HTTP Response Headers {#caching-http-response-headers}

>[!NOTE]
>
>This feature is avaiable with version **4.1.11** of the Dispatcher.

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
>此外，請注意，不允許檔案全域字元。For more details, see [Designing Patterns for glob Properties](#designing-patterns-for-glob-properties).

>[!NOTE]
>
>如果您需要Dispatcher來儲存並提供AEM的eTag回應標題，請執行下列動作：
>
>* Add the header name in the `/cache/headers`section.
>* Add the following [Apache directive](https://httpd.apache.org/docs/2.4/mod/core.html#fileetag) in the Dispatcher related section:
>



```xml
FileETag none
```

### Dispatcher Cache File Permissions {#dispatcher-cache-file-permissions}

`mode` 該屬性指定了哪些檔案權限套用至快取中的新目錄和檔案。This setting is restricted by the `umask` of the calling process. 它是以下列一或多個值之總和建構的八位數數字：

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

### Throttling .stat file touching {#throttling-stat-file-touching}

With the default `/invalidate` property, every activation effectively invalidates all `.html` files (when their path matches the `/invalidate` section). 在流量相當大的網站上，後續啓動會增加後端的CPU負載。In such a scenario, it would be desirable to &quot;throttle&quot; `.stat` file touching to keep the website responsive. You can do this by using the `/gracePeriod` property.

`/gracePeriod` 屬性會定義過時的秒數，自動失敗的資源在最後一次啓動後仍可從快取中提供。您可以在設定中使用屬性，在此設定中，一批次啓動會重復失效整個快取。建議的值為秒。

For additional details, also read the `/invalidate` and `/statfileslevel`sections above.

## Configuring Time Based Cache Invalidation - /enableTTL {#configuring-time-based-cache-invalidation-enablettl}

If set, the `enableTTL` property will evaluate the response headers from the backend, and if they contain a `Cache-Control` max-age or `Expires` date, an auxiliary, empty file next to the cache file is created, with modification time equal to the expiry date. 當已快取的檔案經過請求後，會自動從後端請求修改時間。

You can enable the feature by adding this line to the `dispatcher.any` file:

```xml
/enableTTL "1"
```

>[!NOTE]
>
>This feature is avaiable with version **4.1.11** of the Dispatcher.

## Configuring Load Balancing - /statistics {#configuring-load-balancing-statistics}

`/statistics` 本節定義Dispatcher為每個演算的回應速度評分的檔案類別。Dispatcher會使用分數來決定要傳送請求的轉譯。

您建立的每個類別都會定義glob模式。Dispatcher會比較要求內容的URI，以判斷所請求內容的類別：

* 類別順序會決定與URI比較的順序。
* 符合URI的第一個類別模式是檔案的類別。不會評估類別模式。

Dispatcher最多支援個統計類別。如果定義超過個類別，則只會使用前個類別。

**演算選取範圍**

每次Dispatcher需要轉譯頁面時，它會使用下列演算法來選取演算：

1. If the request contains the render name in a `renderid` cookie, Dispatcher uses that render.
1. If the request includes no `renderid` cookie, Dispatcher compares the render statistics:

   1. Dispatcher會判斷請求URI的目錄。
   1. Dispatcher會判斷哪個演算具有該類別的最低回應分數，並選取該演算。

1. 如果尚未選取演算，請使用清單中的第一個演算。

轉譯類別的分數是根據先前的回應時間，以及Dispatcher嘗試的先前失敗和成功連線。每次嘗試時，都會更新請求之URI類別的分數。

>[!NOTE]
>
>如果您不使用負載平衡，可以忽略此區段。

### Defining Statistics Categories {#defining-statistics-categories}

為您要保留演算選取範圍統計資料的每個文件類型定義類別。/statistics區段包含/categories區段。若要定義類別，請在具有下列格式的/categories區段下方新增一行：

`/name { /glob "pattern"}`

The category `name` must be unique to the farm. The `pattern` is described in the [Designing Patterns for glob Properties](#designing-patterns-for-glob-properties) section.

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

### Reflecting Server Unavailability in Dispatcher Statistics {#reflecting-server-unavailability-in-dispatcher-statistics}

`/unavailablePenalty` 屬性會設定當連線至演算失敗時，套用至演算統計資料的時間(1/10)。Dispatcher會將時間新增至符合要求URI的統計類別。

例如，當AEM未執行(且不會監聽)或因網路相關問題而無法建立指定主機名稱/連接埠時，就會套用罰金。

`/unavailablePenalty` 屬性是 `/farm` 區段的直接子項( `/statistics` 區段的子系)。

If no `/unavailablePenalty` property exists, a value of &quot;1&quot; is used.

```xml
/unavailablePenalty "1"
```

## Identifying a Sticky Connection Folder - /stickyConnectionsFor {#identifying-a-sticky-connection-folder-stickyconnectionsfor}

`/stickyConnectionsFor` 屬性定義一個包含嚴格文件的檔案夾；這會使用URL存取。Dispatcher會將所有請求從單一使用者傳送至相同的轉譯例項。黏著連線可確保所有文件的作業資料呈現和一致。This mechanism uses the `renderid` cookie.

下列範例定義/products檔案夾的嚴格連線：

```xml
/stickyConnectionsFor "/products"
```

When a page is composed of content from several content nodes, include the `/paths` property that lists the paths to the content. For example, a page contains content from `/content/image`, `/content/video`, and `/var/files/pdfs`. 下列組態可對頁面上的所有內容啓用嚴格連線：

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

When sticky connections are enabled, the dispatcher module sets the `renderid` cookie. This cookie doesn&#39;t have the `httponly` flag, which should be added in order to enhance security. You can do this by setting the `httpOnly` property in the `/stickyConnections` node of a `dispatcher.any` configuration file. The property&#39;s value (either 0 or 1) defines whether the `renderid` cookie has the `HttpOnly` attribute appended. 預設值為0，表示不會新增屬性。

For additional information about the `httponly` flag, read [this page](https://www.owasp.org/index.php/HttpOnly).

### secure {#secure}

When sticky connections are enabled, the dispatcher module sets the `renderid` cookie. This cookie doesn&#39;t have the **secure** flag, which should be added in order to enhance security. You can do this by setting the `secure` property in the `/stickyConnections` node of a `dispatcher.any` configuration file. The property&#39;s value (either 0 or 1) defines whether the `renderid` cookie has the `secure` attribute appended. 預設值為0，表示將新增屬性(若**傳入的要求安全)。如果值設為1，則無論傳入的要求是否安全，都會新增安全標幟。

## Handling Render Connection Errors {#handling-render-connection-errors}

當演算伺服器傳回500錯誤或無法使用時，設定Dispatcher行為。

### Specifying a Health Check Page {#specifying-a-health-check-page}

Use the `/health_check` property to specify a URL that is checked when a 500 status code occurs. If this page also returns a 500 status code the instance is considered to be unavailable and a configurable time penalty ( `/unavailablePenalty`) is applied to the render before retrying.

```xml
/health_check
  {
  # Page gets contacted when an instance returns a 500
  /url "/health_check.html"
  }
```

### Specifying the Page Retry Delay {#specifying-the-page-retry-delay}

The / `retryDelay` property sets the time (in seconds) that Dispatcher waits between rounds of connection attempts with the farm renders. 在每個回合中，Dispatcher嘗試連線到演算的次數上限是農場中轉譯的次數。

Dispatcher uses a value of `"1"` if `/retryDelay` is not explicitly defined. 在大多數情況下，預設值是適當的。

```xml
/retryDelay "1"
```

### Configuring the Number of Retries {#configuring-the-number-of-retries}

`/numberOfRetries` 該屬性會設定Dispatcher對轉譯執行的連線嘗試次數上限。如果Dispatcher在重試後無法成功連線到演算，Dispatcher會傳回失敗的回應。

在每個回合中，Dispatcher嘗試連線到演算的次數上限是農場中轉譯的次數。Therefore, the maximum number of times that Dispatcher attempts a connection is ( `/numberOfRetries`) x (the number of renders).

If the value is not explicitly defined, the default value is **5**.

```xml
/numberOfRetries "5"
```

### Using the Failover Mechanism {#using-the-failover-mechanism}

啓用Dispatcher農場上的容錯機制，在原始請求失敗時重新傳送請求至不同的轉譯。啓用容錯時，Dispatcher有下列行為：

* 當對演算的請求傳回HTTP狀態503(無法使用)時，Dispatcher會傳送要求給不同的演算。
* When a request to a render returns HTTP status 50x (other than 503), Dispatcher sends a request for the page that is configured for the `health_check` property.

   * 如果health check傳回500(INTERNAL_ SERVER_ ERROR)，則Dispatcher會傳送原始請求給不同的演算。
   * 如果healthcheck傳回HTTP狀態200，Dispatcher會傳回初始HTTP500錯誤給用戶端。

若要啓用容錯，請將下列行新增至農場(或網站)：

```xml
/failover "1" 
```

>[!NOTE]
>
>To retry HTTP requests that contain a body, Dispatcher sends a `Expect: 100-continue` request header to the render before spooling the actual contents. CQ5.5與CQSE搭配使用，然後以100(READED)或錯誤碼立即回答。其他servlet容器也應該支援此功能。

## Ignoring Interruption Errors - /ignoreEINTR {#ignoring-interruption-errors-ignoreeintr}

>[!CAUTION]
>
>通常不需要此選項。當您看到下列記錄訊息時，才需要使用此項目：
>
>`Error while reading response: Interrupted system call`

Any file system oriented system call can be interrupted `EINTR` if the object of the system call is located on a remote system accessed via NFS. 這些系統呼叫是否可逾時或中斷，是根據底層檔案系統在本機電腦上的掛載方式。

如果您的例項具有此設定，則使用Social參數，記錄檔包含下列訊息：

`Error while reading response: Interrupted system call`

在內部，Dispatcher會使用可呈現為：

`while (response not finished) {  
read more data  
}`

Such messages can be generated when the `EINTR` occurs in the &quot; `read more data`&quot; section and are caused by the reception of a signal before any data was received.

To ignore such interrupts you can add the following parameter to `dispatcher.any` (before `/farms`):

`/ignoreEINTR "1"`

Setting `/ignoreEINTR` to `"1"` causes Dispatcher to continue to attempt to read data until the complete response is read. 預設值為0並停用選項。

## Designing Patterns for glob Properties {#designing-patterns-for-glob-properties}

Several sections in the Dispatcher configuration file use `glob` properties as selection criteria for client requests. glob屬性的值是Dispatcher與請求方面的比較，例如請求資源的路徑，或用戶端的IP位址。For example, the items in the `/filter` section use glob patterns to identify the paths of the pages that Dispatcher acts on or rejects.

glob值可以包含萬用字元和英數字元，以定義模式。

| 萬用字元字元 | 說明 | 範例 |
|--- |--- |--- |
| `*` | 相符項目：字串中任何字元的零或更多連續例項。The final character of the match is determined by either of the following situations: <br/>A character in the string matches the next character in the pattern, and the pattern character has the following characteristics:<br/><ul><li>不是*</li><li>不是嗎？</li><li>常值字元(包括空格)或字元類別。</li><li>隨即到達圖樣結尾。</li></ul>在字元類別中，字元是由字元解譯。 | `*/geo*` 相符項目： `/content/geometrixx` 節點與 `/content/geometrixx-outdoors` 節點下方的任何頁面。The following HTTP requests match the glob pattern: <br/><ul><li>`"GET /content/geometrixx/en.html"`</li><li>`"GET /content/geometrixx-outdoors/en.html"` </li></ul><br/> `*outdoors/*`<br/>相符項目： `/content/geometrixx-outdoors` 節點下方的任何頁面。For example, the following HTTP request matches the glob pattern: <br/><ul><li>`"GET /content/geometrixx-outdoors/en.html"`</li></ul> |
| `?` | 比對任何單一字元。使用外部字元類別。在字元類別中，此字元是由字詞解譯。 | `*outdoors/??/*`<br/> 符合幾何xx戶外網站中任何語言的頁面。For example, the following HTTP request matches the glob pattern: <br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul><br/>下列請求不符合glob模式： <br/><ul><li>「GET/content/geometrixx-outdoors/en.html」</li></ul> |
| `[ and ]` | 標記字元類別的開始和結束。字元類別可包含一個或多個字元範圍和單一字元。<br/>如果目標字元符合字元類別中的任何字元或在定義的範圍內，則會發生相符項目。<br/>如果未包含封閉括號，則圖樣不會產生相符項目。 | `*[o]men.html*`<br/> 符合下列HTTP要求：<br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li></ul><br/>不符合下列HTTP要求：<br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul><br/> `*[o/]men.html*`<br/>符合下列HTTP要求： <br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul> |
| `-` | 表示各種字元範圍。用於字元類別。除了字元類別之外，此字元也是由字元解譯。 | `*[m-p]men.html*` 符合下列HTTP要求： <br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li></ul>Does not match the following HTTP request:<br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul> |
| `!` | 否定追隨的字元或字元類別。僅用於否定字元類別內的字元和字元範圍。Equivalent to the `^ wildcard`. <br/>除了字元類別之外，此字元也是由字元解譯。 | `*[!o]men.html*`<br/> 符合下列HTTP要求： <br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul><br/>不符合下列HTTP要求： <br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li></ul><br/>`*[!o!/]men.html*`<br/> 不符合下列HTTP要求：<br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"` 或 `"GET /content/geometrixx-outdoors/en/men. html"`</li></ul> |
| `^` | 否定追隨的字元或字元範圍。僅用於否定字元類別內的字元和字元範圍。Equivalent to the `!` wildcard character. <br/>除了字元類別之外，此字元也是由字元解譯。 | `!` 套用萬用字元的範例，將 `!` 字元替換為 `^` 字元圖樣中的字元。 |


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

## Logging {#logging}

在Web伺服器組態中，您可以設定：

* Dispatcher記錄檔的位置。
* 記錄層級。

如需詳細資訊，請參閱網頁伺服器文件和Dispatcher實例的讀我檔案。

**Apache旋轉/動態記錄檔**

If using an **Apache** web server you can use the standard functionality for rotated and/or piped logs. 例如，使用piped記錄檔：

`DispatcherLog "| /usr/apache/bin/rotatelogs logs/dispatcher.log%Y%m%d 604800"`

這會自動旋轉：

* dispatcher記錄檔；副檔名中的時間戳記(logs/dispatcher. log%Y%m%d)。
* (60x60x24x=604800秒)。

Please see the Apache web server documentation on Log Rotation and Piped Logs; for example [Apache 2.4](https://httpd.apache.org/docs/2.4/logs.html).

>[!NOTE]
>
>安裝時，預設記錄層級很高(即Level3=除錯)，因此Dispatcher會記錄所有錯誤和警告。這在初始階段非常有用。
>
>However, this requires additional resources, so when the Dispatcher is working smoothly *according to your requirements*, you can(should) lower the log level.

### Trace Logging {#trace-logging}

在Dispatcher的其他增強功能中，4.2.0版也引進了「追蹤記錄」。

這比除錯記錄更上層樓，在記錄檔中顯示其他資訊。它會新增記錄：

* 轉寄標題的值；
* 用於特定動作的規則。

You can enable Trace Logging by setting the log level to `4` in your web server.

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

## Confirming Basic Operation {#confirming-basic-operation}

若要確認web server、Dispatcher和AEM實例的基本操作與互動，您可以執行下列步驟：

1. Set the `loglevel` to `3`.

1. 啓動網頁伺服器；這也會啓動Dispatcher。
1. 啓動AEM實例。
1. 檢查您的網站伺服器和Dispatcher的記錄檔和錯誤檔案。\
   視您的Web伺服器而定，您應該會看見訊息，例如：\
   `[Thu May 30 05:16:36 2002] [notice] Apache/2.0.50 (Unix) configured`\
   和:\
   `[Fri Jan 19 17:22:16 2001] [I] [19096] Dispatcher initialized (build XXXX)`

1. 透過網頁伺服器瀏覽網站。確認內容已可視需要顯示。\
   For example, on a local installation where AEM runs on port `4502` and the web server on `80` access the Websites console using both:\
   ` https://localhost:4502/libs/wcm/core/content/siteadmin.html  
https://localhost:80/libs/wcm/core/content/siteadmin.html  
`結果應相同。確認使用相同機制的其他頁面存取權。

1. 檢查快取目錄是否已填滿。
1. 啓動頁面以檢查快取是否正被清除。
1. If everything is operating correctly you can reduce the `loglevel` to `0`.

## Using Multiple Dispatchers {#using-multiple-dispatchers}

在複雜的設定中，您可以使用多個Dispatchers。例如，您可以使用：

* 一個Dispatcher在內部網路發佈網站
* 另一個Dispatcher，位於不同的位址，並具有不同的安全性設定，可在網際網路上發佈相同的內容。

在這種情況下，請確定每個請求只執行一個Dispatcher。Dispatcher不處理來自其他Dispatcher的請求。因此，請確定這兩個Dispatcher都直接存取AEM網站。

## Debugging {#debugging}

When adding the header `X-Dispatcher-Info` to a request, Dispatcher answers whether the target was cached, returned from cached or not cacheable at all. The response header `X-Cache-Info` contains this information in a readable form. 您可以使用這些回應標題來除錯Dispatcher快取的回應問題。

This functionality is not enabled by default, so in order for the response header `X-Cache-Info` to be included, the farm must contain the following entry:

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

Below is a list containing the response headers that `X-Dispatcher-Info` will return:

* **快取**\
   目標檔案包含在快取中，而傳送程式已判斷為提供此檔案。
* **快取**\
   快取中不包含目標檔案，而傳送程式已判斷快取輸出並提供它。
* **快取：stat檔案太新，** 定位檔案包含在快取中，不過，它會被較新的statter檔案無效。dispatcher會刪除目標檔案，從輸出重新建立它並提供它。
* **不適用：無文件根目錄** 該農場的設定不包含文件根目錄(設定元素 `cache.docroot`)。
* **不適用：快取檔案路徑過長**\
   目標檔案-文件根目錄與URL檔案的串連-超過系統上最長的可能檔案名稱。
* **不適用：暫存檔案路徑過長**\
   暫存檔案名稱範本超過系統上最長的可能檔案名稱。在實際建立或覆寫快取檔案之前，發送器會先建立暫存檔案。The temporary file name is the target file name with the characters `_YYYYXXXXXX` appended to it, where the `Y` and `X` will be replaced to create a unique name.
* **不適用：請求URL沒有副檔名**\
   The request URL has no extension, or there is a path following the file extension, for example: `/test.html/a/path`.
* **不適用：請求不是GET或HEAD**
HTTP方法不是GET或HEAD。傳送程式假設輸出包含不應快取的動態資料。
* **不適用：請求包含查詢字串**\
   請求包含查詢字串。傳送程式假設輸出取決於給定的查詢字串，因此不會快取。
* **不適用：作業管理員未驗證**\
   The farm&#39;s cache is governed by a session manager (the configuration contains a `sessionmanagement` node) and the request didn&#39;t contain the appropriate authentication information.
* **不適用：請求包含授權**\
   The farm is not allowed to cache output ( `allowAuthorized 0`) and the request contains authentication information.
* **不適用：target is a directory**\
   目標檔案為目錄。This might point to some conceptual mistake, where a URL and some sub-URL both contain cacheable output, for example if a request to `/test.html/a/file.ext` comes first and contains cacheable output, the dispatcher will not be able to cache the output of a subsequent request to `/test.html`.
* **不適用：請求URL具有尾隨斜線**\
   請求URL具有結尾斜線。
* **不適用：請求URL不在快取規則中**\
   農場的快取規則明確拒絕快取某些請求URL的輸出。
* **不適用：授權檢查程式拒絕存取**\
   農場的授權檢查程式拒絕存取快取檔案。
* **不適用：作業不有效**，農場的快取由作業管理員管理(設定包含 `sessionmanagement` 節點)，且使用者的作業不會再有效。
* **不適用：回應包含`no_cache `** 遠端伺服器傳回 `Dispatcher: no_cache` 標題，禁止傳送程式快取輸出。
* **不適用：回應內容長度為零**，回應的內容長度為零；dispatcher不會建立零長度檔案。
