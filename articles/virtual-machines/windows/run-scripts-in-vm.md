---
title: 在 Azure Windows VM 中執行指令碼
description: 本主題說明如何執行 Windows 虛擬機器內的指令碼
services: automation
ms.service: virtual-machines
author: bobbytreed
ms.author: robreed
ms.date: 05/02/2018
ms.topic: how-to
manager: carmonm
ms.openlocfilehash: 23abc86e26686d9a23ed94d0311a44ffe3012657
ms.sourcegitcommit: 2bd0a039be8126c969a795cea3b60ce8e4ce64fc
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/14/2021
ms.locfileid: "98201768"
---
# <a name="run-scripts-in-your-windows-vm"></a>在 Windows 虛擬機器中執行指令碼

若要自動執行工作或疑難排解問題，您可能需要在虛擬機器中執行命令。 下文簡要說明可在虛擬機器中執行指令碼和命令的功能。

## <a name="custom-script-extension"></a>自訂指令碼延伸模組

[自訂指令碼擴充功能](../extensions/custom-script-windows.md)主要用於部署後設定和軟體安裝。

* 下載指令碼並在 Azure 虛擬機器中執行。
* 可使用 Azure 資源管理員範本、Azure CLI、REST API、PowerShell 或 Azure 入口網站執行。
* 可從 Azure 儲存體或 GitHub 下載指令檔，或是從 Azure 入口網站執行時，您的電腦也會提供指令檔。
* 在 Windows 機器中執行 PowerShell 指令碼，並在 Linux 機器中執行 Bash 指令碼。
* 適用於部署後設定、軟體安裝，以及其他設定或管理工作。

## <a name="run-command"></a>執行命令

[執行命令](run-command.md)功能可使用指令碼進行虛擬機器和應用程式管理以及疑難排解，即使無法觸達電腦也可進行 (例如在客體防火牆未開啟 RDP 或 SSH 連接埠時)。

* 在 Azure 虛擬機器中執行指令碼。
* 可使用 [Azure 入口網站](run-command.md)、[REST API](/rest/api/compute/virtual%20machines%20run%20commands/runcommand)、[Azure CLI](/cli/azure/vm/run-command#az-vm-run-command-invoke) 或 [PowerShell](/powershell/module/az.compute/invoke-azvmruncommand) 執行
* 快速執行指令碼和檢視輸出，並視需要在 Azure 入口網站中重複進行。
* 您可以直接輸入指令碼，或是執行其中一個內建的指令碼。
* 在 Windows 機器中執行 PowerShell 指令碼，並在 Linux 機器中執行 Bash 指令碼。
* 可用於虛擬機器和應用程式管理，且可用於在無法觸達的虛擬機器中執行指令碼。

## <a name="hybrid-runbook-worker"></a>Hybrid Runbook Worker

[混合式 Runbook 背景工作角色](../../automation/automation-hybrid-runbook-worker.md)提供使用自動化帳戶中儲存之使用者自訂指令碼執行的一般機器、應用程式和環境管理。

* 在 Azure 和非 Azure 機器中執行指令碼。
* 可使用 Azure 入口網站、Azure CLI、REST API、PowerShell、Webhook 執行。
* 在自動化帳戶中儲存和管理的指令碼。
* 執行 PowerShell、PowerShell 工作流程、Python 或圖形化 Runbook
* 指令碼執行時間沒有時間限制。
* 可以同時執行多個指令碼。
* 會傳回並儲存完整的指令碼輸出。
* 作業記錄長達 90 天。
* 能夠以本機系統身分或使用者提供的認證執行指令碼。
* 需要[手動安裝](../../automation/automation-windows-hrw-install.md)

## <a name="serial-console"></a>序列主控台

[序列主控台](../troubleshooting/serial-console-windows.md)可直接存取虛擬機器，類似於鍵盤連接到虛擬機器。

* 在 Azure 虛擬機器中執行命令。
* 可以在 Azure 入口網站中使用以文字為基礎的機器主控台執行。
* 以本機使用者帳戶登入機器。
* 無論機器的網路或作業系統狀態為何，需要存取虛擬機器時均可使用。

## <a name="next-steps"></a>後續步驟

深入了解可在虛擬機器中執行指令碼和命令的不同功能。

* [自訂指令碼擴充功能](../extensions/custom-script-windows.md)
* [執行命令](run-command.md)
* [Hybrid Runbook Worker](../../automation/automation-hybrid-runbook-worker.md)
* [序列主控台](../troubleshooting/serial-console-windows.md)
