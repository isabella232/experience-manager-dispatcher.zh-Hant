---
title: 安裝 Dispatcher
seo-title: Installing AEM Dispatcher
description: 了解如何在 Microsoft Internet Information Server、Apache Web Server 和 Sun Java Web Server-iPlanet 上安裝 Dispatcher 模組。
seo-description: Learn how to install the AEM Dispatcher module on Microsoft Internet Information Server, Apache Web Server and Sun Java Web Server-iPlanet.
uuid: 2384b907-1042-4707-b02f-fba2125618cf
contentOwner: User
converted: true
topic-tags: dispatcher
content-type: reference
discoiquuid: f00ad751-6b95-4365-8500-e1e0108d9536
exl-id: 9375d1c0-8d9e-46cb-9810-fa4162a8c1ba
source-git-commit: 63dc6184b502b517238c60ef6223b39bd7594306
workflow-type: tm+mt
source-wordcount: '0'
ht-degree: 0%

---

# 安裝 Dispatcher {#installing-dispatcher}

<!-- 

Comment Type: draft

<h2>Introduction</h2>

 -->

使用 [Dispatcher 發行說明](release-notes.md)頁面取得您的作業系統和網頁伺服器適用的最新 Dispatcher 安裝檔案。 Dispatcher 版本編號與 Adobe Experience Manager 版本編號無關，但與 Adobe Experience Manager 6.x、5.x 和 Adobe CQ 5.x 版本相容。

>[!NOTE]
>
>請注意，Adobe Experience Manager 6.5 需要 Dispatcher 版本 4.3.2 或更高版本。 舉例來說，Dispatcher 版本與 AEM 無關，但 Dispatcher 版本 4.3.2 也與 Adobe Experience Manager 6.4 相容。

以下是使用的檔案命名慣例：

`dispatcher-<web-server>-<operating-system>-<dispatcher-version-number>.<file-format>`

例如，`dispatcher-apache2.4-linux-x86_64-ssl-4.3.1.tar.gz` 檔案包含 Linux i686 上執行的 Apache 2.4 網頁伺服器適用的 Dispatcher 版本 4.3.1，而且會使用 **tar** 格式封裝。

下表列出每部網頁伺服器的檔案名稱中所使用的網頁伺服器識別碼：

| 網頁伺服器 | 安裝套件 |
|--- |--- |
| Apache 2.4 | dispatcher-apache **2.4**-&lt;其他參數> |
| Microsoft Internet Information Server 7.5、8、8.5 | dispatcher-**iis**-&lt;其他參數> |
| Sun Java Web Server iPlanet | dispatcher-**ns**-&lt;其他參數> |

>[!CAUTION]
>
>您應該安裝您的平台適用的最新版 Dispatcher。 您每年都應該升級 Dispatcher 執行個體來使用最新版，以充分利用產品改良功能。

>[!NOTE]
>
>明確地從版本 4.3.3 升級到版本 4.3.4 的客戶將會注意到，為無法快取的內容設定快取標頭的方式具有不同的行為。 若要進一步了解這項變更，請參閱[發行說明](/help/using/release-notes.md#nov)頁面。

每個封存都包含以下檔案：

* Dispatcher 模組
* 設定檔範例
* 包含安裝指示和最新資訊的讀我檔案
* 列出目前和過去的版本中已修正的問題的變更記錄檔案

>[!NOTE]
>
>在開始安裝前，請查看讀我檔案中的任何最新變更/平台專屬說明。

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

如需如何安裝此網頁伺服器的資訊，請參閱以下資源：

* Microsoft 自己的 Internet Information Server 相關文件
* [「Microsoft IIS 官方網站」](https://www.iis.net/)

### 必要 IIS 元件 {#required-iis-components}

IIS 版本 8.5 和 10 需要安裝以下 IIS 元件：

* ISAPI 擴充程式

此外，您也必須新增網頁伺服器 (IIS) 角色。 使用伺服器管理員新增角色和元件。

## Microsoft IIS - 安裝 Dispatcher 模組 {#microsoft-iis-installing-the-dispatcher-module}

Microsoft Internet Information System 所需的封存檔為：

* `dispatcher-iis-<operating-system>-<dispatcher-release-number>.zip`

ZIP 檔案包含以下檔案：

| 檔案 | 說明 |
|--- |--- |
| `disp_iis.dll` | Dispatcher 動態連結程式庫檔案。 |
| `disp_iis.ini` | IIS 適用的設定檔。 此範例可根據您的需求進行更新。 **注意**：ini 檔案必須具有與 dll 相同的 name-root。 |
| `dispatcher.any` | Dispatcher 的設定檔範例。 |
| `author_dispatcher.any` | 搭配編寫執行個體使用的 Dispatcher 適用的設定檔範例。 |
| 讀我檔案 | 包含安裝指示和最新資訊的讀我檔案。 **注意**：請在開始安裝前查看此檔案。 |
| 變更記錄 | 列出目前和過去的版本中已修正的問題的變更記錄檔案。 |

使用以下程序，將 Dispatcher 檔案複製到正確的位置。

1. 使用 Windows 檔案總管建立 `<IIS_INSTALLDIR>/Scripts` 目錄，例如 `C:\inetpub\Scripts`。

1. 將 Dispatcher 套件中的以下檔案解壓縮到這個 Scripts 目錄中：

   * `disp_iis.dll`
   * `disp_iis.ini`
   * 下列其中一個檔案取決於 Dispatcher 是搭配 AEM 編寫執行個體還是發佈執行個體使用：
      * 編寫執行個體：`author_dispatcher.any`
      * 發佈執行個體：`dispatcher.any`

## Microsoft IIS - 設定 Dispatcher INI 檔案 {#microsoft-iis-configure-the-dispatcher-ini-file}

編輯 `disp_iis.ini` 檔案來設定 Dispatcher 安裝。 `.ini` 檔案的基本格式如下：

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
| configpath | 本機檔案系統內的 `dispatcher.any` 位置 (絕對路徑)。 |
| logfile | `dispatcher.log` 檔案的位置。 如果未設定此屬性，則記錄訊息會記錄到 Windows 事件記錄檔。 |
| loglevel | 定義用來將訊息輸出到事件記錄檔的記錄層級。 可以指定下列值：記錄檔的記錄層級：<br/>0 - 僅限錯誤訊息。 <br/>1 - 錯誤和警告。 <br/>2 - 錯誤、警告和資訊訊息 <br/>3 - 錯誤、警告、資訊訊息和偵錯訊息。 <br/>**注意**：建議在安裝和測試期間將記錄層級設為 3，然後在生產環境執行時則設為 0。 |
| replaceauthorization | 指定如何處理 HTTP 請求中的授權標頭。 以下是有效的值：<br/>0 - 不修改 Authorization 標頭。 <br/>1 - 將任何名為「Authorization」的標頭 (「Basic」除外) 替換為其 `Basic <IIS:LOGON\_USER>` 同等標頭。<br/> |
| servervariables | 定義如何處理伺服器變數。<br/>0 - IIS 伺服器變數既不會傳送給 Dispatcher 也不會傳送給 AEM。 <br/>1 - 所有 IIS 伺服器變數 (例如 `LOGON\_USER, QUERY\_STRING, ...`) 都會傳送給 Dispatcher，連同請求標頭一起傳送 (如果未快取也會傳送給 AEM 執行個體)。 <br/>伺服器變數包括 `AUTH\_USER, LOGON\_USER, HTTPS\_KEYSIZE` 和其他許多變數。 請參閱 IIS 文件以取得完整變數清單，連同詳細資料。 |
| enable_chunked_transfer | 定義是要啟用 (1) 還是停用 (0) 用戶端回應的區塊傳輸。 預設值為 0。 |

設定範例：

```xml
[main]
configpath=C:\Inetpub\Scripts\dispatcher.any
loglevel=1
servervariables=1
replaceauthorization=0
```

### 設定 Microsoft IIS {#configuring-microsoft-iis}

設定 IIS 以整合 Dispatcher ISAPI 模組。 在 IIS 中，您會使用萬用字元應用程式對應。

### 設定匿名存取 - IIS 8.5 和 10 {#configuring-anonymous-access-iis-and}

編寫執行個體上的預設 Flush 複寫代理程式已設定為不隨著清除請求傳送安全性認證。 因此，您要用作 Dispatcher 快取的網站必須允許匿名存取。

如果您的網站使用驗證方法，則必須適當地設定 Flush 複寫代理程式。

1. 開啟「IIS管理器」，並選取您使用作為Dispatcher快取的網站。
1. 使用「功能檢視」模式，在 IIS 區段中按兩下「驗證」。
1. 如果未啟用「匿名驗證」，請選取「匿名驗證」，然後在「動作」區域中按一下「啟用」。

### 整合 Dispatcher ISAPI 模組 - IIS 8.5 和 10 {#integrating-the-dispatcher-isapi-module-iis-and}

使用以下程序，將 Dispatcher ISAPI 模組新增到 IIS。

1. 開啟 IIS 管理員。
1. 選取您要用作 Dispatcher 快取的網站。
1. 使用「功能檢視」模式，在 IIS 區段中按兩下「處理常式對應」。
1. 在「處理常式對應」頁面的「動作」面板中，按一下「新增萬用字元指令碼對應」，新增以下屬性值，然後按一下「確定」：

   * 請求路徑：&#42;
   * 可執行檔：disp_iis.dll 檔案的絕對路徑，例如 `C:\inetpub\Scripts\disp_iis.dll`。
   * 名稱：處理常式對應的說明性名稱，例如 `Dispatcher`。

1. 在出現的對話框中，若要將 disp_iis.dll 程式庫新增到 ISAPI 和 CGI 限制清單，請按一下「是」。

   對於 IIS 7.0 和 7.5，此設定是完整的。 如果您要設定 IIS 8.0，請繼續進行其餘步驟。

1. (IIS 8.0) 在處理常式對應清單中，選取您剛才建立的處理常式對應，然後在「動作」區域中按一下「編輯」。
1. (IIS 8.0) 在「編輯指令碼對應」對話框中，按一下「要求限制」按鈕。
1. (IIS 8.0) 若要確保處理常式是用於尚未快取的檔案和資料夾，請取消選取「只有當要求對應到下列項目時才啟動處理常式」，然後按一下「確定」。
1. (IIS 8.0) 在「編輯指令碼對應」對話框中，按一下「確定」。

### 設定對快取的存取 - IIS 8.5 和 10 {#configuring-access-to-the-cache-iis-and}

為預設應用程式集區使用者提供用作 Dispatcher 快取的資料夾的寫入權限。

1. 以滑鼠右鍵按一下您用作 Dispatcher 快取的網站的根資料夾，例如 `C:\inetpub\wwwroot`，然後按一下「內容」。
1. 在「安全性」索引標籤上，按一下「編輯」，然後在「權限」對話框中按一下「新增」。 隨即開啟一個對話框供您選取使用者帳戶。 按一下「位置」按鈕，並選取您的電腦名稱，然後按一下「確定」。

   當您在完成下一個步驟時，持續開啟此對話框。

1. 在 IIS 管理員中，選取您用作 Dispatcher 快取的 IIS 網站，然後在視窗右側按一下「進階設定」。
1. 選取「應用程式集區」屬性的值，並將其複製到剪貼簿。
1. 返回開啟的對話框。 在「輸入物件名稱來選取」對話框中，輸入 `IIS AppPool\`，然後貼上剪貼簿的內容。 該值看起來應該像下面的範例：

   `IIS AppPool\DefaultAppPool`

1. 按一下「檢查名稱」按鈕。 當 Windows 解析使用者帳戶時，按一下「確定」。
1. 在 Dispatcher 資料夾的「權限」對話框中，選取您剛才新增的帳戶，並為該帳戶啟用&#x200B;**完全控制以外**&#x200B;的所有權限，然後按一下「確定」。 按一下「確定」，關閉該資料夾的「內容」對話框。

### 登錄 JSON Mime 類型 - IIS 8.5 和 10 {#registering-the-json-mime-type-iis-and}

當您希望 Dispatcher 允許 JSON 呼叫時，請使用以下程序來登錄 JSON MIME 類型。

1. 在 IIS 管理員中，選取您的網站，並在「功能檢視」模式中按兩下「MIME 類型」。
1. 如果 JSON 副檔名未出現在清單中，請在「動作」面板中按一下「新增」，並輸入以下屬性值，然後按一下「確定」：

   * 副檔名：`.json`
   * MIME 類型：`application/json`

### 移除 bin 隱藏區段 - IIS 8.5 和 10 {#removing-the-bin-hidden-segment-iis-and}

使用以下程序可移除 `bin` 隱藏區段。 不是新的網站可以包含此隱藏區段。

1. 在 IIS 管理員中，選取您的網站，並在「功能檢視」模式中按兩下「要求篩選」。
1. 選取 `bin` 區段，並按一下「移除」，然後在確認對話框中按一下「是」。

### 將 IIS 訊息記錄到檔案中 - IIS 8.5 和 10 {#logging-iis-messages-to-a-file-iis-and}

使用以下程序，將 Dispatcher 記錄訊息寫入記錄檔，而不是 Windows 事件記錄檔。 您需要設定 Dispatcher 使用記錄檔，並為 IIS 提供該檔案的寫入權限。

1. 使用 Windows 檔案總管可在名為 `dispatcher` 的資料夾底下建立 IIS 安裝的記錄資料夾。 在典型的安裝中，此資料夾的路徑為 `C:\inetpub\logs\dispatcher`。

1. 以滑鼠右鍵按一下 dispatcher 資料夾，然後按一下「內容」。
1. 在「安全性」索引標籤上，按一下「編輯」，然後在「權限」對話框中按一下「新增」。 隨即開啟一個對話框供您選取使用者帳戶。 按一下「位置」按鈕，並選取您的電腦名稱，然後按一下「確定」。

   當您在完成下一個步驟時，持續開啟此對話框。

1. 在 IIS 管理員中，選取您用作 Dispatcher 快取的 IIS 網站，然後在視窗右側按一下「進階設定」。
1. 選取「應用程式集區」屬性的值，並將其複製到剪貼簿。
1. 返回開啟的對話框。 在「輸入物件名稱來選取」對話框中，輸入 `IIS AppPool\`，然後貼上剪貼簿的內容。 該值看起來應該像下面的範例：

   `IIS AppPool\DefaultAppPool`

1. 按一下「檢查名稱」按鈕。 當 Windows 解析使用者帳戶時，按一下「確定」。
1. 在 Dispatcher 資料夾的「權限」對話框中，選取您剛才新增的帳戶，並為該帳戶啟用&#x200B;**完全控制以外**&#x200B;的所有權限，然後按一下「確定」。 按一下「確定」，關閉該資料夾的「內容」對話框。
1. 使用文字編輯器開啟 `disp_iis.ini` 檔案。
1. 新增類似以下範例的一行文字以設定記錄檔的位置，然後儲存該檔案：

   ```xml
   logfile=C:\inetpub\logs\dispatcher\dispatcher.log
   ```

### 後續步驟 {#next-steps}

在您可以開始使用 Dispatcher 之前，您必須知道：

* [設定](dispatcher-configuration.md) Dispatcher
* [設定 AEM](page-invalidate.md) 以搭配 Dispatcher 使用。

## Apache Web Server {#apache-web-server}

>[!CAUTION]
>
>這裡有涵蓋 **Windows** 和 **Unix** 底下的安裝指示。 在執行步驟時請務必謹慎。

### 安裝 Apache Web Server {#installing-apache-web-server}

如需如何安裝 Apache Web Server 的相關資訊，請閱讀安裝手冊：[線上](https://httpd.apache.org/)版或散發版。

>[!CAUTION]
>
>如果您正在編譯來源檔案以建立 Apache 二進位檔，請務必開啟&#x200B;**動態模組支援**。 您可以使用任何 **--enable-shared** 選項進行此操作。 至少需要包含 `mod_so` 模組。
>
>Apache Web Server 安裝手冊中可以找到更多資訊。

也請參閱 Apache HTTP Server [安全性提示](https://httpd.apache.org/docs/2.4/misc/security_tips.html)和[安全性報告](https://httpd.apache.org/security_report.html)。

### Apache Web Server - 新增 Dispatcher 模組 {#apache-web-server-add-the-dispatcher-module}

Dispatcher 會以下列形式提供：

* **Windows**：動態連結程式庫 (DLL)
* **Unix**：動態共用物件 (DSO)

安裝封存檔案包含以下檔案 - 這取決於您已選取 Windows 還是 Unix：

| 檔案 | 說明 |
|--- |--- |
| disp_apache&lt;x.y>.dll | Windows：Dispatcher 動態連結程式庫檔案。 |
| dispatcher-apache&lt;x.y>-&lt;rel-nr>.so | Unix：Dispatcher 共用物件程式庫檔案。 |
| mod_dispatcher.so | Unix：範例連結。 |
| http.conf.disp&lt;x> | Apache Server 的設定檔範例。 |
| dispatcher.any | Dispatcher 的設定檔範例。 |
| 讀我檔案 | 包含安裝指示和最新資訊的讀我檔案。 **注意**：請在開始安裝前查看此檔案。 |
| 變更記錄 | 列出目前和過去的版本中已修正的問題的變更記錄檔案。 |

使用以下步驟，將 Dispatcher 新增到 Apache Web Server：

1. 將 Dispatcher 檔案放到適當的 Apache 模組目錄中：

   * **Windows**：將 `disp_apache<x.y>.dll` 放到 `<APACHE_ROOT>/modules`
   * **Unix**：根據您的安裝找到 `<APACHE_ROOT>/libexec` 或 `<APACHE_ROOT>/modules` 目錄。\
      將 `dispatcher-apache<options>.so` 複製到此目錄中。\
      若要簡化長期維護作業，您也可以建立指向 Dispatcher、名為 `mod_dispatcher.so` 的符號連結：\
      `ln -s dispatcher-apache<x>-<os>-<rel-nr>.so mod_dispatcher.so`

1. 將 dispatcher.any 檔案複製到 `<APACHE_ROOT>/conf` 目錄。

   **注意：**&#x200B;只要已適當設定 Dispatcher 模組的 DispatcherLog 屬性，就可以將這個檔案放在其他位置。 (請參閱底下的「Dispatcher 專屬設定項目」。)

### Apache Web Server - 設定 SELinux 屬性 {#apache-web-server-configure-selinux-properties}

如果您正在啟用了 SELinux 的 RedHat Linux Kernel 2.6 上執行 Dispatcher，您可能會在 dispatcher 記錄檔中看到與此類似的錯誤訊息。

`Mon Jun 30 00:03:59 2013] [E] [16561(139642697451488)] Unable to connect to backend rend01 (10.122.213.248:4502): Permission denied`

這可能是因為啟用了 SELinux 安全性的緣故。 然後您需要執行下列工作：

* 設定 dispatcher 模組檔案的 SELinux 上下文。
* 啟用 HTTPD 指令碼和模組，以建立網路連線。
* 設定快取檔案儲存所在的 docroot 的 SELinux 上下文。

在終端機視窗中輸入以下命令，將 `[path to the dispatcher.so file]` 替換為您安裝到 Apache Web Server 的 Dispatcher 模組的路徑，並將 *`path to the docroot`* 替換為 docroot 所在的路徑 (例如 `/opt/cq/cache`)：

```shell
semanage fcontext -a -t httpd_modules_t [path to the dispatcher.so file]
setsebool -P httpd_can_network_connect on
chcon -R --type httpd_sys_content_t [path to the docroot]
semanage fcontext -a -t httpd_sys_content_t "[path to the docroot](/.*)?"
```

### Apache Web Server - 為 Dispatcher 設定 Apache Web Server {#apache-web-server-configure-apache-web-server-for-dispatcher}

需要使用 `httpd.conf` 設定 Apache Web Server。 在 Dispatcher 安裝套件中，您將會找到名為 `httpd.conf.disp<x>` 的設定檔範例。

這些步驟是必要的：

1. 導覽至 `<APACHE_ROOT>/conf`。
1. 開啟 `httpd.conf` 進行編輯。
1. 必須依照列出的順序新增以下設定項目：

   * **LoadModule**，可在啟動時載入此模組。
   * Dispatcher 專屬的設定項目，包括 **DispatcherConfig、DispatcherLog** 和 **DispatcherLogLevel**。
   * **SetHandler**，可啟用 Dispatcher。 **LoadModule**。
   * **ModMimeUsePathInfo**，可設定 **mod_mime** 的行為。

1. (選擇性) 建議您變更 htdocs 目錄的所有者：

   * Apache Server 會以 root 身分啟動，但是子處理程序會以精靈形式啟動 (基於安全理由)。 DocumentRoot (`<APACHE_ROOT>/htdocs`) 必須屬於使用者精靈：

      ```xml
      cd <APACHE_ROOT>  
      chown -R daemon:daemon htdocs
      ```

**LoadModule**

下表列出可以使用的範例；確切的項目是根據您專屬的 Apache Web Server：

|  |  |
|--- |--- |
| Windows | `... LoadModule dispatcher_module modules\disp_apache.dll ...` |
| Unix (假定符號連結) | `... LoadModule dispatcher_module libexec/mod_dispatcher.so ...` |

>[!NOTE]
>
>每個陳述式的第一個參數都必須完全依照上面的範例來撰寫。
>
>如需這個命令的完整詳細資訊，請參閱提供的設定檔範例及 Apache Web Server 文件。

**Dispatcher 專屬設定項目**

Dispatcher 專屬設定項目會放在 LoadModule 項目後面。 下表列出同時適用於 Unix 和 Windows 的設定範例：

**Windows 和 Unix**

```
...
<IfModule disp_apache2.c>
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
| DispatcherConfig | Dispatcher 設定檔的位置和名稱。 <br/>當這個屬性位在主要伺服器設定中時，所有虛擬主機都會繼承此屬性值。 不過，虛擬主機可以包含 DispatcherConfig 屬性以覆寫主要伺服器設定。 |
| DispatcherLog | 記錄檔的位置和名稱。 |
| DispatcherLogLevel | 記錄檔的記錄層級：<br/>0 - 錯誤 <br/>1 - 警告 <br/>2 - 資訊 <br/>3 - 偵錯 <br/>**注意**：建議在安裝和測試期間將記錄層級設為 3，然後在生產環境執行時則設為 0。 |
| DispatcherNoServerHeader | *此參數已過時，不再有任何效用。*<br/><br/> 定義要使用的伺服器標頭：<br/><ul><li>未定義或 0 - HTTP 伺服器標頭包含 AEM 版本。 </li><li>1 - 使用 Apache 伺服器標頭。</li></ul> |
| DispatcherDeclineRoot | 定義是否拒絕對根目錄「/」的請求：<br/>**0** - 接受對 / 的請求 <br/>**1** - 對 / 的請求不是由 Dispatcher 處理；使用 mod_alias 可獲得正確的對應。 |
| DispatcherUseProcessedURL | 定義是否要使用已預先處理的 URL 以供 Dispatcher 的所有進一步處理使用：<br/>**0** - 使用傳遞給網頁伺服器的原始 URL。 <br/>**1** - Dispatcher 會使用已由其前面的處理常式處理的 URL (亦即 `mod_rewrite`)，而不是使用傳遞給網頁伺服器的原始 URL。 例如，不是原始 URL 就是處理過的 URL 會符合 Dispatcher 篩選條件。 此 URL 也會用作快取檔案結構的基礎。 如需有關 mod_rewrite 的資訊，請參閱 Apache 網站文件；例如 Apache 2.4。在使用 mod_rewrite 時，建議您使用標幟「passthrough | PT」(傳遞給下一個處理常式)，以強制重寫引擎將內部 request_rec 結構的 uri 欄位設定為檔案名稱欄位的值。 |
| DispatcherPassError | 定義如何支援 ErrorDocument 處理的錯誤碼：<br/>**0** - Dispatcher 將所有錯誤回應多工緩衝到用戶端。 <br/>**1** - Dispatcher 不會將錯誤回應多工緩衝到用戶端 (此時的狀態代碼大於或等於 400)，但會將狀態代碼傳遞給 Apache，以允許 ErrorDocument 指示詞處理這類狀態代碼。 <br/>**代碼範圍** - 指定將回應傳遞給 Apache 所適用的錯誤碼範圍。 其他錯誤碼則會傳遞給用戶端。 例如，以下設定會將錯誤 412 的回應傳遞給用戶端，並將其他所有錯誤傳遞給 Apache：DispatcherPassError 400-411,413-417 |
| DispatcherKeepAliveTimeout | 指定保持連線逾時值 (以秒為單位)。 從 Dispatcher 版本 4.2.0 開始，預設保持連線值為 60。 0 的值會停用保持連線。 |
| DispatcherNoCanonURL | 將此參數設為開啟會將原始 URL 傳遞到後端，而不是傳遞規範化 URL，而且將會覆寫 DispatcherUseProcessedURL 的設定。 預設值為關閉。 <br/>**注意**：Dispatcher 設定中的篩選規則一律會根據經過清理的 URL (而不是原始 URL) 進行評估。 |




>[!NOTE]
>
>路徑項目相對於 Apache Web Server 的根目錄。

>[!NOTE]
>
>伺服器標頭的預設設定如下：
>
>`ServerTokens Full`
>
>`DispatcherNoServerHeader 0`
>
>這會顯示 AEM 版本 (基於統計目的)。 如果您想要停用在標頭中提供這類資訊的功能，您可以設定：
>
>`ServerTokens Prod`
>
>如需詳細資訊，請參閱[有關 ServerTokens 指示詞的 Apache 文件 (例如 Apache 2.4)](https://httpd.apache.org/docs/2.4/mod/core.html)。

**SetHandler**

在這些項目後，您必須將 **SetHandler** 陳述式新增到您設定的上下文 (`<Directory>`、`<Location>`) 好讓 Dispatcher 處理傳入的請求。 以下範例會設定 Dispatcher 處理完整網站的請求：

**Windows 和 Unix**

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

以下範例會設定 Dispatcher 處理虛擬網域的請求：

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
>
>**SetHandler** 陳述式的參數必須&#x200B;*完全依照上面的範例來撰寫*，因為這是模組中所定義的處理常式名稱。
>
>如需這個命令的完整詳細資訊，請參閱提供的設定檔範例及 Apache Web Server 文件。

**ModMimeUsePathInfo**

在 **SetHandler** 陳述式之後，您也應該新增 **ModMimeUsePathInfo** 定義。

>[!NOTE]
>
>只有當您使用 Dispatcher 4.0.9 或更高版本時，才應該使用及設定 `ModMimeUsePathInfo` 參數。
>
>(請注意，Dispatcher 版本 4.0.9 已在 2011 年發行。 如果您使用較舊的版本，升級到最近的 Dispatcher 版本會比較恰當)。

所有 Apache 設定都應該將 **ModMimeUsePathInfo** 參數設為 `On`：

`ModMimeUsePathInfo On`

mod_mime 模組 (例如，請參閱 [Apache 模組 mod_mime](https://httpd.apache.org/docs/2.4/mod/mod_mime.html)) 是用來將內容中繼資料指派給為 HTTP 回應選取的內容。 預設設定表示當 mod_mime 判斷內容類型時，只會考量對應到檔案或目錄的 URL 部分。

當設為 `On` 時，`ModMimeUsePathInfo` 參數會指定 `mod_mime` 根據&#x200B;*完整* URL 來判斷內容類型；這表示虛擬資源將會根據其副檔名套用中繼資訊。

以下範例會啟用 **ModMimeUsePathInfo**：

**Windows 和 Unix**

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

### 啟用對 HTTPS 的支援 (Unix 和 Linux) {#enable-support-for-https-unix-and-linux}

Dispatcher 會使用 OpenSSL 來實作透過 HTTP 的安全通訊。 從 Dispatcher 版本 **4.2.0** 開始，就有支援 OpenSSL 1.0.0 和 OpenSSL 1.0.1。 Dispatcher 預設會使用 OpenSSL 1.0.0。 若要使用 OpenSSL 1.0.1，請使用以下程序來建立符號連結，好讓 Dispatcher 使用安裝的 OpenSSL 程式庫。

1. 開啟終端機，並將目前目錄切換到已安裝 OpenSSL 程式庫的目錄，例如：

   ```shell
   cd /usr/lib64
   ```

1. 若要建立符號連結，請輸入以下命令：

   ```shell
   ln -s libssl.so libssl.so.1.0.1
   ln -s libcrypto.so libcrypto.so.1.0.1
   ```

>[!NOTE]
>
>如果您使用自訂版的 Apache，請務必使用相同版本的 [OpenSSL](https://www.openssl.org/source/) 編譯 Apache 和 Dispatcher。

### 後續步驟 {#next-steps-1}

在您可以開始使用 Dispatcher 之前，您必須知道：

* [設定](dispatcher-configuration.md) Dispatcher
* [設定 AEM](page-invalidate.md) 以搭配 Dispatcher 使用。

## Sun Java System Web Server / iPlanet {#sun-java-system-web-server-iplanet}

>[!NOTE]
>
>這裡有涵蓋 Windows 和 Unix 環境的相關指示。
>
>在選取要執行的操作時，請務必謹慎。

### Sun Java System Web Server / iPlanet - 安裝您的網頁伺服器 {#sun-java-system-web-server-iplanet-installing-your-web-server}

如需如何安裝這些網頁伺服器的完整資訊，請參閱其各自的文件：

* Sun Java System Web Server
* iPlanet Web Server

### Sun Java System Web Server / iPlanet - 新增 Dispatcher 模組 {#sun-java-system-web-server-iplanet-add-the-dispatcher-module}

Dispatcher 會以下列形式提供：

* **Windows**：動態連結程式庫 (DLL)
* **Unix**：動態共用物件 (DSO)

安裝封存檔案包含以下檔案 - 這取決於您已選取 Windows 還是 Unix：

| 檔案 | 說明 |
|---|---|
| `disp_ns.dll` | Windows：Dispatcher 動態連結程式庫檔案。 |
| `dispatcher.so` | Unix：Dispatcher 共用物件程式庫檔案。 |
| `dispatcher.so` | Unix：範例連結。 |
| `obj.conf.disp` | iPlanet / Sun Java System Web Server 的設定檔範例。 |
| `dispatcher.any` | Dispatcher 的設定檔範例。 |
| 讀我檔案 | 包含安裝指示和最新資訊的讀我檔案。 注意：請在開始安裝前查看此檔案。 |
| 變更記錄 | 列出目前和過去的版本中已修正的問題的變更記錄檔案。 |

使用以下步驟，將 Dispatcher 新增到您的網頁伺服器：

1. 將 Dispatcher 檔案放到網頁伺服器的 `plugin` 目錄中：

### Sun Java System Web Server / iPlanet - 為 Dispatcher 進行設定 {#sun-java-system-web-server-iplanet-configure-for-the-dispatcher}

需要使用 `obj.conf` 設定網頁伺服器。 在 Dispatcher 安裝套件中，您將會找到名為 `obj.conf.disp` 的設定檔範例。

1. 導覽至 `<WEBSERVER_ROOT>/config`。
1. 開啟 `obj.conf` 進行編輯。
1. 複製開頭如下的那一行：\
   `Service fn="dispService"`\
   ：從 `obj.conf.disp` 到 `obj.conf` 的 initialization 區段。

1. 儲存變更。
1. 開啟 `magnus.conf` 進行編輯。
1. 複製開頭如下的那兩行：\
   `Init funcs="dispService, dispInit"`\
   和\
   `Init fn="dispInit"`\
   ：從 `obj.conf.disp` 到 `magnus.conf` 的 initialization 區段。

1. 儲存變更。

>[!NOTE]
>
>以下設定應該全都位在同一行，且 `$(SERVER_ROOT)` 和 `$(PRODUCT_SUBDIR)` 必須替換為各自的值。

**Init**

下表列出可以使用的範例；確切的項目是根據您專屬的網頁伺服器：

**Windows 和 Unix**

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
| config | 設定檔 `dispatcher.any.` 的位置和名稱 |
| 記錄檔 | 記錄檔的位置和名稱。 |
| loglevel | 將訊息寫入記錄檔時的記錄層級：<br/>**0** 錯誤 <br/>**1** 警告 <br/>**2** 資訊 <br/>**3** 偵錯 <br/>**注意：**&#x200B;建議在安裝和測試期間將記錄層級設為 3，然後在生產環境執行時則設為 0。 |
| keepalivetimeout | 指定保持連線逾時值 (以秒為單位)。 從 Dispatcher 版本 4.2.0 開始，預設保持連線值為 60。 0 的值會停用保持連線。 |

您可以根據您的需求，將 Dispatcher 定義為物件的服務。 若要為整個網站設定 Dispatcher，請修改預設物件：


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

### 後續步驟 {#next-steps-2}

在您可以開始使用 Dispatcher 之前，您必須知道：

* [設定](dispatcher-configuration.md) Dispatcher
* [設定 AEM](page-invalidate.md) 以搭配 Dispatcher 使用。
