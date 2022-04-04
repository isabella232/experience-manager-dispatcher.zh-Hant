---
title: AEM Dispatcher 發行說明
seo-title: AEM Dispatcher Release Notes
description: 特定於Adobe Experience Manager調度程式的發行說明
seo-description: Release notes specific to Adobe Experience Manager Dispatcher
uuid: ae3ccf62-0514-4c03-a3b9-71799a482cbd
topic-tags: release-notes
content-type: reference
products: SG_EXPERIENCEMANAGER/6.4
discoiquuid: ff3d38e0-71c9-4b41-85f9-fa896393aac5
exl-id: b55c7a34-d57b-4d45-bd83-29890f1524de
source-git-commit: 69aa52221e997e0b68c89bf383d36fdb7093ac4d
workflow-type: tm+mt
source-wordcount: '977'
ht-degree: 7%

---

# AEM Dispatcher 發行說明{#aem-dispatcher-release-notes}

## 發行資訊 {#release-information}

|  |  |
|--- |--- |
| 產品 | Adobe Experience Manager(AEM)調度員 |
| 版本 | 4.3.5 |
| 類型 | 次要版本 |
| 日期 | 2022 年 4 月 04 日 |
| 下載URL | <ul><li>[Apache 2.4](release-notes.md#apache)</li><li>[Microsoft網際網路資訊服務(IIS)](release-notes.md#iis)</li></ul> |
| 相容性 | AEM 6.1或更高版本 |

## 系統需求和先決條件 {#system-requirements-and-prerequisites}

請參閱 [支援的平台](https://helpx.adobe.com/experience-manager/6-4/sites/deploying/using/technical-requirements.html)的子菜單。

Adobe強烈建議使用最新AEM版本的Dispatcher從最新的功能、最新的錯誤修復和最佳效能中獲益。

## 安裝指示 {#installation-instructions}

有關詳細說明，請參見 [正在安裝Dispatcher](dispatcher-install.md)。

## 發佈歷史記錄 {#release-history}

### 4.3.5版（2022年4月29日） {#apr}

**改善功能**:

* DISP-954 — 支援失效，即使到期未過
* DISP-949 - Dispatcher返回200而不是404，即使POST請求被篩選器阻止

### 4.3.4版(2021-11-29) {#nov}

**錯誤修正**:

* DISP-833 - X-Forwarded-Host標頭可能包含逗號分隔的主機名清單
* DISP-835 - DispatcherUseForwardedHost吞咽主機標頭（如果該標頭是最後一個）

**改善功能**:

* DISP-874 — 建立調度程式配置以通過標誌開啟或關閉DISP-818的實施 `DispatcherRestrictUncacheableContent`。 預設值為Off。 當開啟時，它將刪除由mod設定的快取標頭，這些標頭將過期，以用於不可取的內容。 這與4.3.3版（預設值為On）中的行為不同，但與4.3.3之前的版本（預設值為Off）相同。 保留 `DispatcherRestrictUncacheableContent`預設關閉是建議的方法，因此瀏覽器快取具有更大的靈活性。 如果在從4.3.3版升級到4.3.4版時希望保留與4.3.3版中相同的行為，則必須顯式設定 `DispatcherRestrictUncacheableContent` 開啟。
* DISP-841 - Dispatcher對504響應代碼不尊重/serverStaleOnError
* DISP-874 — 建立調度程式配置以開啟或關閉DISP-818的實施
* DISP-883 — 顯示Dispatcher中URL請求分解的跟蹤
* DISP-944 — 預載入虛榮區

### 4.3.3版（2019年–10月18日） {#october}

**錯誤修正**:

* DISP-739 - LogLevel調度程式： **級別** 無效
* DISP-749 - Alpine Linux調度程式崩潰，跟蹤日誌級別

**改善功能**:

* DISP-813 - Dispatcher中對openssl 1.1.x的支援
* DISP-814 — 快取刷新期間出現Apache 40x錯誤
* DISP-818 - mod_expires為不可訪問內容添加Cache-Control標頭
* DISP-821 — 不在套接字中儲存日誌上下文
* DISP-822 - Dispatcher應使用ppoll而不是pselect
* DISP-824 — 安全DispatcherUseForwardedHost
* DISP-825 — 當磁碟上沒有更多空間時記錄特殊消息
* DISP-826 — 支援使用查詢字串重取URI

**新功能**:

* DISP-703 — 場特定快取命中率
* DISP-827 — 用於測試的本地伺服器
* DISP-828 — 為調度程式建立測試Docker映像

### 4.3.2版（2019年1月31日） {#jan}

**錯誤修正**:

* DISP-734 — 如果未設定為處理程式，則Dispatcher會導致insert_output_filter崩潰
* DISP-735 - RE在Alpine Linux上不工作
* DISP-740 — 預設情況下，在macOS莫哈韋載入調度程式已禁用
* DISP-742 — 被阻止的請求可能會將資訊洩露給身份驗證檢查器受保護的資源

**改善功能**:

* DISP-746 - Dispatcher.any中未標籤的字串應生成警告

**新功能**:

* DISP-747 — 在Apache環境中提供請求資訊

### 4.3.1版（2018年–10月16日） {#oct}

**錯誤修正**:

* DISP-656 - Dispatcher服務錯誤的ETag標頭
* DISP-694 — 保持活動連接失效時取消警告
* DISP-714 — 基於Cookie的會話管理在IIS中不工作
* DISP-715 - Renderid Cookie的安全標誌
* DISP-720 — 未關閉臨時檔案，可能導致耗盡（開啟的檔案太多）
* DISP-721 — 當Apache正常重新啟動子代時，Dispatcher中斷輪詢()
* DISP-722 — 使用八進位模式0600建立快取檔案
* DISP-723 — 當呈現超時設定為0時隱式10分鐘超時（並重試）
* DISP-725 — 字串後的尾隨字元被靜默轉換為未命名值
* DISP-726 — 當沒有伺服器場實際與傳入主機匹配時記錄警告
* DISP-727 - Dispatcher檢查空快取檔案的請求內容長度
* DISP-730 — 嘗試通過調度程式訪問標頭檔時為404
* DISP-731 - Dispatcher易受日誌注入的攻擊
* DISP-732 - Dispatcher應刪除URL中的連續「/」
* DISP-733 - Dispatcher應設定（計算）年齡報頭

**改善功能**:

* DISP-656 - Dispatcher服務錯誤的ETag標頭
* DISP-694 — 保持活動連接失效時取消警告
* DISP-715 - Renderid Cookie的安全標誌
* DISP-722 — 使用八進位模式0600建立快取檔案
* DISP-726 — 當沒有伺服器場實際與傳入主機匹配時記錄警告

### 4.3.0版（2018年6月13日） {#jun}

**錯誤修正**:

* DISP-682 — 未正確應用數字日誌級別
* DISP-685 - 32位Solaris SPARC二進位檔案對__divdi3有未定義的引用
* DISP-688 - Dispatcher在404響應中不返回「X-Cache-Info」標頭
* DISP-690 — 上次修改的報頭不可快取
* DISP-691 - w3wp.exe中的訪問違規
* DISP-693 — 需要更新Dispatcher下載頁上Solaris伺服器的體系結構詳細資訊
* DISP-695 - Dispatcher模組4.2.3中DispatcherLog級別問題
* DISP-698 - Dispatcher TTL需要支援s-maxage和private指令
* DISP-700 - Linux上的模組無法正常工作
* DISP-704 — 將包含%2b的瀏覽器請求發送到未編碼的發佈伺服器
* DISP-705 — 由於雙重空閒或損壞(fasttop)而導致的Dispatcher崩潰
* DISP-706 — 在失效期間，調度程式正在跟蹤可導致無限循環的回參考符號連結
* DISP-709 — 阻止某些虛擬URL擴展
* DISP-710 - Cent OS 6上不可用的Linux版本

**改善功能**:

* DISP-652 - Dispatcher提供了錯誤的日期標題

## 實用資源 {#helpful-resources}

* [Dispatcher概AEM述](dispatcher.md)

## 下載內容 {#downloads}

### Apache 2.4 {#apache}

| 平台 | 架構 | OpenSSL支援 | 下載 |
|---|---|---|---|
| Linux | i686（32位） | 無 | [dispatcher-apache2.4-linux-i686-4.3.5.tar.gz](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-i686-4.3.5.tar.gz) |
| Linux | i686（32位） | 1.0 | [dispatcher-apache2.4-linux-i686-ssl1.0-4.3.5.tar.gz](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-i686-ssl1.0-4.3.5.tar.gz) |
| Linux | i686（32位） | 1.1 | [dispatcher-apache2.4-linux-i686-ssl1.1-4.3.5.tar.gz](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-i686-ssl1.1-4.3.5.tar.gz) |
| Linux | x86_64（64位） | 無 | [dispatcher-apache2.4-linux-x86_64-4.3.5.tar.gz](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-x86_64-4.3.5.tar.gz) |
| Linux | x86_64（64位） | 1.0 | [dispatcher-apache2.4-linux-x86_64-ssl1.0-4.3.5.tar.gz](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-x86_64-ssl1.0-4.3.5.tar.gz) |
| Linux | x86_64（64位） | 1.1 | [dispatcher-apache2.4-linux-x86_64-ssl1.1-4.3.5.tar.gz](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-x86_64-ssl1.1-4.3.5.tar.gz) |
| macOS | x86_64（64位） | 無 | [dispatcher-apache2.4-darwin-x86_64-4.3.5.tar.gz](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-darwin-x86_64-4.3.5.tar.gz) |

### IIS {#iis}

| 平台 | 架構 | OpenSSL支援 | 下載 |
|---|---|---|---|
| Windows | x86（32位） | 無 | [dispatcher-iis-windows-x86-4.3.5.zip](https://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x86-4.3.5.zip) |
| 窗口 | x86（32位） | 1.0 | [dispatcher-iis-windows-x86-ssl1.0-4.3.5.zip](https://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x86-ssl1.0-4.3.5.zip) |
| 窗口 | x86（32位） | 1.1 | [dispatcher-iis-windows-x86-ssl1.1-4.3.5.zip](https://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x86-ssl1.1-4.3.5.zip) |
| 窗口 | x64（64位） | 無 | [dispatcher-iis-windows-x64-4.3.5.zip](https://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x64-4.3.5.zip) |
| 窗口 | x64（64位） | 1.0 | [dispatcher-iis-windows-x64-ssl1.0-4.3.5.zip](https://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x64-ssl1.0-4.3.5.zip) |
| 窗口 | x64（64位） | 1.1 | [dispatcher-iis-windows-x64-ssl1.1-4.3.5.zip](https://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x64-ssl1.1-4.3.5.zip) |
