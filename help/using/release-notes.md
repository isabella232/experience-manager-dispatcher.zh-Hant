---
title: AEM Dispatcher 發行說明
seo-title: AEM Dispatcher 發行說明
description: Adobe Experience Manager Dispatcher的發行說明
seo-description: Adobe Experience Manager Dispatcher的發行說明
uuid: ae3ccf62-0514-4c03-a3b9-71799a482cbd
topic-tags: 發行說明
content-type: 引用
products: SG_EXPERIENCEMANAGER/6.4
discoiquuid: ff3d38e0-71c9-4b41-85f9-fa896393aac5
translation-type: tm+mt
source-git-commit: 6318765278bdce57a0576531944453a2b17fe2ae

---


# AEM Dispatcher 發行說明{#aem-dispatcher-release-notes}

## 發行資訊 {#release-information}

|  |  |
|--- |--- |
| 產品 | Adobe Experience Manager(AEM)Dispatcher |
| 版本 | 4.3.3 |
| 類型 | 次要版本 |
| 日期 | 2019年10月18日 |
| 下載URL | <ul><li>[Apache 2.4](release-notes.md#apache)</li><li>[Microsoft Internet Information Services(IIS)](release-notes.md#iis)</li></ul> |
| 相容性 | AEM 6.1或更新版本 |

## 系統需求和先決條件 {#system-requirements-and-prerequisites}

如需有關需求和 [先決條件的詳細資訊](https://helpx.adobe.com/experience-manager/6-4/sites/deploying/using/technical-requirements.html) ，請參閱支援的平台頁面。

Adobe強烈建議使用最新版本的AEM Dispatcher，以取得最新的功能、最新的錯誤修正以及最佳的效能。

## 安裝指示 {#installation-instructions}

有關詳細說明，請參 [閱安裝Dispatcher](dispatcher-install.md)。

## 發行記錄 {#release-history}

### 版本4.3.3（2019年10月18日） {#october}

**錯誤修正**:

* DISP-739 - logLevel dispatcher:級 **別不** 起作用
* DISP-749 - Alpine linux調度器當機並具有跟蹤日誌級別

**改進**:

* DISP-813 - Dispatcher中對openssl 1.1.x的支援
* DISP-814 —— 快取刷新期間發生Apache 40x錯誤
* DISP-818 - mod_expires新增快取控制標題，以取得不可取的內容
* DISP-821 —— 請勿將記錄內容儲存在通訊端中
* DISP-822 - Dispatcher應使用ppoll而非pselect
* DISP-824 - Secure DispatcherUseForwardedHost
* DISP-825 —— 當磁碟上沒有空間時記錄特殊訊息
* DISP-826 —— 支援使用查詢字串重新擷取URI

**新功能**:

* DISP-703 —— 場特定快取命中率
* DISP-827 —— 用於測試的本地伺服器
* DISP-828 —— 為調度程式建立測試Docker映像

### 版本4.3.2（2019年1月31日） {#jan}

**錯誤修正**:

* DISP-734 —— 如果未設為處理常式，則Dispatcher會在insert_output_filter中造成當機
* DISP-735 - RE在Alpine linux上無法運作
* DISP-740 —— 在macOS Mojave中載入分派程式預設為停用
* DISP-742 —— 被阻止的請求可能會將資訊洩露給驗證檢查器受保護的資源

**改進**:

* DISP-746 - dispatcher.any中未標籤的字串應產生警告

**新功能**:

* DISP-747 —— 在Apache環境中提供請求資訊

### 版本4.3.1（2018年10月16日） {#oct}

**錯誤修正**:

* DISP-656 - Dispatcher服務錯誤的ETag標題
* DISP-694 —— 在保持活連接失效時抑制警告
* DISP-714 —— 以Cookie為基礎的工作階段管理無法在IIS中運作
* DISP-715 —— 轉譯Cookie的安全標幟
* DISP-720 —— 暫存檔案未關閉，可能導致耗盡（開啟的檔案太多）
* DISP-721 —— 當Apache正常重新啟動子項時，Dispatcher中斷poll()
* DISP-722 —— 以八進位模式0600建立快取檔案
* DISP-723 —— 當演算逾時設為0時，隱式10分鐘逾時（並重試）
* DISP-725 —— 字串後的尾隨字元會無訊息地轉換為未命名值
* DISP-726 —— 記錄沒有群與傳入主機實際相符時的警告
* DISP-727 - Dispatcher會檢查空快取檔案的要求內容長度
* DISP-730 - 404：嘗試在發送器上訪問標頭檔
* DISP-731 - Dispatcher易受日誌插入的攻擊
* DISP-732 - Dispatcher應移除URL中的連續「/」
* DISP-733 - Dispatcher應設定（計算）年齡標題

**改進**:

* DISP-656 - Dispatcher服務錯誤的ETag標題
* DISP-694 —— 在保持活連接失效時抑制警告
* DISP-715 —— 轉譯Cookie的安全標幟
* DISP-722 —— 以八進位模式0600建立快取檔案
* DISP-726 —— 記錄沒有群與傳入主機實際相符時的警告

### 版本4.3.0（2018年6月13日） {#jun}

**錯誤修正**:

* DISP-682 —— 數值記錄層級套用不正確
* DISP-685 - 32位Solaris SPARC二進位檔案對__divdi3的引用未定義
* DISP-688 - Dispatcher在404回應中未傳回「X-Cache-Info」標題
* DISP-690 —— 上次修改的標頭無法快取
* DISP-691 - w3wp.exe中的存取違規
* DISP-693 —— 需要更新Dispatcher下載頁上Solaris伺服器的體系結構詳細資訊
* DISP-695 - Dispatcher模組4.2.3中DispatcherLog級別的問題
* DISP-698 - Dispatcher TTL需要支援s-maxage和private指令
* DISP-700 —— 模組在Alpine linux上無法正常工作
* DISP-704 —— 包含%2b的瀏覽器請求會傳送至未編碼的發佈者
* DISP-705 —— 由於雙重釋放或損壞(fasttop)而導致的Dispatcher崩潰
* DISP-706 —— 在失效期間，調度程式正在跟蹤可導致無限回圈的反向參考符號連結
* DISP-709 —— 封鎖某些虛名URL擴充功能
* DISP-710 - Cent OS 6上無法使用的Linux版本

**改進**:

* DISP-652 - Dispatcher服務錯誤的日期標題

## 實用資源 {#helpful-resources}

* [AEM Dispatcher概觀](dispatcher.md)

## 下載 {#downloads}

### Apache 2.4 {#apache}

| 平台支援 | 建築 | SSL | 下載 |
|---|---|---|---|
| Linux | i686（32位元） | 無 | [dispatcher-apache2.4-linux-i686-4.3.3.tar.gz](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-i686-4.3.3.tar.gz) |
| Linux | i686（32位元） | 1.0 | [dispatcher-apache2.4-linux-i686-ssl1.0-4.3.3.tar.gz](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-i686-ssl1.0-4.3.3.tar.gz) |
| Linux | i686（32位元） | 1.1 | [dispatcher-apache2.4-linux-i686-ssl1.1-4.3.3.tar.gz](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-i686-ssl1.1-4.3.3.tar.gz) |
| Linux | x86_64（64位元） | 無 | [dispatcher-apache2.4-linux-x86_64-4.3.3.tar.gz](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-x86_64-4.3.3.tar.gz) |
| Linux | x86_64（64位元） | 1.0 | [dispatcher-apache2.4-linux-x86_64-ssl1.0-4.3.3.tar.gz](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-x86_64-ssl1.0-4.3.3.tar.gz) |
| Linux | x86_64（64位元） | 1.1 | [dispatcher-apache2.4-linux-x86_64-ssl1.1-4.3.3.tar.gz](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-x86_64-ssl1.1-4.3.3.tar.gz) |
| macOS | x86_64（64位元） | 無 | [dispatcher-apache2.4-darwin-x86_64-4.3.3.tar.gz](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-darwin-x86_64-4.3.3.tar.gz) |

### IIS {#iis}

| 平台支援 | 建築 | SSL | 下載 |
|---|---|---|---|
| Windows | x86（32位元） | 無 | [dispatcher-iis-windows-x86-4.3.3.zip](https://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x86-4.3.3.zip) |
| Windows | x86（32位元） | 1.0 | [dispatcher-iis-windows-x86-ssl1.0-4.3.3.zip](https://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x86-ssl1.0-4.3.3.zip) |
| Windows | x86（32位元） | 1.1 | [dispatcher-iis-windows-x86-ssl1.1-4.3.3.zip](https://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x86-ssl1.1-4.3.3.zip) |
| Windows | x64（64位元） | 無 | [dispatcher-iis-windows-x64-4.3.3.zip](https://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x64-4.3.3.zip) |
| Windows | x64（64位元） | 1.0 | [dispatcher-iis-windows-x64-ssl1.0-4.3.3.zip](https://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x64-ssl1.0-4.3.3.zip) |
| Windows | x64（64位元） | 1.1 | [dispatcher-iis-windows-x64-ssl1.1-4.3.3.zip](https://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x64-ssl1.1-4.3.3.zip) |
