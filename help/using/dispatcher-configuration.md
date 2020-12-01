---
title: 設定 Dispatcher
description: 瞭解如何配置Dispatcher。
translation-type: tm+mt
source-git-commit: 6177dafa64d7c22f72ccb64e343b85f4ee133d73
workflow-type: tm+mt
source-wordcount: '8513'
ht-degree: 2%

---


# 設定 Dispatcher {#configuring-dispatcher}

>[!NOTE]
>
>Dispatcher 版本與 AEM 無關。如果您依循連結至 Dispatcher 文件，且該連結內嵌於舊版 AEM 的文件中，您可能會被重新導向至本頁。

以下各節介紹如何配置Dispatcher的各個方面。

## 支援IPv4和IPv6 {#support-for-ipv-and-ipv}

AEM和Dispatcher的所有元素都可安裝在IPv4和IPv6網路中。 請參見[IPV4和IPV6](https://experienceleague.adobe.com/docs/experience-manager-65/deploying/introduction/technical-requirements.html?lang=en#ipv-and-ipv)。

## Dispatcher Configuration Files {#dispatcher-configuration-files}

預設情況下，Dispatcher配置儲存在`dispatcher.any`文本檔案中，不過您可以在安裝期間更改此檔案的名稱和位置。

配置檔案包含一系列單值或多值屬性，這些屬性控制Dispatcher的行為：

* 屬性名稱的前置詞為正斜線`/`。
* 多值屬性使用大括弧`{ }`括住子項。

示例配置結構如下：

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

您可以包含其他對配置有貢獻的檔案：

* 如果您的設定檔案很大，您可以將它分割成數個較小的檔案（較容易管理），然後加入這些檔案。
* 包含自動生成的檔案。

例如，若要在/farms組態中包含myFarm.any檔案，請使用下列程式碼：

```xml
/farms
  {
  $include "myFarm.any"
  }
```

使用星號(`*`)作為萬用字元，指定要包含的檔案範圍。

例如，如果檔案`farm_1.any`到`farm_5.any`包含1到5的場的配置，則可以按如下方式包括它們：

```xml
/farms
  {
  $include "farm_*.any"
  }
```

## 使用環境變數{#using-environment-variables}

您可以在dispatcher.any檔案的字串值屬性中使用環境變數，而不是硬式編碼值。 要包含環境變數的值，請使用`${variable_name}`格式。

例如，如果dispatcher.any檔案與快取目錄位於同一目錄中，則可使用[docroot](#specifying-the-cache-directory)屬性的以下值：

```xml
/docroot "${PWD}/cache"
```

另一個範例是，如果您建立名為`PUBLISH_IP`的環境變數，以儲存AEM發佈例項的主機名稱，則可使用下列[/renders](#defining-page-renderers-renders)屬性組態：

```xml
/renders {
  /0001 {
    /hostname "${PUBLISH_IP}"
    /port "8443"
  }
}
```

## 命名Dispatcher實例{#naming-the-dispatcher-instance-name}

使用`/name`屬性指定唯一名稱以標識您的Dispatcher實例。 `/name`屬性是配置結構中的頂級屬性。

## 定義場{#defining-farms-farms}

`/farms`屬性定義一組或多組Dispatcher行為，其中每組行為與不同的網站或URL相關聯。 `/farms`屬性可包含單一農場或多個農場：

* 當您希望Dispatcher以相同方式處理所有網頁或網站時，請使用單一場。
* 當您的網站或不同網站的不同區域需要不同的Dispatcher行為時，可建立多個場。

`/farms`屬性是配置結構中的頂級屬性。 要定義群，請將子屬性添加到`/farms`屬性。 使用屬性名稱，可唯一標識Dispatcher實例中的群。

`/farmname`屬性是多值的，包含定義Dispatcher行為的其他屬性：

* 群套用之頁面的URL。
* 一或多個服務URL（通常為AEM發佈例項），用於轉譯檔案。
* 用於負載平衡多份檔案轉譯器的統計資料。
* 其他數種行為，例如要快取的檔案和位置。

值可包含任何英數字元(a-z, 0-9)。 以下示例顯示了名為`/daycom`和`/docsdaycom`的兩個場的骨架定義：

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
>如果您使用多個演算場，系統會自下而上評估清單。 在為您的網站定義[虛擬主機](#identifying-virtual-hosts-virtualhosts)時，這特別相關。

每個農場屬性都可包含下列子屬性：

| 屬性名稱 | 說明 |
|--- |--- |
| [/homepage](#specify-a-default-page-iis-only-homepage) | 預設首頁（選用）（僅限IIS） |
| [/clienders](#specifying-the-http-headers-to-pass-through-clientheaders) | 要傳遞的用戶端HTTP要求的標頭。 |
| [/virtualhosts](#identifying-virtual-hosts-virtualhosts) | 此場的虛擬主機。 |
| [/sessionmanagement](#enabling-secure-sessions-sessionmanagement) | 支援作業管理和驗證。 |
| [/renders](#defining-page-renderers-renders) | 提供轉譯頁面的伺服器（通常為AEM發佈例項）。 |
| [/filter](#configuring-access-to-content-filter) | 定義Dispatcher啟用訪問的URL。 |
| [/vanity_urls](#enabling-access-to-vanity-urls-vanity-urls) | 設定虛名URL的存取權。 |
| [/propagateSyndPost](#forwarding-syndication-requests-propagatesyndpost) | 支援轉送匯集請求。 |
| [/cache](#configuring-the-dispatcher-cache-cache) | 設定快取行為。 |
| [/statistics](#configuring-load-balancing-statistics) | 定義負載平衡計算的統計類別。 |
| [/stickyConnectionsFor](#identifying-a-sticky-connection-folder-stickyconnectionsfor) | 包含自黏檔案的資料夾。 |
| [/health_check](#specifying-a-health-check-page) | 用於確定伺服器可用性的URL。 |
| [/retryDelay](#specifying-the-page-retry-delay) | 重試失敗連接之前的延遲。 |
| [/unavailableDestamy](#reflecting-server-unavailability-in-dispatcher-statistics) | 影響負載平衡計算統計資料的罰款。 |
| [/failover](#using-the-failover-mechanism) | 當原始請求失敗時，將請求重新傳送至不同的轉譯。 |
| [/auth_checker](permissions-cache.md) | 如需權限相關快取，請參閱[快取保全內容](permissions-cache.md)。 |

## 指定預設頁面（僅限IIS）- /homepage {#specify-a-default-page-iis-only-homepage}

>[!CAUTION]
>
>`/homepage`參數（僅限IIS）不再有效。 您應改用[IIS URL Rewrite Module](https://docs.microsoft.com/en-us/iis/extensions/url-rewrite-module/using-the-url-rewrite-module)。
>
>如果您使用Apache，則應使用`mod_rewrite`模組。 如需`mod_rewrite`的相關資訊，請參閱Apache網站檔案（例如[Apache 2.4](https://httpd.apache.org/docs/current/mod/mod_rewrite.html)）。 當使用`mod_rewrite`時，建議使用標幟&#x200B;**[&#39;passthrough|PT&#39;（傳遞至下一個處理常式）](https://helpx.adobe.com/dispatcher/kb/DispatcherModReWrite.html)**&#x200B;來強制重寫引擎將內部`request_rec`結構的`uri`欄位設為`filename`欄位的值。

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

## 指定要傳遞至{#specifying-the-http-headers-to-pass-through-clientheaders}的HTTP標題

`/clientheaders`屬性定義Dispatcher從用戶端HTTP請求傳遞至轉譯器（AEM例項）的HTTP標題清單。

依預設，Dispatcher會將標準HTTP標頭轉送至AEM例項。 在某些情況下，您可能需要轉寄其他標題，或移除特定標題：

* 新增AEM例項在HTTP請求中預期的標題，例如自訂標題。
* 移除僅與Web伺服器相關的標題，例如驗證標題。

如果您自訂要傳遞的標題集，您必須指定完整的標題清單，包括通常預設包含的標題。

例如，處理發佈例項之頁面啟動請求的Dispatcher例項需要`/clientheaders`區段中的`PATH`標題。 `PATH`標頭允許複製代理與調度程式之間的通信。

以下代碼是`/clientheaders`的示例配置：

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

## 標識虛擬主機{#identifying-virtual-hosts-virtualhosts}

`/virtualhosts`屬性定義了Dispatcher為此場接受的所有主機名/URI組合的清單。 您可以使用星號(`*`)字元做為萬用字元。 / `virtualhosts`屬性的值使用下列格式：

```xml
[scheme]host[uri][*]
```

* `scheme`:（可選） `https://` 或  `https://.`
* `host`:主機的名稱或IP地址以及埠號（如果需要）。(請參閱[https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.23](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.23))
* `uri`:（可選）資源的路徑。

以下配置示例處理myCompany的。com和。ch域以及mySubDivision的所有域的請求：

```xml
   /virtualhosts
    {
    "www.myCompany.com"
    "www.myCompany.ch"
    "www.mySubDivison.*"
    }
```

以下配置處理&#x200B;*all*&#x200B;請求：

```xml
   /virtualhosts
    {
    "*"
    }
```

### 解析虛擬主機{#resolving-the-virtual-host}

當Dispatcher收到HTTP或HTTPS請求時，它會找到最符合請求`host,` `uri`和`scheme`標題的虛擬主機值。 Dispatcher會依下列順序評估`virtualhosts`屬性中的值：

* Dispatcher從最低的群開始，並在dispatcher.any檔案中向上進行。
* 對於每個群，Dispatcher從`virtualhosts`屬性中的最頂層值開始，並從值清單中繼續。

Dispatcher以下列方式查找最匹配的虛擬主機值：

* 使用第一個遇到的虛擬主機，它與請求的`host`、`scheme`和`uri`中的所有三個匹配。
* 如果沒有`virtualhosts`值具有與請求的`scheme`和`uri`都匹配的`scheme`和`uri`部分，則使用與請求的`host`匹配的第一個遇到的虛擬主機。
* 如果沒有`virtualhosts`值的主機部分與請求的主機匹配，則使用最頂端群的最頂端虛擬主機。

因此，您應將預設虛擬主機放在`dispatcher.any`檔案最頂端群中`virtualhosts`屬性的頂端。

### 虛擬主機解析度示例{#example-virtual-host-resolution}

以下示例代表`dispatcher.any`檔案中的一個代碼片段，該檔案定義兩個Dispatcher場，每個群定義一個`virtualhosts`屬性。

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

使用此示例時，下表顯示為給定HTTP請求解析的虛擬主機：

| 請求URL | 已解析的虛擬主機 |
|---|---|
| `https://www.mycompany.com/products/gloves.html` | `www.mycompany.com/products/` |
| `https://www.mycompany.com/about.html` | `www.mycompany.com` |

## 啟用安全會話- /sessionmanagement {#enabling-secure-sessions-sessionmanagement}

>[!CAUTION]
>
>`/allowAuthorized` **必** 須在節 `"0"` 中 `/cache` 設定為才能啟用此功能。

建立安全作業以存取轉譯群，讓使用者需要登入才能存取群中的任何頁面。 登入後，使用者可以存取群中的頁面。 有關將此功能與CUG一起使用的資訊，請參閱[建立關閉的用戶組](https://experienceleague.adobe.com/docs/experience-manager-65/administering/security/cug.html?lang=en#creating-the-user-group-to-be-used)。 此外，請在上線前參閱Dispatcher [ Security Checklist](/help/using/security-checklist.md)。

`/sessionmanagement`屬性是`/farms`的子屬性。

>[!CAUTION]
>
>如果網站的區段使用不同的存取需求，您需要定義多個場。

**/** sessionmanagementas幾個子參數：

**/directory** (mandatory)

儲存會話資訊的目錄。 如果目錄不存在，則建立該目錄。

>[!CAUTION]
>
> 配置目錄子參數&#x200B;**時，不要**&#x200B;指向根資料夾(`/directory "/"`)，因為它可能導致嚴重問題。 您應始終指定儲存會話資訊的資料夾的路徑。 例如：

```xml
/sessionmanagement
  {
  /directory "/usr/local/apache/.sessions"
  }
```

**/encode** （選用）

會話資訊的編碼方式。 使用`md5`加密使用md5演算法，或使用`hex`加密十六進位編碼。 如果您加密會話資料，則具有檔案系統訪問權限的用戶無法讀取會話內容。 預設值為`md5`。

**/header** （可選）

儲存授權資訊的HTTP標題或Cookie的名稱。 如果您將資訊儲存在http標題中，請使用`HTTP:<header-name>`。 若要將資訊儲存在Cookie中，請使用`Cookie:<header-name>`。 如果您未指定值`HTTP:authorization`，則會使用。

**/timeout** （可選）

作業在上次使用後逾時的秒數。 如果未指定`"800"`，則會話會在使用者上次要求後13分鐘多一點逾時。

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

## 定義頁面轉譯器{#defining-page-renderers-renders}

/renders屬性定義Dispatcher向其發送請求以呈現文檔的URL。 以下範例`/renders`區段會識別單一AEM例項以進行轉譯：

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

以下範例/renders區段識別與dispatcher在同一部電腦上執行的AEM例項：

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

下列範例/renders區段在兩個AEM例項之間平均分發演算請求：

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

### 呈現選項{#renders-options}

**/timeout**

指定存取AEM例項的連線逾時（以毫秒為單位）。 預設值為`"0"` ，導致Dispatcher無限期等待。

**/receiveTimeout**

指定允許回應花費的時間（以毫秒為單位）。 預設值為`"600000"` ，導致Dispatcher等待10分鐘。 設定`"0"`可完全消除超時。

如果在剖析回應標題時到達逾時，會傳回504（錯誤閘道）的HTTP狀態。 如果在讀取響應主體時到達超時，Dispatcher將向客戶端返回不完整的響應，但刪除可能已寫入的任何快取檔案。

**/ipv4**

指定Dispatcher是使用`getaddrinfo`函式（對於IPv6）還是`gethostbyname`函式（對於IPv4）來獲取渲染的IP地址。 值0會使用`getaddrinfo`。 值`1`會使用`gethostbyname`。 預設值為`0`。

`getaddrinfo`函式返回IP地址清單。 Dispatcher會重複地址清單，直到它建立TCP/IP連接。 因此，當演算主機名與多個IP位址關聯時，`ipv4`屬性很重要，而主機會回應`getaddrinfo`函式，傳回IP位址清單，其順序一律相同。 在這種情況下，您應使用`gethostbyname`函式，以便Dispatcher所連接的IP地址是隨機的。

Amazon Elastic Load Balancing(ELB)是一種服務，它以可能相同的順序IP位址清單回應getaddrinfo。

**/secure**

如果`/secure`屬性的值為`"1"`,Dispatcher會使用HTTPS與AEM例項通訊。 有關其他詳細資訊，另請參閱[將Dispatcher配置為使用SSL](dispatcher-ssl.md#configuring-dispatcher-to-use-ssl)。

**/always-resolve**

使用Dispatcher版本&#x200B;**4.1.6**，您可以按如下方式配置`/always-resolve`屬性：

* 設定為`"1"`時，它將解析每個請求的主機名（Dispatcher絕不會快取任何IP地址）。 由於需要額外呼叫來取得每個請求的主機資訊，因此可能會對效能造成輕微影響。
* 如果未設定屬性，預設會快取IP位址。

此外，此屬性也可用於您遇到動態IP解析度問題時，如下列範例所示：

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

## 設定內容存取{#configuring-access-to-content-filter}

使用`/filter`部分指定Dispatcher接受的HTTP請求。 所有其他請求都會以404錯誤碼（找不到頁面）傳回至網頁伺服器。 如果不存在`/filter`部分，則接受所有請求。

**注意：** statfile的請 [](#naming-the-statfile) 求一律拒絕。

>[!CAUTION]
>
>有關使用Dispatcher限制訪問時的進一步考慮事項，請參見[ Dispatcher Security Checklist](security-checklist.md)。 此外，請閱讀[AEM安全性檢查清單](https://experienceleague.adobe.com/docs/experience-manager-65/administering/security/security-checklist.html?lang=en#security)，以取得有關AEM安裝的其他安全性詳細資訊。

`/filter`區段由一系列規則組成，這些規則會根據HTTP請求的請求行部分的模式拒絕或允許訪問內容。 您應對`/filter`區段使用允許清單策略：

* 首先，拒絕訪問所有內容。
* 視需要允許存取內容。

### 定義過濾器{#defining-a-filter}

`/filter`區段中的每個項目都包括類型和模式，該類型和模式與請求行或整個請求行的特定元素匹配。 每個篩選器都可包含下列項目：

* **類型**:指 `/type` 出是否允許或拒絕對符合模式的請求的訪問。值可以是`allow`或`deny`。

* **請求行的元素：** 包含 `/method`、、 `/url`或 `/query` `/protocol` 根據HTTP請求請求請求行部分的這些特定部分篩選請求的模式。偏好的篩選方法是篩選請求行的元素（而非整個請求行）。

* **請求行的進階元素：** 從Dispatcher 4.2.0開始，有四個新的篩選元素可供使用。這些新元素分別為`/path`、`/selectors`、`/extension`和`/suffix`。 包含一或多個這些項目，以進一步控制URL模式。

>[!NOTE]
>
>如需有關每個元素所參考之請求行中哪一部分的詳細資訊，請參閱[Sling URL Decomposition](https://sling.apache.org/documentation/the-sling-engine/url-decomposition.html) wiki頁面。

* **glob屬性**:該 `/glob` 屬性用於與HTTP請求的整個請求行匹配。

>[!CAUTION]
>
>Dispatcher中不建議使用全域篩選。 因此，您應避免在`/filter`區段中使用全域，因為這可能會導致安全性問題。 所以，它不是：
>
>`/glob "* *.css *"`
>
>您應使用
>
>`/url "*.css"`

#### HTTP請求的請求行部分{#the-request-line-part-of-http-requests}

HTTP/1.1定義[request-line](https://www.w3.org/Protocols/rfc2616/rfc2616-sec5.html)如下：

`Method Request-URI HTTP-Version<CRLF>`

`<CRLF>`字元代表回歸的歸位字元，後面接著行動態消息。 以下範例是當客戶要求WKND網站的美文網頁時收到的請求行：

`GET /content/wknd/us/en.html HTTP.1.1<CRLF>`

您的模式必須考慮請求行中的空格字元和`<CRLF>`字元。

#### 雙引號與單引號{#double-quotes-vs-single-quotes}

建立篩選規則時，請使用雙引號`"pattern"`來建立簡單模式。 如果您使用Dispatcher 4.2.0或更新版本，而您的模式包含規則運算式，則必須將regex模式`'(pattern1|pattern2)'`括在單引號內。

#### 規則運算式{#regular-expressions}

在4.2.0版以後的Dispatcher版本中，您可以在篩選模式中包含POSIX Extended Regular Expressions。

#### 過濾器{#troubleshooting-filters}故障排除

如果您的篩選器未以預期的方式觸發，請啟用分派器上的[追蹤記錄](#trace-logging)，以便您查看哪個篩選器正在攔截請求。

#### 範例篩選：全部拒絕{#example-filter-deny-all}

以下示例過濾器部分使Dispatcher拒絕所有檔案的請求。 您應拒絕存取所有檔案，然後允許存取特定區域。

```xml
  /0001  { /glob "*" /type "deny" }
```

對明確拒絕區域的請求會導致傳回404錯誤碼（找不到頁面）。

#### 範例篩選：拒絕訪問特定區域{#example-filter-deny-access-to-specific-areas}

篩選器也可讓您拒絕存取各種元素，例如發佈例項中的ASP頁面和敏感區域。 下列篩選條件拒絕存取ASP頁面：

```xml
/0002  { /type "deny" /url "*.asp"  }
```

#### 範例篩選：啟用POST請求{#example-filter-enable-post-requests}

下列範例篩選器允許使用POST方法提交表單資料：

```xml
/filter {
    /0001  { /glob "*" /type "deny" }
    /0002 { /type "allow" /method "POST" /url "/content/[.]*.form.html" }
}
```

#### 範例篩選：允許訪問工作流控制台{#example-filter-allow-access-to-the-workflow-console}

下列範例顯示用於拒絕外部存取「工作流程」主控台的篩選器：

```xml
/filter {
    /0001  { /glob "*" /type "deny" }
    /0002  {  /type "allow"  /url "/libs/cq/workflow/content/console*"  }
}
```

如果您的發佈例項使用Web應用程式內容（例如發佈），您也可以將它新增至您的篩選定義。

```xml
/0003   { /type "deny"  /url "/publish/libs/cq/workflow/content/console/archive*"  }
```

如果您仍需要存取受限制區域內的單一頁面，則可以允許存取。 例如，要允許訪問「工作流」控制台中的「存檔」頁籤，請添加以下部分：

```xml
/0004  { /type "allow"  /url "/libs/cq/workflow/content/console/archive*"   }
```

>[!NOTE]
>
>當多個篩選模式套用至請求時，套用的最後一個篩選模式會生效。

#### 範例篩選：使用規則運算式{#example-filter-using-regular-expressions}

此篩選器使用規則運算式（定義於以下單引號之間）啟用非公開內容目錄的擴充功能：

```xml
/005  {  /type "allow" /extension '(css|gif|ico|js|png|swf|jpe?g)' }
```

#### 範例篩選：篩選請求URL {#example-filter-filter-additional-elements-of-a-request-url}的其他元素

以下是使用路徑、選擇器和擴充功能的篩選器，封鎖從`/content`路徑及其子樹擷取內容的規則範例：

```xml
/006 {
        /type "deny"
        /path "/content/*"
        /selectors '(feed|rss|pages|languages|blueprint|infinity|tidy)'
        /extension '(json|xml|html)'
        }
```

### 範例/filter部分{#example-filter-section}

在配置Dispatcher時，應盡可能限制外部訪問。 下列範例提供外部訪客的最低存取權：

* `/content`
* 其他內容，例如設計和用戶端資料庫；例如：

   * `/etc/designs/default*`
   * `/etc/designs/mydesign*`

建立篩選器後，請[測試頁面存取](#testing-dispatcher-security)，以確保您的AEM例項安全。

`dispatcher.any`檔案的以下`/filter`部分可作為[Dispatcher配置檔案的基礎。](#dispatcher-configuration-files)

此示例基於隨Dispatcher提供的預設配置檔案，並作為示例用於生產環境。 前置`#`的項目會停用（已註解），如果您決定啟用其中任一項目（移除該行的`#`），請務必小心，因為這會對安全性造成影響。

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
>當與Apache一起使用時，請根據Dispatcher模組的DispatcherUseProcessedURL屬性來設計篩選器URL模式。 （請參閱[Apache Web Server —— 為Dispatcher配置Apache Web Server ](dispatcher-install.md##apache-web-server-configure-apache-web-server-for-dispatcher)。）

>[!NOTE]
>
>有關動態媒體的篩選`0030`和`0031`適用於AEM 6.0及更新版本。

如果您選擇延伸存取權，請考慮下列建議：

* 如果您使用CQ 5.4版或舊版，對`/admin`的外部存取應一律為&#x200B;*完全*&#x200B;停用。

* 允許訪問`/libs`中的檔案時，必須小心。 應允許個人存取。
* 拒絕對複製配置的訪問，因此無法查看：

   * `/etc/replication.xml*`
   * `/etc/replication.infinity.json*`

* 拒絕對Google小工具的反向代理訪問：

   * `/libs/opensocial/proxy*`

視您的安裝而定，`/libs`、`/apps`或其他位置下可能會有額外的資源，必須提供這些資源。 您可以使用`access.log`檔案作為確定外部訪問的資源的一種方法。

>[!CAUTION]
>
>對控制台和目錄的訪問可能給生產環境帶來安全風險。 除非您有明確的理由，否則應保持停用狀態（加上註解）。

>[!CAUTION]
>
>如果您[在發佈環境中使用報表](https://experienceleague.adobe.com/docs/experience-manager-65/administering/operations/reporting.html?lang=en#using-reports-in-a-publish-environment)，應將Dispatcher配置為拒絕外部訪客對`/etc/reports`的訪問。

### 限制查詢字串{#restricting-query-strings}

自Dispatcher 4.1.5版起，請使用`/filter`區段來限制查詢字串。 強烈建議您透過`allow`篩選元素明確允許查詢字串並排除一般允許。

單個條目可以具有`glob`或`method`、`url`、`query`和`version`的某些組合，但不能同時具有這兩者。 以下示例允許`a=*`查詢字串並拒絕解析到`/etc`節點的URL的所有其它查詢字串：

```xml
/filter {
 /0001 { /type "deny" /method "POST" /url "/etc/*" }
 /0002 { /type "allow" /method "GET" /url "/etc/*" /query "a=*" }
}
```

>[!NOTE]
>
>如果規則包含`/query`，則只會比對包含查詢字串的請求，並比對所提供的查詢模式。
>
>在上例中，如果對`/etc`的請求沒有查詢字串，也應該允許，則需要下列規則：


```xml
/filter {  
>/0001 { /type "deny" /method “*" /url "/path/*" }  
>/0002 { /type "allow" /method "GET" /url "/path/*" }  
>/0003 { /type “deny" /method "GET" /url "/path/*" /query "*" }  
>/0004 { /type "allow" /method "GET" /url "/path/*" /query "a=*" }  
}  
```

### 測試Dispatcher Security {#testing-dispatcher-security}

Dispatcher篩選器應封鎖對AEM發佈例項上下列頁面和指令碼的存取權。 使用網頁瀏覽器嘗試以網站訪客的方式開啟下列頁面，並驗證是否傳回程式碼404。 如果取得其他結果，請調整您的篩選。

請注意，您應該會看到`/content/add_valid_page.html?debug=layout`的一般頁面演算。

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

在終端或命令提示符中發出以下命令，以確定是否啟用了匿名寫訪問。 您不應該能夠將資料寫入節點。

`curl -X POST "https://anonymous:anonymous@hostname:port/content/usergenerated/mytestnode"`

在終端或命令提示符中發出以下命令，嘗試使Dispatcher快取失效，並確保收到代碼404響應：

`curl -H "CQ-Handle: /content" -H "CQ-Path: /content" https://yourhostname/dispatcher/invalidate.cache`

## 啟用虛名URL的存取權{#enabling-access-to-vanity-urls-vanity-urls}

<!-- 

Comment Type: remark
Last Modified By: unknown unknown (sbroders@adobe.com)
Last Modified Date: 2015-03-25T14:23:05.185-0400

<p style="font-family: tahoma, arial, helvetica, sans-serif; font-size: 12px;">For https://jira.corp.adobe.com/browse/DOC-4812</p> 
<p style="font-family: tahoma, arial, helvetica, sans-serif; font-size: 12px;">The "com.adobe.granite.dispatcher.vanityurl.content" package needs to be made public before publishing this contnet.</p>
 -->

設定Dispatcher以啟用對您AEM頁面所設定虛名URL的存取權。

啟用虛名URL的存取時，Dispatcher會定期呼叫在演算例項上執行的服務，以取得虛名URL的清單。 Dispatcher將此清單儲存在本地檔案中。 當頁面請求因`/filter`區段中的篩選器而遭拒時，Dispatcher會查閱虛名URL的清單。 如果清單中有拒絕的URL,Dispatcher會允許存取虛名的URL。

若要啟用虛名URL的存取權，請新增`/vanity_urls`區段至`/farms`區段，類似下列範例：

```xml
 /vanity_urls {
      /url "/libs/granite/dispatcher/content/vanityUrls.html"
      /file "/tmp/vanity_urls"
      /delay 300
 }
```

`/vanity_urls`部分包含以下屬性：

* `/url`:在演算例項上執行的虛名URL服務的路徑。此屬性的值必須為`"/libs/granite/dispatcher/content/vanityUrls.html"`。

* `/file`:Dispatcher儲存虛名URL清單的本機檔案路徑。請確定Dispatcher對此檔案具有寫訪問權限。
* `/delay`:（秒）呼叫虛名URL服務之間的時間。

>[!NOTE]
>
>如果您的演算是AEM的例項，您必須從「軟體散發」安裝[VanityURLS-Components套件](https://experience.adobe.com/#/downloads/content/software-distribution/en/aem.html?package=/content/software-distribution/en/details.html/content/dam/aem/public/adobe/packages/granite/vanityurls-components)，以啟用虛名URL服務。 （如需詳細資訊，請參閱[軟體散發](https://experienceleague.adobe.com/docs/experience-manager-65/administering/contentmanagement/package-manager.html?lang=en#software-distribution)。）

請依照下列程式來啟用虛名URL的存取權。

1. 如果您的轉譯服務是AEM例項，請在發佈例項上安裝「com.adobe.granite.dispatcher.vanityurl.content」套件（請參閱上述附註）。
1. 針對您為AEM或CQ頁面設定的每個虛名URL，請確定[`/filter`](#configuring-access-to-content-filter)組態會拒絕URL。 如有必要，請新增拒絕URL的篩選器。
1. 在`/farms`下方新增`/vanity_urls`區段。
1. 重新啟動Apache Web伺服器。

## 轉發協同內容請求- /propagateSyndPost {#forwarding-syndication-requests-propagatesyndpost}

協同內容請求通常僅針對Dispatcher，因此依預設，它們不會傳送至轉譯器（例如AEM例項）。

如有必要，請將`/propagateSyndPost`屬性設定為`"1"` ，以將匯集請求轉發到Dispatcher。 若已設定，您必須確保篩選區段中不會拒絕POST要求。

## 配置Dispatcher快取- /cache {#configuring-the-dispatcher-cache-cache}

`/cache`部分控制Dispatcher如何快取文檔。 設定數個子屬性以實作快取策略：

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
>對於權限敏感型快取，請讀取[Caching Secured Content](permissions-cache.md)。

### 指定快取目錄{#specifying-the-cache-directory}

`/docroot`屬性標識儲存快取檔案的目錄。

>[!NOTE]
>
>該值必須與Web伺服器的文檔根路徑完全相同，這樣Dispatcher和Web伺服器才能處理相同的檔案。\
>Web伺服器負責在使用調度程式快取檔案時提供正確的狀態代碼，因此，Web伺服器也必須找到它。

如果您使用多個農場，則每個農場必須使用不同的檔案根目錄。

### 命名Statfile {#naming-the-statfile}

`/statfile`屬性標識要用作statfile的檔案。 Dispatcher使用此檔案註冊最近內容更新的時間。 statfile可以是Web伺服器上的任何檔案。

statfile沒有內容。 更新內容時，Dispatcher會更新時間戳記。 預設的statfile名為`.stat` ，並儲存在docroot中。 Dispatcher阻止對statfile的訪問。

>[!NOTE]
>
>如果配置了`/statfileslevel` ，則Dispatcher會忽略`/statfile`屬性，並使用`.stat`作為名稱。

### 發生錯誤時服務過時文檔{#serving-stale-documents-when-errors-occur}

`/serveStaleOnError`屬性控制當渲染伺服器返回錯誤時，Dispatcher是否返回無效的文檔。 預設情況下，當觸動statfile並使快取內容無效時，Dispatcher會在下次請求快取內容時刪除該內容。

如果`/serveStaleOnError`設定為`"1"` ，則Dispatcher不會從快取中刪除無效內容，除非渲染伺服器返回成功響應。 來自AEM的5xx回應或連線逾時會導致Dispatcher提供過時的內容，並回應及HTTP狀態111（重新驗證失敗）。

### 使用驗證時的快取{#caching-when-authentication-is-used}

`/allowAuthorized`屬性控制是否快取包含以下任何驗證資訊的請求：

* `authorization`標題
* 名為`authorization`的Cookie
* 名為`login-token`的Cookie

根據預設，包含此驗證資訊的請求不會快取，因為當快取檔案傳回給用戶端時，不會執行驗證。 此配置可防止Dispatcher向沒有必要權限的用戶提供快取的文檔。

不過，如果您的要求允許快取已驗證的檔案，請將`/allowAuthorized`設為：

`/allowAuthorized "1"`

>[!NOTE]
>
>要啟用會話管理（使用`/sessionmanagement`屬性）,`/allowAuthorized`屬性必須設定為`"0"`。

### 指定要快取的文檔{#specifying-the-documents-to-cache}

`/rules`屬性控制根據文檔路徑快取哪些文檔。 無論`/rules`屬性如何，Dispatcher都不會在下列情況下快取文檔：

* 如果請求URI包含問號(`?`)。
   * 這通常表示動態頁面，例如不需要快取的搜尋結果。
* 缺少檔案副檔名。
   * Web 伺服器需要副檔名來判斷文件類型 (MIME 類型)。
* 驗證標頭已設定 (這可以設定)。
* 如果AEM例項以下列標題回應：

   * `no-cache`
   * `no-store`
   * `must-revalidate`

>[!NOTE]
>
>GET 或 HEAD (用於 HTTP 標頭) 方法可讓 Dispatcher 快取。有關響應標頭快取的其他資訊，請參見[快取HTTP響應標頭](#caching-http-response-headers)部分。

`/rules`屬性中的每個項目都包含[`glob`](#designing-patterns-for-glob-properties)模式和類型：

* `glob`模式用於匹配文檔的路徑。
* 類型指示是否快取與`glob`模式匹配的文檔。 值可以是允許（快取檔案）或拒絕（永遠呈現檔案）。

如果您沒有動態頁面（超出上述規則已排除的頁面），您可以設定Dispatcher以快取所有內容。 此規則區段的外觀如下：

```xml
/rules
  {
    /0000  {  /glob "*"   /type "allow" }
  }
```

有關全局屬性的資訊，請參見[為全局屬性設計模式](#designing-patterns-for-glob-properties)。

如果您頁面中有某些區段是動態的（例如新聞應用程式），或是在關閉的使用者群組中，您可以定義例外：

>[!NOTE]
>
>不得快取已關閉的使用者群組，因為未檢查快取頁面的使用者權限。

```xml
/rules
  {
   /0000  { /glob "*" /type "allow" }
   /0001  { /glob "/en/news/*" /type "deny" }
   /0002  { /glob "*/private/*" /type "deny"  }
  }
```

**壓縮**

在Apache Web伺服器上，您可以壓縮快取的檔案。 壓縮允許Apache在客戶端請求時以壓縮形式返回文檔。 通過啟用Apache模組`mod_deflate`自動執行壓縮，例如：

```xml
AddOutputFilterByType DEFLATE text/plain
```

預設情況下，該模組與Apache 2.x一起安裝。

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

### 按資料夾級別{#invalidating-files-by-folder-level}使檔案失效

使用`/statfileslevel`屬性，根據快取檔案的路徑使其無效：

* Dispatcher會在每個資料夾中，從資料夾建立`.stat`檔案至您指定的層級。 docroot資料夾是0級。
* 通過觸摸`.stat`檔案，檔案將失效。 將`.stat`檔案的上次修改日期與快取文檔的上次修改日期進行比較。 如果`.stat`檔案較新，則會重新提取文檔。

* 當位於特定級別的檔案被無效時，從docroot **到**&#x200B;的&#x200B;**所有**`.stat`檔案都將被觸碰無效檔案或已配置的`statsfilevel`（以較小者為準）的級別。

   * 例如：如果您將`statfileslevel`屬性設為6，而某個檔案在5級失效，則每個從docroot到5的`.stat`檔案都將被觸碰。 繼續此範例，如果檔案在7級時失效，則每隔一次。 `stat` 檔案從docroot移至6時，將會觸動(自此 `/statfileslevel = "6"`起)。

只有沿著無效檔案的路徑&#x200B;**的資源**&#x200B;受到影響。 請考慮下列範例：網站使用結構`/content/myWebsite/xx/.`如果您將`statfileslevel`設為3，則會建立`.stat`檔案，如下所示：

* `docroot`
* `/content`
* `/content/myWebsite`
* `/content/myWebsite/*xx*`

當`/content/myWebsite/xx`中的檔案失效時，從docroot到`/content/myWebsite/xx`的每個`.stat`檔案都會被觸碰。 這僅適用於`/content/myWebsite/xx`，而不適用於`/content/myWebsite/yy`或`/content/anotherWebSite`。

>[!NOTE]
>
>通過發送額外的標題`CQ-Action-Scope:ResourceOnly`可以防止失效。 這可用於刷新特定資源，而不使快取的其他部分失效。 有關詳細資訊，請參見[此頁](https://adobe-consulting-services.github.io/acs-aem-commons/features/dispatcher-flush-rules/index.html)和[手動使Dispatcher Cache](https://experienceleague.adobe.com/docs/experience-manager-dispatcher/using/configuring/page-invalidate.html?lang=en#configuring)無效。

>[!NOTE]
>
>如果為`/statfileslevel`屬性指定值，則會忽略`/statfile`屬性。

### 自動使快取檔案{#automatically-invalidating-cached-files}無效

`/invalidate`屬性定義在更新內容時自動失效的文檔。

自動失效時，Dispatcher不會在內容更新後刪除快取檔案，但會在下次要求時檢查檔案的有效性。 在內容更新明確刪除之前，快取中未自動失效的檔案仍會保留在快取中。

自動失效通常用於HTML頁面。 HTML頁面通常包含其他頁面的連結，因此很難判斷內容更新是否會影響頁面。 若要確保內容更新時所有相關頁面都無效，請自動使所有HTML頁面無效。 下列設定會使所有HTML頁面失效：

```xml
  /invalidate
  {
   /0000  { /glob "*" /type "deny" }
   /0001  { /glob "*.html" /type "allow" }
  }
```

有關全局屬性的資訊，請參見[為全局屬性設計模式](#designing-patterns-for-glob-properties)。

此配置在`/content/wknd/us/en`激活時導致以下活動：

* 所有檔案都帶有pattern en。*已從`/content/wknd/us`資料夾中移除。
* `/content/wknd/us/en./_jcr_content`資料夾已移除。
* 不會立即刪除與`/invalidate`配置匹配的所有其他檔案。 當下次請求發生時，這些檔案將被刪除。 在我們的範例中，`/content/wknd.html`不會刪除，當要求`/content/wknd.html`時，會刪除它。

如果您提供自動產生的PDF和ZIP檔案供下載，則可能也必須自動使這些檔案失效。 配置示例如下所示：

```xml
/invalidate
  {
   /0000 { /glob "*" /type "deny" }
   /0001 { /glob "*.html" /type "allow" }
   /0002 { /glob "*.zip" /type "allow" }
   /0003 { /glob "*.pdf" /type "allow" }
  }
```

AEM與Adobe Analytics的整合可在您網站的`analytics.sitecatalyst.js`檔案中提供設定資料。 隨Dispatcher提供的`dispatcher.any`示例檔案包含此檔案的以下失效規則：

```xml
{
   /glob "*/analytics.sitecatalyst.js"  /type "allow"
}
```

### 使用自定義失效指令碼{#using-custom-invalidation-scripts}

`/invalidateHandler`屬性允許您定義一個指令碼，該指令碼被調用用於Dispatcher收到的每個失效請求。

調用時使用以下引數：

* Handle —— 已失效的內容路徑
* 動作——複製動作（例如啟動、停用）
* 動作範圍——複製動作的範圍（空白，除非傳送`CQ-Action-Scope: ResourceOnly`的標題，否則請參閱[從AEM](page-invalidate.md)停用快取頁面，以取得詳細資訊）

這可用於涵蓋許多不同的使用案例，例如使其他應用程式特定快取無效，或處理頁面的外部化URL及其在Adobe中的位置不符合內容路徑的案例。

下面的示例指令碼記錄對檔案的每個無效請求。

```xml
/invalidateHandler "/opt/dispatcher/scripts/invalidate.sh"
```

#### 示例失效處理程式指令碼{#sample-invalidation-handler-script}

```shell
#!/bin/bash

printf "%-15s: %s %s" $1 $2 $3>> /opt/dispatcher/logs/invalidate.log
```

### 限制可刷新快取{#limiting-the-clients-that-can-flush-the-cache}的客戶端

`/allowedClients`屬性定義允許刷新快取的特定客戶端。 所述環狀模式與所述IP匹配。

以下範例：

1. 拒絕訪問任何客戶端
1. 明確允許訪問localhost

```xml
/allowedClients
  {
   /0001 { /glob "*.*.*.*"  /type "deny" }
   /0002 { /glob "127.0.0.1" /type "allow" }
  }
```

有關全局屬性的資訊，請參見[為全局屬性設計模式](#designing-patterns-for-glob-properties)。

>[!CAUTION]
>
>建議您定義`/allowedClients`。
>
>如果未執行此操作，任何客戶端都可發出清除快取的調用；若重複執行此動作，可能會嚴重影響網站效能。

### 正在忽略URL參數{#ignoring-url-parameters}

`ignoreUrlParams`區段定義當決定頁面是快取還是從快取傳送時，會忽略哪些URL參數：

* 當請求URL包含全部忽略的參數時，會快取頁面。
* 當請求URL包含一或多個未忽略的參數時，不會快取頁面。

當頁面的參數被忽略時，會在第一次要求頁面時快取頁面。 後續的頁面請求會提供至快取的頁面，而不論請求中的參數值為何。

要指定要忽略的參數，請將全局規則添加到`ignoreUrlParams`屬性：

* 要忽略參數，請建立允許該參數的全局屬性。
* 若要防止快取頁面，請建立拒絕參數的全域屬性。

下列範例會使Dispatcher忽略`q`參數，以便快取包含q參數的請求URL:

```xml
/ignoreUrlParams
{
    /0001 { /glob "*" /type "deny" }
    /0002 { /glob "q" /type "allow" }
}
```

使用`ignoreUrlParams`值範例時，下列HTTP請求會因為`q`參數被忽略而快取頁面：

```xml
GET /mypage.html?q=5
```

使用範例`ignoreUrlParams`值時，下列HTTP要求會使至&#x200B;**not**&#x200B;的頁面快取，因為`p`參數不會被忽略：

```xml
GET /mypage.html?q=5&p=4
```

有關全局屬性的資訊，請參見[為全局屬性設計模式](#designing-patterns-for-glob-properties)。

### 快取HTTP回應標題{#caching-http-response-headers}

>[!NOTE]
>
>此功能適用於Dispatcher的&#x200B;**4.1.11**&#x200B;版本。

`/headers`屬性允許您定義Dispatcher將要快取的HTTP標頭類型。 在對未快取資源的第一個請求中，所有與配置值之一匹配的標頭（請參見下面的配置示例）都儲存在快取檔案旁邊的單獨檔案中。 在對快取資源的後續請求中，儲存的標題會新增至回應。

以下是預設組態的範例：

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
>另請注意，不允許使用檔案全域字元。 如需詳細資訊，請參閱[設計全域屬性的圖樣](#designing-patterns-for-glob-properties)。

>[!NOTE]
>
>如果您需要Dispatcher來儲存和傳送來自AEM的ETag回應標頭，請執行下列動作：
>
>* 在`/cache/headers`區段中新增標題名稱。
>* 在與Dispatcher相關的部分中添加以下[Apache指令](https://httpd.apache.org/docs/2.4/mod/core.html#fileetag):

>
>
```xml
>FileETag none
>```

### Dispatcher Cache檔案權限{#dispatcher-cache-file-permissions}

`mode`屬性指定哪些檔案權限應用於快取中的新目錄和檔案。 此設定受調用進程的`umask`限制。 它是由下列一個或多個值之和構成的八位數：

* `0400` 允許所有者讀取。
* `0200` 允許由所有者寫入。
* `0100` 允許所有者在目錄中搜索。
* `0040` 允許由組成員讀取。
* `0020` 允許按組成員寫入。
* `0010` 允許組成員在目錄中搜索。
* `0004` 允許他人閱讀。
* `0002` 允許他人寫。
* `0001` 允許其他人在目錄中搜索。

預設值為`0755`，允許所有者讀取、寫入或搜索組和其他成員讀取或搜索。

### 與{#throttling-stat-file-touching}接觸的調節。stat檔案

使用預設的`/invalidate`屬性，每次啟動都會有效地使所有`.html`檔案失效（當檔案的路徑符合`/invalidate`區段時）。 在流量可觀的網站上，多次後續啟動會增加後端的CPU負載。 在這種情況下，最好「限制」`.stat`檔案觸控，以保持網站回應速度。 您可以使用`/gracePeriod`屬性來執行此動作。

`/gracePeriod`屬性定義在上次啟動後，過時、自動失效的資源仍可從快取中服務的秒數。 該屬性可用於設定中，否則，批次激活將反複使整個快取失效。 建議值為2秒。

如需詳細資訊，請閱讀上述的`/invalidate`和`/statfileslevel`章節。

### 配置基於時間的快取失效- /enableTTL {#configuring-time-based-cache-invalidation-enablettl}

如果設定，`/enableTTL`屬性將評估後端的響應標頭，如果這些標頭包含`Cache-Control`最大使用期或`Expires`日期，則會建立快取檔案旁的輔助空檔案，修改時間等於到期日。 當快取檔案在修改時間之後被要求時，會自動從後端重新要求。

>[!NOTE]
>
>此功能適用於Dispatcher的&#x200B;**4.1.11**&#x200B;版或更新版本。

## 配置負載平衡- /statistics {#configuring-load-balancing-statistics}

`/statistics`部分定義了Dispatcher為其評分每個渲染響應的檔案類別。 Dispatcher使用分數來判斷要傳送請求的演算。

您建立的每個類別都會定義全域模式。 Dispatcher將請求內容的URI與這些模式進行比較，以確定請求內容的類別：

* 類別的順序決定它們與URI的比較順序。
* 與URI匹配的第一個類別模式是檔案的類別。 不再評估類別模式。

Dispatcher最多支援8個統計類別。 如果您定義8個以上的類別，則僅使用前8個類別。

**演算選擇**

每次Dispatcher需要渲染頁時，它都使用以下演算法來選擇渲染：

1. 如果請求包含`renderid` Cookie中的演算名稱，Dispatcher會使用該演算。
1. 如果請求未包含`renderid` Cookie,Dispatcher會比較演算統計資料：

   1. Dispatcher會決定請求URI的類別。
   1. Dispatcher會決定哪個演算具有該類別最低的回應分數，並選取該演算。

1. 如果尚未選取任何演算，請使用清單中的第一個演算。

渲染類別的分數基於以前的響應時間，以及Dispatcher嘗試的以前失敗和成功連接。 對於每次嘗試，會更新所請求URI類別的分數。

>[!NOTE]
>
>如果不使用負載平衡，則可以忽略此部分。

### 定義統計類別{#defining-statistics-categories}

為要保留用於渲染選擇的統計資訊的每種類型的文檔定義類別。 `/statistics`節包含`/categories`節。 要定義類別，請在`/categories`節下添加一行，該行的格式如下：

`/name { /glob "pattern"}`

類別`name`必須是農場的唯一類別。 `pattern`在[設計全局屬性的模式部分中有說明。](#designing-patterns-for-glob-properties)

要確定URI的類別，Dispatcher會將URI與每個類別模式進行比較，直到找到匹配項。 Dispatcher從清單中的第一個類別開始，並按順序繼續。 因此，請先放置具有更具體模式的類別。

例如，預設`dispatcher.any`檔案的Dispatcher定義了HTML類別和其它類別。 HTML類別更具體，因此會先顯示：

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

### 反映Dispatcher Statistics中的伺服器不可用{#reflecting-server-unavailability-in-dispatcher-statistics}

`/unavailablePenalty`屬性設定當與渲染器的連接失敗時，應用到渲染統計資訊的時間（以十分之一秒為單位）。 Dispatcher將時間添加到與請求的URI匹配的統計資訊類別。

例如，當無法建立指定主機名稱／埠的TCP/IP連線時，會套用此懲罰，因為AEM未執行（且未監聽），或因為網路相關問題。

`/unavailablePenalty`屬性是`/farm`節的直接子項（`/statistics`節的同級項）。

如果不存在`/unavailablePenalty`屬性，則使用`"1"`值。

```xml
/unavailablePenalty "1"
```

## 識別嚴格連線資料夾- /stickyConnectionsFor {#identifying-a-sticky-connection-folder-stickyconnectionsfor}

`/stickyConnectionsFor`屬性定義一個包含自黏檔案的檔案夾；這將使用URL加以存取。 Dispatcher會從單一使用者將位於此資料夾中的所有請求傳送至相同的演算例項。 嚴格連線可確保所有檔案都有一致的作業資料。 此機制使用`renderid` Cookie。

以下範例定義與/products資料夾的嚴格連線：

```xml
/stickyConnectionsFor "/products"
```

當頁面由來自多個內容節點的內容組成時，請包含`/paths`屬性，該屬性列出內容的路徑。 例如，頁面包含`/content/image`、`/content/video`和`/var/files/pdfs`的內容。 下列設定可啟用頁面上所有內容的嚴格連線：

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

啟用自黏連線時，分派程式模組會設定`renderid` Cookie。 此Cookie沒有`httponly`標幟，應加入此標幟以增強安全性。 通過在`dispatcher.any`配置檔案的`/stickyConnections`節點中設定`httpOnly`屬性，可以執行此操作。 屬性的值（`0`或`1`）定義`renderid` Cookie是否附加了`HttpOnly`屬性。 預設值為`0`，表示不會新增屬性。

有關`httponly`標誌的其他資訊，請閱讀[此頁](https://www.owasp.org/index.php/HttpOnly)。

### 安全{#secure}

啟用自黏連線時，分派程式模組會設定`renderid` Cookie。 此Cookie沒有`secure`標幟，應加入此標幟以增強安全性。 通過在`dispatcher.any`配置檔案的`/stickyConnections`節點中設定`secure`屬性，可以執行此操作。 屬性的值（`0`或`1`）定義`renderid` Cookie是否附加了`secure`屬性。 預設值為`0`，這表示如果傳入請求安全，則將添加&#x200B;**屬性。**&#x200B;如果值設定為`1`，則無論傳入請求是否安全，都將添加安全標誌。

## 處理渲染連接錯誤{#handling-render-connection-errors}

當演算伺服器傳回500錯誤或無法使用時，設定Dispatcher行為。

### 指定運行狀況檢查頁{#specifying-a-health-check-page}

使用`/health_check`屬性來指定當發生500個狀態代碼時被檢查的URL。 如果此頁還返回500狀態代碼，則該實例被認為不可用，並且在重試之前將可配置的時間補償(`/unavailablePenalty`)應用於渲染。

```xml
/health_check
  {
  # Page gets contacted when an instance returns a 500
  /url "/health_check.html"
  }
```

### 指定頁面重試延遲{#specifying-the-page-retry-delay}

`/retryDelay`屬性設定Dispatcher在群呈現的連接嘗試輪次之間等待的時間（以秒為單位）。 對於每輪，Dispatcher嘗試連接到渲染器的最大次數是場中的渲染次數。

如果未明確定義`/retryDelay`，則Dispatcher使用`"1"`值。 在大多數情況下，預設值都適合。

```xml
/retryDelay "1"
```

### 配置重試次數{#configuring-the-number-of-retries}

`/numberOfRetries`屬性設定Dispatcher在轉譯時執行的連接嘗試的最大輪數。 如果Dispatcher在此次重試次數後無法成功連接到渲染器，Dispatcher將返回失敗的響應。

對於每輪，Dispatcher嘗試連接到渲染器的最大次數是場中的渲染次數。 因此，Dispatcher嘗試連接的最大次數是(`/numberOfRetries`)x（渲染次數）。

如果未明確定義該值，則預設值為`5`。

```xml
/numberOfRetries "5"
```

### 使用故障切換機制{#using-the-failover-mechanism}

啟用Dispatcher群上的故障切換機制，在原始請求失敗時將請求重新發送到不同的呈現。 啟用故障切換後，Dispatcher具有以下行為：

* 當對演算的請求傳回HTTP狀態503(UNAVAILABLE)時，Dispatcher會將請求傳送至不同的演算。
* 當對演算的請求傳回HTTP狀態50x（503除外）時，Dispatcher會傳送為`health_check`屬性所設定之頁面的請求。
   * 如果運行狀況檢查返回500(INTERNAL_SERVER_ERROR),Dispatcher會將原始請求發送到不同的渲染器。
   * 如果Healtch檢查返回HTTP狀態200,Dispatcher會將初始HTTP 500錯誤返回給客戶端。

要啟用故障切換，請將以下行添加到群（或網站）:

```xml
/failover "1"
```

>[!NOTE]
>
>要重試包含內文的HTTP請求，Dispatcher會在假設實際內容之前，將`Expect: 100-continue`請求標頭髮送到渲染器。 含CQSE的CQ 5.5會立即回答100（繼續）或錯誤碼。 其他servlet容器也應支援此功能。

## 正在忽略中斷錯誤- /ignoreEINTR {#ignoring-interruption-errors-ignoreeintr}

>[!CAUTION]
>
>通常不需要此選項。 只有在您看到下列日誌消息時，才需要使用此選項：
>
>`Error while reading response: Interrupted system call`

如果系統調用的對象位於通過NFS訪問的遠程系統上，則任何面向檔案系統的系統調用都可能被中斷`EINTR`。 這些系統呼叫是否可逾時或中斷，取決於本機機器上安裝基礎檔案系統的方式。

如果實例具有此類配置且日誌包含以下消息，請使用`/ignoreEINTR`參數：

`Error while reading response: Interrupted system call`

在內部，Dispatcher使用可表示為：

```text
while (response not finished) {  
read more data  
}
```

當`EINTR`出現在&quot; `read more data`&quot;部分中時，可以生成這樣的消息，其原因是在接收到任何資料之前接收到信號。

要忽略此類中斷，可將以下參數添加到`dispatcher.any`（`/farms`之前）:

`/ignoreEINTR "1"`

將`/ignoreEINTR`設定為`"1"`會導致Dispatcher繼續嘗試讀取資料，直到讀取完整響應。 預設值為`0`，並停用選項。

## 設計全局屬性的模式{#designing-patterns-for-glob-properties}

Dispatcher配置檔案中的幾個部分使用`glob`屬性作為客戶端請求的選擇標準。 `glob`屬性的值是Dispatcher與請求的一個方面進行比較的模式，如所請求資源的路徑或客戶機的IP地址。 例如，`/filter`節中的項使用`glob`模式來標識Dispatcher所操作或拒絕的頁的路徑。

`glob`值可包含通配符和字母數字字元以定義模式。

| 萬用字元 | 說明 | 範例 |
|--- |--- |--- |
| `*` | 相符項目：字串中任何字元的零個或多個連續例項。 符合的最終字元由下列任一情況決定：字串中的<br/>字元與模式中的下一個字元相符，而模式字元具有下列特性：<br/><ul><li>不是*</li><li>不是？</li><li>常值字元（包括空格）或字元類別。</li><li>到達模式的結尾。</li></ul>在字元類中，字元將逐字解釋。 | `*/geo*` 與節點和節點下 `/content/geometrixx` 的任何頁 `/content/geometrixx-outdoors` 面匹配。下列HTTP請求與全域模式相符：<br/><ul><li>`"GET /content/geometrixx/en.html"`</li><li>`"GET /content/geometrixx-outdoors/en.html"` </li></ul><br/> `*outdoors/*` <br/>與節點下的任何頁 `/content/geometrixx-outdoors` 面相符。例如，下列HTTP要求符合全域模式：<br/><ul><li>`"GET /content/geometrixx-outdoors/en.html"`</li></ul> |
| `?` | 符合任何單一字元。 使用外部字元類別。 在字元類中，該字元將按字面方式解釋。 | `*outdoors/??/*`<br/> 相符項目：geometrixx-outdoors網站中任何語言的頁面。例如，下列HTTP要求符合全域模式：<br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul><br/>下列請求不符合全域模式：  <br/><ul><li>&quot;取得/content/geometrixx-outdoors/en.html&quot;</li></ul> |
| `[ and ]` | 定義字元類的開頭和結尾。 字元類別可包含一或多個字元範圍和單一字元。<br/>如果目標字元符合字元類別中的任何字元，或在定義的範圍內，就會發生相符。<br/>如果未包括右括弧，則陣列不會產生匹配。 | `*[o]men.html*`<br/> 符合下列HTTP要求：<br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li></ul><br/>不符合下列HTTP要求：<br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul><br/> `*[o/]men.html*` <br/>符合下列HTTP要求：  <br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul> |
| `-` | 表示字元範圍。 用於字元類。  在字元類之外，將逐字解釋該字元。 | `*[m-p]men.html*` 符合下列HTTP要求：  <br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li></ul>不符合下列HTTP要求：<br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul> |
| `!` | 否定後面的字元或字元類。 僅用於否定字元類別中的字元和字元範圍。 相當於`^ wildcard`。 <br/>在字元類之外，將逐字解釋該字元。 | `*[!o]men.html*`<br/> 符合下列HTTP要求：  <br/><ul><li>`"GET /content/geometrixx-outdoors/en/men.html"`</li></ul><br/>不符合下列HTTP要求：  <br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"`</li></ul><br/>`*[!o!/]men.html*`<br/> 不符合下列HTTP要求：<br/><ul><li>`"GET /content/geometrixx-outdoors/en/women.html"` 或 `"GET /content/geometrixx-outdoors/en/men. html"`</li></ul> |
| `^` | 否定後面的字元或字元範圍。 僅用於否定字元類內的字元和字元範圍。 等效於`!`通配符。 <br/>在字元類之外，將逐字解釋該字元。 | `!`萬用字元的範例會套用，以`^`字元取代範例模式中的`!`字元。 |


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

**Apache已旋轉／管道記錄檔**

如果使用&#x200B;**Apache** Web伺服器，則可以對旋轉和／或管道日誌使用標準功能。 例如，使用管道日誌：

`DispatcherLog "| /usr/apache/bin/rotatelogs logs/dispatcher.log%Y%m%d 604800"`

這會自動旋轉：

* 調度程式日誌檔案；具有分機號(`logs/dispatcher.log%Y%m%d`)中的時間戳記。
* 每週（60 x 60 x 24 x 7 = 604800秒）。

請參閱「日誌輪替」和「管道日誌」上的Apache Web伺服器文檔；例如[Apache 2.4](https://httpd.apache.org/docs/2.4/logs.html)。

>[!NOTE]
>
>在安裝時，預設日誌級別為高（即級別3 =調試），因此Dispatcher會記錄所有錯誤和警告。 這在初期階段非常有用。
>
>但是，這需要額外的資源，因此，當Dispatcher根據您的要求&#x200B;*正常工作時，您可以（應該）降低日誌級別。*

### 跟蹤記錄{#trace-logging}

除了Dispatcher的其他增強功能外，4.2.0版還引入了跟蹤記錄。

這比「除錯」記錄檔更高，在記錄檔中顯示其他資訊。 它新增了下列項目的記錄：

* 轉發標題的值；
* 套用至特定動作的規則。

通過將Web伺服器中的日誌級別設定為`4` ，可以啟用跟蹤日誌。

以下是啟用跟蹤的日誌示例：

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

## 確認基本操作{#confirming-basic-operation}

若要確認Web伺服器、Dispatcher和AEM例項的基本操作與互動，您可使用下列步驟：

1. 將`loglevel`設為`3`。

1. 啟動Web伺服器；這也會啟動Dispatcher。
1. 啟動AEM例項。
1. 檢查Web伺服器和Dispatcher的日誌和錯誤檔案。
   * 視您的網頁伺服器而定，您應該會看到如下訊息：
      * `[Thu May 30 05:16:36 2002] [notice] Apache/2.0.50 (Unix) configured`與
      * `[Fri Jan 19 17:22:16 2001] [I] [19096] Dispatcher initialized (build XXXX)`

1. 透過網頁伺服器瀏覽網站。 確認內容是否依需要顯示。\
   例如，在本機安裝中，AEM在埠`4502`上執行，而在`80`上的Web伺服器會使用下列兩種方式存取網站主控台：
   * `https://localhost:4502/libs/wcm/core/content/siteadmin.html`
   * `https://localhost:80/libs/wcm/core/content/siteadmin.html`
   * 結果應該是一致的。 使用相同機制確認對其他頁面的存取。

1. 檢查是否填充了快取目錄。
1. 啟動頁面以檢查快取是否正確刷新。
1. 如果所有操作都正常，則可將`loglevel`降為`0`。

## 使用多個 Dispatcher {#using-multiple-dispatchers}

您可以透過複雜的設定來使用多個 Dispatcher。例如您可以使用：

* 一個 Dispatcher 在內部網路上發佈網站
* 另一個 Dispatcher 位於不同的位址下，並具有不同的安全設定，以便在網際網路上發佈相同的內容。

在這種情況下，請確定每個請求只通過一個 Dispatcher。Dispatcher 不會處理來自其他 Dispatcher 的請求。因此，請確定兩個 Dispatcher 都直接存取 AEM 網站。

## 調試{#debugging}

將標題`X-Dispatcher-Info`新增至請求時，Dispatcher會回答目標是否已快取、從快取中傳回或完全無法快取。 響應標題`X-Cache-Info`以可讀形式包含此資訊。 您可以使用這些回應標題來除錯與Dispatcher快取的回應相關的問題。

預設情況下，此功能未啟用，因此，為了包括響應標題`X-Cache-Info`，群必須包含以下條目：

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

此外，`X-Dispatcher-Info`標題不需要值，但如果您使用`curl`進行測試，則必須提供值才能傳送標題，例如：

```xml
curl -v -H "X-Dispatcher-Info: true" https://localhost/content/wknd/us/en.html
```

以下是包含`X-Dispatcher-Info`將傳回之回應標題的清單：

* **快取**\
   目標檔案包含在快取中，調度程式已確定傳送該檔案是有效的。
* **快取**\
   目標檔案不包含在快取中，而調度程式已確定快取輸出並傳送輸出是有效的。
* **快取：stat檔案較新。**
目標檔案包含在快取中，但是，它被較新的stat檔案使其失效。調度程式將刪除目標檔案，從輸出中重新建立併發送該檔案。
* **無法進行快取：no document**
root農場的組態不包含document root(configuration element) 
`cache.docroot`)。
* **無法進行快取：快取檔案路徑過長**\
   目標檔案——文檔根檔案和URL檔案的串連——超過系統上最長的檔案名。
* **無法進行快取：臨時檔案路徑過長**\
   臨時檔案名模板超過系統上可能的最長檔案名。 調度程式首先建立臨時檔案，然後實際建立或覆蓋快取檔案。 臨時檔案名是目標檔案名，其中附加了字元`_YYYYXXXXXX`，將替換`Y`和`X`以建立唯一名稱。
* **無法進行快取：請求URL沒有副檔名**\
   請求URL沒有副檔名，或是檔案副檔名後面有路徑，例如：`/test.html/a/path`。
* **無法進行快取：請求不是GET或**
HEAD HTTP方法既不是GET也不是HEAD。調度程式假定輸出將包含不應被快取的動態資料。
* **無法進行快取：包含查詢字串的請求**\
   請求包含查詢字串。 調度器假定輸出取決於給定的查詢字串，因此不進行快取。
* **無法進行快取：會話管理器未驗證**\
   群的快取由會話管理器管理（配置包含`sessionmanagement`節點），而請求不包含相應的驗證資訊。
* **無法進行快取：請求包含授權**\
   群不允許快取輸出(`allowAuthorized 0`)，請求包含驗證資訊。
* **無法進行快取：target是目錄**\
   目標檔案是目錄。 這可能會指出某些概念性錯誤，其中URL和某些子URL都包含可快取的輸出，例如，如果`/test.html/a/file.ext`的請求首先包含可快取的輸出，則調度程式將無法將後續請求的輸出快取到`/test.html`。
* **無法進行快取：請求URL有尾隨斜線**\
   請求URL有尾隨斜線。
* **無法進行快取：請求URL不在快取規則中**\
   群的快取規則會明確拒絕快取某些請求URL的輸出。
* **無法進行快取：授權驗證器拒絕訪問**\
   農場的授權檢查程式拒絕存取快取檔案。
* **無法進行快取：session not**
valid農場的快取由session manager(configuration contains a  `sessionmanagement` node)管理，且使用者的session is not or not extenary.
* **無法進行快取：響應包`no_cache`**
含遠程伺服器返回 
`Dispatcher: no_cache` 標題，禁止調度程式快取輸出。
* **無法進行快取：響應內容長**
度為零響應內容長度為零；調度程式將不建立零長度檔案。
