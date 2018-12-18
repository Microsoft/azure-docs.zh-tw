---
title: 如何使用 Packer 來建立 Linux Azure VM 映像 | Microsoft Docs
description: 了解如何在 Azure 中使用 Packer 建立 Linux 虛擬機器的映像
services: virtual-machines-linux
documentationcenter: virtual-machines
author: cynthn
manager: jeconnoc
editor: tysonn
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-linux
ms.devlang: azurecli
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 05/03/2018
ms.author: cynthn
ms.openlocfilehash: f84626c5a487d52f53a2c8bf492a124c87599ed0
ms.sourcegitcommit: aa988666476c05787afc84db94cfa50bc6852520
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/10/2018
ms.locfileid: "37932388"
---
# <a name="how-to-use-packer-to-create-linux-virtual-machine-images-in-azure"></a>如何在 Azure 中使用 Packer 來建立 Linux 虛擬機器映像
Azure 中的每個虛擬機器 (VM) 都是透過映像所建立，而映像則會定義 Linux 散發套件和作業系統版本。 映像中可包含預先安裝的應用程式與組態。 Azure Marketplace 提供了許多第一方和第三方映像，這些映像適用於最常見的散發套件和應用程式環境，而您也可以建立自己自訂的映像，以符合您的需求。 本文詳述如何使用開放原始碼工具 [Packer](https://www.packer.io/)，在 Azure 中定義並建置自訂映像。


## <a name="create-azure-resource-group"></a>建立 Azure 資源群組
建置程序進行期間，Packer 會在建置來源 VM 時建立暫存的 Azure 資源。 若要擷取該來源 VM 以作為映像，您必須定義資源群組。 Packer 建置程序所產生的輸出會儲存在此資源群組中。

使用 [az group create](/cli/azure/group#az_group_create) 來建立資源群組。 下列範例會在 eastus 位置建立名為 myResourceGroup 的資源群組：

```azurecli
az group create -n myResourceGroup -l eastus
```


## <a name="create-azure-credentials"></a>建立 Azure 認證
Packer 會使用服務主體來向 Azure 驗證。 Azure 服務主體是安全性識別，可供您與應用程式、服務及諸如 Packer 等自動化工具搭配使用。 您可以控制和定義對於服務主體可以在 Azure 中執行哪些作業的權限。

使用 [az ad sp create-for-rbac](/cli/azure/ad/sp#create-for-rbac) 建立服務主體，並將 Packer 所需的認證輸出：

```azurecli
az ad sp create-for-rbac --query "{ client_id: appId, client_secret: password, tenant_id: tenant }"
```

上述命令的輸出範例如下所示：

```azurecli
{
    "client_id": "f5b6a5cf-fbdf-4a9f-b3b8-3c2cd00225a4",
    "client_secret": "0e760437-bf34-4aad-9f8d-870be799c55d",
    "tenant_id": "72f988bf-86f1-41af-91ab-2d7cd011db47"
}
```

若要向 Azure 驗證，您也需要使用 [az account show](/cli/azure/account#az_account_show) 取得 Azure 訂用帳戶識別碼：

```azurecli
az account show --query "{ subscription_id: id }"
```

您將在下一個步驟中使用這兩個命令的輸出。


## <a name="define-packer-template"></a>定義 Packer 範本
若要建置映像，您可以將範本建立為 JSON 檔案。 在此範本中，您必須定義產生器和佈建程式，由它們執行實際的建置程序。 Packer 具有[適用於 Azure 的佈建程式](https://www.packer.io/docs/builders/azure.html)，以供您定義 Azure 資源，例如上述步驟所建立的服務主體認證。

建立名為 ubuntu.json 的檔案，並貼入下列內容。 針對下列參數輸入您自己的值︰

| 參數                           | 取得位置 |
|-------------------------------------|----------------------------------------------------|
| client_id                         | `az ad sp` 建立命令所產生之輸出的第一行 - appId |
| client_secret                     | `az ad sp` 建立命令所產生之輸出的第二行 - password |
| tenant_id                         | `az ad sp` 建立命令所產生之輸出的第三行 - tenant |
| subscription_id                   | `az account show` 命令所產生的輸出 |
| managed_image_resource_group_name | 您在第一個步驟中建立的資源群組名稱 |
| managed_image_name                | 所建立之受控磁碟映像的名稱 |


```json
{
  "builders": [{
    "type": "azure-arm",

    "client_id": "f5b6a5cf-fbdf-4a9f-b3b8-3c2cd00225a4",
    "client_secret": "0e760437-bf34-4aad-9f8d-870be799c55d",
    "tenant_id": "72f988bf-86f1-41af-91ab-2d7cd011db47",
    "subscription_id": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx",

    "managed_image_resource_group_name": "myResourceGroup",
    "managed_image_name": "myPackerImage",

    "os_type": "Linux",
    "image_publisher": "Canonical",
    "image_offer": "UbuntuServer",
    "image_sku": "16.04-LTS",

    "azure_tags": {
        "dept": "Engineering",
        "task": "Image deployment"
    },

    "location": "East US",
    "vm_size": "Standard_DS2_v2"
  }],
  "provisioners": [{
    "execute_command": "chmod +x {{ .Path }}; {{ .Vars }} sudo -E sh '{{ .Path }}'",
    "inline": [
      "apt-get update",
      "apt-get upgrade -y",
      "apt-get -y install nginx",

      "/usr/sbin/waagent -force -deprovision+user && export HISTSIZE=0 && sync"
    ],
    "inline_shebang": "/bin/sh -x",
    "type": "shell"
  }]
}
```

此範本會建置 Ubuntu 16.04 LTS 映像、安裝 NGINX，然後取消佈建 VM。

> [!NOTE]
> 如果您擴充此範本來佈建使用者認證，請將取消佈建 Azure 代理程式的佈建程式命令調整為讀取 `-deprovision` 而不是 `deprovision+user`。
> `+user` 旗標會從來源 VM 中移除所有使用者帳戶。


## <a name="build-packer-image"></a>建置 Packer 映像
如果您尚未在本機電腦上安裝 Packer，請[遵循 Packer 安裝指示](https://www.packer.io/docs/install/index.html)。

請指定 Packer 範本檔案來建置映像，如下所示：

```bash
./packer build ubuntu.json
```

上述命令的輸出範例如下所示：

```bash
azure-arm output will be in this color.

==> azure-arm: Running builder ...
    azure-arm: Creating Azure Resource Manager (ARM) client ...
==> azure-arm: Creating resource group ...
==> azure-arm:  -> ResourceGroupName : ‘packer-Resource-Group-swtxmqm7ly’
==> azure-arm:  -> Location          : ‘East US’
==> azure-arm:  -> Tags              :
==> azure-arm:  ->> dept : Engineering
==> azure-arm:  ->> task : Image deployment
==> azure-arm: Validating deployment template ...
==> azure-arm:  -> ResourceGroupName : ‘packer-Resource-Group-swtxmqm7ly’
==> azure-arm:  -> DeploymentName    : ‘pkrdpswtxmqm7ly’
==> azure-arm: Deploying deployment template ...
==> azure-arm:  -> ResourceGroupName : ‘packer-Resource-Group-swtxmqm7ly’
==> azure-arm:  -> DeploymentName    : ‘pkrdpswtxmqm7ly’
==> azure-arm: Getting the VM’s IP address ...
==> azure-arm:  -> ResourceGroupName   : ‘packer-Resource-Group-swtxmqm7ly’
==> azure-arm:  -> PublicIPAddressName : ‘packerPublicIP’
==> azure-arm:  -> NicName             : ‘packerNic’
==> azure-arm:  -> Network Connection  : ‘PublicEndpoint’
==> azure-arm:  -> IP Address          : ‘40.76.218.147’
==> azure-arm: Waiting for SSH to become available...
==> azure-arm: Connected to SSH!
==> azure-arm: Provisioning with shell script: /var/folders/h1/ymh5bdx15wgdn5hvgj1wc0zh0000gn/T/packer-shell868574263
    azure-arm: WARNING! The waagent service will be stopped.
    azure-arm: WARNING! Cached DHCP leases will be deleted.
    azure-arm: WARNING! root password will be disabled. You will not be able to login as root.
    azure-arm: WARNING! /etc/resolvconf/resolv.conf.d/tail and /etc/resolvconf/resolv.conf.d/original will be deleted.
    azure-arm: WARNING! packer account and entire home directory will be deleted.
==> azure-arm: Querying the machine’s properties ...
==> azure-arm:  -> ResourceGroupName : ‘packer-Resource-Group-swtxmqm7ly’
==> azure-arm:  -> ComputeName       : ‘pkrvmswtxmqm7ly’
==> azure-arm:  -> Managed OS Disk   : ‘/subscriptions/guid/resourceGroups/packer-Resource-Group-swtxmqm7ly/providers/Microsoft.Compute/disks/osdisk’
==> azure-arm: Powering off machine ...
==> azure-arm:  -> ResourceGroupName : ‘packer-Resource-Group-swtxmqm7ly’
==> azure-arm:  -> ComputeName       : ‘pkrvmswtxmqm7ly’
==> azure-arm: Capturing image ...
==> azure-arm:  -> Compute ResourceGroupName : ‘packer-Resource-Group-swtxmqm7ly’
==> azure-arm:  -> Compute Name              : ‘pkrvmswtxmqm7ly’
==> azure-arm:  -> Compute Location          : ‘East US’
==> azure-arm:  -> Image ResourceGroupName   : ‘myResourceGroup’
==> azure-arm:  -> Image Name                : ‘myPackerImage’
==> azure-arm:  -> Image Location            : ‘eastus’
==> azure-arm: Deleting resource group ...
==> azure-arm:  -> ResourceGroupName : ‘packer-Resource-Group-swtxmqm7ly’
==> azure-arm: Deleting the temporary OS disk ...
==> azure-arm:  -> OS Disk : skipping, managed disk was used...
Build ‘azure-arm’ finished.

==> Builds finished. The artifacts of successful builds are:
--> azure-arm: Azure.ResourceManagement.VMImage:

ManagedImageResourceGroupName: myResourceGroup
ManagedImageName: myPackerImage
ManagedImageLocation: eastus
```

Packer 需要幾分鐘的時間來建置 VM、執行佈建程式並清除部署。


## <a name="create-vm-from-azure-image"></a>從 Azure 映像建立 VM
您現在可以使用 [az vm create](/cli/azure/vm#az_vm_create) 從您的映像建立 VM。 指定您使用 `--image` 參數所建立的映像。 下列範例會從 myPackerImage 建立名為 myVM 的 VM，並產生 SSH 金鑰 (如果您還未擁有這些金鑰的話)︰

```azurecli
az vm create \
    --resource-group myResourceGroup \
    --name myVM \
    --image myPackerImage \
    --admin-username azureuser \
    --generate-ssh-keys
```

如果想要在不同於您 Packer 映像的資源群組或區域中建立 VM，請指定映像識別碼，而非映像名稱。 您可以使用 [az image show](/cli/azure/image#az-image-show) 取得映像識別碼。

建立 VM 需要幾分鐘的時間。 在 VM 建立好之後，請記下 Azure CLI 所顯示的 `publicIpAddress`。 此位址可用來透過 Web 瀏覽器存取 NGINX 網站。

若要讓 Web 流量到達您的 VM，請使用 [az vm open-port](/cli/azure/vm#open-port) 從網際網路開啟通訊埠 80：

```azurecli
az vm open-port \
    --resource-group myResourceGroup \
    --name myVM \
    --port 80
```

## <a name="test-vm-and-nginx"></a>測試 VM 和 NGINX
現在，您可以開啟 Web 瀏覽器，並在網址列輸入 `http://publicIpAddress`。 提供您自己從 VM 建立程序中取得的公用 IP 位址。 預設 NGINX 網頁即會顯示，如下列範例所示：

![預設 NGINX 網站](./media/build-image-with-packer/nginx.png) 


## <a name="next-steps"></a>後續步驟
在此範例中，您已使用 Packer 來建立已安裝 NGINX 的 VM 映像。 您可以使用這個 VM 映像以及現有的部署工作流程，來進行「將應用程式部署至使用 Ansible、Chef 或 Puppet 從映像所建立的 VM」之類的作業。

如需其他 Linux 散發套件的其他 Packer 範本範例，請參閱[這個 GitHub 存放庫](https://github.com/hashicorp/packer/tree/master/examples/azure)。