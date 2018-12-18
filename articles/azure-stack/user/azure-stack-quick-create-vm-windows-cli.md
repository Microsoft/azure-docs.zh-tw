---
title: 使用 Azure CLI 在 Azure Stack 上建立 Windows 虛擬機器 | Microsoft Docs
description: 了解如何使用 Azure CLI 在 Azure Stack 上建立 Windows VM
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila
editor: ''
ms.assetid: E26B246E-811D-44C9-9BA6-2B3CE5B62E83
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: quickstart
ms.date: 04/23/2018
ms.author: mabrigg
ms.custom: mvc
ms.openlocfilehash: 4d062b937eb005b8c30a6f7f59b00f80095a7924
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39422285"
---
# <a name="quickstart-create-a-windows-server-virtual-machine-by-using-azure-cli-in-azure-stack"></a>快速入門：在 Azure Stack 中使用 Azure CLI 建立 Windows Server 虛擬機器

*適用於：Azure Stack 整合系統和 Azure Stack 開發套件*

您可以使用 Azure CLI 建立 Windows Server 2016 虛擬機器。 請遵循本文中的步驟建立和使用虛擬機器。 本文也提供下列步驟：

* 使用遠端用戶端連線到虛擬機器。
* 安裝 IIS 網頁伺服器，並且檢視預設首頁。
* 清除資源。

## <a name="prerequisites"></a>必要條件

* 確定您的 Azure Stack 操作員已將**Windows Server 2016** 映像新增至 Azure Stack Marketplace。

* Azure Stack 需要特定版本的 Azure CLI，才能建立和管理資源。 如果您尚未針對 Azure Stack 設定 Azure CLI，請依照步驟來[安裝和設定 Azure CLI](azure-stack-version-profiles-azurecli2.md)。

## <a name="create-a-resource-group"></a>建立資源群組

資源群組是您可以在其中部署和管理 Azure Stack 資源的邏輯容器。 從 Azure Stack 環境，執行 [az group create](/cli/azure/group#az-group-create) 命令來建立資源群組。

>[!NOTE]
 系統會為程式碼範例中的所有變數指派值。 不過，您可以視需要指派新值。

下列範例會在本機位置建立名為 myResourceGroup 的資源群組。

```cli
az group create --name myResourceGroup --location local
```

## <a name="create-a-virtual-machine"></a>建立虛擬機器

使用 [az vm create](/cli/azure/vm#az-vm-create) 命令來建立虛擬機器 (VM)。 下列範例會建立名為 myVM 的 VM。 此範例會使用 Demouser 作為系統管理使用者名稱，並使用 Demouser@123 作為使用者密碼。 將這些值變更為適合您環境的值。

```cli
az vm create \
  --resource-group "myResourceGroup" \
  --name "myVM" \
  --image "Win2016Datacenter" \
  --admin-username "Demouser" \
  --admin-password "Demouser@123" \
  --use-unmanaged-disk \
  --location local
```

建立 VM 時，輸出中的 **PublicIPAddress** 參數包含虛擬機器的公用 IP 位址。 請記下此位址，因為您需要它才能存取虛擬機器。

## <a name="open-port-80-for-web-traffic"></a>針對 Web 流量開啟連接埠 80

因為此 VM 即將執行 IIS 網頁伺服器，所以您需要對網際網路流量開啟連接埠 80。

使用 [az vm open-port](/cli/azure/vm#open-port) 命令開啟連接埠 80。

```cli
az vm open-port --port 80 --resource-group myResourceGroup --name myVM
```

## <a name="connect-to-the-virtual-machine"></a>連接至虛擬機器

使用下一個命令來建立與虛擬機器的遠端桌面連線。 以虛擬機器的 IP 位址取代 [公用 IP 位址]。 出現提示時，輸入您用於虛擬機器的使用者名稱和密碼。

```
mstsc /v <Public IP Address>
```

## <a name="install-iis-using-powershell"></a>使用 PowerShell 安裝 IIS

既然您已登入虛擬機器，您可以使用 PowerShell 來安裝 IIS。 在虛擬機器上啟動 PowerShell 並執行下列命令：

```powershell
Install-WindowsFeature -name Web-Server -IncludeManagementTools
```

## <a name="view-the-iis-welcome-page"></a>檢視 IIS 歡迎使用頁面

您可以使用所選的網頁瀏覽器來查看預設 IIS 歡迎使用頁面。 使用在前一節中記載的公用 IP 位址來瀏覽預設網頁。

![IIS 預設網站](./media/azure-stack-quick-create-vm-windows-cli/default-iis-website.png)

## <a name="clean-up-resources"></a>清除資源

清除您不再需要的資源。 使用 [az group delete](/cli/azure/group#az-group-delete) 命令來移除資源群組、虛擬機器，以及所有相關資源。

```cli
az group delete --name myResourceGroup
```

## <a name="next-steps"></a>後續步驟

在這個快速入門中，您已部署基本 Windows Server 虛擬機器。 若要深入了解 Azure Stack 虛擬機器，請繼續移至 [Azure Stack 中虛擬機器的考量](azure-stack-vm-considerations.md)。
