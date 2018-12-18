---
title: 在 Azure App Service Mobile Apps 上建立 Android 應用程式 | Microsoft Docs
description: 請遵循此教學課程，以開始使用 Azure 行動應用程式後端進行 Android 開發
services: app-service\mobile
documentationcenter: android
author: conceptdev
manager: crdun
editor: ''
ms.assetid: 355f0959-aa7f-472c-a6c7-9eecea3a34a9
ms.service: app-service-mobile
ms.workload: na
ms.tgt_pltfrm: mobile-android
ms.devlang: java
ms.topic: hero-article
ms.date: 10/01/2016
ms.author: crdun
ms.openlocfilehash: 69a11c615a2809d4d4180f0c4bd10a9dbeebc084
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38488767"
---
# <a name="create-an-android-app"></a>建立 Android 應用程式
[!INCLUDE [app-service-mobile-selector-get-started](../../includes/app-service-mobile-selector-get-started.md)]

## <a name="overview"></a>概觀
本教學課程將示範如何使用 Azure 行動應用程式後端，將雲端式後端服務加入 Android 行動應用程式。  您將建立新的行動應用程式後端，以及可在 Azure 中儲存應用程式資料的簡易「待辦事項清單」 Android 應用程式。

對於在 Azure App Service 中使用 Mobile Apps 功能的所有其他 Android 教學課程，完成本教學課程是必要條件。

## <a name="prerequisites"></a>先決條件
若要完成此教學課程，您需要下列項目：

* [Android Developer Tools](https://developer.android.com/sdk/index.html)，其中包括 Android Studio 整合式開發環境，以及最新的 Android 平台。
* Azure Mobile Android SDK，會自動受到參考，包含在您下載的快速入門專案中。
* 有效的 [Azure 帳戶](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="create-a-new-azure-mobile-app-backend"></a>建立新的 Azure 行動應用程式後端
[!INCLUDE [app-service-mobile-dotnet-backend-create-new-service](../../includes/app-service-mobile-dotnet-backend-create-new-service.md)]

## <a name="configure-the-server-project"></a>設定伺服器專案
[!INCLUDE [app-service-mobile-configure-new-backend.md](../../includes/app-service-mobile-configure-new-backend.md)]

## <a name="download-and-run-the-android-app"></a>下載並執行 Android 應用程式
[!INCLUDE [app-service-mobile-android-run-app](../../includes/app-service-mobile-android-run-app.md)]

<!-- URLs -->
[Azure portal]: https://portal.azure.com/
[Visual Studio Community 2013]: https://go.microsoft.com/fwLink/p/?LinkID=534203
