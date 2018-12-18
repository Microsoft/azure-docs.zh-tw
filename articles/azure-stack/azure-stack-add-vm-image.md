---
title: 在 Azure Stack 中新增及移除 VM 映像 | Microsoft Docs
description: 新增或移除組織的自訂 Windows 或 Linux VM 映像以供租用戶使用。
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila
editor: ''
ms.assetid: e5a4236b-1b32-4ee6-9aaa-fcde297a020f
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: PowerShell
ms.topic: get-started-article
ms.date: 09/05/2018
ms.author: mabrigg
ms.reviewer: kivenkat
ms.openlocfilehash: 7a5c6875f080655e69f549e45ec474958128754f
ms.sourcegitcommit: e2ea404126bdd990570b4417794d63367a417856
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/14/2018
ms.locfileid: "45575808"
---
# <a name="make-a-virtual-machine-image-available-in-azure-stack"></a>在 Azure Stack 中提供虛擬機器映像

*適用於：Azure Stack 整合系統和 Azure Stack 開發套件*

在 Azure Stack 中，您可以製作虛擬機器映像供使用者使用。 這些映像可供 Azure Resource Manager 範本參考，或者您可以將其新增到 Azure Marketplace UI 作為 Marketplace 項目。 您可以使用全球 Azure Marketplace 中的映像，或新增自己的自訂映像。 您可以使用入口網站或 Windows PowerShell 新增 VM。

## <a name="add-a-vm-image-through-the-portal"></a>透過入口網站新增 VM 映像

> [!NOTE]
> 使用此方法時，您必須個別建立 Marketplace 項目。

映像必須能夠由 blob 儲存體 URI 參考。 準備 VHD 格式 (非 VHDX) 的 Windows 或 Linux 作業系統映像，然後將該映像上傳到 Azure 或 Azure Stack 中的儲存體帳戶。 如果您的映像已經上傳到 Azure 或 Azure Stack 中的 blob 儲存體，則您可以略過步驟 1。

1. [將 Windows VM 映像上傳至 Azure 供 Resource Manager 部署使用](https://azure.microsoft.com/documentation/articles/virtual-machines-windows-upload-image/)，或如果是 Linux 映像，則依照[在 Azure Stack 上部署 Linux 虛擬機器](azure-stack-linux.md)所述的指示操作。 您上傳映像之前，務必考慮下列因素：

   - Azure Stack 僅支援固定磁碟 VHD 格式的第一代 VM。 固定格式會線性地建構檔案內部的邏輯磁碟，因此磁碟位移 X 會儲存於 Blob 位移 X。Blob 最後的小頁尾將說明 VHD 屬性。 若要確認是否為固定磁碟，請使用 [Get-VHD](https://docs.microsoft.com/powershell/module/hyper-v/get-vhd?view=win10-ps) PowerShell 命令。  

    > [!IMPORTANT]
    >  Azure Stack 不支援動態磁碟 VHD。 對連結至 VM 的動態磁碟調整大小，會導致 VM 處於失敗狀態。 若要解決這個問題，請刪除 VM，但不要刪除 VM 的磁碟 (儲存體帳戶中的 VHD Blob)。 然後，將 VHD 從動態磁碟轉換為固定磁碟，並重新建立虛擬機器。

   * 將映像上傳到 Azure Stack blob 儲存體比上傳到 Azure blob 儲存體有效率，因為將映像推送到 Azure Stack 映像存放庫所需的時間較短。

   * 上傳 [Windows VM 映像](https://azure.microsoft.com/documentation/articles/virtual-machines-windows-upload-image/)時，請務必以[設定 Azure Stack 操作員的 PowerShell 環境](azure-stack-powershell-configure-admin.md)步驟取代「登入 Azure」步驟。  

   * 記下您上傳映像的 blob 儲存體 URI。 blob 儲存體 URI 的格式如下：*&lt;storageAccount&gt;/&lt;blobContainer&gt;/&lt;targetVHDName&gt;*.vhd。

   * 若要讓 Blob 可供匿名存取，請移至將 VM 映像 VHD 上傳時的儲存體帳戶 Blob 容器。 選取 [Blob]，然後選取 [存取原則]。 您可以選擇性地改為產生容器的共用存取簽章，然後將它包含在 Blob URI 中。 此步驟會確保 Blob 可用於將此項目新增為映像。 如果 Blob 不可供匿名存取，則建立的 VM 映像會處於失敗狀態。

   ![移至儲存體帳戶 Blob](./media/azure-stack-add-vm-image/image1.png)

   ![將 Blob 存取權設定為公用](./media/azure-stack-add-vm-image/image2.png)

2. 以操作員身分登入 Azure Stack。 在功能表中，選取 [所有服務]。 接著，在 [系統管理] 類別底下，選取 [計算] > [VM 映像] > [新增]。

3. 在 [新增 VM 映像] 底下，輸入虛擬機器映像的發行者、供應項目、SKU 及版本。 這些名稱區段指的是 Resource Manager 範本中的 VM 映像。 請務必正確選取 **osType** 值。 針對 [OS 磁碟 Blob URI]，輸入上傳映像所在的 Blob URI。 然後，選取 [建立] 以開始建立 VM 映像。

   ![開始建立映像](./media/azure-stack-add-vm-image/image4.png)

   成功建立映像時，VM 映像狀態會變更為「成功」。

4. 若要更容易在 UI 中將虛擬機器映像提供給使用者取用，最好是[建立 Marketplace 項目](azure-stack-create-and-publish-marketplace-item.md)。

## <a name="remove-a-vm-image-through-the-portal"></a>透過入口網站移除 VM 映像

1. 開啟位於 [https://adminportal.local.azurestack.external](https://adminportal.local.azurestack.external) 的管理入口網站。

2. 選取 [Marketplace 管理]，然後選取您要刪除的 VM。

3. 按一下 [刪除] 。

## <a name="add-a-vm-image-to-the-marketplace-by-using-powershell"></a>使用 PowerShell 將 VM 映像新增到 Marketplace

> [!Note]  
> 當您新增映像時，它只適用於以 Azure Resource Manger 為基礎的範本和 PowerShell 部署。 若要將映像當作 Marketplace 項目提供給使用者，請使用[建立及發佈 Marketplace 項目](https://docs.microsoft.com/azure/azure-stack/azure-stack-create-and-publish-marketplace-item)一文中的步驟發佈 Marketplace 項目。

1. [安裝適用於 Azure Stack 的 PowerShell](azure-stack-powershell-install.md)。  

2. 以操作員身分登入 Azure Stack。 如需相關指示，請參閱[以操作員身分登入 Azure Stack](azure-stack-powershell-configure-admin.md)。

3. 使用已提升權限的提示字元開啟 PowerShell，並執行：

  ````PowerShell  
    Add-AzsPlatformimage -publisher "<publisher>" `
      -offer "<offer>" `
      -sku "<sku>" `
      -version "<#.#.#>” `
      -OSType "<ostype>" `
      -OSUri "<osuri>"
  ````

  **Add-AzsPlatformimage** Cmdlet 會指定供 Azure Resource Manager 範本用來參考 VM 映像的值。 這些值包括：
  - **publisher**  
    例如：`Canonical`  
    部署映像時，使用者所使用 VM 映像的發行者名稱區段。 例如 **Microsoft**。 請勿在此欄位中包含空格或其他特殊字元。  
  - **供應項目**  
    例如：`UbuntuServer`  
    部署 VM 映像時，使用者所使用 VM 映像的供應項目名稱區段。 例如 **WindowsServer**。 請勿在此欄位中包含空格或其他特殊字元。  
  - **sku**  
    例如：`14.04.3-LTS`  
    部署 VM 映像時，使用者所使用 VM 映像的 SKU 名稱區段。 例如 **Datacenter2016**。 請勿在此欄位中包含空格或其他特殊字元。  
  - **version**  
    例如：`1.0.0`  
    部署 VM 映像時，使用者所使用 VM 映像的版本。 版本的格式為 *\#.\#.\#*。 例如 **1.0.0**。 請勿在此欄位中包含空格或其他特殊字元。  
  - **osType**  
    例如：`Linux`  
    映像的 osType 必須是 **Windows** 或 **Linux**。  
  - **OSUri**  
    例如：`https://storageaccount.blob.core.windows.net/vhds/Ubuntu1404.vhd`  
    您可以為 `osDisk` 指定 blob 儲存體 URI。  

    如需詳細資訊，請參閱 [Add-AzsPlatformimage](https://docs.microsoft.com/powershell/module/azs.compute.admin/add-azsplatformimage) Cmdlet 和 [New-DataDiskObject](https://docs.microsoft.com/powershell/module/Azs.Compute.Admin/New-DataDiskObject) Cmdlet 的 PowerShell 參考。

## <a name="add-a-custom-vm-image-to-the-marketplace-by-using-powershell"></a>使用 PowerShell 將自訂 VM 映像新增到 Marketplace
 
1. [安裝適用於 Azure Stack 的 PowerShell](azure-stack-powershell-install.md)。

  ```PowerShell  
    # Create the Azure Stack operator's Azure Resource Manager environment by using the following cmdlet:
    Add-AzureRMEnvironment `
      -Name "AzureStackAdmin" `
      -ArmEndpoint $ArmEndpoint

    Set-AzureRmEnvironment `
      -Name "AzureStackAdmin" `
      -GraphAudience $GraphAudience

    $TenantID = Get-AzsDirectoryTenantId `
      -AADTenantName "<myDirectoryTenantName>.onmicrosoft.com" `
      -EnvironmentName AzureStackAdmin

    Add-AzureRmAccount `
      -EnvironmentName "AzureStackAdmin" `
      -TenantId $TenantID
  ```

2. 如果使用 **Active Directory 同盟服務**，請使用下列 Cmdlet：

  ```PowerShell
  # For Azure Stack Development Kit, this value is set to https://adminmanagement.local.azurestack.external. To get this value for Azure Stack integrated systems, contact your service provider.
  $ArmEndpoint = "<Resource Manager endpoint for your environment>"

  # For Azure Stack Development Kit, this value is set to https://graph.local.azurestack.external/. To get this value for Azure Stack integrated systems, contact your service provider.
  $GraphAudience = "<GraphAuidence endpoint for your environment>"

  # Create the Azure Stack operator's Azure Resource Manager environment by using the following cmdlet:
  Add-AzureRMEnvironment `
    -Name "AzureStackAdmin" `
    -ArmEndpoint $ArmEndpoint
    ```

3. 以操作員身分登入 Azure Stack。 如需相關指示，請參閱[以操作員身分登入 Azure Stack](azure-stack-powershell-configure-admin.md)。

4. 在全域 Azure 或 Azure Stack 中建立儲存體帳戶來儲存自訂 VM 映像。 如需相關指示，請參閱[快速入門：使用 Azure 入口網站上傳、下載及列出 blob](https://docs.microsoft.com/azure/storage/blobs/storage-quickstart-blobs-portal)。

5. 準備 VHD 格式 (非 VHDX) 的 Windows 或 Linux 作業系統映像，將該映像上傳到儲存體帳戶，並取得可供 PowerShell 擷取 VM 映像的 URI。  

  ````PowerShell  
    Add-AzureRmAccount `
      -EnvironmentName "AzureStackAdmin" `
      -TenantId $TenantID
  ````

6. (選擇性) 您可以上傳資料磁碟陣列作為 VM 映像的一部分。 使用 New-DataDiskObject Cmdlet 建立資料磁碟。 從已提升權限的提示字元開啟 PowerShell，並執行：

  ````PowerShell  
    New-DataDiskObject -Lun 2 `
    -Uri "https://storageaccount.blob.core.windows.net/vhds/Datadisk.vhd"
  ````

7. 使用已提升權限的提示字元開啟 PowerShell，並執行：

  ````PowerShell  
    Add-AzsPlatformimage -publisher "<publisher>" -offer "<offer>" -sku "<sku>" -version "<#.#.#>” -OSType "<ostype>" -OSUri "<osuri>"
  ````

    如需 Add-AzsPlatformimage Cmdlet 和 New-DataDiskObject Cmdlet 的詳細資訊，請參閱 Microsoft PowerShell [Azure Stack 操作員模組文件](https://docs.microsoft.com/powershell/module/)。

## <a name="remove-a-vm-image-by-using-powershell"></a>使用 PowerShell 來移除 VM 映像

當您已不再需要上傳的虛擬機器映像時，您可以使用下列 Cmdlet 將它從 Marketplace 中刪除：

1. [安裝適用於 Azure Stack 的 PowerShell](azure-stack-powershell-install.md)。

2. 以操作員身分登入 Azure Stack。

3. 使用已提升權限的提示字元開啟 PowerShell，並執行：

  ````PowerShell  
  Remove-AzsPlatformImage `
    -publisher "<publisher>" `
    -offer "<offer>" `
    -sku "<sku>" `
    -version "<version>" `
  ````
  **Remove-AzsPlatformimage** Cmdlet 會指定供 Azure Resource Manager 範本用來參考 VM 映像的值。 這些值包括：
  - **publisher**  
    例如：`Canonical`  
    部署映像時，使用者所使用 VM 映像的發行者名稱區段。 例如 **Microsoft**。 請勿在此欄位中包含空格或其他特殊字元。  
  - **供應項目**  
    例如：`UbuntuServer`  
    部署 VM 映像時，使用者所使用 VM 映像的供應項目名稱區段。 例如 **WindowsServer**。 請勿在此欄位中包含空格或其他特殊字元。  
  - **sku**  
    例如：`14.04.3-LTS`  
    部署 VM 映像時，使用者所使用 VM 映像的 SKU 名稱區段。 例如 **Datacenter2016**。 請勿在此欄位中包含空格或其他特殊字元。  
  - **version**  
    例如：`1.0.0`  
    部署 VM 映像時，使用者所使用 VM 映像的版本。 版本的格式為 *\#.\#.\#*。 例如 **1.0.0**。 請勿在此欄位中包含空格或其他特殊字元。  
    
    如需 Remove-AzsPlatformImage Cmdlet 的詳細資訊，請參閱 Microsoft PowerShell [Azure Stack 操作員模組文件](https://docs.microsoft.com/powershell/module/)。

## <a name="next-steps"></a>後續步驟

[佈建虛擬機器](azure-stack-provision-vm.md)
