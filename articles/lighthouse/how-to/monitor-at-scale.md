---
title: 大規模監視委派的資源
description: 瞭解如何在您所管理的客戶租使用者之間，以可擴充的方式有效地使用 Azure 監視器記錄。
ms.date: 01/07/2021
ms.topic: how-to
ms.openlocfilehash: 797a6159d310f85c35c7eb550f05ff152526b3e8
ms.sourcegitcommit: 431bf5709b433bb12ab1f2e591f1f61f6d87f66c
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/12/2021
ms.locfileid: "98131146"
---
# <a name="monitor-delegated-resources-at-scale"></a>大規模監視委派的資源

作為服務提供者，您可能已上線多個客戶租使用者來 [Azure Lighthouse](../overview.md)。 Azure Lighthouse 可讓服務提供者一次在多個租用戶之間執行大規模作業，讓管理工作更有效率。

本主題說明如何在您所管理的客戶租使用者之間，以可擴充的方式使用 [Azure 監視器記錄](../../azure-monitor/platform/data-platform-logs.md) 。 雖然我們指的是本主題中的服務提供者和客戶，但本指南也適用于 [使用 Azure Lighthouse 來管理多個](../concepts/enterprise.md)租使用者的企業。

> [!NOTE]
> 請確定您管理租使用者中的使用者已被授與管理委派客戶訂用帳戶上的 [Log Analytics 工作區所需的角色](../../azure-monitor/platform/manage-access.md#manage-access-using-azure-permissions) 。

## <a name="create-log-analytics-workspaces"></a>建立 Log Analytics 工作區

為了收集資料，您將需要建立 Log Analytics 工作區。 這些 Log Analytics 工作區是 Azure 監視器所收集資料的唯一環境。 每個工作區都有自己的資料存放庫與設定，而且資料來源和解決方案會設定為將其資料儲存在特定的工作區中。

建議您直接在客戶租使用者中建立這些工作區。 如此一來，它們的資料就會保留在其租使用者中，而不是匯出到您的 這也可讓您集中監視 Log Analytics 所支援的任何資源或服務，讓您在監視的資料類型上有更大的彈性。

> [!TIP]
> 用來存取 Log Analytics 工作區資料的任何自動化帳戶，都必須建立在與工作區相同的租使用者中。

您可以使用 [Azure 入口網站](../../azure-monitor/learn/quick-create-workspace.md)、使用 [Azure CLI](../../azure-monitor/learn/quick-create-workspace-cli.md)，或使用 [Azure PowerShell](../../azure-monitor/platform/powershell-workspace-configuration.md)來建立 Log Analytics 工作區。

> [!IMPORTANT]
> 即使在客戶租使用者中建立了所有的工作區，也必須在管理租使用者的訂用帳戶上註冊 Microsoft Insights 資源提供者。

## <a name="deploy-policies-that-log-data"></a>部署記錄資料的原則

建立 Log Analytics 工作區之後，您可以在客戶階層之間部署 [Azure 原則](../../governance/policy/index.yml) ，以便將診斷資料傳送至每個租使用者中的適當工作區。 您部署的確切原則可能會根據您要監視的資源類型而有所不同。

若要深入瞭解如何建立原則，請參閱 [教學課程：建立和管理原則以強制執行合規性](../../governance/policy/tutorials/create-and-manage.md)。 此「 [社區工具](https://github.com/Azure/Azure-Lighthouse-samples/tree/master/tools/azure-diagnostics-policy-generator) 」提供的腳本可協助您建立原則，以監視您所選擇的特定資源類型。

當您決定要部署的原則時，您可以 [大規模地將其部署到委派](policy-at-scale.md)的訂用帳戶。

## <a name="analyze-the-gathered-data"></a>分析收集的資料

在您部署原則之後，資料會記錄在您于每個客戶租使用者中建立的 Log Analytics 工作區中。 若要深入瞭解所有受管理的客戶，您可以使用 [Azure 監視器活頁簿](../../azure-monitor/platform/workbooks-overview.md) 之類的工具來收集和分析來自多個資料來源的資訊。

## <a name="view-alerts-across-customers"></a>跨客戶查看警示

您可以在您管理的客戶租使用者中，查看委派訂用帳戶的 [警示](../../azure-monitor/platform/alerts-overview.md) 。

若要在多個客戶之間自動重新整理警示，請使用 [Azure Resource Graph](../../governance/resource-graph/overview.md) 查詢來篩選警示。 您可以將查詢釘選到儀表板，並選取所有適當的客戶和訂用帳戶。

下列範例查詢將顯示嚴重性0和1個警示，每60分鐘重新整理一次。

```kusto
alertsmanagementresources
| where type == "microsoft.alertsmanagement/alerts"
| where properties.essentials.severity =~ "Sev0" or properties.essentials.severity =~ "Sev1"
| where properties.essentials.monitorCondition == "Fired"
| where properties.essentials.startDateTime > ago(60m)
| project StartTime=properties.essentials.startDateTime,name,Description=properties.essentials.description, Severity=properties.essentials.severity, subscriptionId
| sort by tostring(StartTime)
```

## <a name="next-steps"></a>後續步驟

- 探索這份 [MVP 建立的範例活頁簿](https://github.com/scautomation/Azure-Automation-Update-Management-Workbooks)，藉由查詢多個 Log Analytics 工作區的 [更新管理記錄](../../automation/update-management/query-logs.md) 來追蹤修補程式合規性報告。 
- 了解 [Azure 監視器](../../azure-monitor/index.yml)。
- 瞭解 [Azure 監視器記錄](../../azure-monitor/platform/data-platform-logs.md)。
- 了解[跨租用戶管理體驗](../concepts/cross-tenant-management-experience.md)。