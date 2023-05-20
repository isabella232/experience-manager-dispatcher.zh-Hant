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
ht-degree: 100%

---

# 疑難排解 Dispatcher 問題 {#troubleshooting-dispatcher-problems}

>[!NOTE]
>
>Dispatcher版本與AEM無關，但Dispatcher文檔嵌入到文檔AEM中。 請始終使用文檔中嵌入的Dispatcher文檔獲取最新版本AEM。
>
>如果您依循連結至 Dispatcher 文件，且該連結內嵌於舊版 AEM 的文件中，您可能會被重新導向至本頁。

>[!NOTE]
>
>請參考 [Dispatcher 知識庫](https://helpx.adobe.com/tw/experience-manager/kb/index/dispatcher.html)。 [疑難排解 Dispatcher 清除問題](https://experienceleague.adobe.com/search.html?lang=zh-Hant#q=troubleshooting%20dispatcher%20flushing%20issues&amp;sort=relevancy&amp;f:el_product=[Experience%20Manager])和 [Dispatcher 熱門問題常見問題集](dispatcher-faq.md)以了解進一步資訊。

## 檢查基本配置 {#check-the-basic-configuration}

與以往一樣，第一步是檢查基本資訊：

* [確認基本操作](/help/using/dispatcher-configuration.md#confirming-basic-operation)
* 檢查 Web 伺服器和 Dispatcher 的所有日誌檔案。 必要時，請提高`loglevel` (用於 Dispatcher [記錄](/help/using/dispatcher-configuration.md#logging))。

* [檢查配置](/help/using/dispatcher-configuration.md)：

   * 您有多個 Dispatcher 嗎？

      * 您確定哪個Dispatcher正在處理您正在調查的網站/頁面嗎？
   * 您是否實施過篩選器？

      * 這些篩選器是否影響您正在調查的事項?


## IIS診斷工具 {#iis-diagnostic-tools}

IIS提供各種跟蹤工具，具體取決於實際版本：

* IIS 6 — 可以下載和配置IIS診斷工具
* IIS 7 — 跟蹤已完全整合

這些工具可幫助您監控活動。

## IIS 和 404 找不到 {#iis-and-not-found}

當您使用 IIS 時，可能會在各種情況下遇到網頁傳回 `404 Not Found`。 如果是，請參閱以下知識文庫文章。

* [IIS 6/7：POST 方法返回 404](https://helpx.adobe.com/tw/experience-manager/kb/IIS6IsapiFilters.html)
* [IIS 6：請求包含基本路徑 `/bin` 的 URL 時傳回 `404 Not Found`](https://helpx.adobe.com/tw/experience-manager/kb/RequestsToBinDirectoryFailInIIS6.html)

另請參考 Dispatcher 快取根目錄和 IIS 文件根目錄是否設定為同一目錄。

## 刪除工作流模型的問題 {#problems-deleting-workflow-models}

**症狀**

在通過 Dispatcher 存取編寫執行個體時嘗試刪 AEM 除工作流模型時出現問題。

**重現問題的步驟：**

1. 登入到編寫執行個體 (確認正透過 Dispatcher 路由請求)。
1. 建立工作流程；例如，將「標題」設定為 workflowToDelete。
1. 確認已成功建立工作流程。
1. 選取並用滑鼠右鍵按一下工作流程，然後按一下「**刪除**」。

1. 按一下「**是**」確認。
1. 將出現一個顯示以下內容的錯誤訊息框：\
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

此程序描述 Dispatcher 如何與 Apache Webserver 中的 `mod_dir` 互動，因為這可能導致各種潛在的意外效果：

### Apache 1.3 {#apache}

在 Apache 1.3 中，`mod_dir` 會處理 URL 對應到檔案系統中的目錄的每個請求。

它也會：

* 將請求重定向到現有 `index.html` 檔案
* 生成目錄清單

在啟用 Dispatcher 時，它會處理這類請求，處理方式是將自己登錄為內容類型 `httpd/unix-directory` 的處理常式。

### Apache 2.x {#apache-x}

在 Apache 2.x 中，情況不同。 一個模組可以處理請求的不同階段，如 URL 修正。 `mod_dir` 會處理此階段，處理方式是將請求 (當 URL 對應到目錄時) 重新導向至已附加 `/` 的 URL。

Dispatcher不會攔截 `mod_dir` 修正，但完全處理對重新導向的 URL 的請求 (即附加 `/` )。 如果遠端伺服器 (例如 AEM) 處理對 `/a_path` 的請求的方式不同於對 `/a_path/` 的請求 (當 `/a_path` 對應到現有目錄)，此程序可能會導致問題。

如果發生這種情況，您必須執行以下任一操作：

* 停用 `mod_dir` (針對 Dispatcher 處理的 `Directory` 或 `Location` 子樹狀結構)

* 使用 `DirectorySlash Off` 配置 `mod_dir` 不追加 `/`
