---
title: 設定Dispatcher以防止CSRF攻擊
seo-title: 設定Adobe AEM Dispatcher以防止CSRF攻擊
description: 瞭解如何設定AEM Dispatcher以防止跨網站偽造要求偽造攻擊。
seo-description: 瞭解如何設定Adobe AEM Dispatcher以防止跨網站偽造要求偽造攻擊。
uuid: f290bdeb-54e2-4649-b0 fc-6257b422 af2 d
topic-tags: dispatcher
content-type: 引用
discoiquuid: d61d021e-b338-4a1 d-91ee-55427557e931
translation-type: tm+mt
source-git-commit: 69edbe7608b46c93d238515e4223606eadad0ac4

---


# 設定Dispatcher以防止CSRF攻擊{#configuring-dispatcher-to-prevent-csrf-attacks}

AEM提供一個架構，旨在防止跨網站偽造要求偽造攻擊。若要正確使用此架構，您必須對傳送程式配置進行下列變更：

>[!NOTE]
>
>請務必根據您現有的設定，在下列範例中更新規則編號。請記住，傳送人員將使用最後一個符合規則授與允許或拒絕，因此將規則放置在現有清單底部附近。

1. 在作者-農場. any和publish-farm. any `/clientheaders` 區段中，新增下列項目至清單底部：\
   `CSRF-Token`
1. 在您 `author-farm.any` 和 `publish-farm.any` 或 `publish-filters.any` 檔案的/filters區段中，新增下列行以允許 `/libs/granite/csrf/token.json` 透過發送器要求請求。\
   `/0999 { /type "allow" /glob " * /libs/granite/csrf/token.json*" }`
1. 在您 `/cache /rules``publish-farm.any`的區段下方，新增規則以封鎖傳送程式快取 `token.json` 檔案。通常作者略過快取，因此您不需要將規則新增 `author-farm.any`至您的規則。\
   `/0999 { /glob "/libs/granite/csrf/token.json" /type "deny" }`

若要驗證設定是否運作正常，請觀看dispatcher. log以確認token. json檔案未被快取，並不會被篩選器封鎖。您應該會看到類似於：\
`... checking [/libs/granite/csrf/token.json]  `
`... request URL not in cache rules: /libs/granite/csrf/token.json`\
`... cache-action for [/libs/granite/csrf/token.json]: NONE`

您也可以驗證請求是否已在您的Apache `access_log`中獲得成功。要求「/libs/granite/csrf/token. json」應傳回HTTP200狀態代碼。
