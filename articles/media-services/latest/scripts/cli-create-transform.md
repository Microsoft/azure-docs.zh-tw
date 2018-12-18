---
title: Azure CLI 指令碼範例 - 建立轉換 | Microsoft Docs
description: 您可以使用 Azure CLI 指令碼來建立轉換。
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.assetid: ''
ms.service: media-services
ms.devlang: azurecli
ms.topic: sample
ms.tgt_pltfrm: multiple
ms.workload: na
ms.date: 10/16/2018
ms.author: juliako
ms.openlocfilehash: 57079e1086b473347ff027a98b794f4c9baeb1c0
ms.sourcegitcommit: 3a7c1688d1f64ff7f1e68ec4bb799ba8a29a04a8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49376562"
---
# <a name="cli-example-create-a-transform"></a>CLI 範例：建立轉換

本文中的 Azure CLI 指令碼會示範如何建立轉換。 轉換會說明用於處理視訊或音訊檔案的簡單工作流程 (通常稱為「配方」)。 請一律檢查是否已有所需名稱和「配方」的轉換。 若是如此，您應該重複使用該轉換。

[!INCLUDE [cloud-shell-try-it.md](../../../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝和使用 CLI，本文會要求您執行 Azure CLI 2.0.20 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI](/cli/azure/install-azure-cli)。 

## <a name="example-script"></a>範例指令碼

[!code-azurecli-interactive[main](../../../../cli_scripts/media-services/create-transform/Create-Transform.sh "Create a transform")]

## <a name="next-steps"></a>後續步驟

如需其他範例，請參閱 [Azure CLI 範例](../cli-samples.md)。
