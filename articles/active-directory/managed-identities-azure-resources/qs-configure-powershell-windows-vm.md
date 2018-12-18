---
title: 如何使用 PowerShell 在 Azure VM 上設定 Azure 資源受控識別
description: 使用 PowerShell 在 Azure VM 上設定 Azure 資源受控識別的逐步指示。
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
ms.date: 11/27/2017
ms.author: daveba
ms.openlocfilehash: e8d85144b89d81e67d5ac225f0b6467230608ce0
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47106299"
---
# <a name="configure-managed-identities-for-azure-resources-on-an-azure-vm-using-powershell"></a>使用 PowerShell 在 Azure VM 上設定 Azure 資源受控識別

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

Azure 資源受控識別會在 Azure Active Directory 中為 Azure 服務提供自動受控識別。 您可以使用此身分識別來向任何支援 Azure AD 驗證的服務進行驗證，不需要任何您程式碼中的認證。 

在本文中，藉由使用 PowerShell，您將了解如何在 Azure VM 上執行下列 Azure 資源受控識別作業：

## <a name="prerequisites"></a>必要條件

- 如果您不熟悉 Azure 資源的受控識別，請參閱[概觀一節](overview.md)。 **請務必檢閱[系統指派和使用者指派受控識別之間的差異](overview.md#how-does-it-work)**。
- 如果您還沒有 Azure 帳戶，請先[註冊免費帳戶](https://azure.microsoft.com/free/)，再繼續進行。
- 若要執行本文中的管理作業，您的帳戶需要下列 Azure 角色型存取控制指派：

    > [!NOTE]
    > 不需要其他 Azure AD 目錄角色指派。

    - [虛擬機器參與者](/azure/role-based-access-control/built-in-roles#virtual-machine-contributor)，可建立 VM，並從 Azure VM 啟用和移除系統和/或使用者指派的受控識別。
    - [受控識別參與者](/azure/role-based-access-control/built-in-roles#managed-identity-contributor)角色，可建立使用者指派的受控識別。
    - [受控識別操作員](/azure/role-based-access-control/built-in-roles#managed-identity-operator)角色，可為虛擬機器指派和移除使用者指派的受控識別。
- 如果您尚未安裝[最新版的 Azure PowerShell](https://www.powershellgallery.com/packages/AzureRM)，請先安裝。

## <a name="system-assigned-managed-identity"></a>系統指派的受控識別

在本節中，您會學習如何使用 Azure PowerShell 啟用和停用系統指派的受控識別。

### <a name="enable-system-assigned-managed-identity-during-creation-of-an-azure-vm"></a>在建立 Azure VM 時，啟用系統指派的受控識別

若要建立已啟用系統指派受控識別的 Azure VM：

1. 請參閱下列 Azure VM 快速入門，完成必要的章節 (「登入 Azure」、「建立資源群組」、「建立網路群組」、「建立VM」)。
    
    當您取得「建立VM」一節時，請稍微修改一下 [New-AzureRmVMConfig](/powershell/module/azurerm.compute/new-azurermvm) Cmdlet 語法。 請務必新增 `-AssignIdentity:$SystemAssigned` 參數，以佈建已啟用系統所指派身分識別的 VM，例如：
      
    ```powershell
    $vmConfig = New-AzureRmVMConfig -VMName myVM -AssignIdentity:$SystemAssigned ...
    ```

   - [使用 PowerShell 建立 Windows 虛擬機器](../../virtual-machines/windows/quick-create-powershell.md)
   - [使用 PowerShell 建立 Linux 虛擬機器](../../virtual-machines/linux/quick-create-powershell.md)

2. (選擇性) 在 [Set-AzureRmVMExtension](/powershell/module/azurerm.compute/set-azurermvmextension) Cmdlet 上使用 `-Type` 參數來新增 Azure 資源受控識別 VM 擴充功能 (已計劃在 2019 年 1 月淘汰)。 您可以傳遞「ManagedIdentityExtensionForWindows」或「ManagedIdentityExtensionForLinux」(取決於 VM 的類型)，並使用 `-Name` 參數為其命名。 `-Settings` 參數會指定 OAuth 權杖端點所使用的連接埠，以用來取得權杖：

   ```powershell
   $settings = @{ "port" = 50342 }
   Set-AzureRmVMExtension -ResourceGroupName myResourceGroup -Location WestUS -VMName myVM -Name "ManagedIdentityExtensionForWindows" -Type "ManagedIdentityExtensionForWindows" -Publisher "Microsoft.ManagedIdentity" -TypeHandlerVersion "1.0" -Settings $settings 
   ```
    > [!NOTE]
    > 此步驟是選擇性的，因為您也可以使用 Azure Instance Metadata Service (IMDS) 識別端點以擷取權杖。 Azure 資源受控識別 VM 擴充功能已計劃在 2019 年 1 月淘汰。 

### <a name="enable-system-assigned-managed-identity-on-an-existing-azure-vm"></a>在現有 Azure VM 上啟用系統指派的受控識別

如果您需要在現有 Azure 虛擬機器上啟用系統指派的受控識別：

1. 使用 `Login-AzureRmAccount` 登入 Azure。 使用與包含虛擬機器的 Azure 訂用帳戶相關聯的帳戶。

   ```powershell
   Login-AzureRmAccount
   ```

2. 先使用 `Get-AzureRmVM` Cmdlet 擷取 VM 屬性。 然後在 [Update-AzureRmVM](/powershell/module/azurerm.compute/update-azurermvm) Cmdlet 上使用 `-AssignIdentity` 參數來啟用系統指派的受控識別：

   ```powershell
   $vm = Get-AzureRmVM -ResourceGroupName myResourceGroup -Name myVM
   Update-AzureRmVM -ResourceGroupName myResourceGroup -VM $vm -AssignIdentity:$SystemAssigned
   ```

3. (選擇性) 在 [Set-AzureRmVMExtension](/powershell/module/azurerm.compute/set-azurermvmextension) Cmdlet 上使用 `-Type` 參數來新增 Azure 資源受控識別 VM 擴充功能 (已計劃在 2019 年 1 月淘汰)。 您可以傳遞「ManagedIdentityExtensionForWindows」或「ManagedIdentityExtensionForLinux」(取決於 VM 的類型)，並使用 `-Name` 參數為其命名。 `-Settings` 參數會指定 OAuth 權杖端點所使用的連接埠，以用來取得權杖。 請比對現有 VM 位置，確認指定正確的 `-Location` 參數：

   ```powershell
   $settings = @{ "port" = 50342 }
   Set-AzureRmVMExtension -ResourceGroupName myResourceGroup -Location WestUS -VMName myVM -Name "ManagedIdentityExtensionForWindows" -Type "ManagedIdentityExtensionForWindows" -Publisher "Microsoft.ManagedIdentity" -TypeHandlerVersion "1.0" -Settings $settings 
   ```
    > [!NOTE]
    > 此步驟是選擇性的，因為您也可以使用 Azure Instance Metadata Service (IMDS) 識別端點以擷取權杖。

## <a name="disable-system-assigned-managed-identity-from-an-azure-vm"></a>停用來自 Azure VM 的系統指派受控識別

如果您的虛擬機器已不再需要系統指派的受控識別，但仍需要使用者指派的受控識別，請使用下列 Cmdlet：

1. 使用 `Login-AzureRmAccount` 登入 Azure。 使用與包含虛擬機器的 Azure 訂用帳戶相關聯的帳戶。

   ```powershell
   Login-AzureRmAccount
   ```

2. 使用 `Get-AzureRmVM` Cmdlet 來擷取 VM 屬性，並將 `-IdentityType` 參數設定為 `UserAssigned`：

   ```powershell   
   $vm = Get-AzureRmVM -ResourceGroupName myResourceGroup -Name myVM    
   Update-AzureRmVm -ResourceGroupName myResourceGroup -VM $vm -IdentityType "UserAssigned"
   ```

如果您的虛擬機器不再需要系統指派的受控識別，而且沒有使用者指派的受控識別，請使用下列命令：

```powershell
$vm = Get-AzureRmVM -ResourceGroupName myResourceGroup -Name myVM
Update-AzureRmVm -ResourceGroupName myResourceGroup -VM $vm -IdentityType None
```

若要移除 Azure 資源受控識別 VM 擴充功能，請使用 -Name 參數搭配 [Remove-AzureRmVMExtension](/powershell/module/azurerm.compute/remove-azurermvmextension) Cmdlet，並指定新增擴充功能時所使用的相同名稱：

   ```powershell
   Remove-AzureRmVMExtension -ResourceGroupName myResourceGroup -Name "ManagedIdentityExtensionForWindows" -VMName myVM
   ```

## <a name="user-assigned-managed-identity"></a>使用者指派的受控識別

本節中，您將了解如何使用 Azure PowerShell 在 VM 中新增和移除使用者指派的受控識別。

### <a name="assign-a-user-assigned-managed-identity-to-a-vm-during-creation"></a>在 VM 建立期間，將使用者指派的受控識別指派給 VM

若要在建立 VM 時，將使用者指派的受控識別指派給 Azure VM：

1. 請參閱下列 Azure VM 快速入門，完成必要的章節 (「登入 Azure」、「建立資源群組」、「建立網路群組」、「建立VM」)。 
  
    當您參閱「建立 VM」一節時，請稍微修改一下 [`New-AzureRmVMConfig`](/powershell/module/azurerm.compute/new-azurermvm)Cmdlet 語法。 新增 `-IdentityType UserAssigned` 和 `-IdentityID ` 參數，以佈建具有使用者所指派身分識別的 VM。  以您自己的值取代 `<VM NAME>`、`<SUBSCRIPTION ID>`、`<RESROURCE GROUP>` 和 `<USER ASSIGNED IDENTITY NAME>`。  例如︰
    
    ```powershell 
    $vmConfig = New-AzureRmVMConfig -VMName <VM NAME> -IdentityType UserAssigned -IdentityID "/subscriptions/<SUBSCRIPTION ID>/resourcegroups/<RESROURCE GROUP>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<USER ASSIGNED IDENTITY NAME>..."
    ```
    
    - [使用 PowerShell 建立 Windows 虛擬機器](../../virtual-machines/windows/quick-create-powershell.md)
    - [使用 PowerShell 建立 Linux 虛擬機器](../../virtual-machines/linux/quick-create-powershell.md)

2. (選擇性) 在 [Set-AzureRmVMExtension](/powershell/module/azurerm.compute/set-azurermvmextension) Cmdlet 上使用 `-Type` 參數來新增 Azure 資源受控識別 VM 擴充功能。 您可以傳遞「ManagedIdentityExtensionForWindows」或「ManagedIdentityExtensionForLinux」(取決於 VM 的類型)，並使用 `-Name` 參數為其命名。 `-Settings` 參數會指定 OAuth 權杖端點所使用的連接埠，以用來取得權杖。 請比對現有 VM 位置，確認指定正確的 `-Location` 參數：
      > [!NOTE]
    > 此步驟是選擇性的，因為您也可以使用 Azure Instance Metadata Service (IMDS) 識別端點以擷取權杖。 Azure 資源受控識別 VM 擴充功能已計劃在 2019 年 1 月淘汰。

   ```powershell
   $settings = @{ "port" = 50342 }
   Set-AzureRmVMExtension -ResourceGroupName myResourceGroup -Location WestUS -VMName myVM -Name "ManagedIdentityExtensionForWindows" -Type "ManagedIdentityExtensionForWindows" -Publisher "Microsoft.ManagedIdentity" -TypeHandlerVersion "1.0" -Settings $settings 
   ```

### <a name="assign-a-user-assigned-managed-identity-to-an-existing-azure-vm"></a>將使用者指派的受控識別指派至現有 Azure VM

若要將使用者指派的受控識別指派至現有 Azure VM：

1. 使用 `Connect-AzureRmAccount` 登入 Azure。 使用與包含虛擬機器的 Azure 訂用帳戶相關聯的帳戶。

   ```powershell
   Connect-AzureRmAccount
   ```

2. 請使用 [New-AzureRmUserAssignedIdentity](/powershell/module/azurerm.managedserviceidentity/new-azurermuserassignedidentity) Cmdlet 來建立使用者指派的受控識別。  請記下輸出中的 `Id`，因為下一個步驟 會需要此項目。

   > [!IMPORTANT]
   > 建立使用者指派的受控識別時，僅支援使用英數字元和連字號 (0-9 或 a-z 或 A-Z 或 -) 字元。 此外，指派至 VM/VMSS 的名稱應該限制為 24 個字元長度，才能正常運作。 請隨時回來查看是否有更新內容。 如需詳細資訊，請參閱[常見問題集和已知問題](known-issues.md)

   ```powershell
   New-AzureRmUserAssignedIdentity -ResourceGroupName <RESOURCEGROUP> -Name <USER ASSIGNED IDENTITY NAME>
   ```
3. 使用 `Get-AzureRmVM` Cmdlet 擷取 VM 屬性。 然後使用 [Update-AzureRmVM](/powershell/module/azurerm.compute/update-azurermvm) Cmdlet 上的 `-IdentityType` 和 `-IdentityID` 參數，將使用者指派的受控識別新增至 Azure VM。  `-IdentityId` 參數的值是您在上一個步驟中記下的 `Id`。  以您自己的值取代 `<VM NAME>`、`<SUBSCRIPTION ID>`、`<RESROURCE GROUP>` 和 `<USER ASSIGNED IDENTITY NAME>`。

   ```powershell
   $vm = Get-AzureRmVM -ResourceGroupName <RESOURCE GROUP> -Name <VM NAME>
   Update-AzureRmVM -ResourceGroupName <RESOURCE GROUP> -VM $vm -IdentityType UserAssigned -IdentityID "/subscriptions/<SUBSCRIPTION ID>/resourcegroups/<RESROURCE GROUP>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<USER ASSIGNED IDENTITY NAME>"
   ```

4. 在 [Set-AzureRmVMExtension](/powershell/module/azurerm.compute/set-azurermvmextension) Cmdlet 上使用 `-Type` 參數來新增 Azure 資源受控識別 VM 擴充功能 (已計劃在 2019 年 1 月淘汰)。 您可以傳遞「ManagedIdentityExtensionForWindows」或「ManagedIdentityExtensionForLinux」(取決於 VM 的類型)，並使用 `-Name` 參數為其命名。 `-Settings` 參數會指定 OAuth 權杖端點所使用的連接埠，以用來取得權杖。 請比對現有 VM 位置，指定正確的 `-Location` 參數。

   ```powershell
   $settings = @{ "port" = 50342 }
   Set-AzureRmVMExtension -ResourceGroupName myResourceGroup -Location WestUS -VMName myVM -Name "ManagedIdentityExtensionForWindows" -Type "ManagedIdentityExtensionForWindows" -Publisher "Microsoft.ManagedIdentity" -TypeHandlerVersion "1.0" -Settings $settings 
   ```

### <a name="remove-a-user-assigned-managed-identity-from-an-azure-vm"></a>從 Azure VM 移除使用者指派的受控識別

如果您的 VM 具有多個使用者指派的受控識別，則您可以使用下列命令移除所有身分識別，但請留下最後一個。 請務必以您自己的值取代 `<RESOURCE GROUP>` 和 `<VM NAME>` 參數的值。 `<USER ASSIGNED IDENTITY NAME>` 是使用者指派受控識別的名稱屬性，它應該保留在 VM 上。 您可以使用 `az vm show`，在 VM 的識別區段中找到此資訊：

```powershell
$vm = Get-AzureRmVm -ResourceGroupName myResourceGroup -Name myVm
Update-AzureRmVm -ResourceGroupName myResourceGroup -VirtualMachine $vm -IdentityType UserAssigned -IdentityID <USER ASSIGNED IDENTITY NAME>
```
如果您的虛擬機器沒有系統指派的受控識別，而且您想要從其中移除所有使用者指派的受控識別，請使用下列命令：

```powershell
$vm = Get-AzureRmVm -ResourceGroupName myResourceGroup -Name myVm
Update-AzureRmVm -ResourceGroupName myResourceGroup -VM $vm -IdentityType None
```
如果您的虛擬機器同時具有系統指派和使用者指派的受控識別，您可以藉由切換為僅使用系統指派的受控識別，來移除所有使用者指派的受控識別。

```powershell 
$vm = Get-AzureRmVm -ResourceGroupName myResourceGroup -Name myVm
Update-AzureRmVm -ResourceGroupName myResourceGroup -VirtualMachine $vm -IdentityType "SystemAssigned"
```

## <a name="next-steps"></a>後續步驟

- [Azure 資源受控識別概觀](overview.md)
- 如需完整的 Azure VM 建立快速入門，請參閱：
  
  - [使用 PowerShell 建立 Windows 虛擬機器](../../virtual-machines/windows/quick-create-powershell.md) 
  - [使用 PowerShell 建立 Linux 虛擬機器](../../virtual-machines/linux/quick-create-powershell.md) 
