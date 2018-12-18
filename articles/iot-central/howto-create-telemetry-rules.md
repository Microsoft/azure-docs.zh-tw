---
title: 在 Azure IoT Central 應用程式中建立和管理遙測規則 | Microsoft Docs
description: Azure IoT Central 遙測規則可讓您近乎即時地監視裝置，以及在觸發規則時自動叫用動作，例如傳送電子郵件。
author: ankitgupta
ms.author: ankitgup
ms.date: 08/14/2018
ms.topic: conceptual
ms.service: iot-central
services: iot-central
manager: peterpr
ms.openlocfilehash: 1b82ac3bf67370a2c39e85bf5691da38539edb74
ms.sourcegitcommit: 1b561b77aa080416b094b6f41fce5b6a4721e7d5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/17/2018
ms.locfileid: "45729302"
---
# <a name="create-a-telemetry-rule-and-set-up-notifications-in-your-azure-iot-central-application"></a>在 Azure IoT Central 應用程式中建立遙測規則並設定通知

*本文適用於操作員、建置員及系統管理員。*

您可以使用 Azure IoT Central 來遠端監視連線的裝置。 Azure IoT Central 規則可讓您近乎即時地監視裝置，以及自動叫用動作，例如傳送電子郵件或觸發 Microsoft Flow。 只要按幾下，就可以定義條件來監視裝置資料以及設定對應的動作。 本文將說明如何建立規則來監視裝置所傳送的遙測。

裝置可以使用遙測量測，從裝置傳送數值資料。 當所選裝置的遙測超出指定閾值時，便會觸發遙測規則。

## <a name="create-a-telemetry-rule"></a>建立遙測規則

若要建立遙測規則，裝置範本必須至少定義一個遙測量測。 此範例使用會傳送溫度和濕度遙測的冷藏自動販賣機裝置。 此規則會監視裝置所報告的遙測，並在裝置溫度超過 80 度時傳送電子郵件。

1. 使用 Device Explorer，瀏覽至您要對其新增規則的裝置範本。

1. 在選取的範本下，按一下現有裝置。 

    >[!TIP] 
    >如果範本沒有任何裝置，則先新增裝置。

1. 如果您尚未建立任何規則，您將會看到下列畫面：

    ![還沒有規則](media\howto-create-telemetry-rules\Rules_Landing_Page.png)

1. 在 [規則] 索引標籤上，依序按一下 [編輯範本] 和 [+ 新增規則]，以查看您可以建立的規則類型。

1. 按一下 [遙測]，以建立規則來監視裝置遙測。

    ![規則類型](media\howto-create-telemetry-rules\Rule_Types.png)

1. 輸入名稱來協助您識別此裝置範本中的規則。

1. 若要立即為針對此範本建立的所有裝置啟用規則，請切換 [為此範本的所有裝置啟用規則]。

   ![規則詳細資訊](media\howto-create-telemetry-rules\Rule_Detail.png)
    
    此規則會自動套用到裝置範本下的所有裝置。
    

### <a name="configure-the-rule-conditions"></a>設定規則條件

條件會定義規則所監視的準則。

1. 按一下 [條件] 旁的 [+] 來新增條件。

1. 從 [量測] 下拉式清單中選取想要監視的遙測。

   ![條件](media\howto-create-telemetry-rules\Aggregate_Condition_Filled_Out.png)

1. 接下來，選擇 [彙總]、[操作員]，並提供**閾值**。
    - 彙總為選擇性的。 如果沒有彙總，規則就會在每個遙測資料點符合條件時觸發。 例如，如果將規則設定為在溫度超過 80 時觸發，則此規則將幾乎會在裝置報告溫度 > 80 時立即觸發。
    - 如果選擇了像是 Average、Min、Max、Count 的彙總函式，則使用者必須提供需要評估條件的**彙總時間範圍**。 例如，如果您將期間設定為「5 分鐘」，而且您的規則會尋找超過 80 的平均溫度，則此規則會在平均溫度超過 80 至少 5 分鐘時觸發。 規則評估頻率與**彙總時間範圍**相同，這表示，在此範例中，此規則每隔 5 分鐘就會評估一次。

    >[!NOTE]
    >您可以在**條件**下新增多個遙測量測。 若指定多個條件，就必須符合所有條件才會觸發規則。 每個條件都會透過 'AND' 子句隱含地加入。 使用彙總時，必須彙總每個量測。
    
    

### <a name="configure-actions"></a>設定動作

本節示範如何設定要在引發規則時採取的動作。 當規則中指定的所有條件都評估為 True 時，即會叫用動作。

1. 選擇 [動作] 旁的 [+]。 在此，您會看到可用動作的清單。  

    ![新增動作](media\howto-create-telemetry-rules\Add_Action.png)

1. 選擇 [傳送電子郵件] 動作，在 [收件者] 欄位中輸入有效的電子郵件地址，然後提供當此規則觸發時要在電子郵件本文中出現的附註。

    > [!NOTE]
    > 只有已新增至應用程式且至少已登入一次的使用者才會收到電子郵件。 深入了解 Azure IoT Central 中的[使用者管理](howto-administer.md)。

   ![設定動作](media\howto-create-telemetry-rules\Configure_Action.png)

1. 若要儲存規則，請選擇 [儲存]。 幾分鐘內，規則就會生效，並開始監視傳送至應用程式的遙測。 當規則中指定的條件相符時，規則就會觸發所設定的電子郵件動作。

1. 選擇 [完成] 以結束 [編輯範本] 模式。

您可以將其他動作新增至規則，例如 Microsoft Flow 和 Webhook。 您可以針對每個規則最多新增 5 個動作。

- 觸發規則時，[Microsoft Flow 動作](howto-add-microsoft-flow.md)會啟動 Microsoft Flow 中的工作流程 
- 而 [Webhook 動作](howto-create-webhooks.md)會在觸發規則時，通知其他服務

## <a name="parameterize-the-rule"></a>將規則參數化

規則可以從 [裝置屬性] 衍生特定值來作為參數。 在不同裝置會有不同遙測閾值的情況下，使用參數會很有幫助。 當您建立規則時，請選擇會指定閾值的裝置屬性 (例如，**理想閾值上限**)，而不要提供絕對值 (例如 80 度)。 當規則執行時，它會比對裝置遙測與裝置屬性中所設定的值。

使用參數可有效減少要針對每個裝置範本管理的規則數目。

您也可以使用 [裝置屬性] 將動作設定為參數。 如果將電子郵件地址儲存為屬性，您就可以在定義 [收件者] 地址時加以使用。

## <a name="delete-a-rule"></a>刪除規則

如果不再需要某個規則，請開啟規則並選擇 [刪除] 來加以刪除。 規則在刪除後就會從裝置範本和所有相關聯的裝置中移除。

## <a name="enable-or-disable-a-rule-for-a-device-template"></a>啟用或停用裝置範本的規則

瀏覽至裝置，然後選擇您要啟用或停用的規則。 切換規則中的 [為此範本的所有裝置啟用規則] 按鈕，可對所有與裝置範本相關聯的裝置啟用或停用規則。

## <a name="enable-or-disable-a-rule-for-a-device"></a>啟用或停用裝置的規則

瀏覽至裝置，然後選擇您要啟用或停用的規則。 切換 [啟用此裝置的規則] 按鈕，可對該裝置啟用或停用規則。

## <a name="next-steps"></a>後續步驟

您現在已了解如何在 Azure IoT Central 應用程式中建立規則，以下是一些後續步驟：

- [在規則中新增 Microsoft Flow 動作](howto-add-microsoft-flow.md)
- [在規則中新增 Webhook 動作](howto-create-webhooks.md)
- [如何管理您的裝置](howto-manage-devices.md)
