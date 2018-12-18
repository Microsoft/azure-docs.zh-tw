---
title: 使用 Azure Marketplace 映像透過受控識別來建立 Terraform Linux 虛擬機器
description: 使用 Azure Marketplace 映像，透過受控識別與遠端狀態管理來建立 Terraform Linux 虛擬機器，輕鬆地將資源部署至 Azure。
services: terraform
ms.service: terraform
keywords: terraform, devops, MSI, 虛擬機器, 遠端狀態, azure
author: tomarcher
manager: jeconnoc
ms.author: tarcher
ms.topic: tutorial
ms.date: 3/12/2018
ms.openlocfilehash: 1ec6228993c516ce2974c64bfa5b6dcdf63e7f91
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49343821"
---
# <a name="use-an-azure-marketplace-image-to-create-a-terraform-linux-virtual-machine-with-managed-identities-for-azure-resources"></a>使用 Azure Marketplace 映像透過 Azure 資源的受控識別來建立 Terraform Linux 虛擬機器

本文示範如何使用 [Terraform Marketplace 映像](https://azuremarketplace.microsoft.com/marketplace/apps/azure-oss.terraform?tab=Overview)來建立 Ubuntu Linux VM (16.04 LTS)，並使用 [Azure 資源的受控識別](https://docs.microsoft.com/azure/active-directory/managed-service-identity/overview)來安裝及設定最新的 [Terraform](https://www.terraform.io/intro/index.html) 版本。 此映像也會設定遠端後端，以使用 Terraform 啟用[遠端狀態](https://www.terraform.io/docs/state/remote.html)管理。 

Terraform Marketplace 映像可讓您輕鬆地在 Azure 上開始使用 Terraform，而不必手動安裝及設定 Terraform。 

這個 Terraform VM 映像沒有任何軟體費用。 您僅需支付 Azure 硬體使用費，這是根據所佈建的虛擬機器大小來評估。 如需費用計算的詳細資料，請參閱 [Linux 虛擬機器定價頁面](https://azure.microsoft.com/pricing/details/virtual-machines/linux/)。

## <a name="prerequisites"></a>必要條件
您必須先具有 Azure 訂用帳戶，才可以建立 Linux Terraform 虛擬機器。 如果您還沒有訂用帳戶，請參閱[立即建立免費的 Azure 帳戶](https://azure.microsoft.com/free/)。  

## <a name="create-your-terraform-virtual-machine"></a>建立 Terraform 虛擬機器 

建立 Linux Terraform 虛擬機器執行個體的步驟如下： 

1. 在 Azure 入口網站中，移至 [建立資源](https://ms.portal.azure.com/#create/hub) 清單。

2. 在 **搜尋 Marketplace** 搜尋列中搜尋 **Terraform**。 選取 **Terraform** 範本。 

3. 在右下方的 Terraform 詳細資料索引標籤中，選取 **建立** 按鈕。

    ![建立 Terraform 虛擬機器](media\terraformmsi.png)

4. 下列各節提供精靈中每個步驟的輸入資訊，可用來建立 Terraform Linux 虛擬機器。 下一節列出設定每個步驟所需的輸入資訊。

## <a name="details-on-the-create-terraform-tab"></a>建立 Terraform 索引標籤的詳細資料

在 [建立 Terraform] 索引標籤中輸入以下的詳細資料：

1. **基本概念**
    
   * **名稱**：Terraform 虛擬機器的名稱。
   * **使用者名稱**：第一個帳戶登入識別碼。
   * **密碼**：第一個帳戶密碼。 (您可以使用 SSH 公開金鑰而不使用密碼。)
   * **訂用帳戶**：要用來建立虛擬機器和開立帳單的訂用帳戶。 您必須有此訂用帳戶的資源建立權限。
   * **資源群組**：新的或現有的資源群組。
   * **位置**：最適合的資料中心。 通常是擁有您大部分資料的資訊中心，或是最接近您實際位置可進行最快速網路存取的資料中心。

2. **其他設定**

   * **大小**：虛擬機器的大小。 
   * **VM 磁碟類型**：SSD 或 HDD。

3. **摘要 Terraform**

   * 請確認您輸入的所有資訊都正確無誤。 

4. **購買**

   * 若要開始佈建程序，選取 [購買]。 會提供一個交易條款的連結。 VM 除了計算您在 [大小] 步驟中所選擇的伺服器大小之外，不會收取任何其他費用。

Terraform VM 映像會執行下列步驟：

* 使用系統根據 Ubuntu 16.04 LTS 映像指派的身分識別來建立 VM。
* 在 VM 上安裝 MSI 擴充功能，以允許要針對 Azure 資源發行的 OAuth 權杖。
* 將 RBAC 權限指派給受控身分識別，從而授與資源群組的擁有者權限。
* 建立 Terraform 範本資料夾 (tfTemplate)。
* 使用 Azure 後端預先設定 Terraform 遠端狀態。

## <a name="access-and-configure-a-linux-terraform-virtual-machine"></a>存取及設定 Linux Terraform 虛擬機器

建立 VM 之後，您就可以使用 SSH 登入 VM。 針對文字殼層介面，使用您在步驟 3 「基本資料」一節中建立的帳戶認證。 在 Windows 上，您可以下載 SSH 用戶端工具，例如 [Putty](http://www.putty.org/)。

使用 SSH 連線至虛擬機器後，您必須將整個訂用帳戶的參與者權限授予虛擬機器上 Azure 資源的受控識別。 

參與者權限可協助 VM 的 MSI 使用 Terraform 來建立 VM 資源群組外部的資源。 您可以執行一次指令碼，輕鬆達成這個動作。 使用下列命令：

`. ~/tfEnv.sh`

先前的指令碼會使用 [AZ CLI 2.0 版互動式登入](https://docs.microsoft.com/cli/azure/authenticate-azure-cli?view=azure-cli-latest#interactive-log-in)機制向 Azure 驗證，並指派整個訂用帳戶上的虛擬機器受控識別參與者權限。 

 VM 有一個 Terraform 遠端狀態後端。 若要在您的 Terraform 部署中啟用此後端，將 remoteState.tf 檔案從 tfTemplate 目錄複製到 Terraform 指令碼的根目錄。  

 `cp  ~/tfTemplate/remoteState.tf .`

 如需遠端狀態管理的詳細資訊，請參閱[與 Terraform 遠端狀態相關的此網頁](https://www.terraform.io/docs/state/remote.html)。 儲存體存取金鑰會在這個檔案中公開，且在將 Terraform 組態檔認可至原始檔控制之前，需要排除儲存體存取金鑰。

## <a name="next-steps"></a>後續步驟
在本文中，您已了解如何在 Azure 上設定 Terraform Linux 虛擬機器。 以下有一些額外的資源，可協助您深入了解 Azure 上的 Terraform： 

 [Microsoft.com 中的 Terraform 中樞](https://docs.microsoft.com/azure/terraform/)  
 [Terraform Azure 提供者文件](http://aka.ms/terraform)  
 [Terraform Azure 提供者來源](http://aka.ms/tfgit)  
 [Terraform Azure 模組](http://aka.ms/tfmodules)
 

















