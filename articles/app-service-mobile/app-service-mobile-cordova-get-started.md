---
title: 在 Azure App Service Mobile Apps 上建立 Cordova 應用程式 | Microsoft Docs
description: 請依照此教學課程中的說明，開始使用 Azure 行動應用程式後端來進行 Apache Cordova 開發
services: app-service\mobile
documentationcenter: javascript
author: conceptdev
manager: crdun
editor: ''
tags: ''
keywords: cordova, javascript, mobile, 用戶端
ms.assetid: 0b08fc12-0a80-42d3-9cc1-9b3f8d3e3a3f
ms.service: app-service-mobile
ms.workload: na
ms.tgt_pltfrm: mobile-html
ms.devlang: javascript
ms.topic: hero-article
ms.date: 07/07/2017
ms.author: crdun
ms.openlocfilehash: 223e9e35fcab347f9b5b8db01a9fd667b9f5d55d
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38488641"
---
# <a name="create-an-apache-cordova-app"></a>建立 Apache Cordova 應用程式
[!INCLUDE [app-service-mobile-selector-get-started](../../includes/app-service-mobile-selector-get-started.md)]

## <a name="overview"></a>概觀
本教學課程示範如何使用 Azure 行動應用程式後端，將雲端式後端服務新增到 Apache Cordova 行動應用程式。  您會建立新的行動應用程式後端，以及可在 Azure 中儲存應用程式資料的簡易「待辦事項清單」 Apache Cordova 應用程式。

所有其他 Apache Cordova 教學課程只要是與使用 Azure App Service 中的 Mobile Apps 功能相關，都必須先完成本教學課程。

## <a name="prerequisites"></a>先決條件
若要完成本教學課程，您需要下列必要條件：

* 具有 [Visual Studio Community 2017] 或更新版本的電腦。
* [Visual Studio Tools for Apache Cordova]。
* 有效的 [Azure 帳戶](https://azure.microsoft.com/pricing/free-trial/)。

您也可以略過 Visual Studio 而直接使用 Apache Cordova 命令列。  在 Mac 電腦上完成本教學課程時，使用命令列相當有用。  本教學課程未涵蓋使用命令列來編譯 Apache Cordova 用戶端應用程式。

## <a name="create-an-azure-mobile-app-backend"></a>建立 Azure 行動應用程式後端
[!INCLUDE [app-service-mobile-dotnet-backend-create-new-service](../../includes/app-service-mobile-dotnet-backend-create-new-service.md)]

[觀看示範類似步驟的影片](https://channel9.msdn.com/series/Azure-connected-services-with-Cordova/Azure-connected-services-task-1-Create-an-Azure-Mobile-App)

## <a name="configure-the-server-project"></a>設定伺服器專案
[!INCLUDE [app-service-mobile-configure-new-backend.md](../../includes/app-service-mobile-configure-new-backend.md)]

## <a name="download-and-run-the-apache-cordova-app"></a>下載並執行 Apache Cordova 應用程式
[!INCLUDE [app-service-mobile-cordova-run-app](../../includes/app-service-mobile-cordova-run-app.md)]

## <a name="next-steps"></a>後續步驟
您現在已經完成本快速入門教學課程，請繼續進行下列其中一個教學課程：

* [新增離線應用程式](app-service-mobile-cordova-get-started-offline-data.md)至 Apache Cordova 應用程式。
* [新增驗證](app-service-mobile-cordova-get-started-users.md) 至您的 Apache Cordova 應用程式。
* [新增推播通知](app-service-mobile-cordova-get-started-push.md) 至您的 Apache Cordova 應用程式。

深入了解使用 Azure App Service 的相關重要概念。

* [離線資料]
* [驗證]
* [推播通知]

了解如何使用 SDK。

* [Apache Cordova SDK]
* [ASP.NET Server SDK]
* [Node.js Server SDK]

<!-- Images. -->

<!-- URLs -->
[Azure portal]: https://portal.azure.com/
[Visual Studio Community 2017]: http://www.visualstudio.com/
[Visual Studio Tools for Apache Cordova]: https://www.visualstudio.com/en-us/features/cordova-vs.aspx
[離線資料]: app-service-mobile-offline-data-sync.md
[驗證]: app-service-mobile-auth.md
[推播通知]: ../notification-hubs/notification-hubs-push-notification-overview.md
[Apache Cordova SDK]: app-service-mobile-cordova-how-to-use-client-library.md
[ASP.NET Server SDK]: app-service-mobile-dotnet-backend-how-to-use-server-sdk.md
[Node.js Server SDK]: app-service-mobile-node-backend-how-to-use-server-sdk.md
