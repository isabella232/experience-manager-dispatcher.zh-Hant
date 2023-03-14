---
title: 疑難排解 Dispatcher 問題
seo-title: Troubleshooting AEM Dispatcher Problems
description: 瞭解Dispatcher問題的疑難解答。
seo-description: Learn to troubleshoot AEM Dispatcher issues.
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
source-git-commit: 26c8edbb142297830c7c8bd068502263c9f0e7eb
workflow-type: tm+mt
source-wordcount: '560'
ht-degree: 43%

---

# 疑難排解 Dispatcher 問題 {#troubleshooting-dispatcher-problems}

>[!NOTE]
>
>Dispatcher版本與AEM無關，但Dispatcher文檔嵌入到文檔AEM中。 請始終使用文檔中嵌入的Dispatcher文檔獲取最新版本AEM。
>
>如果您依循連結至 Dispatcher 文件，且該連結內嵌於舊版 AEM 的文件中，您可能會被重新導向至本頁。

>[!NOTE]
>
>檢查 [Dispatcher知識庫](https://helpx.adobe.com/experience-manager/kb/index/dispatcher.html), [疑難排解Dispatcher排清問題](https://experienceleague.adobe.com/search.html?lang=en#q=troubleshooting%20dispatcher%20flushing%20issues&amp;sort=relevancy&amp;f:el_product=[Experience%20Manager]) 和 [Dispatcher熱門問題常見問題集](dispatcher-faq.md) 以取得更多資訊。

## 檢查基本配置 {#check-the-basic-configuration}

與以往一樣，第一步是檢查基本資訊：

* [確認基本操作](/help/using/dispatcher-configuration.md#confirming-basic-operation)
* 檢查您的Web伺服器和Dispatcher的所有記錄檔。 如有必要，請增加 `loglevel` 用於Dispatcher [記錄](/help/using/dispatcher-configuration.md#logging).

* [檢查配置](/help/using/dispatcher-configuration.md):

   * 您有多個調度程式嗎？

      * 您確定哪個Dispatcher正在處理您正在調查的網站/頁面嗎？
   * 您是否實施過篩選器？

      * 這些過濾器是否會影響您正在調查的問題？


## IIS診斷工具 {#iis-diagnostic-tools}

IIS提供各種跟蹤工具，具體取決於實際版本：

* IIS 6 — 可以下載和配置IIS診斷工具
* IIS 7 — 跟蹤已完全整合

這些工具可協助您監控活動。

## 找不到IIS和404 {#iis-and-not-found}

使用IIS時，您可能會遇到 `404 Not Found` 在各種情況下傳回。 如果是，請參閱以下知識文庫文章。

* [IIS 6/7 :POST方法返回404](https://helpx.adobe.com/experience-manager/kb/IIS6IsapiFilters.html)
* [IIS 6:包含基本路徑的URL請求 `/bin` 傳回 `404 Not Found`](https://helpx.adobe.com/experience-manager/kb/RequestsToBinDirectoryFailInIIS6.html)

另請檢查Dispatcher快取根目錄和IIS檔案根目錄是否已設為相同目錄。

## 刪除工作流模型的問題 {#problems-deleting-workflow-models}

**症狀**

在通過Dispatcher訪問作者實例時嘗試刪AEM除工作流模型時出現問題。

**要再現的步驟：**

1. 登入您的製作例項（確認請求是透過Dispatcher路由）。
1. 建立工作流程；例如，將「標題」設為workflowToDelete。
1. 確認已成功建立工作流。
1. 選取並以滑鼠右鍵按一下工作流程，然後按一下 **刪除**.

1. 按一下 **是** 確認。
1. 出現錯誤訊息方塊，其中顯示下列內容：\
   &quot; `ERROR 'Could not delete workflow model!!`&quot;.

**解析度**

將以下標題添加到 `/clientheaders` 的 `dispatcher.any` 檔案：

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

## 與mod_dir(Apache)的干涉 {#interference-with-mod-dir-apache}

此程式說明Dispatcher如何與 `mod_dir` 在Apache網站伺服器內，因為這可能會導致各種可能未預期的效果：

### Apache 1.3 {#apache}

在Apache 1.3中， `mod_dir` 處理URL映射至檔案系統中目錄的每個請求。

它也會：

* 將請求重定向到現有 `index.html` 檔案
* 生成目錄清單

啟用Dispatcher時，會將本身註冊為內容類型的處理常式，以處理這類請求 `httpd/unix-directory`.

### Apache 2.x {#apache-x}

在Apache 2.x中，情況有所不同。 一個模組可以處理請求的不同階段，如URL修正。 此 `mod_dir` 透過將請求（當URL對應至目錄時）重新導向至URL(使用 `/` 已附加。

Dispatcher不會截取 `mod_dir` 修正，但會完全處理重新導向URL的要求(亦即 `/` 已附加)。 如果遠端伺服器(例如AEM)處理 `/a_path` 與要求不同 `/a_path/` 時 `/a_path` 映射到現有目錄)。

如果發生此情況，您必須：

* disable `mod_dir` 針對 `Directory` 或 `Location` 由Dispatcher處理的子樹

* 使用 `DirectorySlash Off` 配置 `mod_dir` 不追加 `/`
