---
title: 在 Azure Stack 中使用 Azure CLI 來建立 Linux 虛擬機器 | Microsoft Docs
description: 在 Azure Stack 中使用 CLI 來建立 Linux 虛擬機器。
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila
editor: ''
ms.assetid: 21F7D599-1FEC-4827-A5C3-06495C5F53A4
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: quickstart
ms.date: 04/24/2018
ms.author: mabrigg
ms.custom: mvc
ms.openlocfilehash: f6f2860b1ae1e88495e2dad3916a0216bf5d0726
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39428786"
---
# <a name="quickstart-create-a-linux-server-virtual-machine-by-using-azure-cli-in-azure-stack"></a>快速入門：在 Azure Stack 中使用 Azure CLI 建立 Linux 伺服器虛擬機器

*適用於：Azure Stack 整合系統和 Azure Stack 開發套件*

您可以使用 Azure CLI 建立 Ubuntu Server 16.04 LTS 虛擬機器。 請遵循本文中的步驟建立和使用虛擬機器。 本文也提供下列操作的步驟：

* 使用遠端用戶端連線到虛擬機器。
* 安裝 NGINX 網頁伺服器，並且檢視預設首頁。
* 清除未使用的資源。

## <a name="prerequisites"></a>必要條件

* **Azure Stack 市集中的 Linux 映像**

   Azure Stack 市集預設並未包含 Linux 映像。 請求 Azure Stack 操作員提供您所需的**Ubuntu Server 16.04 LTS** 映像。 操作員可以使用[將 Marketplace 項目從 Azure 下載到 Azure Stack](../azure-stack-download-azure-marketplace-item.md) 一文中所述的步驟。

* Azure Stack 需要特定版本的 Azure CLI，才能建立和管理資源。 如果您尚未針對 Azure Stack 設定 Azure CLI，請登入[開發套件](azure-stack-connect-azure-stack.md#connect-to-azure-stack-with-remote-desktop)，或登入以 Windows 為基礎的外部用戶端 (如果您[透過 VPN 連線](azure-stack-connect-azure-stack.md#connect-to-azure-stack-with-vpn)) 並遵循步驟來[安裝和設定 Azure CLI](azure-stack-version-profiles-azurecli2.md)。

* 名稱為 id_rsa.pub 的公開 SSH 金鑰應儲存在 Windows 使用者設定檔的 .ssh 目錄中。 如需有關建立 SSH 金鑰的詳細資訊，請參閱[在 Windows 上建立 SSH 金鑰](../../virtual-machines/linux/ssh-from-windows.md)。

## <a name="create-a-resource-group"></a>建立資源群組

資源群組是您可以在其中部署和管理 Azure Stack 資源的邏輯容器。 從您的開發套件或 Azure Stack 整合系統，執行 [az group create](/cli/azure/group#az-group-create) 命令來建立資源群組。

>[!NOTE]
 系統會為程式碼範例中的所有變數指派值。 不過，您可以視需要指派新值。

下列範例會在本機位置建立名為 myResourceGroup 的資源群組。

```cli
az group create --name myResourceGroup --location local
```

## <a name="create-a-virtual-machine"></a>建立虛擬機器

使用 [az vm create](/cli/azure/vm#az-vm-create) 命令來建立虛擬機器。 下列範例會建立名為 myVM 的 VM。 此範例會使用 Demouser 作為系統管理使用者名稱，並使用 Demouser@123 作為使用者密碼。 將這些值變更為適合您環境的值。

```cli
az vm create \
  --resource-group "myResourceGroup" \
  --name "myVM" \
  --image "UbuntuLTS" \
  --admin-username "Demouser" \
  --admin-password "Demouser@123" \
  --use-unmanaged-disk \
  --location local
```

公用 IP 位址會在 **PublicIpAddress** 參數中傳回。 請記下此位址，因為您需要它才能存取虛擬機器。

## <a name="open-port-80-for-web-traffic"></a>針對 Web 流量開啟連接埠 80

因為此虛擬機器即將執行 IIS 網頁伺服器，所以您需要對網際網路流量開啟連接埠 80。 使用 [az vm open-port](/cli/azure/vm#open-port) 命令來開啟所需的連接埠。

```cli
az vm open-port --port 80 --resource-group myResourceGroup --name myVM
```

## <a name="use-ssh-to-connect-to-the-virtual-machine"></a>使用 SSH 連線至虛擬機器

從已安裝 SSH 的用戶端電腦，連線到虛擬機器。 如果您使用的是 Windows 用戶端，請使用 [Putty](http://www.putty.org/) 來建立連線。 若要連線到虛擬機器，請使用下列命令：

```bash
ssh <publicIpAddress>
```

## <a name="install-the-nginx-web-server"></a>安裝 NGINX 網頁伺服器

若要更新套件資源及安裝最新的 NGINX 套件，請執行下列指令碼：

```bash
#!/bin/bash

# update package source
apt-get -y update

# install NGINX
apt-get -y install nginx
```

## <a name="view-the-nginx-welcome-page"></a>檢視 NGINX 歡迎使用頁面

在已於虛擬機器上安裝 NGINX 並開啟連接埠 80 的情況下，您可以使用虛擬機器的公用 IP 位址存取網頁伺服器。 請開啟網頁瀏覽器，然後瀏覽至 ```http://<public IP address>```。

![NGINX 網頁伺服器歡迎頁面](./media/azure-stack-quick-create-vm-linux-cli/nginx.png)

## <a name="clean-up-resources"></a>清除資源

清除您不再需要的資源。 您可以使用 [az group delete](/cli/azure/group#az-group-delete) 命令來移除這些資源。 若要刪除資源群組和其所有資源，請執行下列命令︰

```cli
az group delete --name myResourceGroup
```

## <a name="next-steps"></a>後續步驟

在本快速入門中，您已使用網頁伺服器部署基本的 Linux 伺服器虛擬機器。 若要深入了解 Azure Stack 虛擬機器，請繼續移至 [Azure Stack 中虛擬機器的考量](azure-stack-vm-considerations.md)。
