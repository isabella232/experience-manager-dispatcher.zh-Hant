---
title: 設定 Dispatcher 以防止 CSRF 攻擊
seo-title: 設定AdobeAEM Dispatcher以防止CSRF攻擊
description: 了解如何設定AEM Dispatcher以防止跨網站請求偽造攻擊。
seo-description: 了解如何設定AdobeAEM Dispatcher以防止跨網站請求偽造攻擊。
uuid: f290bdeb-54e2-4649-b0fc-6257b422af2d
topic-tags: dispatcher
content-type: reference
discoiquuid: d61d021e-b338-4a1d-91ee-55427557e931
exl-id: bcd38878-f977-46a6-b01a-03e4d90aef01
source-git-commit: 3a0e237278079a3885e527d7f86989f8ac91e09d
workflow-type: tm+mt
source-wordcount: '246'
ht-degree: 4%

---

# 設定 Dispatcher 以防止 CSRF 攻擊{#configuring-dispatcher-to-prevent-csrf-attacks}

AEM提供框架，以防止跨網站請求偽造攻擊。 為了正確使用此架構，您必須對Dispatcher設定進行下列變更：

>[!NOTE]
>
>請務必根據您現有的設定，更新下列範例中的規則編號。 請記住，Dispatcher會使用最後一個相符的規則來授與允許或拒絕，請將規則放在現有清單的底部附近。

1. 在author-farm.any和publish-farm.any的`/clientheaders`區段中，將下列項目新增至清單底部：\
   `CSRF-Token`
1. 在`author-farm.any`和`publish-farm.any`或`publish-filters.any`檔案的/filters區段中，新增下列行以允許透過Dispatcher提出`/libs/granite/csrf/token.json`請求。\
   `/0999 { /type "allow" /glob " * /libs/granite/csrf/token.json*" }`
1. 在`publish-farm.any`的`/cache /rules`區段下，新增規則以封鎖Dispatcher不快取`token.json`檔案。 作者通常會略過快取，因此您不需要將規則新增至`author-farm.any`。\
   `/0999 { /glob "/libs/granite/csrf/token.json" /type "deny" }`

若要驗證設定是否正常運作，請以DEBUG模式觀看dispatcher.log ，以驗證token.json檔案是否未經快取，且篩選器未封鎖。 您應該會看到類似下列的訊息：\
`... checking [/libs/granite/csrf/token.json]  `
`... request URL not in cache rules: /libs/granite/csrf/token.json`\
`... cache-action for [/libs/granite/csrf/token.json]: NONE`

您也可以驗證Apache `access_log`中的請求是否成功。 「/libs/granite/csrf/token.json」的要求應會傳回HTTP 200狀態代碼。
