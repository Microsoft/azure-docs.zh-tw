---
title: 使用 .NET SDK 執行 Microsoft Azure StorSimple Data Manager 作業 | Microsoft Docs
description: 了解如何使用 .NET SDK 啟動 StorSimple Data Manager 作業
services: storsimple
documentationcenter: NA
author: alkohli
manager: jeconnoc
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 01/16/2018
ms.author: alkohli
ms.openlocfilehash: d15a5cbda2f0c2a363b40e94c38fed6631aa81b5
ms.sourcegitcommit: 2a70752d0987585d480f374c3e2dba0cd5097880
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/19/2018
ms.locfileid: "27928231"
---
# <a name="use-the-net-sdk-to-initiate-data-transformation"></a>使用 .Net SDK 起始資料轉換

## <a name="overview"></a>概觀

本文說明如何使用 StorSimple Data Manager 服務中的資料轉換功能來轉換 StorSimple 裝置資料。 轉換後的資料由雲端中的其他 Azure 服務取用。

您可以使用下列兩種方式啟動資料轉換作業：

 - 使用 .NET SDK
 - 使用 Azure 自動化 Runbook
 
 本文詳細說明如何建立範例 .NET 主控台應用程式來起始資料轉換作業，然後追蹤直到完成為止。 若要深入了解如何透過自動化起始資料轉換，請前往[使用 Azure 自動化 Runbook 來觸發資料轉換作業](storsimple-data-manager-job-using-automation.md)。

## <a name="prerequisites"></a>先決條件

開始之前，請確定您有︰
*   執行以下軟體的電腦：

    - Visual Studio 2012、2013、2015 或 2017。

    - Azure Powershell。 [下載 Azure Powershell](https://azure.microsoft.com/documentation/articles/powershell-install-configure/)。
*   資源群組內 StorSimple 資料管理員中正確設定的作業定義。
*   所有必要的 dll。 從 [GitHub 儲存機制](https://github.com/Azure-Samples/storsimple-dotnet-data-manager-get-started/tree/master/Data_Manager_Job_Run/dlls)下載這些 dll。
*   GitHub 存放庫中的 [`Get-ConfigurationParams.ps1`](https://github.com/Azure-Samples/storsimple-dotnet-data-manager-get-started/blob/master/Data_Manager_Job_Run/Get-ConfigurationParams.ps1) 指令碼。

## <a name="step-by-step-procedure"></a>逐步程序

執行下列步驟，以使用 .NET 啟動資料轉換作業。

1. 若要擷取設定參數，請執行下列步驟︰
    1. 將 GitHub 存放庫中的 `Get-ConfigurationParams.ps1` 指令碼下載到 `C:\DataTransformation` 位置。
    1. 執行 GitHub 存放庫中的 `Get-ConfigurationParams.ps1` 指令碼。 輸入以下命令：

        ```
        C:\DataTransformation\Get-ConfigurationParams.ps1 -SubscriptionName "AzureSubscriptionName" -ActiveDirectoryKey "AnyRandomPassword" -AppName "ApplicationName"
         ```
        您可以傳入任何值給 ActiveDirectoryKey 和 AppName。

2. 此指令碼會輸出下列值：
    * 用戶端識別碼
    * 租用戶識別碼
    * Active Directory 金鑰 (與上面輸入的金鑰相同)
    * 訂用帳戶識別碼

        ![設定參數指令碼輸出](media/storsimple-data-manager-dotnet-jobs/get-config-parameters.png)

3. 使用 Visual Studio 2012、2013 或 2015 建立 C# .NET 主控台應用程式。

    1. 啟動 **Visual Studio 2012/2013/2015**。
    1. 選取 [檔案] > [新增] > [專案]。

        ![建立專案 1](media/storsimple-data-manager-dotnet-jobs/create-new-project-7.png)        
    2. 選取 [已安裝] > [範本] > [Visual C#] > [主控台應用程式]。
    3. 輸入 **DataTransformationApp** 做為 [名稱]。
    4. 選取 **C:\DataTransformation** 做為 [位置]。
    6. 按一下 [確定]  以建立專案。

        ![建立專案 2](media/storsimple-data-manager-dotnet-jobs/create-new-project-1.png)

4.  現在，在您建立的專案中，將 [dlls](https://github.com/Azure-Samples/storsimple-dotnet-data-manager-get-started/tree/master/Data_Manager_Job_Run/dlls) 資料夾中出現的所有 dll 新增為 [參考]。 若要新增 dll 檔，請執行下列作業：

    1. 在 Visual Studio 中，移至 [檢視] > [方案總管]。
    2. 按一下資料轉換應用程式專案左邊的箭號。 按一下 [參考]，然後按一下滑鼠右鍵以 [新增參考]。
    
        ![新增 dll 1](media/storsimple-data-manager-dotnet-jobs/create-new-project-4.png)

    3. 瀏覽至套件資料夾的位置，選取所有 dll，按一下 [新增]，然後按一下 [確定]。

        ![新增 dll 2](media/storsimple-data-manager-dotnet-jobs/create-new-project-6.png)

5. 將下列 **using** 陳述式加入專案的原始程式檔 (Program.cs) 中。

    ```
    using System;
    using System.Collections.Generic;
    using System.Threading;
    using Microsoft.Azure.Management.HybridData.Models;
    using Microsoft.Internal.Dms.DmsWebJob;
    using Microsoft.Internal.Dms.DmsWebJob.Contracts;
    ```
    
6. 下列程式碼會初始化資料轉換作業執行個體。 請將這段程式碼新增至 **Main 方法** 中。 取代先前取得的設定參數值。 插入 **Resource Group Name** 和 **ResourceName** 的值。 **ResourceGroupName** 與在其上設定作業定義的 StorSimple 資料管理員相關聯。 **ResourceName** 是 StorSimple 資料管理員服務的名稱。

    ```
    // Setup the configuration parameters.
    var configParams = new ConfigurationParams
    {
        ClientId = "client-id",
        TenantId = "tenant-id",
        ActiveDirectoryKey = "active-directory-key",
        SubscriptionId = "subscription-id",
        ResourceGroupName = "resource-group-name",
        ResourceName = "resource-name"
    };

    // Initialize the Data Transformation Job instance.
    DataTransformationJob dataTransformationJob = new DataTransformationJob(configParams);
    ```
   
7. 指定執行作業定義所需的參數

    ```
    string jobDefinitionName = "job-definition-name";

    DataTransformationInput dataTransformationInput = dataTransformationJob.GetJobDefinitionParameters(jobDefinitionName);
    ```

    (或)

    如果您想要在執行階段變更作業定義參數，請新增下列程式碼︰

    ```
    string jobDefinitionName = "job-definition-name";
    // Must start with a '\'
    var rootDirectories = new List<string> {@"\root"};

    // Name of the volume on the StorSimple device.
    var volumeNames = new List<string> {"volume-name"};

    var dataTransformationInput = new DataTransformationInput
    {
        // If you require the latest existing backup to be picked else use TakeNow to trigger a new backup.
        BackupChoice = BackupChoice.UseExistingLatest.ToString(),
        // Name of the StorSimple device.
        DeviceName = "device-name",
        // Name of the container in Azure storage where the files will be placed after execution.
        ContainerName = "container-name",
        // File name filter (search pattern) to be applied on files under the root directory. * - Match all files.
        FileNameFilter = "*",
        // List of root directories.
        RootDirectories = rootDirectories,
        // Name of the volume on StorSimple device on which the relevant data is present. 
        VolumeNames = volumeNames
    };
    ```

8. 初始化之後，在作業定義上新增下列程式碼來觸發資料轉換作業。 插入適當的**作業定義名稱**。

    ```
    // Trigger a job, retrieve the jobId and the retry interval for polling.
    int retryAfter;
    string jobId = dataTransformationJob.RunJobAsync(jobDefinitionName, 
    dataTransformationInput, out retryAfter);
    Console.WriteLine("jobid: ", jobId);
    Console.ReadLine();

    ```
    貼上程式碼之後，就會開始建置解決方案。 以下是用來初始化資料轉換作業執行個體之程式碼片段的螢幕擷取畫面。

   ![用來初始化資料轉換作業的程式碼片段](media/storsimple-data-manager-dotnet-jobs/start-dotnet-job-code-snippet-1.png)

9. 這項作業會轉換 StorSimple 磁碟區內符合根目錄和檔案篩選條件的資料，並將它放入指定的容器/檔案共用。 轉換檔案時，儲存體佇列 (位於與容器/檔案共用相同的儲存體帳戶) 中會新增一則與作業定義同名的訊息。 此訊息可做為觸發程序，以起始檔案的任何進一步處理。

10. 觸發作業後，您可以使用下列程式碼來追蹤作業是否完成。 不一定要新增此程式碼讓作業執行。

    ```
    Job jobDetails = null;

    // Poll the job.
    do
    {
        jobDetails = dataTransformationJob.GetJob(jobDefinitionName, jobId);

        // Wait before polling for the status again.
        Thread.Sleep(TimeSpan.FromSeconds(retryAfter));

    } while (jobDetails.Status == JobStatus.InProgress);

    // Completion status of the job.
    Console.WriteLine("JobStatus: {0}", jobDetails.Status);
    
    // To hold the console before exiting.
    Console.Read();

    ```
 以下是用來觸發使用 .NET 之作業的完整程式碼範例螢幕擷取畫面。

 ![用來觸發 .NET 作業的完整程式碼片段](media/storsimple-data-manager-dotnet-jobs/start-dotnet-job-code-snippet.png)

## <a name="next-steps"></a>後續步驟

[使用 StorSimple Data Manager UI 來轉換資料](storsimple-data-manager-ui.md)。
