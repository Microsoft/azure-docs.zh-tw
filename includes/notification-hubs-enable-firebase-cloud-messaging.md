---
title: 包含檔案
description: 包含檔案
services: notification-hubs
author: spelluru
ms.service: notification-hubs
ms.topic: include
ms.date: 04/05/2018
ms.author: spelluru
ms.custom: include file
ms.openlocfilehash: 4dc67175f2ccc569ab6dfe5338607fa6bebe3882
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33814691"
---
1. 登入 [Firebase 主控台](https://firebase.google.com/console/)。 建立新的 Firebase 專案 (如果您還沒有 Firebase 專案的話)。
2. 建立專案之後，請選取 [將 Firebase 新增至 Android 應用程式]。 然後依照所提供的指示執行。 下載 **google-services.json** 檔案。 

    ![將 Firebase 新增至 Android 應用程式](./media/notification-hubs-enable-firebase-cloud-messaging/notification-hubs-add-firebase-to-android-app.png)
3. 在 Firebase 主控台中，選取您專案的齒輪圖示。 然後選取 [專案設定]。

    ![選取專案設定](./media/notification-hubs-enable-firebase-cloud-messaging/notification-hubs-firebase-console-project-settings.png)
4. 選取您專案設定中的 [一般] 索引標籤。 然後下載其中包含伺服器 API 金鑰和用戶端識別碼的 **google-services.json** 檔案。
5. 選取專案設定中的 [雲端通訊] 索引標籤，然後複製 [舊版伺服器金鑰] 的值。 您可使用此值來設定通知中樞存取原則。
