---
title: 在 Azure 中重新部署 Linux 虛擬機器 | Microsoft Docs
description: 如何在 Azure 中重新部署 Linux 虛擬機器，以減輕 SSH 連線問題。
services: virtual-machines-linux
documentationcenter: virtual-machines
author: genlin
manager: jeconnoc
tags: azure-resource-manager,top-support-issue
ms.assetid: e9530dd6-f5b0-4160-b36b-d75151d99eb7
ms.service: virtual-machines-linux
ms.topic: troubleshooting
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 12/14/2017
ms.author: genli
ms.openlocfilehash: 8f8cab985ebc20290a94df34a9268ad2c1f169d3
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47411481"
---
# <a name="redeploy-linux-virtual-machine-to-new-azure-node"></a>將 Linux 虛擬機器重新部署至新的 Azure 節點
如果您在 Azure 中進行對 Linux 虛擬機器 (VM) 的 SSH 或應用程式存取疑難排解時遇到問題，重新部署 VM 也許可以解決。 重新部署 VM 時會將 VM 移到 Azure 基礎結構內的新節點，然後重新開啟它的電源。 您所有組態選項和相關聯的資源都會加以保留。 本文將說明如何使用 Azure CLI 或 Azure 入口網站來重新部署 VM。

> [!NOTE]
> 重新部署 VM 之後，暫存磁碟會遺失，而系統會更新與虛擬網路介面關聯的動態 IP 位址。 


## <a name="use-the-azure-cli"></a>使用 Azure CLI
請安裝最新的 [Azure CLI](/cli/azure/install-az-cli2) 並使用 [az login](/cli/azure/reference-index#az_login) 來登入 Azure 帳戶。

使用 [az vm redeploy](/cli/azure/vm#az_vm_redeploy) 來重新部署 VM。 下列範例會將名為 *myResourceGroup* 資源群組中名為 *myVM* 的 VM 重新部署：

```azurecli
az vm redeploy --resource-group myResourceGroup --name myVM 
```

## <a name="use-the-azure-classic-cli"></a>使用 Azure 傳統 CLI
請安裝[最新的 Azure 傳統 CLI](../../cli-install-nodejs.md) 並登入 Azure 帳戶。 確定您處於資源管理員模式 (`azure config mode arm`)。

下列範例會將名為 *myResourceGroup* 資源群組中名為 *myVM* 的 VM 重新部署：

```azurecli
azure vm redeploy --resource-group myResourceGroup --vm-name myVM 
```

[!INCLUDE [virtual-machines-common-redeploy-to-new-node](../../../includes/virtual-machines-common-redeploy-to-new-node.md)]

## <a name="next-steps"></a>後續步驟
如果您在連接至 VM 時發生問題，您可以在[針對 SSH 連線進行疑難排解](troubleshoot-ssh-connection.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)或[詳細的 SSH 疑難排解步驟](detailed-troubleshoot-ssh-connection.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)中找到具體的說明。 如果無法存取在您 VM 上執行的應用程式，您也可以參閱[應用程式疑難排解問題](troubleshoot-app-connection.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。


