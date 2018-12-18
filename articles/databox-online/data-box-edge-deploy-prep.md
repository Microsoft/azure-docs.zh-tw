---
title: 準備 Azure 入口網站以部署 Data Box Edge 的教學課程 | Microsoft Docs
description: 部署 Azure Data Box Edge 的第一個教學課程會說明如何準備 Azure 入口網站。
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: tutorial
ms.date: 10/08/2018
ms.author: alkohli
Customer intent: As an IT admin, I need to understand how to prepare the portal to deploy Data Box Edge so I can use it to transfer data to Azure.
ms.openlocfilehash: 35ac28d687c8bc6636a7d8e10f54ffb5b219a776
ms.sourcegitcommit: c282021dbc3815aac9f46b6b89c7131659461e49
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/12/2018
ms.locfileid: "49167455"
---
# <a name="tutorial-prepare-to-deploy-azure-data-box-edge-preview"></a>教學課程：準備部署 Azure Data Box Edge (預覽)


我們提供了一系列的部署教學課程來說明如何完整部署 Azure Data Box Edge，這是此系列的第一個教學課程。 本教學課程說明如何準備 Azure 入口網站以部署 Data Box Edge 資源。 

您需要有系統管理員權限，才能完成安裝和設定程序。 入口網站準備工作不到 10 分鐘就能完成。

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 建立新的資源
> * 取得啟用金鑰

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。


> [!IMPORTANT]
> Data Box Edge 目前處於預覽狀態。 部署訂購並部署此解決方案之前，請檢閱 [Azure 預覽版使用條款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。 

### <a name="get-started"></a>開始使用

若要部署 Data Box Edge，請依指定的順序參閱下列教學課程。

| **#** | **在此步驟中** | **使用這些文件** |
| --- | --- | --- | 
| 1. |**[為 Azure 入口網站進行 Data Box Edge 的準備工作](data-box-edge-deploy-prep.md)** |在安裝 Data Box Edge 實體裝置之前，請先建立並設定您的 Data Box Edge 資源。 |
| 2. |**[安裝 Data Box Edge](data-box-edge-deploy-install.md)**|打開包裝，然後接上 Data Box Edge 實體裝置的機架和纜線。  |
| 3. |**[連線、設定、啟動 Data Box Edge](data-box-edge-deploy-connect-setup-activate.md)** |連線至本機 Web UI、完成裝置設定，並啟動裝置。 裝置已就緒，可設定 SMB 或 NFS 共用。  |
| 4. |**[使用 Data Box Edge 傳輸資料](data-box-edge-deploy-add-shares.md)** |新增共用，並透過 SMB 或 NFS 連線至共用。 |
| 5. |**[使用 Data Box Edge 轉換資料](data-box-edge-deploy-configure-compute.md)** |在裝置上設定 Edge 模組，以在資料移至 Azure 時加以轉換。 |

現在您可以開始設定 Azure 入口網站。

## <a name="prerequisites"></a>必要條件

我們在此提供 Data Box Edge 資源、Data Box Edge 裝置和資料中心網路的設定必要條件。

### <a name="for-the-data-box-edge-resource"></a>針對 Data Box Edge 資源

在您開始前，請確定：

* 您的 Microsoft Azure 訂用帳戶應已啟用 Data Box Edge 資源。
* 您擁有的 Microsoft Azure 儲存體帳戶具有存取認證。

### <a name="for-the-data-box-edge-device"></a>針對 Data Box Edge 裝置

在您部署實體裝置之前，請確定：

- 您在資料中心的標準 19 吋機架中有 1 U 插槽可用來以機架掛接裝置。 
- 確定您有平穩的工作介面可安全地放置裝置，且方便您操作。
- 確認您要安裝裝置的場地具有獨立來源的標準 AC 電源，或是具有不斷電供應系統 (UPS) 的機架電源分配單元 (PDU)。
- 您可以拆裝實體裝置。


### <a name="for-the-datacenter-network"></a>對於資料中心的網路

在您開始前，請確定：

* 資料中心的網路是根據 Data Box Edge 裝置的網路需求所設定的。 如需詳細資訊，請參閱 [Data Box Edge 系統需求](data-box-gateway-system-requirements.md)。

* Data Box Edge 隨時都有專用的 20 Mbps 網際網路頻寬 (或更多) 可用。 此頻寬不應與任何其他應用程式共用。 如果使用網路節流，若要讓節流能夠運作，建議您使用 32 Mbps 或更高的網際網路頻寬。

## <a name="create-a-new-resource"></a>建立新的資源

請執行下列步驟，建立新的 Data Box Edge 資源。 

如果您目前已有可用來管理實體裝置的 Data Box Edge 資源，請略過此步驟，並移至[取得啟用金鑰](#get-the-activation-key)。

請在 Azure 入口網站中執行下列步驟，以建立資料箱資源。

1. 使用您的 Microsoft Azure 認證經由以下 URL 登入 Azure 預覽入口網站：[https://aka.ms/databox-edge](https://aka.ms/databox-edge)。 

2. 挑選您要用於 Data Box Edge 預覽的訂用帳戶。 選取要部署 Data Box Edge 資源的區域。 在 [Data Box Edge] 選項中，按一下 [建立]。

    ![搜尋 Data Box Edge 服務](media/data-box-edge-deploy-prep/data-box-edge-sku.png)

3. 對於新資源，請輸入或選取下列資訊。
    
    |設定  |值  |
    |---------|---------|
    |資源名稱   | 可識別資源的易記名稱。<br>資源名稱介於 2 到 50 個字元之間，並且可包含字母、數字和連字號。<br> 名稱必須以字母或數字為開頭或結尾。        |
    |訂用帳戶    |訂用帳戶會連結到您的帳單帳戶。 |
    |資源群組  |選取現有的群組或建立新的群組。<br>深入了解 [Azure 資源群組](../azure-resource-manager/resource-group-overview.md)。     |
    |位置     |此版本適用於美國東部、美國西部 2、東南亞和西歐等區域。 <br> 請選擇與您要部署裝置的地理區域最接近的位置。|
    
    ![建立 Data Box Edge 資源](media/data-box-edge-deploy-prep/data-box-edge-resource.png)
    
4. 按一下 [確定]。
 
建立資源需要幾分鐘的時間。 當資源成功建立後，系統將會通知您。


## <a name="get-the-activation-key"></a>取得啟用金鑰

在 Data Box Edge 資源啟動並執行後，您必須取得啟用金鑰。 此金鑰可用來啟動 Data Box Edge 裝置，並將其與資源連線。 您現在可以在 Azure 入口網站中取得此金鑰。

1. 按一下您所建立的資源，然後按一下 [概觀]。

2. 按一下 [產生金鑰] 以建立啟用金鑰。 按一下複製圖示以複製此金鑰，並加以儲存，以供日後使用。

    ![取得啟用金鑰](media/data-box-edge-deploy-prep/get-activation-key.png)

> [!IMPORTANT]
> - 啟用金鑰在產生的 3 天後即到期。 
> - 如果金鑰已過期，請產生新的金鑰。 舊金鑰沒有效用。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解 Data Box Edge 的相關主題，像是：

> [!div class="checklist"]
> * 建立新的資源
> * 取得啟用金鑰

請繼續進行下一個教學課程，以了解如何安裝 Data Box Edge。 

> [!div class="nextstepaction"]
> [安裝 Data Box Edge](./data-box-edge-deploy-install.md)



