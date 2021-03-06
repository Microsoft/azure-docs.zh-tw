---
title: 監視您的 Azure Stack Edge Pro 裝置 |Microsoft Docs
description: 說明如何使用 Azure 入口網站和本機 web UI 來監視您的 Azure Stack Edge Pro。
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 04/15/2019
ms.author: alkohli
ms.openlocfilehash: cd75eb0f7de602979f2233a873c01ef742471e37
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/09/2020
ms.locfileid: "90904401"
---
# <a name="monitor-your-azure-stack-edge-pro"></a>監視您的 Azure Stack Edge Pro

本文說明如何監視您的 Azure Stack Edge Pro。 若要監視您的裝置，您可以使用 Azure 入口網站或本機 Web UI。 使用 Azure 入口網站來檢視裝置事件、設定及管理警示和檢視計量。 在您的實體裝置上使用本機 Web UI，來檢視各種裝置元件的硬體狀態。

在本文中，您將學會如何：

> [!div class="checklist"]
>
> * 檢視裝置事件和對應的警示
> * 檢視裝置元件的硬體狀態
> * 檢視您裝置的容量和交易計量
> * 設定和管理警示

## <a name="view-device-events"></a>檢視裝置事件

[!INCLUDE [Supported OS for clients connected to device](../../includes/data-box-edge-gateway-view-device-events.md)]

## <a name="view-hardware-status"></a>檢視硬體狀態

在本機 Web UI 中採取下列步驟來檢視您裝置元件的硬體狀態。

1. 連線至您裝置的本機 Web UI 設定。
2. 移至 [維護] > [硬體狀態]。 您可以檢視各種裝置元件的健康情況。

    ![檢視硬體狀態](media/azure-stack-edge-monitor/view-hardware-status.png)

## <a name="view-metrics"></a>檢視計量

[!INCLUDE [Supported OS for clients connected to device](../../includes/data-box-edge-gateway-view-metrics.md)]

### <a name="metrics-on-your-device"></a>您裝置上的計量

本節說明您裝置上的監視計量。 計量可以：

* 容量計量。 容量計量與裝置容量相關。

* 交易計量。 交易計量與 Azure 儲存體中的讀取和寫入作業相關。

* Edge 計算度量。 Edge 計算計量與您裝置上 Edge 計算的使用量相關。

下表顯示計量的完整清單：

|容量度量                     |描述  |
|-------------------------------------|-------------|
|**可用容量**               | 指的是可以寫入裝置的資料大小。 換句話說，這是在裝置上可取得的容量。 <br></br>您可以藉由刪除在裝置和雲端都有複本的本機複本，以釋放裝置容量。        |
|**容量總計**                   | 指的是裝置上可寫入資料的位元組總計。 這也稱為本機快取大小總計。 <br></br> 您現在可藉由新增資料磁碟來增加現有虛擬裝置的容量。 透過 VM 的 Hypervisor 管理新增資料磁碟，然後將您的 VM 重新開機。 閘道裝置的本機存放集區會擴充，以適應新增的資料磁碟。 <br></br>如需詳細資訊，請移至[為 HYPER-V 虛擬機器新增硬碟](https://www.youtube.com/watch?v=EWdqUw9tTe4)。 |

|交易度量              | 描述         |
|-------------------------------------|---------|
|**上傳的雲端位元組數 (裝置)**    | 在您裝置上所有共用上傳之所有位元組的總和        |
|**上傳的雲端位元組數 (共用)**     | 每個共用上傳的位元組數。 這可以是： <br></br> 平均 (每個共用上傳的位元組總和 / 共用數目)、  <br></br>最大值，即從共用上傳之位元組的最大數目 <br></br>最小值，即從共用上傳之位元組的最小數目      |
|**雲端下載輸送量 (共用)**| 每個共用下載的位元組數。 這可以是： <br></br> 平均 (讀取或下載至共用的位元組總和 / 共用數目) <br></br> 最大值，即從共用下載之位元組的最大數目<br></br> 及最小值，即從共用下載之位元組的最小數目  |
|**雲端讀取輸送量**            | 在您裝置上所有共用從雲端讀取之所有位元組的總和     |
|**雲端上傳輸送量**          | 在您裝置上所有共用寫入至雲端之所有位元組的總和     |
|**雲端上傳輸送量 (共用)**  | 從共用寫入至雲端的所有位元組總和 / 共用數目是每個共用的平均、最大值和最小值      |
|**讀取輸送量 (網路)**           | 包括從雲端讀取之所有位元組的系統網路輸送量。 此檢視可包含未限制共用的資料。 <br></br>分割會顯示裝置上所有網路介面卡的流量。 這包括未連線或未啟用的介面卡。      |
|**寫入輸送量 (網路)**       | 包括寫入至雲端之所有位元組的系統網路輸送量。 此檢視可包含未限制共用的資料。 <br></br>分割會顯示裝置上所有網路介面卡的流量。 這包括未連線或未啟用的介面卡。          |

| Edge 計算計量              | 描述         |
|-------------------------------------|---------|
|**Edge 計算 - 記憶體使用量**      |           |
|**Edge 計算 - 百分比 CPU**    |         |

## <a name="manage-alerts"></a>管理警示

[!INCLUDE [Supported OS for clients connected to device](../../includes/data-box-edge-gateway-manage-alerts.md)]

## <a name="next-steps"></a>後續步驟

了解如何[管理頻寬](azure-stack-edge-manage-bandwidth-schedules.md)。
