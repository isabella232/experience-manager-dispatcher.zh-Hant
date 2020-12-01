---
title: 針對快取效能最佳化網站
seo-title: 針對快取效能最佳化網站
description: 瞭解如何設計您的網站，以充份發揮快取的優點。
seo-description: Dispatcher提供了許多內置機制，可用於優化效能。 瞭解如何設計您的網站，以充份發揮快取的優點。
uuid: 2d4114d1-f464-4e10-b25c-a1b9a9c715d1
contentOwner: User
products: SG_EXPERIENCEMANAGER/DISPATCHER
topic-tags: dispatcher
content-type: reference
discoiquuid: ba323503-1494-4048-941d-c1d14f2e85b2
redirecttarget: https://helpx.adobe.com/experience-manager/6-4/sites/deploying/using/configuring-performance.html
index: y
internal: n
snippet: y
translation-type: tm+mt
source-git-commit: 2ca816ac0776d72be651b76ff4f45e0c3ed1450f
workflow-type: tm+mt
source-wordcount: '1167'
ht-degree: 3%

---


# 為快取效能優化網站{#optimizing-a-website-for-cache-performance}

<!-- 

Comment Type: remark
Last Modified By: Silviu Raiman (raiman)
Last Modified Date: 2017-10-25T04:13:34.919-0400

<p>This is a redirect to /experience-manager/6-2/sites/deploying/using/configuring-performance.html</p>

 -->

>[!NOTE]
>
>Dispatcher 版本與 AEM 無關。如果您依循連結至 Dispatcher 文件，且該連結內嵌於舊版 AEM 的文件中，您可能會被重新導向至本頁。

Dispatcher提供了許多內置機制，可用於優化效能。 本節會告訴您如何設計網站，以充份發揮快取的優點。

>[!NOTE]
>
>它可能有助於您記住Dispatcher將快取儲存在標準Web伺服器上。 這表示您：
>
>* 可以快取您可儲存為頁面並使用URL要求的所有項目
>* 無法儲存其他項目，例如HTTP標題、Cookie、工作階段資料和表單資料。

>
>
一般而言，許多快取策略都需要選取好的URL，而不需仰賴此額外資料。

## 使用一致的頁面編碼{#using-consistent-page-encoding}

HTTP請求標頭不會快取，因此，如果您將頁面編碼資訊儲存在標頭中，就會發生問題。 在這種情況下，當Dispatcher從快取中服務頁面時，該頁面將使用Web伺服器的預設編碼。 要避免此問題，有兩種方法：

* 如果您只使用一個編碼，請確定網頁伺服器上使用的編碼與AEM網站的預設編碼相同。
* 請在HTML `head`區段中使用`<META>`標籤來設定編碼，如下列範例所示：

```xml
        <META http-equiv="Content-Type" content="text/html; charset=EUC-JP">
```

## 避免URL參數{#avoid-url-parameters}

如果可能，請避免您要快取之頁面的URL參數。 例如，如果您有圖片庫，則不會快取下列URL（除非Dispatcher已據以設定[](dispatcher-configuration.md#main-pars_title_24)）:

```xml
www.myCompany.com/pictures/gallery.html?event=christmas&amp;page=1
```

不過，您可以將這些參數放入頁面URL中，如下所示：

```xml
www.myCompany.com/pictures/gallery.christmas.1.html
```

>[!NOTE]
>
>此URL會呼叫與gallery.html相同的頁面和範本。 在範本定義中，您可以指定要轉譯頁面的指令碼，或對所有頁面使用相同的指令碼。

## 依URL {#customize-by-url}自訂

如果您允許使用者變更字型大小（或任何其他版面自訂），請確定URL中會反映不同的自訂。

例如，Cookie不會快取，因此如果您將字型大小儲存在Cookie（或類似機制）中，則快取頁面不會保留字型大小。 因此，Dispatcher會隨機傳回任何字型大小的檔案。

在URL中加入字型大小做為選取器，可避免此問題：

```xml
www.myCompany.com/news/main.large.html
```

>[!NOTE]
>
>對於大部分的版面配置，您也可以使用樣式表和／或用戶端指令碼。 這些功能通常在快取時非常有效。
>
>這對於列印版本也很有用，您可在其中使用URL，例如：&quot;
>
>`www.myCompany.com/news/main.print.html`
>
>使用範本定義的指令碼全域化，您可以指定個別的指令碼，以轉譯列印頁面。

## 使用作標題的映像檔案失效{#invalidating-image-files-used-as-titles}

如果您將頁面標題或其他文字轉譯為圖片，則建議儲存檔案，以便在頁面內容更新時刪除這些檔案：

1. 將影像檔案放置在與頁面相同的檔案夾中。
1. 對影像檔案使用下列命名格式：

   `<page file name>.<image file name>`

例如，您可將頁面的標題myPage.html儲存在myPage.title.gif檔案中。 如果頁面已更新，此檔案會自動刪除，因此頁面標題的任何變更都會自動反映在快取中。

>[!NOTE]
>
>影像檔案不一定實際存在於AEM例項上。 您可以使用動態建立影像檔案的指令碼。 然後，Dispatcher將檔案儲存在Web伺服器上。

## 導覽{#invalidating-image-files-used-for-navigation}時使用的影像檔無效化

如果您使用圖片來輸入導覽項目，這個方法基本上就和標題一樣，只是稍微複雜一點。 將所有導覽影像與目標頁面一起儲存。 如果您使用兩張圖片來處理正常和活動，則可使用下列指令碼：

* 顯示頁面的指令碼，正常顯示。
* 處理&quot;。normal&quot;請求並傳回正常圖片的指令碼。
* 處理&quot;。active&quot;請求並傳回已啟動圖片的指令碼。

請務必使用與頁面相同的命名控制代碼來建立這些圖片，以確保內容更新會同時刪除這些圖片和頁面。

對於未修改的頁面，圖片仍保留在快取中，儘管頁面本身通常會自動失效。

## 個性化 {#personalization}

Dispatcher無法快取個人化資料，因此建議您將個人化限制在必要的位置。 為說明原因：

* 如果您使用可自由自訂的開始頁面，則必須在使用者每次要求時合成該頁面。
* 相反地，如果您提供10種不同的開始頁面選擇，則可以快取每個頁面，從而提升效能。

>[!NOTE]
>
>如果您個人化每個頁面（例如將使用者名稱放入標題列），則無法快取它，這可能會對效能造成重大影響。
>
>不過，如果您必須這麼做，您可以：
>
>* 使用iFrames將頁面分割為一個對所有用戶都相同的部分，以及對用戶所有頁面都相同的部分。 然後，您可以快取這兩部分。
>* 使用用戶端JavaScript來顯示個人化資訊。 不過，您必須確定當使用者關閉JavaScript時，頁面仍能正確顯示。

>



## 自黏連線{#sticky-connections}

[自黏](dispatcher.md#TheBenefitsofLoadBalancing) 連線確保一個使用者的檔案都是在同一伺服器上撰寫。如果使用者離開此資料夾，並稍後返回，連線仍會持續。 定義一個檔案夾，以存放所有需要網站嚴格連線的檔案。 請盡量不要在其中加入其他檔案。 如果您使用個人化頁面和作業資料，這會影響負載平衡。

## MIME類型{#mime-types}

瀏覽器有兩種方式可決定檔案類型：

1. 其延伸(例如.html、.gif、.jpg等)
1. 由伺服器隨檔案發送的MIME類型。

對於大多數檔案，MIME類型隱含在檔案副檔名中。 i.e.:

1. 其延伸(例如.html、.gif、.jpg等)
1. 由伺服器隨檔案發送的MIME類型。

如果檔案名沒有副檔名，則顯示為純文字檔案。

MIME類型是HTTP標頭的一部分，因此Dispatcher不快取它。 如果您的AEM應用程式傳回的檔案沒有識別的檔案結尾，但依賴MIME類型，則這些檔案可能會錯誤顯示。

若要確定檔案已正確快取，請遵循下列准則：

* 請確定檔案的副檔名一律正確。
* 請避免使用一般檔案伺服指令碼，這些指令碼具有URL，例如download.jsp?file=2214。 重寫指令碼，使用包含檔案規範的URL;在上一個範例中，此為download.2214.pdf。

