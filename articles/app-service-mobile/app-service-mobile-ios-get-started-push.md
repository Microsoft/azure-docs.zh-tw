---
title: 使用 Azure 行動應用程式將推播通知新增至 iOS 應用程式
description: 了解如何使用 Azure 行動應用程式將推播通知傳送至 iOS 應用程式。
services: app-service\mobile
documentationcenter: ios
manager: crdun
editor: ''
author: conceptdev
ms.assetid: fa503833-d23e-4925-8d93-341bb3fbab7d
ms.service: app-service-mobile
ms.workload: mobile
ms.tgt_pltfrm: mobile-ios
ms.devlang: objective-c
ms.topic: article
ms.date: 10/10/2016
ms.author: crdun
ms.openlocfilehash: b84990d053d92b9408e119b8aba28c8b9f8b046e
ms.sourcegitcommit: 58c5cd866ade5aac4354ea1fe8705cee2b50ba9f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42818993"
---
# <a name="add-push-notifications-to-your-ios-app"></a>將推播通知新增至您的 iOS 應用程式

[!INCLUDE [app-service-mobile-selector-get-started-push](../../includes/app-service-mobile-selector-get-started-push.md)]

## <a name="overview"></a>概觀

在本教學課程中，您會將推播通知新增至 [iOS 快速入門]專案，以便在每次插入一筆記錄時傳送推播通知至裝置。

如果您不要使用下載的快速入門伺服器專案，則需要推播通知擴充套件。 如需詳細資訊，請參閱[使用適用於 Azure Mobile Apps 的 .NET 後端伺服器 SDK](app-service-mobile-dotnet-backend-how-to-use-server-sdk.md) 指南。

[iOS 模擬器不支援推播通知](https://developer.apple.com/library/ios/documentation/IDEs/Conceptual/iOS_Simulator_Guide/TestingontheiOSSimulator.html)。 您需要實體 iOS 裝置和 [Apple Developer Program 成員資格](https://developer.apple.com/programs/ios/)。

## <a name="configure-hub"></a>設定通知中樞

[!INCLUDE [app-service-mobile-configure-notification-hub](../../includes/app-service-mobile-configure-notification-hub.md)]

## <a id="register"></a>註冊應用程式以取得推播通知

[!INCLUDE [Enable Apple Push Notifications](../../includes/enable-apple-push-notifications.md)]

## <a name="configure-azure-to-send-push-notifications"></a>設定 Azure 來傳送推播通知

[!INCLUDE [app-service-mobile-apns-configure-push](../../includes/app-service-mobile-apns-configure-push.md)]

## <a id="update-server"></a>更新後端程式碼以傳送推播通知

[!INCLUDE [app-service-mobile-dotnet-backend-configure-push-apns](../../includes/app-service-mobile-dotnet-backend-configure-push-apns.md)]

## <a id="add-push"></a>將推播通知新增至應用程式

[!INCLUDE [app-service-mobile-add-push-notifications-to-ios-app.md](../../includes/app-service-mobile-add-push-notifications-to-ios-app.md)]

## <a id="test"></a>測試推播通知

[!INCLUDE [Test Push Notifications in App](../../includes/test-push-notifications-in-app.md)]

## <a id="more"></a>更多資訊

* 範本可讓您彈性地傳送跨平台推播和當地語系化推播。 [如何使用適用於 Azure 行動應用程式的 iOS 用戶端程式庫](app-service-mobile-ios-how-to-use-client-library.md#templates) 一文，說明如何註冊範本。

<!-- Anchors.  -->

<!-- Images. -->

<!-- URLs. -->
[iOS 快速入門]: app-service-mobile-ios-get-started.md
