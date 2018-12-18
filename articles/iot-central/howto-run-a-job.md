---
title: 在 Azure IoT Central 應用程式中建立及執行工作 | Microsoft Docs
description: Azure IoT Central 工作可讓您執行大量裝置管理功能，例如更新裝置屬性、設定或執行命令。
ms.service: iot-central
services: iot-central
author: sarahhubbard
ms.author: sahubbar
ms.date: 09/15/2018
ms.topic: conceptual
manager: peterpr
ms.openlocfilehash: 35db7bf87c7b72fc31d820c9058b1df8415bd553
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47031300"
---
# <a name="create-and-run-a-job-in-your-azure-iot-central-application"></a>在 Azure IoT Central 應用程式中建立及執行工作

您可以使用 Microsoft Azure IoT Central 利用工作來大規模管理您的已連線裝置。 工作功能可讓您對裝置屬性、設定及命令執行大量更新。 此文章將逐步說明如何開始在您自己的應用程式中使用工作。

## <a name="create-and-run-a-job"></a>建立及執行工作

本節說明如何建立及執行工作。 每個步驟都會完成一個範例，此範例會示範如何針對需要提升風扇速度的冷藏自動販賣機裝置執行工作。

1. 從瀏覽窗格中瀏覽至 [工作]。

1. 按一下 [+ 新增] 以開始建立新工作。

    ![建立新工作](./media/howto-run-a-job/createnewjob.png)

1. 輸入可協助您識別所要建立之工作的名稱和描述。

1. 選取要套用工作的裝置集。 選取裝置集之後，您會看到右側填入所選裝置集內的裝置。 如果您選取已損毀的裝置集，則不會顯示任何裝置，您將會看到一則說明裝置集已損毀的訊息。

1. 接著，選擇將定義的工作類型 (設定、屬性或命令)。 按一下所選工作類型旁的 [+]，然後新增您想要的作業。

    ![設定工作](./media/howto-run-a-job/configurejob.png)

1. 在右側，挑選您想要執行工作的裝置。 按一下頂端的核取方塊，即可選取整個裝置集內的所有裝置。 按一下 [名稱] 旁的核取方塊，即可選取目前頁面上的所有裝置。

1. 選取您所需的裝置之後，選擇 [執行]。 該工作現在便會出現在您的主要 [工作] 頁面上。 在此檢視上，您可以查看目前執行中的工作，以及任何先前所執行工作的歷程記錄。 您的執行中工作一律會顯示在清單的頂端。

    ![執行工作](./media/howto-run-a-job/runjob.png)

    ![檢視工作](./media/howto-run-a-job/viewjob.png)

    > [!NOTE]
    > 您將能夠檢視先前所執行工作的歷程記錄，最長可達 30 天。

1. 若要取得您工作的概觀，請從清單中按一下您想要檢視的工作名稱。 此概觀會包含工作詳細資料、裝置及裝置狀態。

    ![檢視裝置狀態](./media/howto-run-a-job/viewdevicestatus.png)

### <a name="stop-a-running-job"></a>停止執行中的工作

如果您想要停止目前執行中的工作，請按一下您想要停止的執行中工作名稱。 選擇面板上的 [停止] 按鈕。 您會看到工作狀態已變更來反映該工作已停止。

> [!NOTE]
> 工作在停止之後即無法重新啟動。 您必須建立另一個具有所需作業和裝置的工作。

## <a name="view-the-job-status"></a>檢視工作狀態

建立工作之後，[狀態] 資料行將會以工作的最新狀態訊息進行更新。 狀態訊息的意義如下：

| 狀態訊息       | 狀態意義                                          |
| -------------------- | ------------------------------------------------------- |
| Completed            | 此工作已在所有裝置上執行。              |
| Failed               | 此工作失敗而未在裝置上完整執行。  |
| Pending              | 此工作尚未在裝置上開始執行。        |
| 執行中              | 此工作目前正在裝置上執行。             |
| 已停止              | 此工作已被使用者手動停止。           |

狀態訊息後面會接著工作內裝置的概觀。 這些裝置狀態的意義如下：

| 狀態訊息       | 狀態意義                                                     |
| -------------------- | ------------------------------------------------------------------ |
| Succeeded            | 工作執行成功的裝置數目。  |
| Failed               | 工作執行失敗的裝置數目。      |

### <a name="view-the-device-status"></a>檢視裝置狀態

若要檢視工作中每個裝置的狀態，請按一下工作名稱。 在這裡，您會看到工作的詳細資料，以及此特定工作所包含的所有裝置。 在每個裝置名稱的旁邊，您會看到下列其中一個狀態訊息：

| 狀態訊息       | 狀態意義                                                                |
| -------------------- | ----------------------------------------------------------------------------- |
| Completed            | 工作已在此裝置上執行。                                     |
| Failed               | 工作在此裝置上執行失敗。 隨附的錯誤訊息將會顯示更多資訊。  |
| Pending              | 工作尚未在此裝置上執行。                                  |

> [!NOTE]
> 如果裝置已被刪除，您將無法選取該裝置，系統會以裝置識別碼將它顯示為已刪除。

## <a name="next-steps"></a>後續步驟

您現在已了解如何在 Azure IoT Central 應用程式中建立工作，以下是一些後續步驟：

- [使用裝置集合](howto-use-device-sets.md)
- [管理您的裝置](howto-manage-devices.md)
- [建立裝置範本版本](howto-version-devicetemplate.md)
