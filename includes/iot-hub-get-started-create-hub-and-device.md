---
title: 包含檔案
description: 包含檔案
services: iot-hub
author: dominicbetts
ms.service: iot-hub
ms.topic: include
ms.date: 05/17/2018
ms.author: dobett
ms.custom: include file
ms.openlocfilehash: 9f7fee71fb2b80be93d978569791dbb57f137949
ms.sourcegitcommit: 30fd606162804fe8ceaccbca057a6d3f8c4dd56d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/30/2018
ms.locfileid: "39346829"
---
## <a name="create-an-iot-hub"></a>建立 IoT 中樞

[!INCLUDE [iot-hub-create-hub](iot-hub-create-hub.md)]

既然您已經建立 IoT 中樞，請找出您用來將裝置和應用程式與 IoT 中樞連線的重要資訊。 

在您的 IoT 中樞導覽功能表中，開啟 [共用存取原則]。 選取 [iothubowner] 原則，然後複製 IoT 中樞的**連接字串---主索引鍵**。 如需詳細資訊，請參閱[控制 IoT 中樞的存取權](../articles/iot-hub/iot-hub-devguide-security.md)。

   > [!NOTE] 
   > 在此設定教學課程中，您不需要 iothubowner 連接字串。 不過，在您完成此設定之後，可能需要在一些教學課程或不同 IoT 案例用到它。

   ![取得 IoT 中樞連接字串](./media/iot-hub-get-started-create-hub-and-device/create-iot-hub5.png)

## <a name="register-your-device-in-the-iot-hub"></a>在 IoT 中樞註冊您的裝置

1. 在您的 IoT 中樞導覽功能表中，開啟 [IoT 裝置]，然後按一下 [新增] 在 IoT 中樞註冊裝置。

   ![在 IoT 中樞的 [IoT 裝置] 中新增裝置](./media/iot-hub-get-started-create-hub-and-device/create-identity-portal.png)

2. 輸入新裝置的 [裝置識別碼]。 裝置識別碼會區分大小寫。

   [!INCLUDE [iot-hub-pii-note-naming-device](iot-hub-pii-note-naming-device.md)]

3. 按一下 [檔案] 。

4. 建立裝置之後，請在 [IoT 裝置] 窗格中的清單開啟裝置。

5. 複製**連接字串---主索引鍵**，以供日後使用。

   ![取得裝置連接字串](./media/iot-hub-get-started-create-hub-and-device/device-connection-string.png)
