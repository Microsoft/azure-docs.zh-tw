---
title: 使用 Power BI 將遠端監視資料視覺化 - Azure | Microsoft Docs
description: 本教學課程使用 Power BI Desktop 和 Cosmos DB，將來自遠端監視解決方案的資料整合為自訂的視覺效果。 如此能讓使用者建置自己的自訂儀表板，並與不在解決方案上的使用者共用它們。
author: asdonald
manager: hegate
ms.author: asdonald
ms.service: iot-accelerators
services: iot-accelerators
ms.date: 05/01/2018
ms.topic: conceptual
ms.openlocfilehash: ae039573cf202059114f23cca86207c117a35ead
ms.sourcegitcommit: f606248b31182cc559b21e79778c9397127e54df
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/12/2018
ms.locfileid: "38970394"
---
# <a name="visualize-remote-monitoring-data-using-power-bi"></a>使用 Power BI 將遠端監視資料視覺化

本教學課程會逐步引導您了解如何將來自 CosmosDB 的遠端監視解決方案資料插入 Power BI。 藉由建立此連線，您就能建立自己的自訂儀表板，並將其新增回遠端監視解決方案儀表板。 這個工作流能讓您建立更多現成圖表之外的特製化圖形。 您接著可以使用本教學課程來整合其他資料流，或建置自訂的儀表板以在遠端監視解決方案外部加以取用。 在 Power BI 中建置儀表板，表示您也可以選取特定部分來讓每個面板彼此互動。 例如，您可以有一個篩選條件只會顯示與模擬卡車有關的資訊，而儀表板的每個部分都會互動來只顯示模擬的卡車資訊。 如果您想要使用 Power BI 以外的工具，也可以擴充這些步驟來使用您偏好的視覺效果工具並連結至 Cosmos 資料庫，或是使用您已設定的自訂資料庫。 

## <a name="prerequisites"></a>先決條件

- 您必須具有正在執行中的遠端監視解決方案
- 您必須能夠存取 [Azure 入口網站](https://portal.azure.com)，以及正在執行 IoT 中樞與解決方案的訂用帳戶
- 您必須已安裝 [Power BI Desktop](https://powerbi.microsoft.com) (任何版本均可)


## <a name="information-needed-from-azure-portal"></a>需要從 Azure 入口網站取得的資訊

1. 瀏覽至 [Azure 入口網站](https://portal.azure.com)並視需要登入

2. 在左側面板上，按一下 [資源群組]

    ![側邊面板導覽](./media/iot-accelerators-integrate-data-powerbi/side_panel.png)

3. 瀏覽至正在執行 IoT 解決方案的資源群組，然後按一下以前往該資源群組的 [概觀] 頁面。 

4. 在該概觀頁面上，按一下類型為 [Azure Cosmos DB 帳戶] 的項目，系統會將您帶往該 IoT 解決方案 Cosmos DB 資料流的概觀頁面。

    ![資源群組](./media/iot-accelerators-integrate-data-powerbi/resource_groups.png)

5. 在左側面板上，按一下 [金鑰] 區段，並記下以下要用於 Power BI 中的值：

    - URI
    - 主索引鍵

    ![金鑰](./media/iot-accelerators-integrate-data-powerbi/keys.png)

## <a name="setting-up-the-stream-in-power-bi"></a>在 Power BI 中設定資料流
  
1. 開啟 Power BI Desktop 應用程式，然後按一下左上角的 [取得資料]。 

    ![[取得資料]](./media/iot-accelerators-integrate-data-powerbi/get_data.png)

2. 當系統要求您輸入資料時，請選擇搜尋 "Azure Cosmos DB"，然後選取此連接器。 此連接器基本上會直接從您 Azure IoT 解決方案的 Cosmos 資料庫提取資料
  
    ![Cosmos DB](./media/iot-accelerators-integrate-data-powerbi/cosmos_db.png)
  
3. 輸入您在上方所記錄的資訊：

    * URI
    * 主索引鍵

4. 選取要匯入 Power BI 的所有資料表。 此動作將會開始載入資料。 您的解決方案執行的時間越長，載入資料的時間就越長 (最多數個小時)。 

    ![匯入資料表](./media/iot-accelerators-integrate-data-powerbi/import_tables.png)

5. 資料完成載入之後，在 Power BI 的頂端列上按一下 [編輯查詢]，然後按一下每個資料表之黃色列中的箭號來展開所有資料表。 這基本上會展開以顯示所有資料行。 您將注意到像是溼度、速度時間等項目的資料類型都不正確。

    ![新增資料行](./media/iot-accelerators-integrate-data-powerbi/new_column.png)
  
    例如，要傳入 Power BI 的資料，在經由連接器傳入時已變更為 UNIX 時間。 若要調整此轉換，您可以建立新的資料行，並使用此方程式來將它轉換成日期時間格式： 

    ```text
    #datetime(1970, 1, 1, 0, 0, 0) + #duration(0, 0, 0, [Document.device.msg.received]/1000)
    ```

    ![更新資料表](./media/iot-accelerators-integrate-data-powerbi/updated_table.png)
  
    Document.device.msg.received 只是其中一個具有 UNIX 格式的資料行，並可使用其他需要轉換的資料行來取代。 
  
    已轉換為 String 類型的其他資料點可使用與上述相同的步驟，適當地變更為 Doubles 或 Int。

## <a name="creating-a-dashboard"></a>建立儀表板

一旦將資料流連線之後，您就已準備好建立個人化的儀表板！ 下列儀表板範例會取得由模擬的裝置所發出的遙測，並顯示其周圍的不同樞紐，例如： 

* 地圖上的裝置位置 (右邊)
* 裝置及其狀態與嚴重性。 (左上方)
* 具有適當規則的裝置，以及是否有任何針對它們發出的警示 (左下方)

![PowerBI 視覺效果](./media/iot-accelerators-integrate-data-powerbi/visual_data.png)

## <a name="publishing-the-dashboard-and-refreshing-the-data"></a>發行儀表板並重新整理資料

成功建立儀表板之後，建議您[發行 Power BI 儀表板](https://docs.microsoft.com/power-bi/desktop-upload-desktop-files)以與其他人共用。

您也會想要在發行的儀表板上[重新整理資料](https://docs.microsoft.com/power-bi/refresh-data)，以確定您具有最新的資料集。

## <a name="next-steps"></a>後續步驟

在本文中，您已了解如何使用 Power BI 來將遠端監視資料視覺化

如需關於自訂遠端監視解決方案的詳細資訊，請參閱：

* [自訂遠端監視解決方案 UI](iot-accelerators-remote-monitoring-customize.md)
* [開發人員參考指南](https://github.com/Azure/azure-iot-pcs-remote-monitoring-dotnet/wiki/Developer-Reference-Guide)
* [開發人員疑難排解指南](https://github.com/Azure/azure-iot-pcs-remote-monitoring-dotnet/wiki/Developer-Troubleshooting-Guide)

