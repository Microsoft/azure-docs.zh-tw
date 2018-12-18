---
title: 使用 Azure 自動化在 StorSimple 資料管理員中啟動作業 | Microsoft Docs
description: 了解如何使用 Azure 自動化來觸發 StorSimple 資料管理員作業
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
ms.openlocfilehash: 1e5fcbee664271058ac1c7fa80bb285e09b8579a
ms.sourcegitcommit: 7edfa9fbed0f9e274209cec6456bf4a689a4c1a6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/17/2018
ms.locfileid: "27862191"
---
# <a name="use-azure-automation-to-trigger-a-job"></a>使用 Azure 自動化來觸發作業

本文說明如何使用 StorSimple Data Manager 服務中的資料轉換功能來轉換 StorSimple 裝置資料。 您可以使用下列兩種方式啟動資料轉換作業： 

 - 使用 .NET SDK
 - 使用 Azure 自動化 Runbook
 
本文詳述如何建立 Azure 自動化 Runbook，然後使用它來起始資料轉換作業。 若要深入了解如何透過 .NET SDK 起始資料轉換，請移至[使用 .NET SDK 來觸發資料轉換作業](storsimple-data-manager-dotnet-jobs.md)。

## <a name="prerequisites"></a>先決條件

開始之前，請確定您有︰

*   在用戶端電腦上安裝的 Azure PowerShell。 [下載 Azure PowerShell](https://docs.microsoft.com/powershell/azure/install-azurerm-ps)。
*   在資源群組內的 StorSimple 資料管理員服務中正確設定的作業定義。
*   從 GitHub 存放庫下載 [`DataTransformationApp.zip`](https://github.com/Azure-Samples/storsimple-dotnet-data-manager-get-started/raw/master/Azure%20Automation%20For%20Data%20Manager/DataTransformationApp.zip) 檔案。 
*   從 GitHub 存放庫下載 [`Trigger-DataTransformation-Job.ps1`](https://github.com/Azure-Samples/storsimple-dotnet-data-manager-get-started/blob/master/Azure%20Automation%20For%20Data%20Manager/Trigger-DataTransformation-Job.ps1) 指令碼。

## <a name="step-by-step-procedure"></a>逐步程序

### <a name="set-up-the-automation-account"></a>設定自動化帳戶

1. 在 Azure 入口網站中建立 Azure 執行身分自動化帳戶。 若要這樣做，請移至 [Azure Marketplace] > [所有項目]，然後搜尋**自動化**。 選取 [自動化帳戶]。

    ![建立執行身分自動化帳戶](./media/storsimple-data-manager-job-using-automation/search-automation-account1.png)

2. 若要新增自動化帳戶，請按一下 [+ 新增]。

    ![建立執行身分自動化帳戶](./media/storsimple-data-manager-job-using-automation/add-automation-account1.png)

3. 在 [新增自動化] 中：

    1. 提供自動化帳戶的 [名稱]。
    2. 選取連結至 StorSimple 裝置管理員服務的 [訂用帳戶]。
    3. 建立新的資源群組，或從現有的資源群組中選取。
    4. 選取 [位置] 。
    5. 保持選取預設值 [建立執行身分帳戶] 選項。
    6. 若要在儀表板上取得可供快速存取的連結，請核取 [釘選到儀表板]。 按一下頁面底部的 [新增] 。

    ![建立執行身分自動化帳戶](./media/storsimple-data-manager-job-using-automation/create-automation-run-as-account.png)
    
    成功建立自動化帳戶之後會通知您。
    
    ![自動化帳戶部署通知](./media/storsimple-data-manager-job-using-automation/deployment-automation-account-notification1.png)

    如需詳細資訊，請移至[建立執行身分帳戶](../automation/automation-create-runas-account.md)。

3. 在新建立的帳戶中，移至 [共用資源] > [模組] 並按一下 [+ 新增模組]。

    ![匯入模組 1](./media/storsimple-data-manager-job-using-automation/import-module-1.png)

4. 從本機電腦瀏覽至 `DataTransformationApp.zip` 檔案的位置，然後選取並開啟模組。 按一下 [確定] 以匯入模組。

    ![匯入模組 2](./media/storsimple-data-manager-job-using-automation/import-module-2.png)

   當 Azure 自動化將模組匯入到您的帳戶時，會擷取有關模組的中繼資料。 此作業可能需要幾分鐘的時間。

   ![匯入模組 4](./media/storsimple-data-manager-job-using-automation/import-module-4.png)

5. 您會在部署模組時收到一則通知，而當程序完成時會收到另一則通知。  [模組] 中的狀態會變更為 [可用]。

    ![匯入模組 5](./media/storsimple-data-manager-job-using-automation/import-module-5.png)

### <a name="import-publish-and-run-automation-runbook"></a>匯入、發佈和執行自動化 Runbook

請執行下列步驟來匯入、發佈及執行用來觸發作業定義的 Runbook。

1. 在 Azure 入口網站中，開啟您的自動化帳戶。 移至 [程序自動化] > [Runbook]，然後按一下 [+ 新增 Runbook]。

    ![新增 Runbook 1](./media/storsimple-data-manager-job-using-automation/add-runbook-1.png)

2. 在 [新增 Runbook] 中，按一下 [匯入現有 Runbook]。

3. 在 [Runbook 檔案] 中指向 Azure PowerShell 指令碼檔案 `Trigger-DataTransformation-Job.ps1`。 會自動選取 Runbook 類型。 提供 Runbook 的名稱和選擇性描述。 按一下頁面底部的 [新增] 。

    ![新增 Runbook 2](./media/storsimple-data-manager-job-using-automation/add-runbook-2.png)

4. 新的 Runbook 會出現在自動化帳戶的 Runbook 清單中。 選取並按一下此 Runbook。

    ![新增 Runbook 3](./media/storsimple-data-manager-job-using-automation/add-runbook-3.png)

5. 編輯 Runbook，然後按一下 [測試] 窗格。

    ![新增 Runbook 4](./media/storsimple-data-manager-job-using-automation/add-runbook-4.png)

6. 提供各項參數，例如 StorSimple 資料管理員服務的名稱、相關聯的資源群組，以及作業定義名稱。 **開始**測試。 執行完成後即會產生報告。 如需詳細資訊，請移至如何[測試 Runbook](../automation/automation-first-runbook-textual-powershell.md#step-3---test-the-runbook)。

    ![新增 Runbook 8](./media/storsimple-data-manager-job-using-automation/add-runbook-8.png)    

7. 在 [測試] 窗格中檢查 Runbook 的輸出。 如果滿意測試結果，請關閉窗格。 按一下 [發佈] 並在提示要求確認時予以確認，並發佈 Runbook。

    ![新增 Runbook 6](./media/storsimple-data-manager-job-using-automation/add-runbook-6.png)

8. 返回 [Runbook]，然後選取新建立的 Runbook。

    ![新增 Runbook 7](./media/storsimple-data-manager-job-using-automation/add-runbook-7.png)

9. **啟動** Runbook。 在 [啟動 Runbook] 中，輸入所有參數。 按一下 [確定]，以提交並啟動資料轉換作業。

10. 若要在 Azure 入口網站中監視作業進度，請移至 StorSimple 資料管理員服務中的 [作業]。 選取並按一下作業，以檢視作業詳細資料。

    ![新增 Runbook 10](./media/storsimple-data-manager-job-using-automation/add-runbook-10.png)

## <a name="next-steps"></a>後續步驟

[使用 StorSimple Data Manager UI 來轉換資料](storsimple-data-manager-ui.md)。