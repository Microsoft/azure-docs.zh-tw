---
title: 快速入門 - 使用 Azure PowerShell 建立 Windows VM | Microsoft Docs
description: 在本快速入門中，您會了解如何使用 Azure PowerShell 來建立 Windows 虛擬機器
services: virtual-machines-windows
documentationcenter: virtual-machines
author: cynthn
manager: jeconnoc
editor: tysonn
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 04/24/2018
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: 4ca8d42c1b2eece82fa31283b0df0d450e2f5afc
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46978874"
---
# <a name="quickstart-create-a-windows-virtual-machine-with-the-azure-cli"></a>快速入門：使用 Azure CLI 建立 Windows 虛擬機器

Azure CLI 可用來從命令列或在指令碼中建立和管理 Azure 資源。 本快速入門示範如何使用 Azure CLI，在 Azure 中部署執行 Windows Server 2016 的虛擬機器 (VM)。 若要查看作用中的 VM，接著要以 RDP 連線至 VM，並安裝 IIS 網頁伺服器。

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝和使用 CLI，本快速入門會要求您執行 Azure CLI 2.0.30 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI]( /cli/azure/install-azure-cli)。

## <a name="create-a-resource-group"></a>建立資源群組

使用 [az group create](/cli/azure/group#az_group_create) 命令來建立資源群組。 Azure 資源群組是在其中部署與管理 Azure 資源的邏輯容器。 下列範例會在 eastus 位置建立名為 myResourceGroup 的資源群組：

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

## <a name="create-virtual-machine"></a>建立虛擬機器

使用 [az vm create](/cli/azure/vm#az_vm_create) 建立 VM。 下列範例會建立名為 myVM 的 VM。 此範例會以 azureuser 作為系統管理使用者名稱並以 myPassword12 作為密碼。 將這些值更新為適合您環境的值。 當您連線到 VM 時需要使用這些值。

```azurecli-interactive
az vm create \
    --resource-group myResourceGroup \
    --name myVM \
    --image win2016datacenter \
    --admin-username azureuser \
    --admin-password myPassword12
```

建立虛擬機器和支援資源需要幾分鐘的時間。 下列範例輸出顯示 VM 建立作業成功。

```azurecli-interactive
{
  "fqdns": "",
  "id": "/subscriptions/<guid>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM",
  "location": "eastus",
  "macAddress": "00-0D-3A-23-9A-49",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.4",
  "publicIpAddress": "52.174.34.95",
  "resourceGroup": "myResourceGroup"
}
```

請注意 VM 輸出中您自己的 `publicIpAddress`。 後續步驟會使用此位址來存取 VM。

## <a name="open-port-80-for-web-traffic"></a>針對 Web 流量開啟連接埠 80

根據預設，只有在 Azure 中部署 Windows VM 時才會開啟 RDP 連線。 使用 [az vm open-port](/cli/azure/vm#az_vm_open_port) 開啟 TCP 連接埠 80 以供搭配 IIS 網頁伺服器使用：

```azurecli-interactive
az vm open-port --port 80 --resource-group myResourceGroup --name myVM
```

## <a name="connect-to-virtual-machine"></a>連線至虛擬機器

從您的本機電腦使用下列命令，建立遠端桌面工作階段。 以 VM 的公用 IP 位址取代 IP 位址。 出現提示時，請輸入在建立 VM 時所使用的認證：

```powershell
mstsc /v:publicIpAddress
```

## <a name="install-web-server"></a>安裝 Web 伺服器

若要查看作用中的 VM，請安裝 IIS 網頁伺服器。 在 VM 上開啟 PowerShell 提示字元並執行下列命令：

```powershell
Install-WindowsFeature -name Web-Server -IncludeManagementTools
```

完成時，關閉與 VM 的 RDP 連線。

## <a name="view-the-web-server-in-action"></a>檢視作用中的網頁伺服器

安裝 IIS 後，現在經由網際網路在您的 VM 上開啟連接埠 80，請使用所選的網頁瀏覽器來檢視預設 IIS 歡迎使用畫面。 使用上一個步驟所取得 VM 的公用 IP 位址。 下列範例示範預設的 IIS 網站：

![IIS 預設網站](./media/quick-create-powershell/default-iis-website.png)

## <a name="clean-up-resources"></a>清除資源

若不再需要，您可以使用 [az group delete](/cli/azure/group#az_group_delete) 命令來移除資源群組、VM 和所有相關資源：

```azurecli-interactive
az group delete --name myResourceGroup
```

## <a name="next-steps"></a>後續步驟

在此快速入門中，您已部署簡單的虛擬機器、對網路流量開啟網路連接埠，以及安裝基本的網頁伺服器。 若要深入了解 Azure 虛擬機器，請繼續 Windows VM 的教學課程。

> [!div class="nextstepaction"]
> [Azure Windows 虛擬機器教學課程](./tutorial-manage-vm.md)
