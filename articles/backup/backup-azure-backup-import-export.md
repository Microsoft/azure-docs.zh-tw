---
title: Azure 備份 - 離線備份或使用 Azure 匯入/匯出服務進行初始植入
description: 了解 Azure 備份如何讓您使用 Azure 匯入/匯出服務在網路上傳送資料。 此文章說明如何使用 Azure 匯入/匯出服務離線植入初始備份資料。
services: backup
author: saurabhsensharma
manager: shivamg
ms.service: backup
ms.topic: conceptual
ms.date: 05/17/2018
ms.author: saurse
ms.openlocfilehash: 5ef44ccf87bc5e40b57dc7fc997c9a827c93484b
ms.sourcegitcommit: 944d16bc74de29fb2643b0576a20cbd7e437cef2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/07/2018
ms.locfileid: "34831445"
---
# <a name="offline-backup-workflow-in-azure-backup"></a>在 Azure 備份中離線備份工作流程
Azure 備份有數個可提升效率的內建功能，能在資料初始完整備份至 Azure 的期間節省網路和儲存體成本。 初始完整備份通常會傳輸大量資料，且需要較多網路頻寬，相較之下，後續備份只會傳輸差異/增量部分。 透過離線植入程序，Azure 備份可以使用磁碟將離線備份資料上傳至 Azure。

Azure 備份的離線植入程序與 [Azure 匯入/匯出服務](../storage/common/storage-import-export-service.md)緊密整合，可讓您使用磁碟將初始備份資料傳輸到 Azure。 如果您的初始備份資料大小達到 TB，且需要透過高延遲和低頻寬網路傳輸時，您可以使用離線植入工作流程將初始備份複本送至 Azure 資料中心的一個或多個硬碟上。 下圖提供工作流程中的步驟概觀。

  ![離線匯入工作流程程序的概觀](./media/backup-azure-backup-import-export/offlinebackupworkflowoverview.png)

離線備份程序涉及下列步驟：

1. 將備份資料寫入到暫存位置，而不是透過網路傳送備份資料。
2. 使用 AzureOfflineBackupDiskPrep 公用程式，將暫存位置的資料寫入到一或多個 SATA 磁碟。
3. 在準備工作中，AzureOfflineBackupDiskPrep 公用程式會建立 Azure 匯入工作。 將 SATA 磁碟機傳送至最接近的 Azure 資料中心，並參考匯入作業以連線活動。
4. 在 Azure 資料中心，磁碟上的資料會複製到 Azure 儲存體帳戶。
5. Azure 備份會將儲存體帳戶中的資料複製到「復原服務」保存庫，而且會排定增量備份。

## <a name="supported-configurations"></a>支援的設定 
下列 Azure 備份功能或工作負載支援使用離線備份。

> [!div class="checklist"]
> * 使用 Microsoft Azure 復原服務 (MARS) 代理程式 (也稱為 Azure 備份代理程式) 來備份檔案和資料夾。 
> * 使用 System Center Data Protection Manager (SC DPM) 來備份所有工作負載和檔案 
> * 使用「Microsoft Azure 備份伺服器」來備份所有工作負載和檔案 <br/>

   > [!NOTE]
   > 針對使用「Azure 備份」代理程式來執行的「系統狀態」備份，不支援「離線備份」。 

[!INCLUDE [backup-upgrade-mars-agent.md](../../includes/backup-upgrade-mars-agent.md)]

## <a name="prerequisites"></a>先決條件

  > [!NOTE]
  > 下列必要條件和工作流程僅適用於使用[最新 MARS 代理程式](https://aka.ms/azurebackup_agent)進行檔案和資料夾的離線備份。 若要使用 System Center DPM 或 Azure 備份伺服器執行工作負載離線備份，請參閱[這篇文章](backup-azure-backup-server-import-export-.md)。 

起始離線備份工作流程之前，請先完成下列必要條件： 
* 建立[復原服務保存庫](backup-azure-recovery-services-vault-overview.md)。 若要建立保存庫，請參閱[這篇文章](tutorial-backup-windows-server-to-azure.md#create-a-recovery-services-vault)中的步驟
* 確定只在 Windows Server/Windows 用戶端 (依據適用情況) 上安裝[最新版的 Azure 備份代理程式](https://aka.ms/azurebackup_agent)，並已向復原服務保存庫註冊該電腦。
* 在執行 Azure 備份代理程式的電腦上，需要有 Azure PowerShell 3.7.0。 建議您下載並[安裝 3.7.0 版的 Azure PowerShell](https://github.com/Azure/azure-powershell/releases/tag/v3.7.0-March2017) \(英文\)。
* 在執行 Azure 備份代理程式的電腦上，確定已安裝 Microsoft Edge 或 Internet Explorer 11，而且已啟用 JavaScript。 
* 在與復原服務保存庫相同的訂用帳戶中建立儲存體帳戶。 
* 確定您有建立 Azure Active Directory 應用程式[所需的權限](../azure-resource-manager/resource-group-create-service-principal-portal.md)。 離線備份工作流程會在與 Azure 儲存體帳戶相關聯的訂用帳戶中建立 Azure Active Directory 應用程式。 應用程式的目標是為 Azure 備份提供安全的限域存取，以便存取離線備份工作流程所需 Azure 匯入服務。 
* 向包含 Azure 儲存體帳戶的訂用帳戶註冊 Microsoft.ImportExport 資源提供者。 若要註冊資源提供者：
    1. 在主功能表中，按一下 [訂用帳戶]。
    2. 如果您訂閱多個訂用帳戶，請選取您用於離線備份的訂用帳戶。 如果您只使用一個訂用帳戶，則您的訂用帳戶隨即出現。
    3. 在訂用帳戶功能表中，按一下 [資源提供者] 以檢視提供者的清單。
    4. 在提供者清單中，向下捲動至 Microsoft.ImportExport。 如果 [狀態] 為 NotRegistered，請按一下 [註冊]。
    ![註冊資源提供者](./media/backup-azure-backup-import-export/registerimportexport.png)
* 已建立具有足夠磁碟空間來存放初始複本的內部或外部暫存位置 (可能是網路共用或電腦上任何額外的磁碟機)。 例如：若您正在嘗試備份 500 GB 的檔案伺服器，請確定預備區域至少有 500 GB 的空間  (由於壓縮的關係，實際使用量會較少)。
* 將磁碟傳送到 Azure 時，僅使用 2.5 英吋的 SSD，或是 2.5 英吋或 3.5 英吋的 SATA II/III 內部硬碟。 您可以使用高達 10 TB 的硬碟。 檢查 [Azure 匯入/匯出服務文件](../storage/common/storage-import-export-requirements.md#supported-hardware)以取得服務所支援的最新磁碟機組合。
* SATA 磁碟機必須連接到一個電腦 (又稱為「複本電腦」)，在其中可將備份資料從「暫存位置」複製到 SATA 磁碟機。 請確定已在「複本電腦」上啟用 Bitlocker。

## <a name="workflow"></a>工作流程
本節描述離線備份工作流程，以便將您的資料傳遞至 Azure 資料中心並上傳至 Azure 儲存體。 若您有關於匯入服務或處理程序任何層面的問題，請參閱稍早的[匯入服務概觀](../storage/common/storage-import-export-service.md)。

## <a name="initiate-offline-backup"></a>起始離線備份
1. 在 MARS 代理程式上排程備份時，您會看到下列畫面。

    ![匯入畫面](./media/backup-azure-backup-import-export/offlinebackup_inputs.png)

  輸入的說明如下：

    * **預備位置**：初始備份所寫入的暫時儲存體位置。 暫存位置可能位於網路共用或本機電腦上。 如果複本電腦和來源電腦不同，則建議您指定預備位置的完整網路路徑。
    * **Azure Resource Manager 儲存體帳戶**：任何 Azure 訂用帳戶中 Resource Manager 類型儲存體帳戶的名稱。
    * **Azure 儲存體容器**：在複製到復原服務保存庫之前，Azure 儲存體帳戶中備份資料的匯入目的地儲存體 Blob 名稱。
    * **Azure 訂用帳戶識別碼**：Azure 儲存體帳戶建立所在的 Azure 訂用帳戶識別碼。
    * **Azure 匯入作業名稱**：Azure 匯入服務和 Azure 備份在追蹤磁碟上傳送至 Azure 之資料的傳輸活動時所使用的唯一名稱。 
  
  在畫面上提供輸入，然後按 [下一步]。 儲存所提供的「暫存位置」和「Azure 匯入作業名稱」 ，因為這是準備磁碟時所需的資訊。

2. 出現系統提示時，登入您的 Azure 訂用帳戶。 您必須先登入，Azure 備份才能建立 Azure Active Directory 應用程式，並提供必要的權限來存取 Azure 匯入服務。

    ![立即備份](./media/backup-azure-backup-import-export/azurelogin.png)

3. 完成工作流程，然後在 Azure 備份代理程式管理主控台中，按一下 [立即備份]。

    ![立即備份](./media/backup-azure-backup-import-export/backupnow.png)

4. 在精靈的 [確認] 頁面中，按一下 [備份]。 在設定過程中，初始備份會寫入暫存區域。

   ![確認您已準備好立即備份](./media/backup-azure-backup-import-export/backupnow-confirmation.png)

    作業完成之後，預備位置便已就緒可供用於準備磁碟。

   ![立即備份](./media/backup-azure-backup-import-export/opbackupnow.png)

## <a name="prepare-sata-drives-and-ship-to-azure"></a>準備 SATA 磁碟機並寄送到 Azure
*AzureOfflineBackupDiskPrep* 公用程式會準備要送到最鄰近之 Azure 資料中心的 SATA 磁碟機。 此公用程式位於 Azure 備份代理程式的安裝目錄 (以下路徑中)：

   *\Microsoft Azure Recovery Services Agent\Utils\\*

1. 移至該目錄，然後將 **AzureOfflineBackupDiskPrep** 目錄複製到已連接 SATA 磁碟機的另一部電腦。 在已連接 SATA 磁碟機的電腦上，請確定：

    * 複本電腦可使用在 **起始離線備份** 工作流程中所提供的相同網路路徑，存取離線植入工作流程的預備位置。
    * 已在複本電腦上啟用 BitLocker。
    * 已安裝 Azure PowerShell 3.7.0。
    * 已安裝最新相容的瀏覽器 (Edge 或 Internet Explorer 11) 並已啟用 JavaScript。 
    * 複本電腦可以存取 Azure 入口網站。 必要時，複本電腦可以與來源電腦相同。
    
    > [!IMPORTANT] 
    > 如果來源電腦是虛擬機器，則複本電腦必須是與來源電腦不同的實體伺服器或用戶端電腦。

2. 在複本電腦上以 *AzureOfflineBackupDiskPrep* 公用程式目錄作為目前的目錄來開啟已提高權限的命令提示字元，然後執行下列命令：

    ```.\AzureOfflineBackupDiskPrep.exe s:<Staging Location Path>```

    | 參數 | 說明 |
    | --- | --- |
    | s:&lt;*預備位置路徑*&gt; |強制性輸入用來提供在**起始離線備份**工作流程中所輸入的暫存位置路徑。 |
    | p:&lt;*PublishSettingsFile 的路徑*&gt; |選擇性輸入內容，用來提供在**起始離線備份**工作流程中所輸入的 **Azure 發佈設定**檔案路徑。 |

    執行命令時，此公用程式會要求選取與所需準備之磁碟機對應的 Azure 匯入作業。 如果只有一個與提供的預備位置相關聯的匯入作業，您會看到類似下面的畫面。

    ![Azure 磁碟準備工具的輸入](./media/backup-azure-backup-import-export/diskprepconsole0_1.png) <br/>

3. 輸入想要準備傳輸到 Azure 之掛接磁碟的磁碟機代號 (不含結尾冒號)。 
4. 在出現提示時確認您要格式化磁碟機。
5. 系統會提示您登入您的 Azure 訂用帳戶。 提供您的認證。

    ![Azure 磁碟準備工具的輸入](./media/backup-azure-backup-import-export/signindiskprep.png) <br/>

    工具接著便會開始準備磁碟並複製備份資料。 當工具顯示提示時，您可能需要連接額外的磁碟，以免所提供的磁碟沒有足夠空間來容納備份資料。 <br/>

    在工具順利執行結束時，命令提示字元會提供三項資訊：
    1. 您所提供的一或多個磁碟已準備好寄送到 Azure。 
    2. 您會收到已建立匯入作業的確認。 匯入作業會使用您提供的名稱。
    3. 此工具會顯示 Azure 資料中心的寄送地址。

    ![Azure 磁碟準備完成](./media/backup-azure-backup-import-export/console2.png)<br/>

6. 在命令執行結束時，您可更新寄送資訊。

7. 將磁碟寄送到工具所提供的地址，並保留追蹤號碼以供日後參考。

   > [!IMPORTANT] 
   > 沒有任何兩個「Azure 匯入作業」可以有相同的追蹤號碼。 請確保將公用程式在單一 Azure 匯入作業下準備的磁碟機放在單一包裹中一起寄送，並且該包裹有單一的獨特追蹤號碼。 請勿將在個別 Azure 匯入作業中準備的磁碟機合併放在一個包裹中。



## <a name="update-shipping-details-on-the-azure-import-job"></a>更新 Azure 匯入作業的寄送詳細資料

下列程序會更新 Azure 匯入作業的寄送詳細資料。 此資訊包含以下詳細資料：
* 將磁碟遞送至 Azure 的運送業者名稱
* 您磁碟的退貨寄送詳細資料

1. 登入您的 Azure 訂用帳戶。
2. 在主功能表中，按一下 [所有服務]，然後在 [所有服務] 對話方塊中輸入匯入。 當您看到 [匯入/匯出作業] 時，按一下它。
    ![輸入寄送資訊](./media/backup-azure-backup-import-export/search-import-job.png)<br/>

    [匯入/匯出作業] 功能表隨即開啟，而所選訂用帳戶中的所有匯入/匯出作業清單隨即出現。 

3. 如果您有多個訂用帳戶，請務必選取用於匯入備份資料的訂用帳戶。 然後選取新建立的匯入作業，以開啟其詳細資料。

    ![檢閱寄送資訊](./media/backup-azure-backup-import-export/import-job-found.png)<br/>

4. 在匯入作業的 [設定] 功能表上，按一下 [管理寄送資訊] 並輸入退貨寄送詳細資料。

    ![儲存寄送資訊](./media/backup-azure-backup-import-export/shipping-info.png)<br/>

5. 當您向運送業者取得追蹤號碼時，按一下 Azure 匯入作業概觀頁面中的橫幅，並輸入下列詳細資料：

   > [!IMPORTANT] 
   > 確定運送業者資訊和追蹤號碼已在建立 Azure 匯入作業的兩週內更新。 若無法在兩週內確認這項資訊，可能會導致作業遭到刪除，而且不會處理磁碟機。

   ![儲存寄送資訊](./media/backup-azure-backup-import-export/joboverview.png)<br/>

   ![儲存寄送資訊](./media/backup-azure-backup-import-export/tracking-info.png)

### <a name="time-to-process-the-drives"></a>處理磁碟機所需的時間 
處理 Azure 匯入作業所需的時間量會因不同的因素而異，例如運送時間、作業類型、所要複製的資料類型和大小，以及所提供磁碟的大小。 Azure 匯入/匯出服務沒有 SLA，但在收到磁碟之後，服務會努力在 7 到 10 天內，完成將備份資料複製到 Azure 儲存體帳戶的作業。 

### <a name="monitoring-azure-import-job-status"></a>監視 Azure 匯入作業狀態
您可以從 Azure 入口網站監視匯入作業的狀態，請瀏覽至 [匯入/匯出作業] 頁面，然後選取您的作業。 如需匯入作業狀態的詳細資訊，請參閱[儲存體匯入匯出服務](../storage/common/storage-import-export-service.md)一文。

### <a name="complete-the-workflow"></a>完成工作流程
在匯入作業順利完成後，儲存體帳戶中就會有初始備份資料可供使用。 到了下一個排定的備份時，Azure 備份會從儲存體帳戶將資料內容複製到「復原服務」保存庫，如下所示： 

   ![將資料複製到復原服務保存庫](./media/backup-azure-backup-import-export/copyingfromstorageaccounttoazurebackup.png)<br/>

到了下一個排定的備份時，Azure 備份會執行增量備份。

### <a name="cleaning-up-resources"></a>清除資源
初始備份完成後，您就可以放心地刪除已匯入 Azure 儲存體容器的資料以及暫存位置中的備份資料。

## <a name="next-steps"></a>後續步驟
* 如有任何關於 Azure 匯入/匯出工作流程的問題，請參閱 [使用 Microsoft Azure 匯入/匯出服務將資料傳輸至 Blob 儲存體](../storage/common/storage-import-export-service.md)。
* 若您有關於工作流程的任何問題，請參閱 Azure 備份 [常見問題集](backup-azure-backup-faq.md) 的＜離線備份＞章節。
