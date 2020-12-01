---
title: 設定 Dispatcher 以防止 CSRF 攻擊
seo-title: 設定Adobe AEM Dispatcher以防止CSRF攻擊
description: 瞭解如何設定AEM Dispatcher以防止跨網站偽造要求攻擊。
seo-description: 瞭解如何設定Adobe AEM Dispatcher以防止跨網站偽造要求攻擊。
uuid: f290bdeb-54e2-4649-b0fc-6257b422af2d
topic-tags: dispatcher
content-type: reference
discoiquuid: d61d021e-b338-4a1d-91ee-55427557e931
translation-type: tm+mt
source-git-commit: 69edbe7608b46c93d238515e4223606eadad0ac4
workflow-type: tm+mt
source-wordcount: '246'
ht-degree: 4%

---


# 設定 Dispatcher 以防止 CSRF 攻擊{#configuring-dispatcher-to-prevent-csrf-attacks}

AEM提供防止跨網站偽造要求攻擊的架構。 要正確使用此框架，您需要對調度程式配置進行以下更改：

>[!NOTE]
>
>請務必根據您現有的設定，更新下列範例中的規則編號。 請記住，調度程式將使用最後一個匹配規則授予允許或拒絕，因此請將規則放在現有清單底部附近。

1. 在author-farm.any和publish-farm.any的`/clientheaders`區段中，將下列項目新增至清單底部：\
   `CSRF-Token`
1. 在`author-farm.any`和`publish-farm.any`或`publish-filters.any`檔案的/filters區段中，新增下列行，以允許透過分派程式提出`/libs/granite/csrf/token.json`請求。\
   `/0999 { /type "allow" /glob " * /libs/granite/csrf/token.json*" }`
1. 在`publish-farm.any`的`/cache /rules`區段下，添加規則以阻止調度程式快取`token.json`檔案。 通常作者會略過快取，因此您不需要將規則新增至`author-farm.any`。\
   `/0999 { /glob "/libs/granite/csrf/token.json" /type "deny" }`

若要驗證設定是否正常運作，請在DEBUG模式中觀看dispatcher.log，以驗證token.json檔案是否未快取且未遭篩選器封鎖。 您應該會看到類似以下的訊息：\
`... checking [/libs/granite/csrf/token.json]  `
`... request URL not in cache rules: /libs/granite/csrf/token.json`\
`... cache-action for [/libs/granite/csrf/token.json]: NONE`

您也可以驗證請求是否在apache `access_log`中成功。 「/libs/granite/csrf/token.json」的要求應傳回HTTP 200狀態碼。
