---
title: Dispatcher問題疑難排解
seo-title: 疑難排解AEM Dispatcher問題
description: 瞭解如何疑難排解Dispatcher問題。
seo-description: 瞭解如何疑難排解AEM Dispatcher問題。
uuid: 9c109a48-d921-4b6e-9626-1158cebc41e7
cmgrlastmodified: 01.11.2007 08 22 29 [aheimoz]
pageversionid: '1193211344162'
template: /apps/docs/templates/contentpage
contentOwner: 使用者
products: SG_EXPERIENCEMANAGER/DISPATCHER
topic-tags: dispatcher
content-type: 引用
discoiquuid: a612e745-f1e6-43de-b25a-9adcaadab5cf
translation-type: tm+mt
source-git-commit: 76cffbfb616cd5601aed36b7076f67a2faf3ed3b

---


# Dispatcher問題疑難排解 {#troubleshooting-dispatcher-problems}

>[!NOTE]
>
>Dispatcher版本與AEM無關，但Dispatcher檔案已內嵌在AEM檔案中。 請務必使用檔案中內嵌的Dispatcher檔案，以取得最新版的AEM。
>
>如果您遵循Dispatcher檔案的連結，且該連結內嵌於舊版AEM的檔案中，您可能會被重新導向至本頁面。

>[!NOTE]
>
>另請查看 [Dispatcher知識庫](https://helpx.adobe.com/cq/kb/index/dispatcher.html)、 [Troubleshooting Dispatcher Flushing Issues](https://helpx.adobe.com/adobe-cq/kb/troubleshooting-dispatcher-flushing-issues.html) 和 [](dispatcher-faq.md) Dispatcher Top Issues常見問答集以取得詳細資訊。

## 檢查基本配置 {#check-the-basic-configuration}

首先要檢查基本知識，一如往常：

* [確認基本操作](#ConfirmBasicOperation)
* 檢查Web伺服器和調度程式的所有日誌檔案。 如有必要，請 `loglevel` 增加用於調度程式 [日誌](#Logging)。

* [檢查您的配置](#ConfiguringtheDispatcher):

   * 您有多個調度程式嗎？

      * 您確定哪個Dispatcher正在處理您正在調查的網站／頁面嗎？
   * 您是否已實作過濾器？

      * 這些對您正在調查的事件有影響嗎？


## IIS診斷工具 {#iis-diagnostic-tools}

IIS提供各種跟蹤工具，取決於實際版本：

* IIS 6 - IIS診斷工具可下載並配置
* IIS 7 —— 跟蹤已完全整合

這些功能可協助您監控活動。

## 找不到IIS和404 {#iis-and-not-found}

使用IIS時，您可能會遇 `404 Not Found` 到在各種情況下傳回的情況。 如果是，請參閱下列知識庫文章。

* [IIS 6/7:POST方法返回404](https://helpx.adobe.com/dispatcher/kb/IIS6IsapiFilters.html)
* [IIS 6:傳回包含基本路徑的URL要 `/bin` 求 `404 Not Found`](https://helpx.adobe.com/dispatcher/kb/RequestsToBinDirectoryFailInIIS6.html)

您還應檢查調度程式快取根目錄和IIS文檔根目錄是否設定為同一目錄。

## 刪除工作流模型的問題 {#problems-deleting-workflow-models}

**症狀**

在透過Dispatcher存取AEM作者例項時嘗試刪除工作流程模型的問題。

**重制步驟：**

1. 登入您的作者例項（確認請求是透過Dispatcher傳送）。
1. 建立新的工作流程；例如，將「標題」設為workflowToDelete。
1. 確認工作流程已成功建立。
1. 選取工作流程並按一下滑鼠右鍵，然後按一下「刪 **除」**。

1. 按一 **下「是** 」以確認。
1. 將出現一個錯誤消息框，其中顯示：\
   " `ERROR 'Could not delete workflow model!!`".

**解析度**

將下列標題新增至 `/clientheaders` 檔案的區 `dispatcher.any` 段：

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

## 與mod_dir(Apache)的干涉 {#interference-with-mod-dir-apache}

這將說明調度程式如何與 `mod_dir` Apache Webserver內部進行交互，因為這可能導致各種可能未預期的效果：

### Apache 1.3 {#apache}

在Apache 1.3中， `mod_dir` 處理URL對應至檔案系統目錄的每個請求。

它或者：

* 將請求重新導向至現有檔 `index.html` 案
* 生成目錄清單

當調度器啟用時，它會將自身註冊為內容類型的處理常式，以處理此類請求 `httpd/unix-directory`。

### Apache 2.x {#apache-x}

在Apache 2.x中，情況不同。 模組可處理請求的不同階段，例如URL修正。 `mod_dir` 將請求（當URL對應至目錄時）重新導向至附加的URL，以處理此階 `/` 段。

Dispatcher不會截取修 `mod_dir` 正，但會完全處理對重新導向URL的請求(即附加 `/` 的)。 如果遠端伺服器（例如AEM）處理的要求與要求不同(當對應至現 `/a_path` 有目錄時), `/a_path/``/a_path` 這可能會造成問題。

如果發生這種情況，您必須：

* 禁 `mod_dir` 用調度 `Directory` 器所處 `Location` 理的或子樹

* 使用 `DirectorySlash Off` 配置 `mod_dir` 不附加 `/`
