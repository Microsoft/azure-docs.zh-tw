---
title: Azure IoT 中樞感應器資料的即時資料視覺效果 – Web Apps | Microsoft Docs
description: 使用 Microsoft Azure App Service 的 Web Apps 功能來視覺化收集自感應器並傳送至 IoT 中樞的溫度和溼度資料。
author: rangv
manager: ''
keywords: 即時資料視覺效果, 即時資料視覺效果, 感應器資料視覺效果
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.tgt_pltfrm: arduino
ms.date: 04/11/2018
ms.author: rangv
ms.openlocfilehash: d40bcc8e6fd47a00618b98972f92c1e6fa019612
ms.sourcegitcommit: 74941e0d60dbfd5ab44395e1867b2171c4944dbe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/15/2018
ms.locfileid: "49318515"
---
# <a name="visualize-real-time-sensor-data-from-your-azure-iot-hub-by-using-the-web-apps-feature-of-azure-app-service"></a>使用 Azure App Service 的 Web Apps 功能將來自 Azure IoT 中樞的即時感應器資料視覺化

![端對端圖表](media/iot-hub-get-started-e2e-diagram/5.png)

[!INCLUDE [iot-hub-get-started-note](../../includes/iot-hub-get-started-note.md)]

## <a name="what-you-learn"></a>您學到什麼

在本教學課程中，您會了解如何執行 Web 應用程式 (web app) 上所裝載的 Web 應用程式 (web application)，將 IoT 中樞收到的即時感應器資料視覺化。 如果您想嘗試使用 Power BI 將 IoT 中樞的資料視覺化，請參閱[使用 Power BI 將 Azure IoT 中樞的即時感應器資料視覺化](iot-hub-live-data-visualization-in-power-bi.md)。

## <a name="what-you-do"></a>您要做什麼

- 在 Azure 入口網站中建立 Web 應用程式。
- 新增取用者群組，讓您的 IoT 中樞準備好進行資料存取。
- 設定 Web 應用程式 (web app) 來讀取 IoT 中樞的感應器資料。
- 上傳要供 Web 應用程式 (web app) 裝載的 Web 應用程式 (web application)。
- 開啟 Web 應用程式 (web app)，以查看 IoT 中樞的即時溫度和溼度資料。

## <a name="what-you-need"></a>您需要什麼

- [設定您的裝置](iot-hub-raspberry-pi-kit-node-get-started.md)，其中涵蓋下列需求︰
  - 作用中的 Azure 訂用帳戶
  - 您訂用帳戶之下的 IoT 中樞
  - 將訊息傳送到 IoT 中樞的用戶端應用程式
- [下載 Git](https://www.git-scm.com/downloads)

## <a name="create-a-web-app"></a>建立 Web 應用程式

1. 在 [Azure 入口網站](https://portal.azure.com/)中，按一下 [建立資源] > [Web + 行動] > [Web 應用程式]。
2. 輸入唯一的作業名稱、驗證訂用帳戶、指定資源群組和位置，選取 [釘選到儀表板]，然後按一下 [建立]。

   我們建議您選取資源群組所在的相同位置。 這麼做有助於提升處理速度並減少資料傳輸成本。

   ![建立 Web 應用程式](media/iot-hub-live-data-visualization-in-web-apps/2_create-web-app-azure.png)

[!INCLUDE [iot-hub-get-started-create-consumer-group](../../includes/iot-hub-get-started-create-consumer-group.md)]

## <a name="configure-the-web-app-to-read-data-from-your-iot-hub"></a>設定 Web 應用程式 (web app) 來讀取 IoT 中樞的資料

1. 開啟您剛才佈建的 Web 應用程式 (web app)。
2. 按一下 [應用程式設定]，然後在 [應用程式設定] 之下，新增下列索引鍵/值組：

   | Key                                   | 值                                                        |
   |---------------------------------------|--------------------------------------------------------------|
   | Azure.IoT.IoTHub.ConnectionString     | 從 Azure CLI 取得                                      |
   | Azure.IoT.IoTHub.ConsumerGroup        | 您新增至 IoT 中樞之取用者群組的名稱  |
   | WEBSITE_NODE_DEFAULT_VERSION          | 8.9.4                                                        |

   ![使用索引鍵/值組將設定新增至 Web 應用程式 (web app)](media/iot-hub-live-data-visualization-in-web-apps/4_web-app-settings-key-value-azure.png)

3. 按一下 [應用程式設定]，在 [一般設定] 之下切換 [Web 通訊端] 選項，然後按一下 [儲存]。

   ![切換 Web 通訊端選項](media/iot-hub-live-data-visualization-in-web-apps/10_toggle_web_sockets.png)

## <a name="upload-a-web-application-to-be-hosted-by-the-web-app"></a>上傳要供 Web 應用程式 (web app) 裝載的 Web 應用程式 (web application)

在 GitHub 上，我們已啟用 Web 應用程式 (web application)，以顯示 IoT 中樞的即時感應器資料。 您只需要設定 Web 應用程式 (web app) 來使用 Git 存放庫、從 GitHub 下載 Web 應用程式 (web application)，然後將它上傳至 Azure 以供 Web 應用程式 (web app) 裝載。

1. 在 Web 應用程式 (web app) 中，按一下 [部署選項] > [選擇來源] > [本機 Git 存放庫]，然後 [確定]。

   ![設定 Web 應用程式 (web app) 部署以使用本機 Git 存放庫](media/iot-hub-live-data-visualization-in-web-apps/5_configure-web-app-deployment-local-git-repository-azure.png)

2. 按一下 [部署認證]，建立要用來連線到 Azure 中 Git 存放庫的使用者名稱和密碼，然後按一下 [儲存]。

3. 按一下 [概觀]，並記下 [Git 複製 url] 的值。

   ![取得 Web 應用程式 (web app) 的 Git 複製 URL](media/iot-hub-live-data-visualization-in-web-apps/7_web-app-git-clone-url-azure.png)

4. 開啟本機電腦上的命令或終端機視窗。

5. 從 GitHub 下載 Web 應用程式 (web app)，然後將它上傳到 Azure 以供 Web 應用程式 (web app) 裝載。 若要這樣做，請執行下列命令：

   ```bash
   git clone https://github.com/Azure-Samples/web-apps-node-iot-hub-data-visualization.git
   cd web-apps-node-iot-hub-data-visualization
   git remote add webapp <Git clone URL>
   git push webapp master:master
   ```

   > [!NOTE]
   > \<Git clone URL\> 是在 Web 應用程式 (web app) 的 [概觀] 頁面上找到的 Git 儲存機制 URL。

## <a name="open-the-web-app-to-see-real-time-temperature-and-humidity-data-from-your-iot-hub"></a>開啟 Web 應用程式 (web app)，以查看 IoT 中樞的即時溫度和溼度資料

在 Web 應用程式 (web app) 的 [概觀] 頁面上，按一下 URL 以開啟 Web 應用程式。

![取得 Web 應用程式的 URL](media/iot-hub-live-data-visualization-in-web-apps/8_web-app-url-azure.png)

您應該會看到 IoT 中樞的即時溫度和溼度資料。

![顯示即時溫度和溼度的 Web 應用程式 (web app) 頁面](media/iot-hub-live-data-visualization-in-web-apps/9_web-app-page-show-real-time-temperature-humidity-azure.png)

> [!NOTE]
> 確保範例應用程式正在您的裝置上執行。 若範例應用程式沒有在執行，您會看到空白的圖表，您可以參考[設定裝置](iot-hub-raspberry-pi-kit-node-get-started.md)下的教學。

## <a name="next-steps"></a>後續步驟
您已成功使用 Web 應用程式 (web app) 將 IoT 中樞的即時感應器資料視覺化。

如需將 IoT 中樞資料視覺化的其他方法，請參閱[使用 Power BI 將 Azure IoT 中樞的即時感應器資料視覺化](iot-hub-live-data-visualization-in-power-bi.md)。

[!INCLUDE [iot-hub-get-started-next-steps](../../includes/iot-hub-get-started-next-steps.md)]
