---
title: Azure CLI 指令碼範例 - 建立 Azure Event Grid 訂用帳戶 | Microsoft Docs
description: 本主題中的 Azure CLI 指令碼會示範如何建立作業狀態變更的帳戶層級 Event Grid 訂用帳戶。
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
ms.openlocfilehash: eae9947b7dcbd6f2c52fb0d4abc65375aed7766e
ms.sourcegitcommit: 3a7c1688d1f64ff7f1e68ec4bb799ba8a29a04a8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49378783"
---
# <a name="cli-example-create-an-azure-event-grid-subscription"></a>CLI 範例：建立 Azure Event Grid 訂用帳戶 

本文中的 Azure CLI 指令碼會示範如何建立作業狀態變更的帳戶層級 Event Grid 訂用帳戶。

[!INCLUDE [cloud-shell-try-it.md](../../../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝和使用 CLI，本文會要求您執行 Azure CLI 2.0.20 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI](/cli/azure/install-azure-cli)。 

## <a name="example-script"></a>範例指令碼

[!code-azurecli-interactive[main](../../../../cli_scripts/media-services/create-event-grid/Create-EventGrid.sh "Create an EventGrid subscription")]

## <a name="next-steps"></a>後續步驟

如需其他範例，請參閱 [Azure CLI 範例](../cli-samples.md)。
