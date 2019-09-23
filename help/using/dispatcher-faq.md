---
title: Dispatcher的主要問題
seo-title: AEM Dispatcher的主要問題
description: AEM Dispatcher的主要問題
seo-description: Adobe AEM Dispatcher的主要問題
translation-type: tm+mt
source-git-commit: 76cffbfb616cd5601aed36b7076f67a2faf3ed3b

---


# AEM Dispatcher熱門問題常見問答集

![配置Dispatcher](assets/CQDispatcher_workflow_v2.png)

## 簡介

### 什麼是Dispatcher?

Dispatcher是Adobe Experience manager的快取和／或負載平衡工具，可協助您建立快速動態的網頁製作環境。 對於快取，Dispatcher會以HTTP伺服器（例如Apache）的一部份運作，以盡可能儲存（或「快取」）靜態網站內容，並盡可能不常存取網站的版面引擎。 在負載平衡角色中，Dispatcher會在不同的AEM例項（轉譯）間分發使用者要求（負載）。

對於快取，Dispatcher模組使用Web伺服器提供靜態內容的能力。 Dispatcher將快取的文檔放在Web伺服器的文檔根目錄中。

### Dispatcher如何執行快取？

Dispatcher使用Web伺服器提供靜態內容的能力。 Dispatcher會將快取的檔案儲存在Web伺服器的檔案根目錄中。 Dispatcher有兩種主要方法，用於在對網站進行更改時更新快取內容。

* **內容更新** ：移除已變更的頁面，以及直接與之關聯的檔案。
* **自動失效** (Auto-Invalidation)會自動使更新後可能過期的快取部分失效。 例如，它會將相關頁面標示為已過時，而不會刪除任何內容。

### 負載平衡有哪些好處？

Load Balancing會將使用者要求（負載）分發至數個AEM例項。下列清單說明負載平衡的優點：

* **提高處理能力**:實際上，這表示Dispatcher會在數個AEM執行個體之間共用檔案請求。 由於每個例項處理的檔案較少，因此您的回應時間更快。 Dispatcher會保留每個文檔類別的內部統計資訊，以便能夠估計負載並高效地分發查詢。
* **增加故障保險範圍**:如果Dispatcher未從實例接收響應，它將自動將請求中繼到另一個實例。 因此，如果某個例項無法使用，唯一的效果是網站的減速，與失去的計算能力成正比。

>[!NOTE]
>
>有關詳細資訊，請參閱「 [Dispatcher概述」頁](dispatcher.md)

## 安裝和配置

### 我要從何處下載Dispatcher模組？

You can download the latest Dispatcher module from the Dispatcher Release Notes page.[](release-notes.md)

### How do I install the Dispatcher module?

Refer to the Installing Dispatcher page[](dispatcher-install.md)

### How do I configure the Dispatcher module?

See the [Configuring Dispatcher](dispatcher-configuration.md) page.

### How do I configure the Dispatcher for the author instance?

See Using Dispatcher with an Author Instance for the detailed steps.[](dispatcher.md#using-a-dispatcher-with-an-author-server)

### How do I configure the Dispatcher with multiple domains?

You can configure the CQ Dispatcher with multiple domains, provided the domains satisfy the following conditions:

* The Web content for both the domains is stored in a single AEM repository
* The files in the Dispatcher cache can be invalidated separately for each domain

Read Using Dispatcher with Multiple Domains for further details.[](dispatcher-domains.md)

### How do I configure the Dispatcher, such that all requests from a user are routed to the same Publish instance?

您可以使用 [嚴格連線](dispatcher-configuration.md#identifying-a-sticky-connection-folder-stickyconnectionsfor) ，這可確保使用者的所有檔案都會在AEM的同一個例項上處理。 如果您使用個人化頁面和作業資料，此功能就很重要。 The data is stored on the instance. Therefore, subsequent requests from the same user must return to that instance or the data is lost.

由於粘滯連接限制了Dispatcher優化請求的能力，因此您應僅在必要時使用此方法。 您可以指定包含「嚴格」檔案的檔案夾，如此可確保該檔案夾中的所有檔案都會在使用者的相同執行個體上處理。

### 我是否可同時使用黏著連線和快取？

對於使用嚴格連線的大部分頁面，您應關閉快取。 否則，不論作業內容為何，頁面的相同例項都會顯示給所有使用者。

對於某些應用程式，可同時使用嚴格連線和快取。 例如，如果您顯示將資料寫入作業的表單，則可同時使用黏著連線和快取。

### Dispatcher和AEM Publish實例是否駐留在同一台物理電腦上？

是的，如果機器足夠強大。 不過，建議您在不同的電腦上設定Dispatcher和AEM Publish執行個體。

通常， Publish實例駐留在防火牆內，而Dispatcher駐留在DMZ中。 如果您決定將Publish實例和Dispatcher都放在同一台物理電腦上，請確保防火牆設定禁止從外部網路直接訪問Publish實例。

### 我是否只能快取具有特定副檔名的檔案？

是. 例如，如果只要快取GIF檔案，請在dispatcher.any配置檔案的快取部分中指定*.gif。

### 如何從快取中刪除檔案？

您可以使用HTTP請求，從快取中刪除檔案。 收到HTTP請求時，Dispatcher會從快取中刪除檔案。 Dispatcher僅在收到頁面的客戶端請求時才會再次快取檔案。 以此方式刪除快取檔案，對於不太可能同時收到相同頁面要求的網站而言是適當的。

HTTP請求具有下列語法：

```
POST /dispatcher/invalidate.cache HTTP/1.1
CQ-Action: Activate
CQ-Handle: path-pattern
Content-Length: 0
```

Dispatcher會刪除名稱與CQ-Handle標題值相符的快取檔案和檔案夾。 例如，CQ-Handle與下 `/content/geomtrixx-outdoors/en` 列項目相符：

在geometrixx-outdoors目錄中命名的所有檔案（任何副檔名）在en目錄下命名的任何目錄（如果存在，則包含頁面子節點的快取轉譯）只有在 `_jcr_content` 為 `CQ-Action` 或時，才會刪除目錄en `Delete``Deactivate`。

有關此主題的詳細資訊，請參 [閱手動使Dispatcher Cache失效](page-invalidate.md)。

### 如何實作權限相關快取？

請參閱快 [取保全內容](permissions-cache.md) 頁面。

### 如何保護Dispatcher和CQ實例之間的通信安全？

See the Dispatcher Security Checklist and the AEM Security Checklist pages.[](security-checklist.md)[](https://helpx.adobe.com/experience-manager/6-4/sites/administering/using/security-checklist.html)

### Dispatcher問題 `jcr:content` 變更為 `jcr%3acontent`

**Question: We have recently faced a problem at dispatcher level wherein one of the ajax call which was getting some data form CQ repository had  in it and that got encoded to  resulting in wrong result set.**`jcr:content``jcr%3acontent`

**Answer: Please use  method to get a 'Friendly' URL to be used / issued get requests from and also to solve the caching issue with Dispatcher.**`ResourceResolver.map()`The map() method encodes the  colon to underscores and the resolve() method decodes them back to SLING JCR readable format.You need to use the map() method to generate the URL that is used in the Ajax call.`:`

Further read: https://sling.apache.org/documentation/the-sling-engine/mappings-for-resource-resolution.html#namespace-mangling[](https://sling.apache.org/documentation/the-sling-engine/mappings-for-resource-resolution.html#namespace-mangling)

## Flush the Dispatcher

### How do I configure Dispatcher flush agents on a Publish instance?

See the Replication page.[](https://helpx.adobe.com/content/help/en/experience-manager/6-4/sites/deploying/using/replication.html#ConfiguringyourReplicationAgents)

### How do I troubleshoot Dispatcher flushing issues?

[Refer to this troubleshooting article that answers the following questions:](https://helpx.adobe.com/content/help/en/experience-manager/kb/troubleshooting-dispatcher-flushing-issues.html)

* How should I debug a situation where no content is getting saved in the Dispatcher cache?
* How do I debug a situation where cache files are not getting updated?
* How do I debug a situation where nothing related to Dispatcher flushing is working?

If Delete operations are causing the Dispatcher to flush, use the workaround in this community blog post by Sensei Martin.[](https://mkalugin-cq.blogspot.in/2012/04/i-have-been-working-on-following.html)

### 如何從Dispatcher快取刷新DAM資產？

You can use the "chain replication" feature.  啟用此功能後，當從作者處收到複製時，調度程式刷新代理會發送刷新請求。

若要啟用它：

1. [請依照此處的步驟](page-invalidate.md#invalidating-dispatcher-cache-from-a-publishing-instance) ，在發佈時建立沖洗代理
1. 轉到這些代理的每個配置，並在「觸發器」( **Triggers** )頁籤上選 **中「接收時** 」框。

## 其他

Dispatcher如何確定文檔是否是最新的？
要確定文檔是否是最新的，Dispatcher將執行以下操作：

它會檢查檔案是否受到自動失效的影響。 否則，檔案會視為最新。
如果文檔已配置為自動失效，Dispatcher將檢查其是否比上次可用更改更舊或更新。 如果版本較舊，Dispatcher會從AEM例項要求目前版本，並取代快取中的版本。

### Dispatcher如何傳回檔案？

您可以定義Dispatcher是否使用 [Dispatcher配置檔案快取文](dispatcher-configuration.md) 檔 `dispatcher.any`,。 Dispatcher會根據可快取文檔的清單檢查請求。 If the document is not in this list, the Dispatcher requests the document from the AEM instance.

The  property controls which documents are cached according to the document path. `/rules`無論屬性 `/rules` 如何，Dispatcher都不會在以下情況下快取文檔：

* If the request URI contains a question mark .`(?)`
* 這通常表示動態頁面，例如不需要快取的搜尋結果。
* The file extension is missing.
* The web server needs the extension to determine the document type (the MIME-type).
* The authentication header is set (this can be configured)
* If the AEM instance responds with the following headers:
   * no-cache
   * no-store
   * must-revalidate

Dispatcher會將快取檔案儲存在Web伺服器上，就像是靜態網站的一部分。 如果用戶請求快取的文檔，Dispatcher將檢查該文檔是否存在於Web伺服器的檔案系統中。 如果是，Dispatcher將返回文檔。 如果沒有，Dispatcher會從AEM例項請求檔案。

>[!NOTE]
>
>GET或HEAD（用於HTTP標頭）方法可由Dispatcher進行快取。 如需回應標頭快取的詳細資訊，請參 [閱快取HTTP回應標頭](dispatcher-configuration.md#caching-http-response-headers) 。

### 我是否可以在設定中實施多個調度程式？

是. 在此情況下，請確定兩個「調度程式」都能直接存取AEM網站。 Dispatcher無法處理來自其他Dispatcher的請求。

