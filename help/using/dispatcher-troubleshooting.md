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
source-git-commit: 3a0e237278079a3885e527d7f86989f8ac91e09d
workflow-type: ht
source-wordcount: '543'
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
>另請檢查 [Dispatcher知識庫](https://helpx.adobe.com/cq/kb/index/dispatcher.html)。 [Dispatcher刷新問題疑難解答](https://helpx.adobe.com/adobe-cq/kb/troubleshooting-dispatcher-flushing-issues.html) 和 [Dispatcher常見問題](dispatcher-faq.md) 的上界。

## 檢查基本配置 {#check-the-basic-configuration}

與以往一樣，第一步是檢查基本資訊：

* [確認基本操作](/help/using/dispatcher-configuration.md#confirming-basic-operation)
* 檢查Web伺服器和調度程式的所有日誌檔案。 如有必要，請增加 `loglevel` 用於調度員 [記錄](/help/using/dispatcher-configuration.md#logging)。

* [檢查配置](/help/using/dispatcher-configuration.md):

   * 您有多個調度程式嗎？

      * 您確定哪個Dispatcher正在處理您正在調查的網站/頁面嗎？
   * 您是否實施過篩選器？

      * 這些是否會影響您正在調查的事項？


## IIS診斷工具 {#iis-diagnostic-tools}

IIS提供各種跟蹤工具，具體取決於實際版本：

* IIS 6 — 可以下載和配置IIS診斷工具
* IIS 7 — 跟蹤已完全整合

這些功能可幫助您監控活動。

## 找不到IIS和404 {#iis-and-not-found}

使用IIS時，您可能會體驗 `404 Not Found` 在各種情況下返回。 如果是，請參閱以下知識文庫文章。

* [IIS 6/7 :POST方法返回404](https://helpx.adobe.com/dispatcher/kb/IIS6IsapiFilters.html)
* [IIS 6:對包含基本路徑的URL的請求 `/bin` 返回 `404 Not Found`](https://helpx.adobe.com/dispatcher/kb/RequestsToBinDirectoryFailInIIS6.html)

您還應檢查調度程式快取根目錄和IIS文檔根目錄是否設定為同一目錄。

## 刪除工作流模型的問題 {#problems-deleting-workflow-models}

**症狀**

在通過Dispatcher訪問作者實例時嘗試刪AEM除工作流模型時出現問題。

**要再現的步驟：**

1. 登錄到作者實例（確認正在通過調度程式路由請求）。
1. 建立新工作流；例如，將「標題」設定為workflowToDelete。
1. 確認已成功建立工作流。
1. 選擇並按一下右鍵工作流，然後按一下 **刪除**。

1. 按一下 **是** 確認。
1. 將出現一個錯誤消息框，其中顯示：\
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

這描述了調度程式如何與 `mod_dir` 在Apache Webserver中，因為這會導致各種潛在意外的效果：

### Apache 1.3 {#apache}

在Apache 1.3中 `mod_dir` 處理URL映射到檔案系統目錄的每個請求。

它也會：

* 將請求重定向到現有 `index.html` 檔案
* 生成目錄清單

當調度程式處於啟用狀態時，它通過將自身註冊為內容類型的處理程式來處理此類請求 `httpd/unix-directory`。

### Apache 2.x {#apache-x}

在Apache 2.x中，情況不同。 一個模組可以處理請求的不同階段，如URL修正。 `mod_dir` 通過將請求（當URL映射到目錄時）重定向到URL來處理此階段， `/` 已附加。

Dispatcher不會截獲 `mod_dir` 修復，但完全處理對重定向URL的請求(即 `/` )。 如果遠程伺服器（例如）處理請求，則AEM可能會出現問題 `/a_path` 與請求不同 `/a_path/` (當 `/a_path` 映射到現有目錄)。

如果發生這種情況，您必須執行以下任一操作：

* 禁用 `mod_dir` 為 `Directory` 或 `Location` 調度程式處理的子樹

* 使用 `DirectorySlash Off` 配置 `mod_dir` 不追加 `/`
