---
title: 快速入門 - 使用 Azure CLI 建立 Linux VM | Microsoft Docs
description: 在本快速入門中，您將了解如何使用 Azure CLI 建立 Linux 虛擬機器
services: virtual-machines-linux
documentationcenter: virtual-machines
author: cynthn
manager: jeconnoc
editor: tysonn
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 10/09/2018
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: beb70b03198589808c20ab17498902367a1c6a3d
ms.sourcegitcommit: 7b0778a1488e8fd70ee57e55bde783a69521c912
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/10/2018
ms.locfileid: "49067481"
---
# <a name="quickstart-create-a-linux-virtual-machine-with-the-azure-cli"></a>快速入門：使用 Azure CLI 建立 Linux 虛擬機器

Azure CLI 可用來從命令列或在指令碼中建立和管理 Azure 資源。 本快速入門示範如何使用 Azure CLI 在 Azure 中部署 Linux 虛擬機器 (VM)。 在本教學課程中，我們將安裝 Ubuntu 160.04 LTS。 若要顯示作用中的 VM，您將使用 SSH 來與之連線，並安裝 NGINX Web 伺服器。

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

## <a name="launch-azure-cloud-shell"></a>啟動 Azure Cloud Shell

Azure Cloud Shell 是免費的互動式 Shell，可讓您用來執行本文中的步驟。 它具有預先安裝和設定的共用 Azure 工具，可與您的帳戶搭配使用。 

若要開啟 Cloud Shell，只要選取程式碼區塊右上角的 [試試看] 即可。 您也可以移至 [https://shell.azure.com/bash](https://shell.azure.com/bash)，從另一個瀏覽器索引標籤啟動 Cloud Shell。 選取 [複製] 即可複製程式碼區塊，將它貼到 Cloud Shell 中，然後按 enter 鍵加以執行。

如果您偏好在本機安裝和使用 CLI，本快速入門需要有 Azure CLI 2.0.30 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI]( /cli/azure/install-azure-cli)。

## <a name="create-a-resource-group"></a>建立資源群組

使用 [az group create](/cli/azure/group#az_group_create) 命令來建立資源群組。 Azure 資源群組是在其中部署與管理 Azure 資源的邏輯容器。 下列範例會在 eastus 位置建立名為 myResourceGroup 的資源群組：

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

## <a name="create-virtual-machine"></a>建立虛擬機器

使用 [az vm create](/cli/azure/vm#az_vm_create) 命令來建立 VM。

下列範例會建立名為 myVM 的 VM，並新增名為 azureuser 的使用者帳戶。 `--generate-ssh-keys` 參數用來自動產生 SSH 金鑰，並將它放在預設金鑰位置 (~/.ssh)。 若要改為使用一組特定金鑰，請使用 `--ssh-key-value` 選項。

```azurecli-interactive
az vm create \
  --resource-group myResourceGroup \
  --name myVM \
  --image UbuntuLTS \
  --admin-username azureuser \
  --generate-ssh-keys
```

建立虛擬機器和支援資源需要幾分鐘的時間。 下列範例輸出顯示 VM 建立作業成功。

```
{
  "fqdns": "",
  "id": "/subscriptions/<guid>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM",
  "location": "eastus",
  "macAddress": "00-0D-3A-23-9A-49",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.4",
  "publicIpAddress": "40.68.254.142",
  "resourceGroup": "myResourceGroup"
}
```

請注意 VM 輸出中您自己的 `publicIpAddress`。 後續步驟會使用此位址來存取 VM。

## <a name="open-port-80-for-web-traffic"></a>針對 Web 流量開啟連接埠 80

根據預設，只有在 Azure 中部署 Linux VM 時才會開啟 SSH 連線。 使用 [az vm open-port](/cli/azure/vm#az_vm_open_port) 開啟 TCP 連接埠 80 以供搭配 NGINX 網頁伺服器使用：

```azurecli-interactive
az vm open-port --port 80 --resource-group myResourceGroup --name myVM
```

## <a name="connect-to-virtual-machine"></a>連線至虛擬機器

如常透過 SSH 連線至您的 VM。 以您 VM 的公用 IP 位址取代 **publicIpAddress**，如下您 VM 先前的輸出所示：

```bash
ssh azureuser@publicIpAddress
```

## <a name="install-web-server"></a>安裝 Web 伺服器

若要查看作用中的 VM，請安裝 NGINX 網頁伺服器。 更新您的套件來源，然後安裝最新的 NGINX 套件。

```bash
sudo apt-get -y update
sudo apt-get -y install nginx
```

完成時，輸入 `exit` 來離開 SSH 工作階段。

## <a name="view-the-web-server-in-action"></a>檢視作用中的網頁伺服器

使用所選的網頁瀏覽器來查看預設 NGINX 歡迎使用頁面。 使用您 VM 的公用 IP 位址作為網址。 下列範例示範預設的 NGINX 網站：

![預設 NGINX 網站](./media/quick-create-cli/nginx.png)

## <a name="clean-up-resources"></a>清除資源

若不再需要，您可以使用 [az group delete](/cli/azure/group#az_group_delete) 命令來移除資源群組、VM 和所有相關資源。 

```azurecli-interactive
az group delete --name myResourceGroup
```

## <a name="next-steps"></a>後續步驟

在此快速入門中，您已部署簡單的虛擬機器、對網路流量開啟網路連接埠，以及安裝基本的網頁伺服器。 若要深入了解 Azure 虛擬機器，請繼續 Linux VM 的教學課程。


> [!div class="nextstepaction"]
> [Azure Linux 虛擬機器教學課程](./tutorial-manage-vm.md)
