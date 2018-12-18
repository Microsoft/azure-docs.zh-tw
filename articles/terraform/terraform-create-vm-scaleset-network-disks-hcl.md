---
title: 使用 Terraform 來建立 Azure 虛擬機器擴展集
description: 關於使用 Terraform 設定以虛擬網路和受控已連結磁碟完成的 Azure 虛擬機器擴展集，並設定版本的教學課程
services: terraform
ms.service: terraform
keywords: terraform, devops, 虛擬機器, Azure, 擴展集, 網路, 儲存體, 模組
author: tomarcher
manager: jeconnoc
ms.author: tarcher
ms.topic: tutorial
ms.date: 06/04/2018
ms.openlocfilehash: 7ae97274b03dda4dcf5150c8faacc7d406dad9fd
ms.sourcegitcommit: f20e43e436bfeafd333da75754cd32d405903b07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49389610"
---
# <a name="use-terraform-to-create-an-azure-virtual-machine-scale-set"></a>使用 Terraform 來建立 Azure 虛擬機器擴展集

[Azure 虛擬機器擴展集](/azure/virtual-machine-scale-sets)可讓您建立和管理一組完全相同、經過負載平衡的虛擬機器，其中的虛擬機器執行個體數目可以自動增加，或回應要求或定義的排程而減少。 

在本教學課程中，您會了解如何使用 [Azure Cloud Shell](/azure/cloud-shell/overview) 來執行下列工作：

> [!div class="checklist"]
> * 設定 Terraform 部署
> * 針對 Terraform 部署使用變數和輸出 
> * 建立和部署網路基礎結構
> * 建立和部署虛擬機器擴展集，並將它連接到網路
> * 建立和部署 Jumpbox 以透過 SSH 連線到 VM

> [!NOTE]
> 本文中使用的最新版本 Terraform 組態檔位於 [Github 上很棒的 Terraform 存放庫](https://github.com/Azure/awesome-terraform/tree/master/codelab-vmss)中。

## <a name="prerequisites"></a>必要條件

- **Azure 訂用帳戶**：如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio)。

- **安裝 Terraform**：請遵循 [Terraform 和設定 Azure 的存取](/azure/virtual-machines/linux/terraform-install-configure)一文中的指示

- **建立 SSH 金鑰組**：如果您還沒有 SSH 金鑰組，請遵循[如何在 Azure 中建立和使用 Linux 虛擬機器的 SSH 公開和私密金鑰組](https://docs.microsoft.com/azure/virtual-machines/linux/mac-create-ssh-keys)一文中的指示。

## <a name="create-the-directory-structure"></a>建立目錄結構

1. 瀏覽至 [Azure 入口網站](http://portal.azure.com)。

1. 開啟 [Azure Cloud Shell](/azure/cloud-shell/overview)。 如果您先前未選取環境，請選取 **Bash** 作為您的環境。

    ![Cloud Shell 提示](./media/terraform-create-vm-scaleset-network-disks-hcl/azure-portal-cloud-shell-button-min.png)

1. 切換至 `clouddrive` 目錄。

    ```bash
    cd clouddrive
    ```

1. 建立名為 `vmss` 的目錄。

    ```bash
    mkdir vmss
    ```

1. 將目錄變更為新的目錄：

    ```bash
    cd vmss
    ```

## <a name="create-the-variables-definitions-file"></a>建立變數定義檔
在本節中，您要定義可自訂 Terraform 所建立資源的變數。

在 Azure Cloud Shell 內，執行下列步驟：

1. 建立名為 `variables.tf` 的檔案。

    ```bash
    vi variables.tf
    ```

1. 選取 I 鍵輸入插入模式。

1. 將下列程式碼貼到編輯器中：

  ```JSON
  variable "location" {
    description = "The location where resources will be created"
  }

  variable "tags" {
    description = "A map of the tags to use for the resources that are deployed"
    type        = "map"

    default = {
      environment = "codelab"
    }
  }

  variable "resource_group_name" {
    description = "The name of the resource group in which the resources will be created"
    default     = "myResourceGroup"
  }
  ```

1. 選取 Esc 鍵結束插入模式。

1. 輸入下列命令來儲存檔案及結束 vi 編輯器：

    ```bash
    :wq
    ```

## <a name="create-the-output-definitions-file"></a>建立輸出定義檔
在本節中，您要建立可描述部署後輸出的檔案。

在 Azure Cloud Shell 內，執行下列步驟：

1. 建立名為 `output.tf` 的檔案。

    ```bash
    vi output.tf
    ```

1. 選取 I 鍵輸入插入模式。

1. 請將下列程式碼貼到編輯器，以公開虛擬機器的完整網域名稱 (FQDN)。 ：

  ```JSON
    output "vmss_public_ip" {
        value = "${azurerm_public_ip.vmss.fqdn}"
    }
  ```

1. 選取 Esc 鍵結束插入模式。

1. 輸入下列命令來儲存檔案及結束 vi 編輯器：

    ```bash
    :wq
    ```

## <a name="define-the-network-infrastructure-in-a-template"></a>在範本中定義網路基礎結構
在本節中，您要在新的 Azure 資源群組中建立下列網路基礎結構： 

  - 一個具有 10.0.0.0/16 位址空間的虛擬網路 (VNET) 
  - 一個具有 10.0.2.0/24 位址空間的子網路
  - 兩個公用 IP 位址。 其中一個是用於虛擬機器擴展集負載平衡器，另一個用於連線到 SSH Jubmox。

在 Azure Cloud Shell 內，執行下列步驟：

1. 建立名為 `vmss.tf` 的檔案來描述虛擬機器擴展集基礎結構。

    ```bash
    vi vmss.tf
    ```

1. 選取 I 鍵輸入插入模式。

1. 請將下列程式碼貼到檔案的結尾，以公開虛擬機器的完整網域名稱 (FQDN)。 

  ```JSON
  resource "azurerm_resource_group" "vmss" {
    name     = "${var.resource_group_name}"
    location = "${var.location}"
    tags     = "${var.tags}"
  }

  resource "random_string" "fqdn" {
    length  = 6
    special = false
    upper   = false
    number  = false
  }

  resource "azurerm_virtual_network" "vmss" {
    name                = "vmss-vnet"
    address_space       = ["10.0.0.0/16"]
    location            = "${var.location}"
    resource_group_name = "${azurerm_resource_group.vmss.name}"
    tags                = "${var.tags}"
  }

  resource "azurerm_subnet" "vmss" {
    name                 = "vmss-subnet"
    resource_group_name  = "${azurerm_resource_group.vmss.name}"
    virtual_network_name = "${azurerm_virtual_network.vmss.name}"
    address_prefix       = "10.0.2.0/24"
  }

  resource "azurerm_public_ip" "vmss" {
    name                         = "vmss-public-ip"
    location                     = "${var.location}"
    resource_group_name          = "${azurerm_resource_group.vmss.name}"
    public_ip_address_allocation = "static"
    domain_name_label            = "${random_string.fqdn.result}"
    tags                         = "${var.tags}"
  }
  ```

1. 選取 Esc 鍵結束插入模式。

1. 輸入下列命令來儲存檔案及結束 vi 編輯器：

  ```bash
  :wq
  ```

## <a name="provision-the-network-infrastructure"></a>佈建網路基礎結構
在您建立組態檔 (.tf) 的目錄中，使用 Azure Cloud Shell 執行下列步驟：

1. 初始化 Terraform。

  ```bash
  terraform init 
  ```

1. 執行下列命令，以在 Azure 中部署定義的基礎結構。

  ```bash
  terraform apply
  ```

  Terraform 會提示您輸入「位置」值，因為 **location** 變數會在 `variables.tf` 中定義，但一律不會設定。 您可以輸入任何有效的位置 - 例如「美國西部」，接著選取 [輸入]。 (使用括號括住任何包含空格的值。)

1. Terraform 會列印 `output.tf` 檔案中所定義的輸出。 如下列螢幕擷取畫面所示，FQDN 會採用 &lt;id>.&lt;location>.cloudapp.azure.com 的格式。 ID 值是經計算的值，而位置則是您在執行 Terraform 時所提供的值。

  ![公用 IP 位址的虛擬機器擴展集完整網域名稱](./media/terraform-create-vm-scaleset-network-disks-hcl/fqdn.png)

1. 在 Azure 入口網站功能表的主功能表中，選取 [資源群組]。

1. 在 [資源群組] 索引標籤上，選取 **myResourceGroup** 以檢視 Terraform 所建立的資源。
  ![虛擬機器擴展集網路資源](./media/terraform-create-vm-scaleset-network-disks-hcl/resource-group-resources.png)

## <a name="add-a-virtual-machine-scale-set"></a>新增虛擬機器擴展集

在本節中，您會了解如何將下列資源新增到範本：

- Azure 負載平衡器，以及用以提供務應用程式，並將負載平衡器連結到本文中稍早所設定公用 IP 位址的規則
- Azure 後端位址集區，並將其指派給負載平衡器 
- 應用程式所使用且在負載平衡器上設定的健康情況探查連接埠 
- 位在負載平衡器幕後的虛擬機器擴展集，在本文中稍早部署的 VNET 上執行
- 在使用 [cloud-init](http://cloudinit.readthedocs.io/en/latest/) 的虛擬機器擴展集節點上的 [Nginx](http://nginx.org/)。

在 Cloud Shell 中，執行下列步驟：

1. 開啟 `vmss.tf` 組態檔。

  ```bash
  vi vmss.tf
  ```

1. 請移至檔案的結尾，並選取 A 鍵來進入附加模式。

1. 將下列程式碼貼到檔案的結尾：

  ```JSON
  resource "azurerm_lb" "vmss" {
    name                = "vmss-lb"
    location            = "${var.location}"
    resource_group_name = "${azurerm_resource_group.vmss.name}"

    frontend_ip_configuration {
      name                 = "PublicIPAddress"
      public_ip_address_id = "${azurerm_public_ip.vmss.id}"
    }

    tags = "${var.tags}"
  }

  resource "azurerm_lb_backend_address_pool" "bpepool" {
    resource_group_name = "${azurerm_resource_group.vmss.name}"
    loadbalancer_id     = "${azurerm_lb.vmss.id}"
    name                = "BackEndAddressPool"
  }

  resource "azurerm_lb_probe" "vmss" {
    resource_group_name = "${azurerm_resource_group.vmss.name}"
    loadbalancer_id     = "${azurerm_lb.vmss.id}"
    name                = "ssh-running-probe"
    port                = "${var.application_port}"
  }

  resource "azurerm_lb_rule" "lbnatrule" {
      resource_group_name            = "${azurerm_resource_group.vmss.name}"
      loadbalancer_id                = "${azurerm_lb.vmss.id}"
      name                           = "http"
      protocol                       = "Tcp"
      frontend_port                  = "${var.application_port}"
      backend_port                   = "${var.application_port}"
      backend_address_pool_id        = "${azurerm_lb_backend_address_pool.bpepool.id}"
      frontend_ip_configuration_name = "PublicIPAddress"
      probe_id                       = "${azurerm_lb_probe.vmss.id}"
  }

  resource "azurerm_virtual_machine_scale_set" "vmss" {
    name                = "vmscaleset"
    location            = "${var.location}"
    resource_group_name = "${azurerm_resource_group.vmss.name}"
    upgrade_policy_mode = "Manual"

    sku {
      name     = "Standard_DS1_v2"
      tier     = "Standard"
      capacity = 2
    }

    storage_profile_image_reference {
      publisher = "Canonical"
      offer     = "UbuntuServer"
      sku       = "16.04-LTS"
      version   = "latest"
    }

    storage_profile_os_disk {
      name              = "osdisk"
      caching           = "ReadWrite"
      create_option     = "FromImage"
      managed_disk_type = "Standard_LRS"
    }

    storage_profile_data_disk {
      lun          = 0
      caching        = "ReadWrite"
      create_option  = "Empty"
      disk_size_gb   = 10
    }

    os_profile {
      computer_name_prefix = "vmlab"
      admin_username       = "${var.admin_user}"
      admin_password       = "${var.admin_password}"
      custom_data          = "${file("web.conf")}"
    }

    os_profile_linux_config {
      disable_password_authentication = false
    }

    network_profile {
      name    = "terraformnetworkprofile"
      primary = true

      ip_configuration {
        name                                   = "IPConfiguration"
        subnet_id                              = "${azurerm_subnet.vmss.id}"
        load_balancer_backend_address_pool_ids = ["${azurerm_lb_backend_address_pool.bpepool.id}"]
      }
    }

    tags = "${var.tags}"
}
  ```

1. 選取 Esc 鍵結束插入模式。

1. 輸入下列命令來儲存檔案及結束 vi 編輯器：

    ```bash
    :wq
    ```

1. 建立名為 `web.conf` 的檔案，作為擴展集所屬虛擬機器的 cloud-init 組態。 

    ```bash
    vi web.conf
    ```

1. 選取 I 鍵輸入插入模式。

1. 將下列程式碼貼到編輯器中：

  ```JSON
  #cloud-config
  packages:
    - nginx
  ```

1. 選取 Esc 鍵結束插入模式。

1. 輸入下列命令來儲存檔案及結束 vi 編輯器：

    ```bash
    :wq
    ```

1. 開啟 `variables.tf` 組態檔。

  ```bash
  vi variables.tf
  ```

1. 請移至檔案的結尾，並選取 A 鍵來進入附加模式。

1. 將下列程式碼貼到檔案的結尾來自訂部署：

  ```JSON
  variable "application_port" {
      description = "The port that you want to expose to the external load balancer"
      default     = 80
  }

  variable "admin_user" {
      description = "User name to use as the admin account on the VMs that will be part of the VM Scale Set"
      default     = "azureuser"
  }

  variable "admin_password" {
      description = "Default password for admin account"
  }
  ``` 

1. 選取 Esc 鍵結束插入模式。

1. 輸入下列命令來儲存檔案及結束 vi 編輯器：

    ```bash
    :wq
    ```

1. 建立 Terraform 計劃以視覺化虛擬機器擴展集部署。 (您需要指定您選擇的密碼及資源的位置)。

  ```bash
  terraform plan
  ```

  此命令的輸出應類似下列螢幕擷取畫面：

  ![建立虛擬機器擴展集的輸出](./media/terraform-create-vm-scaleset-network-disks-hcl/add-mvss-plan.png)

1. 在 Azure 中部署新的資源。

  ```bash
  terraform apply 
  ```

  此命令的輸出應類似下列螢幕擷取畫面：

  ![Terraform 虛擬機器擴展集資源群組](./media/terraform-create-vm-scaleset-network-disks-hcl/resource-group-contents.png)

1. 開啟瀏覽器並連線到命令傳回的 FQDN。 

  ![瀏覽至 FQDN 的結果](./media/terraform-create-vm-scaleset-network-disks-hcl/browser-fqdn.png)

## <a name="add-an-ssh-jumpbox"></a>新增 SSH Jumpbox
SSH jumpbox 是您在存取網路上其他伺服器時所「跳躍」通過的單一伺服器。 在此步驟中，您要設定下列資源：

- 連線到與虛擬機器擴展集相同子網路的網路介面 (或 jumpbox)。

- 使用此網路介面連線的虛擬機器。 此 'jumpbox' 可從遠端存取。 一旦連線之後，您就可以透過 SSH 連線到擴展集中的任何虛擬機器。

1. 開啟 `vmss.tf` 組態檔。

  ```bash
  vi vmss.tf
  ```

1. 請移至檔案的結尾，並選取 A 鍵來進入附加模式。

1. 將下列程式碼貼到檔案的結尾：

  ```JSON
  resource "azurerm_public_ip" "jumpbox" {
    name                         = "jumpbox-public-ip"
    location                     = "${var.location}"
    resource_group_name          = "${azurerm_resource_group.vmss.name}"
    public_ip_address_allocation = "static"
    domain_name_label            = "${random_string.fqdn.result}-ssh"
    tags                         = "${var.tags}"
  }

  resource "azurerm_network_interface" "jumpbox" {
    name                = "jumpbox-nic"
    location            = "${var.location}"
    resource_group_name = "${azurerm_resource_group.vmss.name}"

    ip_configuration {
      name                          = "IPConfiguration"
      subnet_id                     = "${azurerm_subnet.vmss.id}"
      private_ip_address_allocation = "dynamic"
      public_ip_address_id          = "${azurerm_public_ip.jumpbox.id}"
    }

    tags = "${var.tags}"
  }

  resource "azurerm_virtual_machine" "jumpbox" {
    name                  = "jumpbox"
    location              = "${var.location}"
    resource_group_name   = "${azurerm_resource_group.vmss.name}"
    network_interface_ids = ["${azurerm_network_interface.jumpbox.id}"]
    vm_size               = "Standard_DS1_v2"

    storage_image_reference {
      publisher = "Canonical"
      offer     = "UbuntuServer"
      sku       = "16.04-LTS"
      version   = "latest"
    }

    storage_os_disk {
      name              = "jumpbox-osdisk"
      caching           = "ReadWrite"
      create_option     = "FromImage"
      managed_disk_type = "Standard_LRS"
    }

    os_profile {
      computer_name  = "jumpbox"
      admin_username = "${var.admin_user}"
      admin_password = "${var.admin_password}"
    }

    os_profile_linux_config {
      disable_password_authentication = false
    }

    tags = "${var.tags}"
  }
  ```

1. 開啟 `output.tf` 組態檔。

  ```bash
  vi output.tf
  ```

1. 請移至檔案的結尾，並選取 A 鍵來進入附加模式。

1. 將下列程式碼貼到檔案的結尾，以在部署完成時顯示 Jumpbox 的主機名稱：

  ```
  output "jumpbox_public_ip" {
      value = "${azurerm_public_ip.jumpbox.fqdn}"
  }
  ```

1. 選取 Esc 鍵結束插入模式。

1. 輸入下列命令來儲存檔案及結束 vi 編輯器：

    ```bash
    :wq
    ```

1. 部署 Jumpbox。

  ```bash
  terraform apply 
  ```

部署完成之後，資源群組的內容會類似於下列螢幕擷取畫面所示：

![Terraform 虛擬機器擴展集資源群組](./media/terraform-create-vm-scaleset-network-disks-hcl/resource-group-contents-final.png)

> [!NOTE]
> 在 Jumpbox 和您部署的虛擬機器擴展集上，已停用以密碼登入的功能。 請使用 SSH 登入來存取虛擬機器。

## <a name="environment-cleanup"></a>環境清除 

若要刪除本教學課程中所建立的 Terraform 資源，請將下列命令輸入 Cloud Shell 中：

```bash
terraform destroy
```

解構程序可能需要幾分鐘的時間才能完成。

## <a name="next-steps"></a>後續步驟
在本文中，您會了解如何使用 Terraform 來建立 Azure 虛擬機器擴展集。 以下有一些額外的資源，可協助您深入了解 Azure 上的 Terraform： 

 [Microsoft.com 中的 Terraform 中樞](https://docs.microsoft.com/azure/terraform/)  
 [Terraform Azure 提供者文件](http://aka.ms/terraform)  
 [Terraform Azure 提供者來源](http://aka.ms/tfgit)  
 [Terraform Azure 模組](http://aka.ms/tfmodules)