---
title: 安裝 Azure Data Box Edge 實體裝置的教學課程 | Microsoft Docs
description: 安裝 Azure Data Box Edge 的第二個教學課程說明如何打開包裝，然後接上實體裝置的機架和纜線。
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: tutorial
ms.date: 10/08/2018
ms.author: alkohli
Customer intent: As an IT admin, I need to understand how to install Data Box Edge in datacenter so I can use it to transfer data to Azure.
ms.openlocfilehash: 21ac3de793f5ce559c3a03de2a09f11ccb86b12a
ms.sourcegitcommit: c282021dbc3815aac9f46b6b89c7131659461e49
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/12/2018
ms.locfileid: "49167353"
---
# <a name="tutorial-install-azure-data-box-edge-preview"></a>教學課程：安裝 Azure Data Box Edge (預覽)

本教學課程說明如何安裝 Data Box Edge 實體裝置。 安裝程序包括打開包裝、以機架掛接裝置，並接上纜線。 

安裝可能需要約 2 小時才能完成。

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 打開裝置包裝
> * 以機架掛接裝置
> * 接上裝置纜線

> [!IMPORTANT]
> Data Box Edge 目前處於預覽狀態。 部署訂購並部署此解決方案之前，請檢閱 [Azure 預覽版使用條款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。

## <a name="prerequisites"></a>必要條件

安裝實體裝置的必要條件如下。

### <a name="for-the-data-box-edge-resource"></a>針對 Data Box Edge 資源

在您開始前，請確定：

* 您已完成[準備入口網站以使用 Data Box Edge](data-box-edge-deploy-prep.md) 中的所有步驟。
    * 您已建立 Data Box Edge 資源以部署裝置。
    * 您已產生啟用金鑰，以啟用具有 Data Box Edge 資源的裝置。

 
### <a name="for-the-data-box-edge-physical-device"></a>針對 Data Box Edge 實體裝置

在部署裝置之前：

- 請確定裝置安全地放在平坦、穩定的工作介面 (或類似)。
- 確認您要設定的網站具有：
    - 獨立來源的標準 AC 電源，或
    - 具有不斷電供應系統 (UPS) 的機架配電裝置 (PDU)。
- 請確定您要掛接裝置的機架上有 1U 插槽可供使用。

### <a name="for-the-network-in-the-datacenter"></a>針對資料中心內的網路

開始之前：

- 檢閱部署 Data Box Edge 的網路需求，並且根據每個需求設定資料中心網路。 如需詳細資訊，請參閱 [Data Box Edge 網路需求](data-box-gateway-system-requirements.md#networking-requirements)。
- 確定網際網路頻寬至少有 20 Mbps，以便讓裝置能夠發揮最大功能。


## <a name="unpack-the-device"></a>打開裝置包裝

這個裝置是以單一箱裝出貨。 完成下列步驟以打開裝置的包裝。 

1. 將箱子放置在平坦的表面上。
2. 檢查箱子及包裝發泡材料有無損毀、割痕、浸水，或任何其他明顯損傷。 如果箱子或包裝嚴重損毀，不要打開箱子。 請連絡 Microsoft 支援服務，以協助您評估裝置是否正常運作。
3. 打開箱子。 打開箱子包裝之後，請確定您有：
    - 一個單一機箱 Edge 裝置
    - 兩條電源線
    - 一個無工具滑軌掛接套件 (內含兩個側滑軌和掛接硬體)
4. 如果您未收到任何上述項目，請連絡 Data Box Edge 支援。 下一步是利用機架掛接裝置。


## <a name="rack-the-device"></a>安裝裝置機架

裝置必須安裝在標準 19 吋機架中。 請使用下列程序，使用具有前後端柱子的標準 19 吋機架掛接您的裝置。

> [!IMPORTANT]
> Data Box Edge 裝置需要機架掛接才能正常運作。


1. 拉開前閂鎖，將滑軌組件的內部滑軌解鎖。 鬆開緊固鎖，並將中置滑軌向內推，以收回滑軌。 內部和外部滑軌此時應會分開。

    ![安裝機架掛接滑軌](./media/data-box-edge-deploy-install/rack-mount-rail-1.png)

2. 現在，請安裝機櫃垂直面上的外部滑軌。 為了協助辨識方向，滑軌標示為 [前] 的那一端應接至機箱後端正面。 
    
    1. 找出滑軌組件前後方的滑軌插梢。 延伸滑軌，使其置於機架柱之間。 先將滑軌外部連接至機架背面。 調整後端掛接托架，將其置於後端機架掛接孔內。   
    2. 按住背面托架上的制動機，露出金屬鉤。 將其對準並插入掛接孔中，然後釋放制動機。
    3. 將正面托架對準掛接孔。
    4. 正面托架現在應固定在機架上。 如有需要，您可以選擇性地使用 M5 X 10L 螺絲來固定滑軌與柱子。 

    ![安裝機架掛接滑軌](./media/data-box-edge-deploy-install/rack-mount-rail-2.png)

3. 若要連接底座的內部滑軌，請確實將內部滑軌上的鑰匙孔缺口對準底座側邊的定位銷。 請確實將底座定位銷的接頭穿過內部滑軌中的鑰匙孔開口。 將滑軌將底座前端拉動，直到滑軌喀答一聲鎖至定位。 對其他內部滑軌重複相同步驟。 使用內部滑軌將底座推入，以完成機架安裝。

    ![安裝機架掛接滑軌](./media/data-box-edge-deploy-install/rack-mount-rail-3.png)

## <a name="cable-the-device"></a>接上裝置纜線

下列程序說明如何為 Edge 裝置接上電源線和網路線。

## <a name="prerequisites"></a>必要條件

開始您的裝置佈線之前，您需要：

- 已打開包裝並掛接機架的 Edge 實體裝置。
- 兩條電源線。 
- 兩條 1-GbE RJ-45 網路纜線和四條 25-GbE SFP+ 銅纜線。
- 可以存取兩個配電裝置 (建議)。

您的 Edge 裝置有 8 個 NVMe SSD。 前面板也具有狀態 LED 燈與電源按鈕。 裝置背面包含備援的電源供應器 (PSU)。 您的裝置有六個網路介面：兩個 1-Gbps 介面和四個 25-Gbps 介面。 您的裝置有基礎板管理控制器 (BMC)。 識別裝置後擋板上的各個連接埠。
 
  ![已連接纜線的裝置後擋板](./media/data-box-edge-deploy-install/backplane-cabled.png)
 
請執行下列步驟，以連接裝置的電源線和網路線。

1. 將電源線連接至機箱中的每個 PSU。 為了確保高可用性，請安裝兩個 PSU，並將其連接到不同的電源。
2. 將電源線連接至機架配電裝置 (PDU)。 請確定兩個 PSU 使用不同的電源。
3. 將 1-GbE 網路介面連接埠 1 連接到用來設定實體裝置的電腦。 連接埠 1 是專用的管理介面。
4. 透過 RJ-45 網路纜線，將 1-GbE 網路介面連接埠 2 連接到資料中心網路/網際網路。 
5. 使用 SFP+ 銅纜線，將四個 25-GbE 網路介面 (連接埠 3、連接埠 4、連接埠 5 和連接埠 6) 連接到資料中心網路/網際網路。 

> [!NOTE]
> - 至少有一個資料網路介面 (連接埠 2、連接埠 3、連接埠 4、連接埠 5 或連接埠 6) 必須連接到網際網路 (Azure 的連線)。 
> - 建議您使用 25-GbE 網路介面 (例如連接埠 3、連接埠 4、連接埠 5 或連接埠 6) 將資料傳送至 Azure。 
> - Edge 裝置應連接到資料中心網路，以便擷取來自資料來源伺服器的資料。  


## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解 Data Box Edge 的相關主題，像是：

> [!div class="checklist"]
> * 打開裝置包裝
> * 安裝裝置機架
> * 接上裝置纜線

請繼續進行下一個教學課程，以了解如何連線、設定及啟用您的裝置。

> [!div class="nextstepaction"]
> [連線並設定您的 Data Box Edge](./data-box-edge-deploy-connect-setup-activate.md)


