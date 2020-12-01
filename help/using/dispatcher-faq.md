---
title: Dispatcher的主要問題
seo-title: AEM Dispatcher的主要問題
description: AEM Dispatcher的主要問題
seo-description: Adobe AEM Dispatcher的主要問題
translation-type: tm+mt
source-git-commit: eed7c3f77ec64f2e7c5cfff070ef96108886a059
workflow-type: tm+mt
source-wordcount: '1644'
ht-degree: 13%

---


# AEM Dispatcher熱門問題常見問答集

![設定 Dispatcher](assets/CQDispatcher_workflow_v2.png)

## 簡介

### 什麼是Dispatcher?

Dispatcher是Adobe Experience Manager的快取和／或負載平衡工具，可協助您建立快速動態的網頁製作環境。 對於快取，Dispatcher會以HTTP伺服器（例如Apache）的一部份運作，以盡可能儲存（或「快取」）靜態網站內容，並盡可能不常存取網站的版面引擎。 在負載平衡角色中，Dispatcher會在不同的AEM例項（轉譯）間分發使用者要求（負載）。

對於快取，Dispatcher模組使用Web伺服器提供靜態內容的能力。 Dispatcher將快取的文檔放在Web伺服器的文檔根目錄中。

### Dispatcher如何執行快取？

Dispatcher使用Web伺服器提供靜態內容的能力。 Dispatcher會將快取的檔案儲存在Web伺服器的檔案根目錄中。 Dispatcher 有兩種主要方法，用於在對網站進行更改時更新快取內容。

* **內容更新**&#x200B;會移除已變更的頁面，以及與其直接相關的檔案。
* **自動失效**&#x200B;功能會自動讓那些在更新後可能過期的快取失效。例如，它會將相關頁面標示為已過時，而不會刪除任何內容。

### 負載平衡有哪些好處？

Load Balancing會將使用者要求（負載）分發至數個AEM例項。下列清單說明負載平衡的優點：

* **提高處理能力**:實際上，這表示Dispatcher會在數個AEM執行個體之間共用檔案請求。由於每個例項處理的檔案較少，因此您的回應時間更快。 Dispatcher 會保留每個文件類別的內部統計資料，以便能夠估計負載並有效率地分配查詢。
* **增加故障保險範圍**:如果Dispatcher未從實例接收響應，它將自動將請求中繼到另一個實例。因此，如果某個例項無法使用，唯一的影響是網站速度變慢，與失去的運算能力成正比。

>[!NOTE]
>
>有關詳細資訊，請參閱[Dispatcher Overview頁](dispatcher.md)

## 安裝和配置

### 我要從何處下載Dispatcher模組？

您可以從[Dispatcher發行說明](release-notes.md)頁下載最新的Dispatcher模組。

### 如何安裝Dispatcher模組？

請參閱[安裝Dispatcher](dispatcher-install.md)頁

### 如何配置Dispatcher模組？

請參見[配置Dispatcher](dispatcher-configuration.md)頁。

### 如何為作者實例配置Dispatcher?

有關詳細步驟，請參見[ Using Dispatcher with an Author Instance](dispatcher.md#using-a-dispatcher-with-an-author-server)。

### 如何配置具有多個域的Dispatcher?

您可以配置具有多個域的CQ Dispatcher，前提是這些域滿足以下條件：

* 這兩個網域的Web內容都儲存在單一AEM儲存庫中
* Dispatcher快取中的檔案可針對每個域分別失效

有關詳細資訊，請閱讀[Using Dispatcher with Multiple Domains](dispatcher-domains.md)。

### 如何配置Dispatcher，以便將來自用戶的所有請求路由到同一個Publish實例？

您可以使用[嚴格連線](dispatcher-configuration.md#identifying-a-sticky-connection-folder-stickyconnectionsfor)功能，以確保使用者的所有檔案都會在相同的AEM例項上處理。 如果您使用個人化頁面和作業資料，此功能就很重要。 資料會儲存在例項上。 因此，來自相同使用者的後續要求必須傳回至該執行個體，否則資料會遺失。

由於粘滯連接限制了Dispatcher優化請求的能力，因此您應僅在必要時使用此方法。 您可以指定包含「嚴格」檔案的檔案夾，如此可確保該檔案夾中的所有檔案都會在使用者的相同執行個體上處理。

### 我是否可同時使用黏著連線和快取？

對於使用嚴格連線的大部分頁面，您應關閉快取。 否則，不論作業內容為何，頁面的相同例項都會顯示給所有使用者。

對於某些應用程式，可同時使用嚴格連線和快取。 例如，如果您顯示將資料寫入作業的表單，則可同時使用黏著連線和快取。

### Dispatcher和AEM Publish實例是否駐留在同一台物理電腦上？

是的，如果機器足夠強大。 不過，建議您在不同的電腦上設定Dispatcher和AEM Publish執行個體。

通常， Publish實例駐留在防火牆內，而Dispatcher駐留在DMZ內。 如果您決定將Publish實例和Dispatcher都放在同一台物理電腦上，請確保防火牆設定禁止從外部網路直接訪問Publish實例。

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

Dispatcher會刪除名稱與CQ-Handle標題值相符的快取檔案和檔案夾。 例如，`/content/geomtrixx-outdoors/en`的CQ-Handle符合下列項目：

在geometrixx-outdoors目錄中命名的所有檔案（任何副檔名）
en目錄下名為`_jcr_content`的任何目錄（如果存在，則包含頁面子節點的快取轉譯）
僅當`CQ-Action`是`Delete`或`Deactivate`時，才刪除目錄en。

有關此主題的詳細資訊，請參見[手動使Dispatcher Cache](page-invalidate.md)失效。

### 如何實作權限相關快取？

請參閱[快取保全內容](permissions-cache.md)頁面。

### 如何保護Dispatcher和CQ實例之間的通信安全？

請參閱[Dispatcher Security Checklist](security-checklist.md)和[ AEM Security Checklist](https://helpx.adobe.com/experience-manager/6-4/sites/administering/using/security-checklist.html)頁面。

### Dispatcher issue `jcr:content`已更改為`jcr%3acontent`

**問題**:我們最近在調度器級別遇到了一個問題，其中ajax調用中有一個是從CQ儲存庫中獲取一些資料， `jcr:content` 而它被編碼到 `jcr%3acontent` 導致錯誤的結果集。

**答案**:請使 `ResourceResolver.map()` 用方法取得要使用／發出的「好記」URL，並解決Dispatcher的快取問題。map()方法將`:`冒號編碼為底線，resolve()方法會將它們解碼回SLING JCR可讀格式。您需要使用map()方法來產生Ajax呼叫中使用的URL。

進一步閱讀：[https://sling.apache.org/documentation/the-sling-engine/mappings-for-resource-resolution.html#namespace-mangling](https://sling.apache.org/documentation/the-sling-engine/mappings-for-resource-resolution.html#namespace-mangling)

## 刷新調度程式

### 如何在發佈實例上配置Dispatcher flush代理？

請參見[Replication](https://helpx.adobe.com/content/help/en/experience-manager/6-4/sites/deploying/using/replication.html#ConfiguringyourReplicationAgents)頁。

### 如何診斷Dispatcher刷新問題？

[請參閱此解答下](https://helpx.adobe.com/content/help/en/experience-manager/kb/troubleshooting-dispatcher-flushing-issues.html) 列問題的疑難排解文章：

* 如何對Dispatcher快取中未保存任何內容的情況進行調試？
* 如何對未更新快取檔案的情況進行除錯？
* 如何調試與Dispatcher刷新無關的情況？

如果刪除操作導致Dispatcher刷新，[請在此社區部落格中使用Sensei Martin](https://mkalugin-cq.blogspot.in/2012/04/i-have-been-working-on-following.html)的解決方法。

### 如何從Dispatcher快取刷新DAM資產？

您可以使用「鏈複製」功能。  啟用此功能後，當從作者處收到複製時，調度程式刷新代理會發送刷新請求。

若要啟用它：

1. [請依照此處的](page-invalidate.md#invalidating-dispatcher-cache-from-a-publishing-instance) 步驟，在發佈時建立沖洗代理
1. 轉到這些代理的每個配置，並在&#x200B;**Triggers**&#x200B;頁籤上選中&#x200B;**On Receive**&#x200B;框。

## 其他

Dispatcher如何確定文檔是否是最新的？
要確定文檔是否是最新的，Dispatcher將執行以下操作：

它會檢查文件是否會自動失效。如果不會，則可將該文件視為最新版本。如果文件已設定為會自動失效，Dispatcher 會檢查其是否比最後一次可用變更來得舊或更新。如果版本較舊，Dispatcher 會請求來自 AEM 例項的最新版本，並取代快取中的版本。

### Dispatcher如何傳回檔案？

您可以使用[Dispatcher configuration](dispatcher-configuration.md)檔案`dispatcher.any`來定義Dispatcher是否快取文檔。 Dispatcher 會根據可快取文件清單來檢查請求。如果文件不在此清單中，Dispatcher 會請求 AEM 例項的文件。

`/rules`屬性控制根據文檔路徑快取哪些文檔。 無論`/rules`屬性如何，Dispatcher都不會在下列情況下快取文檔：

* 如果請求URI包含問號`(?)`。
* 這通常表示動態頁面，例如不需要快取的搜尋結果。
* 缺少檔案副檔名。
* Web 伺服器需要副檔名來判斷文件類型 (MIME 類型)。
* 驗證標頭已設定 (這可以設定)
* 如果AEM例項以下列標題回應：
   * 無快取
   * 無商店
   * must-revalidate

Dispatcher會將快取檔案儲存在Web伺服器上，就像是靜態網站的一部分。 如果用戶請求快取的文檔，Dispatcher將檢查該文檔是否存在於Web伺服器的檔案系統中。 如果是，Dispatcher將返回文檔。 如果沒有，Dispatcher會從AEM例項請求檔案。

>[!NOTE]
>
>GET 或 HEAD (用於 HTTP 標頭) 方法可讓 Dispatcher 快取。有關響應標頭快取的其他資訊，請參見[快取HTTP響應標頭](dispatcher-configuration.md#caching-http-response-headers)部分。

### 我是否可以在設定中實施多個調度程式？

是. 在此情況下，請確定兩個「調度程式」都能直接存取AEM網站。 Dispatcher無法處理來自其他Dispatcher的請求。

