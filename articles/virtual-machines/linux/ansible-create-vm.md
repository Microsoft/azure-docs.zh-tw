---
title: 使用 Ansible 在 Azure 中建立 Linux 虛擬機器
description: 了解如何使用 Ansible 在 Azure 中建立 Linux 虛擬機器
ms.service: ansible
keywords: ansible, azure, devops, 虛擬機器
author: tomarcher
manager: jeconnoc
ms.author: tarcher
ms.topic: quickstart
ms.date: 08/22/2018
ms.openlocfilehash: bf2116d7fc143e66309e3bbbdefc50bfa33f1521
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46974556"
---
# <a name="use-ansible-to-create-a-linux-virtual-machine-in-azure"></a>使用 Ansible 在 Azure 中建立 Linux 虛擬機器
使用宣告式語言時，Ansible 可讓您透過 Ansible *腳本*自動建立、設定及部署 Azure 資源。 本文的各節將分別顯示 Ansible 腳本的各個區段在建立及設定 Linux 虛擬機器的不同層面時所呈現的樣貌。 [完整 Ansible 腳本](#complete-sample-ansible-playbook)會列在本文結尾處。

## <a name="prerequisites"></a>必要條件

- **Azure 訂用帳戶** - 如果您沒有 Azure 訂用帳戶，請建立[免費帳戶](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio)。

- [!INCLUDE [ansible-prereqs-for-cloudshell-use-or-vm-creation1.md](../../../includes/ansible-prereqs-for-cloudshell-use-or-vm-creation1.md)]

## <a name="create-a-resource-group"></a>建立資源群組
Ansible 需要用來部署資源的資源群組。 下列範例 Ansible 腳本區段會在 `eastus` 位置中建立名為 `myResourceGroup` 的資源群組：

```yaml
- name: Create resource group
    azure_rm_resourcegroup:
      name: myResourceGroup
      location: eastus
```

## <a name="create-a-virtual-network"></a>建立虛擬網路
當您建立 Azure 虛擬機器時，您必須建立[虛擬網路](/azure/virtual-network/virtual-networks-overview)或使用現有的虛擬網路。 您也需要決定如何在虛擬網路上存取您的虛擬機器。 下列範例 Ansible 腳本區段會在 `10.0.0.0/16` 位址空間中建立名為 `myVnet` 的虛擬網路：

```yaml
- name: Create virtual network
  azure_rm_virtualnetwork:
    resource_group: myResourceGroup
    name: myVnet
    address_prefixes: "10.0.0.0/16"
```

所有部署到虛擬網路的 Azure 資源都會部署到虛擬網路內的[子網路](/azure/virtual-network/virtual-network-manage-subnet)。 

下列範例 Ansible 腳本區段會在 `myVnet` 虛擬網路中建立名為 `mySubnet` 的子網路：

```yaml
- name: Add subnet
  azure_rm_subnet:
    resource_group: myResourceGroup
    name: mySubnet
    address_prefix: "10.0.1.0/24"
    virtual_network: myVnet
```

## <a name="create-a-public-ip-address"></a>建立公用 IP 位址
[公用 IP 位址](/azure/virtual-network/virtual-network-ip-addresses-overview-arm)可讓網際網路資源向 Azure 資源進行輸入通訊。 公用 IP 位址也可透過指派給資源的 IP 位址，讓 Azure 資源向網際網路和公眾對應的 Azure 服務進行輸出通訊。 位址是資源專用，直到您取消指派為止。 如果未將公用 IP 位址指派給資源，資源仍可向網際網路進行輸出通訊，但 Azure 會以動態方式指派非資源專用的可用 IP 位址。 

下列範例 Ansible 腳本區段會建立名為 `myPublicIP` 的公用 IP 位址：

```yaml
- name: Create public IP address
  azure_rm_publicipaddress:
    resource_group: myResourceGroup
    allocation_method: Static
    name: myPublicIP
```

## <a name="create-a-network-security-group"></a>建立網路安全性群組
[網路安全性群組](/azure/virtual-network/security-overview)可讓您篩選在 Azure 虛擬網路中進出於 Azure 資源的網路流量。 網路安全性群組包含安全性規則，可允許或拒絕進出於多種 Azure 資源類型的輸入和輸出網路流量。 

下列範例 Ansible 腳本區段會建立名為 `myNetworkSecurityGroup` 的網路安全性群組，並定義規則以允許 TCP 通訊埠 22 上的 SSH 流量：

```yaml
- name: Create Network Security Group that allows SSH
  azure_rm_securitygroup:
    resource_group: myResourceGroup
    name: myNetworkSecurityGroup
    rules:
      - name: SSH
        protocol: Tcp
        destination_port_range: 22
        access: Allow
        priority: 1001
        direction: Inbound
```


## <a name="create-a-virtual-network-interface-card"></a>建立虛擬網路介面卡
虛擬網路介面卡會將您的虛擬機器連線至指定的虛擬網路、公用 IP 位址和網路安全性群組。 

範例 Ansible 腳本的下列區段會建立名為 `myNIC` 的虛擬網路介面卡，並連線至您所建立的虛擬網路資源：

```yaml
- name: Create virtual network inteface card
  azure_rm_networkinterface:
    resource_group: myResourceGroup
    name: myNIC
    virtual_network: myVnet
    subnet: mySubnet
    public_ip_name: myPublicIP
    security_group: myNetworkSecurityGroup
```

## <a name="create-a-virtual-machine"></a>建立虛擬機器
最後一個步驟是建立虛擬機器，並使其使用您在本文的前面幾節中建立的所有資源。 

本節介紹的範例 Ansible 腳本區段會建立名為 `myVM` 的虛擬機器，並連接名為 `myNIC` 的虛擬網路介面卡。 請將 &lt;your-key-data> 預留位置取代為您自己的完整公開金鑰資料。

```yaml
- name: Create VM
  azure_rm_virtualmachine:
    resource_group: myResourceGroup
    name: myVM
    vm_size: Standard_DS1_v2
    admin_username: azureuser
    ssh_password_enabled: false
    ssh_public_keys:
      - path: /home/azureuser/.ssh/authorized_keys
        key_data: <your-key-data>
    network_interfaces: myNIC
    image:
      offer: CentOS
      publisher: OpenLogic
      sku: '7.5'
      version: latest
```

## <a name="complete-sample-ansible-playbook"></a>完整範例 Ansible 腳本

此節列出您在本文的步驟中建置的整個範例 Ansible 腳本。 

```yaml
- name: Create Azure VM
  hosts: localhost
  connection: local
  tasks:
  - name: Create resource group
    azure_rm_resourcegroup:
      name: myResourceGroup
      location: eastus
  - name: Create virtual network
    azure_rm_virtualnetwork:
      resource_group: myResourceGroup
      name: myVnet
      address_prefixes: "10.0.0.0/16"
  - name: Add subnet
    azure_rm_subnet:
      resource_group: myResourceGroup
      name: mySubnet
      address_prefix: "10.0.1.0/24"
      virtual_network: myVnet
  - name: Create public IP address
    azure_rm_publicipaddress:
      resource_group: myResourceGroup
      allocation_method: Static
      name: myPublicIP
    register: output_ip_address
  - name: Dump public IP for VM which will be created
    debug:
      msg: "The public IP is {{ output_ip_address.state.ip_address }}."
  - name: Create Network Security Group that allows SSH
    azure_rm_securitygroup:
      resource_group: myResourceGroup
      name: myNetworkSecurityGroup
      rules:
        - name: SSH
          protocol: Tcp
          destination_port_range: 22
          access: Allow
          priority: 1001
          direction: Inbound
  - name: Create virtual network inteface card
    azure_rm_networkinterface:
      resource_group: myResourceGroup
      name: myNIC
      virtual_network: myVnet
      subnet: mySubnet
      public_ip_name: myPublicIP
      security_group: myNetworkSecurityGroup
  - name: Create VM
    azure_rm_virtualmachine:
      resource_group: myResourceGroup
      name: myVM
      vm_size: Standard_DS1_v2
      admin_username: azureuser
      ssh_password_enabled: false
      ssh_public_keys:
        - path: /home/azureuser/.ssh/authorized_keys
          key_data: <your-key-data>
      network_interfaces: myNIC
      image:
        offer: CentOS
        publisher: OpenLogic
        sku: '7.5'
        version: latest
```

## <a name="run-the-sample-ansible-playbook"></a>執行範例 Ansible 腳本

本節會引導您執行本文中介紹的範例 Ansible 腳本。

1. 登入 [Azure 入口網站](http://go.microsoft.com/fwlink/p/?LinkID=525040)。

1. 開啟 [Cloud Shell](/azure/cloud-shell/overview)。

1. 建立名為 `azure_create_complete_vm.yml` 的檔案 (以包含您腳本)，然後在 VI 編輯器中加以開啟，如下所示：

  ```azurecli-interactive
  vi azure_create_complete_vm.yml
  ```

1. 選取 **I** 鍵輸入插入模式。

1. 將[完整範例 Ansible 腳本](#complete-sample-ansible-playbook)貼到編輯器中。

1. 選取 **Esc** 鍵結束插入模式。

1. 輸入下列命令來儲存檔案及結束 vi 編輯器：

    ```bash
    :wq
    ```

1. 執行範例 Ansible 腳本。

  ```bash
  ansible-playbook azure_create_complete_vm.yml
  ```

1. 輸出看起來會類似於下列內容，在其中您可以看到已成功建立虛擬機器：

  ```bash
  PLAY [Create Azure VM] ****************************************************

  TASK [Gathering Facts] ****************************************************
  ok: [localhost]

  TASK [Create resource group] *********************************************
  changed: [localhost]

  TASK [Create virtual network] *********************************************
  changed: [localhost]

  TASK [Add subnet] *********************************************************
  changed: [localhost]

  TASK [Create public IP address] *******************************************
  changed: [localhost]

  TASK [Dump public IP for VM which will be created] ********************************************************************
  ok: [localhost] => {
      "msg": "The public IP is <ip-address>."
  }

  TASK [Create Network Security Group that allows SSH] **********************
  changed: [localhost]

  TASK [Create virtual network inteface card] *******************************
  changed: [localhost]

  TASK [Create VM] **********************************************************
  changed: [localhost]

  PLAY RECAP ****************************************************************
  localhost                  : ok=8    changed=7    unreachable=0    failed=0
  ```

1. SSH 命令可用來存取您的 Linux VM。 請將 &lt;ip-address> 預留位置取代為先前步驟中的 IP 位址。

  ```bash
  ssh azureuser@<ip-address>
  ```

## <a name="next-steps"></a>後續步驟
> [!div class="nextstepaction"] 
> [使用 Ansible 在 Azure 中管理 Linux 虛擬機器](./ansible-manage-linux-vm.md)
