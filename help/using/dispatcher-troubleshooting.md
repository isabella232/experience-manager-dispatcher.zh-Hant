---
title: 疑難排解 Dispatcher 問題
seo-title: 疑難排解AEM Dispatcher問題
description: 了解如何疑難排解Dispatcher問題。
seo-description: 了解AEM Dispatcher問題的疑難排解。
uuid: 9c109a48-d921-4b6e-9626-1158cebc41e7
cmgrlastmodified: 01.11.2007 08 22 29 [aheimoz]
pageversionid: 1193211344162
template: /apps/docs/templates/contentpage
contentOwner: User
products: SG_EXPERIENCEMANAGER/DISPATCHER
topic-tags: dispatcher
content-type: reference
discoiquuid: a612e745-f1e6-43de-b25a-9adcaadab5cf
exl-id: 29f338ab-5d25-48a4-9309-058e0cc94cff
source-git-commit: 3a0e237278079a3885e527d7f86989f8ac91e09d
workflow-type: tm+mt
source-wordcount: '553'
ht-degree: 6%

---

# 疑難排解 Dispatcher 問題 {#troubleshooting-dispatcher-problems}

>[!NOTE]
>
>Dispatcher版本與AEM無關，但AEM檔案內嵌有Dispatcher檔案。 請一律使用檔案中內嵌的Dispatcher檔案，以取得最新版AEM。
>
>如果您依循連結至 Dispatcher 文件，且該連結內嵌於舊版 AEM 的文件中，您可能會被重新導向至本頁。

>[!NOTE]
>
>如需詳細資訊，另請參閱[Dispatcher知識庫](https://helpx.adobe.com/cq/kb/index/dispatcher.html)、[疑難排解Dispatcher排清問題](https://helpx.adobe.com/adobe-cq/kb/troubleshooting-dispatcher-flushing-issues.html)和[Dispatcher熱門問題常見問題集](dispatcher-faq.md)。

## 檢查基本配置{#check-the-basic-configuration}

如同前幾步一樣，首先要檢查基本知識：

* [確認基本操作](/help/using/dispatcher-configuration.md#confirming-basic-operation)
* 檢查您的Web伺服器和Dispatcher的所有記錄檔。 如有必要，請增加用於Dispatcher [logging](/help/using/dispatcher-configuration.md#logging)的`loglevel`。

* [檢查您的設定](/help/using/dispatcher-configuration.md):

   * 您有多個Dispatcher嗎？

      * 您確定哪個Dispatcher正在處理您正在調查的網站/頁面嗎？
   * 您已實作篩選器嗎？

      * 這些會影響你調查的事嗎？


## IIS診斷工具{#iis-diagnostic-tools}

IIS提供各種跟蹤工具，具體取決於實際版本：

* IIS 6 — 可下載並配置IIS診斷工具
* IIS 7 — 跟蹤已完全整合

這些功能可協助您監控活動。

## IIS和404 Not Found {#iis-and-not-found}

使用IIS時，您可能會在各種情況下遇到`404 Not Found`傳回的情況。 若有，請參閱下列知識庫文章。

* [IIS 6/7:POST方法返回404](https://helpx.adobe.com/dispatcher/kb/IIS6IsapiFilters.html)
* [IIS 6:傳回包含基本路徑的URL要 `/bin` 求  `404 Not Found`](https://helpx.adobe.com/dispatcher/kb/RequestsToBinDirectoryFailInIIS6.html)

您也應檢查Dispatcher快取根目錄和IIS檔案根目錄是否已設為相同目錄。

## 刪除工作流模型{#problems-deleting-workflow-models}時出現問題

**症狀**

透過Dispatcher存取AEM製作例項時，嘗試刪除工作流程模型時發生問題。

**重現問題的步驟：**

1. 登入您的製作例項（確認請求是透過Dispatcher路由）。
1. 建立新的工作流程；例如，將「標題」設為workflowToDelete。
1. 確認工作流程已成功建立。
1. 選取並以滑鼠右鍵按一下工作流程，然後按一下&#x200B;**Delete**。

1. 按一下「**是**」進行確認。
1. 畫面上會顯示錯誤訊息方塊：\
   &quot; `ERROR 'Could not delete workflow model!!`&quot;

**解析度**

將下列標題新增至`dispatcher.any`檔案的`/clientheaders`區段：

* `x-http-method-override`
* `x-requested-with`

```
{  
{  
/clientheaders  
{  
...  
"x-http-method-override"  
"x-requested-with"  
}
```

## 與mod_dir(Apache){#interference-with-mod-dir-apache}的干擾

這說明Dispatcher如何與Apache網站伺服器內的`mod_dir`互動，因為這可能會導致各種可能未預期的效果：

### Apache 1.3 {#apache}

在Apache 1.3 `mod_dir`中，處理URL映射至檔案系統中目錄的每個請求。

它也會：

* 將請求重新導向至現有的`index.html`檔案
* 生成目錄清單

啟用Dispatcher後，會將其本身註冊為內容類型`httpd/unix-directory`的處理常式，以處理此類請求。

### Apache 2.x {#apache-x}

在Apache 2.x中，情況不同。 模組可處理要求的不同階段，例如URL修正。 `mod_dir` 借由將請求（當URL對應至目錄時）重新導向至附加的URL來處理此 `/` 階段。

Dispatcher不會截取`mod_dir`修正，但會完全處理對重新導向URL的要求（亦即附加了`/`）。 如果遠程伺服器(如AEM)處理`/a_path`的請求與`/a_path/`的請求不同（當`/a_path`映射到現有目錄時），則這可能會造成問題。

如果發生此情況，您必須：

* 針對調度程式處理的`Directory`或`Location`子樹狀結構，停用`mod_dir`

* 使用`DirectorySlash Off`配置`mod_dir`不附加`/`
