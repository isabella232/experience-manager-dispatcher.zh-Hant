---
title: 疑難排解Dispatcher問題疑難排解
seo-title: 疑難排解AEM Dispatcher問題
description: 瞭解如何疑難排解Dispatcher問題。
seo-description: 瞭解如何疑難排解AEM Dispatcher問題。
uuid: 9c109a48-d921-4b6 e-9626-1158cbc41 e7
cmgrlastmodified: 01.11.2007082929[ahheimoz]
pageversionid: '1193211344162'
template: /apps/docs/templates/contentpage
contentOwner: 使用者
products: SG_ PERIENCENCENAGER/ADDER
topic-tags: dispatcher
content-type: 引用
discoiquuid: a612e745-f1 e6-43de-b25 a-9adcaadab5 cf
translation-type: tm+mt
source-git-commit: f35c79b487454059062aca6a7c989d5ab2afaf7b

---


# 疑難排解Dispatcher問題疑難排解 {#troubleshooting-dispatcher-problems}

>[!NOTE]
>
>Dispatcher版本與AEM無關，不過Dispatcher文件內嵌於AEM文件中。請務必使用內嵌於最新版AEM文件的Dispatcher文件。
>
>如果您關注Dispatcher文件的連結(內嵌於舊版AEM的文件中)，可能會重新導向至此頁面。

>[!NOTE]
>
>另請檢查 [Dispatcher知識庫](https://helpx.adobe.com/cq/kb/index/dispatcher.html)、 [疑難排解Dispatcher Flooning Issues](https://helpx.adobe.com/adobe-cq/kb/troubleshooting-dispatcher-flushing-issues.html) 和 [Dispatcher熱門問題常見問答集，](dispatcher-faq.md) 以取得更多資訊。

## 檢查基本組態 {#check-the-basic-configuration}

第一步是檢查基本概念：

* [確認基本操作](#ConfirmBasicOperation)
* 檢查您的網站伺服器和調度器的所有記錄檔。如有必要，請增加 `loglevel` dispatcher [記錄](#Logging)檔。

* [檢查您的組態](#ConfiguringtheDispatcher)：

   * 您有多個Dispatchers嗎？

      * 您確定正在處理您正在調查的網站/頁面的Dispatcher嗎？
   * 您已實作篩選器嗎？

      * 這些會影響您調查的內容嗎？


## IIS診斷工具 {#iis-diagnostic-tools}

IIS提供多種追蹤工具，視實際版本而定：

* IIS-可下載和設定IIS診斷工具
* IIS-追蹤完全整合

這些功能可協助您監控活動。

## 找不到IIS和404 {#iis-and-not-found}

使用IIS時，您可能會在 `404 Not Found` 不同情況下傳回體驗。如果是，請參閱下列知識庫文章。

* [IIS6/7：POST方法傳回404](https://helpx.adobe.com/dispatcher/kb/IIS6IsapiFilters.html)
* [IIS6：包含基本路徑 `/bin` 傳回之URL的請求 `404 Not Found`](https://helpx.adobe.com/dispatcher/kb/RequestsToBinDirectoryFailInIIS6.html)

您也應該檢查dispatcher快取根目錄和IIS文件根目錄是否已設為相同目錄。

## 刪除工作流程模型時發生問題 {#problems-deleting-workflow-models}

**症狀**

透過Dispatcher存取AEM作者實例時，嘗試刪除工作流程模型時遇到問題。

**重制步驟：**

1. 登入您的作者實例(確認要透過發送器路由傳送請求)。
1. 建立新工作流程；例如，標題設為WorkflowToDelete。
1. 確認已成功建立工作流程。
1. 選取並按一下工作流程，然後按一下 **「刪除**」。

1. 按一下 **「是** 」以確認。
1. 出現錯誤訊息方塊：\
   &quot; `ERROR 'Could not delete workflow model!!`&quot;.

**解析度**

將下列標題新增至檔案 `/clientheaders` 的區段 `dispatcher.any` ：

* `x-http-method-override`
* `x-requested-with`

`{  
{  
/clientheaders  
{  
...  
"x-http-method-override"  
"x-requested-with"  
}`

## 干擾mod_ dir(Apache) {#interference-with-mod-dir-apache}

這說明發送器如何與Apache網路伺服器 `mod_dir` 互動，因為這可能會導致各種可能的非預期效果：

### Apache1.3 {#apache}

In Apache1.3 `mod_dir` 處理URL對應至檔案系統中目錄的每個請求。

其中包括：

* 將請求重新導向至現有 `index.html` 檔案
* 產生目錄清單

當dispatcher啓用時，會將其登錄為內容類型 `httpd/unix-directory`的處理常式，以處理這些請求。

### Apache2.x {#apache-x}

Apache2.x的內容不同。模組可以處理請求的不同階段，例如URL修正。`mod_dir` 將請求(當URL對應至目錄時)重新導向至URL，以處理此 `/` 階段。

Dispatcher不會攔截 `mod_dir` 修正，但會完全處理重新導向URL(亦即附加 `/` 的URL)的要求。如果遠端伺服器(例如AEM)處理要求與要求 `/a_path` 不同(對應至現有目錄時 `/a_path/``/a_path` )，這可能會造成問題。

如果發生這種情況，您必須：

* dispatcher `mod_dir` 處理的 `Directory` 或 `Location` 子樹狀結構停用

* 用於 `DirectorySlash Off` 設定 `mod_dir` 不附加 `/`
