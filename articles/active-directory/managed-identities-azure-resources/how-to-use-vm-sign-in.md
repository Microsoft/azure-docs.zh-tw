---
title: 如何使用 Azure VM 的 Azure 資源受控識別進行登入
description: 使用 Azure 虛擬機器 Azure 資源的受控識別服務主體來進行指令碼用戶端登入和資源存取的逐步指示與範例。
services: active-directory
documentationcenter: ''
author: daveba
manager: mtillman
editor: ''
ms.service: active-directory
ms.component: msi
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 12/01/2017
ms.author: daveba
ms.openlocfilehash: bf363e4b03fb604e1b9af0d30b6e4ac471a41821
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46980282"
---
# <a name="how-to-use-managed-identities-for-azure-resources-on-an-azure-vm-for-sign-in"></a>如何使用 Azure VM 的 Azure 資源受控識別進行登入 

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]  
本文提供使用 Azure 資源的受控識別服務主體進行登入的 PowerShell 和 CLI 指令碼範例，以及錯誤處理等重要主題的指引。

## <a name="prerequisites"></a>必要條件

[!INCLUDE [msi-qs-configure-prereqs](../../../includes/active-directory-msi-qs-configure-prereqs.md)]

如果您打算使用本文中的 Azure PowerShell 或 Azure CLI 範例，請務必安裝最新版的 [Azure PowerShell](https://www.powershellgallery.com/packages/AzureRM) 或 [Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli)。 

> [!IMPORTANT]
> - 本文中的所有範例指令碼都假設命令列用戶端在已啟用 Azure 資源受控識別的虛擬機器上執行。 在 Azure 入口網站中使用虛擬機器「連線」功能，從遠端連線到您的虛擬機器。 如需有關在虛擬機器上啟用 Azure 資源受控識別的詳細資訊，請參閱[使用 Azure 入口網站在虛擬機器上設定 Azure 資源受控識別](qs-configure-portal-windows-vm.md)，或其中一篇變化文章 (使用 PowerShell、CLI、範本或 Azure SDK)。 
> - 若要避免資源存取期間發生錯誤，虛擬機器的受控識別必須至少取得適當範圍 (該虛擬機器或更高層級) 的「讀者」存取權，以允許在虛擬機器上進行 Azure Resource Manager 作業。 如需詳細資訊，請參閱[使用 Azure 入口網站將 Azure 資源的受控識別指派給資源](howto-assign-access-portal.md)。

## <a name="overview"></a>概觀

Azure 資源的受控識別提供[服務主體物件](../develop/developer-glossary.md#service-principal-object)，這是[啟用 Azure 資源的受控識別時建立](overview.md#how-does-it-work)的物件。 服務主體可以獲得 Azure 資源的存取權，並可讓指令碼/命令列用戶端用來作為登入及存取資源時的身分識別。 傳統上，若要以受保護資源本身的身分識別來存取該資源，指令碼用戶端需要：  

   - 向 Azure AD 註冊並獲得同意成為機密/Web 用戶端應用程式
   - 在其服務主體下登入，並使用應用程式的認證 (這可能會內嵌於指令碼中)

使用 Azure 資源的受控識別，您的指令碼用戶端不再需要執行任一項，因為它可以在 Azure 資源受控識別的服務主體下登入。 

## <a name="azure-cli"></a>Azure CLI

下列指令碼示範如何：

1. 在虛擬機器的 Azure 資源受控識別服務主體下登入 Azure AD  
2. 呼叫 Azure Resource Manager 並取得虛擬機器的服務主體識別碼。 CLI 會自動為您管理權杖的擷取/使用。 務必以您的虛擬機器名稱取代 `<VM-NAME>`。  

   ```azurecli
   az login --identity
   
   spID=$(az resource list -n <VM-NAME> --query [*].identity.principalId --out tsv)
   echo The managed identity for Azure resources service principal ID is $spID
   ```

## <a name="azure-powershell"></a>Azure PowerShell

下列指令碼示範如何：

1. 在虛擬機器的 Azure 資源受控識別服務主體下登入 Azure AD  
2. 呼叫 Azure Resource Manager Cmdlet 以取得虛擬機器的相關資訊。 PowerShell 會自動為您管理權杖的使用。  

   ```azurepowershell
   Add-AzureRmAccount -identity

   # Call Azure Resource Manager to get the service principal ID for the VM's managed identity for Azure resources. 
   $vmInfoPs = Get-AzureRMVM -ResourceGroupName <RESOURCE-GROUP> -Name <VM-NAME>
   $spID = $vmInfoPs.Identity.PrincipalId
   echo "The managed identity for Azure resources service principal ID is $spID"
   ```

## <a name="resource-ids-for-azure-services"></a>Azure 服務的資源識別碼

關於支援 Azure AD 且經過 Azure 資源受控識別所測試的資源及其各自資源識別碼的清單，請參閱[支援 Azure AD 驗證的 Azure 服務](services-support-msi.md#azure-services-that-support-azure-ad-authentication)。

## <a name="error-handling-guidance"></a>錯誤處理指引 

以下所示的回應可能代表虛擬機器的 Azure 資源受控識別未正確設定：

- PowerShell：Invoke-WebRequest：無法連線到遠端伺服器
- CLI：MSI：無法從 'http://localhost:50342/oauth2/token' 擷取權杖，包含 HTTPConnectionPool(主機='localhost'，連接埠=50342) 錯誤 

如果您收到這些錯誤的其中一個，請返回 [Azure 入口網站](https://portal.azure.com)中的 Azure 虛擬機器，並且：

- 移至 [身分識別] 頁面，並確保 [系統指派] 設為 [是]。
- 移至 [延伸模組] 頁面，並確定已成功部署適用於 Azure 資源的受控識別延伸模組 **(已計劃在 2019 年 1 月淘汰)**。

如果其中一者不正確，您可能需要再次在資源上重新部署 Azure 資源受控識別，或針對部署失敗進行疑難排解。 如果您需要設定虛擬機器的協助，請參閱[使用 Azure 入口網站設定虛擬機器的 Azure 資源受控識別](qs-configure-portal-windows-vm.md)。

## <a name="next-steps"></a>後續步驟

- 若要啟用 Azure VM 的 Azure 資源受控識別，請參閱[使用 PowerShell 在 Azure VM 上設定 Azure 資源受控識別](qs-configure-powershell-windows-vm.md)或[使用 Azure CLI 在 Azure VM 上設定 Azure 資源受控識別](qs-configure-cli-windows-vm.md)






