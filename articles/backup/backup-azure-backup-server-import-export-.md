---
title: Azure 備份 - 適用於 DPM 和 Azure 備份伺服器的離線備份
description: 了解 Azure 備份如何讓您使用 Azure 匯入/匯出服務在網路上傳送資料。 此文章說明如何使用 Azure 匯入/匯出服務離線植入初始備份資料。
services: backup
author: saurabhsensharma
manager: shivamg
ms.service: backup
ms.topic: conceptual
ms.date: 5/8/2018
ms.author: saurse
ms.openlocfilehash: 1a0e196f4d96494aca1c19a7527ac7d81837fb5c
ms.sourcegitcommit: a1e1b5c15cfd7a38192d63ab8ee3c2c55a42f59c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/10/2018
ms.locfileid: "34606472"
---
# <a name="offline-backup-workflow-for-dpm-and-azure-backup-server"></a>適用於 DPM 和 Azure 備份伺服器的離線備份工作流程
Azure 備份有數個可提升效率的內建功能，能在資料初始完整備份至 Azure 的期間節省網路和儲存體成本。 初始完整備份通常會傳輸大量資料，且需要較多網路頻寬，相較之下，後續備份只會傳輸差異/增量部分。 Azure 備份會壓縮初始備份。 透過離線植入程序，Azure 備份可以使用磁碟將壓縮後的初始備份資料離線上傳至 Azure。

Azure 備份的離線植入程序與 [Azure 匯入/匯出服務](../storage/common/storage-import-export-service.md) 緊密整合，可讓您使用磁碟將資料傳輸到 Azure 之中。 如果您的初始備份資料大小達到 TB，且需要透過高延遲和低頻寬網路傳輸時，您可以使用離線植入工作流程將初始備份複本送至 Azure 資料中心的一個或多個硬碟上。 本文提供概觀並進一步詳述針對 System Center DPM 和 Azure 備份伺服器可完成此工作流程的步驟。

> [!NOTE]
> Microsoft Azure 復原服務 (MARS) 代理程式的離線備份程序與 System Center DPM 和 Azure 備份伺服器有所區別。 如需透過 MARS 代理程式使用離線備份的詳細資訊，請參閱[本文章](backup-azure-backup-import-export.md)。 針對使用「Azure 備份」代理程式來執行的「系統狀態」備份，不支援「離線備份」。 
>

## <a name="overview"></a>概觀
透過 Azure 備份的離線植入功能和 Azure 匯入/匯出，可以簡單地使用磁碟將資料離線上傳至 Azure。 「離線備份」程序涉及下列步驟：

> [!div class="checklist"]
> * 不透過網路傳送備份資料，而是將備份資料寫入到*暫存位置*中
> * 接著，使用 *AzureOfflineBackupDiskPrep* 公用程式將「暫存位置」上的資料寫入到一或多個 SATA 磁碟中
> * 公用程式會自動建立 Azure 匯入作業
> * 接著，將 SATA 磁碟機送到最鄰近的 Azure 資料中心
> * 將備份資料上傳至 Azure 的作業完成之後，Azure 備份就會將備份資料複製至備份保存庫，並排程增量備份。

## <a name="supported-configurations"></a>支援的設定 
針對所有以離線方式將資料從內部部署環境備份到 Microsoft Cloud 的「Azure 備份」部署模型，都支援「離線備份」。 這包括

> [!div class="checklist"]
> * 使用「Microsoft Azure 復原服務 (MARS) 代理程式」或「Azure 備份」代理程式來備份檔案和資料夾。 
> * 使用 System Center Data Protection Manager (SC DPM) 來備份所有工作負載和檔案 
> * 使用「Microsoft Azure 備份伺服器」來備份所有工作負載和檔案 <br/>

## <a name="prerequisites"></a>先決條件
起始「離線備份」工作流程之前，請先確定已符合下列先決條件
* 已建立[復原服務保存庫](backup-azure-recovery-services-vault-overview.md)。 若要建立保存庫，請參閱[這篇文章](tutorial-backup-windows-server-to-azure.md#create-a-recovery-services-vault)中的步驟
* 已在 Windows Server/Windows 用戶端 (依據適用情況) 上安裝「Azure 備份」代理程式、「Azure 備份伺服器」或 SC DPM，並已向「復原服務保存庫」註冊該電腦。 請確定只使用[最新版的 Azure 備份](https://go.microsoft.com/fwlink/?linkid=229525)。 
* [下載 Azure 發佈設定檔案](https://portal.azure.com/#blade/Microsoft_Azure_ClassicResources/PublishingProfileBlade)。 您從中下載發佈設定檔案的訂用帳戶可以與包含「復原服務保存庫」的訂用帳戶不同。 如果您的訂用帳戶位於主權 Azure 雲端，則請依適用情況使用下列連結來下載「Azure 發佈設定」檔案。

    | 主權雲端區域 | Azure 發佈設定檔案連結 |
    | --- | --- |
    | 美國 | [連結](https://portal.azure.us#blade/Microsoft_Azure_ClassicResources/PublishingProfileBlade) |
    | 中國 | [連結](https://portal.azure.us#blade/Microsoft_Azure_ClassicResources/PublishingProfileBlade) |

* 在您從中下載發佈設定檔案的訂用帳戶中，已建立採用「傳統」部署模型的「Azure 儲存體」帳戶，如下所示： 

 ![建立傳統儲存體帳戶](./media/backup-azure-backup-import-export/storageaccountclassiccreate.png)

* 已建立具有足夠磁碟空間來存放初始複本的內部或外部暫存位置 (可能是網路共用或電腦上任何額外的磁碟機)。 例如：若您正在嘗試備份 500 GB 的檔案伺服器，請確定預備區域至少有 500 GB 的空間  (由於壓縮的關係，實際使用量會較少)。
* 針對將送到 Azure 的磁碟，確保僅使用 2.5 英吋的 SSD，或是 2.5 英吋或 3.5 英吋的 SATA II/III 內部硬碟。 您可以使用高達 10 TB 的硬碟。 檢查 [Azure 匯入/匯出服務文件](../storage/common/storage-import-export-requirements.md#supported-hardware)以取得服務所支援的最新磁碟機組合。
* SATA 磁碟機必須連接至要執行將備份資料從「暫存位置」複製到 SATA 磁碟機之作業的電腦 (稱為「複本電腦」)。 請確定已在「複本電腦」上啟用 Bitlocker 

## <a name="workflow"></a>工作流程
本節資訊可協助您完成離線備份工作流程，以便將您的資料傳遞至 Azure 資料中心，並上傳至 Azure 儲存體。 若您有關於匯入服務或處理程序任何層面的問題，請參閱稍早的 [匯入服務概觀](../storage/common/storage-import-export-service.md) 參考文件。

### <a name="initiate-offline-backup"></a>起始離線備份
1. 當您排程備份時，您會看到下列畫面 (在 Windows Server、Windows 用戶端或 System Center Data Protection Manager 中)。

    ![匯入畫面](./media/backup-azure-backup-import-export/offlineBackupscreenInputs.png)

    以下是在 System Center Data Protection Manager 的對應畫面︰ <br/>
    ![SC DPM 和 Azure 備份伺服器匯入畫面](./media/backup-azure-backup-import-export/dpmoffline.png)

    輸入的說明如下：

    * **預備位置**：初始備份所寫入的暫時儲存體位置。 暫存位置可能位於網路共用或本機電腦上。 如果複本電腦和來源電腦不同，則建議您指定預備位置的完整網路路徑。
    * **Azure 匯入作業名稱**：Azure 匯入服務和 Azure 備份在追蹤磁碟上傳送至 Azure 之資料的傳輸活動時所使用的唯一名稱。
    * **Azure 發佈設定**：提供發佈設定檔案的本機路徑。
    * **Azure 訂用帳戶 ID**：您從中下載「Azure 發佈設定」檔案之訂用帳戶的 Azure 訂用帳戶 ID。 
    * **Azure 儲存體帳戶**：與「Azure 發佈設定」檔案關聯之 Azure 訂用帳戶中的儲存體帳戶名稱。
    * **Azure 儲存體容器**：Azure 儲存體帳戶中備份資料的匯入目的地儲存體 Blob 名稱。

     請儲存您所提供的「暫存位置」和「Azure 匯入作業名稱」 ，因為這是準備磁碟時所需的資訊。  
     
2. 完成工作流程，然後若要起始離線備份複製，請在「Azure 備份」代理程式管理主控台中按一下 [立即備份]。 作為此步驟的一部分，初始備份會寫入預備區域。

    ![立即備份](./media/backup-azure-backup-import-export/backupnow.png)

    若要在 System Center Data Protection Manager 或「Azure 備份」伺服器中完成對應的工作流程，請在 [保護群組] 上按一下滑鼠右鍵，然後選擇 [建立復原點] 選項。 您接著要選擇 [線上保護]  選項。

    ![SC DPM 和 Azure 備份伺服器立即備份](./media/backup-azure-backup-import-export/dpmbackupnow.png)

    作業完成之後，預備位置便已就緒可供用於準備磁碟。

    ![備份進度](./media/backup-azure-backup-import-export/opbackupnow.png)

### <a name="prepare-sata-drives-and-ship-to-azure"></a>準備 SATA 磁碟機並寄送到 Azure
*AzureOfflineBackupDiskPrep* 公用程式可用來準備要送到最鄰近之 Azure 資料中心的 SATA 磁碟機。 此公用程式位於「復原服務」代理程式之安裝目錄的以下路徑中：

   *\Microsoft* *Azure* *Recovery* *Services* \*Agent\Utils\*

1. 移至該目錄，然後將 [AzureOfflineBackupDiskPrep] 目錄複製到已連接要準備之 SATA 磁碟機的複本電腦。 確認下列與複本電腦有關的事項：

    * 複本電腦可使用在 **起始離線備份** 工作流程中所提供的相同網路路徑，存取離線植入工作流程的預備位置。
    * 已在複本電腦上啟用 BitLocker。
    * 複本電腦可以存取 Azure 入口網站。

    必要時，複本電腦可以與來源電腦相同。 
    
    > [!IMPORTANT] 
    > 如果來源電腦是虛擬機器，則必須使用不同的實體伺服器或用戶端電腦作為複本電腦。
    
    
2. 在複本電腦上以 *AzureOfflineBackupDiskPrep* 公用程式目錄作為目前的目錄來開啟已提高權限的命令提示字元，然後執行下列命令：

    `*.\AzureOfflineBackupDiskPrep.exe*   s:<*Staging Location Path*>   [p:<*Path to AzurePublishSettingsFile*>]`

    | 參數 | 說明 |
    | --- | --- |
    | s:&lt;*預備位置路徑*&gt; |強制性輸入內容，用來提供在 **起始離線備份** 工作流程中所輸入的預備位置路徑。 |
    | p:&lt;*PublishSettingsFile 的路徑*&gt; |選擇性輸入內容，用來提供在**起始離線備份**工作流程中所輸入的 **Azure 發佈設定**檔案路徑。 |

    > [!NOTE]
    > 複本電腦與來源電腦不同時，必須提供 &lt;AzurePublishSettingFile 的路徑&gt;值。
    >
    >

    執行命令時，此公用程式會要求選取與所需準備之磁碟機對應的 Azure 匯入作業。 如果只有一個與提供的預備位置相關聯的匯入作業，您會看到類似下面的畫面。

    ![Azure 磁碟準備工具的輸入](./media/backup-azure-backup-import-export/azureDiskPreparationToolDriveInput.png) <br/>
    
3. 輸入想要準備傳輸到 Azure 之掛接磁碟的磁碟機代號 (不含結尾冒號)。 在出現提示時確認您要格式化磁碟機。

    工具接著便會開始準備磁碟並複製備份資料。 當工具顯示提示時，您可能需要連接額外的磁碟，以免所提供的磁碟沒有足夠空間來容納備份資料。 <br/>

    在工具順利執行結束時，您所提供的一或多個磁碟便已準備好可以寄送到 Azure。 此外，會在 Azure 中以您在**起始離線備份**工作流程期間所提供的名稱建立匯入作業。 最後，工具會顯示磁碟所需之寄送目的地的 Azure 資料中心寄送地址。

    ![Azure 磁碟準備完成](./media/backup-azure-backup-import-export/azureDiskPreparationToolSuccess.png)<br/>
    
4. 在命令執行結束時，您還會看到可更新寄送資訊的選項，如下所示：

    ![更新寄送資訊選項](./media/backup-azure-backup-import-export/updateshippingutility.png)<br/>

5. 您可以立即輸入詳細資料。 工具會引導您完成這個涉及一系列輸入的程序。 不過，如果您沒有像是追蹤號碼或其他與寄送相關的詳細資料，則可以結束此工作階段。 本文稍後會提供更新寄送詳細資料的步驟。 

6. 將磁碟寄送到工具所提供的地址，並保留追蹤號碼以供日後參考。

   > [!IMPORTANT] 
   > 沒有任何兩個「Azure 匯入作業」可以有相同的追蹤號碼。 請確保將公用程式在單一「Azure 匯入作業」下準備的磁碟機放在單一包裹中一起寄送，並且該包裹有單一的獨特追蹤號碼。 請勿將在**不同**「Azure 匯入作業」過程中準備的磁碟機合併放在一個包裹中。 

5. 當您取得追蹤號碼資訊時，請移至正在等候完成「匯入作業」的來源電腦，然後在已提升權限的命令提示字元中，以 *AzureOfflineBackupDiskPrep* 公用程式目錄作為目前的目錄來執行下列命令： 

   `*.\AzureOfflineBackupDiskPrep.exe*  u:`

   您可以選擇從不同的電腦 (例如「複本電腦」)，以 *AzureOfflineBackupDiskPrep* 公用程式目錄作為目前的目錄來執行下列命令：
   
   `*.\AzureOfflineBackupDiskPrep.exe*  u:  s:<*Staging Location Path*>   p:<*Path to AzurePublishSettingsFile*>`

    | 參數 | 說明 |
    | --- | --- |
    | u: | 必要輸入項目，用來更新 Azure 匯入工作的寄送詳細資料 |
    | s:&lt;*預備位置路徑*&gt; | 命令不是在來源電腦上執行時，必須輸入。 用來提供在**起始離線備份**工作流程中所輸入的暫存位置路徑。 |
    | p:&lt;*PublishSettingsFile 的路徑*&gt; | 命令不是在來源電腦上執行時，必須輸入。 用來提供在**起始離線備份**工作流程中所輸入的 **Azure 發佈設定**檔案路徑。 |
    
    公用程式會自動偵測來源電腦所等候的「匯入作業」，或是當命令在不同的電腦上執行時，則會偵測與暫存位置關聯的匯入作業。 接著，它會提供可透過一系列輸入來更新寄送資訊的選項，如下所示： 
    
    ![輸入寄送資訊](./media/backup-azure-backup-import-export/shippinginputs.png)<br/>

6. 提供所有輸入資料之後，請仔細檢閱詳細資料，然後輸入 *yes* 來確認您所提供的寄送資訊。 

    ![檢閱寄送資訊](./media/backup-azure-backup-import-export/reviewshippinginformation.png)<br/>

7. 成功更新寄送資訊時，公用程式會提供儲存您所輸入之寄送詳細資料的本機位置，如下所示： 

    ![儲存寄送資訊](./media/backup-azure-backup-import-export/storingshippinginformation.png)<br/>

   > [!IMPORTANT] 
   > 請確保在使用 *AzureOfflineBackupDiskPrep* 公用程式提供寄送資訊的兩週內將磁碟機送達 Azure 資料中心。 如果無法做到，將導致無法處理磁碟機。  

完成上述步驟之後，Azure 資料中心便已準備就緒，可以接收磁碟機並進一步處理這些磁碟機，以將備份資料從磁碟機轉移至您所建立的傳統型 Azure 儲存體帳戶。 

### <a name="time-to-process-the-drives"></a>處理磁碟機所需的時間 
處理 Azure 匯入作業所需的時間量會因不同的因素而異，例如運送時間、作業類型、所要複製的資料類型和大小，以及所提供磁碟的大小。 Azure 匯入/匯出服務沒有 SLA，但在收到磁碟之後，服務會努力在 7 到 10 天內，完成將備份資料複製到 Azure 儲存體帳戶的作業。 下一節將詳細說明如何監視 Azure 匯入作業的狀態。 

### <a name="monitoring-azure-import-job-status"></a>監視 Azure 匯入作業狀態
當您的磁碟機在運送途中或在 Azure 資料中心等候複製到儲存體時，來源電腦上的「Azure 備份」代理程式、SC DPM 或「Azure 備份」伺服器主控台會針對您的已排定備份顯示下列作業狀態。 

  `Waiting for Azure Import Job to complete. Please check on Azure Management portal for more information on job status`

請依照下列步驟來檢查「匯入作業」狀態。 
1. 在來源電腦上開啟已提高權限的命令提示字元，然後執行下列命令：
    
     `AzureOfflineBackupDiskPrep.exe u:`
    
2.  輸出會顯示「匯入作業」的目前狀態，如下所示： 

    ![檢查匯入作業狀態](./media/backup-azure-backup-import-export/importjobstatusreporting.png)<br/>

如需有關各種 Azure 匯入作業狀態的詳細資訊，請參閱[這篇文章](../storage/common/storage-import-export-view-drive-status.md)

### <a name="complete-the-workflow"></a>完成工作流程
匯入作業完成後，儲存體帳戶中就會有初始備份資料可供使用。 到了下一個排定的備份時，「Azure 備份」會從儲存體帳戶將資料內容複製到「復原服務」保存庫，如下所示： 

   ![將資料複製到復原服務保存庫](./media/backup-azure-backup-import-export/copyingfromstorageaccounttoazurebackup.png)<br/>

到了下一個排定的備份時，「Azure 備份」會在初始備份複本上執行增量備份。

## <a name="next-steps"></a>後續步驟
* 如有任何關於 Azure 匯入/匯出工作流程的問題，請參閱 [使用 Microsoft Azure 匯入/匯出服務將資料傳輸至 Blob 儲存體](../storage/common/storage-import-export-service.md)。
* 若您有關於工作流程的任何問題，請參閱 Azure 備份 [常見問題集](backup-azure-backup-faq.md) 的＜離線備份＞章節。
