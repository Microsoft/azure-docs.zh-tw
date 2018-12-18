---
title: 在 Azure 的 Windows 虛擬機器中執行 PowerShell 指令碼
description: 本主題說明如何使用執行命令在 Azure Windows 虛擬機器中執行 PowerShell 指令碼
services: automation
ms.service: automation
author: georgewallace
ms.author: gwallace
ms.date: 06/06/2018
ms.topic: article
manager: carmonm
ms.openlocfilehash: 60cdcbeba0c4149d8f049bbab553bd658c965338
ms.sourcegitcommit: 2ad510772e28f5eddd15ba265746c368356244ae
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2018
ms.locfileid: "43128590"
---
# <a name="run-powershell-scripts-in-your-windows-vm-with-run-command"></a>使用執行命令在 Windows 虛擬機器中執行 PowerShell 指令碼

執行命令會使用 VM 代理程式在 Azure Windows VM 中執行 PowerShell 指令碼。 這些指令碼可用於一般機器或應用程式管理，且可用來快速診斷和修復虛擬機器存取和網路問題，並使虛擬機器恢復正常狀態。

## <a name="benefits"></a>優點

有多個選項可以用來存取您的虛擬機器。 執行命令可以使用虛擬機器代理程式，從遠端在虛擬機器上執行指令碼。 可以透過 Azure 入口網站、[REST API](/rest/api/compute/virtual%20machines%20run%20commands/runcommand)、[Azure CLI](/cli/azure/vm/run-command?view=azure-cli-latest#az-vm-run-command-invoke) 或 [PowerShell](/powershell/module/azurerm.compute/invoke-azurermvmruncommand) 使用執行命令。

您可以要在虛擬機器中執行指令碼的所有情況下使用此功能；且針對因網路或系統管理使用者設定不適當而未開啟 RDP 或 SSH 連接埠的虛擬機器，只有使用此功能才能進行疑難排解和修復。

## <a name="restrictions"></a>限制

使用執行命令時有下列限制：

* 輸出僅限於最後 4096 個位元組
* 執行指令碼的最短時間是大約 20 秒
* 在 Windows 上以系統身分執行指令碼
* 一次可執行一個指令碼
* 您無法取消執行中的指令碼
* 指令碼可以執行的最長時間是 90 分鐘，經過這段時間後會逾時
* 需要有虛擬機器的輸出連線，才能傳回指令碼結果。

## <a name="run-a-command"></a>執行命令

瀏覽至 [Azure](https://portal.azure.com) 中的虛擬機器，並選取 [作業] 下的 [執行命令]。 您會看到將可在虛擬機器上執行之命令的清單。

![執行命令清單](./media/run-command/run-command-list.png)

選擇要執行的命令。 有些命令可能會有選擇性或必要的輸入參數。 對於這些命令，參數會顯示為可讓您提供輸入值的文字欄位。 對於每個命令，您可以展開 [檢視指令碼] 檢視執行中的指令碼。 **RunPowerShellScript** 不同於其他命令，因為它可讓您提供您自己的自訂指令碼。

> [!NOTE]
> 無法編輯內建命令。

選擇命令之後，按一下 [執行] 執行指令碼。 指令碼隨即執行，完成時將在輸出視窗中傳回任何錯誤。 下列螢幕擷取畫面顯示執行 **RDPSettings** 命令的範例輸出。

![執行命令指令碼輸出](./media/run-command/run-command-script-output.png)

## <a name="commands"></a>命令

下表顯示 Windows 虛擬機器可用命令的清單。 **RunPowerShellScript** 命令可以用來執行您想要的任何自訂指令碼。

|**名稱**|**說明**|
|---|---|
|**RunPowerShellScript**|執行 PowerShell 指令碼|
|**EnableRemotePS**|設定機器啟用遠端 PowerShell。|
|**EnableAdminAccount**|檢查本機系統管理員帳戶是否已停用，若已停用，請將它啟用。|
|**IPConfig**| 對於繫結至 TCP/IP 的每個介面卡，顯示 IP 位址、子網路遮罩和預設閘道的詳細資訊。|
|**RDPSettings**|檢查登錄設定和網域原則設定。 如果機器是網域的一部分，或修改設定成為預設值，會建議原則動作。|
|**ResetRDPCert**|移除繫結至 RDP 接聽程式的 SSL 憑證，並將 RDP 接聽程式安全性還原為預設值。 如果您發現憑證有任何問題，請使用此指令碼。|
|**SetRDPPort**|設定遠端桌面連線的預設連接埠號碼或使用者指定連接埠號碼。 對於連接埠的輸入存取，啟用防火牆規則。|

## <a name="powershell"></a>PowerShell

下列是使用 [Invoke-AzureRmVMRunCommand](/powershell/module/azurerm.compute/invoke-azurermvmruncommand) Cmdlet 在 Azure VM 上執行 PowerShell 指令碼的範例。

```azurepowershell-interactive
Invoke-AzureRmVMRunCommand -ResourceGroupName '<myResourceGroup>' -Name '<myVMName>' -CommandId 'RunPowerShellScript' -ScriptPath '<pathToScript>' -Parameter @{"arg1" = "var1";"arg2" = "var2"}
```

## <a name="limiting-access-to-run-command"></a>限制對於執行命令的存取

列出執行命令，或顯示命令的詳細資料，都需要 `Microsoft.Compute/locations/runCommands/read` 權限，這是內建[讀者](../../role-based-access-control/built-in-roles.md#reader)角色或更高角色擁有的權限。

執行命令需要 `Microsoft.Compute/virtualMachines/runCommand/action` 權限，這是[參與者](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor)角色或更高角色擁有的權限。

您可以使用其中一個[內建](../../role-based-access-control/built-in-roles.md)角色或建立[自訂](../../role-based-access-control/custom-roles.md)角色使用執行命令。

## <a name="next-steps"></a>後續步驟

請參閱[在 Windows 虛擬機器中執行指令碼](run-scripts-in-vm.md)，了解在虛擬機器中從遠端執行指令碼和命令的其他方式。
