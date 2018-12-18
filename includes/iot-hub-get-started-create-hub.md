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
ms.openlocfilehash: 08afdcf91499fdb6f2da6e9ccc82313286f5c964
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43111960"
---
## <a name="create-an-iot-hub"></a>建立 IoT 中樞
建立 IoT 中樞以供您的模擬裝置應用程式連接。 下列步驟顯示如何使用 Azure 入口網站完成這項工作。

1. 登入 [Azure 入口網站](https://portal.azure.com/)。

2. 選取 [建立資源] > [物聯網] > [IoT 中樞]。
   
    ![Azure 入口網站 Jumpbar](./media/iot-hub-get-started-create-hub/create-iot-hub1.png)

3. 在 [IoT 中樞] 窗格中，輸入 IoT 中樞的下列資訊︰

   * **訂用帳戶**：選擇您要用於建立此 IoT 中樞的訂用帳戶。

   * **資源群組**：建立用以裝載 IoT 中樞的資源群組，或使用現有資源群組。 如需詳細資訊，請參閱[使用資源群組管理您的 Azure 資源](../articles/azure-resource-manager/resource-group-portal.md)。

   * **區域**：選取最接近您的位置。

   * **名稱**：建立 IoT 中樞的名稱。 如果您輸入的名稱可用，則會出現綠色核取記號。

   [!INCLUDE [iot-hub-pii-note-naming-hub](iot-hub-pii-note-naming-hub.md)]

   ![IoT 中樞基本概念視窗](./media/iot-hub-get-started-create-hub/create-iot-hub2.png)

4. 選取 [下一步：大小和級別] 繼續建立 IoT 中樞。 

5. 選擇您的**定價與級別層**。 在本文中，選取 [F1-免費] 層 (如果仍可用於您的訂用帳戶)。 如需詳細資訊，請參閱[定價與級別層](https://azure.microsoft.com/pricing/details/iot-hub/)。

   ![IoT 中樞大小和級別視窗](./media/iot-hub-get-started-create-hub/create-iot-hub3.png)

6. 選取 [檢閱 + 建立]。

7. 檢閱您的 IoT 中樞資訊，然後按一下 [建立]。 建立 IoT 中樞可能需要數分鐘。 您可以在 [通知] 窗格中監視進度。

8. 當新的 IoT 中樞就緒時，請在 Azure 入口網站中按一下其圖格，以開啟 IoT 中樞的屬性視窗。 既然您已經建立 IoT 中樞，請找出您用來將裝置和應用程式與 IoT 中樞連線的重要資訊。 按一下 [共用存取原則]。
   
9. 在 [共用存取原則] 中，選取 **iothubowner** 原則。 複製 IoT 中樞的**連接字串---主索引鍵**，以供日後使用。 如需詳細資訊，請參閱《IoT 中樞開發人員指南》中的[存取控制](../articles/iot-hub/iot-hub-devguide-security.md)。
   
    ![共用存取原則](./media/iot-hub-get-started-create-hub/create-iot-hub5.png)