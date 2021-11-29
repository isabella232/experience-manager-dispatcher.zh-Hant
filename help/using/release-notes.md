---
title: AEM Dispatcher 發行說明
seo-title: AEM Dispatcher Release Notes
description: Adobe Experience Manager Dispatcher專屬的發行說明
seo-description: Release notes specific to Adobe Experience Manager Dispatcher
uuid: ae3ccf62-0514-4c03-a3b9-71799a482cbd
topic-tags: release-notes
content-type: reference
products: SG_EXPERIENCEMANAGER/6.4
discoiquuid: ff3d38e0-71c9-4b41-85f9-fa896393aac5
exl-id: b55c7a34-d57b-4d45-bd83-29890f1524de
source-git-commit: 4f37bc2571c3272beeb1764ca0bf0347e086cc07
workflow-type: tm+mt
source-wordcount: '852'
ht-degree: 8%

---

# AEM Dispatcher 發行說明{#aem-dispatcher-release-notes}

## 發行資訊 {#release-information}

|  |  |
|--- |--- |
| 產品 | Adobe Experience Manager(AEM)Dispatcher |
| 版本 | 4.3.4 |
| 類型 | 次要版本 |
| 日期 | 2021 年 11 月 29 日 |
| 下載URL | <ul><li>[Apache 2.4](release-notes.md#apache)</li><li>[Microsoft Internet Information Services(IIS)](release-notes.md#iis)</li></ul> |
| 相容性 | AEM 6.1或更高版本 |

## 系統需求和先決條件 {#system-requirements-and-prerequisites}

請參閱 [支援平台](https://helpx.adobe.com/experience-manager/6-4/sites/deploying/using/technical-requirements.html) 頁面，以取得需求和必要條件的詳細資訊。

Adobe強烈建議使用最新版本的AEM Dispatcher，以取得最新功能、最新的錯誤修正，以及最佳的效能。

## 安裝指示 {#installation-instructions}

如需詳細指示，請參閱 [安裝Dispatcher](dispatcher-install.md).

## 發行記錄 {#release-history}

### 版本4.3.4（2021年11月29日） {#nov}

**錯誤修正**:

* DISP-833 - X-Forwarded-Host標題可包含逗號分隔主機名稱的清單
* DISP-835 - DispatcherUseForwardedHost可以吞咽主機標頭（如果最後）


**改善功能**:

* DISP-841 - Dispatcher對504回應代碼不遵守/serverStaleOnError
* DISP-874 — 建立Dispatcher設定，以開啟或關閉DISP-818的實作
* DISP-883 — 在Dispatcher中顯示URL請求分解的追蹤
* DISP-944 — 預先載入虛名url

### 版本4.3.3（2019年10月18日） {#october}

**錯誤修正**:

* DISP-739 - LogLevel Dispatcher: **層級** 不管用
* DISP-749 - Alpine Linux Dispatcher因追蹤記錄層級而當機

**改善功能**:

* DISP-813 - Dispatcher對openssl 1.1.x的支援
* DISP-814 — 快取刷新期間出現Apache 40x錯誤
* DISP-818 - mod_expires為不可執行的內容新增快取控制標題
* DISP-821 — 請勿在通訊端中儲存記錄內容
* DISP-822 - Dispatcher應使用民調問答，而非pselect
* DISP-824 — 安全DispatcherUseForwardedHost
* DISP-825 — 在磁碟上沒有更多空間時記錄特殊消息
* DISP-826 — 支援使用查詢字串重新擷取URI

**新功能**:

* DISP-703 — 伺服器陣列特定快取命中率
* DISP-827 — 用於測試的本地伺服器
* DISP-828 — 為Dispatcher建立測試Docker影像

### 版本4.3.2（2019年1月31日） {#jan}

**錯誤修正**:

* DISP-734 — 若未設為處理常式，Dispatcher會在insert_output_filter中造成當機
* DISP-735 - RE在Alpine Linux上無法運作
* DISP-740 — 在macOS Mojave中載入Dispatcher預設為停用
* DISP-742 — 被阻止的請求可能會將資訊洩露給驗證檢查器受保護的資源

**改善功能**:

* DISP-746 - dispatcher.any中未標示的字串應產生警告

**新功能**:

* DISP-747 — 在Apache環境中提供請求資訊

### 版本4.3.1（2018年10月16日） {#oct}

**錯誤修正**:

* DISP-656 - Dispatcher提供錯誤的ETag標題
* DISP-694 — 當保存連接失效時禁止警告
* DISP-714 - IIS中無法使用Cookie作業管理
* DISP-715 — 轉譯Cookie的安全標幟
* DISP-720 — 暫時檔案未關閉，可能導致用盡（開啟的檔案過多）
* DISP-721 — 當Apache正常重新啟動子項時，Dispatcher中斷輪詢()
* DISP-722 — 使用八進位模式0600建立快取檔案
* DISP-723 — 呈現逾時設為0時，隱式10分鐘逾時（並重試）
* DISP-725 — 字串後的尾隨字元會自動轉換為未命名的值
* DISP-726 — 當沒有伺服器陣列實際符合傳入主機時記錄警告
* DISP-727 - Dispatcher會檢查空快取檔案的要求內容長度
* 嘗試透過Dispatcher存取標題檔案時，DISP-730 - 404
* DISP-731 - Dispatcher容易遭受記錄插入攻擊
* DISP-732 - Dispatcher應在URL中移除連續的「/」
* DISP-733 - Dispatcher應設定（計算）年齡標題

**改善功能**:

* DISP-656 - Dispatcher提供錯誤的ETag標題
* DISP-694 — 當保存連接失效時禁止警告
* DISP-715 — 轉譯Cookie的安全標幟
* DISP-722 — 使用八進位模式0600建立快取檔案
* DISP-726 — 當沒有伺服器陣列實際符合傳入主機時記錄警告

### 版本4.3.0（2018年6月13日） {#jun}

**錯誤修正**:

* DISP-682 — 未正確套用數值記錄層級
* DISP-685 - 32位Solaris SPARC二進位檔案對__divi3有未定義的引用
* DISP-688 - Dispatcher在404回應時未傳回「X-Cache-Info」標題
* DISP-690 — 上次修改的標頭無法快取
* DISP-691 - w3wp.exe中的訪問違規
* DISP-693 — 需要在dispatcher下載頁上更新Solaris伺服器的體系結構詳細資訊
* DISP-695 - Dispatcher模組4.2.3中的DispatcherLog層級問題
* DISP-698 - Dispatcher TTL需要支援s-maxage和私人指示
* DISP-700 — 模組在Alpine Linux上無法正常運作
* DISP-704 — 將包含%2b的瀏覽器請求發送到未編碼的發佈器
* DISP-705 — 由於雙重自由或損毀（快速頂端）而導致Dispatcher當機
* DISP-706 — 在失效期間，Dispatcher會追隨回參考符號連結，而這會造成無限回圈
* DISP-709 — 封鎖某些虛名URL擴充功能
* DISP-710 - Cent OS 6上無法使用的Linux版本編號

**改善功能**:

* DISP-652 - Dispatcher提供錯誤的日期標題

## 實用資源 {#helpful-resources}

* [AEM Dispatcher綜覽](dispatcher.md)

## 下載內容 {#downloads}

### Apache 2.4 {#apache}

| 平台 | 架構 | OpenSSL支援 | 下載 |
|---|---|---|---|
| Linux | i686（32位） | 無 | [dispatcher-apache2.4-linux-i686-4.3.4.tar.gz](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-i686-4.3.4.tar.gz) |
| Linux | i686（32位） | 1.0 | [dispatcher-apache2.4-linux-i686-ssl1.0-4.3.4.tar.gz](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-i686-ssl1.0-4.3.4.tar.gz) |
| Linux | i686（32位） | 1.1 | [dispatcher-apache2.4-linux-i686-ssl1.1-4.3.4.tar.gz](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-i686-ssl1.1-4.3.4.tar.gz) |
| Linux | x86_64（64位） | 無 | [dispatcher-apache2.4-linux-x86_64-4.3.4.tar.gz](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-x86_64-4.3.4.tar.gz) |
| Linux | x86_64（64位） | 1.0 | [dispatcher-apache2.4-linux-x86_64-ssl1.0-4.3.4.tar.gz](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-x86_64-ssl1.0-4.3.4.tar.gz) |
| Linux | x86_64（64位） | 1.1 | [dispatcher-apache2.4-linux-x86_64-ssl1.1-4.3.4.tar.gz](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-x86_64-ssl1.1-4.3.4.tar.gz) |
| macOS | x86_64（64位） | 無 | [dispatcher-apache2.4-darwin-x86_64-4.3.4.tar.gz](https://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-darwin-x86_64-4.3.4.tar.gz) |

### IIS {#iis}

| 平台 | 架構 | OpenSSL支援 | 下載 |
|---|---|---|---|
| Windows | x86（32位） | 無 | [dispatcher-iis-windows-x86-4.3.4.zip](https://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x86-4.3.4.zip) |
| Windows | x86（32位） | 1.0 | [dispatcher-iis-windows-x86-ssl1.0-4.3.4.zip](https://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x86-ssl1.0-4.3.4.zip) |
| Windows | x86（32位） | 1.1 | [dispatcher-iis-windows-x86-ssl1.1-4.3.4.zip](https://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x86-ssl1.1-4.3.4.zip) |
| Windows | x64（64位） | 無 | [dispatcher-iis-windows-x64-4.3.4.zip](https://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x64-4.3.4.zip) |
| Windows | x64（64位） | 1.0 | [dispatcher-iis-windows-x64-ssl1.0-4.3.4.zip](https://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x64-ssl1.0-4.3.4.zip) |
| Windows | x64（64位） | 1.1 | [dispatcher-iis-windows-x64-ssl1.1-4.3.4.zip](https://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x64-ssl1.1-4.3.4.zip) |
