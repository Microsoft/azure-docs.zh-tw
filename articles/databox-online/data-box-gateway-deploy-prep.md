---
title: 準備 Azure 入口網站以部署資料箱閘道的教學課程 | Microsoft Docs
description: 部署 Azure 資料箱閘道的第一個教學課程說明如何準備 Azure 入口網站。
services: databox-edge-gateway
author: alkohli
ms.service: databox
ms.subservice: gateway
ms.topic: tutorial
ms.date: 09/24/2018
ms.author: alkohli
ms.openlocfilehash: b8aac8a6a666bf3f360ef2867ff15933ad72dbd7
ms.sourcegitcommit: c282021dbc3815aac9f46b6b89c7131659461e49
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/12/2018
ms.locfileid: "49166979"
---
# <a name="tutorial-prepare-to-deploy-azure-data-box-gateway-preview"></a>教學課程：準備部署 Azure 資料箱閘道 (預覽)


我們提供了一系列的部署教學課程來說明如何完整部署 Azure 資料箱閘道，這是此系列的第一個教學課程。 本教學課程說明如何準備 Azure 入口網站以部署資料箱閘道資源。 

您需要有系統管理員權限，才能完成安裝和設定程序。 入口網站準備工作不到 10 分鐘就能完成。

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 建立新的資源
> * 下載虛擬裝置映像
> * 取得啟用金鑰

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。


> [!IMPORTANT]
> - 資料箱閘道處於預覽狀態。 部署訂購並部署此解決方案之前，請檢閱 [Azure 預覽版使用條款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。 

### <a name="get-started"></a>開始使用

若要部署資料箱閘道，請依指定的順序參閱下列教學課程。

| **#** | **在此步驟中** | **使用這些文件** |
| --- | --- | --- | 
| 1. |**[為 Azure 入口網站進行資料箱閘道的準備工作](data-box-gateway-deploy-prep.md)** |在佈建資料箱閘道虛擬裝置之前，請先建立並設定資料箱閘道資源。 |
| 2. |**[在 Hyper-V 中佈建資料箱閘道](data-box-gateway-deploy-provision-hyperv.md)** <br><br><br>**[在 VMware 中佈建資料箱閘道](data-box-gateway-deploy-provision-vmware.md)**|針對 Hyper-V，請在使用 Windows Server 2016 或 Windows Server 2012 R2 執行 Hyper-V 的主機系統上，佈建並連線至資料箱閘道虛擬裝置。 <br><br><br> 針對 VMware，請在執行 VMware ESXi 6.0 或 6.5 的主機系統上佈建並連線至資料箱閘道虛擬裝置。<br></br> |
| 3. |**[連線、設定、啟動資料箱閘道](data-box-gateway-deploy-connect-setup-activate.md)** |連線至本機 Web UI、完成裝置設定，並啟動裝置。 接下來，您可以佈建 SMB 共用。  |
| 4. |**[使用資料箱閘道傳輸資料](data-box-gateway-deploy-add-shares.md)** |新增共用，並透過 SMB 或 NFS 連線至共用。 |

現在您可以開始設定 Azure 入口網站。

## <a name="prerequisites"></a>必要條件

我們在此提供資料箱閘道資源、資料箱閘道裝置和資料中心網路的設定必要條件。

### <a name="for-the-data-box-gateway-resource"></a>針對資料箱閘道資源

在您開始前，請確定：

* 您的 Microsoft Azure 訂用帳戶應已啟用資料箱閘道資源。
* 您擁有的 Microsoft Azure 儲存體帳戶具有存取認證。

### <a name="for-the-data-box-gateway-device"></a>針對資料箱閘道裝置

在您部署虛擬裝置之前，請確定：

* 您可以存取在 Windows Server 2012 R2 或更新版本或 VMware (ESXi 6.0 或 6.5) 上執行 Hyper-V，並且可用來佈建裝置的主機系統。
* 此主機系統能夠將下列資源專門用來佈建您的資料箱虛擬裝置：
  
  * 至少 4 顆核心。
  * 至少 8 GB 的 RAM。 
  * 一個網路介面。
  * 250 GB 的 OS 磁碟。
  * 供系統資料使用的 2 TB 虛擬磁碟。

### <a name="for-the-datacenter-network"></a>對於資料中心的網路

在您開始前，請確定：

* 資料中心的網路是根據資料箱閘道裝置的網路需求所設定的。 如需詳細資訊，請參閱[資料箱閘道系統需求](data-box-gateway-system-requirements.md)。

* 資料箱閘道隨時都有專用的 20 Mbps 網際網路頻寬 (或更多) 可用。 此頻寬不應與任何其他應用程式共用。 如果使用網路節流，若要讓節流能夠運作，建議您使用 32 Mbps 或更高的網際網路頻寬。

## <a name="create-a-new-resource"></a>建立新的資源

請執行下列步驟，建立新的資料箱閘道資源。 

如果您目前已有可用來管理虛擬裝置的資料箱閘道資源，請略過此步驟，並移至[取得啟用金鑰](#get-the-activation-key)。

請在 Azure 入口網站中執行下列步驟，以建立資料箱資源。
1. 使用您的 Microsoft Azure 認證經由以下 URL 登入 Azure 預覽入口網站：[https://aka.ms/databox-edge](https://aka.ms/databox-edge)。 

2. 挑選您要用於 Data Box Edge 預覽的訂用帳戶。 選取要部署 Data Box Edge 資源的區域。 在 [資料箱閘道] 選項中，按一下 [建立]。

    ![搜尋資料箱閘道服務](media/data-box-gateway-deploy-prep/data-box-gateway-edge-sku.png)

3. 對於新資源，請輸入或選取下列資訊。
    
    |設定  |值  |
    |---------|---------|
    |資源名稱   | 可識別資源的易記名稱。<br>此名稱介於 2 到 50 個字元之間，並且可包含字母、數字和連字號。<br> 名稱必須以字母或數字為開頭或結尾。        |
    |訂用帳戶    |訂用帳戶會連結到您的帳單帳戶。 |
    |資源群組  |選取現有的群組或建立新的群組。<br>深入了解 [Azure 資源群組](../azure-resource-manager/resource-group-overview.md)。     |
    |位置     |此版本適用於美國東部、美國西部 2、東南亞和西歐等區域。 <br> 請選擇與您要部署裝置的地理區域最接近的位置。|
    
    ![建立資料箱閘道資源](media/data-box-gateway-deploy-prep/data-box-gateway-resource.png)
    
4. 按一下 [確定]。
 
建立資源需要幾分鐘的時間。 當資源成功建立後，系統將會通知您。


## <a name="download-the-virtual-device-image"></a>下載虛擬裝置映像

資料箱閘道資源建立後，請下載適當的虛擬裝置映像，以在主機系統上佈建虛擬裝置。 虛擬裝置映像會隨著作業系統而不同，您可以在 Azure 入口網站中，從資源的 [快速入門] 刀鋒視窗加以下載。

> [!IMPORTANT]
> 在資料箱閘道上執行的軟體只能用於資料箱閘道資源。


請在 [Azure 入口網站](https://portal.azure.com/)中執行下列步驟。

1. 按一下您所建立的資源，然後按一下 [概觀]。 如果您有現有的 Azure 資料箱閘道資源，請按一下該資源，並移至 [概觀]。

    ![新增資料箱閘道資源](media/data-box-gateway-deploy-prep/data-box-gateway-resource-created.png)

4. 在快速入門的右窗格中，按一下與您要下載的映像對應的連結。 映像檔大約是 4.8 GB。
   
   * [VHDX (適用於 Windows Server 2012 R2 及更新版本上的 Hyper-V)](https://aka.ms/dbe-vhdx-2012)。
   * [VMDK (適用於 VMWare ESXi 6.0 或 6.5)](https://aka.ms/dbe-vmdk)。

5. 下載檔案並將檔案解壓縮至本機磁碟機，記下解壓縮檔案的所在位置。


## <a name="get-the-activation-key"></a>取得啟用金鑰

在資料箱閘道資源啟動並執行後，您必須取得啟用金鑰。 此金鑰可用來啟動資料箱閘道裝置，並將其與資源連線。

啟用金鑰可用來註冊所有必須以資料箱閘道資源啟動的資料箱閘道裝置。 您現在可以在 Azure 入口網站中取得此金鑰。

1. 按一下您所建立的資源，然後按一下 [概觀]。

    ![新增資料箱閘道資源](media/data-box-gateway-deploy-prep/data-box-gateway-resource-created.png)

2. 按一下 [產生金鑰] 以建立啟用金鑰。 按一下複製圖示以複製此金鑰，並加以儲存，以供日後使用。

    ![取得啟用金鑰](media/data-box-gateway-deploy-prep/get-activation-key.png)

> [!IMPORTANT]
> - 啟用金鑰在產生的 3 天後即到期。 
> - 如果金鑰已過期，請產生新的金鑰。 舊金鑰沒有效用。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解資料箱閘道的相關主題，像是：

> [!div class="checklist"]
> * 建立新的資源
> * 下載虛擬裝置映像
> * 取得啟用金鑰

請繼續進行下一個教學課程，以了解如何為資料箱閘道佈建虛擬機器。 請根據您的主機作業系統，來參閱下列其中一個詳細指示：

> [!div class="nextstepaction"]
> [在 Hyper-V 中佈建資料箱閘道](./data-box-gateway-deploy-provision-hyperv.md)

或

> [!div class="nextstepaction"]
> [在 VMware 中佈建資料箱閘道](./data-box-gateway-deploy-provision-vmware.md)


