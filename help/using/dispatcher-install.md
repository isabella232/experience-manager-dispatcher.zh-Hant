---
title: 安裝Dispatcher
seo-title: 安裝AEM Dispatcher
description: 瞭解如何在Microsoft Internet Information Server、Apache Web Server和Sun Java Web Server-iPlanet上安裝Dispatcher模組。
seo-description: 瞭解如何在Microsoft Internet Information Server、Apache Web Server和Sun Java Web Server-iPlanet上安裝AEM Dispatcher模組。
uuid: 2384b907-1042-4707-b02 f-fba2125618 cf
contentOwner: 使用者
converted: 'true'
topic-tags: dispatcher
content-type: 引用
discoiquuid: f00ad731-6b95-4365-8500-e1 e0108 d9036
translation-type: tm+mt
source-git-commit: 6d3ff696780ce55c077a1d14d01efeaebcb8db28

---


# Installing Dispatcher {#installing-dispatcher}

<!-- 

Comment Type: draft

<h2>Introduction</h2>

 -->

>[!NOTE]
>
>Dispatcher版本與AEM獨立。如果您關注Dispatcher文件的連結(內嵌於舊版AEM的文件中)，可能會重新導向至此頁面。

Use the [Dispatcher Release Notes](release-notes.md) page to obtain the latest Dispatcher installation file for your operating system and web server. Dispatcher發行編號獨立於Adobe Experience Manager發行編號，並與Adobe Experience Manager6.x、5.x和Adobe CQ5.x版本相容。

使用下列檔案命名慣例：

`dispatcher-<web-server>-<operating-system>-<dispatcher-version-number>.<file-format>`

For example, the `dispatcher-apache2.4-linux-x86_64-ssl-4.3.1.tar.gz` file contains Dispatcher version 4.3.1 for an Apache 2.4 web server that runs on Linux i686 and is packaged using the **tar** format.

下表列出每個網頁伺服器檔案名稱中使用的伺服器識別碼：

| Web Server | 安裝套件 |
|--- |--- |
| Apache2.4 | dispatcher-apache **2.4**-&lt;other parameters&gt; |
| Microsoft Internet Information Server7.5、8、8.5 | dispatcher-**iis**-&lt;other parameters&gt; |
| Sun Java Web Server iPlanet | dispatcher-**ns**-&lt;other parameters&gt; |

>[!NOTE]
>
>您應安裝適用於平台的最新版Dispatcher。您必須依每年的方式升級Dispatcher實例，以使用最新版本來利用產品改進。

每個封存包含下列檔案：

* Dispatcher模組
* 範例組態檔
* README檔案，其中包含安裝指示和最後一分鐘資訊
* 列出已在目前和過去版本中修正問題的變更檔案

>[!NOTE]
>
>開始安裝之前，請先檢查AREME檔案是否有任何臨時變更/平台特定注意事項。

<!-- 

Comment Type: draft

<h3>Supported Web Servers</h3>

 -->

<!-- 

Comment Type: draft

<p>The following web servers are supported for use with Dispatcher version 4.1.12:</p>

 -->

<!-- 

Comment Type: draft

<p>The following sections detail the specific web server installation procedures.</p>

 -->

## Microsoft Internet Information Server {#microsoft-internet-information-server}

如需如何安裝此Web伺服器的詳細資訊，請參閱下列資源：

* Microsoft在Internet Information Server上的專屬說明文件
* [「The Official Microsoft IIS site」](https://www.iis.net/)

### Required IIS Components {#required-iis-components}

IIS8.5和10版需要安裝下列IIS元件：

* ISAPI擴充功能

此外，您必須新增Web Server(IIS)角色。使用伺服器管理員來新增角色和元件。

## Microsoft IIS - Installing the Dispatcher module {#microsoft-iis-installing-the-dispatcher-module}

Microsoft Internet Information System的必要封存為：

* `dispatcher-iis-<operating-system>-<dispatcher-release-number>.zip`

ZIP檔案包含下列檔案：

| 檔案 | 說明 |
|--- |--- |
| `disp_iis.dll` | Dispatcher動態連結程式庫檔案。 |
| `disp_iis.ini` | IIS的設定檔案。此範例可依您的需求進行更新。**注意**：ini檔案的名稱必須與dll相同。 |
| `dispatcher.any` | Dispatcher的設定檔範例。 |
| `author_dispatcher.any` | 用於搭配作者實例使用Dispatcher的設定檔。 |
| README | 讀我檔案，其中包含安裝指示和最後一分鐘資訊。**注意**：請先檢查此檔案，然後再開始安裝。 |
| 變更 | 列出在目前和過去版本中修正問題的變更檔案。 |

請使用下列程序將Dispatcher檔案複製到正確位置。

1. Use Windows Explorer to create the `<IIS_INSTALLDIR>/Scripts` directory, for example, `C:\inetpub\Scripts`.

1. 從Dispatcher套件擷取下列檔案至此指令碼目錄：

   * `disp_iis.dll`
   * `disp_iis.ini`
   * 下列其中一個檔案取決於Dispatcher是否使用AEM作者實例或發佈實例：
      * Author instance: `author_dispatcher.any`
      * Publish instance: `dispatcher.any`

## Microsoft IIS - Configure the Dispatcher INI File {#microsoft-iis-configure-the-dispatcher-ini-file}

Edit the `disp_iis.ini` file to configure the Dispatcher installation. `.ini` 檔案的基本格式如下：

```xml
[main]
configpath=<path to dispatcher.any>
loglevel=1|2|3
servervariables=0|1
replaceauthorization=0|1
```

下表說明每個屬性。

| 參數 | 說明 |
|--- |--- |
| configpath | The location of `dispatcher.any` within the local file system (absolute path). |
| logfile | `dispatcher.log` 檔案的位置。如果未設定，則記錄訊息會前往視窗事件記錄檔。 |
| loggel | 定義用來輸出訊息至事件記錄檔的記錄層級。The following values may be specified:Log level for the log file: <br/>0 - error messages only. <br/>1 - 錯誤和警告。<br/>2 - 錯誤、警告和資訊訊息 <br/>-錯誤、警告、資訊和除錯訊息。<br/>**注意**：建議在安裝和測試期間將記錄層級設為3，然後在生產環境中執行時將記錄層級設為0。 |
| 取代授權 | 指定如何處理HTTP請求中的授權標題。The following values are valid:<br/>0 - Authorization headers are not modified. <br/>1 - 取代名稱 `Basic <IIS:LOGON\_USER>` 為「基本」以外之「授權」以外的任何標題。<br/> |
| server變數 | 定義伺服器變數的處理方式。<br/>0 - IIS伺服器變數不會傳送給Dispatcher或AEM。<br/>1 - 所有IIS伺服器變數(例如 `LOGON\_USER, QUERY\_STRING, ...`)會傳送至Dispatcher，以及要求標頭(若未快取)。<br/>伺服器變數包括 `AUTH\_USER, LOGON\_USER, HTTPS\_KEYSIZE` 和其他許多。請參閱IIS文件以取得完整的變數清單，並詳細說明。 |
| enable_ chunked_ transfer | 定義是否啓用(1)或停用(0)區塊的用戶端回應。預設值為0。 |

設定範例：

```xml
[main]
configpath=C:\Inetpub\Scripts\dispatcher.any
loglevel=1
servervariables=1
replaceauthorization=0
```

### Configuring Microsoft IIS {#configuring-microsoft-iis}

設定IIS以整合Dispatcher ISAPI模組。在IIS中，您使用萬用字元應用程式對應。

### Configuring Anonymous Access - IIS 8.5 and 10 {#configuring-anonymous-access-iis-and}

「作者」例項上的預設刷新重新整理代理程式已設定，使其不會傳送具有衝印請求的安全性認證。因此，您使用Dispatcher快取的網站必須允許匿名存取。

如果您的網站使用驗證方法，則必須據以設定刷新代理程式。

1. 開啓IIS Manager，然後選取您使用Displer快取的網站。
1. 使用功能檢視模式，在IIS區段中按兩下驗證。
1. 如果未啓用「匿名驗證」，請選取「匿名驗證」，然後在「動作」區域按一下「啓用」。

### Integrating the Dispatcher ISAPI Module - IIS 8.5 and 10 {#integrating-the-dispatcher-isapi-module-iis-and}

請使用下列程序將Dispatcher ISAPI模組新增至IIS。

1. 開啓IIS Manager。
1. 選取您要當做Dispatcher快取使用的網站。
1. 使用功能檢視模式，在IIS區段中按兩下「Handler Mapping」。
1. 在「處理常式對應」頁面的「動作」面板中，按一下「新增萬用字元指令碼地圖」，新增下列屬性值，然後按一下「確定」：

   * 請求路徑：*
   * Executable: The absolute path of the disp_iis.dll file, for example `C:\inetpub\Scripts\disp_iis.dll`.
   * Name: A descriptive name for the handler mapping, for example `Dispatcher`.

1. 在出現的對話方塊中，若要將disp_ iis. dll程式庫新增至ISAPI和CGI限制清單，請按一下「是」。

   對於IIS7.0和7.5，設定已完成。如果您要設定IIS8.0，請繼續其餘步驟。

1. (IIS8.0)在處理常式對應的清單中，選取您剛才建立的處理常式，然後在「動作」區域按一下「編輯」。
1. (IIS8.0)在「編輯指令碼地圖」對話方塊中，按一下「請求限制」按鈕。
1. (IIS8.0)若要確保處理常式用於尚未快取的檔案和資料夾，請僅在「請求已映射至」時取消選取「叫用處理常式」，然後按一下「確定」。
1. (IIS8.0)在「編輯指令碼地圖」對話框中，按一下「確定」。

### Configuring Access to the Cache - IIS 8.5 and 10 {#configuring-access-to-the-cache-iis-and}

提供可寫入存取資料夾的預設App Pool使用者，此資料夾被用作Dispatcher快取。

1. Right-click the root folder of the website that you are using as the Dispatcher cache and click Properties, such as `C:\inetpub\wwwroot`.
1. 在「安全性」標籤上按一下「編輯」，然後按一下「權限」對話方塊中的「新增」。隨即開啓對話方塊以選取使用者帳戶。按一下「位置」按鈕，選取您的電腦名稱，然後按一下「確定」。

   當您完成下個步驟時，請將此對話方塊保持開啓。

1. 在IIS Manager中，選取您用於Dispatcher快取的IIS網站，然後在視窗右側按一下「進階設定」。
1. 選取「應用程式集區」屬性的值，然後將其複製到剪貼簿。
1. 返回開啓對話方塊。In the Enter The Object Names To Select box, type `IIS AppPool\` and then paste the contents of your clipboard. 此值看起來應像下列範例：

   `IIS AppPool\DefaultAppPool`

1. 按一下「檢查名稱」按鈕。當Windows解析使用者帳戶時，按一下「確定」。
1. In the Permissions dialog box for the dispatcher folder, select the account that you just added, enable all of the permissions for the account  **except for Full Control** and click OK. 按一下「確定」以關閉資料夾屬性對話方塊。

### Registering the JSON Mime Type - IIS 8.5 and 10 {#registering-the-json-mime-type-iis-and}

當您希望DSON呼叫允許DSON呼叫時，請使用下列程序來註冊JSON MIME類型。

1. 在IIS Manager中，選取您的網站並使用「功能檢視」，按兩下「MIME類型」。
1. 如果JSON擴充功能不在清單中，請在「動作」面板中按一下「新增」，輸入下列屬性值，然後按一下「確定」：

   * File Name Extension: `.json`
   * MIME Type: `application/json`

### Removing the bin Hidden Segment - IIS 8.5 and 10 {#removing-the-bin-hidden-segment-iis-and}

Use the following procedure to remove the `bin` hidden segment. 非新網站的網站可包含此隱藏區段。

1. 在IIS Manager中，選取您的網站並使用「功能檢視」，按兩下「請求篩選」。
1. Select the `bin` segment, click Remove, and in the confirmation dialog box click Yes.

### Logging IIS Messages to a File - IIS 8.5 and 10 {#logging-iis-messages-to-a-file-iis-and}

使用下列程序，將Dispatcher記錄訊息寫入記錄檔，而非Windows事件記錄檔。您需要設定Dispatcher使用記錄檔，並提供IIS對檔案的寫入存取權。

1. Use Windows Explorer to create a folder named `dispatcher` below the logs folder of the IIS installation. The path of this folder for a typical installation is `C:\inetpub\logs\dispatcher`.

1. 以滑鼠右鍵按一下dispatcher資料夾，然後按一下「屬性」。
1. 在「安全性」標籤上按一下「編輯」，然後按一下「權限」對話方塊中的「新增」。隨即開啓對話方塊以選取使用者帳戶。按一下「位置」按鈕，選取您的電腦名稱，然後按一下「確定」。

   當您完成下個步驟時，請將此對話方塊保持開啓。

1. 在IIS Manager中，選取您用於Dispatcher快取的IIS網站，然後在視窗右側按一下「進階設定」。
1. 選取「應用程式集區」屬性的值，然後將其複製到剪貼簿。
1. 返回開啓對話方塊。In the Enter The Object Names To Select box, type `IIS AppPool\` and then paste the contents of your clipboard. 此值看起來應像下列範例：

   `IIS AppPool\DefaultAppPool`

1. 按一下「檢查名稱」按鈕。當Windows解析使用者帳戶時，按一下「確定」。
1. 在dispatcher資料夾的「權限」對話框中，選取您剛新增的帳戶，啓用帳戶**以外的所有權限**，並按一下「完全控制」，然後按一下「確定」。按一下「確定」以關閉資料夾屬性對話方塊。
1. Use a text editor to open the `disp_iis.ini` file.
1. 新增類似下列範例的文字行，以設定記錄檔的位置，然後儲存檔案：

   ```xml
   logfile=C:\inetpub\logs\dispatcher\dispatcher.log
   ```

### Next Steps {#next-steps}

開始使用Dispatcher之前，您必須知道：

* [配置](dispatcher-configuration.md) Dispatcher
* [混淆AEM](page-invalidate.md) 以便搭配Dispatcher運作。

## Apache Web Server {#apache-web-server}

>[!CAUTION]
>
>Instructions for installation under both **Windows** and **Unix** are covered here. 執行步驟時請謹慎。

### Installing Apache Web Server {#installing-apache-web-server}

For Information about how to install an Apache Web Server read the installation manual - either [online](https://httpd.apache.org/) or in the distribution.

>[!CAUTION]
>
>If you are creating an Apache binary by compiling the source files, make sure that you turn on **dynamic modules support**. This can be done by using any of the **--enable-shared** options. At a minimum include the `mod_so` module.
>
>如需詳細資訊，請參閱Apache Web Server安裝手冊。

Also see the Apache HTTP Server [Security Tips](https://httpd.apache.org/docs/2.4/misc/security_tips.html) and [Security Reports](https://httpd.apache.org/security_report.html).

### Apache Web Server - Add the Dispatcher Module {#apache-web-server-add-the-dispatcher-module}

Dispatcher的用途如下：

* **Windows**：動態連結庫(DLL)
* **Unix**：動態共用物件(DSO)

安裝封存檔檔案包含下列檔案-視您選擇的是Windows還是Unix而定：

| 檔案 | 說明 |
|--- |--- |
| disp_ apache&lt; x. y&gt;. dll | Windows：Dispatcher動態連結程式庫檔案。 |
| dispatcher-apache&lt; x. y&gt;-&lt; rel-loc&gt;. so | Unix：Dispatcher共用物件程式庫檔案。 |
| mod_ dispatcher. so | Unix：範例連結。 |
| http. conf. disp&lt; x&gt; | Apache伺服器的設定檔。 |
| dispatcher. any | Dispatcher的設定檔範例。 |
| README | 讀我檔案，其中包含安裝指示和最後一分鐘資訊。**注意**：請先檢查此檔案，然後再開始安裝。 |
| 變更 | 列出在目前和過去版本中修正問題的變更檔案。 |

使用下列步驟將Dispatcher新增至Apache Web Server：

1. 將Dispatcher檔案置於適當的Apache模組目錄中：

   * **Windows**：Place `disp_apache<x.y>.dll``<APACHE_ROOT>/modules`
   * **Unix**：根據您的安裝找出 `<APACHE_ROOT>/libexec` 或 `<APACHE_ROOT>/modules`目錄。\
      Copy `dispatcher-apache<options>.so` into this directory.\
      To simplify long-term maintenance you can also create a symbolic link named `mod_dispatcher.so` to the Dispatcher:\
      `ln -s dispatcher-apache<x>-<os>-<rel-nr>.so mod_dispatcher.so`

1. Copy the dispatcher.any file to the `<APACHE_ROOT>/conf` directory.

   **注意：** 您可以將此檔案放置在不同位置，只要已設定Dispatcher模組的dispatchLog屬性即可。(請參閱下方的Dispatcher特定設定項目)。

### Apache Web Server - Configure SELinux Properties {#apache-web-server-configure-selinux-properties}

如果您在RedHat Linux Core2.6上執行的Dispatcher啓用了SelINUX，則可能會在dispatcher記錄檔中遇到錯誤訊息。

`Mon Jun 30 00:03:59 2013] [E] [16561(139642697451488)] Unable to connect to backend rend01 (10.122.213.248:4502): Permission denied`

這可能是因為啓用了SerialUX安全性。然後您需要執行下列工作：

* 設定dispatcher模組檔案的SeriLinux上下文。
* 啓用HTPD指令碼和模組以建立網路連線。
* 設定docroot的Selinux上下文，其中儲存快取的檔案。

Enter the following commands in a terminal window, replacing `[path to the dispatcher.so file]` with the path to the Dispatcher module that you installed to Apache Web Server, and *`path to the docroot`* with the path where the docroot is located (e.g. `/opt/cq/cache`):

```shell
semanage fcontext -a -t httpd_modules_t [path to the dispatcher.so file]
setsebool -P httpd_can_network_connect on
chcon -R --type httpd_sys_content_t [path to the docroot]
semanage fcontext -a -t httpd_sys_content_t "[path to the docroot](/.*)?"
```

### Apache Web Server - Configure Apache Web Server for Dispatcher {#apache-web-server-configure-apache-web-server-for-dispatcher}

The Apache Web Server needs to be configured, using `httpd.conf`. In the Dispatcher installation kit you will find an example configuration file named `httpd.conf.disp<x>`.

這些步驟為強制：

1. 導航到 `<APACHE_ROOT>/conf`.
1. Open `httpd.conf`for editing.
1. 必須依照列出順序新增下列組態項目：

   * **loadModule** 可在啓動時載入模組。
   * Dispatcher-specific configuration entries, including **DispatcherConfig, DispatcherLog** and **DispatcherLogLevel**.
   * **setHandler** 可啓動Dispatcher。**loadModule**.
   * **modMiMeusePathInfo** 可設定 **mod_ mime的行為**。

1. (選擇性)建議您變更htdocs目錄的擁有者：

   * Apache伺服器會以root的身分開始，但子程序會當做守護程式開始(出於安全目的)。The DocumentRoot (`<APACHE_ROOT>/htdocs`) must belong to the user daemon:

      ```xml
      cd <APACHE_ROOT>  
      chown -R daemon:daemon htdocs
      ```

**LoadModule**

下表列出可使用的範例；確切項目是根據您的Apache Web Server：

|  |  |
|--- |--- |
| Windows | `... LoadModule dispatcher_module modules\disp_apache.dll ...` |
| Unix(假設符號連結) | `... LoadModule dispatcher_module libexec/mod_dispatcher.so ...` |

>[!NOTE]
>
>每個陳述式的第一個參數必須如實寫入上述範例。
>
>請參閱提供的設定檔案範例以及Apache Web Server文件，以取得此命令的完整詳細資訊。

**Dispatcher特定組態項目**

Dispatcher專用的組態項目會放在LoadModule項目之後。下表列出適用於Unix和Windows的範例組態：

**Windows&amp; Unix**

```
...
<fModule disp_apache2.c>
DispatcherConfig conf/dispatcher.any
DispatcherLog logs/dispatcher.log DispatcherLogLevel 3
DispatcherNoServerHeader 0 DispatcherDeclineRoot 0
DispatcherUseProcessedURL 0
DispatcherPassError 0
DispatcherKeepAliveTimeout 60
</IfModule>
...
```

個別組態參數：

| 參數 | 說明 |
|--- |--- |
| DispatchConfig | Dispatcher組態檔的位置和名稱。<br/>當此屬性位於主要伺服器組態中時，所有虛擬主機都會繼承屬性值。不過，虛擬主機可以包含DispatchConfig屬性來覆寫主要伺服器組態。 |
| DispatcherLog | 記錄檔的位置和名稱。 |
| DispatcherLogel | Log level for the log file: <br/>0 - Errors <br/>1 - Warnings <br/>2 - Infos <br/>3 - Debug <br/>**Note**: It is recommended to set the log level to 3 during installation and testing, then to 0 when running in a production environment. |
| DispatcherNoServerHeader | *此參數已過時，不再有任何作用。*<br/><br/> 定義要使用的伺服器標題： <br/><ul><li>未定義或0- HTTP伺服器標題包含AEM版本。 </li><li>1 - Apache伺服器標題便會使用。</li></ul> |
| DispatcherDeclineoT | Defines whether to decline requests to the root &quot;/&quot;: <br/>**0** - accept requests to / <br/>**1** - requests to / are not handled by the dispatcher; use mod_alias for the correct mapping. |
| DispatcherUseProcesedul | Defines whether to use pre-processed URLs for all further processing by Dispatcher: <br/>**0** - use the original URL passed to the web server. <br/>**-** the dispatcher uses the URL algready processed by the handler that prefore dispatcher that prefore the dispatcher(i. e) `mod_rewrite`instead installer to the web server.例如，原始或處理的URL會與Dispatcher篩選器相符。URL也會當做快取檔案結構的基礎。如需mod_ rewrite的相關資訊，請參閱Apache網站文件；例如Apache2.4。使用mod_ rewrite時，建議您使用標幟「pasthrough」 | PT&#39;(傳遞至下一個處理常式)強制重寫引擎將內部request_ rec結構的URI欄位設定為檔案名稱欄位的值。 |
| DispatcherPaseror | Defines how to support error codes for ErrorDocument handling: <br/>**0** - Dispatcher spools all error responses to the client. <br/>**-** Dispatcher不會破壞用戶端的錯誤回應(狀態代碼大於或等於400)，但會將狀態代碼傳送至Apache，例如允許ErrendDocDocument指令處理此狀態碼。<br/>**程式碼範圍** -指定傳送回應給Apache的錯誤碼範圍。其他錯誤碼會傳遞給用戶端。例如，下列組態會將錯誤412的回應傳遞給用戶端，而所有其他錯誤都會傳遞至Apache：DispatcherPaserError400-411,413-417 |
| DispatcherMobileTimeout | 指定保持有效逾時的時間，單位為秒。從Dispatcher4.2.0開始，預設保留值為60。值為0，不會停用。 |
| DispatcherSocialonURL | 將此參數設定為「開啓」會將原始URL傳遞至後端，而非標準化的URL，並覆寫DispatcherUseProceDurl的設定。預設值為「關閉」。<br/>**注意**：Dispatcher組態中的篩選規則一律會對已淨化的URL進行評估，而不是原始URL。 |




>[!NOTE]
>
>路徑項目相對於Apache Web Server的根目錄。

>[!NOTE]
>
>The default settings for the Server Header are: `  
ServerTokens Full` `  
DispatcherNoServerHeader 0`\
顯示AEM版本(用於統計用途)。If you want to disable such information being available in the header you can set: `  
ServerTokens Prod`\
See the [Apache Documentation about ServerTokens Directive (for example, for Apache 2.4)](https://httpd.apache.org/docs/2.4/mod/core.html) for more information.

**setHandler**

After these entries you must add a **SetHandler** statement to the context of your configuration ( `<Directory>`, `<Location>`) for the Dispatcher to handle the incoming requests. 下列範例設定Dispatcher來處理整個網站的請求：

**Windows和Unix**

```
...  
<Directory />  
<IfModule disp\_apache2.c>  
SetHandler dispatcher-handler  
</IfModule>  
  
Options FollowSymLinks  
AllowOverride None  
</Directory>  
...
```

下列範例設定Dispatcher來處理虛擬網域的請求：

**Windows**

```
...  
<VirtualHost 123.45.67.89>  
ServerName www.mycompany.com  
DocumentRoot _\[cache-path\]_\\docs  
<Directory _\[cache-path\]_\\docs>  
<IfModule disp\_apache2.c>  
SetHandler dispatcher-handler  
</IfModule>  
AllowOverride None  
</Directory>  
</VirtualHost>  
...
```

**Unix**

```
...  
<VirtualHost 123.45.67.89>  
ServerName www.mycompany.com  
DocumentRoot /usr/apachecache/docs  
<Directory /usr/apachecache/docs>  
<IfModule disp\_apache2.c>  
SetHandler dispatcher-handler  
</IfModule>  
AllowOverride None  
</Directory>  
</VirtualHost>  
...
```

>[!NOTE]
**setHandler** 陳述式的參數必須與上述範例 *完全相同*，因為這是模組中定義的處理常式名稱。
請參閱提供的設定檔案範例以及Apache Web Server文件，以取得此命令的完整詳細資訊。

**ModMiMeusePathInfo**

After the **SetHandler** statement you should also add the **ModMimeUsePathInfo** definition.

>[!NOTE]
`ModMimeUsePathInfo` 只有當您使用Dispatcher4.0.9或更新版本時，才應使用和設定參數。
(注意，Dispatcher4.0.9版已於2011年推出。如果您使用舊版，則可升級至最近的Dispatcher版本。

The **ModMimeUsePathInfo** parameter should be set `On` for all Apache configurations:

`ModMimeUsePathInfo On`

The mod_mime module (see for example, [Apache Module mod_mime](https://httpd.apache.org/docs/2.4/mod/mod_mime.html)) is used to assign content metadata to the content selected for an HTTP response. 預設設定表示當mod_ mime決定內容類型時，只會考慮對應至檔案或目錄的URL部分。

When `On`, the `ModMimeUsePathInfo` parameter specifies that `mod_mime` is to determine the content type based on the *complete* URL; this means that virtual resources will have metainformation applied based on their extension.

The following example activates **ModMimeUsePathInfo**:

**Windows和Unix**

```
...  
<Directory />  
<IfModule disp\_apache2.c>  
SetHandler dispatcher-handler  
ModMimeUsePathInfo On  
</IfModule>  
  
Options FollowSymLinks  
AllowOverride None  
</Directory>  
...
```

### Enable Support for HTTPS (Unix and Linux) {#enable-support-for-https-unix-and-linux}

Dispatcher使用OpenSSL來建置透過HTTP的安全通訊。Starting from Dispatcher version **4.2.0**, OpenSSL 1.0.0 and OpenSSL 1.0.1 are supported. 依預設，Dispatcher會使用OpenSSL1.0.0。若要使用OpenSSL1.0.1，請使用下列程序建立符號連結，以便Dispatcher使用已安裝的OpenSSL程式庫。

1. 開啓終端並將目前目錄變更為安裝OpenSSL程式庫的目錄，例如：

   ```shell
   cd /usr/lib64
   ```

1. 若要建立符號連結，請輸入下列命令：

   ```shell
   ln -s libssl.so libssl.so.1.0.1
   ln -s libcrypto.so libcrypto.so.1.0.1
   ```

>[!NOTE]
If you are using a customized version of Apache, make sure Apache and Dispatcher are compiled using the same version of [OpenSSL](https://www.openssl.org/source/).

### Next Steps {#next-steps-1}

開始使用Dispatcher之前，您必須現在就：

* [配置](dispatcher-configuration.md) Dispatcher
* [混淆AEM](page-invalidate.md) 以便搭配Dispatcher運作。

## Sun Java System Web Server / iPlanet {#sun-java-system-web-server-iplanet}

>[!NOTE]
這裡涵蓋Windows和Unix環境的指示。
請謹慎選擇要執行的動作。

### Sun Java System Web Server / iPlanet - Installing your Web Server {#sun-java-system-web-server-iplanet-installing-your-web-server}

有關如何安裝這些Web伺服器的完整資訊，請參閱其個別文件：

* Sun Java System Web Server
* iPlanet Web Server

### Sun Java System Web Server / iPlanet - Add the Dispatcher Module {#sun-java-system-web-server-iplanet-add-the-dispatcher-module}

Dispatcher的用途如下：

* **Windows**：動態連結庫(DLL)
* **Unix**：動態共用物件(DSO)

安裝封存檔檔案包含下列檔案-視您選擇的是Windows還是Unix而定：

| 檔案 | 說明 |
|---|---|
| `disp_ns.dll` | Windows：Dispatcher動態連結程式庫檔案。 |
| `dispatcher.so` | Unix：Dispatcher共用物件程式庫檔案。 |
| `dispatcher.so` | Unix：範例連結。 |
| `obj.conf.disp` | iPlanet/Sun Java System網頁伺服器的設定檔。 |
| `dispatcher.any` | Dispatcher的設定檔範例。 |
| README | 讀我檔案，其中包含安裝指示和最後一分鐘資訊。注意：請先檢查此檔案，然後再開始安裝。 |
| 變更 | 列出在目前和過去版本中修正問題的變更檔案。 |

使用下列步驟將Dispatcher新增至您的Web伺服器：

1. Place the Dispatcher file in the web server&#39;s `plugin` directory:

### Sun Java System Web Server / iPlanet - Configure for the Dispatcher {#sun-java-system-web-server-iplanet-configure-for-the-dispatcher}

The web server needs to be configured, using `obj.conf`. In the Dispatcher installation kit you will find an example configuration file named `obj.conf.disp`.

1. 導航到 `<WEBSERVER_ROOT>/config`.
1. Open `obj.conf`for editing.
1. 複製開始的行：\
   `Service fn="dispService"`\
   from `obj.conf.disp` to the initialization section of `obj.conf`.

1. 儲存變更。
1. Open `magnus.conf` for editing.
1. 複製開始的兩行：\
   `Init funcs="dispService, dispInit"`\
   和\
   `Init fn="dispInit"`\
   from `obj.conf.disp` to the initialization section of `magnus.conf`.

1. 儲存變更。

>[!NOTE]
The following configurations should all be on one line and the `$(SERVER_ROOT)` and `$(PRODUCT_SUBDIR)` must be replaced by the respective values.

**Init**

下表列出可使用的範例；確切項目是根據您的特定網頁伺服器：

**Windows和Unix**

```
...  
Init funcs="dispService,dispInit" fn="load-modules" shlib="$(SERVER\_ROOT)/plugins/dispatcher.so"  
Init fn="dispInit" config="$(PRODUCT\_SUBDIR)/dispatcher.any" loglevel="1" logfile="$(PRODUCT\_SUBDIR)/logs/dispatcher.log"  
keepalivetimeout="60"  
...
```

其中：

| 參數 | 說明 |
|--- |--- |
| config | Location and name of the configuration file `dispatcher.any.` |
| logfile | 記錄檔的位置和名稱。 |
| loggel | Log level for when writing messages to the log file: <br/>**0** Errors <br/>**1** Warnings <br/>**2** Infos <br/>**3** Debug <br/>**Note:** It is recommended to set the log level to 3 during installation and testing and to 0 when running in a production environment. |
| socialalive逾out | 指定保持有效逾時的時間，單位為秒。從Dispatcher4.2.0開始，預設保留值為60。值為0，不會停用。 |

根據您的需求，您可以將Dispatcher定義為物件的服務。若要為整個網站設定Dispatcher，請修改預設物件：


**Windows**

```
...  
NameTrans fn="document-root" root="$(PRODUCT\_SUBDIR)\\dispcache"  
...  
Service fn="dispService" method="(GET|HEAD|POST)" type="\*\\\*"  
...
```

**Unix**

```
...  
NameTrans fn="document-root" root="$(PRODUCT\_SUBDIR)/dispcache"  
...  
Service fn="dispService" method="(GET|HEAD|POST)" type="\*/\*"  
...
```

### Next Steps {#next-steps-2}

開始使用Dispatcher之前，您必須現在就：

* [配置](dispatcher-configuration.md) Dispatcher
* [混淆AEM](page-invalidate.md) 以便搭配Dispatcher運作。
