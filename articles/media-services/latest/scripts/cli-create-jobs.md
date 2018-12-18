---
title: Azure CLI 指令碼範例 - 建立和提交作業 | Microsoft Docs
description: 本主題中的 Azure CLI 指令碼會示範如何使用 HTTPs URL 將作業提交給簡單的編碼轉換。
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
ms.openlocfilehash: 3338fa32810b74deaf576d3eb966d5cac09eac1a
ms.sourcegitcommit: 3a7c1688d1f64ff7f1e68ec4bb799ba8a29a04a8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49376323"
---
# <a name="cli-example-create-and-submit-a-job"></a>CLI 範例：建立和提交作業

本文中的 Azure CLI 指令碼會示範如何建立作業，並使用 HTTPs URL 將其提交給簡單的編碼轉換。

[!INCLUDE [cloud-shell-try-it.md](../../../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝和使用 CLI，本文會要求您執行 Azure CLI 2.0.20 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI](/cli/azure/install-azure-cli)。 

## <a name="example-script"></a>範例指令碼

[!code-azurecli-interactive[main](../../../../cli_scripts/media-services/create-jobs/Create-Jobs.sh "Create and submit jobs")]

## <a name="next-steps"></a>後續步驟

如需其他範例，請參閱 [Azure CLI 範例](../cli-samples.md)。
