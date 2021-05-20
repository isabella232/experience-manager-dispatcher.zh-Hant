---
title: 最佳化網站以提升快取效能
seo-title: 最佳化網站以提升快取效能
description: 了解如何設計您的網站，以充份發揮快取的效益。
seo-description: Dispatcher提供許多內建機制，您可使用這些機制來最佳化效能。 了解如何設計您的網站，以充份發揮快取的效益。
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

Dispatcher提供許多內建機制，您可使用這些機制來最佳化效能。 本節說明如何設計您的網站，以發揮快取的最大效益。

>[!NOTE]
>
>這可協助您記住，Dispatcher會將快取儲存在標準Web伺服器上。 這表示您：
>
>* 可以快取您可儲存為頁面的所有項目，並使用URL提出要求
>* 無法儲存其他項目，例如HTTP標題、Cookie、工作階段資料和表單資料。

>
>
一般而言，許多快取策略都需要選取好的URL，而不需仰賴這些額外資料。

## 使用一致的頁面編碼{#using-consistent-page-encoding}

HTTP要求標頭不會進行快取，因此如果您將頁面編碼資訊儲存在標題中，就會發生問題。 在此情況下，當Dispatcher從快取中提供頁面時，頁面會使用Web伺服器的預設編碼。 有兩種方法可以避免此問題：

* 如果您只使用一種編碼，請確定Web伺服器上使用的編碼與AEM網站的預設編碼相同。
* 在HTML `head`區段中使用`<META>`標籤來設定編碼，如下列範例所示：

```xml
        <META http-equiv="Content-Type" content="text/html; charset=EUC-JP">
```

## 避免URL參數{#avoid-url-parameters}

如有可能，請避免要快取之頁面的URL參數。 例如，如果您有圖片庫，則不會快取下列URL（除非Dispatcher已相應地設定[](dispatcher-configuration.md#main-pars_title_24)）:

```xml
www.myCompany.com/pictures/gallery.html?event=christmas&amp;page=1
```

不過，您可以將這些參數放入頁面URL中，如下所示：

```xml
www.myCompany.com/pictures/gallery.christmas.1.html
```

>[!NOTE]
>
>此URL會呼叫與gallery.html相同的頁面和範本。 在範本定義中，您可以指定哪個指令碼轉譯頁面，或對所有頁面使用相同的指令碼。

## 依URL {#customize-by-url}自訂

如果您允許使用者變更字型大小（或任何其他版面自訂），請確定URL中反映不同的自訂。

例如，系統不會快取Cookie，因此若您將字型大小儲存在Cookie中（或類似的機制），則不會為快取頁面保留字型大小。 因此，Dispatcher會隨機傳回任何字型大小的檔案。

在URL中加入字型大小作為選取器可避免此問題：

```xml
www.myCompany.com/news/main.large.html
```

>[!NOTE]
>
>對於大多數的版面方面，也可以使用樣式表和/或用戶端指令碼。 這些功能在快取方面通常非常有效。
>
>在列印版本中，這也很實用，您可以在其中使用URL，例如：&quot;
>
>`www.myCompany.com/news/main.print.html`
>
>使用模板定義的指令碼全域功能，可以指定一個單獨的指令碼來呈現打印頁。

## 使用作標題的影像檔案失效{#invalidating-image-files-used-as-titles}

如果您將頁面標題或其他文字呈現為圖片，則建議儲存這些檔案，以便在頁面上的內容更新時刪除這些檔案：

1. 將影像檔案放置在與頁面相同的資料夾中。
1. 對影像檔案使用以下命名格式：

   `<page file name>.<image file name>`

例如，您可以將頁面myPage.html的標題儲存在myPage.title.gif檔案中。 如果頁面更新，則會自動刪除此檔案，因此對頁面標題所做的任何變更都會自動反映在快取中。

>[!NOTE]
>
>影像檔案不一定實際存在於AEM例項上。 您可以使用動態建立影像檔案的指令碼。 Dispatcher接著會將檔案儲存在Web伺服器上。

## 使用於導航{#invalidating-image-files-used-for-navigation}的影像檔案失效

如果您使用圖片作為導航條目，則方法與標題基本相同，只是略微複雜。 將所有導航影像與目標頁面一起儲存。 如果您將兩張圖片用於正常和活動，則可以使用以下指令碼：

* 正常顯示頁面的指令碼。
* 處理「.normal」請求並傳回一般圖片的指令碼。
* 處理「.active」請求並返回激活圖片的指令碼。

請務必使用與頁面相同的命名控制代碼建立這些圖片，以確保內容更新會刪除這些圖片和頁面。

對於未修改的頁面，圖片仍會保留在快取中，儘管頁面本身通常會自動失效。

## 個性化 {#personalization}

Dispatcher無法快取個人化資料，因此建議您將個人化限制在必要的位置。 說明原因：

* 如果您使用可自由自訂的開始頁面，則每次使用者要求時，都必須組成該頁面。
* 相較之下，如果您提供10個不同的開始頁面選項，則可以快取其中每一個頁面，從而提高效能。

>[!NOTE]
>
>如果您個人化每個頁面（例如將使用者名稱放入標題列），則無法快取該頁面，這可能會對效能造成重大影響。
>
>不過，如果您必須這麼做，您可以：
>
>* 使用iFrame將頁面分割為一個對所有使用者都相同的部分，以及一個對使用者的所有頁面都相同的部分。 然後，您可以快取這兩個部件。
>* 使用用戶端JavaScript來顯示個人化資訊。 不過，若使用者關閉JavaScript，您必須確定頁面仍可正確顯示。

>



## 黏著連線{#sticky-connections}

[黏著](dispatcher.md#TheBenefitsofLoadBalancing) 連線，確保同一個使用者的檔案都是在同一台伺服器上撰寫。如果使用者離開此資料夾，稍後返回該資料夾，連線仍會持續。 定義一個資料夾，以保留網站需要黏著連線的所有檔案。 請盡量不要有其他檔案。 如果您使用個人化頁面和工作階段資料，這會影響負載平衡。

## MIME類型{#mime-types}

瀏覽器有兩種方式可判斷檔案類型：

1. 透過其擴充功能(例如.html、.gif、.jpg等)
1. 由伺服器隨檔案傳送的MIME類型。

對於大多數檔案，副檔名中隱含MIME類型。 i.e.:

1. 透過其擴充功能(例如.html、.gif、.jpg等)
1. 由伺服器隨檔案傳送的MIME類型。

如果檔案名沒有副檔名，則會顯示為純文字。

MIME類型是HTTP標題的一部分，因此Dispatcher不會快取它。 如果您的AEM應用程式傳回的檔案未識別檔案結尾，但需仰賴MIME類型，則這些檔案可能會不正確顯示。

要確保正確快取檔案，請遵循以下准則：

* 請確定檔案的副檔名一律正確。
* 請避免一般檔案伺服器指令碼，這些指令碼的URL如download.jsp?file=2214。 重寫指令碼以使用包含檔案規範的URL;在上一個範例中，這會是download.2214.pdf。

