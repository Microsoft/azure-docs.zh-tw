---
title: Azure CLI 指令碼範例 - 重設您的帳戶認證 | Microsoft Docs
description: 使用 Azure CLI 指令碼來重設您的帳戶認證，並還原為 app.config 設定。
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
ms.openlocfilehash: f97cf87288f5efbeef11a27e98ab3fc2caf27b40
ms.sourcegitcommit: 3a7c1688d1f64ff7f1e68ec4bb799ba8a29a04a8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49376408"
---
# <a name="cli-example-reset-the-account-credentials"></a>CLI 範例：重設帳戶認證

本文中的 Azure CLI 指令碼會示範如何重設您的帳戶認證，並還原為 app.config 設定。

[!INCLUDE [cloud-shell-try-it.md](../../../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝和使用 CLI，本文會要求您執行 Azure CLI 2.0.20 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI](/cli/azure/install-azure-cli)。 

## <a name="example-script"></a>範例指令碼

[!code-azurecli-interactive[main](../../../../cli_scripts/media-services/reset-account-credentials/Reset-Account-Credentials.sh "Reset credentials")]

## <a name="next-steps"></a>後續步驟

如需其他範例，請參閱 [Azure CLI 範例](../cli-samples.md)。
