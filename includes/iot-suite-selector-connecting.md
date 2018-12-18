---
title: 包含檔案
description: 包含檔案
services: iot-suite
author: dominicbetts
ms.service: iot-suite
ms.topic: include
ms.date: 04/24/2018
ms.author: dobett
ms.custom: include file
ms.openlocfilehash: 780a215b66fec845bc1df639fedda870881b4027
ms.sourcegitcommit: bf522c6af890984e8b7bd7d633208cb88f62a841
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/20/2018
ms.locfileid: "39189310"
---
> [!div class="op_single_selector"]
> * [Windows 上的 C](../articles/iot-accelerators/iot-accelerators-connecting-devices.md)
> * [Linux 上的 C](../articles/iot-accelerators/iot-accelerators-connecting-devices-linux.md)
> * [Node.js (一般)](../articles/iot-accelerators/iot-accelerators-connecting-devices-node.md)
> * [Raspberry Pi 上的 Node.js](../articles/iot-accelerators/iot-accelerators-connecting-pi-node.md)
> * [Raspberry Pi 上的 C](../articles/iot-accelerators/iot-accelerators-connecting-pi-c.md)

在此教學課程中，您會實作一個 **Chiller** 裝置，此裝置會將下列遙測資料傳送給遠端監視[解決方案加速器](../articles/iot-accelerators/about-iot-accelerators.md)：

* 溫度
* 壓力
* 溼度

為了簡單起見，程式碼會產生 **Chiller** 的範例遙測值。 您可以將實際感應器連線到您的裝置並傳送實際的遙測來擴充範例。

範例裝置也會：

* 將中繼資料傳送至解決方案來描述其功能。
* 回應從解決方案中的**裝置**頁面觸發的動作。
* 回應從解決方案中的**裝置**頁面傳送的設定變更。

若要完成此教學課程，您需要一個有效的 Azure 帳戶。 如果您沒有帳戶，只需要幾分鐘的時間就可以建立免費試用帳戶。 如需詳細資料，請參閱 [Azure 免費試用](http://azure.microsoft.com/pricing/free-trial/)。

## <a name="before-you-start"></a>開始之前

在您為裝置撰寫任何程式碼之前，請先部署遠端監視解決方案加速器，並將新的實體裝置新增至該解決方案。

### <a name="deploy-your-remote-monitoring-solution-accelerator"></a>部署遠端監視解決方案加速器

您在本教學課程中建立的 **Chiller** 裝置會將資料傳送給[遠端監視](../articles/iot-accelerators/quickstart-remote-monitoring-deploy.md)解決方案加速器的執行個體。 如果您尚未在您的 Azure 帳戶中佈建遠端監視解決方案加速器，請參閱[部署遠端監視解決方案加速器](../articles/iot-accelerators/quickstart-remote-monitoring-deploy.md)

當遠端監視解決方案的部署程序完成之後，請按一下 [啟動]，以在瀏覽器中開啟解決方案儀表板。

![解決方案儀表板](media/iot-suite-selector-connecting/dashboard.png)

### <a name="add-your-device-to-the-remote-monitoring-solution"></a>將您的裝置新增到遠端監視解決方案

> [!NOTE]
> 如果您已經在解決方案中新增裝置，則可以略過此步驟。 不過，下一個步驟需要您裝置的連接字串。 您可以從 [Azure 入口網站](https://portal.azure.com)或使用 [az iot](https://docs.microsoft.com/cli/azure/iot?view=azure-cli-latest) CLI 工具來擷取裝置的連線字串。

對於連線到解決方案加速器的裝置，該裝置必須使用有效的認證向 IoT 中樞識別自己。 當您新增裝置解決方案時，會有機會儲存包含這些認證的裝置連接字串。 稍後在本教學課程中，您會將裝置連接字串包含在您的用戶端應用程式中。

若要在您的遠端監視解決方案中新增裝置，請在解決方案**裝置**頁面中完成下列步驟：

1. 選擇 [+ 新增裝置]，然後選擇 [實體]作為 [裝置類型]：

    ![新增實體裝置](media/iot-suite-selector-connecting/devicesprovision.png)

1. 輸入 **Physical-chiller** 作為裝置識別碼。 選擇 [對稱金鑰] 和 [自動產生金鑰] 選項：

    ![選擇裝置選項](media/iot-suite-selector-connecting/devicesoptions.png)

1. 選擇 [套用]。 然後記下**裝置識別碼**、**主要金鑰**和**連接字串主要金鑰**值：

    ![擷取認證](media/iot-suite-selector-connecting/credentials.png)

您現在已將實體裝置新增至遠端監視解決方案加速器中，並將其裝置連接字串記下。 在下列章節中，您可以實作使用裝置連接字串來連線到您解決方案的用戶端應用程式。

用戶端應用程式會實作內建 **Chiller** 裝置型號。 解決方案加速器裝置型號會指定下列相關裝置資訊：

* 裝置回報至解決方案的屬性。 例如，**Chiller** 裝置會報告其韌體和位置的相關資訊。
* 裝置傳送至解決方案的遙測類型。 例如，**Chiller** 裝置傳會送溫度、溼度和壓力值。
* 您可以從解決方案中排程以在裝置上執行的方法。 例如，**Chiller** 裝置必須實作 **Reboot**、**FirmwareUpdate**、**EmergencyValveRelease** 和 **IncreasePressure** 方法。