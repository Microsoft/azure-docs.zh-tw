---
title: 如何排程 Azure SSIS 整合執行階段 | Microsoft Docs
description: 本文說明如何使用 Azure 自動化及 Data Factory 來排程 Azure SSIS 整合執行階段的啟動和停止。
services: data-factory
documentationcenter: ''
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: ''
ms.devlang: powershell
ms.topic: conceptual
ms.date: 07/16/2018
author: swinarko
ms.author: sawinark
ms.reviewer: douglasl
manager: craigg
ms.openlocfilehash: f83715d2a382db271686210d9df285c255c09216
ms.sourcegitcommit: 7827d434ae8e904af9b573fb7c4f4799137f9d9b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39113972"
---
# <a name="how-to-start-and-stop-the-azure-ssis-integration-runtime-on-a-schedule"></a>如何排程 Azure SSIS 整合執行階段的啟動和停止
本文說明如何使用 Azure 自動化及 Azure Data Factory 來排程 Azure SSIS 整合執行階段 (IR) 的啟動和停止。 執行 Azure SSIS (SQL Server Integration Services) 整合執行階段 (IR) 會產生相關費用。 因此，您通常只應在需要於 Azure 中執行 SSIS 套件時才執行 IR，而在不需要時即應加以停止。 您可以使用 Data Factory UI 或 Azure PowerShell，[以手動方式啟動或停止 Azure SSIS IR](manage-azure-ssis-integration-runtime.md)。

例如，您可以使用 Webhook 建立 Web 活動至 Azure 自動化 PowerShell Runbook，並在它們之間鏈結 Execute SSIS 套件活動。 Web 活動可在您的套件執行前或後即時啟動和停止您的 Azure SSIS IR。 如需有關 Execute SSIS 套件活動的詳細資訊，請參閱[在 Azure Data Factory 中使用 SSIS 活動執行 SSIS 套件](how-to-invoke-ssis-package-ssis-activity.md)。

## <a name="overview-of-the-steps"></a>步驟概觀

以下是本文說明的概要步驟：

1. **建立並測試 Azure 自動化 Runbook。** 在此步驟中，您會使用指令碼建立啟動或停止 Azure SSIS IR 的 PowerShell Runbook。 然後，您會在「啟動」和「停止」的案例中測試 Runbook，並確認 IR 會啟動或停止。 
2. **為 Runbook 建立兩個排程。** 在第一個排程中，您會設定具有 START 作業的 Runbook。 在第二個排程中，則設定具有 STOP 作業的 Runbook。 對這兩個排程，您都會指定執行 Runbook 的頻率。 例如，您可以將第一個 Runbook 排程在每天上午 8 點執行，第二個則在每天晚上 11 點執行。 第一個 Runbook 執行時，即會啟動 Azure SSIS IR。 第二個 Runbook 執行時，則會停止 Azure SSIS IR。 
3. **為 Runbook 建立兩個 Webhook**，一個用於 START 作業，另一個則用於 STOP 作業。 您在 Data Factory 管線中設定 Web 活動時，可以使用這些 Webhook 的 URL。 
4. **建立 Data Factory 管線**。 您建立的管線包含下列三個活動。 第一個 **Web** 活動會叫用第一個 Webhook，以啟動 Azure SSIS IR。 **預存程序**活動會執行 SQL 指令碼以執行 SSIS 套件。 第二個 **Web** 活動會停止 Azure SSIS IR。 如需如何使用預存程序活動從 Data Factory 管線叫用 SSIS 套件的詳細資訊，請參閱[叫用 SSIS 套件](how-to-invoke-ssis-package-stored-procedure-activity.md)。 接著，您可以建立排程觸發程序，以將管線排程為依照您指定的頻率執行。

## <a name="prerequisites"></a>必要條件
如果您尚未佈建 Azure SSIS 整合執行階段，請依照[教學課程](tutorial-create-azure-ssis-runtime-portal.md)中的指示加以佈建。 

## <a name="create-and-test-an-azure-automation-runbook"></a>建立並測試 Azure 自動化 Runbook
在本節中，您將執行下列步驟： 

1. 建立 Azure 自動化帳戶。
2. 在 Azure 自動化帳戶中建立 PowerShell Runbook。 與 Runbook 相關聯的 PowerShell 指令碼會根據您為 OPERATION 參數指定的命令來啟動或停止 Azure SSIS IR。 
3. 在啟動和停止的案例中測試 Runbook，並確認它可正常運作。 

### <a name="create-an-azure-automation-account"></a>建立 Azure 自動化帳戶
如果您沒有 Azure 自動化帳戶，請依照此步驟中的指示建立帳戶。 如需詳細步驟，請參閱[建立 Azure 自動化帳戶](../automation/automation-quickstart-create-account.md)。 在此步驟中，您會建立 **Azure 執行身分**帳戶 (Azure Active Directory 中的服務主體)，並將其新增至您 Azure 訂用帳戶的**參與者**角色。 請確定這就是具有 Azure SSIS IR 的資料處理站所屬訂用帳戶。 Azure 自動化會使用此帳戶對 Azure Resource Manager 進行驗證，並對您的資源運作。 

1. 啟動 **Microsoft Edge** 或 **Google Chrome** 網頁瀏覽器。 目前，只有 Microsoft Edge 和 Google Chrome 網頁瀏覽器支援 Data Factory UI。
2. 登入 [Azure 入口網站](https://portal.azure.com/)。    
3. 在左側功能表上選取 [新增]，再選取 [監視 + 管理]，然後選取 [自動化]。 

    ![新增 -> 監視 + 管理 -> 自動化](./media/how-to-schedule-azure-ssis-integration-runtime/new-automation.png)
2. 在 [新增自動化帳戶] 視窗中，執行下列步驟： 

    1. 指定自動化帳戶的**名稱**。 
    2. 選取具有 Azure SSIS IR 的資料處理站所屬**訂用帳戶**。 
    3. 針對 [資源群組] 選取 [新建]，以建立新的資源群組，或選取 [使用現有的] 以選取現有的資源群組。 
    4. 選取自動化帳戶的**位置**。 
    5. 確認 [建立執行身分帳戶] 設為 [是]。 您的 Azure Active Directory 中會建立服務主體。 它會新增至您 Azure 訂用帳戶的**參與者**角色
    6. 選取 [釘選到儀表板]，以便在入口網站的儀表板上檢視。 
    7. 選取 [建立] 。 

        ![新增 -> 監視 + 管理 -> 自動化](./media/how-to-schedule-azure-ssis-integration-runtime/add-automation-account-window.png)
3. 您可以在儀表板上和通知中檢視**部署狀態**。 
    
    ![部署自動化](./media/how-to-schedule-azure-ssis-integration-runtime/deploying-automation.png) 
4. 自動化帳戶成功建立後會顯示首頁。 

    ![自動化首頁](./media/how-to-schedule-azure-ssis-integration-runtime/automation-home-page.png)

### <a name="import-data-factory-modules"></a>匯入資料處理站模組

1. 在左側功能表的 [共用資源] 區段中選取 [模組]，然後確認您的模組清單中是否有 **AzureRM.Profile** 和 **AzureRM.DataFactoryV2**。

    ![驗證必要的模組](media/how-to-schedule-azure-ssis-integration-runtime/automation-fix-image1.png)

2.  移至 [AzureRM.DataFactoryV2 模組](https://www.powershellgallery.com/packages/AzureRM.DataFactoryV2/)的 PowerShell 資源庫，選取 **[部署至 Azure 自動化]**，選取您的自動化帳戶，然後選取 **[確定]**。 回到左側功能表的 **[共用資源]** 中，檢視 **[模組]**，並等到您看到 **AzureRM.DataFactoryV2** 模組的 **[狀態]** 變更為 **[可用]** 為止。

    ![驗證 Data Factory 模組](media/how-to-schedule-azure-ssis-integration-runtime/automation-fix-image2.png)

3.  移至 [AzureRM.Profile 模組](https://www.powershellgallery.com/packages/AzureRM.profile/)的 PowerShell 資源庫，按一下 **[部署至 Azure 自動化]**，選取您的自動化帳戶，然後選取 **[確定]**。 回到左側功能表的 **[共用資源]** 中，檢視 **[模組]**，並等到您看到 **AzureRM.Profile** 模組的 **[狀態]** 變更為 **[可用]** 為止。

    ![驗證設定檔模組](media/how-to-schedule-azure-ssis-integration-runtime/automation-fix-image3.png)

### <a name="create-a-powershell-runbook"></a>建立 PowerShell Runbook
下列程序提供建立 PowerShell Runbook 的步驟。 與 Runbook 相關聯的指令碼會根據您為 **OPERATION** 參數指定的命令來啟動/停止 Azure SSIS IR。 本節將不提供建立 Runbook 的所有詳細資料。 如需詳細資訊，請參閱[建立 Runbook](../automation/automation-quickstart-create-runbook.md) 一文。

1. 切換至 [Runbook] 索引標籤，然後從工具列中選取 [+ 新增 Runbook]。 

    ![新增 Runbook 按鈕](./media/how-to-schedule-azure-ssis-integration-runtime/runbooks-window.png)
2. 選取 [建立新的 Runbook]，並執行下列步驟： 

    1. 針對 [名稱]，輸入 **StartStopAzureSsisRuntime**。
    2. 針對 [Runbook 類型]，選取 [PowerShell]。
    3. 選取 [建立] 。
    
        ![新增 Runbook 按鈕](./media/how-to-schedule-azure-ssis-integration-runtime/add-runbook-window.png)
3. 在 Runbook 指令碼視窗中複製/貼上下列指令碼。 使用工具列上的 [儲存] 和 [發佈] 按鈕，在儲存 Runbook 後加以發佈。 

    ```powershell
    Param
    (
          [Parameter (Mandatory= $true)]
          [String] $ResourceGroupName,
    
          [Parameter (Mandatory= $true)]
          [String] $DataFactoryName,
    
          [Parameter (Mandatory= $true)]
          [String] $AzureSSISName,
    
          [Parameter (Mandatory= $true)]
          [String] $Operation
    )
    
    $connectionName = "AzureRunAsConnection"
    try
    {
        # Get the connection "AzureRunAsConnection "
        $servicePrincipalConnection=Get-AutomationConnection -Name $connectionName         
    
        "Logging in to Azure..."
        Connect-AzureRmAccount `
            -ServicePrincipal `
            -TenantId $servicePrincipalConnection.TenantId `
            -ApplicationId $servicePrincipalConnection.ApplicationId `
            -CertificateThumbprint $servicePrincipalConnection.CertificateThumbprint 
    }
    catch {
        if (!$servicePrincipalConnection)
        {
            $ErrorMessage = "Connection $connectionName not found."
            throw $ErrorMessage
        } else{
            Write-Error -Message $_.Exception
            throw $_.Exception
        }
    }
    
    if($Operation -eq "START" -or $operation -eq "start")
    {
        "##### Starting #####"
        Start-AzureRmDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -Name $AzureSSISName -Force
    }
    elseif($Operation -eq "STOP" -or $operation -eq "stop")
    {
        "##### Stopping #####"
        Stop-AzureRmDataFactoryV2IntegrationRuntime -DataFactoryName $DataFactoryName -Name $AzureSSISName -ResourceGroupName $ResourceGroupName -Force
    }  
    "##### Completed #####"    
    ```

    ![編輯 PowerShell Runbook](./media/how-to-schedule-azure-ssis-integration-runtime/edit-powershell-runbook.png)
5. 選取工具列上的 [啟動] 按鈕以測試 Runbook。 

    ![啟動 Runbook 按鈕](./media/how-to-schedule-azure-ssis-integration-runtime/start-runbook-button.png)
6. 在 [啟動 Runbook] 視窗中，執行下列步驟： 

    1. 針對 [資源群組名稱]，輸入具有 Azure SSIS IR 的資料處理站所屬的資源群組名稱。 
    2. 針對 [資料處理站名稱]，輸入具有 Azure SSIS IR 的資料處理站名稱。 
    3. 針對 [AZURESSISNAME]，輸入 Azure SSIS IR 的名稱。 
    4. 針對 [OPERATION]，輸入 **START**。 
    5. 選取 [確定] 。  

        ![啟動 Runbook 視窗](./media/how-to-schedule-azure-ssis-integration-runtime/start-runbook-window.png)
7. 在 [作業] 視窗中，選取 [輸出] 圖格。 在作業的輸出視窗中，在您看到 **##### Starting #####** 之後等待 **##### Completed #####** 訊息顯示。 啟動 Azure SSIS IR 約需要 20 分鐘。 關閉 [作業] 視窗，並回到 [Runbook] 視窗。

    ![Azure SSIS IR - 已啟動](./media/how-to-schedule-azure-ssis-integration-runtime/start-completed.png)
8.  重複前兩個步驟，但這次的 [OPERATION] 要使用 **STOP** 值。 選取工具列上的 [啟動] 按鈕，以再次啟動 Runbook。 指定資源群組名稱、資料處理站名稱和 Azure SSIS IR 名稱。 針對 [OPERATION]，輸入 **STOP**。 

    在作業的輸出視窗中，在您看到 **##### Stopping #####** 之後等待 **##### Completed #####** 訊息顯示。 停止 Azure SSIS IR 不會像啟動 Azure SSIS IR 那麼耗時。 關閉 [作業] 視窗，並回到 [Runbook] 視窗。

## <a name="create-schedules-for-the-runbook-to-startstop-the-azure-ssis-ir"></a>建立以 Runbook 啟動/停止 Azure SSIS IR 的排程
在上一節中，您已建立可啟動或停止 Azure SSIS IR 的 Azure 自動化 Runbook。 在本節中，您會為 Runbook 建立兩個排程。 設定第一個排程時，您會為 OPERATION 參數指定 START。 同理，在設定第二個排程時，您會為 OPERATION 參數指定 STOP。 如需建立排程的詳細步驟，請參閱[建立排程](../automation/automation-schedules.md#creating-a-schedule)。

1. 在 [Runbook] 視窗中選取 [排程]，然後選取工具列上的 [+ 新增排程]。 

    ![Azure SSIS IR - 已啟動](./media/how-to-schedule-azure-ssis-integration-runtime/add-schedules-button.png)
2. 在 [排程 Runbook] 視窗中，執行下列步驟： 

    1. 選取 [將排程連結至您的 Runbook]。 
    2. 選取 [建立新的排程]。
    3. 在 [新增排程] 視窗中，輸入**每日啟動 IR** 作為 [名稱]。 
    4. 在 [啟動] 區段中，指定目前時間之後數分鐘的時間。 
    5. 針對 [週期性]，選取 [循環]。 
    6. 在 [重複頻率] 區段中，選取 [天]。 
    7. 選取 [建立] 。 

        ![啟動 Azure SSIS IR 的排程](./media/how-to-schedule-azure-ssis-integration-runtime/new-schedule-start.png)
3. 切換至 [參數與回合設定] 索引標籤。指定資源群組名稱、資料處理站名稱和 Azure SSIS IR 名稱。 針對 [OPERATION]，輸入 **START**。 選取 [確定] 。 再次選取 [確定]，以在 Runbook 的 [排程] 頁面上檢視排程。 

    ![啟動 Azure SSIS IR 的排程](./media/how-to-schedule-azure-ssis-integration-runtime/start-schedule.png)
4. 重複前兩個步驟，以建立名為**每日停止 IR** 的排程。 這次，指定的時間請比**每日啟動 IR** 的排程指定時間至少晚 30 分鐘。 針對 [OPERATION]，指定 [STOP]。 
5. 在 [Runbook] 視窗中，選取左側功能表上的 [作業]。 您應該會看到排程在指定時間建立的作業及其狀態。 您可以檢視作業的相關詳細資料，例如，您先前在測試 Runbook 時所看到的類似作業輸出。 

    ![啟動 Azure SSIS IR 的排程](./media/how-to-schedule-azure-ssis-integration-runtime/schedule-jobs.png)
6. 測試完成後，請編輯排程，並針對 [已啟用] 選取 [否]，以停用排程。 選取左側功能表中的 [排程]，再選取 [每日啟動 IR/每日停止 IR]，然後針對 [已啟用] 選取 [否]。 

## <a name="create-webhooks-to-start-and-stop-the-azure-ssis-ir"></a>建立啟動和停止 Azure SSIS IR 的 Webhook
請依照[建立 Webhook](../automation/automation-webhooks.md#creating-a-webhook) 中的指示，為 Runbook 建立兩個 Webhook。 請為第一個 Webhook 指定 START 作為 OPERATION，第二個則指定 STOP 作為 OPERATION。 將這兩個 Webhook 的 URL 儲存於某處 (例如文字檔或 OneNote 筆記本)。 您在 Data Factory 管線中設定 Web 活動時，可以使用這些 URL。 下圖顯示建立 Webhook 以啟動 Azure SSIS IR 的範例：

1. 在 [Runbook] 視窗中，從左側功能表中選取 [Webhook]，然後選取工具列上的 [+ 新增 Webhook]。 

    ![Webhook -> 新增 Webhook](./media/how-to-schedule-azure-ssis-integration-runtime/add-web-hook-menu.png)
2. 在 [新增 Webhook] 視窗中選取 [建立新的 Webhook]，並執行下列動作： 

    1. 針對 [名稱]，輸入 **StartAzureSsisIR**。 
    2. 確認 [已啟用] 設為 [是]。 
    3. 複製 **URL** 儲存於某處。 此步驟很重要。 您後續不會再看到此 URL。 
    4. 選取 [確定] 。 

        ![新增 Webhook 視窗](./media/how-to-schedule-azure-ssis-integration-runtime/new-web-hook-window.png)
3. 切換至 [參數與回合設定] 索引標籤。指定資源群組名稱、資料處理站名稱和 Azure SSIS IR 名稱。 針對 [OPERATION]，輸入 **START**。 按一下 [確定]。 然後按一下 [ **建立**]。 

    ![Webhook - 參數與回合設定](./media/how-to-schedule-azure-ssis-integration-runtime/webhook-parameters.png)
4. 重複前三個步驟，以建立名為 **StopAzureSsisIR** 的另一個 Webhook。 別忘了複製 URL。 在指定參數與回合設定時，針對 [OPERATION] 輸入 **STOP**。 

您應該有兩個 URL，一個用於 **StartAzureSsisIR** Webhook，另一個則用於 **StopAzureSsisIR** Webhook。 您可以將 HTTP POST 要求傳送至這些 URL，以啟動/停止您的 Azure SSIS IR。 

## <a name="create-and-schedule-a-data-factory-pipeline-that-startsstops-the-ir"></a>建立和排程啟動/停止 IR 的 Data Factory 管線
本節說明如何使用 Web 活動叫用您在上一節建立 Webhook。

您建立的管線包含下列三個活動。 

1. 第一個 **Web** 活動會叫用第一個 Webhook，以啟動 Azure SSIS IR。 
2. **執行 SSIS 套件**活動或**預存程序**活動會執行 SSIS 套件。
3. 第二個 **Web** 活動會叫用停止 Azure SSIS IR 的 Webhook。 

在建立並測試管線後，您會建立排程觸發程序，並使其與管線產生關聯。 排程觸發程序會定義管線的排程。 假設您建立了排程於每天晚上 11 點執行的觸發程序。 此觸發程序會在每天晚上 11 點執行管線。 管線會啟動 Azure SSIS IR、執行 SSIS 套件，然後停止 Azure SSIS IR。 

### <a name="create-a-data-factory"></a>建立 Data Factory

1. 登入 [Azure 入口網站](https://portal.azure.com/)。    
2. 按一下左邊功能表上的 [新增]、[資料 + 分析]，再按一下 [Data Factory]。 
   
   ![新增->DataFactory](./media/tutorial-create-azure-ssis-runtime-portal/new-data-factory-menu.png)
3. 在 [新增資料處理站] 頁面中，輸入 **MyAzureSsisDataFactory** 作為 [名稱]。 
      
     ![新增資料處理站頁面](./media/tutorial-create-azure-ssis-runtime-portal/new-azure-data-factory.png)
 
   Azure Data Factory 的名稱必須是 **全域唯一的**。 如果您收到下列錯誤，請變更資料處理站的名稱 (例如 yournameMyAzureSsisDataFactory)，然後試著重新建立。 請參閱 [Data Factory - 命名規則](naming-rules.md)一文，以了解 Data Factory 成品的命名規則。
  
       `Data factory name �MyAzureSsisDataFactory� is not available`
3. 選取您要在其中建立資料處理站的 Azure **訂用帳戶**。 
4. 針對 [資源群組]，請執行下列其中一個步驟︰
     
      - 選取 [使用現有的] ，然後從下拉式清單選取現有的資源群組。 
      - 選取 [建立新的] ，然後輸入資源群組的名稱。   
         
      若要了解資源群組，請參閱 [使用資源群組管理您的 Azure 資源](../azure-resource-manager/resource-group-overview.md)。  
4. 針對 [版本] 選取 [V2]。
5. 選取 Data Factory 的 [位置]  。 清單中只會顯示資料處理站建立所支援的位置。
6. 選取 [釘選到儀表板]。     
7. 按一下頁面底部的 [新增] 。
8. 在儀表板上，您會看到狀態如下的下列圖格︰**正在部署資料處理站**。 

    ![部署資料處理站圖格](media/tutorial-create-azure-ssis-runtime-portal/deploying-data-factory.png)
9. 建立完成之後，您會看到如圖中所示的 [Data Factory] 頁面。
   
   ![Data Factory 首頁](./media/tutorial-create-azure-ssis-runtime-portal/data-factory-home-page.png)
10. 按一下 [編寫與監視]，以在另一個索引標籤中啟動 Data Factory 使用者介面 (UI)。

### <a name="create-a-pipeline"></a>建立管線

1. 在 [開始使用] 頁面中，選取 [建立管線]。 

   ![開始使用頁面](./media/how-to-schedule-azure-ssis-integration-runtime/get-started-page.png)
2. 在 [活動] 工具箱中展開 [一般]，然後將 **Web** 活動拖放到管線設計工具介面上。 在 [屬性] 視窗的 [一般] 索引標籤中，將活動的名稱變更為 **StartIR**。

   ![第一個 Web 活動 - 一般索引標籤](./media/how-to-schedule-azure-ssis-integration-runtime/first-web-activity-general-tab.png)
3. 切換 [屬性] 視窗中的 [設定] 索引標籤，並執行下列動作： 

    1. 針對 [URL]，貼上會啟動 Azure SSIS IR 之 Webhook 的 URL。 
    2. 針對 [方法]，選取 [POST]。 
    3. 針對 [本文]，輸入 `{"message":"hello world"}`。 
   
        ![第一個 Web 活動 - 設定索引標籤](./media/how-to-schedule-azure-ssis-integration-runtime/first-web-activity-settnigs-tab.png)

4. 從 [活動] 工具箱的 [一般] 區段中拖放「執行 SSIS 套件」活動或「預存程序」活動。 將活動的名稱設為 **RunSSISPackage**。 

5. 如果您選取「執行 SSIS 套件」活動，請依照[在 Azure Data Factory 中使用 SSIS 活動執行 SSIS 套件](how-to-invoke-ssis-package-ssis-activity.md)中的指示來完成活動建立。  確定您指定了足夠的重試次數，且該次數頻繁到足以等待 Azure-SSIS IR 的可用性，因為它最多需要 30 分鐘來啟動。 

    ![重試設定](media/how-to-schedule-azure-ssis-integration-runtime/retry-settings.png)

6. 如果您選取「預存程序」活動，請依照[在 Azure Data Factory 使用預存程序活動叫用 SSIS 套件](how-to-invoke-ssis-package-stored-procedure-activity.md)中的指示來完成活動建立。 確定您插入的 Transact-SQL 指令碼會等候 Azure-SSIS IR 的可用性，因為它最多需要 30 分鐘來啟動。
    ```sql
    DECLARE @return_value int, @exe_id bigint, @err_msg nvarchar(150)

    -- Wait until Azure-SSIS IR is started
    WHILE NOT EXISTS (SELECT * FROM [SSISDB].[catalog].[worker_agents] WHERE IsEnabled = 1 AND LastOnlineTime > DATEADD(MINUTE, -10, SYSDATETIMEOFFSET()))
    BEGIN
        WAITFOR DELAY '00:00:01';
    END

    EXEC @return_value = [SSISDB].[catalog].[create_execution] @folder_name=N'YourFolder',
        @project_name=N'YourProject', @package_name=N'YourPackage',
        @use32bitruntime=0, @runincluster=1, @useanyworker=1,
        @execution_id=@exe_id OUTPUT 

    EXEC [SSISDB].[catalog].[set_execution_parameter_value] @exe_id, @object_type=50, @parameter_name=N'SYNCHRONIZED', @parameter_value=1

    EXEC [SSISDB].[catalog].[start_execution] @execution_id = @exe_id, @retry_count = 0

    -- Raise an error for unsuccessful package execution, check package execution status = created (1)/running (2)/canceled (3)/
    -- failed (4)/pending (5)/ended unexpectedly (6)/succeeded (7)/stopping (8)/completed (9) 
    IF (SELECT [status] FROM [SSISDB].[catalog].[executions] WHERE execution_id = @exe_id) <> 7 
    BEGIN
        SET @err_msg=N'Your package execution did not succeed for execution ID: '+ CAST(@execution_id as nvarchar(20))
        RAISERROR(@err_msg, 15, 1)
    END
    ```

7. 將 **Web** 活動連線至**執行 SSIS 套件**或**預存程序**活動。 

    ![將 Web 和預存程序活動連線](./media/how-to-schedule-azure-ssis-integration-runtime/connect-web-sproc.png)

8. 將另一個 **Web** 活動拖放到**執行 SSIS 套件**或**預存程序**活動右側。 將活動的名稱設為 **StopIR**。 
9. 切換 [屬性] 視窗中的 [設定] 索引標籤，並執行下列動作： 

    1. 針對 [URL]，貼上會停止 Azure SSIS IR 之 Webhook 的 URL。 
    2. 針對 [方法]，選取 [POST]。 
    3. 針對 [本文]，輸入 `{"message":"hello world"}`。  
10. 將**執行 SSIS 套件**活動或**預存程序**活動連線至最後一個 **Web** 活動。

    ![完整管線](./media/how-to-schedule-azure-ssis-integration-runtime/full-pipeline.png)
11. 按一下工具列上的 [驗證]，以驗證管線設定。 按一下 **>>** 按鈕，以關閉 [管線驗證報告]。 

    ![驗證管線](./media/how-to-schedule-azure-ssis-integration-runtime/validate-pipeline.png)

### <a name="test-run-the-pipeline"></a>對管線執行測試

1. 在管線的工具列上，選取 [測試執行]。 您會在底部窗格的 [輸出] 視窗中看到輸出。 

    ![測試執行](./media/how-to-schedule-azure-ssis-integration-runtime/test-run-output.png)
2. 在您 Azure 自動化帳戶的 [Runbook] 頁面中，您可以確認作業是否已執行而啟動及停止 Azure SSIS IR。 

    ![Runbook 作業](./media/how-to-schedule-azure-ssis-integration-runtime/runbook-jobs.png)
3. 啟動 SQL Server Management Studio。 在 [連線到伺服器] 視窗中，執行下列動作： 

    1. 針對 [伺服器名稱]，指定 **&lt;您的 Azure SQL 資料庫&gt;.database.windows.net**。
    2. 選取 [選項 >>]。
    3. 針對 [連線到資料庫]，選取 [SSISDB]。
    4. 選取 [ **連接**]。 
    5. 展開 [Integration Services 目錄] -> [SSISDB] -> [您的資料夾]-> [專案] -> [您的 SSIS 專案]-> [套件]。 
    6. 以滑鼠右鍵按一下 SSIS 套件，然後選取 [報告] -> [標準報告] -> [所有執行]。 
    7. 確認 SSIS 套件已執行。 

        ![確認 SSIS 套件已執行](./media/how-to-schedule-azure-ssis-integration-runtime/verfiy-ssis-package-run.png)

### <a name="schedule-the-pipeline"></a>排程管線 
現在，管線已如您預期的方式運作，您可以建立觸發程序，以指定的頻率執行此管線。 如需將排程觸發程序與管線產生關聯的詳細資訊，請參閱[觸發排程上的管線](quickstart-create-data-factory-portal.md#trigger-the-pipeline-on-a-schedule)。

1. 在管線的工具列上選取 [觸發程序]，然後選取 [新增/編輯]。 

    ![觸發程序 -> 新增/編輯](./media/how-to-schedule-azure-ssis-integration-runtime/trigger-new-menu.png)
2. 在 [新增觸發程序] 視窗中，選取 [+ 新增]。

    ![新增觸發程序 - 新增](./media/how-to-schedule-azure-ssis-integration-runtime/add-triggers-new.png)
3. 在 [新增觸發程序] 中，執行下列動作： 

    1. 針對 [名稱]，指定觸發程序的名稱。 在下列範例中，**每日執行**是觸發程序的名稱。 
    2. 針對 [類型]，選取 [排程]。 
    3. 針對 [開始日期]，選取開始日期和時間。 
    4. 針對 [週期性]，指定觸發程序的頻率。 在下列範例中是每天一次。 
    5. 針對 [結束]，您可以藉由選取 [日期] 選項來指定日期和時間。 
    6. 選取 [已啟動]。 觸發程序會在您將解決方案發佈至資料處理站之後隨即啟動。 
    7. 選取 [下一步] 。

        ![觸發程序 -> 新增/編輯](./media/how-to-schedule-azure-ssis-integration-runtime/new-trigger-window.png)
4. 在 [觸發程序執行參數] 頁面中檢閱警告，然後選取 [完成]。 
5. 選取左窗格中的 [全部發佈]，將解決方案發佈至資料處理站。 

    ![全部發佈](./media/how-to-schedule-azure-ssis-integration-runtime/publish-all.png)

### <a name="monitor-the-pipeline-and-trigger-in-the-azure-portal"></a>在 Azure 入口網站中監視管線和觸發程序

1. 若要監視觸發程序執行和管線執行，請使用左側的 [監視] 索引標籤。 如需詳細步驟，請參閱[監視管線](quickstart-create-data-factory-portal.md#monitor-the-pipeline)。

    ![管線執行](./media/how-to-schedule-azure-ssis-integration-runtime/pipeline-runs.png)
2. 若要檢視與管線執行相關聯的所有活動執行，請選取 [動作] 資料行中的第一個連結 ([檢視活動執行])。 您會看到和管線中每個活動相關聯的三個活動執行 (第一個 Web 活動、預存程序活動和第二個 Web 活動)。 若要切換回去以檢視管線執行，請選取頂端的 [管線] 連結。

    ![活動執行](./media/how-to-schedule-azure-ssis-integration-runtime/activity-runs.png)
3. 您可以從頂端的 [管線執行] 旁邊的下拉式清單中選取 [觸發程序執行]，以檢視觸發程序執行。 

    ![觸發程序執行](./media/how-to-schedule-azure-ssis-integration-runtime/trigger-runs.png)

### <a name="monitor-the-pipeline-and-trigger-with-powershell"></a>使用 PowerShell 監視管線和觸發程序

使用如以下範例的指令碼來監視管線和觸發程序。

1. 取得管線執行的狀態。

  ```powershell
  Get-AzureRmDataFactoryV2PipelineRun -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -PipelineRunId $myPipelineRun
  ```

2. 取得有關觸發程序的資訊。

  ```powershell
  Get-AzureRmDataFactoryV2Trigger -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -Name  "myTrigger"
  ```

3. 取得觸發程序執行的狀態。

  ```powershell
  Get-AzureRmDataFactoryV2TriggerRun -ResourceGroupName $ResourceGroupName -DataFactoryName $DataFactoryName -TriggerName "myTrigger" -TriggerRunStartedAfter "2018-07-15" -TriggerRunStartedBefore "2018-07-16"
  ```

## <a name="next-steps"></a>後續步驟
請參閱下列部落格文章：
-   [使用 ADF 管線中的 SSIS 活動來現代化及擴充您的 ETL/ELT 工作流程](https://blogs.msdn.microsoft.com/ssis/2018/05/23/modernize-and-extend-your-etlelt-workflows-with-ssis-activities-in-adf-pipelines/)

請參閱 SSIS 文件中的下列文章： 

- [在 Azure 上部署、執行和監視 SSIS 套件](/sql/integration-services/lift-shift/ssis-azure-deploy-run-monitor-tutorial)   
- [連線到 Azure 上的 SSIS 目錄](/sql/integration-services/lift-shift/ssis-azure-connect-to-catalog-database)
- [在 Azure 上排程套件執行](/sql/integration-services/lift-shift/ssis-azure-schedule-packages)
- [透過 Windows 驗證連線至內部部署資料來源](/sql/integration-services/lift-shift/ssis-azure-connect-with-windows-auth)
