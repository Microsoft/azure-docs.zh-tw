---
title: 包含檔案
description: 包含檔案
services: notification-hubs
author: spelluru
ms.service: notification-hubs
ms.topic: include
ms.date: 04/06/2018
ms.author: spelluru
ms.custom: include file
ms.openlocfilehash: ae84b5fd5647ef6c54006c0411e334b14173392e
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33836463"
---
1. 瀏覽至 [Google 雲端主控台](https://console.developers.google.com/cloud-resource-manager)，並使用您的 Google 帳戶認證登入。 
2. 選取工具列上的 [建立專案]。 
   
    ![建立新專案](./media/mobile-services-enable-google-cloud-messaging/mobile-services-google-new-project.png)   
3. 在 [專案名稱] 中，輸入您專案的名稱，然後按一下 [建立]。
4. 選取工具列上的 [警示] 按鈕，然後在清單中選取您的專案。 您會看到您專案的儀表板。 您也可以使用下列 URL 來直接巡覽至儀表板：https://console.developers.google.com/home/dashboard?project=<YOUR PROJECT NAME>

    ![在警示中選取您的專案](./media/mobile-services-enable-google-cloud-messaging/alert-new-project.png)
5. 請記下儀表板之 [專案資訊] 磚中的 [專案編號]。 

    ![專案識別碼](./media/mobile-services-enable-google-cloud-messaging/project-number.png)
6. 在儀表板的 [API] 磚上，選取 [Got to APIs overview] \(前往 API 概觀\)。 

    ![API 概觀連結](./media/mobile-services-enable-google-cloud-messaging/go-to-api-overview.png)
7. 在 [API] 頁面上，選取 [啟用 API 和服務]。 

    ![[啟用 API 和服務] 按鈕](./media/mobile-services-enable-google-cloud-messaging/enable-api-services-button.png)
8. 搜尋並選取 [Google 雲端通訊]。 

    ![搜尋並選取 [Google 雲端通訊]](./media/mobile-services-enable-google-cloud-messaging/search-select-gcm.png)
9. 若要啟用專案的 Google 雲端通訊，請選取 [啟用]。

    ![啟用 Google 雲端通訊](./media/mobile-services-enable-google-cloud-messaging/enable-gcm-button.png)
10. 選取工具列上的 [建立認證]。 

    ![[建立認證] 按鈕](./media/mobile-services-enable-google-cloud-messaging/create-credentials-button.png)
11. 在 [Add credentials to your project] \(將認證新增至您的專案\) 頁面上，選取 [API 金鑰] 連結。 

    ![[建立認證] 按鈕](./media/mobile-services-enable-google-cloud-messaging/api-key-button.png)    
12. 在 [API 金鑰] 頁面上，選取 [建立/儲存]。 在下列範例中，已選取 [IP 位址] 選項，並針對允許的 IP 位址輸入 **0.0.0.0/0**。 您應該適當地限制 API 金鑰。 

    ![API 金鑰 - [建立] 按鈕](./media/mobile-services-enable-google-cloud-messaging/api-key-create-button.png)
13. 將 [API 金鑰] 複製至剪貼簿，並將它儲存在某個位置。 

    ![複製 API 金鑰](./media/mobile-services-enable-google-cloud-messaging/copy-api-key.png)
   
    您將使用此 API 金鑰值，讓 Azure 能夠使用 GCM 進行驗證，並代表您的應用程式傳送推播通知。 若要瀏覽回專案儀表板，請使用 URL：https://console.developers.google.com/home/dashboard?project=<YOUR PROJECT NAME>

