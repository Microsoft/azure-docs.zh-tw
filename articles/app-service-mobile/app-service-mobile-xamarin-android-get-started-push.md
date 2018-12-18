---
title: 將推播通知新增至 Xamarin.Android 應用程式 | Microsoft Docs
description: 了解如何使用 Azure App Service 及 Azure 通知中樞將推播通知傳送到 Xamarin.Android 應用程式
services: app-service\mobile
documentationcenter: xamarin
author: conceptdev
manager: crdun
editor: ''
ms.assetid: 6f7e8517-e532-4559-9b07-874115f4c65b
ms.service: app-service-mobile
ms.workload: mobile
ms.tgt_pltfrm: mobile-xamarin-android
ms.devlang: dotnet
ms.topic: article
ms.date: 10/12/2016
ms.author: crdun
ms.openlocfilehash: e2388c887f4a96883aa64a0a6fec3a5a9df5b8cc
ms.sourcegitcommit: 58c5cd866ade5aac4354ea1fe8705cee2b50ba9f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42818189"
---
# <a name="add-push-notifications-to-your-xamarinandroid-app"></a>將推播通知新增至 Xamarin.Android 應用程式

[!INCLUDE [app-service-mobile-selector-get-started-push](../../includes/app-service-mobile-selector-get-started-push.md)]

## <a name="overview"></a>概觀

在本教學課程中，您會將推播通知新增至 [Xamarin.Android 快速入門](app-service-mobile-windows-store-dotnet-get-started.md)專案，以便在每次插入一筆記錄時傳送推播通知至裝置。

如果您不要使用下載的快速入門伺服器專案，則需要推播通知擴充套件。 如需詳細資訊，請參閱[使用適用於 Azure Mobile Apps 的 .NET 後端伺服器 SDK](app-service-mobile-dotnet-backend-how-to-use-server-sdk.md) 指南。

## <a name="prerequisites"></a>必要條件

本教學課程需要下列設定：

* 有效的 Google 帳戶。 您可以在 [accounts.google.com](http://go.microsoft.com/fwlink/p/?LinkId=268302)註冊 Google 帳戶。
* [Google Cloud Messaging 用戶端元件](http://components.xamarin.com/view/GCMClient/)。

## <a name="configure-hub"></a>設定通知中樞

[!INCLUDE [app-service-mobile-configure-notification-hub](../../includes/app-service-mobile-configure-notification-hub.md)]

## <a id="register"></a>啟用 Firebase 雲端傳訊

[!INCLUDE [notification-hubs-enable-firebase-cloud-messaging](../../includes/notification-hubs-enable-firebase-cloud-messaging.md)]

## <a name="configure-azure-to-send-push-requests"></a>設定 Azure 來傳送推播要求

[!INCLUDE [app-service-mobile-android-configure-push](../../includes/app-service-mobile-android-configure-push-for-firebase.md)]

## <a id="update-server"></a>更新伺服器專案以傳送推播通知

[!INCLUDE [app-service-mobile-update-server-project-for-push-template](../../includes/app-service-mobile-update-server-project-for-push-template.md)]

## <a id="configure-app"></a>設定用於推播通知的用戶端專案

[!INCLUDE [mobile-services-xamarin-android-push-configure-project](../../includes/mobile-services-xamarin-android-push-configure-project.md)]

## <a id="add-push"></a>將推播通知程式碼新增至應用程式

[!INCLUDE [app-service-mobile-xamarin-android-push-add-to-app](../../includes/app-service-mobile-xamarin-android-push-add-to-app.md)]

## <a name="test"></a>在應用程式中測試推播通知

您可以在模擬器中使用虛擬裝置來測試應用程式。 在模擬器上執行時，您需要採取其他設定步驟。

1. 虛擬裝置必須將 Google API 設定為 Android 虛擬裝置 (AVD) 中的目標。

    ![](./media/app-service-mobile-xamarin-android-get-started-push/google-apis-avd-settings.png)

2. 按一下 [應用程式] > [設定] > [新增帳戶] 將 Google 帳戶加入 Android 裝置，然後遵循提示。

    ![](./media/app-service-mobile-xamarin-android-get-started-push/add-google-account.png)

3. 按照先前的方法執行 todolist 應用程式，然後插入新的 todo 項目。 這次，通知圖示會顯示在通知區域中。 您可以開啟通知抽屜來檢視通知的完整內容。

    ![](./media/app-service-mobile-xamarin-android-get-started-push/android-notifications.png)

<!-- URLs. -->
[Xamarin.Android quick start]: app-service-mobile-xamarin-android-get-started.md
[Google Cloud Messaging Client Component]: http://components.xamarin.com/view/GCMClient/
[Azure Mobile Services Component]: http://components.xamarin.com/view/azure-mobile-services/
