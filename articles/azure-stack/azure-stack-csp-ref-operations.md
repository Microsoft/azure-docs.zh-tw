---
title: 註冊租用戶以追蹤 Azure Stack 中的使用量 | Microsoft Docs
description: 關於用來管理租用戶註冊之作業和 Azure Stack 中之租用戶使用量追蹤方式的詳細資料。
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila
editor: ''
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/12/2018
ms.author: mabrigg
ms.reviewer: alfredo
ms.openlocfilehash: 9396d49f455f8f4af1abf7f0020e95e8fd0a14cc
ms.sourcegitcommit: 1b561b77aa080416b094b6f41fce5b6a4721e7d5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/17/2018
ms.locfileid: "45729581"
---
# <a name="manage-tenant-registration-in-azure-stack"></a>管理 Azure Stack 中的租用戶註冊

「適用於：Azure Stack 整合系統」

本文包含您可用來管理租用戶註冊之作業以及租用戶使用量追蹤方式的詳細資料。 您可以找到有關如何新增、列出或移除租用戶對應的詳細資料。 您可以使用 PowerShell 或帳單 API 端點來管理您的使用追蹤。

## <a name="add-tenant-to-registration"></a>將租用戶新增至註冊

當您想要將新的租用戶新增至您的註冊時，請使用這項作業，以便在與 Azure Active Directory (Azure AD) 租用戶連線的 Azure 訂用帳戶下報告其使用量。

如果您想要變更與租用戶相關聯的訂用帳戶，也可以使用這項作業，且您可以再次呼叫 PUT/New-AzureRMResource。 系統會覆寫舊的對應。

請注意，您只能將一個 Azure 訂用帳戶與租用戶相關聯。 如果您嘗試將第二個訂用帳戶新增至現有租用戶，系統就會覆寫第一個訂用帳戶。 

### <a name="use-api-profiles"></a>使用 API 設定檔

本文中的 Cmdlet 會要求您在執行 PowerShell 時指定 API 設定檔。 API 設定檔代表一組 Azure 資源提供者和其 API 版本。 這些設定檔可協助您在與多個 Azure 雲端互動時使用正確的 API 版本，例如，在使用全域 Azure 和 Azure Stack 時。 設定檔的指定是透過與其發行日期相符的名稱來完成。 在本文中，您必須使用 **2017-09-03** 設定檔。

如需 Azure Stack 和 API 設定檔的詳細資訊，請參閱[管理 Azure Stack 中的 API 版本設定檔](user/azure-stack-version-profiles.md)。 如需有關在 PowerShell 中使用 API 設定檔來啟動並執行的指示，請參閱[在 Azure Stack 中使用適用於 PowerShell 的 API 版本設定檔](user/azure-stack-version-profiles-powershell.md)。

### <a name="parameters"></a>參數

| 參數                  | 說明 |
|---                         | --- |
| registrationSubscriptionID | 用於進行初始註冊的 Azure 訂用帳戶。 |
| customerSubscriptionID     | 屬於要註冊之客戶的 Azure 訂用帳戶 (非 Azure Stack)。 必須建立在雲端服務提供者 (CSP) 供應項目中。 實務上，這表示要透過合作夥伴中心。 如果客戶有多個租用戶，則必須將此訂用帳戶建立在要用來登入 Azure Stack 的租用戶中。 |
| resourceGroup              | Azure 中用來儲存註冊的資源群組。 |
| registrationName           | 您 Azure Stack 註冊的名稱。 它是儲存在 Azure 中的物件。 名稱的格式通常是 azurestack-CloudID，其中 CloudID 是您 Azure Stack 部署的雲端識別碼。 |

> [!Note]  
> 租用戶必須向他們所使用的每個 Azure Stack 註冊。 如果租用戶使用多個 Azure Stack，則您必須使用租用戶訂用帳戶更新每個部署的初始註冊。

### <a name="powershell"></a>PowerShell

使用 New-AzureRmResource Cmdlet 來更新註冊資源。 使用初始註冊所用的帳戶來登入 Azure (`Add-AzureRmAccount`)。 以下是如何新增租用戶的範例：

```powershell
  New-AzureRmResource -ResourceId "subscriptions/{registrationSubscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.AzureStack/registrations/{registrationName}/customerSubscriptions/{customerSubscriptionId}" -ApiVersion 2017-06-01 -Properties
```

### <a name="api-call"></a>API 呼叫

**作業**：PUT  
**RequestURI**：`subscriptions/{registrationSubscriptionId}/resourceGroups/{resourceGroup}  /providers/Microsoft.AzureStack/registrations/{registrationName}/customerSubscriptions/  
{customerSubscriptionId}?api-version=2017-06-01 HTTP/1.1`  
**回應**：201 已建立  
**回應本文**：空白  

## <a name="list-all-registered-tenants"></a>列出所有已註冊的租用戶

取得已新增至註冊的所有租用戶清單。

 > [!Note]  
 > 如果尚未註冊租用戶，您就不會收到回應。

### <a name="parameters"></a>參數

| 參數                  | 說明          |
|---                         | ---                  |
| registrationSubscriptionId | 用於進行初始註冊的 Azure 訂用帳戶。   |
| resourceGroup              | Azure 中用來儲存註冊的資源群組。    |
| registrationName           | 您 Azure Stack 註冊的名稱。 它是儲存在 Azure 中的物件。 名稱的格式通常是 **azurestack**-***CloudID***，其中 ***CloudID*** 是您 Azure Stack 部署的雲端識別碼。   |

### <a name="powershell"></a>PowerShell

使用 Get-AzureRmResource Cmdlet 來列出所有已註冊的租用戶。 使用初始註冊所用的帳戶來登入 Azure (`Add-AzureRmAccount`)。 以下是如何新增租用戶的範例：

```powershell
  Get-AzureRmResource -ResourceId "subscriptions/{registrationSubscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.AzureStack/registrations/{registrationName}/customerSubscriptions" -ApiVersion 2017-06-01
```

### <a name="api-call"></a>API 呼叫

您可以使用 GET 作業取得所有租用戶對應的清單

**作業**：GET  
**RequestURI**：`subscriptions/{registrationSubscriptionId}/resourceGroups/{resourceGroup}  
/providers/Microsoft.AzureStack/registrations/{registrationName}/customerSubscriptions?  
api-version=2017-06-01 HTTP/1.1`  
**回應**：200  
**回應本文**： 

```JSON  
{
    "value": [{
            "id": " subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.AzureStack/registrations/{registrationName}/customerSubscriptions/{ cspSubscriptionId 1}”,
            "name": " cspSubscriptionId 1",
            "type": “Microsoft.AzureStack\customerSubscriptions”,
            "properties": { "tenantId": "tId1" }
        },
        {
            "id": " subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.AzureStack/registrations/{registrationName}/customerSubscriptions/{ cspSubscriptionId 2}”,
            "name": " cspSubscriptionId2 ",
            "type": “Microsoft.AzureStack\customerSubscriptions”,
            "properties": { "tenantId": "tId2" }
        }
    ],
    "nextLink": "{originalRequestUrl}?$skipToken={opaqueString}"
}
```

## <a name="remove-a-tenant-mapping"></a>移除租用戶對應

您可以將已新增至註冊的租用戶移除。 如果該租用戶仍在使用 Azure Stack 上的資源，則會向初始 Azure Stack 註冊中使用的訂用帳戶收取其使用量的費用。

### <a name="parameters"></a>參數

| 參數                  | 說明          |
|---                         | ---                  |
| registrationSubscriptionId | 註冊的訂用帳戶識別碼。   |
| resourceGroup              | 註冊的資源群組。   |
| registrationName           | 註冊的名稱。  |
| customerSubscriptionId     | 客戶的訂用帳戶識別碼。  |

### <a name="powershell"></a>PowerShell

```powershell
  Remove-AzureRmResource -ResourceId "subscriptions/{registrationSubscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.AzureStack/registrations/{registrationName}/customerSubscriptions/{customerSubscriptionId}" -ApiVersion 2017-06-01
```

### <a name="api-call"></a>API 呼叫

您可以使用 DELETE 作業移除租用戶對應。

**作業**：DELETE  
**RequestURI**：`subscriptions/{registrationSubscriptionId}/resourceGroups/{resourceGroup}  
/providers/Microsoft.AzureStack/registrations/{registrationName}/customerSubscriptions/  
{customerSubscriptionId}?api-version=2017-06-01 HTTP/1.1`  
**回應**：204 沒有內容  
**回應本文**：空白

## <a name="next-steps"></a>後續步驟

 - 若要深入了解如何取出 Azure Stack 的資源使用量資訊，請參閱 [Azure Stack 中的使用量與帳單](azure-stack-billing-and-chargeback.md)。
