---
title: Dispatcher熱門問題
seo-title: AEM Dispatcher的常見問題
description: AEM Dispatcher的常見問題
seo-description: AdobeAEM Dispatcher的常見問題
exl-id: 4dcc7318-aba5-4b17-8cf4-190ffefbba75
source-git-commit: 3a0e237278079a3885e527d7f86989f8ac91e09d
workflow-type: tm+mt
source-wordcount: '1644'
ht-degree: 13%

---

# AEM Dispatcher熱門問題常見問題集

![設定 Dispatcher](assets/CQDispatcher_workflow_v2.png)

## 簡介

### 什麼是Dispatcher?

Dispatcher是Adobe Experience Manager的快取和/或負載平衡工具，有助於實現快速且動態的Web製作環境。 對於快取，Dispatcher是HTTP伺服器（例如Apache）的一部分，其目的是盡可能儲存（或「快取」）靜態網站內容，並盡可能少地存取網站的配置引擎。 在負載平衡角色中，Dispatcher會在不同的AEM例項（轉譯）間分發使用者請求（負載）。

針對快取，Dispatcher模組會使用Web伺服器的功能來提供靜態內容。 Dispatcher會將快取的檔案置於Web伺服器的檔案根目錄中。

### Dispatcher如何執行快取？

Dispatcher會使用Web伺服器的功能來提供靜態內容。 Dispatcher會將快取的檔案儲存在Web伺服器的檔案根目錄中。 Dispatcher 有兩種主要方法，用於在對網站進行更改時更新快取內容。

* **內容更新**&#x200B;會移除已變更的頁面，以及與其直接相關的檔案。
* **自動失效**&#x200B;功能會自動讓那些在更新後可能過期的快取失效。例如，它會有效將相關頁面標示為過期，而不會刪除任何項目。

### 負載平衡有哪些好處？

Load Balancing將用戶請求（負載）分發到多個AEM實例。以下清單描述了負載平衡的優點：

* **提高處理能力**:在實務中，這表示Dispatcher會與多個AEM例項共用檔案請求。因為每個執行個體要處理的檔案較少，因此您的回應時間會更快。 Dispatcher 會保留每個文件類別的內部統計資料，以便能夠估計負載並有效率地分配查詢。
* **增加故障保護範圍**:如果Dispatcher未收到來自例項的回應，則會自動將要求轉送至另一個例項。因此，如果某個例項無法使用，唯一的影響是網站速度變慢，與失去的運算能力成正比。

>[!NOTE]
>
>如需詳細資訊，請參閱[Dispatcher綜覽頁面](dispatcher.md)

## 安裝和配置

### 我該從哪裡下載Dispatcher模組？

您可以從[Dispatcher發行說明](release-notes.md)頁面下載最新的Dispatcher模組。

### 如何安裝Dispatcher模組？

請參閱[安裝Dispatcher](dispatcher-install.md)頁面

### 如何配置Dispatcher模組？

請參閱[設定Dispatcher](dispatcher-configuration.md)頁面。

### 如何為製作例項設定Dispatcher?

如需詳細步驟，請參閱[搭配使用Dispatcher與Author例項](dispatcher.md#using-a-dispatcher-with-an-author-server) 。

### 如何使用多個網域來設定Dispatcher?

如果網域符合下列條件，您就可以使用多個網域來設定CQ Dispatcher:

* 這兩個網域的Web內容都儲存在單一AEM存放庫中
* Dispatcher快取中的檔案可針對每個網域分別失效

如需詳細資訊，請參閱[使用具有多個網域的Dispatcher](dispatcher-domains.md) 。

### 如何設定Dispatcher，讓來自使用者的所有請求都路由至相同的Publish例項？

您可以使用[黏著連線](dispatcher-configuration.md#identifying-a-sticky-connection-folder-stickyconnectionsfor)功能，確保在AEM的相同例項上處理使用者的所有檔案。 如果您使用個人化頁面和工作階段資料，此功能就很重要。 資料會儲存在例項上。 因此，來自相同使用者的後續請求必須傳回至該例項，否則資料會遺失。

由於黏著連線會限制Dispatcher最佳化請求的能力，因此您應僅在必要時使用此方法。 您可以指定包含「黏著」檔案的資料夾，如此可確保該資料夾中的所有檔案都會在使用者的相同例項上處理。

### 我可以同時使用黏著連線和快取嗎？

對於大部分使用黏著連線的頁面，您應關閉快取。 否則，無論工作階段內容為何，都會向所有使用者顯示相同的頁面例項。

對於某些應用程式，可同時使用黏著連線和快取。 例如，如果顯示將資料寫入工作階段的表單，您可以同時使用黏著連線和快取。

### Dispatcher和AEM Publish例項是否可駐留在相同的實體電腦上？

是的，如果機器足夠強大。 不過，建議您在不同的電腦上設定Dispatcher和AEM Publish例項。

通常， Publish實例駐留在防火牆內，而Dispatcher駐留在DMZ中。 如果您決定將Publish例項和Dispatcher同時放在同一台物理電腦上，請確定防火牆設定禁止從外部網路直接存取Publish例項。

### 我是否只能快取具有特定副檔名的檔案？

是. 例如，如果您只想快取GIF檔案，請在dispatcher.any設定檔案的「快取」區段中指定*.gif。

### 如何從快取中刪除檔案？

您可以使用HTTP要求從快取中刪除檔案。 收到HTTP要求時，Dispatcher會從快取中刪除檔案。 Dispatcher只有在收到頁面的用戶端請求時，才會再次快取檔案。 以此方式刪除快取檔案對於不太可能同時收到相同頁面請求的網站來說是適當的。

HTTP要求具有下列語法：

```
POST /dispatcher/invalidate.cache HTTP/1.1
CQ-Action: Activate
CQ-Handle: path-pattern
Content-Length: 0
```

Dispatcher會刪除名稱與CQ-Handle標題值相符的快取檔案和資料夾。 例如，`/content/geomtrixx-outdoors/en`的CQ-Handle符合下列項目：

在geometrixx-outdoors目錄中命名為en的所有檔案（任何副檔名的）
en目錄下名為`_jcr_content`的任何目錄（如果存在，則包含頁面子節點的快取呈現）
只有`CQ-Action`為`Delete`或`Deactivate`時，才會刪除目錄en。

如需此主題的詳細資訊，請參閱[手動使Dispatcher快取失效](page-invalidate.md)。

### 如何實作需要權限的快取？

請參閱「[快取安全內容](permissions-cache.md)」頁。

### 如何保護Dispatcher與CQ例項之間的通訊安全？

請參閱[Dispatcher安全性檢查清單](security-checklist.md)和[AEM安全性檢查清單](https://helpx.adobe.com/experience-manager/6-4/sites/administering/using/security-checklist.html)頁面。

### Dispatcher問題`jcr:content`已變更為`jcr%3acontent`

**問題**:我們最近在Dispatcher層級遇到問題，其中ajax呼叫從CQ存放庫取得某些資料， `jcr:content` 而經過編碼後 `jcr%3acontent` 產生錯誤的結果集。

**答案**:請使 `ResourceResolver.map()` 用方法來取得要使用/發出的「易記」URL，以及從和取得請求，以解決Dispatcher的快取問題。map()方法將`:`冒號編碼為底線，resolve()方法會將它們解碼回SLING JCR可讀格式。您需要使用map()方法來產生Ajax呼叫中使用的URL。

進一步閱讀：[https://sling.apache.org/documentation/the-sling-engine/mappings-for-resource-resolution.html#namespace-mangling](https://sling.apache.org/documentation/the-sling-engine/mappings-for-resource-resolution.html#namespace-mangling)

## 排清Dispatcher

### 如何在發佈執行個體上配置Dispatcher排清代理？

請參閱[復寫](https://helpx.adobe.com/content/help/en/experience-manager/6-4/sites/deploying/using/replication.html#ConfiguringyourReplicationAgents)頁面。

### 如何疑難排解Dispatcher排清問題？

[請參閱本疑難排](https://helpx.adobe.com/content/help/en/experience-manager/kb/troubleshooting-dispatcher-flushing-issues.html) 解文章，解答下列問題：

* 如果Dispatcher快取中未儲存任何內容，我應如何除錯？
* 如何對快取檔案未更新的情況進行除錯？
* 如果沒有與Dispatcher排清相關的項目運作正常，我該如何除錯？

如果「刪除」操作導致Dispatcher排清，請[在此社群部落格文章中使用Sensei Martin](https://mkalugin-cq.blogspot.in/2012/04/i-have-been-working-on-following.html)的因應措施。

### 如何從Dispatcher快取排清DAM資產？

您可以使用「鏈式復寫」功能。  啟用此功能後，當從作者收到復寫時，Dispatcher排清代理會傳送排清請求。

若要啟用：

1. [請依照下列步](page-invalidate.md#invalidating-dispatcher-cache-from-a-publishing-instance) 驟，在發佈時建立排清代理
1. 轉到這些代理的每個配置，然後在&#x200B;**Triggers**&#x200B;頁簽上選中&#x200B;**On Receive**&#x200B;框。

## 其他

Dispatcher如何判斷檔案是否為最新狀態？
為了判斷檔案是否為最新狀態，Dispatcher會執行下列動作：

它會檢查文件是否會自動失效。如果不會，則可將該文件視為最新版本。如果文件已設定為會自動失效，Dispatcher 會檢查其是否比最後一次可用變更來得舊或更新。如果版本較舊，Dispatcher 會請求來自 AEM 例項的最新版本，並取代快取中的版本。

### Dispatcher如何傳回檔案？

您可以使用[Dispatcher設定](dispatcher-configuration.md)檔案`dispatcher.any`來定義Dispatcher是否快取檔案。 Dispatcher 會根據可快取文件清單來檢查請求。如果文件不在此清單中，Dispatcher 會請求 AEM 例項的文件。

`/rules`屬性控制根據文檔路徑快取哪些文檔。 無論`/rules`屬性為何，Dispatcher在下列情況下都不會快取檔案：

* 如果請求URI包含問號`(?)`。
* 這通常表示有動態頁面，例如不需要快取的搜尋結果。
* 缺少檔案副檔名。
* Web 伺服器需要副檔名來判斷文件類型 (MIME 類型)。
* 驗證標頭已設定 (這可以設定)
* 如果AEM例項使用下列標題回應：
   * 無快取
   * 無商店
   * 必須重新驗證

Dispatcher會將快取的檔案儲存在Web伺服器上，就像它們是靜態網站的一部分。 如果使用者要求快取的檔案，Dispatcher會檢查該檔案是否存在於Web伺服器的檔案系統中。 如果是，Dispatcher會傳回檔案。 如果沒有，Dispatcher會從AEM例項請求檔案。

>[!NOTE]
>
>GET 或 HEAD (用於 HTTP 標頭) 方法可讓 Dispatcher 快取。有關響應標頭快取的詳細資訊，請參閱[快取HTTP響應標頭](dispatcher-configuration.md#caching-http-response-headers)部分。

### 我可以在設定中實作多個Dispatcher嗎？

是. 在這種情況下，請確定兩個Dispatcher都可以直接存取AEM網站。 Dispatcher無法處理來自其他Dispatcher的請求。
