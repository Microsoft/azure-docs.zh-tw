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
ms.openlocfilehash: 63acf0297a694ff442d56e67d52fd9b4e49f812d
ms.sourcegitcommit: e0a678acb0dc928e5c5edde3ca04e6854eb05ea6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/13/2018
ms.locfileid: "39008581"
---
第一個步驟是使用 Azure 入口網站在您的訂用帳戶中建立 IoT 中樞。 IoT 中樞可讓您從許多裝置將大量的遙測擷取到雲端。 然後中樞會啟用在雲端執行的一或多項後端服務，以讀取和處理該遙測。

1. 登入 [Azure 入口網站](http://portal.azure.com)。

1. 選取 [建立資源] > [物聯網] > [IoT 中樞]。

    ![選擇安裝 IoT 中樞][1]

1. 在 [IoT 中樞] 窗格中，輸入 IoT 中樞的下列資訊︰

   * **訂用帳戶**：選擇您要用於建立此 IoT 中樞的訂用帳戶。
   * **資源群組**：建立用以包含 IoT 中樞的資源群組，或使用現有資源群組。 您可以將所有相關資源一起放在群組中，例如 **TestResources**，一併加以管理。 例如，刪除資源群組時，將會刪除該群組中包含的所有資源。 如需詳細資訊，請參閱[使用資源群組管理您的 Azure 資源][lnk-resource-groups]。
   * **區域**：選取最接近裝置的位置。
   * **名稱**：建立 IoT 中樞的唯一名稱。 如果您輸入的名稱可用，則會出現綠色核取記號。

   [!INCLUDE [iot-hub-pii-note-naming-hub](iot-hub-pii-note-naming-hub.md)]

   ![IoT 中樞基本概念視窗][2]

2. 選取 [下一步：大小和級別] 繼續建立 IoT 中樞。 

3. 選擇您的**定價與級別層**。 在本文中，選取 [F1-免費] 層 (如果仍可用於您的訂用帳戶)。 如需詳細資訊，請參閱[定價與級別層][lnk-pricing]。

   ![IoT 中樞大小和級別視窗][3]

4. 選取 [檢閱 + 建立]。

1. 檢閱您的 IoT 中樞資訊，然後按一下 [建立]。 建立 IoT 中樞可能需要數分鐘。 您可以在 [通知] 窗格中監視進度。


<!-- Images -->
[1]: ./media/iot-hub-create-hub/create-iot-hub1.png
[2]: ./media/iot-hub-create-hub/create-iot-hub2.png
[3]: ./media/iot-hub-create-hub/create-iot-hub3.png
<!-- Links -->
[lnk-portal]: https://portal.azure.com/
[lnk-pricing]: https://azure.microsoft.com/pricing/details/iot-hub/
[lnk-resource-groups]: ../articles/azure-resource-manager/resource-group-portal.md