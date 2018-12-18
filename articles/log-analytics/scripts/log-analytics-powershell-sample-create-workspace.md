---
title: Azure PowerShell 指令碼範例 - 建立 Log Analytics 工作區 | Microsoft Docs
description: Azure PowerShell 指令碼範例 - 建立 Log Analytics 工作區
services: log-analytics
documentationcenter: ''
author: mgoedtel
manager: carmonm
editor: tysonn
tags: ''
ms.assetid: ''
ms.service: log-analytics
ms.devlang: na
ms.topic: sample
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/07/2017
ms.author: magoedte
ms.openlocfilehash: 30d036ae56acc3a798d2776f292243f65cbea43d
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23021333"
---
# <a name="create-a-log-analytics-workspace-with-powershell"></a>使用 PowerShell 建立 Log Analytics 工作區

如果您想要開始收集資料，並對資料作分析及採取行動，這個指令碼可讓您快速啟動並執行必要的 Azure Log Analytics 工作區。  

[!INCLUDE [sample-powershell-install](../../../includes/sample-powershell-install-no-ssh.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>範例指令碼

[!code-powershell[main](../../../powershell_scripts/log-analytics/log-analytics-create-new-resource/log-analytics-create-new-resource.ps1 "Create new Log Analytics workspace")]

## <a name="script-explanation"></a>指令碼說明

這個指令碼會使用下列命令，在您的訂用帳戶中建立新的 Log Analytics 工作區。 下表中的每個命令都會連結至命令特定的文件。

| 命令 | 注意 |
|---|---|
| [Get-AzureRmOperationalInsightsWorkspace](/powershell/module/azurerm.operationalinsights/get-azurermoperationalinsightsworkspace) | 取得現有工作區的相關資訊。 |
| [New-AzureRmOperationalInsightsWorkspace](/powershell/module/azurerm.operationalinsights/new-azurermoperationalinsightsworkspace) | 在指定的資源群組和位置建立工作區。 |


## <a name="next-steps"></a>後續步驟

如需有關 Azure PowerShell 模組的詳細資訊，請參閱 [Azure PowerShell 文件](/powershell/azure/overview)。

