---
title: 為 StorSimple Virtual Array 準備入口網站 | Microsoft Docs
description: 部署 StorSimple Virtual Array 的第一個教學課程，內容為如何準備 Azure 入口網站
services: storsimple
documentationcenter: NA
author: alkohli
manager: jeconnoc
editor: ''
ms.assetid: 68a4cfd3-94c9-46cb-805c-46217290ce02
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 11/14/2017
ms.author: alkohli
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 2126ff7ffd503e1d7b30997f3f32f30429cffefb
ms.sourcegitcommit: 8ebcecb837bbfb989728e4667d74e42f7a3a9352
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/21/2018
ms.locfileid: "42145565"
---
# <a name="deploy-storsimple-virtual-array---prepare-the-azure-portal"></a>部署 StorSimple Virtual Array - 準備 Azure 入口網站

![](./media/storsimple-virtual-array-deploy1-portal-prep/getstarted4.png)
## <a name="overview"></a>概觀

我們提供一系列如何將虛擬陣列完全部署為檔案伺服器或 iSCSI 伺服器 (使用 Resource Manager 模型) 的佈署教學課程，而這是該系列的第一篇文章。 本文章說明在佈建虛擬陣列之前，要建立並設定 StorSimple 裝置管理員服務所需的準備工作。 本文章也連結到部署設定檢查清單及設定必要條件。

您需要有系統管理員權限，才能完成安裝和設定程序。 我們建議您在開始之前，先檢閱部署設定檢查清單。 入口網站準備工作不到 10 分鐘就能完成。

本文中發佈的資訊適用於在 Azure 入口網站及 Microsoft Azure 政府服務雲端部署 StorSimple Virtual Array。

### <a name="get-started"></a>開始使用
部署的工作流程包括準備入口網站、在您的虛擬環境中佈建虛擬陣列，以及完成安裝程序。 若要開始將 StorSimple Virtual Array 部署為檔案伺服器和 iSCSI 伺服器，您必須參閱下表中的資源。

#### <a name="deployment-articles"></a>部署相關文章

若要部署 StorSimple Virtual Array，請依指定的順序參閱下列文章。

| **#** | **在此步驟中** | **執行此動作...** | **並使用這些文件。** |
| --- | --- | --- | --- |
| 1. |**設定 Azure 入口網站** |在佈建 StorSimple Virtual Array 之前，建立並設定 StorSimple 裝置管理員服務。 |[準備入口網站](storsimple-virtual-array-deploy1-portal-prep.md) |
| 2. |**佈建 Virtual Array** |對於 Hyper-V，在執行 Windows Server 2012 R2、Windows Server 2012 或 Windows Server 2008 R2 之 Hyper-V 的主機系統上，佈建並連接至 StorSimple Virtual Array。 <br></br> <br></br> 對於 VMware，請在執行 VMware ESXi 5.0、5.5 或 6.0 的主機系統上，佈建並連線至 StorSimple Virtual Array。<br></br> |[在 Hyper-V 中佈建虛擬陣列](storsimple-virtual-array-deploy2-provision-hyperv.md) <br></br> <br></br> [在 VMware 中佈建虛擬陣列](storsimple-virtual-array-deploy2-provision-vmware.md) |
| 3. |**設定 Virtual Array** |對於檔案伺服器，請執行初始安裝程序、註冊 StorSimple 檔案伺服器，以及完成裝置安裝程序。 接下來，您可以佈建 SMB 共用。 <br></br> <br></br> 對於 iSCSI 伺服器，請執行初始安裝、註冊 StorSimple iSCSI 伺服器，並完成裝置安裝程序。 接下來，您可以佈建 iSCSI 磁碟區。 |[將虛擬陣列設定為檔案伺服器](storsimple-virtual-array-deploy3-fs-setup.md)<br></br> <br></br>[將虛擬陣列設定為 iSCSI 伺服器](storsimple-virtual-array-deploy3-iscsi-setup.md) |

現在您可以開始設定 Azure 入口網站。

## <a name="configuration-checklist"></a>設定檢查清單

設定檢查清單說明您在設定 StorSimple Virtual Array 上的軟體之前需要收集的資訊。 事先備妥此資訊可協助簡化在環境中部署 StorSimple 裝置的程序。 視 StorSimple Virtual Array 部署為檔案伺服器或 iSCSI 伺服器而定，您需要下列其中一個檢查清單。

* 下載 [StorSimple Virtual Array 檔案伺服器的設定檢查清單](http://download.microsoft.com/download/E/E/6/EE690BB0-B442-4B84-8165-4731EE727ACF/MicrosoftAzureStorSimpleVirtualArrayFileServerConfigurationChecklist.pdf)。
* 下載 [StorSimple Virtual Array iSCSI 伺服器的設定檢查清單](http://download.microsoft.com/download/E/E/6/EE690BB0-B442-4B84-8165-4731EE727ACF/MicrosoftAzureStorSimpleVirtualArrayiSCSIServerConfigurationChecklist.pdf)。

## <a name="prerequisites"></a>必要條件

我們在此提供 StorSimple 裝置管理員服務、StorSimple Virtual Array 及資料中心網路的設定必要條件。

### <a name="for-the-storsimple-device-manager-service"></a>StorSimple 裝置管理員服務

在您開始前，請確定：

* 您擁有的 Microsoft 帳戶具有存取認證。
* 您擁有的 Microsoft Azure 儲存體帳戶具有存取認證。
* 您的 Microsoft Azure 訂用帳戶應該已啟用來使用 StorSimple 裝置管理員服務。

### <a name="for-the-storsimple-virtual-array"></a>StorSimple Virtual Array

在您部署虛擬陣列之前，請確定：

* 您可以存取可用來佈建裝置、在 Windows Server 2008 R2 或更新版本或 VMware (ESXi 5.0、5.5 或 6.0) 上執行 Hyper-V 的主機系統。
* 主機系統能夠把下列資源專門用來佈建虛擬陣列：
  
  * 至少 4 顆核心。
  * 至少 8 GB 的 RAM。 若計畫將虛擬陣列設定為檔案伺服器，8 GB 支援 2 百萬個檔案。 您需要 16 GB RAM 才能支援 2 - 4 百萬個檔案。
  * 一個網路介面。
  * 供系統資料使用的 500 GB 虛擬磁碟。

### <a name="for-the-datacenter-network"></a>對於資料中心的網路

在您開始前，請確定：

* 資料中心的網路是根據 StorSimple 裝置的網路需求所設定的。 如需詳細資訊，請參閱 [StorSimple Virtual Array 的系統需求](storsimple-ova-system-requirements.md)。
* StorSimple Virtual Array 隨時都有專用的 5 Mbps 網際網路頻寬 (或更多) 可用。 此頻寬不應與任何其他應用程式共用。

## <a name="step-by-step-preparation"></a>逐步的準備程序

請依照下列逐步指示，為您的 StorSimple 裝置管理員服務準備入口網站。

## <a name="step-1-create-a-new-service"></a>步驟 1：建立新的服務

單一 StorSimple 裝置管理員服務執行個體就能管理多個 StorSimple Virtual Array。 執行下列步驟來建立 StorSimple 裝置管理員服務的執行個體。 如果您已經有 StorSimple 裝置管理員服務來管理虛擬陣列，請略過此步驟，並移至[步驟 2：取得服務註冊金鑰](#step-2-get-the-service-registration-key)。

[!INCLUDE [storsimple-virtual-array-create-new-service](../../includes/storsimple-virtual-array-create-new-service.md)]

> [!IMPORTANT]
> 如果您並未啟用服務自動建立儲存體帳戶，您將必須在成功建立服務後，至少建立一個儲存體帳戶。
> 
> * 如果您未自動建立儲存體帳戶，請移至 [針對服務設定新的儲存體帳戶](#optional-step-configure-a-new-storage-account-for-the-service) 以取得詳細指示。
> * 如果您已啟用自動建立儲存體帳戶，請移至 [步驟 2：取得服務註冊金鑰](#step-2-get-the-service-registration-key)。
> 
> 

## <a name="step-2-get-the-service-registration-key"></a>步驟 2：取得服務註冊金鑰

當 StorSimple 裝置管理員服務已啟動並執行之後，您就必須取得服務註冊金鑰。 這個金鑰是用來註冊和將 StorSimple 裝置與服務連接。

請在 [Azure 入口網站](https://portal.azure.com/)中執行下列步驟。

[!INCLUDE [storsimple-virtual-array-get-service-registration-key](../../includes/storsimple-virtual-array-get-service-registration-key.md)]

> [!NOTE]
> 服務註冊金鑰用來註冊所有必須向 StorSimple 裝置管理員服務註冊的 StorSimple 裝置管理員裝置。
> 
> 

## <a name="step-3-download-the-virtual-array-image"></a>步驟 3：下載虛擬陣列映像

當您取得服務註冊金鑰之後，必須下載適當的虛擬陣列映像，以便在主機系統上佈建虛擬陣列。 虛擬陣列映像依作業系統而定，您可以從 Azure 入口網站的 [快速啟動] 頁面下載。

> [!IMPORTANT]
> StorSimple Virtual Array 上執行的軟體只能搭配 Storsimple 裝置管理員服務來使用。
> 
> 

請在 [Azure 入口網站](https://portal.azure.com/)中執行下列步驟。

#### <a name="to-get-the-virtual-array-image"></a>若要取得虛擬陣列映像

1. 登入 [Azure 入口網站](https://portal.azure.com/)。 
2. 在 Azure 入口網站中，按一下 [瀏覽] > [StorSimple 裝置管理員]。
3. 選取現有的 StorSimple 裝置管理員服務。 在 [StorSimple 裝置管理員] 刀鋒視窗中，按一下 [快速啟動]。 
4. 按一下與您想要從「Microsoft 下載中心」下載的映像對應的連結。 映像檔大約是 4.8 GB。
   
   * VHDX (適用於 Windows Server 2012 及更新版本上的 Hyper-V)
   * VHD (適用於 Windows Server 2008 R2 及更新版本上的 Hyper-V)
   * VMDK (適用於 VMWare ESXi 5.0、5.5 或 6.0)
5. 下載檔案並將檔案解壓縮至本機磁碟機，記下解壓縮檔案的所在位置。

## <a name="optional-step-configure-a-new-storage-account-for-the-service"></a>選用步驟：為服務設定新的儲存體帳戶

這是選擇性步驟，只有在您未啟用隨著服務自動建立儲存體帳戶時才需要執行。

如果您需要在不同區域建立 Azure 儲存體帳戶，請參閱[如何建立儲存體帳戶](../storage/common/storage-quickstart-create-account.md)來取得逐步指示。

請在 [Azure 入口網站](https://ms.portal.azure.com/)的 [StorSimple 裝置管理員服務] 頁面上執行下列步驟，以新增現有的 Microsoft Azure 儲存體帳戶。

#### <a name="to-add-a-storage-account-credential-that-has-the-same-azure-subscription-as-the-device-manager-service"></a>若要新增儲存體帳戶認證，而且其 Azure 訂用帳戶與裝置管理員服務相同

1. 瀏覽至您的裝置管理員服務，選取它並按兩下。 這會開啟 [概觀] 刀鋒視窗。
2. 在 [設定] 區段內選取 [儲存體帳戶認證]。
3. 按一下 [新增] 。
4. 在 [加入儲存體帳戶] 刀鋒視窗中，執行下列動作︰
   
    1. 在 [訂用帳戶] 中，選取 [目前]。
   
    2. 提供您 Azure 儲存體帳戶的名稱。
   
    3. 選取 [啟用]，為 StorSimple 裝置與雲端之間的網路通訊建立安全通道。 只有當您在私人雲端內操作時，才選取 [停用]。
   
    4. 按一下 [新增]。 成功建立儲存體帳戶之後會通知您。<br></br>
   
     ![新增現有儲存體帳戶認證](./media/storsimple-virtual-array-manage-storage-accounts/ova-add-storageacct.png)

## <a name="next-step"></a>後續步驟

下一個步驟是為 StorSimple Virtual Array 佈建虛擬機器。 請根據您的主機作業系統，來參閱下列其中一個詳細指示：

* [在 Hyper-V 中佈建 StorSimple Virtual Array](storsimple-virtual-array-deploy2-provision-hyperv.md)
* [在 VMware 中佈建 StorSimple Virtual Array](storsimple-virtual-array-deploy2-provision-vmware.md)

