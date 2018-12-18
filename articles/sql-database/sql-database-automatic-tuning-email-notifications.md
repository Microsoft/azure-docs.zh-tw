---
title: 自動調整電子郵件通知操作說明指南 - Azure SQL Database | Microsoft Docs
description: 啟用 Azure SQL Database 自動查詢調整的電子郵件通知。
services: sql-database
ms.service: sql-database
ms.subservice: performance
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: danimir
ms.author: v-daljep
ms.reviewer: carlrab
manager: craigg
ms.date: 10/15/2018
ms.openlocfilehash: 1177703dc67e81e537d7682dcf9bbeb475748315
ms.sourcegitcommit: 8e06d67ea248340a83341f920881092fd2a4163c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49353929"
---
# <a name="email-notifications-for-automatic-tuning"></a>針對自動調整的電子郵件通知

SQL Database 的調整建議是由 Azure SQL Database [自動調整](sql-database-automatic-tuning.md)產生。 這個解決方案會持續監視並分析 SQL Database 的工作負載，針對與建立索引、刪除索引和最佳化查詢執行計畫相關的個別資料庫提供自訂的調整建議。

您可以在 [Azure 入口網站](sql-database-advisor-portal.md)中檢視使用 [REST API](https://docs.microsoft.com/rest/api/sql/databaserecommendedactions/databaserecommendedactions_listbydatabaseadvisor) 呼叫或使用 [T-SQL](https://azure.microsoft.com/blog/automatic-tuning-introduces-automatic-plan-correction-and-t-sql-management/) 和 [PowerShell](https://docs.microsoft.com/powershell/module/azurerm.sql/get-azurermsqldatabaserecommendedaction) 命令擷取的 SQL Database 自動調整建議。 本文使用 PowerShell 指令碼來擷取自動調整建議。

## <a name="automate-email-notifications-for-automatic-tuning-recommendations"></a>對自動調整建議的電子郵件通知進行自動化

下列解決方案會自動傳送包含自動調整建議的電子郵件通知。 所描述的解決方案包含使用 [Azure 自動化](https://docs.microsoft.com/azure/automation/automation-intro)來自動執行 PowerShell 指令碼以擷取調整建議，以及使用 [Microsoft Flow](https://flow.microsoft.com) 來自動排程電子郵件傳送工作。

## <a name="create-azure-automation-account"></a>建立 Azure 自動化帳戶

要使用 Azure 自動化，第一步是建立自動化帳戶，並設定要用於執行 PowerShell 指令碼的 Azure 資源。 若要深入了解 Azure 自動化和其功能，請參閱[開始使用 Azure 自動化](https://docs.microsoft.com/azure/automation/automation-offering-get-started)。

遵循下列步驟，以透過從 Marketplace 選取和設定自動化應用程式的方法來建立 Azure 自動化帳戶：

- 登入 Azure 入口網站
- 按一下左上角的 [+ 建立資源]
- 搜尋 [自動化] (按 Enter)
- 按一下搜尋結果中的自動化應用程式

![加入 Azure 自動化](./media/sql-database-automatic-tuning-email-notifications/howto-email-01.png)

- 進入 [建立自動化帳戶] 窗格後，按一下 [建立]
- 填入所需的資訊：輸入此自動化帳戶的名稱，選取要用於執行 PowerShell 指令碼的 Azure 訂用帳戶識別碼和 Azure 資源
- 針對 [建立 Azure 執行身分帳戶] 選項，選取 [是] 以設定 Azure 自動化協助執行 PowerShell 指令碼時所使用的帳戶類型。 若要深入了解帳戶類型，請參閱[執行身分帳戶](https://docs.microsoft.com/azure/automation/automation-create-runas-account)
- 按一下 [建立] 以結束建立自動化帳戶

> [!TIP]
> 正確記錄 (例如複製並貼到 [記事本]) 建立自動化應用程式時所輸入的 Azure 自動化帳戶名稱、訂用帳戶識別碼和資源。 稍後會需要此資訊。
>

如果您想為多個 Azure 訂用帳戶建置相同的自動化功能，則必須針對其他訂用帳戶重複此程序。

## <a name="update-azure-automation-modules"></a>更新 Azure 自動化模組

擷取自動調整建議的 PowerShell 指令碼會使用 [Get AzureRmResource](https://docs.microsoft.com/powershell/module/AzureRM.Resources/Get-AzureRmResource) 和 [Get AzureRmSqlDatabaseRecommendedAction](https://docs.microsoft.com/powershell/module/AzureRM.Sql/Get-AzureRmSqlDatabaseRecommendedAction) 命令，因此必須將 Azure 模組更新到第 4 版及更新版本。

遵循下列步驟來更新 Azure PowerShell 模組：

- 存取 [自動化應用程式] 窗格，然後選取左側功能表中的 [模組] (請向下捲動，因為這個功能表項目位於 [共用資源] 底下)。
- 在 [模組] 窗格中，按一下上方的 [更新 Azure 模組]，並等候「已更新 Azure 模組」訊息顯示。 此程序需要數分鐘的時間完成。

![更新 Azure 自動化模組](./media/sql-database-automatic-tuning-email-notifications/howto-email-02.png)

AzureRM.Resources 和 AzureRM.Sql 模組的必要版本必須是第 4 版及更新版本。

## <a name="create-azure-automation-runbook"></a>建立 Azure 自動化 Runbook

下一步是在 Azure 自動化中建立擷取調整建議的 PowerShell 指令碼所在的 Runbook。

遵循下列步驟來建立新的 Azure 自動化 Runbook：

- 存取您在上一個步驟中建立的 Azure 自動化帳戶
- 進入 [自動化帳戶] 窗格後，按一下左側的 [Runbook] 功能表項目，以使用 PowerShell 指令碼建立新的 Azure 自動化 Runbook。 若要深入了解建立自動化 Runbook，請參閱[建立新的 Runbook](../automation/automation-creating-importing-runbook.md)。
- 若要加入新的 Runbook，請按一下 [+加入 Runbook] 功能表選項，然後按一下 [快速建立 - 建立新的 Runbook]。
- 在 [Runbook] 窗格中，輸入您的 Runbook 名稱 (針對此範例用途，我們將使用 **AutomaticTuningEmailAutomation**)，將 Runbook 類型選取為 [PowerShell]，然後編寫此 Runbook 的描述以描述其目的。
- 按一下 [建立] 按鈕以完成建立新的 Runbook

![加入 Azure 自動化 Runbook](./media/sql-database-automatic-tuning-email-notifications/howto-email-03.png)

遵循下列步驟來將 PowerShell 指令碼載入已建立的 Runbook 中：

- 在 [編輯 PowerShell Runbook] 窗格的功能表樹狀上，選取 [RUNBOOK]並展開檢視，直到您看到 Runbook 的名稱 (此範例中為 **AutomaticTuningEmailAutomation**)。 選取此 Runbook。
- 在 [編輯 PowerShell Runbook] 的第一行 (從數字 1 開始)，複製貼上下列 PowerShell 指令碼。 這個 PowerShell 指令碼是依現狀提供，以協助您開始使用。 請修改指令碼以符合您的需求。

在提供的 PowerShell 指令碼標頭中，必須以您的 Azure 訂用帳戶識別碼取代 `<SUBSCRIPTION_ID_WITH_DATABASES>`。 若要深入了解如何擷取您的 Azure 訂用帳戶識別碼，請參閱[取得您的 Azure 訂用帳戶 GUID](https://blogs.msdn.microsoft.com/mschray/2016/03/18/getting-your-azure-subscription-guid-new-portal/) \(英文\)。

若有多個訂用帳戶，您能以逗號分隔的格式將它們加入到指令碼標頭中的 "$subscriptions" 屬性。

```PowerShell
# PowerShell script to retrieve Azure SQL Database Automatic tuning recommendations.
#
# Provided “as-is” with no implied warranties or support.
# The script is released to the public domain.
#
# Replace <SUBSCRIPTION_ID_WITH_DATABASES> in the header with your Azure subscription ID.
#
# Microsoft Azure SQL Database team, 2018-01-22.

# Set subscriptions : IMPORTANT – REPLACE <SUBSCRIPTION_ID_WITH_DATABASES> WITH YOUR SUBSCRIPTION ID
$subscriptions = ("<SUBSCRIPTION_ID_WITH_DATABASES>", "<SECOND_SUBSCRIPTION_ID_WITH_DATABASES>", "<THIRD_SUBSCRIPTION_ID_WITH_DATABASES>")

# Get credentials
$Conn = Get-AutomationConnection -Name AzureRunAsConnection
Connect-AzureRmAccount -ServicePrincipal -Tenant $Conn.TenantID -ApplicationId $Conn.ApplicationID -CertificateThumbprint $Conn.CertificateThumbprint

# Define the resource types
$resourceTypes = ("Microsoft.Sql/servers/databases")
$advisors = ("CreateIndex", "DropIndex");
$results = @()

# Loop through all subscriptions
foreach($subscriptionId in $subscriptions) {
    Select-AzureRmSubscription -SubscriptionId $subscriptionId
    $rgs = Get-AzureRmResourceGroup

    # Loop through all resource groups
    foreach($rg in $rgs) {
        $rgname = $rg.ResourceGroupName;

        # Loop through all resource types
        foreach($resourceType in $resourceTypes) {
            $resources = Get-AzureRmResource -ResourceGroupName $rgname -ResourceType $resourceType

            # Loop through all databases
            # Extract resource groups, servers and databases
            foreach ($resource in $resources) {
                $resourceId = $resource.ResourceId
                if ($resourceId -match ".*RESOURCEGROUPS/(?<content>.*)/PROVIDERS.*") {
                    $ResourceGroupName = $matches['content']
                } else {
                    continue
                }
                if ($resourceId -match ".*SERVERS/(?<content>.*)/DATABASES.*") {
                    $ServerName = $matches['content']
                } else {
                    continue
                }
                if ($resourceId -match ".*/DATABASES/(?<content>.*)") {
                    $DatabaseName = $matches['content']
                } else {
                    continue
                }

                # Skip if master
                if ($DatabaseName -eq "master") {
                    continue
                }

                # Loop through all Automatic tuning recommendation types
                foreach ($advisor in $advisors) {
                    $recs = Get-AzureRmSqlDatabaseRecommendedAction -ResourceGroupName $ResourceGroupName -ServerName $ServerName  -DatabaseName $DatabaseName -AdvisorName $advisor
                    foreach ($r in $recs) {
                        if ($r.State.CurrentValue -eq "Active") {
                            $object = New-Object -TypeName PSObject
                            $object | Add-Member -Name 'SubscriptionId' -MemberType Noteproperty -Value $subscriptionId
                            $object | Add-Member -Name 'ResourceGroupName' -MemberType Noteproperty -Value $r.ResourceGroupName
                            $object | Add-Member -Name 'ServerName' -MemberType Noteproperty -Value $r.ServerName
                            $object | Add-Member -Name 'DatabaseName' -MemberType Noteproperty -Value $r.DatabaseName
                            $object | Add-Member -Name 'Script' -MemberType Noteproperty -Value $r.ImplementationDetails.Script
                            $results += $object
                        }
                    }
                }
            }
        }
    }
}

# Format and output results for the email
$table = $results | Format-List
Write-Output $table
```

按一下右上角的 [儲存] 按鈕以儲存指令碼。 當您對指令碼感到滿意之後，請按一下 [發佈] 按鈕以發佈此 Runbook。

在主要 Runbook 窗格中，您可以選擇按一下 [啟動] 按鈕以**測試**指令碼。 按一下 [輸出] 以檢視所執行指令碼的結果。 此輸出將是您的電子郵件內容。 在下列螢幕擷取畫面中，可以看到指令碼的範例輸出。

![使用 Azure 自動化執行檢視自動調整建議](./media/sql-database-automatic-tuning-email-notifications/howto-email-04.png)

請務必自訂 PowerShell 指令碼來調整內容，以符合您的需求。

使用上述步驟，擷取自動調整建議的 PowerShell 指令碼就會載入 Azure 自動化中。 下一步是自動化和排程電子郵件傳送工作。

## <a name="automate-the-email-jobs-with-microsoft-flow"></a>使用 Microsoft Flow 自動化電子郵件工作

若要完成此解決方案，最後一個步驟是在 Microsoft Flow 中建立包含三個動作 (工作) 的自動化流程：

1. **Azure 自動化 - 建立工作**：用來執行 PowerShell 指令碼以擷取 Azure 自動化 Runbook 內的自動調整建議
2. **Azure 自動化 - 取得工作輸出**：用來擷取所執行 PowerShell 指令碼的輸出
3. **Office 365 Outlook – 傳送電子郵件**：用來傳送電子郵件。 電子郵件會使用建立流程之人員的 Office 365 帳戶傳送。

若要深入了解 Microsoft Flow 功能，請參閱[開始使用 Microsoft Flow](https://docs.microsoft.com/flow/getting-started)。

此步驟的先決條件是註冊 [Microsoft Flow](https://flow.microsoft.com) 帳戶並登入。 一旦進入解決方案，請遵循下列步驟來設定**新的流程**：

- 存取 [我的流程] 功能表項目
- 在 [我的流程] 中，選取頁面頂端的 [+從空白建立]
- 按一下頁面底部的 [搜尋數百個連接器和觸發程序] 連結
- 在搜尋欄位輸入「週期」，並從搜尋結果中選取 [排程 - 週期] 以排程要執行的電子郵件傳送工作。
- 在 [頻率] 欄位的 [週期] 窗格中，選取執行此流程的排程頻率，例如以每分鐘、每小時、每天、每週等週期傳送自動化電子郵件。

下一步是在新建立的週期性流程中加入三個工作 (建立、取得輸出和傳送電子郵件)。 若要將必要的工作加入流程，請遵循下列步驟：

1. 建立動作來執行 PowerShell 指令碼，以擷取調整建議

   - 在 [週期] 流程窗格內選取 [+新增步驟]，然後選取 [加入動作]
   - 在搜尋欄位中輸入「自動化」，並從搜尋結果中選取 [Azure 自動化 – 建立工作]
   - 在 [建立工作] 窗格中，設定工作屬性。 針對此設定，您將需要先前在 [自動化帳戶] 窗格中記錄的 Azure 訂用帳戶識別碼、資源群組和自動化帳戶的詳細資料。 若要深入了解此區段所提供的選項，請參閱 [Azure 自動化 - 建立工作](https://docs.microsoft.com/connectors/azureautomation/#create-job) \(英文\)。
   - 按一下 [儲存流程] 以完成建立此動作

2. 建立動作以從已執行的 PowerShell 指令碼擷取輸出

   - 在 [週期] 流程窗格內選取 [+新增步驟]，然後選取 [加入動作]
   - 在搜尋欄位中輸入「自動化」，並從搜尋結果中選取 [Azure 自動化 – 取得工作輸出]。 若要深入了解此區段所提供的選項，請參閱 [Azure 自動化 – 取得工作輸出](https://docs.microsoft.com/connectors/azureautomation/#get-job-output) \(英文\)。
   - 填入必要欄位 (與建立先前的工作類似) - 填入 Azure 訂用帳戶識別碼、資源群組及自動化帳戶 (如同在 [自動化帳戶] 窗格中所輸入的)
   - 在 [工作識別碼] 欄位內按一下，以顯示 [動態內容] 功能表。 從這個功能表內，選取 [工作識別碼] 選項。
   - 按一下 [儲存流程] 以完成建立此動作

3. 使用 Office 365 整合建立動作以傳送電子郵件

   - 在 [週期] 流程窗格內選取 [+新增步驟]，然後選取 [加入動作]
   - 在搜尋欄位中輸入「傳送電子郵件」，並從搜尋結果中選取 [Office 365 Outlook – 傳送電子郵件]
   - 在 [收件者] 欄位中，輸入必須傳送通知電子郵件的電子郵件地址
   - 在 [主旨] 欄位中，輸入您的電子郵件主旨，例如「自動調整建議電子郵件通知」
   - 在 [本文] 欄位內按一下，以顯示 [動態內容] 功能表。 從這個功能表內，在 [取得工作輸出] 底下，選取 [內容]
   - 按一下 [儲存流程] 以完成建立此動作

> [!TIP]
> 若要傳送自動化電子郵件給不同的收件者，請建立個別流程。 在這些額外的流程中，變更 [收件者] 欄位中的收件者電子郵件地址，以及 [主旨] 欄位中的電子郵件主旨列。 使用自訂 PowerShell 指令碼 (例如變更 Azure 訂用帳戶識別碼) 在 Azure 自動化中建立新的 Runbook 可進一步自訂自動化案例，例如針對個別的訂用帳戶，傳送有關自動調整建議的電子郵件給個別的收件者。
>

以上是設定電子郵件傳送工作之工作流程所需的步驟。 整個流程包含三個動作 (顯示於下圖)。

![檢視自動調整電子郵件通知流程](./media/sql-database-automatic-tuning-email-notifications/howto-email-05.png)

若要測試流程，請按一下流程窗格內右上角的 [立即執行]。

從 [流程分析] 窗格中可以看到執行自動化工作的統計資料 (顯示成功送出電子郵件通知)。

![自動調整電子郵件通知的執行流程](./media/sql-database-automatic-tuning-email-notifications/howto-email-06.png)

[流程] 分析可協助您監視工作執行是否成功，以及是否需要進行疑難排解。  在進行疑難排解時，您也可以檢查可透過 Azure 自動化應用程式存取的 PowerShell 指令碼執行記錄。

自動化電子郵件的最終輸出，看起來類似在建置和執行此解決方案後所收到的電子郵件：

![自動調整電子郵件通知的範例電子郵件輸出](./media/sql-database-automatic-tuning-email-notifications/howto-email-07.png)

藉由調整 PowerShell 指令碼，您可以調整自動化電子郵件的輸出和格式，以符合您的需求。

視您的自訂案例而定，您可進一步將解決方案自訂為根據特定調整事件建立電子郵件通知，以及針對多個訂用帳戶或資料庫傳送給多個收件者。

## <a name="next-steps"></a>後續步驟

- 若要了解自動調整如何能協助您提升資料庫效能，請參閱 [Azure SQL Database 中的自動調整](sql-database-automatic-tuning.md)。
- 若要在 Azure SQL Database 中啟用自動調整以管理您的工作負載，請參閱[啟用自動調整](sql-database-automatic-tuning-enable.md)。
- 若要手動檢閱及套用自動調整建議，請參閱[尋找及套用效能建議](sql-database-advisor-portal.md)。
