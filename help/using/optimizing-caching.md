---
title: 最佳化網站快取效能
seo-title: 最佳化網站快取效能
description: 瞭解如何設計網站，以最大化快取的優點。
seo-description: Dispatcher提供一些內建機制，可用來最佳化效能。瞭解如何設計網站，以最大化快取的優點。
uuid: 2d4114d1-f464-4e10-b25 c-a1 b9 a9 b9 a9 c715 d1
contentOwner: 使用者
products: SG_ PERIENCENCENAGER/ADDER
topic-tags: dispatcher
content-type: 引用
discoiquuid: ba323503-1494-4048-941d-c1 d14 f2 e85 b2
redirecttarget: https://helpx.adobe.com/experience-manager/6-4/sites/deploying/using/configuring-performance.html
index: y
internal: n
snippet: y
translation-type: tm+mt
source-git-commit: f35c79b487454059062aca6a7c989d5ab2afaf7b

---


# 最佳化網站快取效能 {#optimizing-a-website-for-cache-performance}

<!-- 

Comment Type: remark
Last Modified By: Silviu Raiman (raiman)
Last Modified Date: 2017-10-25T04:13:34.919-0400

<p>This is a redirect to /experience-manager/6-2/sites/deploying/using/configuring-performance.html</p>

 -->

>[!NOTE]
>
>Dispatcher版本與AEM獨立。如果您關注Dispatcher文件的連結(內嵌於舊版AEM的文件中)，可能會重新導向至此頁面。

Dispatcher提供一些內建機制，可用來最佳化效能。本節說明如何設計網站，以最大化快取的優點。

>[!NOTE]
>
>它可協助您記住Dispatcher將快取儲存在標準Web伺服器上。這表示您：
>
>* 可以快取您可以儲存為頁面並使用URL的所有項目
>* 無法儲存其他項目，例如HTTP標題、Cookie、作業資料和表單資料。
>
>
一般而言，許多快取策略涉及選擇良好URL，而不是依賴額外的資料。

## 使用一致的頁面編碼 {#using-consistent-page-encoding}

HTTP要求標題不會快取，因此如果您將頁面編碼資訊儲存在標題中，就會發生問題。在此情況下，當Dispatcher從快取服務頁面時，網頁伺服器的預設編碼會用於頁面。有兩種方法可避免此問題：

* 如果您只使用一個編碼，請確定網頁伺服器上使用的編碼與AEM網站的預設編碼相同。
* 使用 `<META>` HTML `head` 區段中的標籤來設定編碼，如下範例所示：

```xml
        <META http-equiv="Content-Type" content="text/html; charset=EUC-JP">
```

## 避免URL參數 {#avoid-url-parameters}

如有可能，請避免您要快取頁面的URL參數。例如，如果您有圖片收藏館，則絕不會快取下列URL(除非已 [設定Dispatcher)](dispatcher-configuration.md#main-pars_title_24)：

```xml
www.myCompany.com/pictures/gallery.html?event=christmas&amp;page=1
```

不過，您可以將這些參數放入頁面URL中，如下所示：

```xml
www.myCompany.com/pictures/gallery.christmas.1.html
```

>[!NOTE]
>
>此URL會呼叫相同頁面和與gallery. html相同的範本。在範本定義中，您可以指定哪個指令碼轉譯該頁面，或者您可以對所有頁面使用相同的指令碼。

## 依URL自訂 {#customize-by-url}

如果您允許使用者變更字體大小(或任何其他版面自訂)，請確定不同自訂會反映在URL中。

例如，不快取Cookie，因此，如果您將字型大小儲存在Cookie(或類似機制)中，則不會針對快取頁面保留字體大小。因此，Dispatcher會隨機傳回任何字體大小的文件。

將URL中的字型大小納入選取器中，可避免此問題：

```xml
www.myCompany.com/news/main.large.html
```

>[!NOTE]
>
>對於大部分版面配置，也可以使用樣式表和/或用戶端指令碼。這通常可搭配快取運作。
>
>這也適用於列印版本，您可以在其中使用URL，例如：
>
>`www.myCompany.com/news/main.print.html`
>
>使用範本定義的指令碼全域，您可以指定個別的指令碼來產生列印頁面。

## 使用作標題的影像檔案失效 {#invalidating-image-files-used-as-titles}

如果您將頁面標題或其他文字視為圖片，則建議您儲存檔案，以便在頁面上的內容更新時刪除它們：

1. 將影像檔案放置在與頁面相同的檔案夾中。
1. 請針對影像檔案使用下列命名格式：

   `<page file name>.<image file name>`

例如，您可以將頁面的標題儲存在檔案MyPage. title. gif中。如果更新頁面，此檔案會自動刪除，因此頁面標題的任何變更都會自動反映在快取中。

>[!NOTE]
>
>影像檔案不一定實體存在於AEM實例上。您可以使用動態建立影像檔案的指令碼。然後Dispatcher會將檔案儲存在Web伺服器上。

## 使用於導覽的影像檔案失效 {#invalidating-image-files-used-for-navigation}

如果您使用圖片來瀏覽導覽項目，方法基本上與標題相同，只會稍微複雜一些。使用目標頁面儲存所有導覽影像。如果您使用兩張一般和作用中的圖片，可以使用下列指令碼：

* 以正常方式顯示頁面的指令碼。
* 處理「.一般」請求並傳回一般圖片的指令碼。
* 處理「. active」要求並傳回已啓動圖片的指令碼。

您必須使用與頁面相同的命名控點來建立這些圖片，以確保內容更新會刪除這些圖片以及頁面。

對於未修改的頁面，圖片仍會保留在快取中，但頁面本身通常會自動失效。

## 個性化 {#personalization}

Dispatcher無法快取個人化資料，因此建議您將個人化限制在需要的位置。說明為何：

* 如果您使用可自由自訂的開始頁面，則每次使用者提出要求時，都必須加以組合。
* 相反地，如果您提供10個不同起始頁面的選擇，則可快取每個頁面，進而提高效能。

>[!NOTE]
>
>如果您個人化每個頁面(例如，將使用者名稱放入標題列)，您就無法快取它，因而造成重大效能影響。
>
>不過，如果您必須這麼做，可以：
>
>* 使用iFrames將頁面分割為所有使用者都相同的部分，而且所有使用者的頁面都相同。然後，您可以快取這兩部分。
>* 使用用戶端JavaScript顯示個人化資訊。不過，如果使用者關閉JavaScript，您必須確定頁面仍然正確顯示。
>



## 黏著連線 {#sticky-connections}

[黏著連線](dispatcher.md#TheBenefitsofLoadBalancing) 可確保一個使用者的文件都在同一伺服器上構成。如果使用者離開此資料夾，並稍後返回，連線仍會維持不變。定義一個資料夾，以放置所有需要網站連線的文件。請勿在其中使用其他文件。如果您使用個人化頁面和工作階段資料，這會影響負載平衡。

## MIME類型 {#mime-types}

瀏覽器可判斷檔案類型的兩種方式：

1. 依其擴充功能(例如.html、. gif、.jpg等)
1. 由伺服器隨檔案傳送的MIME類型。

對於大部分檔案，MIME類型會隱含在副檔名中。例如：

1. 依其擴充功能(例如.html、. gif、.jpg等)
1. 由伺服器隨檔案傳送的MIME類型。

如果檔案名稱沒有副檔名，則會顯示為純文字。

MIME類型是HTTP標題的一部分，因此，Dispatcher不會快取它。如果您的AEM應用程式傳回未識別檔案結尾的檔案，而是依賴MIME類型，則這些檔案可能會錯誤顯示。

為了確保正確快取檔案，請遵循下列方針：

* 請確定檔案的副檔名永遠正確。
* 避免一般檔案伺服指令碼，其中包含下載. jsp之類的URL？file=214.重新編寫指令碼以使用包含檔案規格的URL；上一個範例為下載.214.pdf。

