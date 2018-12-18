---
title: 包含檔案
description: 包含檔案
services: azure-digital-twins
author: alinamstanciu
ms.service: azure-digital-twins
ms.topic: include
ms.date: 09/19/2018
ms.author: alinast
ms.custom: include file
ms.openlocfilehash: d08b7a1256a26d1d7d39481ba15a8637339063ea
ms.sourcegitcommit: 74941e0d60dbfd5ab44395e1867b2171c4944dbe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/15/2018
ms.locfileid: "49322718"
---
1. 在 [Azure 入口網站](https://portal.azure.com)中，從左側導覽窗格開啟 **Azure Active Directory**，然後開啟 [屬性] 窗格。 將 [目錄識別碼] 複製到暫存檔。 您會使用此值來設定下一節的應用程式範例。

    ![Azure Active Directory 目錄識別碼](./media/digital-twins-permissions/aad-app-reg-tenant.png)

1. 開啟 [應用程式註冊] 窗格，然後按一下 [新增應用程式註冊] 按鈕。
    
    ![Azure Active Directory 應用程式註冊新增](./media/digital-twins-permissions/aad-app-reg-start.png)

1. 在 [名稱] 欄位中為此應用程式註冊提供易記的名稱。 將 [應用程式類型] 選擇為 [原生]，然後將 [重新導向 URI] 選擇為 **https://microsoft.com**。 按一下頁面底部的 [新增] 。

    ![Azure Active Directory 應用程式註冊建立](./media/digital-twins-permissions/aad-app-reg-create.png)

1. 開啟已註冊的應用程式，並將 [應用程式識別碼] 欄位的值複製到暫存檔；這個值可識別 Azure Active Directory 應用程式。 您會使用應用程式識別碼來設定後續章節的應用程式範例。

    ![Azure Active Directory 應用程式識別碼](./media/digital-twins-permissions/aad-app-reg-app-id.png)

1. 開啟 [應用程式註冊] 窗格，然後按一下 [設定] > [所需的權限]：
    - 按一下左上方的 [新增] 來開啟 [新增 API 存取權] 窗格。
    - 按一下 [選取 API]，並搜尋 **Azure Digital Twins**。 如果搜尋沒有找到 API，請改為搜尋 **Azure 智慧空間**。
    - 選取 Azure Digital Twins (Azure 智慧空間服務) 選項，然後按一下 選取。
    - 按一下 [選取權限]。 檢查 [讀取/寫入權限] 委派權限方塊，然後按一下 [選取]。
    - 在 [新增 API 存取權] 窗格中，按一下 [完成]。
    - 在 [所需的權限] 窗格中，按一下 [授與權限] 按鈕，然後接受所出現的通知。

       ![Azure Active Directory 應用程式註冊新增 API](./media/digital-twins-permissions/aad-app-req-permissions.png)
