---
title: 將 Windows IoT 核心版裝置連線到 Azure IoT 中心應用程式 | Microsoft Docs
description: 如何以裝置開發人員身分，將 MXChip IoT DevKit 裝置連線到 Azure IoT 中心應用程式。
author: miriambrus
ms.author: miriamb
ms.date: 04/09/2018
ms.topic: conceptual
ms.service: iot-central
services: iot-central
manager: peterpr
ms.openlocfilehash: 73a23ace23d2373e238c6887c4a41c6037d233de
ms.sourcegitcommit: 63613e4c7edf1b1875a2974a29ab2a8ce5d90e3b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/29/2018
ms.locfileid: "43188999"
---
# <a name="connect-a-windows-iot-core-device-to-your-azure-iot-central-application"></a>將 Windows IoT 核心版裝置連線到 Azure IoT 中心應用程式

本文說明如何以裝置開發人員身分，將 Windows IoT 核心版裝置連線到 Microsoft Azure IoT 中心應用程式。

## <a name="before-you-begin"></a>開始之前

若要完成這篇文章中的步驟，您需要下列項目︰

1. Azure IoT Central 應用程式是從**範例 Devkits** 應用程式範本建立而來。 如需詳細資訊，請參閱[建立 Azure IoT Central 應用程式](howto-create-application.md)。
2. 執行 Windows 10 IoT 核心版作業系統的裝置。 在此逐步解說中，我們將使用 Raspberry Pi。


## <a name="sample-devkits-application"></a>**範例 Devkits** 應用程式

從**範例 Devkits** 應用程式範本建立的應用程式包含具有下列特性的 **Windows IoT 核心版**裝置範本： 

- 包含裝置**溼度**、**溫度**和**壓力**等測量值的遙測。 
- 顯示**風扇速度**的設定。
- 包含裝置屬性**骰子數字**和**位置**雲端屬性的屬性。


如需裝置範本的詳細設定資訊，請參閱 [Windows IoT 核心版裝置範本詳細資料](howto-connect-windowsiotcore.md#windows-iot-core-device-template-details)

## <a name="add-a-real-device"></a>新增真實裝置

在 Azure IoT 中心應用程式中，從 **Windows IoT 核心版**裝置範本新增真實裝置，並記下裝置連接字串。 如需詳細資訊，請參閱[將真實裝置新增至 Azure IoT 中心應用程式](tutorial-add-device.md)。

### <a name="prepare-the-windows-iot-core-device"></a>準備 Windows IoT 核心版裝置

若要設定 Windows IoT 核心版裝置，請遵位於 [設定 Windows IoT 核心版裝置] (https://github.com/Azure/iot-central-firmware/tree/master/WindowsIoT#setup-a-physical-device) 的循逐步指南。

### <a name="add-a-real-device"></a>新增真實裝置

在 Azure IoT 中心應用程式中，從 **Windows IoT 核心版**裝置範本新增真實裝置，並記下裝置連接字串。 如需詳細資訊，請參閱[將真實裝置新增至 Azure IoT 中心應用程式](tutorial-add-device.md)。

## <a name="prepare-the-windows-10-iot-core-device"></a>準備 Windows 10 IoT 核心版裝置

### <a name="what-youll-need"></a>您所需的項目

若要設定實體 Windows 10 IoT 核心版裝置，您必須先具有執行 Windows 10 IoT 核心版的裝置。 [在此](https://developer.microsoft.com/en-us/windows/iot/getstarted/prototype/setupdevice)了解如何設定 Windows 10 IoT 核心版裝置。

您也需要可與 Azure IoT 中心通訊的用戶端應用程式。 您可以使用 Azure SDK 建置自己的自訂應用程式，並使用 Visual Studio 將它部署到您的裝置，ㄝ可以下載[預先建立的範例](https://developer.microsoft.com/en-us/windows/iot/samples)且只需在裝置上部署和執行該範例。 

### <a name="deploying-the-sample-client-application"></a>部署範例用戶端應用程式

若要將上一個步驟中的用戶端應用程式部署至 Windows 10 IoT 裝置，以便準備它：

**確定連接字串儲存在裝置上以供用戶端應用程式使用**
* 在桌面上，在名為 connection.string.iothub 的文字檔案中儲存連接字串。
* 將文字檔案複製到裝置的文件資料夾：`[device-IP-address]\C$\Data\Users\DefaultAccount\Documents\connection.string.iothub`

完成之後，您必須在任何瀏覽器中輸入 http://[device-IP-address]:8080，以開啟 [Windows 裝置入口網站](https://docs.microsoft.com/en-us/windows/iot-core/manage-your-device/deviceportal)。

從這裡 (如下圖所示)，您會想要：
1. 展開左側的 [應用程式] 節點。
2. 按一下 [快速執行範例]。
3. 按一下 [Azure IoT 中樞用戶端]。
4. 按一下 [部署並執行]。

![Windows 裝置入口網站上 Azure IoT 中樞用戶端的 Gif](./media/howto-connect-windowsiotcore/iothubapp.gif)

成功時，應用程式會在裝置上啟動，如下所示：

![Azure IoT 中樞用戶端應用程式的螢幕擷取畫面](./media/howto-connect-windowsiotcore/IoTHubForegroundClientScreenshot.png)

在 Azure IoT 中心，您可以看到在 Raspberry Pi 上執行的程式碼如何與應用程式互動：

* 在真實裝置的 [量值] 頁面上，您可以看到遙測。
* 在 [屬性] 頁面上，您可以看到所報告 [Die 數字] 屬性的值。
* 在 [設定] 頁面上，您可以變更 Raspberry Pi 的各種設定，例如電壓和風扇。

## <a name="download-the-source-code"></a>下載原始程式碼

如果您想要探索並修改用戶端應用程式的原始程式碼，您可以從 GitHub ([這裡](https://github.com/Microsoft/Windows-iotcore-samples/tree/develop/Samples/Azure/IoTHubClients)) 下載它。 如果您打算修改程式碼，您應該遵循您的桌上型電腦作業系統的讀我檔案 ([這裡](https://github.com/Microsoft/Windows-iotcore-samples)) 中的指示。

> [!NOTE]
> 如果您的開發環境中未安裝 **git**，您可以從 [https://git-scm.com/download](https://git-scm.com/download) 下載它。

## <a name="windows-iot-core-device-template-details"></a>Windows IoT 核心版裝置範本詳細資料

從**範例 Devkits** 應用程式範本建立的應用程式包含具有下列特性的 **Windows IoT 核心版**裝置範本：

### <a name="telemetry-measurements"></a>遙測量測

| 欄位名稱     | Units  | 最小值 | 最大值 | 小數位數 |
| -------------- | ------ | ------- | ------- | -------------- |
| 溼度       | %      | 0       | 100     | 0              |
| temp           | °C     | -40     | 120     | 0              |
| pressure       | hPa    | 260     | 1260    | 0              |

### <a name="settings"></a>設定

數值設定

| 顯示名稱 | 欄位名稱 | Units | 小數位數 | 最小值 | 最大值 | Initial |
| ------------ | ---------- | ----- | -------------- | ------- | ------- | ------- |
| 風扇速度    | fanSpeed   | RPM   | 0              | 0       | 1000    | 0       |


### <a name="properties"></a>properties

| 類型            | 顯示名稱 | 欄位名稱 | 資料類型 |
| --------------- | ------------ | ---------- | --------- |
| 裝置屬性 | 模具編號   | dieNumber  | number    |
| 文字            | 位置     | location   | N/A       |
