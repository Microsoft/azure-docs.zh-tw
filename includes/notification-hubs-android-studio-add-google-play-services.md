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
ms.openlocfilehash: 540c9775ae56798ce2d2d87fed4924244c31412f
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33835751"
---
1. 按一下 Android Studio 工具列上的圖示以開啟 Android SDK Manager，或按一下工能表上的 [工具]  >  [Android]  >  [SDK Manager]。 尋找專案中使用之 Android SDK 的目標版本，請按一下[顯示套件詳細資料] 來開啟該版本，然後選擇 [Google API] \(如果尚未安裝的話)。
2. 按一下 [SDK 工具] 索引標籤。如果您尚未安裝 Google Play 服務，按一下 [Google Play 服務]，如下所示。 然後按一下 [套用] 來安裝。 
   
    請注意在稍後步驟中使用的 SDK 路徑。 
   
    ![](./media/notification-hubs-android-studio-add-google-play-services/notification-hubs-android-studio-sdk-manager.png)
3. 開啟應用程式目錄中的 **build.gradle** 檔案。
   
    ![](./media/notification-hubs-android-studio-add-google-play-services/notification-hubs-android-studio-add-google-play-dependency.png)
4. 新增此行至 *相依項目*下方： 
    
    ```java
        compile 'com.google.android.gms:play-services-gcm:12.0.0'
    ```
5. 按一下工具列中的 [ **同步處理專案與 Gradle 檔案** ] 圖示。
6. 開啟 **AndroidManifest.xml** 並將此標記新增至「應用程式」標記。
   
    ```java
    <meta-data android:name="com.google.android.gms.version"
            android:value="@integer/google_play_services_version" />
    ```

