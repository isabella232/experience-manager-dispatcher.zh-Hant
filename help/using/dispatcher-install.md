---
title: 安裝 Dispatcher
seo-title: Installing AEM Dispatcher
description: 瞭解如何在MicrosoftInternet Information Server、Apache Web Server和Sun Java Web Server-iPlanet上安裝Dispatcher模組。
seo-description: Learn how to install the AEM Dispatcher module on Microsoft Internet Information Server, Apache Web Server and Sun Java Web Server-iPlanet.
uuid: 2384b907-1042-4707-b02f-fba2125618cf
contentOwner: User
converted: true
topic-tags: dispatcher
content-type: reference
discoiquuid: f00ad751-6b95-4365-8500-e1e0108d9536
exl-id: 9375d1c0-8d9e-46cb-9810-fa4162a8c1ba
source-git-commit: d19a27256c44ec00fd440b2f8a2fe408a4a4b7c8
workflow-type: tm+mt
source-wordcount: '3693'
ht-degree: 1%

---

# 安裝 Dispatcher {#installing-dispatcher}

<!-- 

Comment Type: draft

<h2>Introduction</h2>

 -->

使用 [《 Dispatcher發行說明》](release-notes.md) 頁，以獲取作業系統和Web伺服器的最新Dispatcher安裝檔案。 Dispatcher版本號與Adobe Experience Manager版本號無關，與Adobe Experience Manager6.x、5.x和Adobe CQ5.x版本相容。

>[!NOTE]
>
>請注意，Adobe Experience Manager6.5要求Dispatcher版本4.3.2或更高版本。 儘管如此， Dispatcher版本AEM與Adobe Experience Manager6.4相容，例如， Dispatcher版本4.3.2也與Dispatcher版本無關。

使用以下檔案命名約定：

`dispatcher-<web-server>-<operating-system>-<dispatcher-version-number>.<file-format>`

例如， `dispatcher-apache2.4-linux-x86_64-ssl-4.3.1.tar.gz` 檔案包含運行在Linux i686上的Apache 2.4 Web伺服器的4.3.1版Dispatcher ，並使用 **焦油** 的子菜單。

下表列出了在每個Web伺服器的檔案名中使用的Web伺服器標識符：

| Web伺服器 | 安裝套件 |
|--- |--- |
| Apache 2.4 | Dispatcher-Apache **2.4**-&lt;other parameters=&quot;&quot;> |
| MicrosoftInternet Information Server 7.5、8、8.5 | 調度程式&#x200B;**ii**-&lt;other parameters=&quot;&quot;> |
| Sun Java Web Server iPlanet | 調度程式&#x200B;**ns**-&lt;other parameters=&quot;&quot;> |

>[!CAUTION]
>
>您應安裝適用於您的平台的最新版本的Dispatcher。 每年，您應升級Dispatcher實例以使用最新版本來利用產品改進。

>[!NOTE]
>
>從4.3.3版專門升級到4.3.4版的客戶將注意到如何設定快取標題以獲取不可訪問內容的不同行為。 若要瞭解有關此更改的詳細資訊，請參閱 [發行說明](/help/using/release-notes.md#nov) 的子菜單。

每個存檔都包含以下檔案：

* 調度器模組
* 示例配置檔案
* 包含安裝說明和最後一分鐘資訊的自述檔案
* 列出當前版本和過去版本中已修復問題的CHANGES檔案

>[!NOTE]
>
>在開始安裝之前，請查看自述檔案，瞭解任何最後時刻的更改/特定於平台的說明。

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

## Microsoft網際網路資訊伺服器 {#microsoft-internet-information-server}

有關如何安裝此Web伺服器的資訊，請參閱以下資源：

* Microsoft在Internet Information Server上的文檔
* [&quot;官方MicrosoftIIS站點&quot;](https://www.iis.net/)

### 所需的IIS元件 {#required-iis-components}

IIS 8.5和10版要求安裝以下IIS元件：

* ISAPI擴展

此外，還必須添加Web伺服器(IIS)角色。 使用伺服器管理器添加角色和元件。

## MicrosoftIIS — 安裝Dispatcher模組 {#microsoft-iis-installing-the-dispatcher-module}

Microsoft網際網路資訊系統所需的檔案是：

* `dispatcher-iis-<operating-system>-<dispatcher-release-number>.zip`

ZIP檔案包含以下檔案：

| 檔案 | 說明 |
|--- |--- |
| `disp_iis.dll` | Dispatcher動態連結庫檔案。 |
| `disp_iis.ini` | IIS的配置檔案。 此示例可以根據您的要求進行更新。 **注釋**:ini檔案必須與dll具有相同的name-root。 |
| `dispatcher.any` | Dispatcher的示例配置檔案。 |
| `author_dispatcher.any` | Dispatcher使用作者實例的示例配置檔案。 |
| 自述檔案 | 包含安裝說明和最後一分鐘資訊的自述檔案。 **注釋**:請在開始安裝之前檢查此檔案。 |
| 更改 | 更改列出當前版本和過去版本中已修復問題的檔案。 |

請按下列步驟將Dispatcher檔案複製到正確的位置。

1. 使用Windows資源管理器建立 `<IIS_INSTALLDIR>/Scripts` 例如， `C:\inetpub\Scripts`。

1. 將以下檔案從Dispatcher包解壓到此Scripts目錄：

   * `disp_iis.dll`
   * `disp_iis.ini`
   * 根據Dispatcher是使用作者實例還是發佈實例，以AEM下檔案之一：
      * 作者實例： `author_dispatcher.any`
      * 發佈實例： `dispatcher.any`

## MicrosoftIIS — 配置Dispatcher INI檔案 {#microsoft-iis-configure-the-dispatcher-ini-file}

編輯 `disp_iis.ini` 檔案以配置Dispatcher安裝。 的基本格式 `.ini` 檔案如下所示：

```xml
[main]
configpath=<path to dispatcher.any>
loglevel=1|2|3
servervariables=0|1
replaceauthorization=0|1
```

下表介紹了每個屬性。

| 參數 | 說明 |
|--- |--- |
| 配置路徑 | 位置 `dispatcher.any` 本地檔案系統（絕對路徑）中。 |
| 日誌檔案 | 位置 `dispatcher.log` 的子菜單。 如果未設定此設定，則日誌消息將轉到windows事件日誌。 |
| 日誌 | 定義用於將消息輸出到事件日誌的日誌級別。 可以指定以下值：日誌檔案的日誌級別： <br/>0 — 僅錯誤消息。 <br/>1 — 錯誤和警告。 <br/>2 — 錯誤、警告和資訊性消息 <br/>3 — 錯誤、警告、資訊和調試消息。 <br/>**注釋**:建議在安裝和測試期間將日誌級別設定為3，在生產環境中運行時將日誌級別設定為0。 |
| 替換授權 | 指定如何處理HTTP請求中的授權標頭。 以下值有效：<br/>0 — 未修改授權標頭。 <br/>1 — 將除「Basic」外的任何名為「Authorization」的標題替換為 `Basic <IIS:LOGON\_USER>` 等效。<br/> |
| 伺服器變數 | 定義處理伺服器變數的方式。<br/>0 - IIS伺服器變數既不發送給Dispatcher，也不發送AEM。 <br/>1 — 所有IIS伺服器變數(如 `LOGON\_USER, QUERY\_STRING, ...`)與請求標頭一起發送到Dispatcher(如果未快取，也AEM會發送到實例)。  <br/>伺服器變數包括 `AUTH\_USER, LOGON\_USER, HTTPS\_KEYSIZE` 和其他很多。 有關變數的完整清單，請參閱IIS文檔，並提供詳細資訊。 |
| enable_chunked_transfer | 定義是啟用(1)還是禁用(0)客戶端響應的分塊傳輸。 預設值為 0。 |

配置示例：

```xml
[main]
configpath=C:\Inetpub\Scripts\dispatcher.any
loglevel=1
servervariables=1
replaceauthorization=0
```

### 配置MicrosoftIIS {#configuring-microsoft-iis}

配置IIS以整合Dispatcher ISAPI模組。 在IIS中，您使用通配符應用程式映射。

### 配置匿名訪問 — IIS 8.5和10 {#configuring-anonymous-access-iis-and}

已配置Author實例上的預設刷新複製代理，以便它不會發送具有刷新請求的安全憑據。 因此，您使用的Dispatcher快取的網站必須允許匿名訪問。

如果網站使用身份驗證方法，則必須相應地配置刷新複製代理。

1. 開啟IIS管理器，並選擇您正用作Disptcher快取的網站。
1. 使用「功能視圖」模式，在IIS部分中按兩下「驗證」。
1. 如果未啟用匿名身份驗證，請選擇匿名身份驗證，然後在「操作」區域按一下啟用。

### 整合Dispatcher ISAPI模組 — IIS 8.5和10 {#integrating-the-dispatcher-isapi-module-iis-and}

請按下列步驟將Dispatcher ISAPI模組添加到IIS。

1. 開啟IIS管理器。
1. 選擇您用作Dispatcher Cache的網站。
1. 使用「功能視圖」模式，在IIS部分中按兩下「處理程式映射」。
1. 在「處理程式映射」頁的「操作」面板中，按一下添加通配符指令碼映射，添加以下屬性值，然後按一下確定：

   * 請求路徑：*
   * 執行檔：disp_iis.dll檔案的絕對路徑，例如 `C:\inetpub\Scripts\disp_iis.dll`。
   * 名稱：處理程式映射的描述性名稱，例如 `Dispatcher`。

1. 在顯示的對話框中，要將disp_iis.dll庫添加到ISAPI和CGI限制清單中，請按一下「是」。

   對於IIS 7.0和7.5，配置已完成。 如果要配置IIS 8.0，請繼續執行其餘步驟。

1. (IIS 8.0)在處理程式映射清單中，選擇剛建立的映射，然後在「操作」區域中按一下「編輯」。
1. (IIS 8.0)在「編輯指令碼映射」對話框中，按一下「請求限制」按鈕。
1. (IIS 8.0)要確保處理程式用於尚未快取的檔案和資料夾，請取消選擇僅在請求映射到時調用處理程式，然後按一下確定。
1. (IIS 8.0)在「編輯指令碼映射」對話框上，按一下「確定」。

### 配置對快取的訪問 — IIS 8.5和10 {#configuring-access-to-the-cache-iis-and}

為預設的應用程式池用戶提供對用作Dispatcher快取的資料夾的寫訪問權限。

1. 按一下右鍵用作Dispatcher快取的網站的根資料夾，然後按一下「屬性」，如 `C:\inetpub\wwwroot`。
1. 在「安全性」頁籤上，按一下「編輯」，然後在「權限」對話框中，按一下「添加」。 將開啟一個對話框，用於選擇用戶帳戶。 按一下「Locations（位置）」按鈕，選擇電腦名稱，然後按一下「OK（確定）」。

   完成下一步時，保持此對話框開啟。

1. 在IIS管理器中，選擇您正用於Dispatcher快取的IIS站點，然後在窗口的右側按一下「高級設定」。
1. 選擇「應用程式池」屬性的值，並將其複製到剪貼簿。
1. 返回到開啟的對話框。 在「輸入要選擇的對象名稱」(Enter the Object Names To Select)框中，鍵入 `IIS AppPool\` 然後貼上剪貼簿中的內容。 該值應與以下示例類似：

   `IIS AppPool\DefaultAppPool`

1. 按一下「檢查名稱」按鈕。 當Windows解析用戶帳戶時，按一下確定。
1. 在調度程式資料夾的「權限」對話框中，選擇您剛添加的帳戶，啟用該帳戶的所有權限  **除完全控制外** 然後按一下「確定」。 按一下「確定」關閉資料夾「屬性」對話框。

### 註冊JSON Mime類型 — IIS 8.5和10 {#registering-the-json-mime-type-iis-and}

如果希望Dispatcher允許JSON調用，請使用以下過程註冊JSON MIME類型。

1. 在IIS管理器中，選擇網站，然後使用「功能視圖」按兩下「Mime類型」。
1. 如果JSON擴展不在清單中，請在「操作」面板中按一下「添加」，輸入以下屬性值，然後按一下「確定」：

   * 檔案副檔名： `.json`
   * MIME類型： `application/json`

### 刪除紙盒隱藏段 — IIS 8.5和10 {#removing-the-bin-hidden-segment-iis-and}

請按下列步驟刪除 `bin` 隱藏段。 非新網站可包含此隱藏段。

1. 在IIS管理器中，選擇網站，然後使用「功能視圖」按兩下「請求篩選」。
1. 選擇 `bin` 段，按一下「刪除」，然後在確認對話框中按一下「是」。

### 將IIS消息記錄到檔案 — IIS 8.5和10 {#logging-iis-messages-to-a-file-iis-and}

請按下列步驟將Dispatcher日誌消息寫入日誌檔案，而不是寫入Windows事件日誌。 您需要配置Dispatcher以使用日誌檔案，並為IIS提供對該檔案的寫訪問。

1. 使用Windows資源管理器建立名為 `dispatcher` IIS安裝的logs資料夾下。 此資料夾的典型安裝路徑是 `C:\inetpub\logs\dispatcher`。

1. 按一下右鍵調度程式資料夾，然後按一下「屬性」。
1. 在「安全性」頁籤上，按一下「編輯」，然後在「權限」對話框中，按一下「添加」。 將開啟一個對話框，用於選擇用戶帳戶。 按一下「Locations（位置）」按鈕，選擇電腦名稱，然後按一下「OK（確定）」。

   完成下一步時，保持此對話框開啟。

1. 在IIS管理器中，選擇您正用於Dispatcher快取的IIS站點，然後在窗口的右側按一下「高級設定」。
1. 選擇「應用程式池」屬性的值，並將其複製到剪貼簿。
1. 返回到開啟的對話框。 在「輸入要選擇的對象名稱」(Enter the Object Names To Select)框中，鍵入 `IIS AppPool\` 然後貼上剪貼簿中的內容。 該值應與以下示例類似：

   `IIS AppPool\DefaultAppPool`

1. 按一下「檢查名稱」按鈕。 當Windows解析用戶帳戶時，按一下確定。
1. 在調度程式資料夾的「權限」對話框中，選擇您剛添加的帳戶，啟用該帳戶的所有權限 **除了完全控制，** 然後按一下「確定」。 按一下「確定」關閉資料夾「屬性」對話框。
1. 使用文本編輯器開啟 `disp_iis.ini` 的子菜單。
1. 添加一行與以下示例類似的文本，以配置日誌檔案的位置，然後保存檔案：

   ```xml
   logfile=C:\inetpub\logs\dispatcher\dispatcher.log
   ```

### 後續步驟 {#next-steps}

在開始使用調度程式之前，您必須知道：

* [配置](dispatcher-configuration.md) 調度程式
* [配置AEM](page-invalidate.md) 與Dispatcher合作。

## Apache Web伺服器 {#apache-web-server}

>[!CAUTION]
>
>兩者下的安裝說明 **窗口** 和 **Unix** 在這裡。 執行步驟時請小心。

### 安裝Apache Web Server {#installing-apache-web-server}

有關如何安裝Apache Web Server的資訊，請閱讀安裝手冊 —  [線上](https://httpd.apache.org/) 或分發。

>[!CAUTION]
>
>如果通過編譯源檔案建立Apache二進位檔案，請確保開啟 **動態模組支援**。 這可以通過使用 **— 啟用共用** 頁籤 至少包括 `mod_so` 中。
>
>有關詳細資訊，請參閱Apache Web Server安裝手冊。

另請參見Apache HTTP Server [安全提示](https://httpd.apache.org/docs/2.4/misc/security_tips.html) 和 [安全報告](https://httpd.apache.org/security_report.html)。

### Apache Web Server — 添加Dispatcher模組 {#apache-web-server-add-the-dispatcher-module}

Dispatcher的格式為：

* **窗口**:動態連結庫(DLL)
* **Unix**:動態共用對象(DSO)

安裝存檔檔案包含以下檔案 — 取決於您是選擇了Windows還是Unix:

| 檔案 | 說明 |
|--- |--- |
| disp_apache&lt;x.y>.dll | 窗口：Dispatcher動態連結庫檔案。 |
| Dispatcher-Apache&lt;x.y>-&lt;rel-nr>.so | Unix:Dispatcher共用對象庫檔案。 |
| mod_dispatcher.so | Unix:示例連結。 |
| http.conf.disp&lt;x> | Apache伺服器的示例配置檔案。 |
| dispatcher.any | Dispatcher的示例配置檔案。 |
| 自述檔案 | 包含安裝說明和最後一分鐘資訊的自述檔案。 **注釋**:請在開始安裝之前檢查此檔案。 |
| 更改 | 更改列出當前版本和過去版本中已修復問題的檔案。 |

使用以下步驟將Dispatcher添加到Apache Web Server:

1. 將Dispatcher檔案置於相應的Apache模組目錄中：

   * **窗口**:位置 `disp_apache<x.y>.dll` `<APACHE_ROOT>/modules`
   * **Unix**:查找 `<APACHE_ROOT>/libexec` 或 `<APACHE_ROOT>/modules`的子菜單。\
      複製 `dispatcher-apache<options>.so` 到此目錄。\
      為簡化長期維護，您還可以建立一個名為 `mod_dispatcher.so` 調度程式：\
      `ln -s dispatcher-apache<x>-<os>-<rel-nr>.so mod_dispatcher.so`

1. 將dispatcher.any檔案複製到 `<APACHE_ROOT>/conf` 的子菜單。

   **注：** 只要相應地配置了Dispatcher模組的DispatcherLog屬性，就可以將此檔案放置在其他位置。 （請參閱下面的Dispatcher-Specific Configuration Entries。）

### Apache Web Server — 配置SELinux屬性 {#apache-web-server-configure-selinux-properties}

如果在啟用了SELinux的RedHat Linux Kernel 2.6上運行Dispatcher，則可能會在dispatcher日誌檔案中遇到類似這樣的錯誤消息。

`Mon Jun 30 00:03:59 2013] [E] [16561(139642697451488)] Unable to connect to backend rend01 (10.122.213.248:4502): Permission denied`

這可能是由於啟用了SELinux安全性。 然後，您需要執行以下任務：

* 配置調度程式模組檔案的SELinux上下文。
* 啟用HTTPD指令碼和模組以建立網路連接。
* 配置域的SELinux上下文，其中儲存快取檔案。

在終端窗口中輸入以下命令，替換 `[path to the dispatcher.so file]` 指向您安裝到Apache Web Server的Dispatcher模組的路徑， *`path to the docroot`* 與docroot所在的路徑(例如 `/opt/cq/cache`):

```shell
semanage fcontext -a -t httpd_modules_t [path to the dispatcher.so file]
setsebool -P httpd_can_network_connect on
chcon -R --type httpd_sys_content_t [path to the docroot]
semanage fcontext -a -t httpd_sys_content_t "[path to the docroot](/.*)?"
```

### Apache Web Server — 為Dispatcher配置Apache Web Server {#apache-web-server-configure-apache-web-server-for-dispatcher}

需要配置Apache Web Server，使用 `httpd.conf`。 在Dispatcher安裝工具包中，您將找到一個名為 `httpd.conf.disp<x>`。

這些步驟是強制性的：

1. 導航到 `<APACHE_ROOT>/conf`.
1. 開啟 `httpd.conf`的子菜單。
1. 必須按所列順序添加以下配置條目：

   * **載入模組** 啟動時載入模組。
   * 特定於調度程式的配置項，包括 **DispatcherConfig、DispatcherLog** 和 **DispatcherLogLevel**。
   * **集處理程式** 激活調度程式。 **載入模組**。
   * **ModMimeUsePathInfo** 配置行為 **mod_mime**。

1. （可選）建議更改htdocs目錄的所有者：

   * apache伺服器以root身份啟動，但子進程以守護進程身份啟動（出於安全目的）。 文檔根(`<APACHE_ROOT>/htdocs`)必須屬於用戶守護程式：

      ```xml
      cd <APACHE_ROOT>  
      chown -R daemon:daemon htdocs
      ```

**載入模組**

下表列出了可使用的示例；準確的條目取決於您的特定Apache Web Server:

|  |  |
|--- |--- |
| Windows | `... LoadModule dispatcher_module modules\disp_apache.dll ...` |
| Unix（假定為符號連結） | `... LoadModule dispatcher_module libexec/mod_dispatcher.so ...` |

>[!NOTE]
>
>每個語句的第一個參數必須與上述示例中完全相同。
>
>有關此命令的完整詳細資訊，請參閱提供的配置檔案示例和Apache Web Server文檔。

**調度程式特定的配置項**

Dispatcher特定的配置項放在LoadModule項之後。 下表列出了適用於Unix和Windows的示例配置：

**Windows和Unix**

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

單個配置參數：

| 參數 | 說明 |
|--- |--- |
| DispatcherConfig | Dispatcher配置檔案的位置和名稱。 <br/>當此屬性位於主伺服器配置中時，所有虛擬主機都繼承屬性值。 但是，虛擬主機可以包含DispatcherConfig屬性以覆蓋主伺服器配置。 |
| DispatcherLog | 日誌檔案的位置和名稱。 |
| DispatcherLogLevel | 日誌檔案的日誌級別： <br/>0 — 錯誤 <br/>1 — 警告 <br/>2 — 資訊 <br/>3 — 調試 <br/>**注釋**:建議在安裝和測試期間將日誌級別設定為3，在生產環境中運行時將日誌級別設定為0。 |
| DispatcherNoServerHeader | *此參數已棄用，不再有任何效果。*<br/><br/> 定義要使用的伺服器標頭： <br/><ul><li>undefined或0 - HTTP伺服器標頭包含AEM版本。 </li><li>1 — 使用Apache伺服器標頭。</li></ul> |
| DispatcherDellineRoot | 定義是否拒絕對根「/」的請求： <br/>**0**  — 接受請求/ <br/>**1**  — 請求/未由調度程式處理；使用mod_alias進行正確的映射。 |
| DispatcherUseProcessedURL | 定義是否將預處理的URL用於Dispatcher進一步處理的所有URL: <br/>**0**  — 使用傳遞到Web伺服器的原始URL。 <br/>**1**  — 調度程式使用調度程式之前的處理程式已處理的URL(即 `mod_rewrite`)，而不是傳遞到Web伺服器的原始URL。  例如，原始URL或已處理的URL與Dispatcher篩選器匹配。 該URL還用作快取檔案結構的基礎。   有關mod_rewrite；的資訊，請參閱Apache網站文檔。例如，Apache 2.4。使用mod_rewrite時，建議使用標誌「passthrough」 | PT&#39;（傳遞到下一個處理程式）強制重寫引擎將內部request_rec結構的uri欄位設定為filename欄位的值。 |
| DispatcherPassError | 定義如何支援ErrorDocument處理的錯誤代碼： <br/>**0** - Dispatcher將所有錯誤響應轉發到客戶端。 <br/>**1** - Dispatcher不會假離線對客戶端（其中狀態代碼大於或等於400）的錯誤響應，而是將狀態代碼傳遞給Apache，例如，ErrorDocument指令允許處理此類狀態代碼。 <br/>**代碼範圍**  — 指定將響應傳遞給Apache的錯誤代碼範圍。 其他錯誤代碼會傳遞給客戶端。 例如，以下配置將錯誤412的響應傳遞給客戶端，而所有其他錯誤則傳遞給Apache:DispatcherPassError 400-411,413-417 |
| DispatcherKeepAliveTimeout | 指定保持活動超時（秒）。 從Dispatcher 4.2.0版開始，預設的keep-alive值為60。 值0將禁用保持活動。 |
| DispatcherNoCanonURL | 將此參數設定為On將將原始URL傳遞給後端，而不是標準化的URL，並將覆蓋DispatcherUseProcessedURL的設定。 預設值為Off。 <br/>**注釋**:Dispatcher配置中的篩選器規則將始終根據經過清理的URL而不是原始URL進行評估。 |




>[!NOTE]
>
>路徑項相對於Apache Web Server的根目錄。

>[!NOTE]
>
>伺服器標頭的預設設定為：
>
>`ServerTokens Full`
>
>`DispatcherNoServerHeader 0`
>
>其中顯AEM示版本（用於統計目的）。 如果要禁用標題中可用的此類資訊，可以設定：
>
>`ServerTokens Prod`
>
>查看 [關於ServerTokens指令（例如，對於Apache 2.4）的Apache文檔](https://httpd.apache.org/docs/2.4/mod/core.html) 的子菜單。

**集處理程式**

在這些條目之後，您必須添加 **集處理程式** 語句到配置上下文( `<Directory>`。 `<Location>`)，以便Dispatcher處理傳入的請求。 以下示例將Dispatcher配置為處理完整網站的請求：

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

以下示例將Dispatcher配置為處理虛擬域的請求：

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
>的參數 **集處理程式** 必須寫語句 *正如上例中*，因為這是模組中定義的處理程式的名稱。
>
>有關此命令的完整詳細資訊，請參閱提供的配置檔案示例和Apache Web Server文檔。

**ModMimeUsePathInfo**

在 **集處理程式** 還應添加的語句 **ModMimeUsePathInfo** 定義。

>[!NOTE]
>
>的 `ModMimeUsePathInfo` 只有在使用Dispatcher版本4.0.9或更高版本時才應使用和配置參數。
>
>(請注意， Dispatcher 4.0.9版已於2011年發佈。 如果您使用的是較舊版本，則升級到最新的Dispatcher版本是合適的)。

的 **ModMimeUsePathInfo** 應設定參數 `On` 對於所有Apache配置：

`ModMimeUsePathInfo On`

mod_mime模組(請參見， [Apache模組mod_mime](https://httpd.apache.org/docs/2.4/mod/mod_mime.html))用於將內容元資料分配給為HTTP響應選擇的內容。 預設設定意味著當mod_mime確定內容類型時，將只考慮映射到檔案或目錄的URL的一部分。

當 `On`，也請參見Wiki頁。 `ModMimeUsePathInfo` 參數指定 `mod_mime` 是根據 *完整* URL;這意味著虛擬資源將根據其擴展應用元資訊。

以下示例激活 **ModMimeUsePathInfo**:

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

### 啟用對HTTPS（Unix和Linux）的支援 {#enable-support-for-https-unix-and-linux}

Dispatcher使用OpenSSL通過HTTP實現安全通信。 從Dispatcher版本啟動 **4.2.0**，支援OpenSSL 1.0.0和OpenSSL 1.0.1。 預設情況下，Dispatcher使用OpenSSL 1.0.0。 要使用OpenSSL 1.0.1，請使用以下過程建立符號連結，以便Dispatcher使用已安裝的OpenSSL庫。

1. 開啟終端並將當前目錄更改為安裝OpenSSL庫的目錄，例如：

   ```shell
   cd /usr/lib64
   ```

1. 要建立符號連結，請輸入以下命令：

   ```shell
   ln -s libssl.so libssl.so.1.0.1
   ln -s libcrypto.so libcrypto.so.1.0.1
   ```

>[!NOTE]
>
>如果您使用的是Apache的自定義版本，請確保Apache和Dispatcher使用的版本相同 [OpenSSL](https://www.openssl.org/source/)。

### 後續步驟 {#next-steps-1}

開始使用Dispatcher之前，您必須立即：

* [配置](dispatcher-configuration.md) 調度程式
* [配置AEM](page-invalidate.md) 與Dispatcher合作。

## Sun Java System Web Server / iPlanet {#sun-java-system-web-server-iplanet}

>[!NOTE]
>
>此處介紹了Windows和Unix環境的說明。
>
>在選擇要執行的時候請小心。

### Sun Java System Web Server / iPlanet — 安裝Web伺服器 {#sun-java-system-web-server-iplanet-installing-your-web-server}

有關如何安裝這些Web伺服器的完整資訊，請參閱其各自的文檔：

* Sun Java System Web伺服器
* iPlanet Web伺服器

### Sun Java System Web Server / iPlanet — 添加Dispatcher模組 {#sun-java-system-web-server-iplanet-add-the-dispatcher-module}

Dispatcher的格式為：

* **窗口**:動態連結庫(DLL)
* **Unix**:動態共用對象(DSO)

安裝存檔檔案包含以下檔案 — 取決於您是選擇了Windows還是Unix:

| 檔案 | 說明 |
|---|---|
| `disp_ns.dll` | 窗口：Dispatcher動態連結庫檔案。 |
| `dispatcher.so` | Unix:Dispatcher共用對象庫檔案。 |
| `dispatcher.so` | Unix:示例連結。 |
| `obj.conf.disp` | iPlanet / Sun Java System Web伺服器的示例配置檔案。 |
| `dispatcher.any` | Dispatcher的示例配置檔案。 |
| 自述檔案 | 包含安裝說明和最後一分鐘資訊的自述檔案。 注：請在開始安裝之前檢查此檔案。 |
| 更改 | 更改列出當前版本和過去版本中已修復問題的檔案。 |

使用以下步驟將Dispatcher添加到Web伺服器：

1. 將Dispatcher檔案放在Web伺服器 `plugin` 目錄：

### Sun Java System Web Server / iPlanet — 為Dispatcher配置 {#sun-java-system-web-server-iplanet-configure-for-the-dispatcher}

需要配置Web伺服器，使用 `obj.conf`。 在Dispatcher安裝工具包中，您將找到一個名為 `obj.conf.disp`。

1. 導航到 `<WEBSERVER_ROOT>/config`.
1. 開啟 `obj.conf`的子菜單。
1. 複製開始的行：\
   `Service fn="dispService"`\
   從 `obj.conf.disp` 到的初始化部分 `obj.conf`。

1. 儲存變更。
1. 開啟 `magnus.conf` 的子菜單。
1. 複製開始的兩行：\
   `Init funcs="dispService, dispInit"`\
   和\
   `Init fn="dispInit"`\
   從 `obj.conf.disp` 到的初始化部分 `magnus.conf`。

1. 儲存變更。

>[!NOTE]
>
>以下配置應全部在一行上，並且 `$(SERVER_ROOT)` 和 `$(PRODUCT_SUBDIR)` 必須用相應的值替換。

**初始化**

下表列出了可使用的示例；具體條目取決於您的特定web伺服器：

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
| 配置 | 配置檔案的位置和名稱 `dispatcher.any.` |
| 日誌檔案 | 日誌檔案的位置和名稱。 |
| 日誌 | 將消息寫入日誌檔案時的日誌級別： <br/>**0** 錯誤 <br/>**1** 警告 <br/>**2** 資訊 <br/>**3** 調試 <br/>**注：** 建議在安裝和測試期間將日誌級別設定為3，在生產環境中運行時將日誌級別設定為0。 |
| keepalitive超時 | 指定保持活動超時（秒）。 從Dispatcher 4.2.0版開始，預設的keep-alive值為60。 值0將禁用保持活動。 |

根據您的要求，您可以將Dispatcher定義為對象的服務。 要為整個網站配置Dispatcher，請修改預設對象：


**窗口**

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

開始使用Dispatcher之前，您必須立即：

* [配置](dispatcher-configuration.md) 調度程式
* [配置AEM](page-invalidate.md) 與Dispatcher合作。
