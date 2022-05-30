---
title: Dispatcher 總覽
seo-title: Adobe AEM Dispatcher Overview
description: 了解使用 Dispatcher 來改善 AEM Cloud Service 安全性、快取行為及其他等。
seo-description: This article provides a general overview of Adobe Experience Manager Dispatcher.
uuid: 71766f86-5e91-446b-a078-061b179d090d
pageversionid: 1193211344162
topic-tags: dispatcher
content-type: reference
discoiquuid: 1d449ee2-4cdd-4b7a-8b4e-7e6fc0a1d7ee
exl-id: c9266683-6890-4359-96db-054b7e856dd0
source-git-commit: 85a7f38ecb5f5fe620ff51a7cdbe29f546559288
workflow-type: ht
source-wordcount: '3190'
ht-degree: 100%

---

# Dispatcher 總覽 {#dispatcher-overview}

>[!NOTE]
>
>Dispatcher 版本與 AEM 無關。如果您依循了內嵌到舊版 AEM 文件中的 Dispatcher 文件的連結，您可能會被重新導向至本頁。

Dispatcher 是 Adobe Experience Manager 的快取及/或負載平衡工具，可搭配企業級網頁伺服器使用。

部署 Dispatcher 的程序與所選的網頁伺服器和作業系統平台無關：

1. 了解 Dispatcher (本頁)。 此外，也請參閱[關於 Dispatcher 的常見問題集](https://helpx.adobe.com/experience-manager/using/dispatcher-faq.html)。
1. 根據網頁伺服器文件安裝[支援的網頁伺服器](https://helpx.adobe.com/experience-manager/6-5/sites/deploying/using/technical-requirements.html)。
1. [在網頁伺服器上安裝 Dispatcher 模組](dispatcher-install.md)，並相應地設定網頁伺服器。
1. [設定 Dispatcher](dispatcher-configuration.md) (dispatcher.any 檔案)。
1. [設定 AEM](page-invalidate.md)，如此一來內容更新即可讓快取失效。

>[!NOTE]
>
>若要更了解 Dispatcher 如何搭配 AEM 使用：
>
>* 請觀看 [2017 年 7 月的向 AEM 社群專家詢問](https://bit.ly/ATACE0717)。
>* 存取[此存放庫](https://github.com/adobe/aem-dispatcher-experiments)。 其中包含一系列「帶回家」實驗室格式的實驗。



視需要使用下列資訊：

* [Dispatcher 安全性檢查清單](security-checklist.md)
* [Dispatcher 知識庫](https://helpx.adobe.com/cq/kb/index/dispatcher.html)
* [將網站快取效能最佳化](https://helpx.adobe.com/experience-manager/6-4/sites/deploying/using/configuring-performance.html)
* [在多個網域中使用 Dispatcher](dispatcher-domains.md)
* [搭配 Dispatcher 使用 SSL](dispatcher-ssl.md)
* [實作權限敏感型快取](permissions-cache.md)
* [Dispatcher 疑難排解](dispatcher-troubleshooting.md)
* [Dispatcher 熱門問題常見問題集](dispatcher-faq.md)

>[!NOTE]
>
>**Dispatcher 最常見的用法是快取來自 AEM** Publish 執行個體的回應&#x200B;****，以提高您對外發佈網站的回應速度與安全性。大多數的討論均強調此用途。
>
>但是，Dispatcher 也可用來提高您&#x200B;**編寫執行個體**&#x200B;的回應速度，尤其是如果您有大量使用者編輯和更新您的網站時特別實用。如需此情況特有的詳細資訊，請參閱下方的[搭配撰寫伺服器使用 Dispatcher](#using-a-dispatcher-with-an-author-server)。

## 為何要使用 Dispatcher 實作快取？ {#why-use-dispatcher-to-implement-caching}

網路出版的基本方式有兩種:

* **靜態網頁伺服器**：例如 Apache 或 IIS，都非常簡便而快速。
* **內部管理伺服器**：提供動態且即時的智慧型內容，但是需要較多的運算時間和其他資源。

Dispatcher 有助於實現快速和動態的環境。這可當成是靜態 HTML 伺服器 (例如 Apache) 的一部分，但其目的是：

* 以靜態網站的形式，盡可能儲存 (或「快取」) 網站內容
* 儘可能減少存取版面引擎。

這表示:

* **處理靜態內容**&#x200B;的速度和簡便性與在靜態網頁伺服器上完全相同；*此外，您可以使用可用於靜態網頁伺服器的管理和安全工具*。

* **動態內容**&#x200B;會視需要產生，除非絕對需要，否則並不會讓系統變慢。

Dispatcher 包含的機制可根據動態網站上的內容來產生和更新靜態的 HTML。您可以詳細指定要將哪些文件儲存為靜態檔案，哪些一律動態產生。

本節說明這背後的原理。

### 靜態網頁伺服器 {#static-web-server}

![](assets/chlimage_1-3.png)

靜態網頁伺服器 (例如 Apache 或 IIS) 會為網站的訪客提供靜態 HTML 檔案。靜態頁面只需建立一次，之後會針對每項請求傳送相同的內容。

此過程非常簡單，因此非常有效率。如果訪客要求檔案 (例如 HTML 頁面)，則通常會直接從記憶體擷取檔案，不然頂多是從本機磁碟讀取檔案。靜態網頁伺服器已上市相當長一段時間，因此已經有多種管理和安全管理工具，並且與網路基礎架構高度整合。

### 內容管理伺服器 {#content-management-servers}

![](assets/chlimage_1-4.png)

如果您使用的是內容管理伺服器 (例如 AEM)，進階版面引擎會處理訪客的請求。引擎會從存放庫讀取內容，並結合樣式、格式和存取權限，將內容轉換為符合訪客需求和權限的文件。

這可讓您建立更豐富的動態內容，進而提高網站的彈性和功能。不過，版面引擎比靜態伺服器需要更強大的處理能力，因此，如果有許多訪客使用系統，此設定可能會讓速度緩慢。

## Dispatcher 如何執行快取 {#how-dispatcher-performs-caching}

![](assets/chlimage_1-5.png)

**快取目錄**&#x200B;針對快取，Dispatcher 模組會使用網頁伺服器的功能提供靜態內容。Dispatcher 會將快取的文件置於網頁伺服器的主目錄。

>[!NOTE]
>
>當缺少 HTTP 標題快取的設定時，Dispatcher 只會儲存頁面的 HTML 程式碼，不會儲存 HTTP 標 題。如果您的網站使用不同的編碼，可能會發生問題，因為這些編碼可能會遺失。如要啟用 HTTP 標題快取，請參閱[設定 Dispatcher 快取。](https://helpx.adobe.com/experience-manager/dispatcher/using/dispatcher-configuration.html)

>[!NOTE]
>
>在網路連接儲存裝置 (NAS) 上尋找網頁伺服器的主目錄會導致效能降低。此外，當位於 NAS 上的主目錄在多部網頁伺服器之間共用時，執行複製操作時可能會發生間歇鎖定。

>[!NOTE]
>
>Dispatcher 將快取的文件以與請求的 URL 相同的結構儲存。
>
>檔案名稱的長度可能受到作業系統層級的限制，若您有包含多個選取器的 URL 就會如此。

### 快取方法

Dispatcher 有兩種主要方法，用於在對網站進行更改時更新快取內容。

* **內容更新**&#x200B;會移除已變更的頁面，以及與其直接相關的檔案。
* **自動失效**&#x200B;功能會自動讓那些在更新後可能過期的快取失效。即此功能會有效率地將相關頁面標示為過期，而不會刪除任何內容。

### 內容更新

每次內容更新會變更一或多個 AEM 文件。AEM 會傳送整合請求給 Dispatcher，Dispatcher 會隨之更新快取:

1. 它會從快取中刪除修改過的檔案。
1. 它會從快取中刪除以相同名稱開頭的所有檔案。例如如果 file /en/index.html 已更新，則所有以 /en/index 開頭的檔案都會遭到刪除。此機制可讓您設計快取效率高的網站，尤其是在圖片導覽方面。
1. 它&#x200B;*會接觸到*&#x200B;所謂的 **statfile**；進而更新 statfile 的時間戳記，以顯示上次變更的日期。

請注意下列幾點:

* 內容更新通常與編寫系統搭配使用，因而「知道」必須取代的內容。
* 受內容更新影響的檔案會遭到移除，但不會立即替換。下次請求此類檔案時，Dispatcher 會從 AEM 執行個體擷取新檔案，並將其置於快取中，藉此來覆寫舊內容。
* 通常，包含頁面文字的自動產生圖片，會儲存在開頭為相同名稱的圖片檔案中，因而可確保要刪除的檔案的關聯性存在。例如您可以將頁面 mypage.html 的標題文字儲存為相同檔案夾中名稱為 mypage.titlePicture.gif 的圖片。如此一來，每次更新頁面時，圖片都會自動從快取中刪除，因此您可以確定圖片會一律反映頁面的最新版本。
* 您可能有數個 statfile，例如每個語言檔案夾一個。如果頁面已更新，AEM 會尋找下一個包含 statfile 的上層檔案夾，並&#x200B;*接觸*&#x200B;該檔案。

### 自動失效

自動失效功能會自動讓部分的快取失效，而不會實際刪除任何檔案。在每次內容更新時，都會接觸到所謂的 statfile，因此其時間戳記都會顯示最後一次的內容更新。

Dispatcher 有一個檔案清單，這些檔案會自動失效。請求該清單中的文件時，Dispatcher 會將快取文件的日期與 statfile 的時間戳相比較:

* 如果快取的文件較新，則 Dispatcher 會傳回快取的文件。
* 如果快取的文件版本較舊，則 Dispatcher 會從 AEM 執行個體擷取最新版本。

同樣請您注意下列幾點：

* 當內部關係複雜 (例如 HTML 頁面) 時，通常會使用自動失效。這些頁面包含連結和導覽項目，因此通常必須在內容更新後加以更新。如果您已自動產生 PDF 或圖片檔，也可以選擇自動使這些檔案無效。
* 除了會接觸到 statfile，自動失效不涉及 Dispatcher 在更新時執行的任何操作。不過，觸及 statfile 會自動使快取內容逾時，而不會從快取中實際刪除檔案。

## Dispatcher 如何傳回文件 {#how-dispatcher-returns-documents}

![](assets/chlimage_1-6.png)

### 判斷文件是否受限於快取

您可以[定義 Dispatcher 會在設定檔案中快取哪些文件](https://helpx.adobe.com/experience-manager/dispatcher/using/dispatcher-configuration.html)。 Dispatcher 會根據可快取文件清單來檢查請求。如果文件不在此清單中，Dispatcher 會請求 AEM 執行個體的文件。

在下列情況下，** Dispatcher 一律會直接從 AEM 執行個體要求文件:

* 如果請求 URI 包含問號「?」。這通常表示這是一個不需要快取的動態頁面，例如搜尋結果。
* 缺少副檔名。 網頁伺服器需要副檔名來判斷文件類型 (MIME 類型)。
* 驗證標題已設定 (這可以設定)

>[!NOTE]
>
>Dispatcher 可快取 GET 或 HEAD (用於 HTTP 標題) 方法。 如需有關回應標題快取的其他資訊，請參閱[快取 HTTP 回應標題](https://helpx.adobe.com/experience-manager/dispatcher/using/dispatcher-configuration.html)一節。

### 確定是否已快取文件

Dispatcher 會將快取的檔案儲存在網頁伺服器上，就像是靜態網站的一部分一樣。如果使用者請求可快取文件，Dispatcher 會將檢查該文件是否存在於網頁伺服器的檔案系統中：

* 如果已快取該文件，則 Dispatcher 會將檔案傳回。
* 如果未快取，則 Dispatcher 會從 AEM 執行個體要求該文件。

### 確定文件是否為最新版本

為了要瞭解文件是否為最新版本，Dispatcher 會執行下列兩個步驟:

1. 它會檢查文件是否會自動失效。如果不會，則將該文件視為最新版本。
1. 如果文件已設定為會自動失效，Dispatcher 會檢查其是否比最後一次可用變更來得舊或更新。如果版本較舊，Dispatcher 會請求來自 AEM 執行個體的最新版本，並取代快取中的版本。

>[!NOTE]
>
>未&#x200B;**自動失效**&#x200B;的文件會保留在快取中，直到實際上遭到刪除 (例如透過網站上的內容更新) 為止。

## 負載平衡的優點 {#the-benefits-of-load-balancing}

「負載平衡」是將網站的運算負載分配至數個 AEM 執行個體的作法。

![](assets/chlimage_1-7.png)

其優點如下: 

* **提高處理能力**&#x200B;在實務中，這表示 Dispatcher 會將文件請求在數個 AEM 執行個體之間分享。 由於每個執行個體現在要處理的文件數量較少，因此您的回應時間就會變快。Dispatcher 會保留每個文件類別的內部統計資料，以便能夠估計負載並有效率地分配查詢。

* **增加防故障的涵蓋範圍**&#x200B;如果 Dispatcher 沒有收到來自執行個體的回應，就會自動將請求轉送給其他執行個體之一。 因此，如果某個執行個體無法使用，唯一的影響是網站速度變慢，與失去的運算能力成正比。但是所有服務都將繼續。

* 您也可以從同一部靜態網頁伺服器上管理不同的網站。

>[!NOTE]
>
>雖然負載平衡可以有效率地分散負載，但快取可有助於降低負載。因此，在設定負載平衡之前，請嘗試最佳化快取並降低整體的負載。良好的快取可以提高負載平衡器的效能，或不需要進行負載平衡。

>[!CAUTION]
>
>雖然單一 Dispatcher 通常能夠讓可用的 Publish 執行個體的容量達到飽和，但對於某些罕見的應用程式而言，額外平衡兩個 Dispatcher 執行個體的負載是有必要的。設定多個 Dispatcher 前請先詳加考量，因為額外的 Dispatcher 將增加可用的 Publish 執行個體的負載，並且很容易就可能會降低大多數應用程式的效能。

## Dispatcher 如何執行負載平衡 {#how-the-dispatcher-performs-load-balancing}

### 效能統計資料

Dispatcher 會保留內部統計資料，瞭解 AEM 每個執行個體處理檔案的速度。Dispatcher 會根據此資料，估計在回應請求時哪個執行個體可提供最快的回應時間，藉此來保留該執行個體所需的運算時間。

不同類型的請求可能會有不同的平均完成時間，因此 Dispatcher 允許您指定文件類別。然後在運算時間估計值時加以考量。例如您可以區分 HTML 頁面和影像，因為此二者的一般回應時間會有所不同。

如果您使用複雜的搜尋功能，則可建立新的搜尋查詢類別。這可幫助 Dispatcher 將搜索查詢傳送給回應速度最快的執行個體。藉此來避免速度較慢的執行個體收到數個「昂貴」的搜尋查詢時停頓下來，而其他執行個體反而收到「較便宜」的請求。

### 個人化內容 (黏著連線)

黏著連線可確保同一個使用者的文件都是在 AEM 的同一個執行個體上撰寫。如果您使用個人化頁面和工作階段資料，這一點就很重要。資料會儲存在同一執行個體上，因此來自相同使用者的後續請求都必須傳回給該執行個體，否則資料會遺失。

由於黏著連線會限制 Dispatcher 最佳化請求的功能，因此您應該視需要來使用。您可以指定包含「黏著」文件的檔案夾，如此可確保該檔案夾中的所有文件都是針對每位使用者在相同執行個體上所撰寫。

>[!NOTE]
>
>針對使用黏著連線的大部分頁面，您必須關閉快取功能，否則無論工作階段內容為何，對所有使用者而言頁面看起來都一樣。
>
>對於&#x200B;*少數*&#x200B;應用程式而言，可以同時使用黏著連線和快取；例如如果顯示將資料寫入工作階段的表單。

## 使用多個 Dispatcher {#using-multiple-dispatchers}

您可以透過複雜的設定來使用多個 Dispatcher。例如您可以使用：

* 一個 Dispatcher 在內部網路上發佈網站
* 另一個 Dispatcher 位於不同的位址下，並具有不同的安全設定，以便在網際網路上發佈相同的內容。

在這種情況下，請確定每個請求只通過一個 Dispatcher。Dispatcher 不會處理來自其他 Dispatcher 的請求。因此，請確定兩個 Dispatcher 都直接存取 AEM 網站。

## 搭配 CDN 使用 Dispatcher {#using-dispatcher-with-a-cdn}

內容傳遞網路 (CDN) (例如 Akamai Edge Delivery 或 Amazon Cloud Front) 會從接近使用者的位置傳遞內容。藉此可

* 加速使用者的回應時間
* 卸除伺服器的負載

CDN 為 HTTP 基礎架構元件，其運作方式與 Dispatcher 類似：當 CDN 節點收到請求時，會盡可能從其快取中提供請求 (該資源可在快取中取得並且有效)。否則，會前往下一個最近的伺服器，為進一步的請求擷取資源並加以快取 (如適用)。

「下一個最近的伺服器」要取決於您的特定設定。例如在 Akamai 設定中，請求可採取下列路徑:

* Akamai Edge 節點
* Akamai Midgres 層
* 防火牆
* 您的負載平衡器
* Dispatcher
* AEM

在大多數的情況下，Dispatcher 是下一個最近的伺服器，可從快取中提供檔案，並影響傳回至 CDN 伺服器的回應標題。

## 控制 CDN 快取 {#controlling-a-cdn-cache}

CDN 從 Dispatcher 重新擷取前，有許多方法可控制 CDN 快取資源的時長。

1. 明確設定\
   視 MIME 類型、副檔名、請求類型等，設定 CDN 快取保存特定資源的時長。

1. 到期日和快取控制標題\
   如果是由上游伺服器傳送，大部分的 CDN 都會遵守 `Expires:` 和 `Cache-Control:` HTTP 標題。例如，可藉由使用 [mod_expires](https://httpd.apache.org/docs/2.4/mod/mod_expires.html) Apache 模組來達成這件事。

1. 手動失效\
   CDN 可透過 Web 介面從快取中移除資源。
1. API 型失效\
   大部分的 CDN 也提供 REST 和/或 SOAP API，可從快取中移除資源。

在一般的 AEM 設定中，只要設定副檔名和/或路徑 (可透過上述第 1 和第 2 點達成)，即可為經常使用的資源 (例如設計影像和用戶端程式庫) 設定合理的快取期限。部署新版本時，通常需要手動失效。

如果將此方法用於快取受管理的內容，則表示只有在已設定的快取期限到期，並且已再次從 Dispatcher 中擷取文件後，使用者才能看到內容變更。

為了達到更精細的控制，API 型的失效允許您在 Dispatcher 快取失效時將 DN 的快取判定為失效。您可以根據 CDN API 來實作您自己的 [ContentBuilder](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/com/day/cq/replication/ContentBuilder.html) 和 [TransportHandler](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/com/day/cq/replication/TransportHandler.html) (如果 API 不是 REST 型)，並設定複寫代理程式，它將利用這兩個工具讓 CDN 的快取失效。

>[!NOTE]
>
>另請參閱 [AEM (CQ) Dispatcher 安全性和 CDN+瀏覽器快取](https://www.slideshare.net/andrewmkhoury/dispatcher-caching-aemgemspart2jan2015)，以及關於 [Dispatcher 快取](https://docs.adobe.com/content/ddc/en/gems/dispatcher-caching---new-features-and-optimizations.html)的相關錄製簡報。

## 搭配撰寫伺服器使用 Dispatcher {#using-a-dispatcher-with-an-author-server}

>[!CAUTION]
>
>如果您正在使用[具有 Touch UI 的 AEM](https://helpx.adobe.com/experience-manager/6-3/sites/developing/using/touch-ui-concepts.html)，您就&#x200B;**不**&#x200B;應該快取編寫執行個體內容。 如果已為編寫執行個體啟用快取，則需要停用並刪除快取目錄的內容。如要停用快取，應編輯`author_dispatcher.any`檔案並修改區段`/rule`屬性`/cache`如下:

```xml
/rules
{
/0000
{ /type "deny" /glob "*"}
}
```

Dispatcher 可用於編寫執行個體之前，以改善編寫效能。如要設定編寫 Dispatcher，請執行以下操作: 

1. 在網頁伺服器中安裝 Dispatcher (可以是 Apache 或 IIS 網頁伺服器，請參閱[安裝 Dispatcher](dispatcher-install.md))。
1. 您可能希望根據正在運作的 AEM Publish 執行個體來測試新安裝的 Dispatcher，以確保已達成基準正確安裝。
1. 現在，請確定 Dispatcher 能夠透過 TCP/IP 連接到您的編寫執行個體。
1. 將範例 dispatcher.any 檔替換為 [Dispatcher 下載內容](release-notes.md#downloads)隨附的 author_dispatcher.any 檔。
1. 在文字編輯器中開啟 `author_dispatcher.any`，並進行下列變更: 

   1. 變更 `/hostname` 和 `/renders` 區段的 `/port`，以指向您的編寫執行個體。
   1. 變更 `/cache` 區段的 `/docroot`，以指向快取目錄。如果您正在使用[具有 Touch UI 的 AEM](https://helpx.adobe.com/experience-manager/6-3/sites/developing/using/touch-ui-concepts.html)，請參閱上面的警告。
   1. 儲存變更。

1. 刪除您在上面設定的 `/cache` > `/docroot` 目錄中的所有現有檔案。
1. 重新啟動網頁伺服器。

>[!NOTE]
>
>請注意，在提供的 `author_dispatcher.any` 設定中，當您安裝 CQ5 功能套件、Hotfix 或應用程式程式碼套件時，如果 `/libs` 或 `/apps` 下的任何內容受到影響，則您必須刪除 Dispatcher 快取中在這些目錄下的快取檔案，以確保下次請求時會擷取新升級的檔案，而非舊的快取檔案。

>[!CAUTION]
>
>如果您使用先前設定的編寫 Dispatcher 並啟用 *Dispatcher 清除代理程式*，請執行以下操作：

1. 在您的 AEM 編寫執行個體上刪除或停用&#x200B;**編寫 Dispatcher 的**&#x200B;清除代理程式。
1. 依照上述新指示，重新執行編寫 Dispatcher 設定。

<!--
[Author Dispatcher configuration file (Dispatcher 4.1.2 or later)](assets/author_dispatchernew.any)
-->
<!--[!NOTE]
>
>A related knowledge base article can be found here:  
>[How to configure the dispatcher in front of an authoring environment](https://helpx.adobe.com/cq/kb/HowToConfigureDispatcherForAuthoringEnvironment.html)
-->
