---
title: 在 Azure 監視器中建立、檢視及管理活動記錄警示
description: 如何從 Azure 入口網站、資源範本和 PowerShell 建立活動記錄警示。
author: msvijayn
services: azure-monitor
ms.service: azure-monitor
ms.topic: conceptual
ms.date: 09/15/2018
ms.author: vinagara
ms.openlocfilehash: 6d0c8f62d109d07a9f08e5190a5a2caa0d66a0c1
ms.sourcegitcommit: 7cd706612a2712e4dd11e8ca8d172e81d561e1db
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/18/2018
ms.locfileid: "53579323"
---
# <a name="create-view-and-manage-activity-log-alerts-using-azure-monitor"></a>使用 Azure 監視器中建立、檢視及管理活動記錄警示  

## <a name="overview"></a>概觀
活動記錄警示是發生符合警示中指定條件的新活動記錄事件時所啟動的警示。

這些警示都適用於 Azure 資源，可以使用 Azure Resource Manager 範本來建立。 也可以在 Azure 入口網站中將它們建立、更新或刪除。 一般而言，您可以建立活動記錄警示，以在 Azure 訂用帳戶中的資源發生特定變更時接收通知 (通常會將範圍限定在特定資源群組或資源)。 例如，在 (範例資源群組) **myProductionResourceGroup** 中的任何虛擬機器被刪除時，您可能會想要收到通知，或者，您可能想要在有任何新角色被指派給訂用帳戶中的使用者時收到通知。

> [!IMPORTANT]
> 服務健康情況通知的警示是無法透過活動記錄警示建立的介面來建立的。 若要深入了解建立及使用服務健康情況通知，請參閱[在服務健康情況通知上接收活動記錄警示](alerts-activity-log-service-notifications.md)。

## <a name="azure-portal"></a>Azure 入口網站

> [!NOTE]

>  在建立警示規則時，請確定下列各項：

> - 範圍中的訂用帳戶和警示建立所在訂用帳戶並無不同。
- 設定警示必須以層級/狀態/呼叫端/資源群組/資源識別碼/資源類型/事件類別目錄為準則。
- 警示設定 JSON 中不得有 “anyOf” 條件或巢狀條件 (基本上，只允許 allOf，不得進一步使用 allOf/anyOf)。
- 當類別為「系統管理」時。 您必須在您的警示中指定至少一個上述準則。 您不能建立會在每次活動記錄中建立事件時即啟動的警示。

### <a name="create-with-azure-portal"></a>使用 Azure 入口網站建立

請使用下列程序：

1. 從 Azure 入口網站選取 [監視器] > [警示]
2. 按一下 [警示] 視窗頂端的 [新增警示規則]。

     ![新增警示規則](media/alerts-activity-log/AlertsPreviewOption.png)

     [建立規則] 視窗隨即出現。

      ![新增警示規則選項](media/alerts-activity-log/create-new-alert-rule-options.png)

3. 在 [定義警示條件] 下提供下列資訊，然後按一下 [完成]。

    - **警示目標：** 若要檢視和選取新警示的目標，請使用 [依訂用帳戶篩選] / [依資源類型篩選]，然後從顯示的清單中選取資源或資源群組。

    > [!NOTE]

    > 您可以為活動記錄訊號選取資源、資源群組或整個訂用帳戶。

    **警示目標範例檢視**![選取目標](media/alerts-activity-log/select-target.png)

    - 在 [目標準則] 之下，按一下 [新增準則]，接著會顯示適用於目標的所有可用訊號，包括來自 [活動記錄] 各種類別的訊號；具有附加在 [監視服務] 名稱中的類別名稱。

    - 從類型 [活動記錄] 的各種可能作業顯示的清單中選取訊號。

    您可以選取此目標訊號的記錄歷程記錄時間軸和對應的警示邏輯：

    **新增準則畫面**

    ![新增準則](media/alerts-activity-log/add-criteria.png)

    **歷程記錄時間**：所選作業的可用事件是可在過去 6/12/24 小時 (或) 過去一週繪製的。

    **警示邏輯**：

     - **事件等級**- 事件的嚴重性等級。 _詳細資訊_ _資訊_ _警告_ _錯誤_ 或 _嚴重_。
     - **狀態**：事件的狀態。 _已啟動_ _失敗_ 或 _成功_。
     - **事件起始者**：也稱為呼叫端；執行作業之使用者的電子郵件地址或 Azure Active Directory 識別碼。

        已套用警示邏輯的範例訊號圖表：

        ![ 選取的準則](media/alerts-activity-log/criteria-selected.png)

4. 在 [定義警示規則詳細資料] 下，提供下列詳細資料：

    - **警示規則名稱**：新警示規則的名稱
    - **描述**︰新警示規則描述
    - **將警示儲存到資源群組**：選取您儲存此新規則的資源群組。

5. 在 [動作群組] 下，從下拉式功能表中指定您想要指派給此新警示規則的動作群組。 或者，[建立新的動作群組](../../azure-monitor/platform/action-groups.md)並指派給新規則。 若要建立新群組，請按一下 [+ 新增群組]。

6. 若要在建立規則之後啟用，請按一下 [在建立時啟用規則] 選項的 [是]。
7. 按一下 [建立警示規則]。

    活動記錄的新警示規則隨即建立，而且視窗右上方會顯示確認訊息。

    您可以啟用、停用、編輯或刪除規則。 [深入了解](#view-and-manage-activity-log-alert-rules-in-azure-portal)如何管理活動記錄規則。


或者，有一個簡單類比可用來了解哪些警示規則可在活動記錄上建立，就是透過 [Azure 入口網站中的活動記錄](../../azure-monitor/platform/activity-logs-overview.md#query-the-activity-log-in-the-azure-portal)探索或篩選事件。 在 Azure 監視器 - 活動記錄中，您可以篩選或尋找必要事件，然後透過使用 [新增活動記錄警示] 按鈕來建立警示；然後依照以上教學課程中所述的步驟 4 開始。
    
 ![ 從活動記錄新增警示](media/alerts-activity-log/add-activity-log.png)
    

### <a name="view-and-manage-in-azure-portal"></a>在 Azure 入口網站中檢視及管理

1. 從 Azure 入口網站按一下 [監視器] > [警示]，然後按一下視窗左上方的 [管理規則]。

    ![ 管理警示規則](media/alerts-activity-log/manage-alert-rules.png)

    可用規則清單隨即出現。

2. 搜尋要修改的活動記錄規則。

    ![ 搜尋活動記錄警示規則](media/alerts-activity-log/searth-activity-log-rule-to-edit.png)

    您可以使用可用的篩選條件 ([訂用帳戶][資源群組][資源][訊號類型] 或 [狀態]) 來尋找您想要編輯的活動規則。

    > [!NOTE]

    > 您只能編輯 [描述]、[目標準則] 與 [動作群組]。

3.  選取規則，然後按兩下以編輯規則選項。 進行所需的變更，然後按一下 [儲存]。

    ![ 管理警示規則](media/alerts-activity-log/activity-log-rule-edit-page.png)

4.  您可以停用、啟用或刪除規則。 選取規則之後，在視窗頂端選取適當選項，詳細資料如步驟 2 所示。


## <a name="azure-resource-template"></a>Azure 資源範本
若要使用 Resource Manager 範本建立活動記錄警示，您要建立 `microsoft.insights/activityLogAlerts` 類型的資源。 然後要填入所有相關的屬性。 以下是建立活動記錄警示的範本。

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "activityLogAlertName": {
      "type": "string",
      "metadata": {
        "description": "Unique name (within the Resource Group) for the Activity log alert."
      }
    },
    "activityLogAlertEnabled": {
      "type": "bool",
      "defaultValue": true,
      "metadata": {
        "description": "Indicates whether or not the alert is enabled."
      }
    },
    "actionGroupResourceId": {
      "type": "string",
      "metadata": {
        "description": "Resource Id for the Action group."
      }
    }
  },
  "resources": [   
    {
      "type": "Microsoft.Insights/activityLogAlerts",
      "apiVersion": "2017-04-01",
      "name": "[parameters('activityLogAlertName')]",      
      "location": "Global",
      "properties": {
        "enabled": "[parameters('activityLogAlertEnabled')]",
        "scopes": [
            "[subscription().id]"
        ],        
        "condition": {
          "allOf": [
            {
              "field": "category",
              "equals": "Administrative"
            },
            {
              "field": "operationName",
              "equals": "Microsoft.Resources/deployments/write"
            },
            {
              "field": "resourceType",
              "equals": "Microsoft.Resources/deployments"
            }
          ]
        },
        "actions": {
          "actionGroups":
          [
            {
              "actionGroupId": "[parameters('actionGroupResourceId')]"
            }
          ]
        }
      }
    }
  ]
}
```
基於本逐步解說的目的，上述 json 範例可儲存為 (假設) sampleActivityLogAlert.json，而且可以使用 [Azure 入口網站中的 Azure Resource Manager](../../azure-resource-manager/resource-group-template-deploy-portal.md) 進行部署。

> [!NOTE]
> 新活動記錄警示規則最多可能需要 5 分鐘的時間才會運作

## <a name="rest-api"></a>REST API 
[Azure 監視器 - 活動記錄警示 API](https://docs.microsoft.com/rest/api/monitor/activitylogalerts) 是 REST API，並且可與 Azure Resource Manager REST API 完全相容。 因此它可以透過 Powershell 和 Azure CLI 搭配資源管理員 Cmdlet 來使用。

## <a name="powershell"></a>PowerShell
以下說明透過 Azure Resource Manager PowerShell Cmdlet 使用上述[資源範本一節](#manage-alert-rules-for-activity-log-using-azure-resource-template)中資源範本範例 (sampleActivityLogAlert.json) 的方式：
```powershell
New-AzureRmResourceGroupDeployment -ResourceGroupName "myRG" -TemplateFile sampleActivityLogAlert.json -TemplateParameterFile sampleActivityLogAlert.parameters.json
```
其中，sampleActivityLogAlert.parameters.json 具有提供給警示規則建立所需參數的值。

## <a name="cli"></a>CLI
以下說明透過 Azure CLI 中的 Azure Resource Manager 命令使用上述[資源範本一節](#manage-alert-rules-for-activity-log-using-azure-resource-template)中資源範本範例 (sampleActivityLogAlert.json) 的方式：

```azurecli
az group deployment create --resource-group myRG --template-file sampleActivityLogAlert.json --parameters @sampleActivityLogAlert.parameters.json
```
*sampleActivityLogAlert.parameters.json* 檔案具有提供給警示規則建立所需參數的值。


## <a name="next-steps"></a>後續步驟

- [活動記錄的 Webhook 結構描述](../../azure-monitor/platform/activity-log-alerts-webhook.md)
- [活動記錄概觀](../../azure-monitor/platform/activity-log-alerts.md) 
- 深入了解[動作群組](../../azure-monitor/platform/action-groups.md)。  
- 深入了解[服務健康狀態通知](../../azure-monitor/platform/service-notifications.md)。
