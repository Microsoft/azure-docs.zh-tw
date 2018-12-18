---
title: 使用 Azure IoT 中樞大規模設定和監視 IoT 裝置 | Microsoft Docs
description: 使用 Azure IoT 中樞自動裝置設定來將設定指派給多個裝置
author: ChrisGMsft
manager: bruz
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 04/13/2018
ms.author: chrisgre
ms.openlocfilehash: 2edde122b109779794bb86752d69a5318edb9235
ms.sourcegitcommit: a2ae233e20e670e2f9e6b75e83253bd301f5067c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/13/2018
ms.locfileid: "42145069"
---
# <a name="configure-and-monitor-iot-devices-at-scale-using-the-azure-portal"></a>使用 Azure 入口網站大規模設定和監視 IoT 裝置

[!INCLUDE [iot-edge-how-to-deploy-monitor-selector](../../includes/iot-hub-auto-device-config-selector.md)]

Azure IoT 中樞中的自動裝置管理，可將大批裝置在其生命週期全程中的許多重複且複雜的管理工作自動化。 使用自動裝置管理時，您可以根據裝置的屬性，以特定的裝置集合作為目標來定義所需的設定，並讓 IoT 中樞在這些裝置進入範圍內時更新裝置。  執行此動作的方式是使用自動裝置設定，這也可讓您對完成與合規性進行摘要處理、處理合併和衝突，以及階段式地推出設定。

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-whole.md)]

自動裝置設定的運作方式，是以所需的屬性更新一組裝置對應項，並根據裝置對應項報告屬性來報告摘要。  它推出名為*設定*的新類別和 JSON 文件，其共有三部分：

* **目標條件**會定義要更新的裝置對應項範圍。 目標條件會以查詢的方式指定於裝置對應項標籤和/或報告屬性上。

* **目標內容**會定義要在目標裝置對應項中新增或更新的所需屬性。 內容包含了一個路徑，連往所要變更屬性的區段。

* **計量**會定義各種設定狀態 (例如 **Success**、**Progress** 及 **Error**) 的摘要計數。 自訂計量會以查詢的形式指定於裝置對應項報告屬性上。  系統計量是測量對應項更新狀態的預設計量，例如作為目標的裝置對應項數目，以及成功更新的對應項數目。 

## <a name="implement-device-twins-to-configure-devices"></a>實作裝置對應項來設定裝置

自動裝置設定需要使用裝置對應項，以同步處理雲端和裝置之間的狀態。 如需使用裝置對應項的指引，請參閱[了解和使用 IoT 中樞的裝置對應項](iot-hub-devguide-device-twins.md)。

## <a name="identify-devices-using-tags"></a>使用標記識別裝置

在您可建立設定之前，必須指定所要影響的裝置。 Azure IoT 中樞會使用裝置對應項中的標籤來識別裝置。 每個裝置都可以有多個標記，而您可以利用任何對您解決方案有意義的方式來定義它們。 例如，如果您管理的裝置散置在不同位置，可以將下列標籤新增至裝置對應項：

```json
"tags": {
    "location": {
        "state": "Washington",
        "city": "Tacoma"
    }
},
```

## <a name="create-a-configuration"></a>建立設定

1. 在 [Azure 入口網站](https://portal.azure.com)中，移至您的 IoT 中樞。 

2. 選取 [IoT 裝置組態]。

3. 選取 [新增設定]。

建立設定有五個步驟。 下列各節將逐步解說每一個步驟。 

### <a name="name-and-label"></a>名稱與標籤

1. 為設定指定唯一的名稱，最長為 128 個小寫字母。 避免空格和下列無效字元：`& ^ [ ] { } \ | " < > /`。

2. 新增標籤以協助追蹤您的設定。 標籤是可描述設定的成對「**名稱**, **值**」。 例如，`HostPlatform, Linux` 或 `Version, 3.0.1`。

3. 選取 [下一步] 以移至下一個步驟。 

### <a name="specify-settings"></a>指定設定

本節指定要在目標裝置對應項中設定的目標內容。 每組設定都有兩個輸入。 第一個是裝置對應項路徑，該路徑可前往所要設定對應項屬性內的 JSON 區段。  第二個是要插入該區段的 JSON 內容。 例如，將 [裝置對應項路徑] 和 [內容] 設定如下：

![設定 [裝置對應項路徑] 和 [內容]](./media/iot-hub-auto-device-config/create-configuration-full-browser.png)

您也可以在 [裝置對應項路徑] 中指定完整路徑，並在 [內容] 中指定不含括號的值。 例如，將 [裝置對應項路徑] 設定為 `properties.desired.chiller-water.temperature`，並將 [內容] 設定為 `66`。

如果有兩個以上的設定都以相同的裝置對應項路徑作為目標，則會套用來自優先順序最高設定的內容 (優先順序會在步驟 4 中定義)。

如果您想要移除屬性，請將屬性值指定為 `null`。

您可以選取 [新增裝置對應項設定] 來新增其他設定。

### <a name="specify-metrics-optional"></a>指定計量 (選擇性)

計量能提供在套用設定內容後，裝置所回報各種狀態的摘要計數。 例如，您可以建立擱置設定變更的計量、錯誤的計量，以及成功設定變更的計量。

1. 在 [計量名稱] 輸入名稱。

2. 在 [計量準則] 輸入查詢。  查詢是根據裝置對應項報告屬性。  此計量代表查詢所傳回的資料列數目。

例如︰

```sql
SELECT deviceId FROM devices 
  WHERE properties.reported.chillerWaterSettings.status='pending'
```

您可以加入已套用設定的子句，例如： 

```sql
/* Include the double brackets. */
SELECT deviceId FROM devices 
  WHERE configurations.[[yourconfigname]].status='Applied'
```

### <a name="target-devices"></a>目標裝置

使用裝置對應項的標籤屬性，來將目標設為應該接收此設定的特定裝置。  您也可以依裝置對應項報告屬性來選擇目標裝置。

由於多個設定可能會以相同裝置為目標，您應該為每個設定提供優先順序號碼。 如果發生衝突，則會優先選擇優先順序最高的設定。 

1. 為設定的 [優先順序] 輸入一個正整數。 最高的數值會視為最高優先順序。 如果有兩個設定具有相同的優先順序號碼，則會優先選擇最新建立的設定。 

2. 輸入 [目標條件] 來判斷這個設定會將哪些裝置設為目標。 條件會以裝置對應項標籤或裝置對應項報告屬性為基礎，且應符合運算式格式。 例如，`tags.environment='test'` 或 `properties.reported.chillerProperties.model='4000x'`。 您可以指定 `*` 以將所有裝置設為目標。

3. 選取 [下一步] 移到最後一個步驟。

### <a name="review-configuration"></a>檢閱設定

檢閱您的設定資訊，然後選取 [提交]。

## <a name="monitor-a-configuration"></a>監視設定

若要檢視設定的詳細資料，並監視正在執行該設定的裝置，請使用下列步驟：

1. 在 [Azure 入口網站](https://portal.azure.com)中，移至您的 IoT 中樞。 

2. 選取 [IoT 裝置組態]。

3. 檢查設定清單。 針對每個設定，您可以檢視下列詳細資料：

   * **識別碼**：設定的名稱。

   * **目標條件**：用來定義目標裝置的查詢。

   * **優先順序**：指派給設定的優先順序號碼。

   * **建立時間**：建立設定時的時間戳記。 當有兩個具有相同優先順序的設定時，系統會使用此時間戳記來判斷應優先選擇的設定。 

   * **系統計量**：由 IoT 中樞所計算且無法由開發人員自訂的計量。 [目標] 會指定符合目標條件的裝置對應項數目。 [套用] 會指定由設定所修改的裝置對應項數目，這在有優先順序較高的個別設定同時也做出變更的情況下，可能會包含局部修改。 

   * **自訂計量**：由開發人員針對裝置對應項報告屬性指定為查詢的計量。  每個設定最多可定義五個自訂計量。 
   
4. 選取您想要監視的設定。  

5. 檢查設定詳細資料。 針對接收設定的裝置，您可以使用索引標籤來檢視裝置的特定詳細資料。

   * **目標條件**：符合目標條件的裝置。 

   * **計量**：系統計量和自訂計量的清單。  您可以選取下拉式清單中的計量，然後選取 [檢視裝置] 來檢視每個計量所計算裝置的清單。

   * **裝置對應項設定**：由設定所設定的裝置對應項設定。 

   * **設定標籤**：用來描述設定的機碼值組。  標籤對功能沒有任何影響。 

## <a name="modify-a-configuration"></a>修改設定

當您修改設定時，所做的變更會立即複寫到所有目標裝置。 

如果您更新目標條件，就會發生下列更新：

* 如果裝置對應項不符合舊的目標條件，但符合新的目標條件，而且此設定為該裝置對應項的最高優先順序設定，則會將此設定套用到該裝置對應項。 

* 如果裝置對應項不再符合目標條件，系統會將設定中的設定移除，並以優先順序次高的設定來修改該裝置對應項。 

* 如果目前執行此設定的裝置對應項不再符合目標條件，也不符合任何其他設定的目標條件，則系統會將設定中的設定移除，且不會對該對應項進行任何其他變更。 

若要修改設定，請使用下列步驟： 

1. 在 [Azure 入口網站](https://portal.azure.com)中，移至您的 IoT 中樞。 

2. 選取 [IoT 裝置組態]。 

3. 選取您想要修改的設定。 

4. 對下列欄位進行更新： 

   * 目標條件 
   * 標籤 
   * 優先順序 
   * 度量

4. 選取 [ **儲存**]。

5. 依照[監視設定](#monitor-a-configuration)中的步驟來監看所推出的變更。 

## <a name="delete-a-configuration"></a>刪除設定

當您刪除設定時，任何裝置對應項都會採用其優先順序次高的設定。 如果裝置對應項不符合任何其他設定的目標條件，則不會套用任何其他設定。 

1. 在 [Azure 入口網站](https://portal.azure.com)中，移至您的 IoT 中樞。 

2. 選取 [IoT 裝置組態]。 

3. 使用核取方塊來選取您想要刪除的設定。 

4. 選取 [刪除] 。

5. 您會看到要求確認的提示。

## <a name="next-steps"></a>後續步驟

在本文中，您已了解如何大規模設定和監視 IoT 裝置。 遵循下列連結以深入了解如何管理 Azure IoT 中樞：

* [管理大量的 IoT 中樞裝置身分識別](iot-hub-bulk-identity-mgmt.md)
* [IoT 中樞計量](iot-hub-metrics.md)
* [作業監視](iot-hub-operations-monitoring.md)

若要進一步探索 IoT 中樞的功能，請參閱︰

* [IoT 中樞開發人員指南](iot-hub-devguide.md)
* [使用 Azure IoT Edge 將 AI 部署到 Edge 裝置](../iot-edge/tutorial-simulate-device-linux.md)

若要探索使用 IoT 中樞裝置佈建服務進行 Just-In-Time 自動佈建，請參閱： 

* [Azure IoT 中樞裝置佈建服務](/azure/iot-dps)
