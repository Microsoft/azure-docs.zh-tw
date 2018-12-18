---
title: 使用 Ansible 在 Azure 中管理 Linux 虛擬機器
description: 了解如何使用 Ansible 在 Azure 中管理 Linux 虛擬機器
ms.service: ansible
keywords: ansible、azure、devops、bash、cloudshell、腳本、bash
author: tomarcher
manager: jeconnoc
ms.author: tarcher
ms.topic: quickstart
ms.date: 08/22/2018
ms.openlocfilehash: 79b10a30eea9e19f7ec21f9f9b7ebb95b4c34bf2
ms.sourcegitcommit: 58c5cd866ade5aac4354ea1fe8705cee2b50ba9f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42813380"
---
# <a name="use-ansible-to-manage-a-linux-virtual-machine-in-azure"></a>使用 Ansible 在 Azure 中管理 Linux 虛擬機器
Ansible 可讓您將環境中的資源部署和設定自動化。 您可以使用 Ansible 管理 Azure 虛擬機器，如同管理任何其他資源一樣。 本文將說明如何使用 Ansible 腳本來啟動和停止 Linux 虛擬機器。 

## <a name="prerequisites"></a>必要條件

- **Azure 訂用帳戶** - 如果您沒有 Azure 訂用帳戶，請建立[免費帳戶](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio)。

- [!INCLUDE [ansible-prereqs-for-cloudshell-use-or-vm-creation1.md](../../../includes/ansible-prereqs-for-cloudshell-use-or-vm-creation1.md)] [!INCLUDE [ansible-prereqs-for-cloudshell-use-or-vm-creation2.md](../../../includes/ansible-prereqs-for-cloudshell-use-or-vm-creation2.md)]

## <a name="use-ansible-to-deallocate-stop-an-azure-virtual-machine"></a>使用 Ansible 來解除配置 (停止) Azure 虛擬機器
本節說明如何使用 Ansible 來解除配置 (停止) Azure 虛擬機器

1. 登入 [Azure 入口網站](http://go.microsoft.com/fwlink/p/?LinkID=525040)。

1. 開啟 [Cloud Shell](/azure/cloud-shell/overview)。

1. 建立名為 `azure_vm_stop.yml` 的檔案 (以包含您腳本)，然後在 VI 編輯器中加以開啟，如下所示：

  ```azurecli-interactive
  vi azure_vm_stop.yml
  ```

1. 選取 **I** 鍵輸入插入模式。

1. 將下列範例程式碼貼到編輯器中：

    ```yaml
    - name: Stop Azure VM
    hosts: localhost
    connection: local
    tasks:
    - name: Deallocate the virtual machine
        azure_rm_virtualmachine:
            resource_group: myResourceGroup
            name: myVM
            allocated: no 
    ```

1. 選取 **Esc** 鍵結束插入模式。

1. 輸入下列命令來儲存檔案及結束 vi 編輯器：

    ```bash
    :wq
    ```

1. 執行範例 Ansible 腳本。

  ```bash
  ansible-playbook azure_vm_stop.yml
  ```

1. 輸出看起來會類似於下列範例，顯示已成功解除配置 (停止) 虛擬機器：

    ```bash
    PLAY [Stop Azure VM] ********************************************************

    TASK [Gathering Facts] ******************************************************
    ok: [localhost]

    TASK [Deallocate the Virtual Machine] ***************************************
    changed: [localhost]

    PLAY RECAP ******************************************************************
    localhost                  : ok=2    changed=1    unreachable=0    failed=0
    ```

## <a name="use-ansible-to-start-a-deallocated-stopped-azure-virtual-machine"></a>使用 Ansible 來啟動已解除配置 (停止) 的 Azure 虛擬機器
本節說明如何使用 Ansible 來啟動已解除配置 (停止) 的 Azure 虛擬機器

1. 登入 [Azure 入口網站](http://go.microsoft.com/fwlink/p/?LinkID=525040)。

1. 開啟 [Cloud Shell](/azure/cloud-shell/overview)。

1. 建立名為 `azure_vm_start.yml` 的檔案 (以包含您腳本)，然後在 VI 編輯器中加以開啟，如下所示：

  ```azurecli-interactive
  vi azure_vm_start.yml
  ```

1. 選取 **I** 鍵輸入插入模式。

1. 將下列範例程式碼貼到編輯器中：

    ```yaml
    - name: Start Azure VM
    hosts: localhost
    connection: local
    tasks:
    - name: Start the virtual machine
        azure_rm_virtualmachine:
            resource_group: myResourceGroup
            name: myVM
    ```

1. 選取 **Esc** 鍵結束插入模式。

1. 輸入下列命令來儲存檔案及結束 vi 編輯器：

    ```bash
    :wq
    ```

1. 執行範例 Ansible 腳本。

  ```bash
  ansible-playbook azure_vm_start.yml
  ```

1. 輸出看起來會類似於下列範例，顯示已成功啟動虛擬機器：

    輸出看起來會類似於下列範例，顯示已成功啟動虛擬機器：

    ```bash
    PLAY [Stop Azure VM] ********************************************************

    TASK [Gathering Facts] ******************************************************
    ok: [localhost]

    TASK [Start the Virtual Machine] ********************************************
    changed: [localhost]

    PLAY RECAP ******************************************************************
    localhost                  : ok=2    changed=1    unreachable=0    failed=0
    ```

## <a name="next-steps"></a>後續步驟
> [!div class="nextstepaction"] 
> [使用 Ansible 來管理 Azure 動態清查](../../ansible/ansible-manage-azure-dynamic-inventories.md)