---
title: 使用 Terraform 在 Azure 中建立完整的 Linux VM | Microsoft Docs
description: 了解如何使用 Terraform 在 Azure 中建立及管理完整的 Linux 虛擬機器環境
services: virtual-machines-linux
documentationcenter: virtual-machines
author: echuvyrov
manager: jeconnoc
editor: na
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 09/14/2017
ms.author: echuvyrov
ms.openlocfilehash: dfebda8f92837f8573fb3362c9210bce9b70d23d
ms.sourcegitcommit: b5ac31eeb7c4f9be584bb0f7d55c5654b74404ff
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/23/2018
ms.locfileid: "42751560"
---
# <a name="create-a-complete-linux-virtual-machine-infrastructure-in-azure-with-terraform"></a>在 Azure 中使用 Terraform 建立完整的 Linux 虛擬機器基礎結構

Terraform 可讓您在 Azure 中定義和建立完整的基礎結構部署。 您以人類看得懂的格式建立 Terraform 範本，以一致、可重現的方式建立和設定 Azure 資源。 本文說明如何使用 Terraform 建立完整的 Linux 環境與支援的資源。 您也可以了解如何[安裝及設定 Terraform](terraform-install-configure.md)。


## <a name="create-azure-connection-and-resource-group"></a>建立 Azure 連線和資源群組

讓我們逐步執行 Terraform 範本的每個章節。 您也會看到可複製並貼上之完整版本的 [Terraform 範本](#complete-terraform-script)。

`provider`一節說明如何將 Terraform 作為 Azure 提供者。 若要取得 *subscription_id*、*client_id*、*client_secret* 和 *tenant_id* 值，請參閱[安裝和設定 Terraform](terraform-install-configure.md)。 

> [!TIP]
> 如果您為這些值建立環境變數，或使用 [Azure Cloud Shell 體驗](/azure/cloud-shell/overview)，則不需要在此區段中併入變數宣告。

```tf
provider "azurerm" {
    subscription_id = "xxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
    client_id       = "xxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
    client_secret   = "xxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
    tenant_id       = "xxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
}
```

下一節會在 `eastus` 位置建立名為 `myResourceGroup` 的資源群組：

```tf
resource "azurerm_resource_group" "myterraformgroup" {
    name     = "myResourceGroup"
    location = "eastus"

    tags {
        environment = "Terraform Demo"
    }
}
```

在其他章節中，您會使用 *${azurerm_resource_group.myterraformgroup.name}* 參考資源群組。

## <a name="create-virtual-network"></a>建立虛擬網路
以下區段可在 *10.0.0.0/16* 位址空間中建立名為 *myVnet* 的虛擬網路：

```tf
resource "azurerm_virtual_network" "myterraformnetwork" {
    name                = "myVnet"
    address_space       = ["10.0.0.0/16"]
    location            = "eastus"
    resource_group_name = "${azurerm_resource_group.myterraformgroup.name}"

    tags {
        environment = "Terraform Demo"
    }
}
```

下列區段會在 *myVnet* 虛擬網路中建立名為 *mySubnet* 的子網路：

```tf
resource "azurerm_subnet" "myterraformsubnet" {
    name                 = "mySubnet"
    resource_group_name  = "${azurerm_resource_group.myterraformgroup.name}"
    virtual_network_name = "${azurerm_virtual_network.myterraformnetwork.name}"
    address_prefix       = "10.0.2.0/24"
}
```


## <a name="create-public-ip-address"></a>建立公用 IP 位址
若要存取網際網路上的資源，請建立公用 IP 位址並指派給您的 VM。 以下區段會建立名為 *myPublicIP* 的公用 IP 位址：

```tf
resource "azurerm_public_ip" "myterraformpublicip" {
    name                         = "myPublicIP"
    location                     = "eastus"
    resource_group_name          = "${azurerm_resource_group.myterraformgroup.name}"
    public_ip_address_allocation = "dynamic"

    tags {
        environment = "Terraform Demo"
    }
}
```


## <a name="create-network-security-group"></a>建立網路安全性群組
網路安全性群組會控制進出 VM 的網路流量。 以下區段會建立名為 *myNetworkSecurityGroup* 的網路安全性群組，並定義規則以允許 TCP 通訊埠 22 上的 SSH 流量：

```tf
resource "azurerm_network_security_group" "myterraformnsg" {
    name                = "myNetworkSecurityGroup"
    location            = "eastus"
    resource_group_name = "${azurerm_resource_group.myterraformgroup.name}"
    
    security_rule {
        name                       = "SSH"
        priority                   = 1001
        direction                  = "Inbound"
        access                     = "Allow"
        protocol                   = "Tcp"
        source_port_range          = "*"
        destination_port_range     = "22"
        source_address_prefix      = "*"
        destination_address_prefix = "*"
    }

    tags {
        environment = "Terraform Demo"
    }
}
```


## <a name="create-virtual-network-interface-card"></a>建立虛擬網路介面卡
虛擬網路介面卡 (NIC) 會將您的 VM 連線至指定的虛擬網路、公用 IP 位址，及網路安全性群組。 Terraform 範本中的下列區段會建立名為 myNIC 的虛擬 NIC，以連線至您所建立的虛擬網路資源：

```tf
resource "azurerm_network_interface" "myterraformnic" {
    name                = "myNIC"
    location            = "eastus"
    resource_group_name = "${azurerm_resource_group.myterraformgroup.name}"

    ip_configuration {
        name                          = "myNicConfiguration"
        subnet_id                     = "${azurerm_subnet.myterraformsubnet.id}"
        private_ip_address_allocation = "dynamic"
        public_ip_address_id          = "${azurerm_public_ip.myterraformpublicip.id}"
    }

    tags {
        environment = "Terraform Demo"
    }
}
```


## <a name="create-storage-account-for-diagnostics"></a>建立診斷用的儲存體帳戶
若要儲存 VM 的開機診斷，您需要儲存體帳戶。 這些開機診斷可協助您疑難排解問題及監視您的 VM 狀態。 您建立的儲存體帳戶只是用來儲存開機診斷資料。 由於每個儲存體帳戶都必須有唯一的名稱，因此下列區段會產生一些隨機文字：

```tf
resource "random_id" "randomId" {
    keepers = {
        # Generate a new ID only when a new resource group is defined
        resource_group = "${azurerm_resource_group.myterraformgroup.name}"
    }
    
    byte_length = 8
}
```

現在您可以建立儲存體帳戶。 以下區段會根據上一個步驟中產生的隨機文字的名稱來建立儲存體帳戶：

```tf
resource "azurerm_storage_account" "mystorageaccount" {
    name                = "diag${random_id.randomId.hex}"
    resource_group_name = "${azurerm_resource_group.myterraformgroup.name}"
    location            = "eastus"
    account_replication_type = "LRS"
    account_tier = "Standard"

    tags {
        environment = "Terraform Demo"
    }
}
```


## <a name="create-virtual-machine"></a>建立虛擬機器

最後一個步驟是建立 VM，然後使用建立的所有資源。 以下區段會建立名為 *myVM* 的 VM，並附加名為 *myNIC* 的虛擬 NIC。 會使用最新的 *Ubuntu 16.04-LTS* 映像，並在停用密碼驗證的情況下建立名為 *azureuser* 的使用者。

 SSH 金鑰資料是在 *ssh_keys* 中提供。 在 *key_data* 欄位中提供有效的公用 SSH 金鑰。

```tf
resource "azurerm_virtual_machine" "myterraformvm" {
    name                  = "myVM"
    location              = "eastus"
    resource_group_name   = "${azurerm_resource_group.myterraformgroup.name}"
    network_interface_ids = ["${azurerm_network_interface.myterraformnic.id}"]
    vm_size               = "Standard_DS1_v2"

    storage_os_disk {
        name              = "myOsDisk"
        caching           = "ReadWrite"
        create_option     = "FromImage"
        managed_disk_type = "Premium_LRS"
    }

    storage_image_reference {
        publisher = "Canonical"
        offer     = "UbuntuServer"
        sku       = "16.04.0-LTS"
        version   = "latest"
    }

    os_profile {
        computer_name  = "myvm"
        admin_username = "azureuser"
    }

    os_profile_linux_config {
        disable_password_authentication = true
        ssh_keys {
            path     = "/home/azureuser/.ssh/authorized_keys"
            key_data = "ssh-rsa AAAAB3Nz{snip}hwhqT9h"
        }
    }

    boot_diagnostics {
        enabled     = "true"
        storage_uri = "${azurerm_storage_account.mystorageaccount.primary_blob_endpoint}"
    }

    tags {
        environment = "Terraform Demo"
    }
}
```

## <a name="complete-terraform-script"></a>完成 Terraform 指令碼

若要將所有這些區段結合在一起，並查看運作中的 Terraform，請建立名為 *terraform_azure.tf* 的檔案並貼上下列內容：

```tf
variable "resourcename" {
  default = "myResourceGroup"
}

# Configure the Microsoft Azure Provider
provider "azurerm" {
    subscription_id = "xxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
    client_id       = "xxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
    client_secret   = "xxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
    tenant_id       = "xxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
}

# Create a resource group if it doesn’t exist
resource "azurerm_resource_group" "myterraformgroup" {
    name     = "myResourceGroup"
    location = "eastus"

    tags {
        environment = "Terraform Demo"
    }
}

# Create virtual network
resource "azurerm_virtual_network" "myterraformnetwork" {
    name                = "myVnet"
    address_space       = ["10.0.0.0/16"]
    location            = "eastus"
    resource_group_name = "${azurerm_resource_group.myterraformgroup.name}"

    tags {
        environment = "Terraform Demo"
    }
}

# Create subnet
resource "azurerm_subnet" "myterraformsubnet" {
    name                 = "mySubnet"
    resource_group_name  = "${azurerm_resource_group.myterraformgroup.name}"
    virtual_network_name = "${azurerm_virtual_network.myterraformnetwork.name}"
    address_prefix       = "10.0.1.0/24"
}

# Create public IPs
resource "azurerm_public_ip" "myterraformpublicip" {
    name                         = "myPublicIP"
    location                     = "eastus"
    resource_group_name          = "${azurerm_resource_group.myterraformgroup.name}"
    public_ip_address_allocation = "dynamic"

    tags {
        environment = "Terraform Demo"
    }
}

# Create Network Security Group and rule
resource "azurerm_network_security_group" "myterraformnsg" {
    name                = "myNetworkSecurityGroup"
    location            = "eastus"
    resource_group_name = "${azurerm_resource_group.myterraformgroup.name}"
    
    security_rule {
        name                       = "SSH"
        priority                   = 1001
        direction                  = "Inbound"
        access                     = "Allow"
        protocol                   = "Tcp"
        source_port_range          = "*"
        destination_port_range     = "22"
        source_address_prefix      = "*"
        destination_address_prefix = "*"
    }

    tags {
        environment = "Terraform Demo"
    }
}

# Create network interface
resource "azurerm_network_interface" "myterraformnic" {
    name                      = "myNIC"
    location                  = "eastus"
    resource_group_name       = "${azurerm_resource_group.myterraformgroup.name}"
    network_security_group_id = "${azurerm_network_security_group.myterraformnsg.id}"

    ip_configuration {
        name                          = "myNicConfiguration"
        subnet_id                     = "${azurerm_subnet.myterraformsubnet.id}"
        private_ip_address_allocation = "dynamic"
        public_ip_address_id          = "${azurerm_public_ip.myterraformpublicip.id}"
    }

    tags {
        environment = "Terraform Demo"
    }
}

# Generate random text for a unique storage account name
resource "random_id" "randomId" {
    keepers = {
        # Generate a new ID only when a new resource group is defined
        resource_group = "${azurerm_resource_group.myterraformgroup.name}"
    }
    
    byte_length = 8
}

# Create storage account for boot diagnostics
resource "azurerm_storage_account" "mystorageaccount" {
    name                        = "diag${random_id.randomId.hex}"
    resource_group_name         = "${azurerm_resource_group.myterraformgroup.name}"
    location                    = "eastus"
    account_tier                = "Standard"
    account_replication_type    = "LRS"

    tags {
        environment = "Terraform Demo"
    }
}

# Create virtual machine
resource "azurerm_virtual_machine" "myterraformvm" {
    name                  = "myVM"
    location              = "eastus"
    resource_group_name   = "${azurerm_resource_group.myterraformgroup.name}"
    network_interface_ids = ["${azurerm_network_interface.myterraformnic.id}"]
    vm_size               = "Standard_DS1_v2"

    storage_os_disk {
        name              = "myOsDisk"
        caching           = "ReadWrite"
        create_option     = "FromImage"
        managed_disk_type = "Premium_LRS"
    }

    storage_image_reference {
        publisher = "Canonical"
        offer     = "UbuntuServer"
        sku       = "16.04.0-LTS"
        version   = "latest"
    }

    os_profile {
        computer_name  = "myvm"
        admin_username = "azureuser"
    }

    os_profile_linux_config {
        disable_password_authentication = true
        ssh_keys {
            path     = "/home/azureuser/.ssh/authorized_keys"
            key_data = "ssh-rsa AAAAB3Nz{snip}hwhqT9h"
        }
    }

    boot_diagnostics {
        enabled = "true"
        storage_uri = "${azurerm_storage_account.mystorageaccount.primary_blob_endpoint}"
    }

    tags {
        environment = "Terraform Demo"
    }
}
```


## <a name="build-and-deploy-the-infrastructure"></a>建立和部署基礎結構
使用建立的 Terraform 範本，第一個步驟是初始化 Terraform。 這個步驟可確保 Terraform 已具備在 Azure 中建立範本的所有必要條件。

```bash
terraform init
```

下一個步驟是讓 Terraform 檢閱及驗證範本。 這個步驟會比較要求的資源與 Terraform 儲存的狀態資訊，然後輸出規劃的執行。 *不會*在 Azure 中建立資源。

```bash
terraform plan
```

執行上述命令之後，您應該會看到如下列畫面的內容：

```bash
Refreshing Terraform state in-memory prior to plan...
The refreshed state will be used to calculate this plan, but will not be
persisted to local or remote state storage.


...

Note: You didn’t specify an “-out” parameter to save this plan, so when
“apply” is called, Terraform can’t guarantee this is what will execute.
  + azurerm_resource_group.myterraform
      <snip>
  + azurerm_virtual_network.myterraformnetwork
      <snip>
  + azurerm_network_interface.myterraformnic
      <snip>
  + azurerm_network_security_group.myterraformnsg
      <snip>
  + azurerm_public_ip.myterraformpublicip
      <snip>
  + azurerm_subnet.myterraformsubnet
      <snip>
  + azurerm_virtual_machine.myterraformvm
      <snip>
Plan: 7 to add, 0 to change, 0 to destroy.
```

如果一切看起來正確，而且您已準備好在 Azure 中組建基礎結構，請在 Terraform 中套用範本：

```bash
terraform apply
```

Terraform 完成之後，您的 VM 基礎結構即已備妥。 使用 [az vm show](/cli/azure/vm#az_vm_show) 取得虛擬機器的 IP 位址：

```azurecli
az vm show --resource-group myResourceGroup --name myVM -d --query [publicIps] --o tsv
```

之後您便可以透過 SSH 連線至您的 VM：

```bash
ssh azureuser@<publicIps>
```

## <a name="next-steps"></a>後續步驟
您已使用 Terraform 在 Azure 中建立基本基礎結構。 如需更多複雜案例 (包括使用負載平衡器、虛擬機器擴展集的範例)，請參閱許多[適用於 Azure 的 Terraform 範例](https://github.com/hashicorp/terraform/tree/master/examples)。 如需支援的 Azure 提供者最新清單，請參閱 [Terraform 文件](https://www.terraform.io/docs/providers/azurerm/index.html) \(英文\)。
