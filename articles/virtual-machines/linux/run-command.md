---
title: 在 Azure 上的 Linux VM 中執行殼層指令碼
description: 本主題說明如何使用執行命令在 Azure Linux 虛擬機器中執行指令碼
services: automation
ms.service: automation
author: georgewallace
ms.author: gwallace
ms.date: 06/06/2018
ms.topic: article
manager: carmonm
ms.openlocfilehash: 850c5ac4df8ff3bd0e35567060b3b90dad7baacc
ms.sourcegitcommit: 4597964eba08b7e0584d2b275cc33a370c25e027
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/02/2018
ms.locfileid: "37342686"
---
# <a name="run-shell-scripts-in-your-linux-vm-with-run-command"></a>使用執行命令在 Linux VM 中執行殼層指令碼

執行命令會使用 VM 代理程式在 Azure Linux VM 中執行殼層指令碼。 這些指令碼可用於一般機器或應用程式管理，且可用來快速診斷和修復虛擬機器存取和網路問題，並使虛擬機器恢復正常狀態。

## <a name="benefits"></a>優點

有多個選項可以用來存取您的虛擬機器。 執行命令可以使用虛擬機器代理程式，從遠端在虛擬機器上執行指令碼。 可以透過 Azure 入口網站、[REST API](/rest/api/compute/virtual%20machines%20run%20commands/runcommand)、[Azure CLI](/cli/azure/vm/run-command?view=azure-cli-latest#az-vm-run-command-invoke) 或 [PowerShell](/powershell/module/azurerm.compute/invoke-azurermvmruncommand) 使用執行命令。

您可以要在虛擬機器中執行指令碼的所有情況下使用此功能；且針對因網路或系統管理使用者設定不適當而未開啟 RDP 或 SSH 連接埠的虛擬機器，只有使用此功能才能進行疑難排解和修復。

## <a name="restrictions"></a>限制

以下列出在使用執行命令時所受到的限制。

* 輸出僅限於最後 4096 個位元組
* 執行指令碼的最短時間大約 20 秒
* 指令碼依預設會以提高權限的使用者身分在 Linux 上執行
* 一次可執行一個指令碼
* 不支援提示資訊 (互動模式) 的指令碼。
* 您無法取消執行中的指令碼
* 指令碼可以執行的最長時間是 90 分鐘，經過這段時間後會逾時
* 需要有虛擬機器的輸出連線，才能傳回指令碼結果。

## <a name="azure-cli"></a>Azure CLI

以下是使用 [az vm run-command](/cli/azure/vm/run-command?view=azure-cli-latest#az-vm-run-command-invoke) 命令在 Azure Linux VM 上執行殼層指令碼的範例。

```azurecli-interactive
az vm run-command invoke -g myResourceGroup -n myVm --command-id RunShellScript --scripts "sudo apt-get update && sudo apt-get install -y nginx"
```

> [!NOTE]
> 若要以不同的使用者身分執行命令，您可以使用 `sudo -u` 指定要使用的使用者帳戶。

## <a name="azure-portal"></a>Azure 入口網站

瀏覽至 [Azure](https://portal.azure.com) 中的 VM，並選取 [作業] 下的 [執行命令]。 您會看到將可在虛擬機器上執行之命令的清單。

![執行命令清單](./media/run-command/run-command-list.png)

選擇要執行的命令。 有些命令可能會有選擇性或必要的輸入參數。 對於這些命令，參數會顯示為可讓您提供輸入值的文字欄位。 對於每個命令，您可以展開 [檢視指令碼] 檢視執行中的指令碼。 **RunShellScript** 不同於其他命令，因為它可讓您提供您自己的自訂指令碼。 

> [!NOTE]
> 內建命令是無法編輯的。

選擇命令之後，按一下 [執行] 執行指令碼。 指令碼隨即執行，完成時將在輸出視窗中傳回任何錯誤。 下列螢幕擷取畫面顯示執行 **ifconfig** 命令的範例輸出。

![執行命令指令碼輸出](./media/run-command/run-command-script-output.png)

## <a name="available-commands"></a>可用的命令

下表顯示 Linux VM 的可用命令清單。 **RunShellScript** 命令可用來執行您想要的任何自訂指令碼。

|**名稱**|**說明**|
|---|---|
|**RunShellScript**|執行 Linux 殼層指令碼。|
|**ifconfig**| 取得所有網路介面的組態。|

## <a name="limiting-access-to-run-command"></a>限制於執行命令的存取

列出執行命令，或顯示命令的詳細資料，都需要 `Microsoft.Compute/locations/runCommands/read` 權限，這是內建[讀者](../../role-based-access-control/built-in-roles.md#reader)角色或更高角色擁有的權限。

執行命令需要 `Microsoft.Compute/virtualMachines/runCommand/action` 權限，這是[參與者](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor)角色或更高角色擁有的權限。

您可以使用其中一個[內建](../../role-based-access-control/built-in-roles.md)角色或建立[自訂](../../role-based-access-control/custom-roles.md)角色使用執行命令。

## <a name="next-steps"></a>後續步驟

請參閱[在 Linux VM 中執行指令碼](run-scripts-in-vm.md)，了解如何以其他方式從遠端在您的 VM 中執行指令碼和命令。
