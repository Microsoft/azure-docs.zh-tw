---
title: 使用 Ansible 將應用程式部署至 Azure 中的虛擬機器擴展集
description: 了解如何使用 Ansible 來設定虛擬機器擴展集，並在 Azure 中的虛擬機器擴展集上部署應用程式
ms.service: ansible
keywords: ansible, azure, devops, bash, 腳本, 虛擬機器, 虛擬機器擴展集, vmss
author: tomarcher
manager: jeconnoc
ms.author: tarcher
ms.topic: tutorial
ms.date: 09/11/2018
ms.openlocfilehash: 3512cb7eda9f9e5a6e18dc83f6523029b17a9de2
ms.sourcegitcommit: 794bfae2ae34263772d1f214a5a62ac29dcec3d2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44391465"
---
# <a name="deploy-applications-to-virtual-machine-scale-sets-in-azure-using-ansible"></a>使用 Ansible 將應用程式部署至 Azure 中的虛擬機器擴展集
Ansible 可讓您將環境中的資源部署和設定自動化。 您可以使用 Ansible 將應用程式部署至 Azure。 本文說明如何將 Java 應用程式部署至 Azure 虛擬機器擴展集 (VMSS)。  

## <a name="prerequisites"></a>必要條件
- **Azure 訂用帳戶** - 如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio)。
- [!INCLUDE [ansible-prereqs-for-cloudshell-use-or-vm-creation1.md](../../includes/ansible-prereqs-for-cloudshell-use-or-vm-creation1.md)] [!INCLUDE [ansible-prereqs-for-cloudshell-use-or-vm-creation2.md](../../includes/ansible-prereqs-for-cloudshell-use-or-vm-creation2.md)]
- **虛擬機器擴展集** - 如果您還沒有虛擬機器擴展集，您可以[使用 Ansible 建立虛擬機器擴展集](ansible-create-configure-vmss.md)。 
- **git** - [git](https://git-scm.com) 可用來下載本教學課程中使用的 Java 範例。
- **Java SE 開發套件 (JDK)** - JDK 可用來建置範例 Java 專案。
- **Apache Maven 建置工具** - [Apache Maven 建置工具](https://maven.apache.org/download.cgi)可用來建置範例 Java 專案。

> [!Note]
> 必須使用 Ansible 2.6，才能執行本教學課程中的下列範例腳本。 

## <a name="get-host-information"></a>取得主機資訊

本節說明如何使用 Ansible 擷取 Azure 虛擬機器群組的主機資訊。 以下是範例 Ansible 腳本。 下列程式碼會取得資源群組內指定的公用 IP 位址和負載平衡器，並在清查中建立名為 **scalesethosts** 的主機群組。 

請下列範例腳本儲存為 `get-hosts-tasks.yml`： 

  ```yaml
  - name: Get facts for all Public IPs within a resource groups
    azure_rm_publicipaddress_facts:
      resource_group: "{{ resource_group }}"
    register: output_ip_address

  - name: Get loadbalancer info
    azure_rm_loadbalancer_facts:
      resource_group: "{{ resource_group }}"
      name: "{{ loadbalancer_name }}"
    register: output

  - name: Add all hosts
    add_host:
      groups: scalesethosts
      hostname: "{{ output_ip_address.ansible_facts.azure_publicipaddresses[0].properties.ipAddress }}_{{ item.properties.frontendPort }}"
      ansible_host: "{{ output_ip_address.ansible_facts.azure_publicipaddresses[0].properties.ipAddress }}"
      ansible_port: "{{ item.properties.frontendPort }}"
      ansible_ssh_user: "{{ admin_username }}"
      ansible_ssh_pass: "{{ admin_password }}"
    with_items:
      - "{{ output.ansible_facts.azure_loadbalancers[0].properties.inboundNatRules }}"
  ```

## <a name="prepare-an-application-for-deployment"></a>準備應用程式以進行開發  

在本節中，您會使用 git 從 GitHub 複製 Java 範例專案，然後建置專案。 請下列腳本儲存為 `app.yml`：

  ```yaml
  - hosts: localhost
    vars:
      repo_url: https://github.com/spring-guides/gs-spring-boot.git
      workspace: ~/src/helloworld

    tasks: 
    - name: Git Clone sample app
      git:
        repo: "{{ repo_url }}"
        dest: "{{ workspace }}"

    - name: Build sample app
      shell: mvn package chdir="{{ workspace }}/complete"
  ```

使用下列命令執行範例 Ansible 腳本：

  ```bash
  ansible-playbook app.yml
  ```

ansible-playbook 命令的輸出會顯示如下，在其中您可以看到，此命令建置了從 GitHub 複製的範例應用程式：

  ```bash
  PLAY [localhost] **********************************************************

  TASK [Gathering Facts] ****************************************************
  ok: [localhost]

  TASK [Git Clone sample app] ***************************************************************************
  changed: [localhost]

  TASK [Build sample app] ***************************************************
  changed: [localhost]

  PLAY RECAP ***************************************************************************
  localhost                  : ok=3    changed=2    unreachable=0    failed=0

  ```

## <a name="deploy-the-application-to-vmss"></a>將應用程式部署至 VMSS

Ansible 腳本中的下一節會在名為 **saclesethosts** 的主機群組上安裝 JRE (Java Runtime Environment)，並 Java 應用程式部署至名為 **saclesethosts** 的主機群組： 

(請將 `admin_password` 變更為您自己的密碼。)

  ```yaml
  - hosts: localhost
    vars:
      resource_group: myResourceGroup
      scaleset_name: myVMSS
      loadbalancer_name: myVMSSlb
      admin_username: azureuser
      admin_password: "your_password"
    tasks:   
    - include: get-hosts-tasks.yml

  - name: Install JRE on VMSS
    hosts: scalesethosts
    become: yes
    vars:
      workspace: ~/src/helloworld
      admin_username: azureuser

    tasks:
    - name: Install JRE
      apt:
        name: default-jre
        update_cache: yes

    - name: Copy app to Azure VM
      copy:
        src: "{{ workspace }}/complete/target/gs-spring-boot-0.1.0.jar"
        dest: "/home/{{ admin_username }}/helloworld.jar"
        force: yes
        mode: 0755

    - name: Start the application
      shell: java -jar "/home/{{ admin_username }}/helloworld.jar" >/dev/null 2>&1 &
      async: 5000
      poll: 0
  ```

您可以將上述範例 Ansible 腳本儲存為 `vmss-setup-deploy.yml`，或[下載整個範例腳本](https://github.com/Azure-Samples/ansible-playbooks/blob/master/vmss)。 

若要使用需要密碼的 SSH 連線類型，您必須安裝 sshpass 程式。 
  - 針對 Ubunto 16.04，請執行命令 `apt-get install sshpass`。
  - 針對 CentOS 7.4，請執行命令 `yum install sshpass`。

您可能會看到如下的錯誤：**無法使用 SSH 密碼而不使用金鑰，因為已啟用主機金鑰檢查，但 sshpass 不支援此功能。請將此主機的指紋新增至您的 known_hosts 檔案，以管理此主機。** 如果出現此錯誤，您可以在 `/etc/ansible/ansible.cfg` 檔案或 `~/.ansible.cfg` 檔案中加入以下一行，以停用主機金鑰檢查：
  ```bash
  [defaults]
  host_key_checking = False
  ```

使用下列命令執行腳本：

  ```bash
  ansible-playbook vmss-setup-deploy.yml
  ```

執行 ansible-playbook 命令的輸出會指出範例 Java 應用程式已安裝至虛擬機器擴展集的主機群組：

  ```bash
  PLAY [localhost] **********************************************************

  TASK [Gathering Facts] ****************************************************
  ok: [localhost]

  TASK [Get facts for all Public IPs within a resource groups] **********************************************
  ok: [localhost]

  TASK [Get loadbalancer info] ****************************************************************************
  ok: [localhost]

  TASK [Add all hosts] *****************************************************************************
  changed: [localhost] ...

  PLAY [Install JRE on VMSS] *****************************************************************************

  TASK [Gathering Facts] *****************************************************************************
  ok: [40.114.30.145_50000]
  ok: [40.114.30.145_50003]

  TASK [Copy app to Azure VM] *****************************************************************************
  changed: [40.114.30.145_50003]
  changed: [40.114.30.145_50000]

  TASK [Start the application] ********************************************************************
  changed: [40.114.30.145_50000]
  changed: [40.114.30.145_50003]

  PLAY RECAP ************************************************************************************************
  40.114.30.145_50000        : ok=4    changed=3    unreachable=0    failed=0
  40.114.30.145_50003        : ok=4    changed=3    unreachable=0    failed=0
  localhost                  : ok=4    changed=1    unreachable=0    failed=0
  ```

恭喜！ 您的應用程式現在已執行於 Azure 中。 現在，您可以瀏覽至虛擬機器擴展集的負載平衡器 URL：

![在 Azure 中的虛擬機器擴展集上執行的 Java 應用程式。](media/ansible-deploy-app-vmss/ansible-deploy-app-vmss.png)

## <a name="next-steps"></a>後續步驟
> [!div class="nextstepaction"] 
> [VMSS 的 Ansible 範例腳本](https://github.com/Azure-Samples/ansible-playbooks/tree/master/vmss)