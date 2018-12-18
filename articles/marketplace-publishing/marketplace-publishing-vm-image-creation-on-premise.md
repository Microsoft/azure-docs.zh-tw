---
title: 建立 Azure Marketplace 的內部部署虛擬機器映像 | Microsoft Docs
description: 了解及執行建立內部部署的 VM 映像，並且部署至 Azure Marketplace 以供他人購買的步驟。
services: marketplace-publishing
documentationcenter: ''
author: HannibalSII
manager: hascipio
editor: ''
ms.assetid: 26dfbd5a-8685-4b19-987e-c20ca60540ec
ms.service: marketplace
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: Azure
ms.workload: na
ms.date: 04/29/2016
ms.author: hascipio; v-divte
ms.openlocfilehash: f68dadab96e27cc7b90f44681d87ffa7cce8126b
ms.sourcegitcommit: f20e43e436bfeafd333da75754cd32d405903b07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49390052"
---
# <a name="develop-an-on-premises-virtual-machine-image-for-the-azure-marketplace"></a>開發 Azure Marketplace 的內部部署虛擬機器映像
強烈建議您使用遠端桌面通訊協定，在雲端中直接開發您的 Azure 虛擬硬碟 (VHD)。 不過如果需要，可以使用內部部署基礎結構來下載 VHD 並進行開發。  

如果是內部部署開發，您必須下載已建立之 VM 的作業系統 VHD。 這些步驟會在前述的步驟 3.3 進行。  

## <a name="download-a-vhd-image"></a>下載 VHD 映像
### <a name="locate-a-blob-url"></a>找出 Blob URL
若要下載 VHD，請先找出作業系統磁碟的 Blob URL。

從新的 [Microsoft Azure 入口網站](https://portal.azure.com)找出 Blob URL：

1. 移至 [瀏覽] > [VM]，然後選取已部署的 VM。
2. 在 [設定] 之下，選取 [磁碟] 磚，以開啟 [磁碟] 刀鋒視窗。
   
   ![繪圖](media/marketplace-publishing-vm-image-creation-on-premise/img01.png)
3. 選取 [作業系統磁碟] ，開啟另一個刀鋒視窗來顯示磁碟屬性 (包括 VHD 位置)。
4. 複製此 Blob URL。
   
   ![繪圖](media/marketplace-publishing-vm-image-creation-on-premise/img02.png)
5. 現在，刪除已部署的 VM 而不要刪除備份磁碟。 您也可以停止 VM，而不是刪除它。 不要在 VM 執行時下載作業系統 VHD。
   
   ![繪圖](media/marketplace-publishing-vm-image-creation-on-premise/img03.png)

### <a name="download-a-vhd"></a>下載 VHD
知道 Blob URL 之後，您就可以使用 [Azure 入口網站](http://manage.windowsazure.com/) 或 PowerShell 下載 VHD。  

> [!NOTE]
> 本指南建立時，新的 Microsoft Azure 入口網站還沒有下載 VHD 的功能。  
> 
> 

**透過目前的 [Azure 入口網站](http://manage.windowsazure.com/)下載作業系統 VHD**

1. 如果您尚未登入 Azure 入口網站，請先登入。
2. 按一下 [儲存體]  索引標籤。
3. 選取儲存 VHD 的儲存體帳戶。
   
   ![繪圖](media/marketplace-publishing-vm-image-creation-on-premise/img04.png)
4. 這樣即會顯示儲存體帳戶屬性 選取 [作業系統磁碟]  索引標籤。
   
   ![繪圖](media/marketplace-publishing-vm-image-creation-on-premise/img05.png)
5. 選取儲存 VHD 的容器。 根據預設，如果 VHD 是從入口網站建立，則會儲存在 vhds 容器中。
   
   ![繪圖](media/marketplace-publishing-vm-image-creation-on-premise/img06.png)
6. 比較該 URL 和您儲存的 URL，選取正確的作業系統 VHD。
7. 按一下 [下載] 。
   
   ![繪圖](media/marketplace-publishing-vm-image-creation-on-premise/img07.png)

### <a name="download-a-vhd-by-using-powershell"></a>使用 PowerShell 下載 VHD
除了使用 Azure 入口網站，還可以使用 [Save-AzureVhd](http://msdn.microsoft.com/library/dn495297.aspx) Cmdlet 下載作業系統 VHD。

        Save-AzureVhd –Source <storageURIOfVhd> `
        -LocalFilePath <diskLocationOnWorkstation> `
        -StorageKey <keyForStorageAccount>
例如，Save-AzureVhd -Source “https://baseimagevm.blob.core.windows.net/vhds/BaseImageVM-6820cq00-BaseImageVM-os-1411003770191.vhd” -LocalFilePath “C:\Users\Administrator\Desktop\baseimagevm.vhd” -StorageKey <String>

> [!NOTE]
> **Save-AzureVhd** 也有 **NumberOfThreads** 選項，可用來增加平行處理以將可用頻寬善用於下載。
> 
> 

## <a name="upload-vhds-to-an-azure-storage-account"></a>上傳 VHD 到 Azure 儲存體帳戶
如果您準備好內部部署的 VHD，就必須將它們上傳到 Azure 中的儲存體帳戶。 這個步驟會在建立內部部署的 VHD 之後，但在取得 VM 映像認證之前進行。

### <a name="create-a-storage-account-and-container"></a>建立儲存體帳戶和容器
我們建議將 VHD 上傳到美國境內某個區域的儲存體帳戶中。 單一 SKU 的所有 VHD 都應該放在單一儲存體帳戶內的單一容器中。

若要建立儲存體帳戶，可以使用 [Microsoft Azure 入口網站](https://portal.azure.com/)、PowerShell 或 Linux 命令列工具。  

**從 Microsoft Azure 入口網站建立儲存體帳戶**

1. 按一下 [建立資源]。
2. 選取 [儲存體] 。
3. 填入儲存體帳戶名稱，然後選取位置。
   
   ![繪圖](media/marketplace-publishing-vm-image-creation-on-premise/img08.png)
4. 按一下頁面底部的 [新增] 。
5. 此時應該會開啟已建立的儲存體帳戶的刀鋒視窗。 如果沒有，請選取 [瀏覽] > [儲存體帳戶]。 在 [儲存體帳戶] 刀鋒視窗中，選取先前建立的儲存體帳戶。
6. 選取 [容器] 。
   
   ![繪圖](media/marketplace-publishing-vm-image-creation-on-premise/img09.png) 
7. 在 [容器] 刀鋒視窗中，選取 [新增] ，然後輸入容器名稱和容器權限。 為容器權限選取 [私人]  。

> [!TIP]
> 建議您為每個計畫發佈的 SKU 建立一個容器。
> 
> 

  ![繪圖](media/marketplace-publishing-vm-image-creation-on-premise/img10.png)

### <a name="create-a-storage-account-by-using-powershell"></a>使用 PowerShell 建立儲存體帳戶
使用 PowerShell，執行 [New-AzureStorageAccount](https://docs.microsoft.com/powershell/module/servicemanagement/azure/new-azurestorageaccount) Cmdlet 建立儲存體帳戶。

        New-AzureStorageAccount -StorageAccountName “mystorageaccount” -Location “West US”

然後可以使用 [New-AzureStorageContainer](https://docs.microsoft.com/powershell/module/azure.storage/new-azurestoragecontainer) Cmdlet，在該儲存體帳戶內建立容器。

        New-AzureStorageContainer -Name “containername” -Permission “Off”

> [!NOTE]
> 這些命令假設目前的儲存體帳戶內容已在 PowerShell 中設定。   請參閱 [設定 Azure PowerShell](marketplace-publishing-powershell-setup.md) ，來了解 PowerShell 安裝程式的詳細資訊。  
> 
> ### <a name="create-a-storage-account-by-using-the-command-line-tool-for-mac-and-linux"></a>使用適用於 Mac 和 Linux 的命令列工具建立儲存體帳戶
> 從 [Linux 命令列工具](../virtual-machines/linux/cli-manage.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)建立儲存體帳戶，如下所示。
> 
> 

        azure storage account create mystorageaccount --location "West US"

如下所示建立容器。

        azure storage container create containername --account-name mystorageaccount --accountkey <accountKey>

## <a name="upload-a-vhd"></a>上傳 VHD
建立儲存體帳戶和容器之後，就可以上傳您準備好的 VHD。 您可以使用 PowerShell、Linux 命令列工具或其他 Azure 儲存體管理工具。

### <a name="upload-a-vhd-via-powershell"></a>透過 PowerShell 上傳 VHD
使用 [Add-AzureVhd](http://msdn.microsoft.com/library/dn495173.aspx) Cmdlet。

        Add-AzureVhd –Destination “http://mystorageaccount.blob.core.windows.net/containername/vmsku.vhd” -LocalFilePath “C:\Users\Administrator\Desktop\vmsku.vhd”

### <a name="upload-a-vhd-by-using-the-command-line-tool-for-mac-and-linux"></a>使用適用於 Mac 和 Linux 的命令列工具上傳 VHD
透過 [Linux 命令列工具](https://docs.microsoft.com/cli/azure/get-started-with-az-cli2)，使用下列語法：azure vm image create <image name> --location <Location of the data center> --OS Linux <LocationOfLocalVHD>

## <a name="see-also"></a>另請參閱
* [建立 Marketplace 的虛擬機器映像](marketplace-publishing-vm-image-creation.md)
* [設定 Azure PowerShell](marketplace-publishing-powershell-setup.md)

