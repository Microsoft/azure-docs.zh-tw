---
title: 包含檔案
description: 包含檔案
services: iot-accelerators
author: dominicbetts
ms.service: iot-accelerators
ms.topic: include
ms.date: 08/20/2018
ms.author: dobett
ms.custom: include file
ms.openlocfilehash: 8c114ed137089e70899e601ebdc1d4d39f562601
ms.sourcegitcommit: a3a0f42a166e2e71fa2ffe081f38a8bd8b1aeb7b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/01/2018
ms.locfileid: "43383034"
---
## <a name="create-a-consumer-group"></a>建立取用者群組

您必須在 IoT 中樞內建立專屬的取用者群組，以便將遙測資料串流至時間序列深入解析。 時間序列深入解析中的事件來源應該獨佔使用 IoT 中樞取用者群組。

下列步驟會在 Azure Cloud Shell 中使用 Azure CLI 來建立取用者群組：

1. IoT 中樞是您在部署裝置模擬解決方案加速器時所建立的其中一個資源。 請執行下列命令來尋找 IoT 中樞的名稱 - 請記得使用解決方案加速器的名稱：

    ```azurecli-interactive
    az resource list --resource-group contoso-simulation -o table
    ```

    IoT 中樞是 **Microsoft.Devices/IotHubs** 類型的資源。

1. 在中樞內新增名為 **devicesimulationtsi** 的取用者群組。 在下列命令中，使用中樞和解決方案加速器的名稱：

    ```azurecli-interactive
    az iot hub consumer-group create --hub-name contoso-simulation7d894 --name devicesimulationtsi --resource-group contoso-simulation
    ```

    您現在可以關閉 Azure Cloud Shell。

## <a name="create-a-new-time-series-insights-environment"></a>建立新的時間序列深入解析環境

[Azure 時間序列深入解析](../articles/time-series-insights/time-series-insights-overview.md)是完全受管理的分析、儲存及視覺化服務，可讓您在雲端上管理 IoT 規模的時間序列資料。 若要建立新的時間序列深入解析環境：

1. 登入 [Azure 入口網站](http://portal.azure.com/)。

1. 選取 [建立資源] > [物聯網] > [時間序列深入解析]：

    ![新增時間序列深入解析](./media/iot-accelerators-create-tsi/new-time-series-insights.png)

1. 若要在和解決方案加速器相同的資源群組中建立時間序列深入解析環境，請使用下表中的值：

    | 設定 | 值 |
    | ------- | ----- |
    | 環境名稱 | 下列螢幕擷取畫面會使用名稱 **Contoso-TSI**。 當您完成此步驟時，請選擇您自己的唯一名稱。 |
    | 訂用帳戶 | 在下拉式清單中選取您的 Azure 訂用帳戶。 |
    | 資源群組 | **contoso-simulation**。 使用解決方案加速器的名稱。 |
    | 位置 | 此範例使用**美國東部**。 在和裝置模擬加速器相同的區域中建立環境。 |
    | SKU |**S1** |
    | Capacity | **1** |

    ![建立時間序列深入解析](./media/iot-accelerators-create-tsi/new-time-series-insights-create.png)

    > [!NOTE]
    > 將時間序列深入解析環境新增至和解決方案加速器相同的資源群組中，表示當您刪除解決方案加速器時，該環境也會刪除。

1. 按一下頁面底部的 [新增] 。 可能需要幾分鐘的時間來建立環境。

## <a name="create-event-source"></a>建立事件來源

建立新的事件來源以連線到 IoT 中樞。 使用您在先前步驟中所建立的取用者群組。 時間序列深入解析事件來源需要其他服務未使用的專屬取用者群組。

1. 在 Azure 入口網站中，瀏覽至新的時間序列環境。

1. 按一下左側的 [事件來源]：

    ![檢視事件來源](./media/iot-accelerators-create-tsi/time-series-insights-event-sources.png)

1. 按一下 [新增]：

    ![新增事件來源](./media/iot-accelerators-create-tsi/time-series-insights-event-sources-add.png)

1. 若要設定您的 IoT 中樞作為新的事件來源，請使用下表中的值：

    | 設定 | 值 |
    | ------- | ----- |
    | 事件來源名稱 | 下列螢幕擷取畫面會使用名稱 **contoso-iot-hub**。 當您完成此步驟時，請使用您自己的唯一名稱。 |
    | 來源 | **IoT 中心** |
    | 匯入選項 | **從可用的訂用帳戶使用 IoT 中樞** |
    | 訂用帳戶識別碼 | 在下拉式清單中選取您的 Azure 訂用帳戶。 |
    | IoT 中樞名稱 | **contoso-simulation7d894**。 使用來自裝置模擬解決方案加速器的 IoT 中樞名稱。 |
    | IoT 中樞原則名稱 | **iothubowner** |
    | IoT 中樞原則金鑰 | 系統會自動填入此欄位。 |
    | IoT 中樞取用者群組 | **devicesimulationtsi** |
    | 事件序列化格式 | **JSON** |
    | 時間戳記屬性名稱 | 保留空白 |

    ![建立事件來源](./media/iot-accelerators-create-tsi/time-series-insights-event-source-create.png)

1. 按一下頁面底部的 [新增] 。

> [!NOTE]
> 您可以[對其他使用者授與存取權](../articles/time-series-insights/time-series-insights-data-access.md#grant-data-access)，使其能夠存取時間序列深入解析總管。