---
title: 檢視 Azure 活動記錄以監視資源 | Microsoft Docs
description: 使用活動記錄檢閱使用者動作和錯誤。 顯示 Azure 入口網站 PowerShell、Azure CLI 和 REST。
services: azure-resource-manager
documentationcenter: ''
author: tfitzmac
manager: timlt
editor: tysonn
ms.assetid: fcdb3125-13ce-4c3b-9087-f514c5e41e73
ms.service: azure-resource-manager
ms.workload: multiple
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 04/04/2018
ms.author: tomfitz
ms.openlocfilehash: 2dcf93a635a8eb0a01ec266d2478b6e5a336ec00
ms.sourcegitcommit: b6319f1a87d9316122f96769aab0d92b46a6879a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/20/2018
ms.locfileid: "34358688"
---
# <a name="view-activity-logs-to-audit-actions-on-resources"></a>檢視活動記錄以稽核對資源的動作

透過活動記錄檔，您可以判斷︰

* 訂用帳戶的資源在進行哪些作業
* 誰起始作業 (雖然由後端服務起始的作業不會傳回使用者做為呼叫端)
* 作業進行的時間
* 作業的狀態
* 其他可能協助您研究作業的屬性值

活動記錄包含在您的資源上執行的所有寫入作業 (PUT、POST、DELETE)。 不包含讀取作業 (GET)。 如需資源動作的清單，請參閱 [Azure Resource Manager 資源提供者作業](../role-based-access-control/resource-provider-operations.md)。 您可以使用稽核記錄檔在進行疑難排解時發現錯誤，或是監視貴組織使用者修改資源的方式。

活動記錄檔會保留 90 天。 您可以查詢任何的日期範圍，只要開始日期不是在過去 90 天以前。



您可以透過入口網站、PowerShell、Azure CLI、Insights REST API 或 [Insights .NET Library](https://www.nuget.org/packages/Microsoft.Azure.Insights/)擷取活動記錄檔中的資訊。

## <a name="portal"></a>入口網站

1. 若要透過入口網站檢視活動記錄，請選取 [監視]。
   
    ![檢視活動記錄檔](./media/resource-group-audit/select-monitor.png)

   或者，若要自動篩選特定資源或資源群組的活動記錄檔，請選取 [活動記錄]。 請注意，選取的資源會自動篩選活動記錄檔。
   
    ![依資源篩選](./media/resource-group-audit/filtered-by-resource.png)
2. 在 [活動記錄] 中，您會看到最近作業的摘要。
   
    ![顯示動作](./media/resource-group-audit/audit-summary.png)
3. 若要限制顯示的作業數目，可以選取不同的條件。 例如，下圖顯示變更 [時間範圍] 和 [事件起始者] 欄位，以檢視特定使用者或應用程式在上個月所採取的動作。 選取 [套用]  以檢視查詢的結果。
   
    ![設定篩選選項](./media/resource-group-audit/set-filter.png)

4. 如果您稍後需要再執行查詢，請選取 [儲存]  並給查詢一個名稱。
   
    ![儲存查詢](./media/resource-group-audit/save-query.png)
5. 若要快速執行查詢，可以選取其中一個內建的查詢，例如失敗的部署。

    ![選取查詢](./media/resource-group-audit/select-quick-query.png)

   選取的查詢會自動設定必要的篩選值。

    ![檢視部署錯誤](./media/resource-group-audit/view-failed-deployment.png)   

6. 選取其中一項作業以查看事件的摘要。

    ![檢視作業](./media/resource-group-audit/view-operation.png)  

## <a name="powershell"></a>PowerShell

1. 若要擷取記錄檔項目，請執行 **Get-AzureRmLog** 命令。 您可提供額外的參數來篩選項目清單。 如果未指定開始和結束時間，則會傳回最後一個小時的項目。 例如，若要在過去一小時執行期間擷取資源群組的作業：

  ```powershell
  Get-AzureRmLog -ResourceGroup ExampleGroup
  ```
   
    下列範例示範如何使用活動記錄來研究指定期間所採取的作業。 以日期格式指定開始和結束日期。

  ```powershell
  Get-AzureRmLog -ResourceGroup ExampleGroup -StartTime 2015-08-28T06:00 -EndTime 2015-09-10T06:00
  ```

    或者，您可以使用日期函數來指定日期範圍，例如過去 14 天。
   
  ```powershell 
  Get-AzureRmLog -ResourceGroup ExampleGroup -StartTime (Get-Date).AddDays(-14)
  ```

2. 視您指定的開始時間而定，先前的命令可以傳回該資源群組的一長串作業。 您可以提供搜尋準則，以篩選您所尋找的結果。 例如，假如您想研究 Web 應用程式停止執行的方式，可以執行下列命令：

  ```powershell
  Get-AzureRmLog -ResourceGroup ExampleGroup -StartTime (Get-Date).AddDays(-14) | Where-Object OperationName -eq Microsoft.Web/sites/stop/action
  ```

    並藉此了解停止動作是由 someone@contoso.com 所執行。 

  ```powershell 
  Authorization     :
  Scope     : /subscriptions/xxxxx/resourcegroups/ExampleGroup/providers/Microsoft.Web/sites/ExampleSite
  Action    : Microsoft.Web/sites/stop/action
  Role      : Subscription Admin
  Condition :
  Caller            : someone@contoso.com
  CorrelationId     : 84beae59-92aa-4662-a6fc-b6fecc0ff8da
  EventSource       : Administrative
  EventTimestamp    : 8/28/2015 4:08:18 PM
  OperationName     : Microsoft.Web/sites/stop/action
  ResourceGroupName : ExampleGroup
  ResourceId        : /subscriptions/xxxxx/resourcegroups/ExampleGroup/providers/Microsoft.Web/sites/ExampleSite
  Status            : Succeeded
  SubscriptionId    : xxxxx
  SubStatus         : OK
  ```

3. 您可以查閱由特定使用者採取的動作，即使是針對已不存在的資源群組。

  ```powershell 
  Get-AzureRmLog -ResourceGroup deletedgroup -StartTime (Get-Date).AddDays(-14) -Caller someone@contoso.com
  ```

4. 您可以篩選失敗的作業。

  ```powershell
  Get-AzureRmLog -ResourceGroup ExampleGroup -Status Failed
  ```

5. 您可以查看該項目的狀態訊息，專注於一個錯誤。
   
        ((Get-AzureRmLog -Status Failed -ResourceGroup ExampleGroup -DetailedOutput).Properties[1].Content["statusMessage"] | ConvertFrom-Json).error
   
    它會傳回：
   
        code           message                                                                        
        ----           -------                                                                        
        DnsRecordInUse DNS record dns.westus.cloudapp.azure.com is already used by another public IP. 


## <a name="azure-cli"></a>Azure CLI

若要取出記錄項目，請執行 [az monitor activity-log list](/cli/azure/monitor/activity-log#az-monitor-activity-log-list) 命令。

  ```azurecli
  az monitor activity-log list --resource-group <group name>
  ```


## <a name="rest-api"></a>REST API

可用來處理活動記錄檔的 REST 作業屬於 [Insights REST API](https://msdn.microsoft.com/library/azure/dn931943.aspx)的一部分。 若要擷取活動記錄檔事件，請參閱 [列出訂用帳戶中的管理事件](https://msdn.microsoft.com/library/azure/dn931934.aspx)。

## <a name="next-steps"></a>後續步驟

* Azure 活動記錄檔可以搭配 Power BI 用來更深入了解訂用帳戶中的動作。 請參閱 [在 Power BI 和其他工具中檢視和分析 Azure 活動記錄檔](https://azure.microsoft.com/blog/analyze-azure-audit-logs-in-powerbi-more/)。
* 如要了解如何設定安全性原則，請參閱 [Azure 角色型存取控制](../role-based-access-control/role-assignments-portal.md)。
* 若要深入了解檢視部署作業的命令，請參閱[檢視部署作業](resource-manager-deployment-operations.md)。
* 若要了解如何防止刪除所有使用者的資源，請參閱 [使用 Azure Resource Manager 鎖定資源](resource-group-lock-resources.md)。
* 若要查看每個 Microsoft Azure Resource Manager 提供者可用的作業清單，請參閱 [Azure Resource Manager 資源提供者作業](../role-based-access-control/resource-provider-operations.md)

