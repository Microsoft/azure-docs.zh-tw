---
title: 教學課程 - 監視 Azure 防火牆記錄和計量
description: 在本教學課程中，您會了解如何啟用及管理 Azure 防火牆記錄和計量。
services: firewall
author: vhorne
ms.service: firewall
ms.topic: tutorial
ms.date: 9/24/2018
ms.author: victorh
ms.openlocfilehash: 1940fb210481dc75fe48d110776185e90cb3e42f
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46991040"
---
# <a name="tutorial-monitor-azure-firewall-logs-and-metrics"></a>教學課程：監視 Azure 防火牆記錄和計量

您可以使用防火牆記錄來監視 Azure 防火牆。 您也可以使用活動記錄來稽核 Azure 防火牆資源上的作業。 使用計量，您可以在入口網站中檢視效能計數器。 

您可以透過入口網站存取其中一些記錄。 您可以將記錄傳送到 [Log Analytics](../log-analytics/log-analytics-azure-networking-analytics.md)、儲存體和事件中樞，並在 Log Analytics 中或透過不同的工具 (例如 Excel 和 Power BI) 來分析記錄。

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 透過 Azure 入口網站啟用記錄功能
> * 使用 PowerShell 啟用記錄
> * 檢視和分析活動記錄檔
> * 檢視及分析網路和應用程式規則記錄
> * 檢視計量

## <a name="prerequisites"></a>必要條件

在開始本教學課程之前，您應該閱讀 [Azure 防火牆記錄和計量](logs-and-metrics.md)，以取得適用於 Azure 防火牆的診斷記錄和計量概觀。


## <a name="enable-diagnostic-logging-through-the-azure-portal"></a>透過 Azure 入口網站啟用診斷記錄

在您完成此程序來開啟診斷記錄之後，資料可能需要幾分鐘的時間，才會出現在您的記錄中。 如果您一開始沒有看到任何項目，請在幾分鐘後重新檢查一次。

1. 在 Azure 入口網站中，開啟您的防火牆資源群組，然後按一下防火牆。
2. 在 [監視] 底下，按一下 [診斷記錄]。

   針對 Azure 防火牆，可以使用兩個服務特定的記錄：

   * AzureFirewallApplicationRule
   * AzureFirewallNetworkRule

3. 若要開始收集資料，請按一下 [開啟診斷]。
4. [診斷設定] 頁面中提供診斷記錄的設定。 
5. 在此範例中，Log Analytics 會儲存記錄，因此請輸入**防火牆記錄分析**作為名稱。
6. 按一下 [傳送至 Log Analytics] 來設定您的工作區。 您也可以使用事件中樞和儲存體帳戶來儲存診斷記錄。
7. 在 [Log Analytics] 下，按一下 [設定]。
8. 在 [OMS 工作區] 頁面中，按一下 [建立新的工作區]。
9. 在 [Log Analytics 工作區] 頁面上，輸入 **firewall-oms** 作為新的 **OMS 工作區**名稱。
10. 選取您的訂用帳戶、使用現有的防火牆資源群組 (**Test-FW-RG**)、選取 [美國東部] 作為位置，然後選取 [免費] 定價層。
11. 按一下 [確定]。
   ![啟動設定程序][1]
12. 在 [記錄] 底下，按一下 [AzureFirewallApplicationRule] 和 [AzureFirewallNetworkRule]，以收集應用程式規則和網路規則的記錄。
   ![儲存診斷設定][2]
13. 按一下 [檔案] 。

## <a name="enable-logging-with-powershell"></a>使用 PowerShell 啟用記錄

每個 Resource Manager 資源都會自動啟用活動記錄功能。 您必須啟用診斷記錄，才能開始收集可透過這些記錄取得的資料。

若要啟用診斷記錄，請使用下列步驟：

1. 請記下您的儲存體帳戶的資源識別碼 (記錄資料的儲存之處)。 此值的形式為：*/subscriptions/\<subscriptionId\>/resourceGroups/\<資源群組名稱\>/providers/Microsoft.Storage/storageAccounts/\<儲存體帳戶名稱\>*。

   您可以使用訂用帳戶中的所有儲存體帳戶。 您可以使用 Azure 入口網站來尋找此資訊。 資訊位於資源的 [屬性] 頁面中。

2. 請記下您防火牆的資源識別碼 (將為其啟用記錄功能)。 此值的形式為：*/subscriptions/\<subscriptionId\>/resourceGroups/\<資源群組名稱\>/providers/Microsoft.Network/azureFirewalls/\<防火牆名稱\>*。

   您可以使用入口網站來尋找此資訊。

3. 使用下列 Powershell Cmdlet 啟用診斷記錄功能：

    ```powershell
    Set-AzureRmDiagnosticSetting  -ResourceId /subscriptions/<subscriptionId>/resourceGroups/<resource group name>/providers/Microsoft.Network/azureFirewalls/<Firewall name> `
   -StorageAccountId /subscriptions/<subscriptionId>/resourceGroups/<resource group name>/providers/Microsoft.Storage/storageAccounts/<storage account name> `
   -Enabled $true     
    ```
    
> [!TIP] 
>診斷記錄不需要個別的儲存體帳戶。 將儲存體用於記錄存取和效能會產生服務費用。

## <a name="view-and-analyze-the-activity-log"></a>檢視和分析活動記錄檔

您可以使用下列任何方法，檢視和分析活動記錄資料：

* **Azure 工具**：透過 Azure PowerShell、Azure CLI、Azure REST API 或 Azure 入口網站，從活動記錄擷取資訊。 [活動作業與 Resource Manager](../azure-resource-manager/resource-group-audit.md) 一文會詳述每個方法的逐步指示。
* **Power BI**：如果還沒有 [Power BI](https://powerbi.microsoft.com/pricing) 帳戶，您可以免費試用。 使用 [Power BI 的 Azure 活動記錄內容套件](https://powerbi.microsoft.com/en-us/documentation/powerbi-content-pack-azure-audit-logs/)，您可以使用預先設定的儀表板 (可按原樣使用或加以自訂) 來分析資料。

## <a name="view-and-analyze-the-network-and-application-rule-logs"></a>檢視及分析網路和應用程式規則記錄

Azure [Log Analytics](../log-analytics/log-analytics-azure-networking-analytics.md) 會收集計數器和事件記錄檔。 它也納入了視覺效果和強大的搜尋功能來分析您的記錄。

您也可以連接到儲存體帳戶並擷取存取和效能記錄檔的 JSON 記錄檔項目。 下載 JSON 檔案後，可以將它們轉換成 CSV，並在 Excel、PowerBI 或任何其他資料視覺化工具中檢視它們。

> [!TIP]
> 如果您熟悉 Visual Studio 以及在 C# 中變更常數和變數值的基本概念，您可以使用 GitHub 所提供的[記錄檔轉換器工具 (英文)](https://github.com/Azure-Samples/networking-dotnet-log-converter)。

## <a name="view-metrics"></a>檢視計量
瀏覽至 Azure 防火牆，在 [監視] 之下按一下 [計量]。 若要檢視可用的值，請選取 [計量] 下拉式清單。

## <a name="next-steps"></a>後續步驟

既然您已設定防火牆來收集記錄，您可以探索 Log Analytics 以檢視您的資料。

> [!div class="nextstepaction"]
> [Log Analytics 中的網路監視解決方案](../log-analytics/log-analytics-azure-networking-analytics.md)

[1]: ./media/tutorial-diagnostics/figure1.png
[2]: ./media/tutorial-diagnostics/figure2.png
