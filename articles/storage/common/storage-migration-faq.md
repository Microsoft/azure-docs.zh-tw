---
title: Azure 儲存體移轉常見問題集 | Microsoft Docs
description: 移轉 Azure 儲存體常見問題的解答
services: storage
author: genlin
ms.service: storage
ms.topic: article
ms.date: 05/11/2018
ms.author: genli
ms.component: common
ms.openlocfilehash: 3c671fe6dcb1af25bfb0aa09fecf8ad5e5ceb97b
ms.sourcegitcommit: 1b561b77aa080416b094b6f41fce5b6a4721e7d5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/17/2018
ms.locfileid: "45732255"
---
# <a name="frequently-asked-questions-about-azure-storage-migration"></a>關於 Azure 儲存體移轉的常見問題集

本文提供 Azure 儲存體移轉的常見問題解答。 

## <a name="faq"></a>常見問題集

**如何建立指令碼以將檔案從一個容器複製到另一個容器？**

若要在容器之間複製檔案，可以使用 AzCopy。 請參閱下列範例：

    AzCopy /Source:https://xxx.blob.core.windows.net/xxx
    /Dest:https://xxx.blob.core.windows.net/xxx /SourceKey:xxx /DestKey:xxx
    /S

AzCopy 使用[複製 Blob API](https://docs.microsoft.com/rest/api/storageservices/copy-blob) \(英文\) 來複製容器中的各個檔案。  
  
只要具有網際網路存取權限，任何虛擬機器或本機電腦都可以用來執行 AzCopy。 您也可以使用 Azure Batch 排程自動執行此操作，但是會比較複雜。  
  
自動化指令碼是專門為了部署 Azure Resource Manager 所設計，而不是用來操作儲存體內容。 如需詳細資訊，請參閱[使用 Resource Manager 範本與 Azure PowerShell 來部署資源](../../azure-resource-manager/resource-group-template-deploy.md)。

**有兩個檔案共用，位在相同區域的相同儲存體帳戶上，在這兩者間複製資料是否收取費用？**

否。 此流程不會收取費用。

**如何將我的整個儲存體帳戶備份到其他儲存體帳戶？**

沒有選項可直接備份整個儲存體帳戶。 但您可以使用 AzCopy 或儲存體總管，將該儲存體帳戶中的容器手動移動至另一個帳戶。 下列步驟示範了如何使用 AzCopy 移動容器：  
 

1.  安裝 [AzCopy](storage-use-azcopy.md) 命令列工具。 此工具可協助您在儲存體帳戶之間移動 VHD 檔案。

2.  以安裝程式在 Windows 上安裝好 AzCopy 之後，請開啟 [命令提示字元] 視窗，然後瀏覽至您電腦上的 AzCopy 安裝資料夾。 根據預設，AzCopy 會安裝到 **%ProgramFiles(x86)%\Microsoft SDKs\Azure\AzCopy** 或 **%ProgramFiles%\Microsoft SDKs\Azure\AzCopy**。

3.  執行下列命令以移動容器。 您必須以實際值來取代文字。   
     
            AzCopy /Source:https://sourceaccount.blob.core.windows.net/mycontainer1
            /Dest:https://destaccount.blob.core.windows.net/mycontainer2
            /SourceKey:key1 /DestKey:key2 /S

    - `/Source`：提供來源儲存體帳戶的 URI (視容器而定)。  
    - `/Dest`：提供目標儲存體帳戶的 URI (視容器而定)。  
    - `/SourceKey`：提供來源儲存體帳戶的主索引鍵。 您可以透過選取儲存體帳戶，從 Azure 入口網站複製這個索引鍵。  
    - `/DestKey`：提供目標儲存體帳戶的主索引鍵。 您可以透過選取儲存體帳戶，從入口網站複製這個索引鍵。

執行此命令之後，容器檔案會移到目標儲存體帳戶。

> [!NOTE]
> 從某個 Azure Blob 複製到另一個 Azure Blob 時，AzCopy CLI 不能與 **Pattern** 參數一起使用。
>
> 您可以直接複製和編輯 AzCopy 命令，並交叉檢查以確定 **Pattern** 符合來源。 另請確認 **/S** 萬用字元有效。 如需詳細資訊，請參閱 [AzCopy 參數](storage-use-azcopy.md)。

**如何在儲存體容器之間移動資料？**

請遵循下列步驟：

1.  在目的地 Blob 中建立容器 (資料夾)。

2.  請使用 [AzCopy](https://azure.microsoft.com/blog/azcopy-5-1-release/) 將原始 Blob 容器的內容複製到其他 Blob 容器。

**如何建立 PowerShell 指令碼，以將 Azure 檔案共用的資料移至 Azure 儲存體中的其他檔案共用？**

請使用 AzCopy 將 Azure 檔案共用的資料移至 Azure 儲存體中的其他檔案共用。 如需詳細資訊，請參閱[使用 AzCopy on Windows 傳送資料](storage-use-azcopy.md)和[使用 AzCopy on Linux 傳送資料](storage-use-azcopy-linux.md)。

**如何將大型 .csv 檔案上傳到 Azure 儲存體？**

請使用 AzCopy 將大型 .csv 檔案上傳到 Azure 儲存體。 如需詳細資訊，請參閱[使用 AzCopy on Windows 傳送資料](storage-use-azcopy.md)和[使用 AzCopy on Linux 傳送資料](storage-use-azcopy-linux.md)。

**我必須每天將記錄從磁碟機 D 移至 Azure 儲存體帳戶。如何自動進行此作業？**

您可以使用 AzCopy，並在工作排程器中建立工作。 請使用 AzCopy 批次指令碼，將檔案上傳到 Azure 儲存體帳戶。 如需詳細資訊，請參閱[如何設定和執行雲端服務的啟動工作](../../cloud-services/cloud-services-startup-tasks.md)。

**如何在訂用帳戶之間移動我的儲存體帳戶？**

請使用 AzCopy 在訂用帳戶之間移動您的儲存體帳戶。 如需詳細資訊，請參閱[使用 AzCopy on Windows 傳送資料](storage-use-azcopy.md)和[使用 AzCopy on Linux 傳送資料](storage-use-azcopy-linux.md)。

**如何將 10 TB 的資料移至其他區域的儲存體？**

請使用 AzCopy 移動資料。 如需詳細資訊，請參閱[使用 AzCopy on Windows 傳送資料](storage-use-azcopy.md)和[使用 AzCopy on Linux 傳送資料](storage-use-azcopy-linux.md)。

**如何將內部部署的資料複製到 Azure 儲存體？**

使用 AzCopy 以複製資料。 如需詳細資訊，請參閱[使用 AzCopy on Windows 傳送資料](storage-use-azcopy.md)和[使用 AzCopy on Linux 傳送資料](storage-use-azcopy-linux.md)。

**如何將內部部署的資料移至 Azure 檔案？**

使用 AzCopy 以移動資料。 如需詳細資訊，請參閱[使用 AzCopy on Windows 傳送資料](storage-use-azcopy.md)和[使用 AzCopy on Linux 傳送資料](storage-use-azcopy-linux.md)。

**如何在虛擬機器上對應容器資料夾？**

使用 Azure 檔案共用。

**如何備份 Azure 檔案儲存體？**

沒有任何備份解決方案。 但是 Azure 檔案也支援非同步複製。 因此，您可以將檔案：

- 從儲存體帳戶內的共用複製到其他共用，或複製到不同的儲存體帳戶。

- 從儲存體帳戶內的共用複製到 Blob 容器，或複製到不同的儲存體帳戶。

如需詳細資訊，請參閱[使用 AzCopy on Windows 傳送資料](storage-use-azcopy.md)。

**如何將受控磁碟移至另一個儲存體帳戶？**

請遵循下列步驟：

1.  停止受控磁碟附加至的虛擬機器。

2.  執行下列 Azure PowerShell 指令碼，將受控磁碟 VHD 從一個區域複製到另一個區域：

    ```
    Connect-AzureRmAccount

    Select-AzureRmSubscription -SubscriptionId <ID>

    $sas = Grant-AzureRmDiskAccess -ResourceGroupName <RG name> -DiskName <Disk name> -DurationInSecond 3600 -Access Read

    $destContext = New-AzureStorageContext –StorageAccountName contosostorageav1 -StorageAccountKey <your account key>

    Start-AzureStorageBlobCopy -AbsoluteUri $sas.AccessSAS -DestContainer 'vhds' -DestContext $destContext -DestBlob 'MyDestinationBlobName.vhd'
    ```

3.  請使用您將 VHD 複製過去的區域內的 VHD 檔案來建立受控的磁碟。 您可以執行下列 Azure PowerShell 指令碼來進行此動作：  

    ```
    $resourceGroupName = 'MDDemo'

    $diskName = 'contoso\_os\_disk1'

    $vhdUri = 'https://contoso.storageaccou.com.vhd

    $storageId = '/subscriptions/<ID>/resourceGroups/<RG name>/providers/Microsoft.Storage/storageAccounts/contosostorageaccount1'

    $location = 'westus'

    $storageType = 'StandardLRS'

    $diskConfig = New-AzureRmDiskConfig -AccountType $storageType -Location $location -CreateOption Import -SourceUri $vhdUri -StorageAccountId $storageId -DiskSizeGB 128

    $osDisk = New-AzureRmDisk -DiskName $diskName -Disk $diskConfig -ResourceGroupName $resourceGroupName
    ``` 

如需進一步瞭解如何部署受控磁碟中的虛擬機器，請參閱 [CreateVmFromManagedOsDisk.ps1](https://github.com/Azure-Samples/managed-disks-powershell-getting-started/blob/master/CreateVmFromManagedOsDisk.ps1)。

**如何從 Azure 入口網站下載 1-2 TB 的資料？**

請使用 AzCopy 下載資料。 如需詳細資訊，請參閱[使用 AzCopy on Windows 傳送資料](storage-use-azcopy.md)和[使用 AzCopy on Linux 傳送資料](storage-use-azcopy-linux.md)。

**如何將儲存體帳戶的次要位置變更為歐洲地區？**

建立儲存體帳戶時，您可以為帳戶選取主要區域。 次要地區的選擇是以主要地區為準，無法變更。 如需詳細資訊，請參閱[異地備援儲存體 (GRS)：適用於 Azure 儲存體的跨區域複寫](storage-redundancy.md)。

**我可以從哪裡取得更多關於 Azure 儲存體服務加密 (SSE) 的資訊？**  
  
請參閱下列文章：

-  [Azure 儲存體安全性指南](storage-security-guide.md)

-  [待用資料的 Azure 儲存體服務加密](storage-service-encryption.md)

**如何移動資料或從儲存體帳戶下載資料？**

請使用 AzCopy 下載資料。 如需詳細資訊，請參閱[使用 AzCopy on Windows 傳送資料](storage-use-azcopy.md)和[使用 AzCopy on Linux 傳送資料](storage-use-azcopy-linux.md)。


**如何加密儲存體帳戶的資料？**

在儲存體帳戶中啟用加密之後，現有的資料並未加密。 若要加密現有資料，您必須將它再次上傳至儲存體帳戶。

請使用 AzCopy，將資料複製到不同的儲存體帳戶，再將資料移回。 您也可以使用[待用加密](storage-service-encryption.md)。

**如何不使用入口網站的下載選項，將 VHD 下載到本機電腦？**

可以使用[儲存體總管](https://azure.microsoft.com/features/storage-explorer/)下載 VHD。

**將儲存體帳戶的複寫從異地備援儲存體變更為本地備援儲存體是否有任何必要條件？**

否。 

**如何存取 Azure 檔案備援儲存體？**

需有讀取權限異地備援儲存體，才能存取備援儲存體。 不過，Azure 檔案只支援本機備援儲存體和不允許唯讀權限的異地備援儲存體。 

**如何從進階儲存體移至標準儲存體帳戶？**

請遵循下列步驟：

1.  建立標準儲存體帳戶。 (或使用訂用帳戶中的現有標準儲存體帳戶。)

2.  下載 AzCopy。 執行下列其中一個 AzCopy 命令。
      
    若要複製儲存體帳戶中的整個磁碟：

        AzCopy /Source:https://sourceaccount.blob.core.windows.net/mycontainer1
        /Dest:https://destaccount.blob.core.windows.net/mycontainer2
        /SourceKey:key1 /DestKey:key2 /S 

    若只要複製一個磁碟，請在 **Pattern**  中提供磁碟的名稱：

        AzCopy /Source:https://sourceaccount.blob.core.windows.net/mycontainer1
        /Dest:https://destaccount.blob.core.windows.net/mycontainer2
        /SourceKey:key1 /DestKey:key2 /Pattern:abc.vhd

   
此作業可能需花費數小時才能完成。

若要確定是否順利完成傳輸，可以至 Azure 入口網站檢查目的地儲存體帳戶容器。 當磁碟複製到標準儲存體帳戶之後，您可以將其附加到虛擬機器作為現有磁碟使用。 如需詳細資訊，請參閱[如何在 Azure 入口網站中將受控資料磁碟連結至 Windows 虛擬機器](../../virtual-machines/windows/attach-managed-disk-portal.md)。  
  
**如何將檔案共用轉換至 Azure 進階儲存體？**

Azure 檔案共用不允許使用進階儲存體。

**如何從標準儲存體帳戶升級至進階儲存體帳戶？如何從進階儲存體帳戶降級至標準儲存體帳戶？**

您必須建立目的地儲存體帳戶，將資料從來源帳戶複製到目的地帳戶，然後刪除來源帳戶。 您可以使用 AzCopy 之類的工具來複製資料。

如果您有虛擬機器，則必須先執行額外的步驟，才能移轉儲存體帳戶資料。 如需詳細資訊，請參閱[移轉至 Azure 進階儲存體 (非受控磁碟)](storage-migration-to-premium-storage.md)。

**如何從傳統儲存體帳戶移至 Azure Resource Manager 儲存體帳戶？**

您可以使用 **Move-AzureStorageAccount** Cmdlet。 此 Cmdlet 具有多個步驟 (驗證、準備、認可)。 您可以在進行之前先驗證移動。

如果您有虛擬機器，則必須先執行額外的步驟，才能移轉儲存體帳戶資料。 如需詳細資訊，請參閱[使用 Azure PowerShell 將 IaaS 資源從傳統移轉至 Azure Resource Manager](../..//virtual-machines/windows/migration-classic-resource-manager-ps.md)。

**如何從 Azure 儲存體帳戶下載資料至 Linux 電腦，或從 Linux 電腦上傳資料？**

您可以使用 Azure CLI。

- 下載單一 Blob：

      azure storage blob download -k "<Account Key>" -a "<Storage Account Name>" --container "<Blob Container Name>" -b "<Remote File Name>" -d "<Local path where the file will be downloaded to>"

- 上傳單一 Blob： 

      azure storage blob upload -k "<Account Key>" -a "<Storage Account Name>" --container "<Blob Container Name>" -f "<Local File Name>"

**如何讓其他人存取我的儲存體資源？**

若要讓其他人存取儲存體資源：

-   請使用共用存取簽章 (SAS) 權杖來提供資源的存取權。 

-   將儲存體帳戶的主要或次要金鑰提供給使用者。 如需詳細資訊，請參閱[管理您的儲存體帳戶](storage-account-manage.md#access-keys)。

-   變更存取原則，以允許匿名存取。 如需詳細資訊，請參閱[授與容器和 Blob 的匿名使用者權限](../blobs/storage-manage-access-to-resources.md#grant-anonymous-users-permissions-to-containers-and-blobs)。

**AzCopy 的安裝位置？**

-   如果您是從 Microsoft Azure 儲存體命令列存取 AzCopy，請輸入 **AzCopy**。 命令列是與 AzCopy 一同安裝的。

-   如果您安裝的是 32 位元版本，則位於：**%ProgramFiles(x86)%\\Microsoft SDKs\\Azure\\AzCopy**。

-   如果您安裝的是 64 位元版本，則位於：**%ProgramFiles%\\Microsoft SDKs\\Azure\\AzCopy**。

**對於複寫的儲存體帳戶 (例如，區域備援儲存體、異地備援儲存體或讀取權限異地備援儲存體)，如何存取儲存在次要區域的資料？**

-   如果您使用的是區域備援儲存體或異地備援儲存體，除非發生容錯移轉，否則無法從次要區域存取資料。 如需更多容錯移轉程序的相關資訊，請參閱[如果 Azure 儲存體發生中斷怎麼辦](storage-disaster-recovery-guidance.md#what-to-expect-if-a-storage-failover-occurs)。

-   如果您使用的是讀取權限異地備援儲存體，可以隨時存取次要區域中的資料。 使用下列其中一種方法：  
      
    - **AzCopy**：將 **-secondary** 附加至 URL 中的儲存體帳戶名稱，即可存取次要端點。 例如︰  
     
      https://storageaccountname-secondary.blob.core.windows.net/vhds/BlobName.vhd

    - **SAS 權杖**︰使用 SAS 權杖以存取來自端點的資料。 如需詳細資訊，請參閱[使用共用存取簽章](storage-dotnet-shared-access-signature-part-1.md)。

**如何將 HTTPS 自訂網域與我的儲存體帳戶搭配使用？例如，如何讓 "https://mystorageaccountname.blob.core.windows.net/images/image.gif" 顯示為 "https://www.contoso.com/images/image.gif"？**

具有自訂網域的儲存體帳戶目前不支援 SSL。
但可以使用非 HTTPS 的自訂網域。 如需詳細資訊，請參閱[針對 Blob 儲存體端點設定自訂網域名稱](../blobs/storage-custom-domain-name.md)。

**如何使用 FTP 存取儲存體帳戶中的資料？**

無法使用 FTP 直接存取儲存體帳戶。 不過可以安裝一個 Azure 虛擬機器，然後將 FTP 伺服器安裝在虛擬機器上。 您可以讓 FTP 伺服器將檔案儲存在 Azure 檔案共用中，或是儲存在虛擬機器可用的資料磁碟裡。

如果您只是要下載資料，而不想使用儲存體總管或類似的應用程式，則可以使用 SAS 權杖。 如需詳細資訊，請參閱[使用共用存取簽章](storage-dotnet-shared-access-signature-part-1.md)。

**如何在儲存體帳戶之間移轉 Blob？**

 您可以使用 [Blob 移轉指令碼](../scripts/storage-common-transfer-between-storage-accounts.md)來達成。

## <a name="need-help-contact-support"></a>需要協助嗎？ 請連絡支援人員。

如果仍需要協助，請[連絡支援人員](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)以快速解決您的問題。
