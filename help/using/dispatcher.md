---
title: Dispatcher概述
seo-title: Adobe AEM Dispatcher概觀
description: 本文提供Dispatcher的一般概述。
seo-description: 本文提供Adobe Experience Manager Dispatcher的一般概觀。
uuid: 71766f86-5e91-446b-a078-061b179d090d
pageversionid: '1193211344162'
topic-tags: dispatcher
content-type: 引用
discoiquuid: 1d449ee2-4cdd-4b7a-8b4e-7e6fc0a1d7ee
translation-type: tm+mt
source-git-commit: de6a513baf3e6b1a1463a442fa840e59f2196e8e

---


# Dispatcher概述 {#dispatcher-overview}

>[!NOTE]
>
>Dispatcher版本獨立於AEM。 如果您遵循Dispatcher檔案的連結，且該連結內嵌於舊版AEM的檔案中，您可能會被重新導向至本頁面。

Dispatcher是Adobe Experience manager的快取和／或負載平衡工具。 使用AEM的Dispatcher也有助於保護您的AEM伺服器不受攻擊。 因此，您可以搭配使用Dispatcher與企業級Web伺服器，以提高AEM例項的安全性。

部署Dispatcher的過程與所選的Web伺服器和作業系統平台無關：

1. 瞭解Dispatcher（此頁）。 此外，請參 [閱有關dispatcher的常見問題](https://helpx.adobe.com/experience-manager/using/dispatcher-faq.html)。
1. 根據網 [頁伺服器檔案](https://helpx.adobe.com/experience-manager/6-3/sites/deploying/using/technical-requirements.html) ，安裝支援的網頁伺服器。

1. [在Web伺服器上安裝Dispatcher](dispatcher-install.md) 模組，並相應地配置Web伺服器。
1. [配置Dispatcher](dispatcher-configuration.md) （dispatcher.any檔案）。

1. [設定AEM](page-invalidate.md) ，讓內容更新使快取失效。

>[!NOTE]
>
>若要進一步瞭解Dispatcher如何與AEM搭配運作，請參 [閱2017年7月向AEM社群專家提問](https://bit.ly/ATACE0717)。

視需要使用下列資訊：

* [Dispatcher Security Checklist](security-checklist.md)
* [Dispatcher知識庫](https://helpx.adobe.com/cq/kb/index/dispatcher.html)
* [針對快取效能最佳化網站](https://helpx.adobe.com/experience-manager/6-4/sites/deploying/using/configuring-performance.html)
* [將Dispatcher與多個域一起使用](dispatcher-domains.md)
* [將SSL與Dispatcher搭配使用](dispatcher-ssl.md)
* [實作權限相關快取](permissions-cache.md)
* [Dispatcher問題疑難排解](dispatcher-troubleshooting.md)
* [Dispatcher熱門問題常見問答集](dispatcher-faq.md)

>[!NOTE]
>
>**Dispatcher最常用的用途是從AEM** publish實例快取回應 ****，以提高您對外發佈網站的回應速度與安全性。 大部分討論都集中在此案例上。
>
>但是，Dispatcher也可用來提高您作者例項的回 **應速度**，尤其是當您有大量使用者編輯和更新您的網站時。 有關此案例的詳細資訊，請 [參閱以下的Using a Dispatcher with an Author Server](#using-a-dispatcher-with-an-author-server)。

## 為何使用Dispatcher來實作快取？ {#why-use-dispatcher-to-implement-caching}

Web發佈有兩種基本方式：

* **靜態Web伺服器**:如Apache或IIS，雖然簡單但快速。
* **內容管理伺服器**:它提供動態、即時、智慧的內容，但需要更多的運算時間和其他資源。

Dispatcher有助於實現既快速又動態的環境。 它可當成靜態HTML伺服器（例如Apache）的一部分，其目的是：

* 以靜態網站的形式，儲存（或「快取」）盡可能多的網站內容
* 盡可能少地存取版面引擎。

這意味著：

* **靜態內容** ，處理速度和簡單性完全與靜態Web伺服器相同；*此外，您還可以使用靜態Web伺服器可用的管理和安全工具*。

* **動態內容** 會視需要產生，而不會讓系統變慢，而非完全必要。

Dispatcher包含根據動態網站內容產生和更新靜態HTML的機制。 您可以詳細指定哪些檔案儲存為靜態檔案，哪些檔案一律會動態產生。

本節說明其背後的原則。

### 靜態Web伺服器 {#static-web-server}

![](assets/chlimage_1-3.png)

靜態Web伺服器（例如Apache或IIS）會為網站的訪客提供靜態HTML檔案。 靜態頁面建立一次，因此每個請求都會傳送相同的內容。

該過程非常簡單，因此非常有效。 如果訪客要求檔案（例如HTML頁面），則通常會直接從記憶體擷取檔案，往壞了說，則會從本機磁碟讀取檔案。 靜態Web伺服器已有相當長的一段時間，因此有各種管理和安全管理工具，它們與網路基礎架構非常整合。

### 內容管理伺服器 {#content-management-servers}

![](assets/chlimage_1-4.png)

如果您使用Content Management Server（例如AEM），進階版面引擎會處理訪客的請求。 引擎會從儲存庫讀取內容，並結合樣式、格式和存取權限，將內容轉換為符合訪客需求和權限的檔案。

這可讓您建立更豐富的動態內容，進而提高網站的彈性和功能。 不過，版面引擎比靜態伺服器需要更強大的處理能力，因此，如果有許多訪客使用系統，此設定可能會有所放緩。

## Dispatcher如何執行快取 {#how-dispatcher-performs-caching}

![](assets/chlimage_1-5.png)

**Cache Directory** （快取目錄）Dispatcher模組使用Web伺服器提供靜態內容的能力。 Dispatcher會將快取的檔案置於Web伺服器的檔案根目錄中。

>[!NOTE]
>
>當缺少HTTP標頭快取的設定時，Dispatcher只會儲存頁面的HTML程式碼——不會儲存HTTP標頭。 如果您在網站中使用不同的編碼，可能會發生此問題，因為這些編碼可能會遺失。 要啟用HTTP標頭快取，請參 [閱配置Dispatcher快取。](https://helpx.adobe.com/experience-manager/dispatcher/using/dispatcher-configuration.html)

>[!NOTE]
>
>在網路連接儲存(NAS)上查找Web伺服器的文檔根目錄會導致效能降低。 此外，當位於NAS上的文檔根在多個Web伺服器之間共用時，執行複製操作時可能會發生間歇鎖定。

>[!NOTE]
>
>Dispatcher將快取的文檔以等於請求URL的結構儲存。
>
>檔案名的長度可能受到作業系統級別的限制；例如，如果您有包含許多選擇器的URL。

### 快取方法

Dispatcher有兩種主要方法，用於在對網站進行更改時更新快取內容。

* **內容更新** ：移除已變更的頁面，以及直接與之關聯的檔案。
* **自動失效** (Auto-Invalidation)會自動使更新後可能過期的快取部分失效。 例如，它有效地將相關頁面標示為過期，而不會刪除任何內容。

### 內容更新

在內容更新中，一或多份AEM檔案會變更。 AEM會傳送匯集請求給Dispatcher,Dispatcher會相應更新快取：

1. 它會從快取中刪除修改過的檔案。
1. 它會從快取中刪除以相同句柄開頭的所有檔案。 例如，如果檔案/en/index.html已更新，則所有以/tw/index開頭的檔案。 刪除。 此機制可讓您設計快取效率的網站，尤其是在圖片導覽方面。
1. 它 *觸及* 所謂的 **statfile**;這會更新statfile的時間戳記，以指出上次變更的日期。

應注意以下幾點：

* 內容更新通常與編寫系統搭配使用，而製作系統「知道」必須取代的內容。
* 受內容更新影響的檔案會移除，但不會立即取代。 下次請求此類檔案時，Dispatcher會從AEM例項擷取新檔案，並將其置於快取中，從而覆寫舊內容。
* 通常，自動生成並合併來自頁面的文本的圖片會以相同的句柄開始儲存在圖片檔案中——從而確保關聯存在以供刪除。 例如，您可以將頁面mypage.html的標題文字儲存為mypage.titlePicture.gif圖片的標題文字儲存在相同的檔案夾中。 這樣，每次更新頁面時，圖片都會自動從快取中刪除，因此您可以確定圖片始終反映頁面的當前版本。
* 您可能有數個statfile，例如每個語言資料夾一個。 如果頁面已更新，AEM會尋找下一個包含statfile的上層檔案夾，並 *接觸* 該檔案。

### 自動失效

自動失效會自動使快取的部分失效，而不會實際刪除任何檔案。 在每次內容更新時，都會觸及所謂的statfile，因此其時間戳記會反映上次的內容更新。

Dispatcher有一個檔案清單，這些檔案會自動失效。 請求該清單中的文檔時，Dispatcher會將快取文檔的日期與statfile的時間戳進行比較：

* 如果快取的文檔較新，則Dispatcher會返回它。
* 如果版本較舊，Dispatcher會從AEM例項擷取目前版本。

同樣，我們應當指出一些要點：

* 當內部關係複雜（例如HTML頁面）時，通常會使用自動失效。 這些頁面包含連結和導覽項目，因此通常必須在內容更新後進行更新。 如果您已自動產生PDF或圖片檔案，也可以選擇自動使這些檔案無效。
* 除了觸碰statfile之外，自動失效不涉及調度程式在更新時執行的任何操作。 不過，觸摸statfile會自動使快取內容過時，而不從快取中實際刪除它。

## Dispatcher如何返回文檔 {#how-dispatcher-returns-documents}

![](assets/chlimage_1-6.png)

### 確定文檔是否受快取的約束

您可以定 [義Dispatcher在配置檔案中快取哪些文檔](https://helpx.adobe.com/experience-manager/dispatcher/using/dispatcher-configuration.html)。 Dispatcher會根據可快取文檔的清單檢查請求。 如果檔案不在此清單中，Dispatcher會從AEM例項要求檔案。

在下列情 *況下* ,Dispatcher一律會直接從AEM例項要求檔案：

* 如果請求URI包含問號"?"。 這通常表示不需要快取的動態頁面，例如搜尋結果。
* 缺少檔案副檔名。 Web伺服器需要擴充功能來判斷檔案類型（MIME類型）。
* 驗證標題已設定（可進行設定）

>[!NOTE]
>
>GET或HEAD（用於HTTP標頭）方法可由Dispatcher進行快取。 如需回應標頭快取的詳細資訊，請參 [閱快取HTTP回應標頭](https://helpx.adobe.com/experience-manager/dispatcher/using/dispatcher-configuration.html) 。

### 確定是否快取檔案

Dispatcher會將快取檔案儲存在Web伺服器上，就像是靜態網站的一部分。 如果用戶請求可快取文檔，Dispatcher將檢查該文檔是否存在於Web伺服器的檔案系統中：

* 如果文檔被快取，Dispatcher將返回該檔案。
* 如果未快取，Dispatcher會從AEM例項要求檔案。

### 確定文檔是否是最新的

要瞭解文檔是否是最新的，Dispatcher將執行兩個步驟：

1. 它會檢查檔案是否受到自動失效的影響。 否則，檔案會視為最新。
1. 如果文檔已配置為自動失效，Dispatcher將檢查其是否比上次可用更改更舊或更新。 如果版本較舊，Dispatcher會從AEM例項要求目前版本，並取代快取中的版本。

>[!NOTE]
>
>未自動失 **效的檔案** ，會保留在快取中，直到實際刪除為止；例如，透過網站上的內容更新。

## 負載平衡的優點 {#the-benefits-of-load-balancing}

「負載平衡」是將網站的計算負載分配至數個AEM例項的慣例。

![](assets/chlimage_1-7.png)

您可以獲得：

* **提高處理**&#x200B;能力在實際中，這表示Dispatcher會在數個AEM執行個體之間共用檔案請求。 由於每個例項現在處理的檔案較少，因此您的回應時間較快。 Dispatcher會保留每個文檔類別的內部統計資訊，以便能夠估計負載並高效地分發查詢。

* **增加故障安全保**&#x200B;護範圍如果Dispatcher沒有從實例接收響應，它將自動將請求中繼到另一個實例。 因此，如果某個例項無法使用，唯一的效果是網站的減速，與失去的計算能力成正比。 但是，所有服務都將繼續。

* 您也可以在同一靜態Web伺服器上管理不同的網站。

>[!NOTE]
>
>雖然負載平衡有效率地分散負載，但快取有助於降低負載。 因此，在設定負載平衡之前，請嘗試最佳化快取並降低整體負載。 良好的快取可能會提高負載平衡器的效能，或不需要進行負載平衡。

>[!CAUTION]
>
>雖然單個Dispatcher通常能夠飽和可用Publish實例的容量，但對於某些罕見的應用程式，另外平衡兩個Dispatcher實例之間的負載是有意義的。 需要仔細考慮具有多個調度程式的配置，因為額外的Dispatcher將增加可用Publish實例的負載，並可輕鬆降低大多數應用程式的效能。

## Dispatcher如何執行負載平衡 {#how-the-dispatcher-performs-load-balancing}

### 效能統計資料

Dispatcher會保留內部統計資料，瞭解AEM的每個例項處理檔案的速度。 Dispatcher會根據此資料估計在回應請求時哪個執行個體會提供最快的回應時間，因此會保留該執行個體所需的計算時間。

不同類型的請求可能具有不同的平均完成時間，因此Dispatcher允許您指定文檔類別。 然後在計算時間估計時考慮這些。 例如，您可以區分HTML頁面和影像，因為一般的回應時間可能不同。

如果您使用複雜的搜尋函式，則可建立新的搜尋查詢類別。 這可幫助Dispatcher將搜索查詢發送到響應速度最快的實例。 這可防止較慢的例項在收到數個「昂貴」的搜尋查詢時停止，而其他例項則收到「較便宜」的請求。

### 個人化內容（黏著連線）

嚴格連線可確保一個使用者的檔案都是在AEM的同一個例項上撰寫。 如果您使用個人化頁面和作業資料，這很重要。 資料會儲存在例項上，因此來自相同使用者的後續要求必須傳回該例項，否則資料會遺失。

由於自黏連線會限制Dispatcher最佳化請求的能力，因此您只應視需要使用請求。 您可以指定包含「嚴格」檔案的檔案夾，如此可確保該檔案夾中的所有檔案都是針對每位使用者在同一執行個體上所撰寫。

>[!NOTE]
>
>對於使用嚴格連線的大部分頁面，您必須關閉快取——否則，無論作業內容為何，頁面在所有使用者看來都相同。
>
>對於少 *數應用* ，可以同時使用黏著連線和快取；例如，如果顯示的表單將資料寫入會話。

## 使用多個調度程式 {#using-multiple-dispatchers}

在複雜的設定中，您可以使用多個調度程式。 例如，您可以使用：

* 一個Dispatcher，在內部網路上發佈網站
* 第二個Dispatcher，位於不同的地址下，並具有不同的安全設定，以便在Internet上發佈相同的內容。

在這種情況下，請確定每個請求只通過一個Dispatcher。 Dispatcher不處理來自其他Dispatcher的請求。 因此，請確定兩個Dispatcher都直接存取AEM網站。

## 搭配使用Dispatcher與CDN {#using-dispatcher-with-a-cdn}

內容傳送網路(CDN)（例如Akamai Edge Delivery或Amazon Cloud Front）會從接近使用者的位置傳送內容。 由此

* 加速使用者的回應時間
* 卸載伺服器

作為HTTP基礎架構元件，CDN的運作方式與Dispatcher類似：當CDN節點收到請求時，它會盡可能從其快取中提供請求（該資源可在快取中使用且有效）。 否則，它會前往下一個最靠近的伺服器，以擷取資源並快取資源，以取得進一步的請求（如果適用）。

「下一個最近的伺服器」取決於您的特定設定。 例如，在Akamai設定中，請求可採用下列路徑：

* Akamai Edge節點
* Akamai Midgres圖層
* 防火牆
* 您的負載平衡器
* Dispatcher
* AEM

在大多數情況下，Dispatcher是下一個伺服器，可能會從快取中提供檔案，並影響傳回至CDN伺服器的回應標題。

## 控制CDN快取 {#controlling-a-cdn-cache}

CDN從Dispatcher重新回遷之前，有許多方法可控制其快取資源的時間。

1. 顯式配置\
   設定CDN快取中特定資源的保存時間，視MIME類型、擴充功能、請求類型等而定。

1. 過期和快取控制標題\
   如果上游伺服器 `Expires:` 傳送， `Cache-Control:` 大部分的CDN都會遵守和HTTP標題。 這可以實現，例如使用 [mod_expires](https://httpd.apache.org/docs/2.4/mod/mod_expires.html) Apache模組。

1. 手動失效\
   CDN可讓資源透過Web介面從快取中移除。
1. 基於API的失效\
   大部分的CDN也提供REST和／或SOAP API，可讓資源從快取中移除。

在一般的AEM設定中，依擴充和／或路徑進行的設定（可透過上述第1和第2點進行）提供為經常使用的資源（例如設計影像和用戶端程式庫）設定合理快取期限的可能性。 部署新版本時，通常需要手動失效。

如果此方法用於快取受管理的內容，則表示只有在配置的快取期間到期且文檔再次從Dispatcher中提取後，最終用戶才能看到內容更改。

為了更精細的控制，基於API的失效允許您在Dispatcher快取失效時使CDN的快取失效。 您可以根據CDNs API，實作您自己的 [ContentBuilder](https://docs.adobe.com/docs/en/cq/current/javadoc/com/day/cq/replication/ContentBuilder.html) 和 [TransportHandler](https://docs.adobe.com/docs/en/cq/current/javadoc/com/day/cq/replication/TransportHandler.html) （如果API不是REST型），並設定複製代理，使用這些來使CDN的快取失效。

>[!NOTE]
>
>另請參 [閱AEM(CQ)Dispatcher Security和CDN+Browser Caching](https://www.slideshare.net/andrewmkhoury/dispatcher-caching-aemgemspart2jan2015) ，以及 [Dispatcher Caching的錄制簡報](https://docs.adobe.com/content/ddc/en/gems/dispatcher-caching---new-features-and-optimizations.html)。

## 將Dispatcher與Author Server搭配使用 {#using-a-dispatcher-with-an-author-server}

>[!CAUTION]
>
>如果您將 [AEM與Touch UI搭配使用](https://helpx.adobe.com/experience-manager/6-3/sites/developing/using/touch-ui-concepts.html) , **則不應快取** 作者實例內容。 如果為作者實例啟用了快取，則需要禁用它並刪除快取目錄的內容。 要禁用快取，應編輯文 `author_dispatcher.any` 件並按如下方 `/rule` 式修改 `/cache` 該節的屬性：

```xml
/rules
{
/0000
{ /type "deny" /glob "*"}
}
```

Dispatcher可用於作者實例的前面，以提高編寫效能。 要配置編寫Dispatcher，請執行以下操作：

1. 在Web伺服器中安裝Dispatcher(這可以是Apache或IIS web伺服器，請參 [閱安裝Dispatcher](dispatcher-install.md))。
1. 您可能希望根據正在運作的AEM發佈例項來測試新安裝的Dispatcher，以確保已取得基準正確安裝。
1. 現在，請確定Dispatcher能夠通過TCP/IP連接到您的作者實例。
1. 將sample dispatcher.any檔案替換為 [Dispatcher下載隨附的author_dispatcher.any檔案](release-notes.md#downloads)。
1. 在文字 `author_dispatcher.any` 編輯器中開啟，並進行下列變更：

   1. 變更 `/hostname` 和 `/port` 區 `/renders` 段，以指向您的作者實例。
   1. 將節 `/docroot` 更改為 `/cache` 指向快取目錄。 如果您正在搭配 [Touch UI使用AEM](https://helpx.adobe.com/experience-manager/6-3/sites/developing/using/touch-ui-concepts.html)，請參閱上述警告。
   1. 儲存變更。

1. 刪除您在上面配置的 `/cache` &gt;目 `/docroot` 錄中的所有現有檔案。
1. 重新啟動Web伺服器。

>[!NOTE]
>
>請注意，在提供的配置中 `author_dispatcher.any` ，當您安裝CQ5功能套件、修補程式或應用程式程式碼套件時，如果CQ5功能套件會影響下方的任何內容，或者 `/libs``/apps` ，則您必須刪除Dispatcher cache中這些目錄下的快取檔案，以確保下次請求時，會擷取新升級的檔案，而非舊快取檔案。

>[!CAUTION]
>
>如果您使用了先前配置的作者調度器並啟用了 *調度器刷新代理* ，則請執行以下操作：

1. 在您的AEM作者 **例項上刪除或停用作者** dispatcher的Flushing Agent。
1. 依照上述新指示，重新執行作者分派器設定。

<!--
[Author Dispatcher configuration file (Dispatcher 4.1.2 or later)](assets/author_dispatchernew.any)
-->
<!--[!NOTE]
>
>A related knowledge base article can be found here:  
>[How to configure the dispatcher in front of an authoring environment](https://helpx.adobe.com/cq/kb/HowToConfigureDispatcherForAuthoringEnvironment.html)
-->
