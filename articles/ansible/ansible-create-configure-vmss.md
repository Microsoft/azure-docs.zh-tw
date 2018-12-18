---
title: 使用 Ansible 在 Azure 中建立虛擬機器擴展集
description: 了解如何使用 Ansible 在 Azure 中建立及設定虛擬機器擴展集
ms.service: ansible
keywords: ansible, azure, devops, bash, 腳本, 虛擬機器, 虛擬機器擴展集, vmss
author: tomarcher
manager: jeconnoc
ms.author: tarcher
ms.topic: tutorial
ms.date: 08/24/2018
ms.openlocfilehash: f3b08c41d3bf083c7cca5897cee11a1a4b9c9092
ms.sourcegitcommit: ebb460ed4f1331feb56052ea84509c2d5e9bd65c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42918570"
---
# <a name="create-virtual-machine-scale-sets-in-azure-using-ansible"></a>使用 Ansible 在 Azure 中建立虛擬機器擴展集
Ansible 可讓您將環境中的資源部署和設定自動化。 您可以使用 Ansible 在 Azure 中管理虛擬機器擴展集 (VMSS)，就像管理任何其他 Azure 資源一樣。 本文說明如何使用 Ansible 來建立及相應放大虛擬機器擴展集。 

## <a name="prerequisites"></a>必要條件
- **Azure 訂用帳戶** - 如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio)。
- [!INCLUDE [ansible-prereqs-for-cloudshell-use-or-vm-creation1.md](../../includes/ansible-prereqs-for-cloudshell-use-or-vm-creation1.md)] [!INCLUDE [ansible-prereqs-for-cloudshell-use-or-vm-creation2.md](../../includes/ansible-prereqs-for-cloudshell-use-or-vm-creation2.md)]

> [!Note]
> 必須使用 Ansible 2.6，才能執行本教學課程中的下列範例腳本。 

## <a name="create-a-vmss"></a>建立 VMSS
本節提供會定義下列資源的範例 Ansible 腳本：
- 將會部署您所有資源的**資源群組**
- 10.0.0.0/16 位址空間中的**虛擬網路**
- 虛擬網路內的**子網路**
- 允許您透過網際網路存取資源的**公用 IP 位址**
- 可對進出於虛擬機器擴展集的網路流量進行控制的**網路安全性群組**
- 會使用負載平衡器規則將流量分散於一組已定義 VM 的**負載平衡器**
- 使用所有已建立資源的**虛擬機器擴展集**

輸入您自己的密碼作為 *admin_password* 值。

  ```yaml
  - hosts: localhost
    vars:
      resource_group: myResourceGroup
      vmss_name: myVMSS
      vmss_lb_name: myVMSSlb
      location: eastus
      admin_username: azureuser
      admin_password: "your_password"
    tasks:
      - name: Create a resource group
        azure_rm_resourcegroup:
          name: "{{ resource_group }}"
          location: "{{ location }}"
      - name: Create virtual network
        azure_rm_virtualnetwork:
          resource_group: "{{ resource_group }}"
          name: "{{ vmss_name }}"
          address_prefixes: "10.0.0.0/16"
      - name: Add subnet
        azure_rm_subnet:
          resource_group: "{{ resource_group }}"
          name: "{{ vmss_name }}"
          address_prefix: "10.0.1.0/24"
          virtual_network: "{{ vmss_name }}"
      - name: Create public IP address
        azure_rm_publicipaddress:
          resource_group: "{{ resource_group }}"
          allocation_method: Static
          name: "{{ vmss_name }}"
      - name: Create Network Security Group that allows SSH
        azure_rm_securitygroup:
          resource_group: "{{ resource_group }}"
          name: "{{ vmss_name }}"
          rules:
            - name: SSH
              protocol: Tcp
              destination_port_range: 22
              access: Allow
              priority: 1001
              direction: Inbound

      - name: Create a load balancer
        azure_rm_loadbalancer:
          name: "{{ vmss_lb_name }}"
          location: "{{ location }}"
          resource_group: "{{ resource_group }}"
          public_ip: "{{ vmss_name }}"
          probe_protocol: Tcp
          probe_port: 8080
          probe_interval: 10
          probe_fail_count: 3
          protocol: Tcp
          load_distribution: Default
          frontend_port: 80
          backend_port: 8080
          idle_timeout: 4
          natpool_frontend_port_start: 50000
          natpool_frontend_port_end: 50040
          natpool_backend_port: 22
          natpool_protocol: Tcp

      - name: Create VMSS
        azure_rm_virtualmachine_scaleset:
          resource_group: "{{ resource_group }}"
          name: "{{ vmss_name }}"
          vm_size: Standard_DS1_v2
          admin_username: "{{ admin_username }}"
          admin_password: "{{ admin_password }}"
          ssh_password_enabled: true
          capacity: 2
          virtual_network_name: "{{ vmss_name }}"
          subnet_name: "{{ vmss_name }}"
          upgrade_policy: Manual
          tier: Standard
          managed_disk_type: Standard_LRS
          os_disk_caching: ReadWrite
          image:
            offer: UbuntuServer
            publisher: Canonical
            sku: 16.04-LTS
            version: latest
          load_balancer: "{{ vmss_lb_name }}"
          data_disks:
            - lun: 0
              disk_size_gb: 20
              managed_disk_type: Standard_LRS
              caching: ReadOnly
            - lun: 1
              disk_size_gb: 30
              managed_disk_type: Standard_LRS
              caching: ReadOnly
  ```

若要使用 Ansible 建立虛擬機器擴展集環境，請將上述腳本儲存為 `vmss-create.yml`，或[下載範例 Ansible 腳本](https://github.com/Azure-Samples/ansible-playbooks/blob/master/vmss/vmss-create.yml)。

若要執行 Ansible 腳本，請使用 **ansible-playbook** 命令，如下所示：

  ```bash
  ansible-playbook vmss-create.yml
  ```

執行腳本後，類似於下列範例的輸出會顯示虛擬機器擴展集已成功建立：

  ```bash
  PLAY [localhost] ***********************************************************

  TASK [Gathering Facts] *****************************************************
  ok: [localhost]

  TASK [Create a resource group] ****************************************************************************
  changed: [localhost]

  TASK [Create virtual network] ****************************************************************************
  changed: [localhost]

  TASK [Add subnet] **********************************************************
  changed: [localhost]

  TASK [Create public IP address] ****************************************************************************
  changed: [localhost]

  TASK [Create Network Security Group that allows SSH] ****************************************************************************
  changed: [localhost]

  TASK [Create a load balancer] ****************************************************************************
  changed: [localhost]

  TASK [Create VMSS] *********************************************************
  changed: [localhost]

  PLAY RECAP *****************************************************************
  localhost                  : ok=8    changed=7    unreachable=0    failed=0

  ```

## <a name="scale-out-a-vmss"></a>相應放大 VMSS
建立的虛擬機器擴展集有兩個執行個體。 如果您瀏覽至 Azure 入口網站中的虛擬機器擴展集，您會看到 **Standard_DS1_v2 (2 個執行個體)**。 您也可以使用 [Azure Cloud Shell](https://shell.azure.com/)，在 Cloud Shell 中執行下列命令：

  ```azurecli-interactive
  az vmss show -n myVMSS -g myResourceGroup --query '{"capacity":sku.capacity}' 
  ```

您會看到類似下列輸出的結果：

  ```bash
  {
    "capacity": 2,
  }
  ```

現在，我們要從兩個執行個體擴充為三個執行個體。 下列 Ansible 腳本程式碼會擷取虛擬機器調整的相關資訊，並將其容量從兩個變更為三個。 

  ```yaml
  - hosts: localhost
    vars:
      resource_group: myResourceGroup
      vmss_name: myVMSS
    tasks: 
      - name: Get scaleset info
        azure_rm_virtualmachine_scaleset_facts:
          resource_group: "{{ resource_group }}"
          name: "{{ vmss_name }}"
          format: curated
        register: output_scaleset

      - name: Dump scaleset info
        debug:
          var: output_scaleset

      - name: Modify scaleset (change the capacity to 3)
        set_fact:
          body: "{{ output_scaleset.ansible_facts.azure_vmss[0] | combine({'capacity': 3}, recursive=True) }}"

      - name: Update something in that VMSS
        azure_rm_virtualmachine_scaleset: "{{ body }}"
  ```

若要將您已建立的虛擬機器擴展集相應放大，請將上述腳本儲存為 `vmss-scale-out.yml`，或[下載範例 Ansible 腳本](https://github.com/Azure-Samples/ansible-playbooks/blob/master/vmss/vmss-scale-out.yml)。 

下列命令將執行腳本：

  ```bash
  ansible-playbook vmss-scale-out.yml
  ```

執行 Ansible 腳本後的輸出會顯示虛擬機器擴展集已成功相應放大：

  ```bash
  PLAY [localhost] **********************************************************

  TASK [Gathering Facts] ****************************************************
  ok: [localhost]

  TASK [Get scaleset info] ***************************************************************************
  ok: [localhost]

  TASK [Dump scaleset info] ***************************************************************************
  ok: [localhost] => {
      "output_scaleset": {
          "ansible_facts": {
              "azure_vmss": [
                  {
                      ......
                  }
              ]
          },
          "changed": false,
          "failed": false
      }
  }

  TASK [Modify scaleset (set upgradePolicy to Automatic and capacity to 3)] ***************************************************************************
  ok: [localhost]

  TASK [Update something in that VMSS] ***************************************************************************
  changed: [localhost]

  PLAY RECAP ****************************************************************
  localhost                  : ok=5    changed=1    unreachable=0    failed=0
  ```

如果瀏覽至您在 Azure 入口網站中設定的虛擬機器擴展集，您會看到 **Standard_DS1_v2 (3 個執行個體)**。 您也可以執行下列命令，以使用 [Azure Cloud Shell](https://shell.azure.com/) 確認變更：

  ```azurecli-interactive
  az vmss show -n myVMSS -g myResourceGroup --query '{"capacity":sku.capacity}' 
  ```

在 Cloud Shell 中執行命令的結果，會顯示目前有三個執行個體存在。 

  ```bash
  {
    "capacity": 3,
  }
  ```

## <a name="next-steps"></a>後續步驟
> [!div class="nextstepaction"] 
> [VMSS 的 Ansible 範例腳本](https://github.com/Azure-Samples/ansible-playbooks/tree/master/vmss)