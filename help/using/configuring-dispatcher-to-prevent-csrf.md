---
title: 設定 Dispatcher 以防止 CSRF 攻擊
seo-title: Configuring Adobe AEM Dispatcher to Prevent CSRF Attacks
description: 瞭解如何配置AEMDispatcher以防止跨站點請求偽造攻擊。
seo-description: Learn how to configure Adobe AEM Dispatcher to prevent Cross-Site Request Forgery attacks.
uuid: f290bdeb-54e2-4649-b0fc-6257b422af2d
topic-tags: dispatcher
content-type: reference
discoiquuid: d61d021e-b338-4a1d-91ee-55427557e931
exl-id: bcd38878-f977-46a6-b01a-03e4d90aef01
source-git-commit: 3a0e237278079a3885e527d7f86989f8ac91e09d
workflow-type: tm+mt
source-wordcount: '225'
ht-degree: 100%

---

# 設定 Dispatcher 以防止 CSRF 攻擊{#configuring-dispatcher-to-prevent-csrf-attacks}

提AEM供了旨在防止跨站點請求偽造攻擊的框架。 為了正確使用此框架，您需要對調度程式配置進行以下更改：

>[!NOTE]
>
>請確保根據現有配置更新以下示例中的規則編號。 請記住，調度程式將使用最後一個匹配規則來授予允許或拒絕，因此將規則放在現有清單的底部附近。

1. 在 `/clientheaders` 在author-farm.any和publish-farm.any的部分，將以下條目添加到清單底部：\
   `CSRF-Token`
1. 在的/filters部分 `author-farm.any` 和 `publish-farm.any` 或 `publish-filters.any` 檔案，添加以下行以允許請求 `/libs/granite/csrf/token.json` 通過調度員。\
   `/0999 { /type "allow" /glob " * /libs/granite/csrf/token.json*" }`
1. 在 `/cache /rules` 的 `publish-farm.any`，添加規則以阻止調度程式快取 `token.json` 的子菜單。 通常作者會繞過快取，因此您不必將規則添加到 `author-farm.any`。\
   `/0999 { /glob "/libs/granite/csrf/token.json" /type "deny" }`

要驗證配置是否正在工作，請在DEBUG模式下查看dispatcher.log以驗證token.json檔案是否未被快取且未被篩選器阻止。 您應看到類似於：\
`... checking [/libs/granite/csrf/token.json]  `
`... request URL not in cache rules: /libs/granite/csrf/token.json`\
`... cache-action for [/libs/granite/csrf/token.json]: NONE`

您還可以驗證apache中的請求是否成功 `access_log`。 「/libs/granite/csrf/token.json」的請求應返回HTTP 200狀態代碼。
