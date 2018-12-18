---
title: 使用 Azure REST API 檢閱資訊安全中心原則合規性 | Microsoft Docs
description: 了解如何使用 Azure REST API 檢閱資訊安全中心原則的目前合規性。
services: security-center
documentationcenter: na
author: lleonard-msft
manager: MBaldwin
editor: ''
ms.assetid: 82D50B98-40F2-44B1-A445-4391EA9EBBAA
ms.service: security-center
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/05/2017
ms.author: alleonar
ms.openlocfilehash: 1443486590859aac5591aff2ab0551bed9228d7b
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44301744"
---
# <a name="review-security-center-policy-compliance-using-rest-apis"></a>使用 REST API 檢閱資訊安全中心原則合規性

資訊安全中心會根據您定義的安全性原則，定期驗證您的 Azure 資源。 資訊安全中心也會提供 REST API，讓您檢閱自己的應用程式合規性；您可以直接查詢服務，或將 JSON 結果匯入其他應用程式。 

在本文中，您將了解如何從與某個訂用帳戶建立關聯的所有 Azure 資源，擷取目前的建議集。

擷取目前的建議集：
``` http
GET https://management.azure.com/subscriptions/{subscription-id}/providers/Microsoft.Security/tasks?api-version={api-version}
Content-Type: application/json   
Authorization: Bearer
```

## <a name="build-the-request"></a>建立要求  

`{subscription-id}` 是必要參數，而且應該包含定義原則之 Azure 訂用帳戶的訂用帳戶識別碼。 如果您有多個訂用帳戶，請參閱[使用多個訂用帳戶](https://docs.microsoft.com/cli/azure/manage-azure-subscriptions-azure-cli?view=azure-cli-latest#working-with-multiple-subscriptions)。  

`api-version` 是必要參數。 目前只有 `api-version=2015-06-01-preview` 支援這些端點。 

以下是必要標頭： 

|要求標頭|說明|  
|--------------------|-----------------|  
|*Content-Type:*|必要。 設定為 `application/json`。|  
|*Authorization:*|必要。 設定為無效的 `Bearer` [存取權杖](https://docs.microsoft.com/rest/api/azure/#authorization-code-grant-interactive-clients)。 |  

## <a name="response"></a>Response  

傳回狀態碼 200 (確定) 表示成功回應，其中包含用來保護您 Azure 資源的建議工作清單。

``` json
{  
  "value": [  
    {  
       "id": "/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.Security/locations/{region}/tasks/{task-id}",
       "name": "{task_id}",
       "type": "Microsoft.Security/locations/{region}/tasks",
       "properties": {
       "state": "Active",
       "subState": "NA",
       "creationTimeUtc": "{create-time}",
       "lastStateChangeTimeUtc": "{last-state-change}",
       "securityTaskParameters": "{security-task-properties}"
    } 
  ]  
}  
```  

**value** 中的每個項目代表一個建議：

|回應屬性|說明|
|----------------|----------|
|**state** | 指出建議為 `active` 或 `resolved`。 |
|**creationTimeUtc** | 以 UTC 表示的日期和時間，顯示建議的建立時間。 |
|**lastStateChangeUtc** | 上次狀態變更的日期和時間，以 UTV 表示 (如果有的話)。 |
|**securityTaskParameters** | 建議的詳細資料；屬性會根據基礎建議而有所不同。 |
||
  
如需目前支援的建議，請參閱[實作安全性建議](https://docs.microsoft.com/azure/security-center/security-center-recommendations)。

其他狀態碼表示錯誤狀況。 在這些情況下，回應物件會包含說明要求為何失敗的描述。

``` json
{  
  "value": [  
    {  
      "description": "Error response describing why the operation failed."  
    }  
  ]  
}  
```  

## <a name="example-response"></a>範例回應  

``` json
{  
  "value": [  
        {
            "id": "/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.Security/locations/{region}/tasks/{task-id}",
            "name": "{task_id}",
            "type": "Microsoft.Security/locations/{region}/tasks",
            "properties": {
                "state": "Active",
                "subState": "NA",
                "creationTimeUtc": "{create-time}",
                "lastStateChangeTimeUtc": "{last-state-change}",
                "securityTaskParameters": {
                    "vmId": "/subscriptions/{subscription-id}/resourceGroups/{resource_group}/providers/Microsoft.Compute/virtualMachines/{vm_name}",
                    "vmName": "{vm_name}",
                    "severity": "{severity}",
                    "isOsDiskEncrypted": {is_os_disk_encrypted},
                    "isDataDiskEncrypted": {is_data_disk_encrypted},
                    "name": "EncryptionOnVm",
                    "uniqueKey": "EncryptionOnVmTaskParameters_/subscriptions/{subscription-id}/resourceGroups/{resoource_group}/providers/Microsoft.Compute/virtualMachines/{vm_name}",
                    "resourceId": "/subscriptions/{subscription_id}/resourceGroups/{resource_group}/providers/Microsoft.Compute/virtualMachines/{vm_name}"
                }
            }
        },
        {
            "id": "/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.Security/locations/{location}/tasks/{task-id}",
            "name": "{task-id}",
            "type": "Microsoft.Security/locations/{region}/tasks",
            "properties": {
                "state": "Active",
                "subState": "NA",
                "creationTimeUtc": "{create-time}",
                "lastStateChangeTimeUtc": "{last-state-change}",
                "securityTaskParameters": {
                    "serverName": "{sql-server-name}",
                    "serverId": "/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.Sql/servers/{server-id}",
                    "name": "Enable auditing for the SQL server",
                    "uniqueKey": "/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.Sql/servers/{server-id}/auditingPolicies/Default",
                    "resourceId": "/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.Sql/servers/{server-id}"
                }
            }
        }  ]  
}  
```  

此回應顯示兩個建議；清單中的每個項目會對應至特定建議。 第一個建議對 Linux 虛擬機器上的儲存體加密，第二個建議您啟用 SQL Server 的稽核功能。

建議會根據您已啟用的原則而有所不同。 如需詳細資訊，包括目前可用的建議，請參閱[管理 Azure 資訊安全中心的安全性建議](https://docs.microsoft.com/azure/security-center/security-center-recommendations)。


## <a name="see-also"></a>另請參閱  
- [設定安全性原則](https://docs.microsoft.com/azure/security-center/security-center-policies-overview)
- [Azure Security Resource provider REST API](https://msdn.microsoft.com/library/azure/mt704034.aspx) (Azure 資訊安全資源提供者 REST API)   
- [Get started with Azure REST API](https://docs.microsoft.com/rest/api/azure/) (開始使用 Azure REST API)   
- [Azure 資訊安全中心 PowerShell 模組](https://www.powershellgallery.com/packages/Azure-Security-Center/0.0.22)
