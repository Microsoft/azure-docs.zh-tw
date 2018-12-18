---
title: 在 Azure 入口網站中設定裝置佈建 | Microsoft Docs
description: Azure 快速入門 - 在 Azure 入口網站中設定 Azure IoT 中樞裝置佈建服務
author: wesmc7777
ms.author: wesmc
ms.date: 07/12/2018
ms.topic: quickstart
ms.service: iot-dps
services: iot-dps
manager: timlt
ms.custom: mvc
ms.openlocfilehash: 5533dacc4a782cbdb6026c4a4d5c5bed9e39147a
ms.sourcegitcommit: 74941e0d60dbfd5ab44395e1867b2171c4944dbe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/15/2018
ms.locfileid: "49320147"
---
# <a name="set-up-the-iot-hub-device-provisioning-service-with-the-azure-portal"></a>使用 Azure 入口網站設定 IoT 中樞裝置佈建服務

這些步驟顯示如何在入口網站中設定 Azure 雲端資源，以便佈建裝置。 本文包含以下作業的步驟：建立 IoT 中樞、建立新的 IoT 中樞裝置佈建服務，以及將兩項服務連結在一起。 

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。


## <a name="create-an-iot-hub"></a>建立 IoT 中樞

[!INCLUDE [iot-hub-quickstarts-create-hub](../../includes/iot-hub-quickstarts-create-hub.md)]


## <a name="create-a-new-instance-for-the-iot-hub-device-provisioning-service"></a>建立 IoT 中樞裝置佈建服務的新執行個體

1. 按一下 Azure 入口網站左上角的 [建立資源] 按鈕。

2. 針對**裝置佈建服務***搜尋 Marketplace*。 選取 [IoT 中樞裝置佈建服務]，然後按一下 [建立] 按鈕。 

3. 提供新的「裝置佈建服務」執行個體的下列資訊，然後按一下 [建立]。

    * **名稱：** 提供新的「裝置佈建服務」執行個體的唯一名稱。 如果您輸入的名稱可用，則會出現綠色核取記號。
    * **訂用帳戶：** 選擇您想要用於建立此裝置佈建服務執行個體的訂用帳戶。
    * **資源群組：** 此欄位可讓您建立新的資源群組，或選擇現有群組以包含新的執行個體。 選擇包含您先前所建立 IoT 中樞的相同資源群組，例如 **TestResources**。 您可以將所有相關資源一起放在群組中，一併加以管理。 例如，刪除資源群組時，將會刪除該群組中包含的所有資源。 如需詳細資訊，請參閱[使用資源群組管理您的 Azure 資源](../azure-resource-manager/resource-group-portal.md)。
    * **位置：** 選取最接近裝置的位置。

    ![在入口網站刀鋒視窗中輸入關於裝置佈建服務執行個體的基本資訊](./media/quick-setup-auto-provision/create-iot-dps-portal.png)  

4. 按一下通知按鈕，以監視資源執行個體的建立情形。 在服務成功部署後，按一下 [釘選到儀表板]，然後按 [移至資源]。

    ![監視部署通知](./media/quick-setup-auto-provision/pin-to-dashboard.png)

## <a name="link-the-iot-hub-and-your-device-provisioning-service"></a>連結 IoT 中樞與裝置佈建服務

在本節中，您會將組態新增至裝置佈建服務執行個體。 此組態會設定要對其佈建裝置的 IoT 中樞。

1. 按一下 Azure 入口網站左側功能表中的 [所有資源] 按鈕。 選取您在上一節中建立的裝置佈建服務執行個體。  

2. 在裝置佈建服務摘要刀鋒視窗上，選取 [連結的 IoT 中樞]。 按一下位於頂端的 [+ 新增] 按鈕。 

3. 在 [將連結新增至 IoT 中樞] 頁面上，提供下列資訊，以將新「裝置佈建服務」執行個體連結至 IoT 中樞。 然後按一下 [儲存]。 

    * **訂用帳戶：** 選取訂用帳戶，其中包含您想要與新「裝置佈建服務」執行個體連結的 IoT 中樞。
    * **IoT 中樞：** 選取要與新「裝置佈建服務」執行個體連結的 IoT 中樞。
    * **存取原則：** 選取 [iothubowner] 作為認證，以建立與 IoT 中樞的連結。  

    ![在入口網站刀鋒視窗中連結中樞名稱，以連結至裝置佈建服務執行個體](./media/quick-setup-auto-provision/link-iot-hub-to-dps-portal.png)  

3. 您現在應會在 [連結的 IoT 中樞] 刀鋒視窗之下看到所選的中樞。 您可能需要按一下 [重新整理] 以顯示 [連結的 IoT 中樞]。



## <a name="clean-up-resources"></a>清除資源

此集合中的其他快速入門會以本快速入門為基礎。 如果您打算繼續進行後續的快速入門或教學課程，請勿清除在此快速入門中建立的資源。 如果您不打算繼續，請使用下列步驟，在 Azure 入口網站中刪除本快速入門所建立的所有資源。

1. 從 Azure 入口網站的左側功能表中，按一下 [所有資源]，然後選取您的裝置佈建服務。 在 [所有資源] 刀鋒視窗的頂端，按一下 [刪除]。  
2. 從 Azure 入口網站的左側功能表中，按一下 [所有資源]，然後選取您的 IoT 中樞。 在 [所有資源] 刀鋒視窗的頂端，按一下 [刪除]。  

## <a name="next-steps"></a>後續步驟

在本快速入門中，您已部署 IoT 中樞和裝置佈建服務執行個體，並已連結這兩個資源。 若要深入了解如何使用這項設定來佈建模擬裝置，請繼續進行建立模擬裝置的快速入門。

> [!div class="nextstepaction"]
> [建立模擬裝置的快速入門](./quick-create-simulated-device.md)
