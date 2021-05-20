---
title: 安裝 Dispatcher
seo-title: 安裝AEM Dispatcher
description: 了解如何在Microsoft Internet Information Server、Apache Web Server和Sun Java Web Server-iPlanet上安裝Dispatcher模組。
seo-description: 了解如何在Microsoft Internet Information Server、Apache Web Server和Sun Java Web Server-iPlanet上安裝AEM Dispatcher模組。
uuid: 2384b907-1042-4707-b02f-fba2125618cf
contentOwner: User
converted: true
topic-tags: dispatcher
content-type: reference
discoiquuid: f00ad751-6b95-4365-8500-e1e0108d9536
exl-id: 9375d1c0-8d9e-46cb-9810-fa4162a8c1ba
source-git-commit: 3a0e237278079a3885e527d7f86989f8ac91e09d
workflow-type: tm+mt
source-wordcount: '3684'
ht-degree: 1%

---

# 安裝 Dispatcher {#installing-dispatcher}

<!-- 

Comment Type: draft

<h2>Introduction</h2>

 -->

使用[Dispatcher發行說明](release-notes.md)頁面，取得作業系統和Web伺服器的最新Dispatcher安裝檔案。 Dispatcher版本編號與Adobe Experience Manager版本編號無關，並與Adobe Experience Manager 6.x、5.x和Adobe CQ 5.x版本相容。

>[!NOTE]
>
>請注意，Adobe Experience Manager 6.5需使用Dispatcher 4.3.2版或更新版本。 也就是說，Dispatcher版本與AEM無關，例如Dispatcher 4.3.2版也與Adobe Experience Manager 6.4相容。

使用下列檔案命名慣例：

`dispatcher-<web-server>-<operating-system>-<dispatcher-version-number>.<file-format>`

例如，`dispatcher-apache2.4-linux-x86_64-ssl-4.3.1.tar.gz`檔案包含適用於在Linux i686上執行且使用&#x200B;**tar**&#x200B;格式封裝的Apache 2.4 Web伺服器的Dispatcher 4.3.1版。

下表列出了用於每個Web伺服器的檔案名的Web伺服器標識符：

| Web伺服器 | 安裝套件 |
|--- |--- |
| Apache 2.4 | dispatcher-apache **2.4**-&lt;其他參數> |
| Microsoft Internet Information Server 7.5、8、8.5 | dispatcher-**iis**-&lt;其他參數> |
| Sun Java Web Server iPlanet | dispatcher-**ns**-&lt;其他參數> |

>[!CAUTION]
>
>您應安裝適用於您平台的最新版Dispatcher。 您應每年升級Dispatcher例項，以使用最新版本，以利用產品改良功能。

每個封存都包含下列檔案：

* Dispatcher模組
* 組態檔範例
* 包含安裝說明和最後時刻資訊的自述檔案
* 列出目前和過去版本中修正問題的CHANGES檔案

>[!NOTE]
>
>開始安裝前，請查看自述檔案，了解最後時刻的任何更改/平台特定說明。

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

## Microsoft Internet資訊伺服器{#microsoft-internet-information-server}

有關如何安裝此Web伺服器的資訊，請參閱以下資源：

* Microsoft在Internet Information Server上自己的文檔
* [&quot;官方的Microsoft IIS站點&quot;](https://www.iis.net/)

### 必需的IIS元件{#required-iis-components}

IIS 8.5和10版要求安裝以下IIS元件：

* ISAPI擴充功能

同時，您必須添加Web伺服器(IIS)角色。 使用「伺服器管理器」添加角色和元件。

## Microsoft IIS — 安裝Dispatcher模組{#microsoft-iis-installing-the-dispatcher-module}

Microsoft Internet Information System所需的歸檔檔案是：

* `dispatcher-iis-<operating-system>-<dispatcher-release-number>.zip`

ZIP檔案包含下列檔案：

| 檔案 | 說明 |
|--- |--- |
| `disp_iis.dll` | Dispatcher動態連結程式庫檔案。 |
| `disp_iis.ini` | IIS的配置檔案。 此範例可依您的需求更新。 **注意**:ini檔案的名稱根必須與dll相同。 |
| `dispatcher.any` | Dispatcher的範例設定檔案。 |
| `author_dispatcher.any` | Dispatcher搭配製作例項使用的範例設定檔案。 |
| 讀我檔案 | 包含安裝說明和最後時刻資訊的自述檔案。 **注意**:開始安裝之前，請檢查此檔案。 |
| 變更 | 變更列出目前和過去版本中已修正問題的檔案。 |

請依照下列步驟，將Dispatcher檔案複製到正確的位置。

1. 使用Windows資源管理器建立`<IIS_INSTALLDIR>/Scripts`目錄，例如`C:\inetpub\Scripts`。

1. 將下列檔案從Dispatcher套件解壓至此Scripts目錄：

   * `disp_iis.dll`
   * `disp_iis.ini`
   * 視Dispatcher是否使用AEM製作例項或發佈例項而定，下列其中一個檔案：
      * 製作例項：`author_dispatcher.any`
      * 發佈例項：`dispatcher.any`

## Microsoft IIS — 配置Dispatcher INI檔案{#microsoft-iis-configure-the-dispatcher-ini-file}

編輯`disp_iis.ini`檔案以設定Dispatcher安裝。 `.ini`檔案的基本格式如下：

```xml
[main]
configpath=<path to dispatcher.any>
loglevel=1|2|3
servervariables=0|1
replaceauthorization=0|1
```

下表說明了每個屬性。

| 參數 | 說明 |
|--- |--- |
| configpath | `dispatcher.any`在本地檔案系統內的位置（絕對路徑）。 |
| 記錄檔 | `dispatcher.log`檔案的位置。 如果未設定，則日誌消息將轉至windows事件日誌。 |
| loglevel | 定義用於將消息輸出到事件日誌的日誌級別。 可以指定以下值：日誌檔案的日誌級別：<br/>0 — 僅錯誤消息。 <br/>1 — 錯誤和警告。<br/>2 — 錯誤、警告和資訊性訊 <br/>息3 — 錯誤、警告、資訊性和除錯訊息。<br/>**注意**:建議在安裝和測試期間將日誌級別設定為3，然後在生產環境中運行時將日誌級別設定為0。 |
| replaceauthorization | 指定如何處理HTTP請求中的授權標題。 以下值有效：<br/>0 — 未修改授權標題。 <br/>1 — 以「Basic」以外的任何標題取代「Authorization」，代之 `Basic <IIS:LOGON\_USER>` 相等。<br/> |
| 伺服器變數 | 定義伺服器變數的處理方式。<br/>0 - IIS伺服器變數不會傳送至Dispatcher或AEM。<br/>1 — 所有IIS伺服器變數(例如 `LOGON\_USER, QUERY\_STRING, ...`)會連同請求標頭(若未快取，也會傳送至AEM例項)一併傳送至Dispatcher。<br/>伺服器變數包 `AUTH\_USER, LOGON\_USER, HTTPS\_KEYSIZE` 含及其他許多變數。請參閱IIS檔案，以取得變數的完整清單及詳細資訊。 |
| enable_chunked_transfer | 定義是啟用(1)還是停用(0)客戶回應的切分傳輸。 預設值為 0。 |

範例設定：

```xml
[main]
configpath=C:\Inetpub\Scripts\dispatcher.any
loglevel=1
servervariables=1
replaceauthorization=0
```

### 配置Microsoft IIS {#configuring-microsoft-iis}

配置IIS以整合Dispatcher ISAPI模組。 在IIS中，使用通配符應用程式映射。

### 配置匿名訪問 — IIS 8.5和10 {#configuring-anonymous-access-iis-and}

已設定Author例項上的預設排清復寫代理，使其不會傳送具有排清請求的安全憑證。 因此，您使用Dispatcher快取的網站必須允許匿名存取。

如果您的網站使用驗證方法，則必須據以設定排清復寫代理。

1. 開啟「IIS管理器」，並選擇您使用的網站作為Disptcher快取。
1. 在「使用功能視圖」模式中，在IIS部分按兩下「身份驗證」。
1. 如果未啟用匿名身份驗證，請選擇匿名身份驗證，然後在「操作」區域中按一下啟用。

### 整合Dispatcher ISAPI模組 — IIS 8.5和10 {#integrating-the-dispatcher-isapi-module-iis-and}

請依照下列程式，將Dispatcher ISAPI模組新增至IIS。

1. 開啟IIS管理器。
1. 選取您用作Dispatcher快取的網站。
1. 在「IIS」部分中，使用「功能視圖」模式，按兩下「處理程式映射」。
1. 在「處理程式映射」頁的「操作」面板中，按一下「添加通配符指令碼映射」，添加以下屬性值，然後按一下「確定」：

   * 請求路徑：*
   * 執行檔：disp_iis.dll檔案的絕對路徑，例如`C:\inetpub\Scripts\disp_iis.dll`。
   * 名稱：處理程式映射的描述性名稱，例如`Dispatcher`。

1. 在出現的對話框中，要將disp_iis.dll庫添加到ISAPI和CGI限制清單中，請按一下「是」。

   對於IIS 7.0和7.5 ，配置已完成。 如果要配置IIS 8.0，請繼續執行其餘步驟。

1. (IIS 8.0)在處理程式映射清單中，選擇剛建立的映射，然後在「操作」區域中按一下「編輯」。
1. (IIS 8.0)在「編輯指令碼映射」對話方塊中，按一下「請求限制」按鈕。
1. (IIS 8.0)要確保處理程式用於尚未快取的檔案和資料夾，請取消選擇「僅在將請求映射到時調用處理程式」，然後按一下「確定」。
1. (IIS 8.0)在「編輯指令碼映射」對話框上，按一下「確定」。

### 配置對快取的訪問 — IIS 8.5和10 {#configuring-access-to-the-cache-iis-and}

為預設的「應用程式池」使用者提供寫入存取權，存取正用作Dispatcher快取的資料夾。

1. 以滑鼠右鍵按一下您用作Dispatcher快取之網站的根資料夾，然後按一下「屬性」，例如`C:\inetpub\wwwroot`。
1. 在「安全性」頁簽上，按一下「編輯」，然後在「權限」對話框上，按一下「添加」。 將開啟一個對話框，用於選擇用戶帳戶。 按一下「位置」按鈕，選擇電腦名稱，然後按一下「確定」。

   完成下一步時，請保持此對話框開啟。

1. 在「IIS管理器」中，選擇您用於Dispatcher快取的IIS站點，然後在窗口的右側按一下「高級設定」。
1. 選擇應用程式池屬性的值，並將其複製到剪貼簿。
1. 返回開啟的對話框。 在輸入要選擇的對象名稱框中，鍵入`IIS AppPool\`，然後貼上剪貼簿的內容。 值應如下列範例所示：

   `IIS AppPool\DefaultAppPool`

1. 按一下「檢查名稱」按鈕。 Windows解析用戶帳戶時，按一下「確定」。
1. 在Dispatcher資料夾的「權限」對話方塊中，選取您剛新增的帳戶，啟用帳戶&#x200B;**的所有權限，但完全控制**&#x200B;除外，然後按一下「確定」。 按一下「確定」(OK)以關閉資料夾「屬性」(Properties)對話框。

### 註冊JSON Mime類型 — IIS 8.5和10 {#registering-the-json-mime-type-iis-and}

當您希望Dispatcher允許JSON呼叫時，請依照下列程式來註冊JSON MIME類型。

1. 在IIS管理器中，選擇您的網站，並使用功能視圖，按兩下Mime類型。
1. 如果清單中沒有JSON擴充功能，請在「動作」面板中按一下「新增」，輸入下列屬性值，然後按一下「確定」：

   * 副檔名：`.json`
   * MIME類型：`application/json`

### 刪除bin隱藏段 — IIS 8.5和10 {#removing-the-bin-hidden-segment-iis-and}

請依照下列步驟移除`bin`隱藏的區段。 非新網站可包含此隱藏區段。

1. 在「IIS管理器」中，選擇您的網站，並使用「功能視圖」，按兩下「請求篩選」。
1. 選擇`bin`段，按一下「刪除」，然後在確認對話框中按一下「是」。

### 將IIS消息記錄到檔案 — IIS 8.5和10 {#logging-iis-messages-to-a-file-iis-and}

請依照下列程式將Dispatcher記錄檔訊息寫入記錄檔，而非Windows事件記錄檔。 您需要將Dispatcher設定為使用記錄檔，並為IIS提供檔案的寫入存取權。

1. 使用Windows資源管理器在IIS安裝的日誌資料夾下建立名為`dispatcher`的資料夾。 典型安裝的此資料夾路徑為`C:\inetpub\logs\dispatcher`。

1. 以滑鼠右鍵按一下Dispatcher資料夾，然後按一下「屬性」。
1. 在「安全性」頁簽上，按一下「編輯」，然後在「權限」對話框上，按一下「添加」。 將開啟一個對話框，用於選擇用戶帳戶。 按一下「位置」按鈕，選擇電腦名稱，然後按一下「確定」。

   完成下一步時，請保持此對話框開啟。

1. 在「IIS管理器」中，選擇您用於Dispatcher快取的IIS站點，然後在窗口的右側按一下「高級設定」。
1. 選擇應用程式池屬性的值，並將其複製到剪貼簿。
1. 返回開啟的對話框。 在輸入要選擇的對象名稱框中，鍵入`IIS AppPool\`，然後貼上剪貼簿的內容。 值應如下列範例所示：

   `IIS AppPool\DefaultAppPool`

1. 按一下「檢查名稱」按鈕。 Windows解析用戶帳戶時，按一下「確定」。
1. 在Dispatcher資料夾的「權限」對話方塊中，選取您剛新增的帳戶，啟用帳戶&#x200B;**的所有權限，但「完整控制」除外，**&#x200B;然後按一下「確定」。 按一下「確定」(OK)以關閉資料夾「屬性」(Properties)對話框。
1. 使用文本編輯器開啟`disp_iis.ini`檔案。
1. 新增類似下列範例的文字行，以設定記錄檔的位置，然後儲存檔案：

   ```xml
   logfile=C:\inetpub\logs\dispatcher\dispatcher.log
   ```

### 後續步驟{#next-steps}

開始使用Dispatcher之前，您必須知道：

* [](dispatcher-configuration.md) ConfigureDispatcher
* [設定](page-invalidate.md) AEM以與Dispatcher搭配使用。

## Apache Web Server {#apache-web-server}

>[!CAUTION]
>
>此處介紹了&#x200B;**Windows**&#x200B;和&#x200B;**Unix**&#x200B;下的安裝說明。 執行步驟時請小心。

### 安裝Apache Web Server {#installing-apache-web-server}

有關如何安裝Apache Web Server的資訊，請參閱安裝手冊 — [online](https://httpd.apache.org/)或分發中。

>[!CAUTION]
>
>如果要通過編譯源檔案來建立Apache二進位檔案，請確保開啟&#x200B;**動態模組支援**。 您可以使用&#x200B;**—enable-shared**&#x200B;選項中的任一選項來完成此操作。 至少包括`mod_so`模組。
>
>有關詳細資訊，請參閱《Apache Web Server安裝手冊》。

另請參閱Apache HTTP Server [ Security Tips](https://httpd.apache.org/docs/2.4/misc/security_tips.html)和[ Security Reports](https://httpd.apache.org/security_report.html)。

### Apache Web伺服器 — 新增Dispatcher模組{#apache-web-server-add-the-dispatcher-module}

Dispatcher會以下列任一方式提供：

* **Windows**:動態連結庫(DLL)
* **Unix**:動態共用物件(DSO)

安裝歸檔檔案包含下列檔案（取決於您是選擇了Windows還是Unix）:

| 檔案 | 說明 |
|--- |--- |
| disp_apache&lt;x.y>.dll | 窗口：Dispatcher動態連結程式庫檔案。 |
| dispatcher-apache&lt;x.y>-&lt;rel-nr>so | Unix:Dispatcher共用物件程式庫檔案。 |
| mod_dispatcher.so | Unix:範例連結。 |
| http.conf.disp&lt;x> | Apache伺服器的示例配置檔案。 |
| dispatcher.any | Dispatcher的範例設定檔案。 |
| 讀我檔案 | 包含安裝說明和最後時刻資訊的自述檔案。 **注意**:開始安裝之前，請檢查此檔案。 |
| 變更 | 變更列出目前和過去版本中已修正問題的檔案。 |

使用下列步驟將Dispatcher新增至您的Apache Web Server:

1. 將Dispatcher檔案放置在適當的Apache模組目錄中：

   * **Windows**:位置  `disp_apache<x.y>.dll` `<APACHE_ROOT>/modules`
   * **Unix**:根據您 `<APACHE_ROOT>/libexec` 的安 `<APACHE_ROOT>/modules`裝找到或目錄。\
      將`dispatcher-apache<options>.so`複製到此目錄。\
      若要簡化長期維護，您也可以建立Dispatcher的符號連結`mod_dispatcher.so`:\
      `ln -s dispatcher-apache<x>-<os>-<rel-nr>.so mod_dispatcher.so`

1. 將dispatcher.any檔案複製到`<APACHE_ROOT>/conf`目錄。

   **注意：** 只要Dispatcher模組的DispatcherLog屬性已據以設定，您就可以將此檔案放置在不同位置。（請參閱下方的Dispatcher專用設定項目）。

### Apache Web伺服器 — 配置SELinux屬性{#apache-web-server-configure-selinux-properties}

如果您在已啟用SELinux的RedHat Linux Kernel 2.6上執行Dispatcher，在Dispatcher記錄檔中可能會遇到類似的錯誤訊息。

`Mon Jun 30 00:03:59 2013] [E] [16561(139642697451488)] Unable to connect to backend rend01 (10.122.213.248:4502): Permission denied`

這可能是由於啟用了SELinux安全性。 然後，您需要執行以下任務：

* 設定Dispatcher模組檔案的SELinux內容。
* 啟用HTTPD指令碼和模組以建立網路連接。
* 配置快取檔案的資料夾的SELinux上下文。

在終端窗口中輸入以下命令，將`[path to the dispatcher.so file]`替換為安裝到Apache Web Server的Dispatcher模組的路徑，並將&#x200B;*`path to the docroot`*&#x200B;替換為docroot所在的路徑(例如`/opt/cq/cache`):

```shell
semanage fcontext -a -t httpd_modules_t [path to the dispatcher.so file]
setsebool -P httpd_can_network_connect on
chcon -R --type httpd_sys_content_t [path to the docroot]
semanage fcontext -a -t httpd_sys_content_t "[path to the docroot](/.*)?"
```

### Apache Web伺服器 — 為Dispatcher {#apache-web-server-configure-apache-web-server-for-dispatcher}配置Apache Web伺服器

需要使用`httpd.conf`配置Apache Web伺服器。 在Dispatcher安裝套件中，您會找到名為`httpd.conf.disp<x>`的範例設定檔案。

這些步驟是強制性的：

1. 導航到 `<APACHE_ROOT>/conf`.
1. 開啟`httpd.conf`進行編輯。
1. 必須依所列順序新增下列設定項目：

   * **** LoadModule以在啟動時載入模組。
   * Dispatcher專用的設定項目，包括&#x200B;**DispatcherConfig、DispatcherLog**&#x200B;和&#x200B;**DispatcherLogLevel**。
   * **** SetHandlerto啟動Dispatcher。**LoadModule**。
   * **** ModMimeUsePathInfo以配置mod_mime **的行為**。

1. （選用）建議您變更htdocs目錄的擁有者：

   * Apache伺服器以root啟動，但子進程以守護程式啟動（出於安全目的）。 DocumentRoot(`<APACHE_ROOT>/htdocs`)必須屬於用戶守護程式：

      ```xml
      cd <APACHE_ROOT>  
      chown -R daemon:daemon htdocs
      ```

**LoadModule**

下表列出可使用的範例；確切的條目是根據您的特定Apache Web Server:

|  |  |
|--- |--- |
| Windows | `... LoadModule dispatcher_module modules\disp_apache.dll ...` |
| Unix（假設為符號連結） | `... LoadModule dispatcher_module libexec/mod_dispatcher.so ...` |

>[!NOTE]
>
>每個陳述式的第一個參數必須與上述範例完全相同。
>
>有關此命令的完整詳細資訊，請參閱提供的示例配置檔案和Apache Web Server文檔。

**Dispatcher特定設定項目**

Dispatcher專用的設定項目會放在LoadModule項目之後。 下表列出了適用於Unix和Windows的示例配置：

**Windows和Unix**

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

個別設定參數：

| 參數 | 說明 |
|--- |--- |
| DispatcherConfig | Dispatcher設定檔案的位置和名稱。 <br/>當此屬性位於主伺服器配置中時，所有虛擬主機都將繼承該屬性值。不過，虛擬主機可包含DispatcherConfig屬性，以覆寫主要伺服器設定。 |
| DispatcherLog | 日誌檔案的位置和名稱。 |
| DispatcherLogLevel | 記錄檔的記錄層級：<br/>0 — 錯誤<br/>1 — 警告<br/>2 — 資訊<br/>3 — 調試&#x200B;<br/>**注意**:建議在安裝和測試期間將日誌級別設定為3，然後在生產環境中運行時將日誌級別設定為0。 |
| DispatcherNoServerHeader | *此參數已遭取代，不再有任何作用。*<br/><br/> 定義要使用的伺服器標題：  <br/><ul><li>未定義或0 - HTTP伺服器標題包含AEM版本。 </li><li>1 — 使用Apache伺服器標題。</li></ul> |
| DispatcherDescineRoot | 定義是否拒絕對根「/」的請求：<br/>**0** — 接受對/ <br/>**1**&#x200B;的要求 — 對/的要求未由Dispatcher處理；使用mod_alias進行正確的映射。 |
| DispatcherUseProcessedURL | 定義是否要將預先處理的URL用於Dispatcher的所有進一步處理：<br/>**0** — 使用傳遞至Web伺服器的原始URL。 <br/>**1**  - Dispatcher會使用先於Dispatcher的處理常式已處理的URL(即 `mod_rewrite`)，而非傳遞至Web伺服器的原始URL。例如，原始URL或已處理的URL都與Dispatcher篩選器相符。 URL也是快取檔案結構的基礎。   如需mod_rewrite的相關資訊，請參閱Apache網站檔案；例如，Apache 2.4。使用mod_rewrite時，建議使用標幟「passthrough」 | PT&#39;（傳遞至下一個處理常式）強制重寫引擎將內部request_rec結構的uri欄位設定為檔案名欄位的值。 |
| DispatcherPassError | 定義如何支援ErrorDocument處理的錯誤代碼：<br/>**0** - Dispatcher會對用戶端進行所有錯誤回應。 <br/>**1**  - Dispatcher不會將錯誤回應假設為用戶端（其中狀態代碼大於或等於400），而是將狀態代碼傳遞至Apache，例如允許ErrorDocument指令處理此類狀態代碼。<br/>**代碼範圍**  — 指定將回應傳遞至Apache的錯誤代碼範圍。其他錯誤代碼將傳遞給客戶端。 例如，下列設定會將錯誤412的回應傳遞至用戶端，而所有其他錯誤則會傳遞至Apache:DispatcherPassError 400-411,413-417 |
| DispatcherKeepAliveTimeout | 指定保存超時（以秒為單位）。 從Dispatcher 4.2.0版開始，預設的「保存」值為60。 值0將禁用保持活動。 |
| DispatcherNoCanonURL | 將此參數設為「開啟」會將原始URL傳遞至後端，而非標準化的URL，並會覆寫DispatcherUseProcessedURL的設定。 預設值為Off。 <br/>**注意**:Dispatcher設定中的篩選規則一律會根據處理過的URL（而非原始URL）來評估。 |




>[!NOTE]
>
>路徑條目相對於Apache Web Server的根目錄。

>[!NOTE]
>
>伺服器標題的預設設定為：`  
ServerTokens Full` `  
DispatcherNoServerHeader 0`\
這會顯示AEM版本（用於統計用途）。 如果要停用標題中可用的此類資訊，可以設定：`  
ServerTokens Prod`\
如需詳細資訊，請參閱[關於ServerToken指令的Apache檔案（例如，適用於Apache 2.4）](https://httpd.apache.org/docs/2.4/mod/core.html) 。

**SetHandler**

在這些項目之後，您必須將&#x200B;**SetHandler**&#x200B;陳述式新增至設定的內容(`<Directory>`、`<Location>`),Dispatcher才能處理傳入的請求。 下列範例會將Dispatcher設定為處理完整網站的請求：

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

下列範例會將Dispatcher設定為處理虛擬網域的請求：

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
**SetHandler**&#x200B;語句的參數必須寫入&#x200B;*，與上述示例*&#x200B;中的參數完全相同，因為這是模組中定義的處理程式的名稱。
有關此命令的完整詳細資訊，請參閱提供的示例配置檔案和Apache Web Server文檔。

**ModMimeUsePathInfo**

在&#x200B;**SetHandler**&#x200B;語句後，您還應添加&#x200B;**ModMimeUsePathInfo**&#x200B;定義。

>[!NOTE]
`ModMimeUsePathInfo`參數僅應在您使用Dispatcher 4.0.9版或更新版本時使用並設定。
(請注意，Dispatcher 4.0.9版已於2011年發行。 如果您使用舊版，則升級至最新的Dispatcher版本是適當的)。

應為所有Apache配置設定`On`**ModMimeUsePathInfo**&#x200B;參數：

`ModMimeUsePathInfo On`

mod_mime模組（例如，[Apache模組mod_mime](https://httpd.apache.org/docs/2.4/mod/mod_mime.html)）用於將內容元資料分配給為HTTP響應選擇的內容。 預設設定表示當mod_mime決定內容類型時，將只考慮映射至檔案或目錄的URL的一部分。

當`On`時，`ModMimeUsePathInfo`參數指定`mod_mime`是根據&#x200B;*complete* URL來判斷內容類型；這表示虛擬資源將會根據其擴充功能套用元資訊。

以下示例激活&#x200B;**ModMimeUsePathInfo**:

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

### 啟用對HTTPS（Unix和Linux）的支援{#enable-support-for-https-unix-and-linux}

Dispatcher使用OpenSSL來實作透過HTTP的安全通訊。 從Dispatcher版本&#x200B;**4.2.0**&#x200B;開始，支援OpenSSL 1.0.0和OpenSSL 1.0.1。 Dispatcher預設會使用OpenSSL 1.0.0。 若要使用OpenSSL 1.0.1，請使用下列程式建立符號連結，讓Dispatcher使用已安裝的OpenSSL程式庫。

1. 開啟終端機，並將目前目錄變更為安裝OpenSSL程式庫的目錄，例如：

   ```shell
   cd /usr/lib64
   ```

1. 要建立符號連結，請輸入以下命令：

   ```shell
   ln -s libssl.so libssl.so.1.0.1
   ln -s libcrypto.so libcrypto.so.1.0.1
   ```

>[!NOTE]
如果您使用的是自訂版本的Apache，請確定Apache和Dispatcher是使用相同版本的[OpenSSL](https://www.openssl.org/source/)編譯。

### 後續步驟{#next-steps-1}

開始使用Dispatcher之前，您現在必須：

* [](dispatcher-configuration.md) ConfigureDispatcher
* [設定](page-invalidate.md) AEM以與Dispatcher搭配使用。

## Sun Java System Web Server / iPlanet {#sun-java-system-web-server-iplanet}

>[!NOTE]
此處介紹了Windows和Unix環境的說明。
選取要執行的時候請小心。

### Sun Java System Web Server / iPlanet — 安裝Web伺服器{#sun-java-system-web-server-iplanet-installing-your-web-server}

有關如何安裝這些Web伺服器的完整資訊，請參閱其各自的文檔：

* Sun Java System Web Server
* iPlanet Web Server

### Sun Java System Web Server / iPlanet — 添加Dispatcher模組{#sun-java-system-web-server-iplanet-add-the-dispatcher-module}

Dispatcher會以下列任一方式提供：

* **Windows**:動態連結庫(DLL)
* **Unix**:動態共用物件(DSO)

安裝歸檔檔案包含下列檔案（取決於您是選擇了Windows還是Unix）:

| 檔案 | 說明 |
|---|---|
| `disp_ns.dll` | 窗口：Dispatcher動態連結程式庫檔案。 |
| `dispatcher.so` | Unix:Dispatcher共用物件程式庫檔案。 |
| `dispatcher.so` | Unix:範例連結。 |
| `obj.conf.disp` | iPlanet / Sun Java System Web伺服器的示例配置檔案。 |
| `dispatcher.any` | Dispatcher的範例設定檔案。 |
| 讀我檔案 | 包含安裝說明和最後時刻資訊的自述檔案。 注意：開始安裝之前，請檢查此檔案。 |
| 變更 | 變更列出目前和過去版本中已修正問題的檔案。 |

使用下列步驟將Dispatcher新增至您的Web伺服器：

1. 將Dispatcher檔案放置在Web伺服器的`plugin`目錄中：

### Sun Java System Web Server / iPlanet — 為Dispatcher {#sun-java-system-web-server-iplanet-configure-for-the-dispatcher}配置

需要使用`obj.conf`配置Web伺服器。 在Dispatcher安裝套件中，您會找到名為`obj.conf.disp`的範例設定檔案。

1. 導航到 `<WEBSERVER_ROOT>/config`.
1. 開啟`obj.conf`進行編輯。
1. 複製開始的行：\
   `Service fn="dispService"`\
   從`obj.conf.disp`到`obj.conf`的初始化部分。

1. 儲存變更。
1. 開啟`magnus.conf`進行編輯。
1. 複製開始的兩行：\
   `Init funcs="dispService, dispInit"`\
   和\
   `Init fn="dispInit"`\
   從`obj.conf.disp`到`magnus.conf`的初始化部分。

1. 儲存變更。

>[!NOTE]
下列設定應全部位於一行，且`$(SERVER_ROOT)`和`$(PRODUCT_SUBDIR)`必須取代為個別值。

**初始化**

下表列出可使用的範例；確切的輸入值會根據您的特定網站伺服器：

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
| 設定 | 配置檔案`dispatcher.any.`的位置和名稱 |
| 記錄檔 | 日誌檔案的位置和名稱。 |
| loglevel | 將消息寫入日誌檔案時的日誌級別：<br/>**0**&#x200B;錯誤&#x200B;<br/>**1**&#x200B;警告&#x200B;<br/>**2**&#x200B;資訊&#x200B;<br/>**3**&#x200B;除錯&#x200B;<br/>**注意：**&#x200B;建議在安裝和測試期間將日誌級別設定為3，在生產環境中運行時將日誌級別設定為0。 |
| keepalitimeout | 指定保存超時（以秒為單位）。 從Dispatcher 4.2.0版開始，預設的「保存」值為60。 值0將禁用保持活動。 |

您可以根據您的需求，將Dispatcher定義為物件的服務。 若要為整個網站設定Dispatcher，請修改預設物件：


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

### 後續步驟{#next-steps-2}

開始使用Dispatcher之前，您現在必須：

* [](dispatcher-configuration.md) ConfigureDispatcher
* [設定](page-invalidate.md) AEM以與Dispatcher搭配使用。
