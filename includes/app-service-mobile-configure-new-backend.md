---
title: 包含檔案
description: 包含檔案
services: app-service\mobile
author: conceptdev
ms.service: app-service-mobile
ms.topic: include
ms.date: 05/25/2018
ms.author: crdun
ms.custom: include file
ms.openlocfilehash: 1d3bfb7bc8a5432392dba3b0c5019902b3e59773
ms.sourcegitcommit: eaad191ede3510f07505b11e2d1bbfbaa7585dbd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39514005"
---
1. 按一下 [應用程式服務] 按鈕，依序選取您的 Mobile Apps 後端、[快速入門]，以及您的用戶端平台 (iOS、Android、Xamarin、Cordova)。

    ![已醒目提示 [Mobile Apps 快速入門] 的 Azure 入口網站][quickstart]

1. 如果未設定資料庫連線，請執行下列動作來建立一個：

    ![內含 Mobile Apps 連線到資料庫的 Azure 入口網站][connect]

    a. 建立新的 SQL Database 和伺服器。 您可能必須將連接字串名稱欄位保留為 MS_TableConnectionString 的預設值，以完成下方的步驟 3。

    ![內含 Mobile Apps 建立新資料庫和伺服器的 Azure 入口網站][server]

    b. 等到成功建立資料連線為止。

    ![成功建立資料連線的 Azure 入口網站通知][notification]

    c. 資料連線必須成功。

    ![Azure 入口網站通知：「您已經有資料連線」][already-connection]

1. 在 **2.建立資料表 API** 之下，選取 Node.js 作為**後端語言**。

1. 接受通知，然後選取 [建立 TodoItem 資料表] 。
    這會在您的資料庫中建立新的待辦事項資料表。

    >[!IMPORTANT]
    > 將現有後端切換到 Node.js 會覆寫所有內容。 若要改為建立 .NET 後端，請參閱[使用適用於 Mobile Apps 的 .NET 後端伺服器 SDK][instructions]。

<!-- Images. -->
[quickstart]: ./media/app-service-mobile-configure-new-backend/quickstart.png
[connect]: ./media/app-service-mobile-configure-new-backend/connect-to-bd.png
[notification]: ./media/app-service-mobile-configure-new-backend/notification-data-connection-create.png
[server]: ./media/app-service-mobile-configure-new-backend/create-new-server.png
[already-connection]: ./media/app-service-mobile-configure-new-backend/already-connection.png

<!-- URLs -->
[instructions]: ../articles/app-service-mobile/app-service-mobile-dotnet-backend-how-to-use-server-sdk.md#create-app
