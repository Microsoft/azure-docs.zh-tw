---
title: Azure CLI 指令碼範例 - 將檔案上傳至容器 | Microsoft Docs
description: 您可以使用 Azure CLI 指令碼，將本機檔案上傳至儲存體容器。
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
ms.openlocfilehash: d317c39a1a4bd409aff02d85a5a924f7d2c99597
ms.sourcegitcommit: 3a7c1688d1f64ff7f1e68ec4bb799ba8a29a04a8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49376732"
---
# <a name="cli-example-upload-a-local-file-to-a-container"></a>CLI 範例：將本機檔案上傳至容器 

本文中的 Azure CLI 指令碼會示範如何將本機檔案上傳至儲存體容器。

[!INCLUDE [cloud-shell-try-it.md](../../../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝和使用 CLI，本文會要求您執行 Azure CLI 2.0.20 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI](/cli/azure/install-azure-cli)。 

## <a name="example-script"></a>範例指令碼

[!code-azurecli-interactive[main](../../../../cli_scripts/media-services/upload-file-asset/UploadFile-Asset.sh "Upload a file")]

## <a name="next-steps"></a>後續步驟

如需其他範例，請參閱 [Azure CLI 範例](../cli-samples.md)。
