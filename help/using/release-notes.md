---
title: AEM Dispatcher發行說明
seo-title: AEM Dispatcher發行說明
description: Adobe Experience Manager Dispatcher專用的發行說明
seo-description: Adobe Experience Manager Dispatcher專用的發行說明
uuid: ae3cf62-0514-4c03-a3 b9-71799a482 cbd
topic-tags: 版本注意事項
content-type: 引用
products: SG_ PERIENCENCENAGER/6.4
discoiquuid: ff3d38e0-71c9-4b41-85f9-fa896393 aac5
translation-type: tm+mt
source-git-commit: f35c79b487454059062aca6a7c989d5ab2afaf7b

---


# AEM Dispatcher發行說明{#aem-dispatcher-release-notes}

## 發行資訊 {#release-information}

|  |
|--- |--- |
| 產品 | Adobe Experience Manager(AEM) Dispatcher |
| 版本 | 4.3.2 |
| 類型 | 次要版本 |
| 日期 | 2019年月31日 |
| 下載URL | <ul><li>[Apache2.4](release-notes.md#apache)</li><li>[Microsoft Internet Information Services(IIS)](release-notes.md#iis)</li></ul> |
| 相容性 | AEM6.1或更新版本 |

## 系統需求與必要條件 {#system-requirements-and-prerequisites}

如需需求和必要條件的詳細資訊，請參閱 [「支援的平台](https://helpx.adobe.com/experience-manager/6-4/sites/deploying/using/technical-requirements.html) 」頁面。

Adobe強烈建議使用最新版本的AEM Dispatcher來檢查最新功能、最新錯誤修正以及最佳效能。

## 安裝指示 {#installation-instructions}

如需詳細指示，請參閱 [安裝Dispatcher](dispatcher-install.md)。

## 發行歷史記錄 {#release-history}

### 版本4.3.2(2019-Jan-31) {#jan}

**錯誤修正**：

* Disp-734-如果未設定為handler，Dispatcher會造成插入_ output_ filter損毀
* DISP-735- Rights無法在Alpine Linux上運作
* Disp-740-在macOS Mojave中載入dispatcher預設為停用
* Disp-742-封鎖的要求可能會洩露資訊至驗證檢查程式的資源

**改進**：

* Disp-746-傳送程式中不受影響的字串。任何應產生警告

**新功能**：

* Disp-747-在Apache環境中提供要求資訊

### 版本4.3.1(2018年10月16日) {#oct}

**錯誤修正**：

* Disp-656- Dispatcher提供錯誤的eTag標題
* Disp-694-在保持連線的狀態過時時抑制警告
* Disp-714-基於Cookie的作業管理無法在IIS中運作
* DisP-715- renderid Cookie的安全標幟
* Disp-720-暫時未關閉的暫存檔案可能導致整合性(太多開啓檔案)
* Disp-721- Dispatcher會中斷民調問答()當Apache正常重新啓動子項時
* Disp-722-快取檔案是以ocar模式0600建立
* Disp-723-將逾時設為0時，隱含10分鐘逾時(及重試)
* DISP-725-字串在無訊息轉換為未命名值後的尾隨字元
* Disp-726-當沒有農場實際符合傳入主機時記錄警告
* DISP-727- Dispatcher檢查空快取檔案的請求內容長度
* DISP-730-404嘗試存取header檔案over dispatcher
* DisP-731- Dispatcher很容易受到記錄植入攻擊
* Disp-732- Dispatcher應移除URL中的連續&#39;/&#39;
* Disp-733- Dispatcher應設定(計算)年齡表頭

**改進**：

* Disp-656- Dispatcher提供錯誤的eTag標題
* Disp-694-在保持連線的狀態過時時抑制警告
* DisP-715- renderid Cookie的安全標幟
* Disp-722-快取檔案是以ocar模式0600建立
* Disp-726-當沒有農場實際符合傳入主機時記錄警告

### 版本4.3.0(2018-月13日) {#jun}

**錯誤修正**：

* Disp-682-數值記錄層級未正確套用
* DisP-685-32位元Solaris SPARC二進位檔已定義為__ divdi3
* Disp-688- Dispatcher在404回應中不會傳回「X-Cache-Info」標題
* Disp-690-上次修改的標題不可用於
* DisP-691- w3wp. exe中的存取違規
* Disp-693-需要更新傳送程式下載頁面上Solaris伺服器的架構詳細資訊
* DisP-695- Dispatcher模組4.2.3中的DispatcherLog層級問題
* Disp-698- Dispatcher TTL需要支援s-maxage和私人指示
* DISP-700- Alpon Linux無法正確運作
* Disp-704-包含%2b的瀏覽器要求會傳送至重新編碼的發行者
* Disp-705- Dispatcher因雙重釋放或損毀造成當機(頂端)
* DisP-706-在失效期間，傳送程式會遵循反向連結的參考連結，可產生無限回圈
* DisP-709-封鎖一些虛名URL擴充功能
* Disp-710-適用於Linux的Builds不適用於Dent OS6

**改進**：

* DISP-652- Dispatcher提供錯誤的日期標題

## 實用資源 {#helpful-resources}

* [AEM Dispatcher概述](dispatcher.md)

## 下載 {#downloads}

### Apache2.4 {#apache}

| 平台 | 架構 | SSL支援 | 下載 |
|---|---|---|---|
| AIX | PowerPC(32位元) | 否 | [dispatcher-apache2.4-aix-powerpc-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-aix-powerpc-4.3.2.tar.gz) |
| AIX | PowerPC(32位元) | 是 | [dispatcher-apache2.4-aix-powerpc-ssl-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-aix-powerpc-ssl-4.3.2.tar.gz) |
| AIX | PowerPC(64位元) | 否 | [dispatcher-apache2.4-aix-powerpc64-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-aix-powerpc64-4.3.2.tar.gz) |
| AIX | PowerPC(64位元) | 是 | [dispatcher-apache2.4-aix-powerpc64-ssl-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-aix-powerpc64-ssl-4.3.2.tar.gz) |
| Linux | i686(32位元) | 否 | [dispatcher-apache2.4-linux-i686-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-i686-4.3.2.tar.gz) |
| Linux | i686(32位元) | 是 | [dispatcher-apache2.4-linux-i686-ssl-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-i686-ssl-4.3.2.tar.gz) |
| Linux | x86_64(64位元) | 否 | [dispatcher-apache2.4-linux-x86_64-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-x86_64-4.3.2.tar.gz) |
| Linux | x86_64(64位元) | 是 | [dispatcher-apache2.4-linux-x86_64-ssl-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-linux-x86_64-ssl-4.3.2.tar.gz) |
| macOS作業系統 | x86_64(64位元) | 否 | [dispatcher-apache2.4-darwin-x86_64-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-darwin-x86_64-4.3.2.tar.gz) |
| Solaris | AMD(32位元) | 否 | [dispatcher-apache2.4-solaris-i386-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-solaris-i386-4.3.2.tar.gz) |
| Solaris | AMD(32位元) | 是 | [dispatcher-apache2.4-solaris-i386-ssl-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-solaris-i386-ssl-4.3.2.tar.gz) |
| Solaris | AMD(64位元) | 否 | [dispatcher-apache2.4-solaris-amd64-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-solaris-amd64-4.3.2.tar.gz) |
| Solaris | AMD(64位元) | 是 | [dispatcher-apache2.4-solaris-amd64-ssl-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-solaris-amd64-ssl-4.3.2.tar.gz) |
| Solaris | SPARC(32位元) | 否 | [dispatcher-apache2.4-solaris-sparc-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-solaris-sparc-4.3.2.tar.gz) |
| Solaris | SPARC(32位元) | 是 | [dispatcher-apache2.4-solaris-sparc-ssl-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-solaris-sparc-ssl-4.3.2.tar.gz) |
| Solaris | SPARC(64位元) | 否 | [dispatcher-apache2.4-solaris-sparcv9-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-solaris-sparcv9-4.3.2.tar.gz) |
| Solaris | SPARC(64位元) | 是 | [dispatcher-apache2.4-solaris-sparcv9-ssl-4.3.2.tar.gz](http://download.macromedia.com/dispatcher/download/dispatcher-apache2.4-solaris-sparcv9-ssl-4.3.2.tar.gz) |

### IIS {#iis}

| 平台 | 架構 | SSL支援 | 下載 |
|---|---|---|---|
| Windows | x86(32位元) | 否 | [dispatcher-iis-windows-x86-4.3.2.zip](http://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x86-4.3.2.zip) |
| Windows | x86(32位元) | 是 | [dispatcher-iis-windows-x86-ssl-4.3.2.zip](http://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x86-ssl-4.3.2.zip) |
| Windows | x64(64位元) | 否 | [dispatcher-iis-windows-x64-4.3.2.zip](http://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x64-4.3.2.zip) |
| Windows | x64(64位元) | 是 | [dispatcher-iis-windows-x64-ssl-4.3.2.zip](http://download.macromedia.com/dispatcher/download/dispatcher-iis-windows-x64-ssl-4.3.2.zip) |
