---
title: 在 Azure IoT Central 應用程式中設定裝置範本 | Microsoft Docs
description: 了解如何使用量測、設定、屬性、規則和儀表板來設定裝置範本。
author: viv-liu
ms.author: viviali
ms.date: 04/16/2018
ms.topic: conceptual
ms.service: iot-central
services: iot-central
manager: peterpr
ms.openlocfilehash: a076ee5275a4d053613902a1980542590263385c
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47034264"
---
# <a name="set-up-a-device-template"></a>設定裝置範本

裝置範本是一份藍圖，可為連線至 Microsoft Azure IoT Central 應用程式的某種裝置，定義其所具有的特性和行為。

例如，建置者可為 IoT 連線風扇建立包含下列項目的裝置範本：

- 溫度遙測量測

- 風扇馬達錯誤事件量測

- 風扇運轉狀態量測

- 風扇速度設定

- 位置屬性

- 可傳送警示的規則

- 可讓您檢視裝置整體的儀表板

透過此裝置範本，操作員可以使用 **fan-1** 和 **fan-2** 等名稱，建立真實的風扇裝置並將其連網。 所有這些風扇都有量測、設定、屬性、規則和儀表板，可供應用程式使用者監視和管理。

> [!NOTE]
> 只有建置者和系統管理員可以建立、編輯和刪除裝置範本。 任何使用者都可以在 [Device Explorer] 頁面，透過現有裝置範本來建立裝置。

## <a name="create-a-device-template"></a>建立裝置範本

1. 移至 [應用程式建置者] 頁面。

2. 若要建立空白範本，請選取 [建立裝置範本]，然後選取 [自訂]。

3. 輸入新裝置範本的名稱，然後選取 [建立]。

   ![範本名稱為 "Refrigerator" 的裝置詳細資料頁面](./media/howto-set-up-template/devicedetailspage.png)

4. 現在您位於新模擬裝置的 [裝置詳細資料] 頁面。 當您建立裝置範本時，系統會自動為您建立模擬裝置。 模擬裝置會報告資料，並可透過和真實裝置一樣的方式來控制。

現在，請看看 [裝置詳細資料] 頁面上的每個索引標籤。

## <a name="measurements"></a>量測

量測是來自裝置的資料。 您可以在裝置範本中新增多個量測，以符合裝置的功能。

- **遙測**量測是您裝置隨著時間收集的資料點。 這些資料點會以連續資料流的形式呈現。 例如，溫度。
- **事件**量測是時間點資料，代表裝置上某個重要的事物。 嚴重性層級代表事件的重要性。 例如，風扇馬達錯誤。
- **狀態**量測代表裝置或其元件在一段時間內的狀態。 例如，風扇模式可定義為具有 [運轉中] 和 [已停止] 這兩種可能的狀態。

### <a name="create-a-telemetry-measurement"></a>建立遙測量測
若要新增遙測量測值，請選取 [Edit Template] \(編輯範本\)，然後按一下 [+ New Measurement] \(+ 新增量測值\) 按鈕。 選取 [遙測] 作為量測類型，然後在 [建立遙測] 表單上輸入詳細資料。

> [!NOTE]
> 將真實裝置連線時，請注意裝置所回報的量測名稱。 該名稱必須與量測的 [欄位名稱] 項目完全相符。

例如，您可以新增溫度遙測量測：

![含溫度量測詳細資料的 [建立遙測] 表單](./media/howto-set-up-template/measurementsform.png)

選取 [Done] \(完成\) 之後，[Temperature] \(溫度\) 量測值就會出現在量測值清單中。 操作員可以看到裝置正在收集的溫度資料以視覺化方式呈現。

> [!NOTE]
  遙測量值的資料類型為 double。

### <a name="create-an-event-measurement"></a>建立事件量測
若要新增事件量測值，請選取 [Edit Template] \(編輯範本\)，然後按一下 [+ New Measurement] \(+ 新增量測值\) 按鈕。 選取 [事件] 作為量測類型，然後在 [建立事件] 表單上輸入詳細資料。

提供事件的 [顯示名稱]、[欄位名稱] 及 [嚴重性]。 可供選擇的嚴重性層級有三個 - [錯誤]、[警告] 及 [資訊]。  

例如，您可以新增 [風扇馬達錯誤] 事件。

![含風扇馬達事件詳細資料的 [建立事件] 表單](./media/howto-set-up-template/eventmeasurementsform.png)

選取 [Done] \(完成\) 之後，[Fan Motor Error] \(風扇馬達錯誤\) 量測值就會出現在量測值清單中。 操作員可以看到裝置正在傳送的事件資料以視覺化方式呈現。

![事件測量圖表](./media/howto-set-up-template/eventmeasurementschart.png)

若要檢視事件的更多相關詳細資料，請選取圖表上的事件圖示。

![[風扇馬達錯誤] 事件的詳細資料](./media/howto-set-up-template/eventmeasurementsdetail.png)

> [!NOTE]
  事件量值的資料類型為字串。

### <a name="create-a-state-measurement"></a>建立狀態量測
若要新增狀態量測值，請選取 [Edit Template] \(編輯範本\)，然後按一下 [+ New Measurement] \(+ 新增量測值\) 按鈕。 選取 [狀態] 作為量測類型，然後在 [建立事件] 表單上輸入詳細資料。

提供狀態之 [顯示名稱]、[欄位名稱] 及 [值] 的詳細資料。 每個值也可以有顯示名稱，當值顯示在圖表和資料表上時，便會使用該顯示名稱。

例如，您可以新增 [風扇模式] 狀態，使其包含兩個可由裝置傳送的可能值，分別是 [運轉中] 和 [已停止]。

![含風扇模式詳細資料的 [編輯狀態] 表單](./media/howto-set-up-template/statemeasurementsform.png)

選取 [Done] \(完成\) 之後，[Fan Mode] \(風扇模式\) 狀態量測值就會出現在量測值清單中。 操作員可以看到裝置正在傳送的狀態資料以視覺化方式呈現。

![狀態測量圖表](./media/howto-set-up-template/statemeasurementschart.png)

如果裝置在一小段時間內傳送太多資料點，狀態測量就會以不同的視覺效果顯示，如以下螢幕擷取畫面所示。 如果您按一下圖表，則該時段內的所有資料點將會依時間先後順序顯示。 您也可以縮小時間範圍，以查看圖表上所繪製的量測。

> [!NOTE]
  狀態量值的資料類型為字串。

## <a name="settings"></a>設定

設定可控制裝置。 設定可讓應用程式操作員對裝置提供輸入。 您可以在裝置範本中新增多個設定，這些設定會在 [設定] 索引標籤上顯示為圖格供操作員使用。 您可以新增六種類型的設定：數字、文字、日期、切換、挑選清單及區段標籤。

> [!NOTE]
> 當真實裝置連線時，請注意裝置所回報的設定名稱。 該名稱必須與設定的 [欄位名稱] 項目完全相符。

設定的狀態可為三者其中之一。 裝置會回報下列狀態。

- **已同步處理**：裝置已變更來反映設定值。

- **擱置**：裝置目前正在變更為設定值。

- **錯誤**：裝置傳回錯誤。

例如，您可以選取 [Edit Template] \(編輯範本\) 並輸入新設定，來新增新的風扇速度設定：

![含速度設定詳細資料的 [設定數字] 表單](./media/howto-set-up-template/settingsform.png)

選取 [儲存] 之後，[風扇速度] 設定就會顯示為圖格，並已可供用來變更裝置的風扇速度。

在建立新圖格之後，您便可以試用新的設定。 首先，選取螢幕右上方的 [Done] \(完成\)。

![含圖格 [設計模式] 切換的 [設定] 索引標籤](./media/howto-set-up-template/settingstile.png)

## <a name="properties"></a>properties

屬性是與裝置相關聯的裝置中繼資料，例如裝置位置和序號。 您可以在裝置範本中新增多個屬性，這些屬性會在 [屬性] 索引標籤上顯示為圖格。操作員可以在建立裝置時指定屬性值，並可隨時編輯這些值。 您可以新增六種類型的屬性：數字、文字、日期、切換、裝置屬性及標籤。

屬性分為兩種類別：

- 裝置回報的「裝置」屬性。
- 僅儲存在應用程式中的「應用程式」屬性。 裝置無法辨識應用程式屬性。

> [!NOTE]
> 針對裝置屬性，當真實裝置連線時，請注意裝置所回報的屬性名稱。 該名稱必須與屬性的 [欄位名稱] 項目完全相符。 針對應用程式屬性，欄位名稱可為所需的任意值，只要該名稱是裝置範本中的唯一名稱即可。

例如，您可以選取 [Edit Template] \(編輯範本\) 並輸入新屬性，來新增裝置位置作為新屬性：

![[屬性] 索引標籤上的 [設定文字] 表單](./media/howto-set-up-template/propertiesform.png)

選取 [儲存] 之後，裝置位置就會顯示為圖格：

![[位置] 圖格](./media/howto-set-up-template/propertiestile.png)

建立圖格之後，您可以變更屬性值。 首先，選取螢幕右上方的 [Done] \(完成\)。

### <a name="create-a-location-property-through-azure-maps"></a>透過 Azure 地圖服務建立位置屬性
您可以在 Azure IoT Central 中將地理內容提供給位置資料，然後繪製街道地址的任何經緯度座標地圖。 或是只繪製經緯度座標地圖。 「Azure 地圖服務」會在 IoT Central 中啟用此功能。

您可以新增兩種類型的位置屬性：
- **將位置新增為應用程式屬性**：此屬性只儲存在應用程式中。 裝置無法辨識應用程式屬性。
- **將位置新增為裝置屬性**：裝置將會回報此屬性。

#### <a name="add-location-as-an-application-property"></a>將位置新增為應用程式屬性 
您可以在 IoT Central 應用程式中使用「Azure 地圖服務」來建立位置屬性作為應用程式屬性。 例如，您可以新增裝置安裝地址。 

1. 在 [Properties] \(屬性\) 索引標籤上，選取 [Edit Template] \(編輯範本\)。

   ![已開啟設計模式的 [屬性] 索引標籤](./media/howto-set-up-template/locationcloudproperty1.png)

2. 在資源庫中，選取 [位置]。
3. 設定位置的 [顯示名稱]、[欄位名稱] 及 (選擇性) [初始值]。 

   ![含位置詳細資料的 [設定位置] 表單](./media/howto-set-up-template/locationcloudproperty2.png)

   新增位置有兩種支援的格式：
   - **以地址新增位置**
   - **以座標新增位置** 

4. 選取 [Save] \(儲存\) 和 [Done] \(完成\)。 現在，操作員可更新位置欄位表單中的位置值。 

#### <a name="add-location-as-a-device-property"></a>將位置新增為裝置屬性 

您可以將位置屬性建立成裝置會回報的裝置屬性。 例如，如果您想要追蹤裝置位置：

1. 在 [Properties] \(屬性\) 索引標籤上，選取 [Edit Template] \(編輯範本\)。

   ![已開啟設計模式的 [屬性] 索引標籤](./media/howto-set-up-template/locationdeviceproperty1.png)

2. 從資源庫中選取 [裝置屬性]。
3. 設定顯示名稱和欄位名稱，然後選取 [位置] 作為資料類型。 

   > [!NOTE]
   > 欄位名稱必須與裝置所回報的屬性名稱完全相符。 

   ![含位置詳細資料的 [設定裝置屬性] 表單](./media/howto-set-up-template/locationdeviceproperty2.png)

既然您已設定位置屬性，現在即可[在裝置儀表板中新增地圖來以視覺化方式呈現位置](#add-an-azure-maps-location-in-the-dashboard)。

## <a name="commands"></a>命令

命令可用來從遠端管理裝置。 它們可讓應用程式操作員在裝置上立即執行命令。 您可以在裝置範本中新增多個命令，這些命令會在 [命令] 索引標籤上顯示為圖格，以供操作員使用。 身為裝置建置者，您可以有彈性地根據需求定義命令。

命令與設定有何不同？ 

* **設定**：設定是您想要在裝置套用的設定，並且會希望裝置保存該設定，直到您變更該設定為止。 例如，您想要設定冰箱的溫度，並且希望即使冰箱重新啟動也保有該設定。 

* **命令**：您會使用命令從 IoT Central 以遠端方式在裝置上立即執行命令。 如果裝置未連線，命令就會逾時並失敗。 例如，您想要重新啟動某個裝置。  

當您執行命令時，視裝置是否收到該命令而定，可為三種狀態其中之一。

例如，您可以選取 [Editing Template] \(編輯範本\)，然後按一下 [+ 新增命令]，接著輸入新命令，來新增新的 **Echo** 命令：

![含回應詳細資料的 [設定命令] 表單](./media/howto-set-up-template/commandsecho.png)

選取 [Save] \(儲存\) 和 [Done] \(完成\) 之後，**Echo** 命令就會顯示為圖格，並已可供用來回應裝置。

在建立圖格之後，您便可以試用新的命令。

## <a name="rules"></a>規則

規則可讓操作員近乎即時地監視裝置。 規則會在觸發規則時自動叫用動作，例如傳送電子郵件。 目前有一種類型的規則可供使用：

- **遙測規則**：當所選裝置的遙測超出指定閾值時，便會觸發此規則。 [深入了解遙測規則](howto-create-telemetry-rules.md)。

## <a name="dashboard"></a>儀表板

儀表板可供操作員查看裝置的相關資訊。 身為建置者，您可以在此頁面新增圖格，以協助操作員了解裝置的運作情形。 您可以在裝置範本中新增多個儀表板圖格。 您可以新增六種類型的儀表板圖格：影像、折線圖、橫條圖、KPI、設定和屬性，以及標籤。

例如，您可以選取 [Edit Template] \(編輯範本\)，然後從資源庫中選取 [Settings and Properties] \(設定和屬性\) 圖格來新增該圖格，以顯示目前選取的設定和屬性值：

![含設定和屬性詳細資料的 [設定裝置詳細資料] 表單](./media/howto-set-up-template/dashboardsettingsandpropertiesform.png)

現在，當操作員檢視儀表板時，就會看到這個可顯示裝置屬性和設定的圖格：

![含針對圖格顯示之設定和屬性的 [儀表板] 索引標籤](./media/howto-set-up-template/dashboardtile.png)

### <a name="add-an-azure-maps-location-in-the-dashboard"></a>在儀表板中新增 Auzre 地圖服務位置

如果您稍早已在[透過 Azure 地圖服務建立位置屬性](#create-a-location-property-through-azure-maps)中設定位置屬性，就可以在裝置儀表板中使用地圖，以視覺化方式呈現位置。

1. 在 [Dashboard] \(儀表板\) 索引標籤上，選取 [Edit Template] \(編輯範本\)。

   ![已開啟設計模式的 [儀表板] 索引標籤](./media/howto-set-up-template/locationcloudproperty4map.png)

2. 在 [裝置儀表板] 上，從資源庫中選取 [地圖]。 
3. 為它提供一個標題，然後選擇您先前在裝置屬性中設定的位置屬性。

   ![含標題和屬性詳細資料的 [設定地圖] 表單](./media/howto-set-up-template/locationcloudproperty5map.png)

4. 選取 [ **儲存**]。 地圖圖格現在會顯示您選取的位置。 

   ![含所選位置的 [地圖] 圖格](./media/howto-set-up-template/locationcloudproperty6map.png) 

您可以將地圖調整為所需的大小。 現在，當操作員檢視儀表板時，他們可以看到您已設定的所有儀表板圖格，包括位置地圖。

## <a name="next-steps"></a>後續步驟

了解如何在 Azure IoT Central 應用程式中設定裝置範本之後，現在就可以：

> [!div class="nextstepaction"]
> [建立新的裝置範本版本](howto-version-devicetemplate.md)
