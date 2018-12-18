---
title: 手動容錯移轉 Azure IoT 中樞 | Microsoft Docs
description: 示範如何執行手動容錯移轉 Azure IoT 中樞
author: robinsh
manager: timlt
ms.service: iot-hub
services: iot-hub
ms.topic: tutorial
ms.date: 07/11/2018
ms.author: robinsh
ms.custom: mvc
ms.openlocfilehash: f0e8bf922f142b795dd1a2ded4b3ec265c43481a
ms.sourcegitcommit: 156364c3363f651509a17d1d61cf8480aaf72d1a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/25/2018
ms.locfileid: "39249935"
---
# <a name="tutorial-perform-manual-failover-for-an-iot-hub-public-preview"></a>教學課程：執行手動容錯移轉 IoT 中樞 (公開預覽)

手動容錯移轉是 IoT 中樞服務的一項功能，可讓客戶將其中樞的作業從主要區域[容錯移轉](https://en.wikipedia.org/wiki/Failover)到對應的 Azure 異地配對區域。 發生區域性災難或延伸服務中斷的情況時，可以進行手動容錯移轉。 您也可以執行規劃的容錯移轉來測試您的災害復原功能，但建議使用測試 IoT 中樞，而不是生產環境中執行的 IoT 中樞。 對客戶提供手動容錯移轉功能無需額外費用。

在本教學課程中，您會執行下列工作：

> [!div class="checklist"]
> * 使用 Azure 入口網站來建立 IoT 中樞。 
> * 執行容錯移轉。 
> * 查看在次要位置中執行的中樞。
> * 執行容錯回復以將 IoT 中樞的作業傳回主要位置。 
> * 確認中樞在正確的位置中正確執行。

## <a name="prerequisites"></a>必要條件

- Azure 訂用帳戶。 如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

## <a name="create-an-iot-hub"></a>建立 IoT 中樞

1. 登入 [Azure 入口網站](https://portal.azure.com)。 

2. 按一下 [+ 建立資源]，然後依序選取 [物聯網]、[IoT 中樞]。

   ![顯示建立 IoT 中樞的螢幕擷取畫面](./media/tutorial-manual-failover/create-hub-01.png)

3. 選取 [基本] 索引標籤。填寫下列欄位。

    **訂用帳戶**：選取您需要使用的 Azure 訂用帳戶。

    **資源群組**：按一下 [建立新項目]，然後在資源群組指定 **ManlFailRG**。

    **區域**：選取您鄰近且屬於預覽的區域。 本教學課程會使用 `westus2`。 只能在 Azure 異地配對區域之間進行容錯移轉。 與 westus2 異地配對的區域是 WestCentralUS。
    
   > [!NOTE]
   > 手動容錯移轉目前處於公開預覽狀態，且在以下的 Azure 區域「不」提供：美國東部、美國西部、北歐、西歐、巴西南部和美國中南部。

   **IoT 中樞名稱**：指定 IoT 中樞的名稱。 中樞名稱必須是全域唯一的。 

   ![顯示建立 IoT 中樞 [基本] 窗格的螢幕擷取畫面](./media/tutorial-manual-failover/create-hub-02-basics.png)

   按一下 [檢閱 + 建立]。 (它會使用大小和級別的預設值。) 

4. 請檢閱資訊，然後按一下 [建立] 來建立 IoT 中樞。 

   ![顯示建立 IoT 中樞最後一個步驟的螢幕擷取畫面](./media/tutorial-manual-failover/create-hub-03-create.png)

## <a name="perform-a-manual-failover"></a>執行手動容錯移轉

請注意，IoT 中樞每天有兩次容錯移轉和兩次容錯回復的限制。

1. 按一下 [資源群組]，然後選取資源群組 **ManlFailRG**。 在資源清單中，按一下您的中樞。 

2. 在 [IoT 中樞] 窗格的 [復原] 底下，按一下 [手動容錯移轉 (預覽)]。 請注意，如果您的中樞不是在有效的區域中設定，將會停用手動容錯移轉選項。

   ![顯示 [IoT 中樞屬性] 窗格的螢幕擷取畫面](./media/tutorial-manual-failover/trigger-failover-01.png)

3. 在 [手動容錯移轉] 窗格中，您會看到 [IoT 中樞的主要位置] 和 [IoT 中樞的次要位置]。 主要位置一開始會設為您在建立 IoT 中樞時所指定的位置，且一律會指出中樞目前的使用中位置。 次要位置是標準 [Azure 異地配對區域](../best-practices-availability-paired-regions.md)，會與主要位置配對。 您無法變更位置的值。 在本教學課程中，主要位置是 `westus2`，而次要位置為 `WestCentralUS`。

   ![顯示 [手動容錯移轉] 窗格的螢幕擷取畫面](./media/tutorial-manual-failover/trigger-failover-02.png)

3. 在 [手動容錯移轉] 窗格頂端，按一下 [起始容錯移轉]。 您會看到 [確認手動容錯移轉] 窗格。 填寫您 IoT 中樞的名稱，以確認它是您想要容錯移轉的 IoT 中樞。 然後，若要起始容錯移轉，請按一下 [確定]。

   執行手動容錯移轉所花費的時間與您中樞註冊的裝置數目成正比。 例如，如果您有 100,000 部裝置，可能需要 15 分鐘，但如果您有 5,000,000 部裝置，則可能需要一小時或更久。

4. 在 [確認手動容錯移轉] 窗格中，填寫您 IoT 中樞的名稱，以確認它是您想要容錯移轉的 IoT 中樞。 然後，若要起始容錯移轉，請按一下 [確定]。 

   ![顯示 [手動容錯移轉] 窗格的螢幕擷取畫面](./media/tutorial-manual-failover/trigger-failover-03-confirm.png)

   執行手動容錯移轉程序時，[手動容錯移轉] 窗格中會出現橫幅，告知您正在進行手動容錯移轉。 

   ![顯示 [手動容錯移轉] 進行中的螢幕擷取畫面](./media/tutorial-manual-failover/trigger-failover-04-in-progress.png)

   如果您關閉 [IoT 中樞] 窗格，然後按一下 [資源群組] 窗格再次將它開啟，您會看到橫幅，告知您的中樞不在使用中狀態。 

   ![顯示 [IoT 中樞] 非使用中的螢幕擷取畫面](./media/tutorial-manual-failover/trigger-failover-05-hub-inactive.png)

   完成之後，會翻轉 [手動容錯移轉] 頁面上的主要和次要地區，且中樞會回到使用中狀態。 在此範例中，主要位置現在是 `WestCentralUS`，而次要位置為 `westus2`。 

   ![顯示容錯移轉已完成的螢幕擷取畫面](./media/tutorial-manual-failover/trigger-failover-06-finished.png)

## <a name="perform-a-failback"></a>執行容錯移轉 

在您執行手動容錯移轉之後，可以將中樞的作業切換回原始的主要區域 -- 這稱為容錯回復。 如果您剛執行完容錯移轉，必須等候大約一小時，才能要求容錯回復。 如果您嘗試在短時間內執行容錯回復，就會顯示一則錯誤訊息。

容錯回復的執行方式就如同手動容錯移轉一樣。 步驟如下： 

1. 若要執行容錯回復，請返回您 Iot 中樞的 [Iot 中樞] 窗格。

2. 在 [IoT 中樞] 窗格的 [復原] 底下，按一下 [手動容錯移轉 (預覽)]。 

3. 在 [手動容錯移轉] 窗格頂端，按一下 [起始容錯移轉]。 您會看到 [確認手動容錯移轉] 窗格。 

4. 在 [確認手動容錯移轉] 窗格中，填寫您 IoT 中樞的名稱，以確認它是您想要容錯回復的 IoT 中樞。 接著，若要起始容錯回復，請按一下 [確定]。 

   ![手動容錯回復要求的螢幕擷取畫面](./media/tutorial-manual-failover/trigger-failback-01-regions.png)

   橫幅的顯示方式會如同[執行容錯移轉](#perform-a-failover)一節中所述。 容錯回復完成之後，它會如同最初的設定，再次顯示 `westus2` 作為主要位置，以及顯示 `WestCentralUS` 作為次要位置。

## <a name="clean-up-resources"></a>清除資源 

若要將您已針對本教學課程所建立的資源移除，請刪除資源群組。 此動作會同時刪除群組內含的所有資源。 在此情況下，它會移除 IoT 中樞和資源群組本身。 

1. 按一下 [資源群組] 。 

2. 請找出並選取資源群組 **ManlFailRG**。 按一下開啟它。 

3. 按一下 [刪除資源群組]。 系統出現提示時，請輸入資源群組的名稱，然後按一下 [刪除] 以確認。 

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何設定及執行手動容錯移轉，以及如何執行下列工作來要求容錯回復：

> [!div class="checklist"]
> * 使用 Azure 入口網站來建立 IoT 中樞。 
> * 執行容錯移轉。 
> * 查看在次要位置中執行的中樞。
> * 執行容錯回復以將 IoT 中樞的作業傳回主要位置。 
> * 確認中樞在正確的位置中正確執行。

前進至下一個教學課程，以了解如何管理 IoT 裝置的狀態。 

> [!div class="nextstepaction"]
[管理 IoT 裝置的狀態](tutorial-device-twins.md)