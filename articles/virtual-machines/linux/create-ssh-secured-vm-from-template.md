---
title: 從範本在 Azure 中建立 Linux VM | Microsoft Docs
description: 如何使用 Azure CLI 從 Resource Manager 範本建立 Linux VM
services: virtual-machines-linux
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: 721b8378-9e47-411e-842c-ec3276d3256a
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: azurecli
ms.topic: article
ms.date: 05/30/2018
ms.author: cynthn
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 33b0b2b4148a8cfbe97413bf3cdec23d375206bb
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46983882"
---
# <a name="how-to-create-a-linux-virtual-machine-with-azure-resource-manager-templates"></a>如何使用 Azure Resource Manager 範本建立 Linux 虛擬機器

本文示範如何使用 Azure Resource Manager 範本和 Azure CLI，快速部署 Linux 虛擬機器 (VM)。 您也可以使用 [Azure 傳統 CLI](create-ssh-secured-vm-from-template-nodejs.md) 來執行這些步驟。


本文示範如何使用 Azure Resource Manager 範本和 Azure CLI，快速部署 Linux 虛擬機器 (VM)。 

## <a name="templates-overview"></a>範本概觀
Azure Resource Manager 範本是 JSON 檔案，檔案中定義您的 Azure 解決方案的基礎結構和組態。 透過範本，您可以在整個生命週期中重複部署方案，並確信您的資源會以一致的狀態部署。 若要深入了解範本格式和其建構方式，請參閱[建立第一個 Azure Resource Manager 範本](../../azure-resource-manager/resource-manager-create-first-template.md)。 若要檢視資源類型的 JSON 語法，請參閱[在 Azure Resource Manager 範本中定義資源](/azure/templates/)。


## <a name="create-a-resource-group"></a>建立資源群組
Azure 資源群組是在其中部署與管理 Azure 資源的邏輯容器。 資源群組必須在虛擬機器之前建立。 以下範例會在 eastus 區域建立名為 myResourceGroupVM 的資源群組：

```azurecli
az group create --name myResourceGroup --location eastus
```

## <a name="create-a-virtual-machine"></a>建立虛擬機器
以下範例使用 [az group deployment create](/cli/azure/group/deployment#az_group_deployment_create)，從[這個 Azure Resource Manager 範本](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-vm-sshkey/azuredeploy.json)建立 VM︰ 只允許 SSH 驗證。 出現提示時，提供您自己 SSH 公開金鑰的值，例如 ~/.ssh/id_rsa.pub 的內容。 如果您需要建立 SSH 金鑰組，請參閱[如何為 Azure 中的 Linux VM 建立和使用的 SSH 金鑰組](mac-create-ssh-keys.md)。

```azurecli
az group deployment create \
    --resource-group myResourceGroup \
    --template-uri https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/101-vm-sshkey/azuredeploy.json
```

在先前範例中，您已指定存放在 GitHub 中的範本。 您也可以下載或建立範本，並用 `--template-file` 參數指定本機路徑。


## <a name="connect-to-virtual-machine"></a>連線至虛擬機器
若要透過 SSH 連線至您的 VM，使用 [az vm show](/cli/azure/vm#az-vm-show) 取公用 IP 位址：

```azurecli
az vm show \
    --resource-group myResourceGroup \
    --name sshvm \
    --show-details \
    --query publicIps \
    --output tsv
```

之後您便可以如常透過 SSH 連線至您的 VM。 提供您自己的公用 IP 位址 (透過先前命令取得)：

```bash
ssh azureuser@<ipAddress>
```

## <a name="next-steps"></a>後續步驟
在此範例中，您建立基本的 Linux VM。 如需其他 Resource Manager 範本，包含應用程式架構，或建立更複雜的環境，瀏覽 [Azure 快速入門範本資源庫](https://azure.microsoft.com/documentation/templates/)。
