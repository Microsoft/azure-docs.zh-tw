---
title: 設定連線的處理站拓撲 | Microsoft Docs
description: 如何設定連線處理站解決方案加速器的拓撲。
author: dominicbetts
manager: timlt
ms.service: iot-accelerators
services: iot-accelerators
ms.topic: conceptual
ms.date: 12/12/2017
ms.author: dobett
ms.openlocfilehash: 8cb3cae396016545c5d78a2ff7ccde4a053c4cf1
ms.sourcegitcommit: b5ac31eeb7c4f9be584bb0f7d55c5654b74404ff
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/23/2018
ms.locfileid: "42746974"
---
# <a name="configure-the-connected-factory-solution-accelerator"></a>設定連線的處理站解決方案加速器

連線的處理站解決方案加速器會針對虛構公司 Contoso 顯示模擬的儀表板。 此公司在全球許多地點都有工廠。

本文使用 Contoso 作為範例，來說明如何設定連線處理站解決方案的拓撲。

## <a name="simulated-factories-configuration"></a>模擬的工廠設定

每個 Contoso 工廠都有各由三個站台所組成的生產線。 每個站台都是具有特定角色的實際 OPC UA 伺服器：

* 組件站台
* 測試站台
* 封裝站台

這些 OPC UA 伺服器會讓 OPC UA 節點和 [OPC 發行者](https://github.com/Azure/iot-edge-opc-publisher) \(英文\) 將這些節點的值傳送至連線的處理站。 其中包括：

* 目前的操作狀態，例如目前的耗電量。
* 生產資訊，例如所生產的產品數目。

您可以使用儀表板將 Contoso 工廠拓撲從全球檢視向下鑽研到站台層級檢視。 連線的處理站儀表板允許：

* 針對拓撲中的每個圖層，將 OEE 和 KPI 數據視覺化。
* 將站台中 OPC UA 節點的目前值視覺化。
* 將 OEE 和 KPI 數據從站台層級彙總到全球層級。
* 在值達到特定閾值時，將警示和要執行的動作視覺化。

## <a name="connected-factory-topology"></a>連線的處理站拓撲

工廠、生產線和站台的拓撲是階層式：

* 全球層級將工廠節點當作子系。
* 工廠將生產線節點當作子系。
* 生產線將站台節點當作子系。
* 站台 (OPC UA 伺服器) 將 OPC UA 節點當作子系。

拓撲中的每個節點都有一組共同的屬性，這些屬性定義：

* 拓撲節點的唯一識別碼。
* 名稱。
* 描述。
* 影像。
* 拓撲節點的子系。
* OEE 和 KPI 數據的最小值、目標值和最大值與要執行的警示動作。

## <a name="topology-configuration-file"></a>拓撲設定檔

為了設定上一節中所列的屬性，連線的處理站解決方案會使用稱為 [ContosoTopologyDescription.json](https://github.com/Azure/azure-iot-connected-factory/blob/master/WebApp/Contoso/Topology/ContosoTopologyDescription.json) 的設定檔。

您可以在 `WebApp/Contoso/Topology` 資料夾的解決方案原始程式碼中找到這個檔案。

下列程式碼片段顯示 `ContosoTopologyDescription.json` 設定檔的大綱：

```json
{
  <global_configuration>,
  "Factories": [
    <factory_configuration>,
    "ProductionLines": [
      <production_line_configuration>,
      "Stations": [
        <station_configuration>,
        <more station_configurations>
      ],
      <more production_line_configurations>
    ]
    <more factory_configurations>
  ]
}
```

`<global_configuration>`、`<factory_configuration>`、`<production_line_configuration>` 和 `<station_configuration>` 的通用屬性是：

* **Name** (字串類型)

  定義描述性名稱，這應該是拓撲節點在儀表板中顯示的唯一一個文字。

* **Description** (字串類型)

  更詳細描述拓撲節點。

* **Image** (字串類型)

  WebApp 解決方案中影像的路徑，可在儀表板中顯示拓撲節點相關資訊時顯示。

* **OeeOverall**、**OeePerformance**、**OeeAvailability**、**OeeQuality**、**Kpi1**、**Kpi2** (類型 `<performance_definition>`)

  這些屬性定義產生警示所使用之操作數據的最小值、目標值和最大值。 這些屬性也會定義偵測到警示時要執行的動作。

`<factory_configuration>` 和 `<production_line_configuration>` 項目具有屬性：

* **Guid** (字串類型)

  可唯一識別拓撲節點。

`<factory_configuration>` 具有屬性：

* **Location** (類型 `<location_definition>`)

  指定工廠所在位置。

`<station_configuration>` 具有屬性：

* **OpcUri** (字串類型)

  此屬性必須設定為 OPC UA 伺服器的 OPC UA 應用程式 URI。
  依照 OPC UA 規格，此屬性必須是全域唯一的，因此可用來識別站台拓撲節點。

* **OpcNodes**，它是 OPC UA 節點的陣列 (類型 `<opc_node_description>`)

`<location_definition>` 具有屬性：

* **City** (字串類型)

  最接近位置之城市的名稱

* **Country** (字串類型)

  位置所在的國家/地區

* **Latitude** (雙精確度類型)

  位置的緯度

* **Longitude** (雙精確度類型)

  位置的經度

`<performance_definition>` 具有屬性：

* **Minimum** (雙精確度類型)

  值可達到的閾值下限。 如果目前的值低於此閾值，則會產生警示。

* **Target** (雙精確度類型)

  理想的目標值。

* **Maximum** (雙精確度類型)

  值可達到的閾值上限。 如果目前的值高於此閾值，則會產生警示。

* **MinimumAlertActions** (類型 `<alert_action>`)

  定義可當作最小值警示回應所採取的一組動作。

* **MaximumAlertActions** (類型 `<alert_action>`)

  定義可當作最大值警示回應所採取的一組動作。

`<alert_action`> 具有屬性：

* **Type** (字串類型)

  警示動作的類型。 下面列出已知類型：

  * **AcknowledgeAlert**：警示的狀態應該變更為已認可。
  * **CloseAlert**：相同類型的所有較舊警示應該不會再顯示在儀表板中。
  * **CallOpcMethod**：應該呼叫 OPC UA 方法。
  * **OpenWebPage**：應該開啟可顯示其他內容相關資訊的瀏覽器視窗。

* **Description** (字串類型)

  儀表板中顯示的動作描述。

* **Parameter** (字串類型)

  執行動作所需的參數。 此值取決於動作類型。

  * **AcknowledgeAlert**：不需要參數。
  * **CloseAlert**：不需要參數。
  * **CallOpcMethod**：以「父節點的 NodeId、要呼叫之方法的 NodeId、OPC UA 伺服器的 URI」格式呼叫之 OPC UA 方法的節點資訊與參數。
  * **OpenWebPage**：要在瀏覽器視窗中顯示的 URL。

`<opc_node_description>` 包含站台 (OPC UA 伺服器) 中 OPC UA 節點的相關資訊。 目前未呈現任何 OPC UA 節點，但要在連線處理站計算邏輯中用來作為儲存體的節點也有效。 它具有下列屬性：

* **NodeId** (字串類型)

  站台 (OPC UA 伺服器) 的位址空間中 OPC UA 節點的位址。 語法必須如在 OPC UA 規格中為 NodeId 所指定的語法。

* **SymbolicName** (字串類型)

  顯示此 OPC UA 節點的值時，要在儀表板中顯示的名稱。

* **Relevance** (字串類型的陣列)

  指出與 OPC UA 節點值相關的 OEE 或 KPI 計算。 每個陣列元素都可以是下列其中一個值：

  * **OeeAvailability_Running**：此值與 OEE 可用性的計算相關。
  * **OeeAvailability_Fault**：此值與 OEE 可用性的計算相關。
  * **OeePerformance_Ideal**：此值與 OEE 效能的計算相關，而且通常是常數值。
  * **OeePerformance_Actual**：此值與 OEE 效能的計算相關。
  * **OeeQuality_Good**：此值與 OEE 品質的計算相關。
  * **OeeQuality_Bad**：此值與 OEE 品質的計算相關。
  * **Kpi1**：此值與 KPI1 的計算相關。
  * **Kpi2**：此值與 KPI2 的計算相關。

* **OpCode** (字串類型)

  指出 OPC UA 節點的值在「時間序列深入解析」查詢和 OEE/KPI 計算中的處理方式。 每個「時間序列深入解析」查詢的目標都是特定的時間範圍，也就是查詢的參數，並提供結果。 OpCode 可控制結果的計算方式，而且可以是下列其中一個值：

  * **Diff**：時間範圍中最後一個與第一個值之間的差異。
  * **Avg**：時間範圍中所有值的平均值。
  * **Sum**：時間範圍中所有值的總和。
  * **Last**：目前未使用。
  * **Count**：時間範圍中的值數目。
  * **Max**：時間範圍中的最大值。
  * **Min**：時間範圍中的最小值。
  * **Const**：結果是 ConstValue 屬性所指定的值。
  * **SubMaxMin**：最大值和最小值之間的差異。
  * **Timespan**：時間範圍。

* **Units** (字串類型)

  定義要在儀表板中顯示之值的單位。

* **Visible** (布林值類型)

  控制是否應該在儀表板中顯示值。

* **ConstValue** (雙精確度類型)

  如果 **OpCode** 是 **Const**，則這個屬性為節點的值。

* **Minimum** (雙精確度類型)

  如果目前的值低於此值，則會產生最小警示。

* **Maximum** (雙精確度類型)

  如果目前的值高於此值，則會產生最大警示。

* **MinimumAlertActions** (類型 `<alert_action>`)

  定義可當作最小值警示回應所採取的一組動作。

* **MaximumAlertActions** (類型 `<alert_action>`)

  定義可當作最大值警示回應所採取的一組動作。

在站台層級中，您也會看到**模擬**物件。 這些物件只能用於設定連線的處理站模擬，不應用於設定實際拓撲。

## <a name="how-the-configuration-data-is-used-at-runtime"></a>設定資料在執行階段的使用方式

設定檔中使用的所有屬性都可以根據使用方式而分成不同的類別。 這些類別包括：

### <a name="visual-appearance"></a>視覺外觀

此分類中的屬性會定義連線處理站儀表板的視覺外觀。 範例包括：

* 名稱
* 說明
* 映像
* 位置
* Units
* 可見

### <a name="internal-topology-tree-addressing"></a>內部拓樸樹狀結構定址

WebApp 可維護其中包含所有拓撲節點相關資訊的內部資料字典。 **Guid** 和 **OpcUri** 屬性是當作存取此字典的索引鍵使用，因此必須是唯一的。

### <a name="oeekpi-computation"></a>OEE/KPI 計算

連線處理站模擬的 OEE/KPI 數據會透過下列值進行參數化：

* 要包含在計算中的 OPC UA 節點值。
* 如何從遙測值計算數據。

連線的處理站會使用如 http://www.oeefoundation.org 所發行的 OEE 公式。

站台中的 OPC UA 節點物件會啟用可在 OEE/KPI 計算中使用的標記。 **Relevance** 屬性指出 OPC UA 節點值應該用於哪個 OEE/KPI 數據。 **OpCode** 屬性定義如何將值包含在計算中。

### <a name="alert-handling"></a>警示處理

連線的處理站支援簡易的最小/最大閾值型警示產生機制。 為回應那些警示，有數個可以設定的預先定義動作。 下列屬性可控制這個機制：

* 最大值
* 最小值
* MaximumAlertActions
* MinimumAlertActions

## <a name="correlating-to-telemetry-data"></a>與遙測資料相互關聯

針對某些作業 (例如，將最後一個值視覺化，或建立「時間序列深入解析」查詢)，WebApp 需要定址配置，以供內嵌的遙測資料使用。 傳送至連線處理站的遙測也必須儲存於內部資料結構中。 啟用這些作業的兩個屬性分別位於站台 (OPC UA 伺服器) 和 OPC UA 節點層級：

* **OpcUri**

  識別遙測來自哪部 (全球唯一的) OPC UA 伺服器。 在內嵌的訊息中，此屬性會當作 **ApplicationUri** 傳送。

* **NodeId**

  識別 OPC UA 伺服器中的節點值。 此屬性的格式必須是如在 OPC UA 規格中所指定。 在內嵌訊息中，此屬性會當作 **NodeId** 傳送。

如需如何使用 OPC 發行者來將遙測資料內嵌於連線處理站的詳細資訊，請查閱[這個](https://github.com/Azure/iot-edge-opc-publisher) \(英文\) GitHub 頁面。

## <a name="example-how-kpi1-is-calculated"></a>範例：如何計算 KPI1

`ContosoTopologyDescription.json` 檔案中的設定可控制 OEE/KPI 數據的計算方式。 下列範例會示範此檔案中的屬性如何控制 KPI1 的計算。

在連線的處理站中，KPI1 用於測量過去一小時內成功製造的產品數目。 連線處理站模擬中的每個站台 (OPC UA 伺服器) 都提供一個 OPC UA 節點 (`NodeId: "ns=2;i=385"`)，此節點可提供用來計算此 KPI 的遙測。

此 OPC UA 節點的設定看起來像下列程式碼片段︰

```json
{
  "NodeId": "ns=2;i=385",
  "SymbolicName": "NumberOfManufacturedProducts",
  "Relevance": [ "Kpi1", "OeeQuality_Good" ],
  "OpCode": "SubMaxMin"
},
```

此設定允許使用「時間序列深入解析」來查詢此節點的遙測值。 「時間序列深入解析」查詢會擷取：

* 值數目。
* 最小值。
* 最大值。
* 所有值的平均值。
* 指定的時間範圍內，所有唯一 **OpcUri** (**ApplicationUri**)、**NodeId** 組中的所有值總和。

**NumberOfManufactureredProducts** 節點值的其中一個特性是它只會增加。 為了計算時間範圍內製造的產品數目，連線的處理站會使用 **OpCode** **SubMaxMin**。 計算會在時間範圍開始時擷取最小值，並在時間範圍結束時擷取最大值。

設定中的 **OpCode** 會設定計算邏輯，以計算最大值和最小值差異的結果。 接著，這些結果會由下而上累積到根 (全球) 層級，並顯示在儀表板中。

## <a name="next-steps"></a>後續步驟

下一個建議步驟是了解如何[在 Windows 或 Linux 上部署連線的處理站解決方案加速器的閘道](iot-accelerators-connected-factory-gateway-deployment.md)。
