---
title: Azure CLI 指令碼範例 - 啟動實驗室中的虛擬機器 | Microsoft Docs
description: 此 Azure CLI 指令碼會啟動 Azure DevTest Labs 中的虛擬機器。
services: lab-services
author: spelluru
manager: ''
editor: ''
ms.assetid: ''
ms.service: lab-services
ms.devlang: azurecli
ms.topic: sample
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 04/12/2018
ms.author: spelluru
ms.custom: mvc
ms.openlocfilehash: a635766c1a7fb9ae10a651d09ecd7da9a5f01e51
ms.sourcegitcommit: 6116082991b98c8ee7a3ab0927cf588c3972eeaa
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/05/2018
ms.locfileid: "34763131"
---
# <a name="use-azure-cli-to-start-a-virtual-machine-in-a-lab-in-azure-devtest-labs"></a>使用 Azure CLI 在 Azure DevTest Labs 中啟動實驗室中的虛擬機器

此 Azure CLI 指令碼會啟動實驗室中的虛擬機器 (VM)。 

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>範例指令碼

[!code-azurecli-interactive[main](../../../cli_scripts/devtest-lab/start-connect-virtual-machine-in-lab/start-connect-virtual-machine-in-lab.sh "Start a VM")]


## <a name="script-explanation"></a>指令碼說明

此指令碼會使用下列命令：

| 命令 | 注意 |
|---|---|
| [az lab vm start ](/cli/azure/lab/vm?view=azure-cli-latest#az-lab-vm-start) | 啟動實驗室中的虛擬機器 (VM)。 此作業可能需要一些時間來完成。 |

## <a name="next-steps"></a>後續步驟

如需 Azure CLI 的詳細資訊，請參閱 [Azure CLI 文件](https://docs.microsoft.com/cli/azure)。

您可以在 [Azure 實驗室服務 CLI 範例](../samples-cli.md)中找到其他的 Azure 實驗室服務 CLI 指令碼範例。
