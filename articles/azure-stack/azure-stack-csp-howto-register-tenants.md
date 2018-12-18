---
title: 將用於使用量與帳單的租用戶新增至 Azure Stack | Microsoft Docs
description: 必要步驟會將終端使用者新增至由雲端服務提供者 (CSP) 所管理的 Azure Stack 中。
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
ms.author: brenduns
ms.reviewer: alfredo
ms.openlocfilehash: d3fc3ef6c5fdcf5a87c691c73169ef2bec95805e
ms.sourcegitcommit: a3a0f42a166e2e71fa2ffe081f38a8bd8b1aeb7b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/01/2018
ms.locfileid: "43382683"
---
# <a name="add-tenant-for-usage-and-billing-to-azure-stack"></a>將用於使用量與帳單的租用戶新增至 Azure Stack

「適用於：Azure Stack 整合系統」

本文說明必要步驟會將終端使用者新增至雲端服務提供者 (CSP) 所管理的 Azure Stack。 當新的租用戶使用資源時，Azure Stack 將會向其 CSP 訂用帳戶報告使用量。

CSP 通常會為其 Azure Stack 部署上的多位終端客戶 (租用戶) 提供服務。 將租用戶新增到 Azure Stack 註冊可確保系統會報告每個租用戶的使用量，且費用會計入相對應的 CSP 訂用帳戶。 如果您未完成本文中的步驟，租用戶使用量費用會計入 Azure Stack 初始註冊所使用的訂用帳戶。 您必須先將 Azure Stack 設定為 CSP 後，才能將終端客戶新增至 Azure Stack，以進行使用量追蹤並管理其租用戶。 如需步驟和資源，請參閱[以雲端服務提供者身分管理 Azure Stack 的使用量和帳單](azure-stack-add-manage-billing-as-a-csp.md)。

下圖顯示 CSP 要讓新客戶能夠使用 Azure Stack 以及設定客戶使用量追蹤所必須遵循的步驟。 透過新增終端客戶，您也將能夠管理 Azure Stack 中的資源。 您有兩個選項可管理其資源：

1. 您可以保有終端客戶，並將本機 Azure Stack 訂用帳戶的認證提供給終端客戶。  
2. 或者終端客戶可在本機使用其訂用帳戶，並將 CSP 新增為具有擁有者權限的來賓。  

**新增終端客戶的步驟**

![設定雲端服務提供者以進行使用量追蹤，以及管理終端客戶帳戶](media\azure-stack-csp-enable-billing-usage-tracking\process-csp-enable-billing.png)

## <a name="create-a-new-customer-in-partner-center"></a>在合作夥伴中心建立新的客戶

在合作夥伴中心為客戶建立新的 Azure 訂用帳戶。 如需指示，請參閱[新增客戶](https://msdn.microsoft.com/partner-center/add-a-new-customer)。


##  <a name="create-an-azure-subscription-for-the-end-customer"></a>建立終端客戶的 Azure 訂用帳戶

在合作夥伴中心內建立客戶的記錄之後，您就可以向他們銷售目錄中產品的訂用帳戶。 如需指示，請參閱[建立、暫止或取消客戶的訂用帳戶](https://msdn.microsoft.com/partner-center/create-a-new-subscription)。

## <a name="create-a-guest-user-in-the-end-customer-directory"></a>在終端客戶目錄中建立來賓使用者

如果終端客戶會管理他們自己的帳戶，請在其目錄中建立來賓使用者，並將資訊傳送給他們。 接著，終端使用者會新增來賓，並將來賓權限提高為 Azure Stack CSP 帳戶的**擁有者**。
 
## <a name="update-the-registration-with-the-end-customer-subscription"></a>使用終端客戶訂用帳戶更新註冊

使用新客戶的訂用帳戶更新您的註冊。 Azure 會使用合作夥伴中心的客戶身分識別來報告客戶的使用量。 這個步驟可確保每個客戶的使用量都會報告在該客戶的個別 CSP 訂用帳戶底下。 這可讓您更輕鬆地追蹤使用者使用量與帳單。

> [!Note]  
> 若要執行此步驟，您必須[已註冊 Azure Stack](azure-stack-register.md)。

1. 使用提升權限提示字元開啟 Windows PowerShell，並執行：  
    `Add-AzureRmAccount`
2. 輸入您的 Azure 認證。
3. 在 PowerShell 工作階段中，執行：

```powershell
    New-AzureRmResource -ResourceId "subscriptions/{registrationSubscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.AzureStack/registrations/{registrationName}/customerSubscriptions/{customerSubscriptionId}" -ApiVersion 2017-06-01 -Properties <PSObject>
```
### <a name="new-azurermresource-powershell-parameters"></a>New-AzureRmResource PowerShell 參數
| 參數 | 說明 |
| --- | --- | 
|registrationSubscriptionID | 用於進行 Azure Stack 初始註冊的 Azure 訂用帳戶。 |
| customerSubscriptionID | 屬於要註冊之客戶的 Azure 訂用帳戶 (非 Azure Stack)。 必須建立在 CSP 供應項目中；實務上，這表示要透過合作夥伴中心。 如果客戶有多個 Azure Active Directory 租用戶，則必須將此訂用帳戶建立在要用來登入 Azure Stack 的租用戶中。
| resourceGroup | Azure 中用來儲存註冊的資源群組。 
| registrationName | 您 Azure Stack 註冊的名稱。 它是儲存在 Azure 中的物件。 | 
| properties | 指定資源的屬性。 使用此參數來指定資源類型特有的屬性值。


> [!Note]  
> 租用戶必須向他們所使用的每個 Azure Stack 註冊。 如果您有兩個 Azure Stack 部署，且租用戶要使用這兩個部署，則您必須使用租用戶訂用帳戶更新每個部署的初始註冊。

## <a name="onboard-tenant-to-azure-stack"></a>將租用戶上架到 Azure Stack

設定 Azure Stack，以支援來自多重 Azure AD 租用戶的使用者在 Azure Stack 中使用服務。 如需指示，請參閱[在 Azure Stack 中啟用多重租用](azure-stack-enable-multitenancy.md)。


## <a name="create-a-local-resource-in-the-end-customer-tenant-in-azure-stack"></a>在 Azure Stack 的終端客戶租用戶中建立本機資源

一旦您已將新客戶新增至 Azure Stack ，或是終端客戶租用戶已啟用您具有擁有者權限的來賓帳戶後，請確認您可以在其租用戶中建立資源。 例如，他們可以[使用 Azure Stack 入口網站建立 Windows 虛擬機器](user\azure-stack-quick-windows-portal.md)。

## <a name="next-steps"></a>後續步驟

 - 如果錯誤訊息在您的註冊過程中觸發，且您要檢閱這些錯誤訊息，請參閱[租用戶註冊錯誤訊息](azure-stack-csp-ref-infrastructure.md#usage-and-billing-error-codes)。
 - 若要深入了解如何取出 Azure Stack 的資源使用量資訊，請參閱 [Azure Stack 中的使用量與帳單](azure-stack-billing-and-chargeback.md)。
 - 若要檢閱終端客戶如何將身為 CSP 的您新增作為其 Azure Stack (租用戶) 的管理員，請參閱[讓雲端服務提供者能夠管理您的 Azure Stack 訂用帳戶](user\azure-stack-csp-enable-billing-usage-tracking.md)。
