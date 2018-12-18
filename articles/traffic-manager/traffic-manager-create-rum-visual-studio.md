---
title: 使用 Visual Studio Mobile Center 將實際使用者度量傳送給 Azure 流量管理員 | Microsoft Docs
description: 將您使用 Visual Studio Mobile Center 開發的行動應用程式設定為傳送實際使用者度量給流量管理員
services: traffic-manager
documentationcenter: traffic-manager
author: KumudD
manager: timlt
editor: ''
tags: ''
ms.assetid: ''
ms.service: traffic-manager
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: ''
ms.workload: infrastructure
ms.date: 03/16/2018
ms.author: kumud
ms.custom: ''
ms.openlocfilehash: 893e84b07b365fb0b534e0ddc021b2249c4174cf
ms.sourcegitcommit: 48ab1b6526ce290316b9da4d18de00c77526a541
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/23/2018
ms.locfileid: "30181005"
---
# <a name="how-to-send-real-user-measurements-to-traffic-manager-with-visual-studio-mobile-center"></a>如何使用 Visual Studio Mobile Center 將實際使用者度量傳送給流量管理員

您可以藉由遵循以下步驟：將您使用 Visual Studio Mobile Center 開發的行動應用程式設定為傳送實際使用者度量給流量管理員：

>[!NOTE]
> 目前，將實際使用者度量傳送給流量管理員僅支援 Android。

若要設定實際使用者度量，您需要使用 RUM 套件取得金鑰並且檢測應用程式。

## <a name="step-1-obtain-a-key"></a>步驟 1：取得金鑰
    
您從用戶端應用程式取得並且傳送至流量管理員的度量，是由使用獨特字串的服務來識別，該字串稱為實際使用者度量 (RUM) 金鑰。 您可以使用 Azure 入口網站、REST API 或使用 PowerShell/CLI 介面，來取得 RUM 金鑰。

若要使用 Azure 入口網站取得 RUM 金鑰，請使用下列程序：
   1. 從瀏覽器登入 Azure 入口網站。 如果您沒有帳戶，可以註冊免費試用一個月。
   2. 在入口網站的搜尋列中，搜尋您想要修改的流量管理員設定檔名稱，然後按一下結果中顯示的流量管理員設定檔。
   3. 在流量管理員設定檔分頁上，按一下 [設定] 底下的 [實際使用者度量]。
   4. 按一下 [產生金鑰] 以建立新的 RUM 金鑰。
        
   ![產生實際使用者度量金鑰](./media/traffic-manager-create-rum-visual-studio/generate-rum-key.png)

   **圖 1：實際使用者度量金鑰產生**

   5.   分頁會顯示產生的 RUM 金鑰，以及必須內嵌至 HTML 分頁的 JavaScript 程式碼片段。
 
   ![實際使用者度量金鑰的 Javascript 程式碼](./media/traffic-manager-create-rum-visual-studio/rum-key.png)

   **圖 2：實際使用者度量金鑰和度量 JavaScript**
 
   6. 按一下 [複製] 按鈕以複製 RUM 金鑰。 

## <a name="step-2-instrument-your-app-with-the-rum-package-of-mobile-center-sdk"></a>步驟 2：使用 Mobile Center SDK 的 RUM 套件檢測應用程式

如果您還不熟悉 Visual Studio Mobile Center，請造訪其[網站](https://mobile.azure.com)。 如需 SDK 整合的詳細指示，請參閱[開始使用 Android SDK](https://docs.microsoft.com/mobile-center/sdk/getting-started/Android)。

若要使用實際使用者度量，請完成下列程序：

1.  將 SDK 新增至專案

    在 ATM RUM SDK 的預覽期間，您需要明確參考套件存放庫。

    在您的 **app/build.gradle** 檔案中新增下列行：

    ```groovy
    repositories {
        maven {
            url "https://dl.bintray.com/mobile-center/mobile-center-snapshot"
        }
    }
    ```
    在您的 **app/build.gradle** 檔案中新增下列行：

    ```groovy
    dependencies {   
     
        def mobileCenterSdkVersion = '0.12.1-16+3fe5b08'
        compile "com.microsoft.azure.mobile:mobile-center-rum:${mobileCenterSdkVersion}"
    }
    ```

2. 開始 SDK

    開啟應用程式的主要活動類別，並且新增下列匯入陳述式：

    ```java
    import com.microsoft.azure.mobile.MobileCenter;
    import com.microsoft.azure.mobile.rum.RealUserMeasurements;
    ```

    在相同檔案中尋找 `onCreate` 回呼，並且新增下列程式碼：

    ```java
    RealUserMeasurements.setRumKey("<Your RUM Key>");
    MobileCenter.start(getApplication(), "<Your Mobile Center AppSecret>", RealUserMeasurements.class);
    ```

## <a name="next-steps"></a>後續步驟
- 深入了解[實際使用者度量](traffic-manager-rum-overview.md)
- 了解 [流量管理員的運作方式](traffic-manager-overview.md)
- 深入了解 [Mobile Center](https://docs.microsoft.com/mobile-center/)
- [註冊](https://mobile.azure.com) Mobile Center
- 深入了解流量管理員支援的 [流量路由方法](traffic-manager-routing-methods.md)
- 了解如何 [建立流量管理員設定檔](traffic-manager-create-profile.md)

