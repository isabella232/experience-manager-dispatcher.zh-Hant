---
title: 安裝 Dispatcher
seo-title: 安裝AEM Dispatcher
description: 瞭解如何在Microsoft Internet Information Server、Apache Web Server和Sun Java Web Server-iPlanet上安裝Dispatcher模組。
seo-description: 瞭解如何在Microsoft Internet Information Server、Apache Web Server和Sun Java Web Server-iPlanet上安裝AEM Dispatcher模組。
uuid: 2384b907-1042-4707-b02f-fba2125618cf
contentOwner: User
converted: true
topic-tags: dispatcher
content-type: reference
discoiquuid: f00ad751-6b95-4365-8500-e1e0108d9536
translation-type: tm+mt
source-git-commit: 024348672c2a9a4f8a01429572eba27ea8b8a490
workflow-type: tm+mt
source-wordcount: '3684'
ht-degree: 0%

---


# 安裝 Dispatcher {#installing-dispatcher}

<!-- 

Comment Type: draft

<h2>Introduction</h2>

 -->

使用「[Dispatcher發行說明」](release-notes.md)頁可以獲取您的作業系統和Web伺服器的最新Dispatcher安裝檔案。 Dispatcher發行號碼與Adobe Experience Manager發行號碼無關，並與Adobe Experience Manager 6.x、5.x和Adobe CQ 5.x版本相容。

>[!NOTE]
>
>請注意，Adobe Experience Manager 6.5需要Dispatcher 4.3.2版或更高版本。 儘管如此，Dispatcher版本與AEM無關，例如Dispatcher 4.3.2版也與Adobe Experience Manager 6.4相容。

使用下列檔案命名慣例：

`dispatcher-<web-server>-<operating-system>-<dispatcher-version-number>.<file-format>`

例如，`dispatcher-apache2.4-linux-x86_64-ssl-4.3.1.tar.gz`檔案包含Dispatcher 4.3.1版，適用於在Linux i686上執行的Apache 2.4 Web伺服器，並使用&#x200B;**tar**&#x200B;格式封裝。

下表列出了用於每個Web伺服器的檔案名的Web伺服器標識符：

| Web伺服器 | 安裝套件 |
|--- |--- |
| Apache 2.4 | dispatcher-apache **2.4**-&lt;other parameters> |
| Microsoft Internet Information Server 7.5、8、8.5 | dispatcher-**iis**-&lt;other parameters) |
| Sun Java Web Server iPlanet | dispatcher-**ns**-&lt;other parameters> |

>[!CAUTION]
>
>您應安裝適用於您平台的最新版Dispatcher。 您應每年升級您的Dispatcher實例，以使用最新版本，以利用產品改進。

每個封存檔都包含下列檔案：

* dispatcher模組
* 示例配置檔案
* 自述檔案，包含安裝說明和最後一分鐘資訊
* 列出當前版本和過去版本中已修復問題的CHANGES檔案

>[!NOTE]
>
>在開始安裝之前，請查看自述檔案，以瞭解任何最後時刻的更改／平台特定注意事項。

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

有關如何安裝此Web伺服器的資訊，請參見以下資源：

* Internet Information Server上的Microsoft自己的文檔
* [&quot;The Official Microsoft IIS site&quot;](https://www.iis.net/)

### 所需的IIS元件{#required-iis-components}

IIS 8.5和10版需要安裝下列IIS元件：

* ISAPI擴充功能

此外，您還必須添加Web伺服器(IIS)角色。 使用「伺服器管理器」添加角色和元件。

## Microsoft IIS —— 安裝Dispatcher模組{#microsoft-iis-installing-the-dispatcher-module}

Microsoft Internet Information System所需的歸檔檔案為：

* `dispatcher-iis-<operating-system>-<dispatcher-release-number>.zip`

ZIP檔案包含下列檔案：

| 檔案 | 說明 |
|--- |--- |
| `disp_iis.dll` | Dispatcher動態連結庫檔案。 |
| `disp_iis.ini` | IIS的配置檔案。 此範例可隨您的需求更新。 **注意**:ini檔案必須與dll具有相同的name-root。 |
| `dispatcher.any` | Dispatcher的示例配置檔案。 |
| `author_dispatcher.any` | Dispatcher使用作者實例的示例配置檔案。 |
| 自述檔案 | 自述檔案，包含安裝說明和最後一分鐘資訊。 **注意**:請先檢查此檔案，然後再開始安裝。 |
| 變更 | 變更列出目前和舊版中已修正問題的檔案。 |

請按下列步驟將Dispatcher檔案複製到正確的位置。

1. 使用Windows資源管理器建立`<IIS_INSTALLDIR>/Scripts`目錄，例如`C:\inetpub\Scripts`。

1. 將下列檔案從Dispatcher包解壓到此Scripts目錄：

   * `disp_iis.dll`
   * `disp_iis.ini`
   * 根據Dispatcher是使用AEM作者例項或發佈例項，下列其中一個檔案：
      * 作者實例：`author_dispatcher.any`
      * 發佈例項：`dispatcher.any`

## Microsoft IIS —— 配置Dispatcher INI檔案{#microsoft-iis-configure-the-dispatcher-ini-file}

編輯`disp_iis.ini`檔案以配置Dispatcher安裝。 `.ini`檔案的基本格式如下：

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
| configpath | `dispatcher.any`在本地檔案系統中的位置（絕對路徑）。 |
| 日誌檔案 | `dispatcher.log`檔案的位置。 如果未設定，則日誌消息將轉至Windows事件日誌。 |
| loglevel | 定義用於將消息輸出到事件日誌的日誌級別。 可以指定以下值：日誌檔案的日誌級別：<br/>0 —— 僅錯誤消息。 <br/>1 —— 錯誤和警告。<br/>2 —— 錯誤、警告和資訊性消 <br/>息3 —— 錯誤、警告、資訊性和調試消息。<br/>**注意**:建議在安裝和測試期間將日誌級別設定為3，然後在生產環境中運行時將日誌級別設定為0。 |
| 替換授權 | 指定HTTP請求中的授權標題的處理方式。 以下值有效：<br/>0 —— 不修改授權標題。 <br/>1 —— 將名為&quot;Authorization&quot;（非&quot;Basic&quot;）的標題替換為其相 `Basic <IIS:LOGON\_USER>` 同值。<br/> |
| servervariables | 定義處理伺服器變數的方式。<br/>0 - IIS伺服器變數不會傳送至Dispatcher或AEM。<br/>1 —— 所有IIS伺服器變數(例如 `LOGON\_USER, QUERY\_STRING, ...`)都會與請求標題一起傳送至Dispatcher（若未快取，也會傳送至AEM例項）。<br/>伺服器變數包 `AUTH\_USER, LOGON\_USER, HTTPS\_KEYSIZE` 括許多其他變數。請參閱IIS檔案，以取得完整的變數清單，並提供詳細資訊。 |
| enable_chunked_transfer | 定義是啟用(1)還是停用(0)客戶端回應的區塊化傳輸。 預設值為0。 |

配置示例：

```xml
[main]
configpath=C:\Inetpub\Scripts\dispatcher.any
loglevel=1
servervariables=1
replaceauthorization=0
```

### 配置Microsoft IIS {#configuring-microsoft-iis}

配置IIS以整合Dispatcher ISAPI模組。 在IIS中，您使用萬用字元應用程式對應。

### 配置匿名訪問- IIS 8.5和10 {#configuring-anonymous-access-iis-and}

Author實例上的預設Flush複製代理已配置，因此不會發送具有刷新請求的安全憑據。 因此，您使用Dispatcher快取的網站必須允許匿名存取。

如果您的網站使用驗證方法，則必須相應地配置刷新複製代理。

1. 開啟IIS管理器，並選取您要用作Disptcher快取的網站。
1. 使用功能視圖模式，在IIS部分中按兩下身份驗證。
1. 如果未啟用匿名驗證，請選擇匿名驗證，然後在「動作」區域中按一下啟用。

### 整合Dispatcher ISAPI模組- IIS 8.5和10 {#integrating-the-dispatcher-isapi-module-iis-and}

請按下列步驟將Dispatcher ISAPI模組添加到IIS。

1. 開啟IIS管理器。
1. 選擇您用作Dispatcher Cache的網站。
1. 使用功能視圖模式，在IIS部分中按兩下處理程式映射。
1. 在「處理程式映射」頁的「操作」面板中，按一下添加通配符指令碼映射，添加以下屬性值，然後按一下確定：

   * 請求路徑：*
   * 執行檔：disp_iis.dll檔案的絕對路徑，例如`C:\inetpub\Scripts\disp_iis.dll`。
   * 名稱：處理程式映射的描述性名稱，例如`Dispatcher`。

1. 在出現的對話方塊中，若要將disp_iis.dll程式庫新增至ISAPI和CGI限制清單，請按一下「是」。

   對於IIS 7.0和7.5 ，配置已完成。 如果要配置IIS 8.0，請繼續其餘步驟。

1. (IIS 8.0)在處理常式映射清單中，選取您剛建立的對應，然後在「動作」區域中按一下「編輯」。
1. (IIS 8.0)在「編輯指令碼對應」對話方塊中，按一下「請求限制」按鈕。
1. (IIS 8.0)為確保處理常式用於尚未快取的檔案和檔案夾，請取消選取「僅在請求映射至時叫用處理常式」，然後按一下「確定」。
1. (IIS 8.0)在「編輯指令碼映射」對話框中，按一下「確定」。

### 配置對快取的訪問- IIS 8.5和10 {#configuring-access-to-the-cache-iis-and}

為預設的App Pool用戶提供對用作Dispatcher快取的資料夾的寫訪問權限。

1. 按一下右鍵用作Dispatcher快取的網站的根資料夾，然後按一下屬性，如`C:\inetpub\wwwroot`。
1. 在「安全性」標籤上，按一下「編輯」，然後在「權限」對話方塊中按一下「新增」。 隨即開啟一個對話方塊，供您選取使用者帳戶。 按一下「Locations（位置）」按鈕，選擇您的電腦名稱，然後按一下「OK（確定）」。

   完成下一步驟時，請保持此對話框的開啟狀態。

1. 在「IIS管理器」中，選擇用於Dispatcher快取的IIS站點，然後在窗口的右側按一下「高級設定」。
1. 選擇「應用程式池」屬性的值，並將其複製到剪貼簿。
1. 返回開啟的對話框。 在「輸入要選擇的對象名稱」框中，鍵入`IIS AppPool\` ，然後貼上剪貼簿的內容。 該值應如下例所示：

   `IIS AppPool\DefaultAppPool`

1. 按一下「檢查名稱」按鈕。 當Windows解析使用者帳戶時，按一下「確定」。
1. 在調度程式資料夾的「權限」對話框中，選擇剛添加的帳戶，啟用帳戶&#x200B;**（完全控制**&#x200B;除外）的所有權限，然後按一下「確定」。 按一下「確定」(OK)關閉資料夾「屬性」(Properties)對話框。

### 註冊JSON Mime類型- IIS 8.5和10 {#registering-the-json-mime-type-iis-and}

當您想要讓Dispatcher允許JSON呼叫時，請使用下列程式來註冊JSON MIME類型。

1. 在「IIS管理器」中，選擇您的網站，然後使用「功能視圖」，按兩下「Mime類型」。
1. 如果JSON副檔名不在清單中，請在「動作」面板中按一下「新增」，輸入下列屬性值，然後按一下「確定」:

   * 副檔名：`.json`
   * MIME類型：`application/json`

### 刪除bin隱藏段- IIS 8.5和10 {#removing-the-bin-hidden-segment-iis-and}

請按下列步驟移除`bin`隱藏的區段。 非新網站可包含此隱藏區段。

1. 在「IIS管理器」中，選擇您的網站，然後使用功能檢視，按兩下請求篩選。
1. 選擇`bin`區段，按一下「移除」，然後在確認對話方塊中按一下「是」。

### 將IIS消息記錄到檔案- IIS 8.5和10 {#logging-iis-messages-to-a-file-iis-and}

使用以下過程將Dispatcher日誌消息寫入日誌檔案，而不寫入Windows事件日誌。 您需要配置Dispatcher以使用日誌檔案，並為IIS提供對該檔案的寫訪問權。

1. 使用Windows資源管理器在IIS安裝的logs資料夾下建立一個名為`dispatcher`的資料夾。 典型安裝的此資料夾路徑為`C:\inetpub\logs\dispatcher`。

1. 按一下右鍵調度程式資料夾，然後按一下「屬性」。
1. 在「安全性」標籤上，按一下「編輯」，然後在「權限」對話方塊中按一下「新增」。 隨即開啟一個對話方塊，供您選取使用者帳戶。 按一下「Locations（位置）」按鈕，選擇您的電腦名稱，然後按一下「OK（確定）」。

   完成下一步驟時，請保持此對話框的開啟狀態。

1. 在「IIS管理器」中，選擇您用於Dispatcher快取的IIS站點，然後在窗口的右側按一下「高級設定」。
1. 選擇「應用程式池」屬性的值，並將其複製到剪貼簿。
1. 返回開啟的對話框。 在「輸入要選擇的對象名稱」框中，鍵入`IIS AppPool\` ，然後貼上剪貼簿的內容。 該值應如下例所示：

   `IIS AppPool\DefaultAppPool`

1. 按一下「檢查名稱」按鈕。 當Windows解析使用者帳戶時，按一下「確定」。
1. 在調度程式資料夾的「權限」對話框中，選擇剛添加的帳戶，啟用帳戶&#x200B;**（完全控制除外）的所有權限，然後按一下「確定」。**&#x200B;按一下「確定」(OK)關閉資料夾「屬性」(Properties)對話框。
1. 使用文字編輯器開啟`disp_iis.ini`檔案。
1. 新增類似下列範例的文字行，以設定記錄檔的位置，然後儲存檔案：

   ```xml
   logfile=C:\inetpub\logs\dispatcher\dispatcher.log
   ```

### 後續步驟{#next-steps}

開始使用Dispatcher之前，您必須知道：

* [](dispatcher-configuration.md) ConfigureDispatcher
* [設定](page-invalidate.md) AEM以搭配Dispatcher運作。

## Apache Web Server {#apache-web-server}

>[!CAUTION]
>
>這裡介紹了在&#x200B;**Windows**&#x200B;和&#x200B;**Unix**&#x200B;下進行安裝的說明。 執行步驟時請務必小心。

### 安裝Apache Web Server {#installing-apache-web-server}

有關如何安裝Apache Web Server的資訊，請閱讀安裝手冊- [online](https://httpd.apache.org/)或在分發中。

>[!CAUTION]
>
>如果要通過編譯源檔案建立Apache二進位檔案，請確保開啟&#x200B;**動態模組support**。 這可以使用&#x200B;**- enable-shared**&#x200B;選項來完成。 至少包括`mod_so`模組。
>
>有關詳細資訊，請參閱Apache Web Server安裝手冊。

另請參閱Apache HTTP Server [安全提示](https://httpd.apache.org/docs/2.4/misc/security_tips.html)和[安全報告](https://httpd.apache.org/security_report.html)。

### Apache Web Server —— 添加Dispatcher模組{#apache-web-server-add-the-dispatcher-module}

Dispatcher的功能如下：

* **Windows**:動態連結庫(DLL)
* **Unix**:動態共用物件(DSO)

安裝歸檔檔案包含以下檔案——取決於您是否選擇了Windows或Unix:

| 檔案 | 說明 |
|--- |--- |
| disp_apache&lt;x.y>.dll | Windows:Dispatcher動態連結庫檔案。 |
| dispatcher-apache&lt;x.y>-&lt;rel-nr>.so | Unix:Dispatcher共用對象庫檔案。 |
| mod_dispatcher.so | Unix:範例連結。 |
| http.conf.disp&lt;x> | Apache伺服器的示例配置檔案。 |
| dispatcher.any | Dispatcher的示例配置檔案。 |
| 自述檔案 | 自述檔案，包含安裝說明和最後一分鐘資訊。 **注意**:請先檢查此檔案，然後再開始安裝。 |
| 變更 | 變更列出目前和過去版本中已修正問題的檔案。 |

使用以下步驟將Dispatcher添加到Apache Web Server:

1. 將Dispatcher檔案放在相應的Apache模組目錄中：

   * **Windows**:地點  `disp_apache<x.y>.dll` `<APACHE_ROOT>/modules`
   * **Unix**:根據您的 `<APACHE_ROOT>/libexec` 安裝 `<APACHE_ROOT>/modules`找到或目錄。\
      將`dispatcher-apache<options>.so`複製到此目錄。\
      為簡化長期維護，您還可以建立名為`mod_dispatcher.so`的符號連結到Dispatcher:\
      `ln -s dispatcher-apache<x>-<os>-<rel-nr>.so mod_dispatcher.so`

1. 將dispatcher.any檔案複製到`<APACHE_ROOT>/conf`目錄。

   **注意：只** 要Dispatcher模組的DispatcherLog屬性已相應配置，您就可以將此檔案放置在不同的位置。（請參見下面的Dispatcher-Specific Configuration Entries。）

### Apache Web Server —— 配置SELinux屬性{#apache-web-server-configure-selinux-properties}

如果在啟用SELinux的RedHat Linux Kernel 2.6上運行Dispatcher，在dispatcher日誌檔案中可能會遇到類似這樣的錯誤消息。

`Mon Jun 30 00:03:59 2013] [E] [16561(139642697451488)] Unable to connect to backend rend01 (10.122.213.248:4502): Permission denied`

這可能是由於啟用了SELinux安全性。 然後，您需要執行以下任務：

* 配置調度器模組檔案的SELinux上下文。
* 啟用HTTPD指令碼和模組以建立網路連線。
* 配置Docroot的SELinux上下文，其中儲存快取檔案。

在終端窗口中輸入以下命令，將`[path to the dispatcher.so file]`替換為您安裝到Apache Web Server的Dispatcher模組的路徑，將&#x200B;*`path to the docroot`*&#x200B;替換為Docroot所在的路徑(如`/opt/cq/cache`):

```shell
semanage fcontext -a -t httpd_modules_t [path to the dispatcher.so file]
setsebool -P httpd_can_network_connect on
chcon -R --type httpd_sys_content_t [path to the docroot]
semanage fcontext -a -t httpd_sys_content_t "[path to the docroot](/.*)?"
```

### Apache Web Server —— 為Dispatcher {#apache-web-server-configure-apache-web-server-for-dispatcher}配置Apache Web Server

需要使用`httpd.conf`配置Apache Web Server。 在Dispatcher安裝工具包中，您將找到名為`httpd.conf.disp<x>`的示例配置檔案。

這些步驟是強制性的：

1. 導航到 `<APACHE_ROOT>/conf`.
1. 開啟`httpd.conf`進行編輯。
1. 必須按所列順序添加以下配置條目：

   * **LoadModule** 可在啟動時載入模組。
   * Dispatcher特定的配置條目，包括&#x200B;**DispatcherConfig、DispatcherLog**&#x200B;和&#x200B;**DispatcherLogLevel**。
   * **SetHandler** 以啟動Dispatcher。**LoadModule**。
   * **** ModMimeUsePathInfo以配置 **mod_mime的行為**。

1. （可選）建議您變更htdocs目錄的擁有者：

   * apache伺服器以root身份啟動，但子進程以守護進程身份啟動（出於安全考慮）。 DocumentRoot(`<APACHE_ROOT>/htdocs`)必須屬於用戶守護程式：

      ```xml
      cd <APACHE_ROOT>  
      chown -R daemon:daemon htdocs
      ```

**LoadModule**

下表列出可使用的範例；確切的條目是根據您特定的Apache Web Server:

|  |  |
|--- |--- |
| Windows | `... LoadModule dispatcher_module modules\disp_apache.dll ...` |
| Unix（假定符號連結） | `... LoadModule dispatcher_module libexec/mod_dispatcher.so ...` |

>[!NOTE]
>
>每個語句的第一個參數必須與上述示例中完全相同。
>
>有關此命令的完整詳細資訊，請參見提供的示例配置檔案和Apache Web Server文檔。

**Dispatcher特定配置條目**

Dispatcher特定的配置條目放在LoadModule條目之後。 下表列出了適用於Unix和Windows的示例配置：

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

個別配置參數：

| 參數 | 說明 |
|--- |--- |
| DispatcherConfig | Dispatcher配置檔案的位置和名稱。 <br/>當此屬性位於主伺服器配置中時，所有虛擬主機都將繼承該屬性值。但是，虛擬主機可以包含DispatcherConfig屬性來覆蓋主伺服器配置。 |
| DispatcherLog | 日誌檔案的位置和名稱。 |
| DispatcherLogLevel | 日誌檔案的日誌級別：<br/>0 —— 錯誤<br/>1 —— 警告<br/>2 —— 資訊<br/>3 —— 調試&#x200B;<br/>**注意：建議在安裝和測試期間將日誌級別設定為3，然後在生產環境中運行時將日誌級別設定為0。** |
| DispatcherNoServerHeader | *此參數已過時，不再有任何作用。*<br/><br/> 定義要使用的伺服器標題：  <br/><ul><li>undefined或0 - HTTP伺服器標題包含AEM版本。 </li><li>1 —— 使用Apache伺服器標題。</li></ul> |
| DispatcherCliseRoot | 定義是否拒絕對根&quot;/&quot;的請求：<br/>**0** —— 接受對/ <br/>**1**&#x200B;的請求——發送方／未處理請求；使用mod_alias進行正確映射。 |
| DispatcherUseProcessedURL | 定義是否對Dispatcher的所有進一步處理使用預處理的URL:<br/>**0** —— 使用傳遞至Web伺服器的原始URL。 <br/>**1** -調度程式使用調度程式之前的處理程式已處理的URL(即 `mod_rewrite`)，而非傳遞至網頁伺服器的原始URL。例如，原始或處理過的URL與Dispatcher篩選器匹配。 此URL也用作快取檔案結構的基礎。   有關mod_rewrite；的資訊，請參閱Apache網站文檔；例如，Apache 2.4。使用mod_rewrite時，建議使用標幟&#39;passthrough | PT&#39;（傳遞至下一個處理常式），以強制重寫引擎將內部request_rec結構的uri欄位設為檔案名稱欄位的值。 |
| DispatcherPassError | 定義如何支援ErrorDocument處理的錯誤碼：<br/>**0** - Dispatcher會將所有錯誤響應轉發到客戶端。 <br/>**1** - Dispatcher不會對客戶端（其中狀態代碼大於或等於400）執行錯誤響應，而是將狀態代碼傳遞給Apache，例如允許ErrorDocument指令處理此類狀態代碼。<br/>**代碼範圍** -指定回應傳遞至Apache的錯誤代碼範圍。其他錯誤代碼會傳遞給用戶端。 例如，以下配置將錯誤412的響應傳遞給客戶機，而所有其它錯誤都傳遞給Apache:DispatcherPassError 400-411,413-417 |
| DispatcherKeepAliveTimeout | 指定保持活動超時（秒）。 從Dispatcher 4.2.0版開始，預設的keep-alive值為60。 值0會停用keep-alive。 |
| DispatcherNoCanonURL | 將此參數設為On會將原始URL傳遞至後端，而非標準化的URL，並會覆寫DispatcherUseProcessedURL的設定。 預設值為Off。 <br/>**注意**:Dispatcher配置中的篩選規則一律會根據淨化的URL（而非原始URL）進行評估。 |




>[!NOTE]
>
>路徑條目相對於Apache Web Server的根目錄。

>[!NOTE]
>
>伺服器標題的預設設定為：`  
ServerTokens Full` `  
DispatcherNoServerHeader 0`\
這會顯示AEM版本（用於統計用途）。 如果要禁用標題中提供的此類資訊，可以設定：`  
ServerTokens Prod`\
如需詳細資訊，請參閱[關於ServerToken指令的Apache檔案（例如，針對Apache 2.4）](https://httpd.apache.org/docs/2.4/mod/core.html)。

**SetHandler**

在這些條目之後，您必須將&#x200B;**SetHandler**&#x200B;語句添加到配置的上下文(`<Directory>`, `<Location>`)中，Dispatcher才能處理傳入的請求。 下面的示例將Dispatcher配置為處理整個網站的請求：

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

下面的示例將Dispatcher配置為處理虛擬域的請求：

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
**SetHandler**&#x200B;語句的參數必須寫入&#x200B;*，與上述示例中的*&#x200B;完全相同，因為這是模組中定義的處理程式的名稱。
有關此命令的完整詳細資訊，請參見提供的示例配置檔案和Apache Web Server文檔。

**ModMimeUsePathInfo**

在&#x200B;**SetHandler**&#x200B;語句後，您還應添加&#x200B;**ModMimeUsePathInfo**&#x200B;定義。

>[!NOTE]
`ModMimeUsePathInfo`參數僅應在您使用Dispatcher 4.0.9版或更高版本時使用和配置。
(請注意，Dispatcher 4.0.9版已於2011年發行。 如果您使用舊版，則升級到最新的Dispatcher版本是合適的)。

對於所有Apache配置，應設定&#x200B;**ModMimeUsePathInfo**&#x200B;參數：`On`

`ModMimeUsePathInfo On`

mod_mime模組（例如，[Apache模組mod_mime](https://httpd.apache.org/docs/2.4/mod/mod_mime.html)）用於將內容元資料分配給為HTTP響應選擇的內容。 預設設定表示，當mod_mime決定內容類型時，只會考慮映射至檔案或目錄的URL部分。

當`On`時，`ModMimeUsePathInfo`參數指定`mod_mime`是根據&#x200B;*complete* URL來決定內容類型；這意味著虛擬資源將根據其擴展應用元資訊。

下列範例會啟動&#x200B;**ModMimeUsePathInfo**:

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

### 啟用對HTTPS（Unix和Linux）{#enable-support-for-https-unix-and-linux}的支援

Dispatcher使用OpenSSL透過HTTP實作安全通訊。 從Dispatcher版本&#x200B;**4.2.0**&#x200B;開始，支援OpenSSL 1.0.0和OpenSSL 1.0.1。 預設情況下，Dispatcher使用OpenSSL 1.0.0。 要使用OpenSSL 1.0.1，請使用以下過程建立符號連結，以便Dispatcher使用已安裝的OpenSSL庫。

1. 開啟終端機並將目前目錄變更為安裝OpenSSL程式庫的目錄，例如：

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

您現在必須先開始使用Dispatcher:

* [](dispatcher-configuration.md) ConfigureDispatcher
* [設定](page-invalidate.md) AEM以搭配Dispatcher運作。

## Sun Java System Web Server / iPlanet {#sun-java-system-web-server-iplanet}

>[!NOTE]
這裡介紹了Windows和Unix環境的說明。
在選擇要執行的時候請務必小心。

### Sun Java System Web Server / iPlanet —— 安裝Web伺服器{#sun-java-system-web-server-iplanet-installing-your-web-server}

有關如何安裝這些Web伺服器的完整資訊，請參閱其各自的文檔：

* Sun Java System Web Server
* iPlanet Web Server

### Sun Java System Web Server / iPlanet —— 添加Dispatcher模組{#sun-java-system-web-server-iplanet-add-the-dispatcher-module}

Dispatcher的功能如下：

* **Windows**:動態連結庫(DLL)
* **Unix**:動態共用物件(DSO)

安裝歸檔檔案包含以下檔案——取決於您是否選擇了Windows或Unix:

| 檔案 | 說明 |
|---|---|
| `disp_ns.dll` | Windows:Dispatcher動態連結庫檔案。 |
| `dispatcher.so` | Unix:Dispatcher共用對象庫檔案。 |
| `dispatcher.so` | Unix:範例連結。 |
| `obj.conf.disp` | iPlanet / Sun Java System web server的示例配置檔案。 |
| `dispatcher.any` | Dispatcher的示例配置檔案。 |
| 自述檔案 | 自述檔案，包含安裝說明和最後一分鐘資訊。 注意：請先檢查此檔案，然後再開始安裝。 |
| 變更 | 變更列出目前和過去版本中已修正問題的檔案。 |

使用以下步驟將Dispatcher添加到Web伺服器：

1. 將Dispatcher檔案放在Web伺服器的`plugin`目錄中：

### Sun Java System Web Server / iPlanet —— 為Dispatcher {#sun-java-system-web-server-iplanet-configure-for-the-dispatcher}配置

需要使用`obj.conf`配置Web伺服器。 在Dispatcher安裝工具包中，您將找到名為`obj.conf.disp`的示例配置檔案。

1. 導航到 `<WEBSERVER_ROOT>/config`.
1. 開啟`obj.conf`進行編輯。
1. 複製以下行：\
   `Service fn="dispService"`\
   從`obj.conf.disp`到`obj.conf`的初始化部分。

1. 儲存變更。
1. 開啟`magnus.conf`進行編輯。
1. 複製以下兩行：\
   `Init funcs="dispService, dispInit"`\
   和\
   `Init fn="dispInit"`\
   從`obj.conf.disp`到`magnus.conf`的初始化部分。

1. 儲存變更。

>[!NOTE]
以下配置應全部在一行上，`$(SERVER_ROOT)`和`$(PRODUCT_SUBDIR)`必須由各個值替換。

**初始化**

下表列出可使用的範例；確切的條目是根據您的特定Web伺服器：

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
| config | 配置檔案`dispatcher.any.`的位置和名稱 |
| 日誌檔案 | 日誌檔案的位置和名稱。 |
| loglevel | 將消息寫入日誌檔案時的日誌級別：<br/>**0**&#x200B;錯誤&#x200B;<br/>**1**&#x200B;警告&#x200B;<br/>**2**&#x200B;資訊&#x200B;<br/>**3**&#x200B;除錯&#x200B;<br/>**注意：**&#x200B;建議在安裝和測試期間將記錄級別設定為3，在生產環境中運行時將日誌級別設定為0。 |
| keepalivetimeout | 指定保持活動超時（秒）。 從Dispatcher 4.2.0版開始，預設的keep-alive值為60。 值0會停用keep-alive。 |

根據您的需求，您可以將Dispatcher定義為對象的服務。 要為整個網站配置Dispatcher，請修改預設對象：


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

您現在必須先開始使用Dispatcher:

* [](dispatcher-configuration.md) ConfigureDispatcher
* [設定](page-invalidate.md) AEM以搭配Dispatcher運作。
