---
title: 使用 Azure Resource Manager 進行的部署作業 | Microsoft Docs
description: 說明如何使用入口網站、PowerShell、Azure CLI 及 REST API 來檢視 Azure Resource Manager 部署作業。
services: azure-resource-manager,virtual-machines
documentationcenter: ''
tags: top-support-issue
author: tfitzmac
ms.assetid: ''
ms.service: azure-resource-manager
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: vm-multiple
ms.workload: infrastructure
ms.date: 09/28/2018
ms.author: tomfitz
ms.openlocfilehash: 9320e3089e02e1ca6b6bcce0287946baaf0558d9
ms.sourcegitcommit: f31bfb398430ed7d66a85c7ca1f1cc9943656678
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47452032"
---
# <a name="view-deployment-operations-with-azure-resource-manager"></a>使用 Azure Resource Manager 來檢視部署作業

您可以透過 Azure 入口網站檢視部署的作業。 當您在部署期間收到錯誤時，您可能對於檢視作業最感興趣，所以此文章著重於檢視失敗的作業。 入口網站提供介面，讓您輕鬆地找到錯誤並判斷可能的修正方法。

您可以透過查看稽核記錄檔或部署作業來針對您的部署進行疑難排解。 此文章示範這兩種方法。 如需解決特定部署錯誤的說明，請參閱 [針對使用 Azure Resource Manager 將資源部署至 Azure 時常見的錯誤進行疑難排解](resource-manager-common-deployment-errors.md)。

## <a name="portal"></a>入口網站
若要查看部署作業，請使用下列步驟 ︰

1. 針對部署所含的資源群組，請注意最後一個部署的狀態。 您可以選取此狀態，以取得更多詳細資料。
   
    ![deployment status](./media/resource-manager-deployment-operations/deployment-status.png)
2. 您會看到最近的部署歷程記錄。 選取失敗的部署。
   
    ![deployment status](./media/resource-manager-deployment-operations/select-deployment.png)
3. 選取連結以查看部署失敗的原因描述。 在下圖中，DNS 記錄不是唯一的。  
   
    ![檢視失敗的部署](./media/resource-manager-deployment-operations/view-error.png)
   
    此錯誤訊息應足以讓您開始進行疑難排解。 不過，如果您需要有關哪些工作已完成的詳細資料，您可以檢視如下列步驟中所示的作業。
4. 您可以檢視所有的部署作業。 選取任一作業以查看詳細資料。
   
    ![檢視作業](./media/resource-manager-deployment-operations/view-operations.png)
   
    在此情況下，您會看到儲存體帳戶、虛擬網路和可用性設定組都已成功建立。 公用 IP 位址失敗，並未嘗試其他資源。
5. 您可以選取 [事件] ，以檢視部署的事件。
   
    ![檢視事件](./media/resource-manager-deployment-operations/view-events.png)
6. 您會看見部署的所有事件，選取任何一個事件即可取得詳細資料。 請注意相互關聯識別碼。 與技術支援人員合作來排解部署問題時，此值會相當有用。
   
    ![查看事件](./media/resource-manager-deployment-operations/see-all-events.png)

## <a name="powershell"></a>PowerShell
1. 若要取得部署的整體狀態，請使用 **Get-AzureRmResourceGroupDeployment** 命令。 

  ```powershell
  Get-AzureRmResourceGroupDeployment -ResourceGroupName ExampleGroup
  ```

   或者，您也可以篩選結果，只找出失敗的部署。

  ```powershell
  Get-AzureRmResourceGroupDeployment -ResourceGroupName ExampleGroup | Where-Object ProvisioningState -eq Failed
  ```
   
1. 若要取得相互關連識別碼，請使用：

  ```powershell
  (Get-AzureRmResourceGroupDeployment -ResourceGroupName ExampleGroup -DeploymentName azuredeploy).CorrelationId
  ```

1. 每個部署都包括多個作業。 每個作業皆代表部署程序中的一個步驟。 若要探索部署有何問題，您通常需要查看有關部署作業的詳細資訊。 您可以利用 **Get-AzureRmResourceGroupDeploymentOperation**查看作業的狀態。

  ```powershell 
  Get-AzureRmResourceGroupDeploymentOperation -ResourceGroupName ExampleGroup -DeploymentName vmDeployment
  ```

    以下列格式傳回多個作業︰

  ```powershell
  Id             : /subscriptions/{guid}/resourceGroups/ExampleGroup/providers/Microsoft.Resources/deployments/Microsoft.Template/operations/A3EB2DA598E0A780
  OperationId    : A3EB2DA598E0A780
  Properties     : @{provisioningOperation=Create; provisioningState=Succeeded; timestamp=2016-06-14T21:55:15.0156208Z;
                   duration=PT23.0227078S; trackingId=11d376e8-5d6d-4da8-847e-6f23c6443fbf;
                   serviceRequestId=0196828d-8559-4bf6-b6b8-8b9057cb0e23; statusCode=OK; targetResource=}
  PropertiesText : {duration:PT23.0227078S, provisioningOperation:Create, provisioningState:Succeeded,
                   serviceRequestId:0196828d-8559-4bf6-b6b8-8b9057cb0e23...}
  ```

1. 若要取得有關失敗作業的詳細資訊，請擷取具有 [失敗]  狀態的作業屬性。

  ```powershell
  (Get-AzureRmResourceGroupDeploymentOperation -DeploymentName Microsoft.Template -ResourceGroupName ExampleGroup).Properties | Where-Object ProvisioningState -eq Failed
  ```
   
    這會以下列格式傳回所有失敗的作業︰

  ```powershell
  provisioningOperation : Create
  provisioningState     : Failed
  timestamp             : 2016-06-14T21:54:55.1468068Z
  duration              : PT3.1449887S
  trackingId            : f4ed72f8-4203-43dc-958a-15d041e8c233
  serviceRequestId      : a426f689-5d5a-448d-a2f0-9784d14c900a
  statusCode            : BadRequest
  statusMessage         : @{error=}
  targetResource        : @{id=/subscriptions/{guid}/resourceGroups/ExampleGroup/providers/
                          Microsoft.Network/publicIPAddresses/myPublicIP;
                          resourceType=Microsoft.Network/publicIPAddresses; resourceName=myPublicIP}
  ```

    請記下此作業的 serviceRequestId 和 trackingId。 與技術支援人員合作來排解部署問題時，serviceRequestId 會相當有用。 您將在下一個步驟中運用 trackingId 將重點放在特定的作業。
1. 若要取得特定失敗作業的狀態訊息，請使用下列命令︰

  ```powershell
  ((Get-AzureRmResourceGroupDeploymentOperation -DeploymentName Microsoft.Template -ResourceGroupName ExampleGroup).Properties | Where-Object trackingId -eq f4ed72f8-4203-43dc-958a-15d041e8c233).StatusMessage.error
  ```

    它會傳回：

  ```powershell
  code           message                                                                        details
  ----           -------                                                                        -------
  DnsRecordInUse DNS record dns.westus.cloudapp.azure.com is already used by another public IP. {}
  ```
1. Azure 中的每個部署作業包含要求和回應內容。 要求內容是部署期間您傳送至 Azure 的內容 (例如，建立 VM、作業系統磁碟和其他資源)。 回應內容是 Azure 從您的部署要求傳送回來的內容。 部署期間，您可以使用 **DeploymentDebugLogLevel** 參數來指定要求和/或回應會保留在記錄檔中。 

  您會使用下列 PowerShell 命令從記錄檔取得該資訊，並將它儲存在本機︰

  ```powershell
  (Get-AzureRmResourceGroupDeploymentOperation -DeploymentName "TestDeployment" -ResourceGroupName "Test-RG").Properties.request | ConvertTo-Json |  Out-File -FilePath <PathToFile>

  (Get-AzureRmResourceGroupDeploymentOperation -DeploymentName "TestDeployment" -ResourceGroupName "Test-RG").Properties.response | ConvertTo-Json |  Out-File -FilePath <PathToFile>
  ```

## <a name="azure-cli"></a>Azure CLI

1. 利用 **azure group deployment show** 命令取得部署的整體狀態。

  ```azurecli
  az group deployment show -g ExampleGroup -n ExampleDeployment
  ```
  
1. 其中一個傳回的值是 **correlationId**。 此值可用來追蹤相關的事件，並且在與技術支援人員合作來排解部署問題時會相當有用。

  ```azurecli
  az group deployment show -g ExampleGroup -n ExampleDeployment --query properties.correlationId
  ```

1. 若要查看某個部署的作業，請使用：

  ```azurecli
  az group deployment operation list -g ExampleGroup -n ExampleDeployment
  ```

## <a name="rest"></a>REST

1. 利用 [取得範本部署的相關資訊](https://docs.microsoft.com/rest/api/resources/deployments#Deployments_Get) 作業，來取得部署相關資訊。

  ```http
  GET https://management.azure.com/subscriptions/{subscription-id}/resourcegroups/{resource-group-name}/providers/microsoft.resources/deployments/{deployment-name}?api-version={api-version}
  ```

    在回應中，請特別注意 **provisioningState**、**correlationId** 和 **error** 元素。 **correlationId** 可用來追蹤相關的事件，並且在與技術支援人員合作來排解部署問題時會相當有用。

  ```json
  { 
    ...
    "properties": {
      "provisioningState":"Failed",
      "correlationId":"d5062e45-6e9f-4fd3-a0a0-6b2c56b15757",
      ...
      "error":{
        "code":"DeploymentFailed","message":"At least one resource deployment operation failed. Please list deployment operations for details. Please see http://aka.ms/arm-debug for usage details.",
        "details":[{"code":"Conflict","message":"{\r\n  \"error\": {\r\n    \"message\": \"Conflict\",\r\n    \"code\": \"Conflict\"\r\n  }\r\n}"}]
      }  
    }
  }
  ```

2. 使用[列出所有範本部署作業](https://docs.microsoft.com/rest/api/resources/deployments#Deployments_List) \(英文\) ，來取得部署的相關資訊。 

  ```http
  GET https://management.azure.com/subscriptions/{subscription-id}/resourcegroups/{resource-group-name}/providers/microsoft.resources/deployments/{deployment-name}/operations?$skiptoken={skiptoken}&api-version={api-version}
  ```
   
    回應會根據您在部署期間於 **debugSetting** 屬性中指定的內容，來包含要求和 (或) 回應資訊。

  ```json
  {
    ...
    "properties": 
    {
      ...
      "request":{
        "content":{
          "location":"West US",
          "properties":{
            "accountType": "Standard_LRS"
          }
        }
      },
      "response":{
        "content":{
          "error":{
            "message":"Conflict","code":"Conflict"
          }
        }
      }
    }
  }
  ```


## <a name="next-steps"></a>後續步驟
* 如需解決特定部署錯誤的說明，請參閱 [針對使用 Azure Resource Manager 將資源部署至 Azure 時常見的錯誤進行疑難排解](resource-manager-common-deployment-errors.md)。
* 若要了解如何使用活動記錄來監視其他類型的動作，請參閱[檢視活動記錄以管理 Azure 資源](resource-group-audit.md)。
* 若要在執行之前驗證您的部署，請參閱 [使用 Azure Resource Manager 範本部署資源群組](resource-group-template-deploy.md)。

