---
title: 使用 Ansible 來管理 Azure 動態清查
description: 了解如何使用 Ansible 來管理 Azure 動態清查
ms.service: ansible
keywords: ansible, azure, devops, bash, cloudshell, 動態清查
author: tomarcher
manager: jeconnoc
ms.author: tarcher
ms.date: 08/09/2018
ms.topic: article
ms.openlocfilehash: 1b8c1ba80b4c69f36e8304cbe978452a359ac911
ms.sourcegitcommit: cb61439cf0ae2a3f4b07a98da4df258bfb479845
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43698072"
---
# <a name="use-ansible-to-manage-your-azure-dynamic-inventories"></a>使用 Ansible 來管理 Azure 動態清查
您可以使用 Ansible 從各種來源 (包括如 Azure 等雲端來源) 將清查資訊提取至動態清查。 在本文中，您使用 [Azure Cloud Shell](./ansible-run-playbook-in-cloudshell.md) 來設定 Ansible Azure 動態清查，並在其中建立兩部虛擬機器、標記其中一部虛擬機器，然後在已標記的虛擬機器上安裝 Nginx。

## <a name="prerequisites"></a>必要條件

- **Azure 訂用帳戶** - 如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio)。

- **Azure 認證** - [建立 Azure 認證和設定 Ansible](/azure/virtual-machines/linux/ansible-install-configure#create-azure-credentials)

## <a name="create-the-test-virtual-machines"></a>建立測試虛擬機器

1. 登入 [Azure 入口網站](http://go.microsoft.com/fwlink/p/?LinkID=525040)。

1. 開啟 [Cloud Shell](https://docs.microsoft.com/azure/cloud-shell/overview)。

1. 建立 Azure 資源群組來保存此教學課程中的虛擬機器。

    > [!IMPORTANT]  
    > 您在此步驟中建立的 Azure 資源群組必須使用全部都是小寫的名稱。 否則將無法產生動態清查。

    ```azurecli-interactive
    az group create --resource-group ansible-inventory-test-rg --location eastus
    ```

1. 使用下列其中一項技術，在 Azure 上建立兩個 Linux 虛擬機器：

    - **Ansible 腳本** - [使用 Ansible 在 Azure 中建立基本虛擬機器](/azure/virtual-machines/linux/ansible-create-vm)一文中說明如何從 Ansible 腳本建立虛擬機器。 如果您使用腳本來定義一個或兩個虛擬機器，請確定您是使用 SSH 連線而不是使用密碼。

    - **Azure CLI** - 在 Cloud Shell 中發出以下每一個命令，以建立兩個虛擬機器：

        ```azurecli-interactive
        az vm create --resource-group ansible-inventory-test-rg \
                     --name ansible-inventory-test-vm1 \
                     --image UbuntuLTS --generate-ssh-keys
        ```

        ```azurecli-interactive
        az vm create --resource-group ansible-inventory-test-rg \
                     --name ansible-inventory-test-vm2 \
                     --image UbuntuLTS --generate-ssh-keys
        ```

## <a name="tag-a-virtual-machine"></a>標記虛擬機器
您可以依使用者定義的類別[使用標記來組織 Azure 資源](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags#azure-cli)。 

輸入下列 [az resource tag](/cli/azure/resource?view=azure-cli-latest.md#az-resource-tag) 命令來標記具有 `nginx` 金鑰的虛擬機器 `ansible-inventory-test-vm1`：

```azurecli-interactive
az resource tag --tags nginx --id /subscriptions/<YourAzureSubscriptionID>/resourceGroups/ansible-inventory-test-rg/providers/Microsoft.Compute/virtualMachines/ansible-inventory-test-vm1
```

## <a name="generate-a-dynamic-inventory"></a>產生動態清查
一旦您定義虛擬機器 (並標記) 後，就可以產生動態清查。 Ansible 提供名為 [azure_rm.py](https://github.com/ansible/ansible/blob/devel/contrib/inventory/azure_rm.py) 的 Python 指令碼，會向 Azure Resource Manager 提出 API 要求，以產生您 Azure 資源的動態清單。 下列步驟將引導您使用 `azure_rm.py` 指令碼，以連線到您的兩部測試 Azure 虛擬機器：

1. 使用 GNU `wget` 命令來取出 `azure_rm.py` 指令碼：

    ```azurecli-interactive
    wget https://raw.githubusercontent.com/ansible/ansible/devel/contrib/inventory/azure_rm.py
    ```

1. 使用 `chmod` 命令將存取權限變更為 `azure_rm.py` 指令碼。 下列命令會使用 `+x` 參數，以允許執行 (執行) 指定的檔案 (`azure_rm.py`)：

    ```azurecli-interactive
    chmod +x azure_rm.py
    ```

1. 使用 [ansible 命令](https://docs.ansible.com/ansible/2.4/ansible.html)連線到您的資源群組： 

    ```azurecli-interactive
    ansible -i azure_rm.py ansible-inventory-test-rg -m ping 
    ```

1. 一旦連線後，您會看到類似下列輸出的結果：

    ```Output
    ansible-inventory-test-vm1 | SUCCESS => {
        "changed": false,
        "failed": false,
        "ping": "pong"
    }
    ansible-inventory-test-vm2 | SUCCESS => {
        "changed": false,
        "failed": false,
        "ping": "pong"
    }
    ```

## <a name="enable-the-virtual-machine-tag"></a>啟用虛擬機器標記
一旦您設定所需的標記後，必須「啟用」標記。 啟用標記的方式之一，是透過 **export** 命令將標記匯出至名為 `AZURE_TAGS` 的環境變數：

```azurecli-interactive
export AZURE_TAGS=nginx
```

將標記匯出後，您就可以再次嘗試 `ansible` 命令：

```azurecli-interactive
ansible -i azure_rm.py ansible-inventory-test-rg -m ping 
```

您現在只會看到一部虛擬機器 (虛擬機器的標記符合匯出至 **AZURE_TAGS** 環境變數的值)：

```Output
ansible-inventory-test-vm1 | SUCCESS => {
    "changed": false,
    "failed": false,
    "ping": "pong"
}
```

## <a name="set-up-nginx-on-the-tagged-vm"></a>設定已標記 VM 上的 Nginx
標記的目的是要讓您能夠快速且輕鬆地使用您虛擬機器的子群組。 例如，假設您只需要在已指派 `nginx` 標記的虛擬機器上安裝 Nginx。 下列步驟說明可如何輕鬆完成：

1. 建立名為 `nginx.yml` 的檔案 (以包含您腳本)，如下所示：

  ```azurecli-interactive
  vi nginx.yml
  ```

1. 將下列程式碼插入新建的 `nginx.yml` 檔案：

    ```yml
    ---
    - name: Install and start Nginx on an Azure virtual machine
    hosts: azure
    become: yes
    tasks:
    - name: install nginx
      apt: pkg=nginx state=installed
      notify:
      - start nginx

    handlers:
    - name: start nginx
      service: name=nginx state=started
    ```

1. 執行 `nginx.yml` 腳本：

    ```azurecli-interactive
    ansible-playbook -i azure_rm.py nginx.yml
    ```

1. 一旦執行腳本後，您會看到類似下列輸出的結果：

    ```Output
    PLAY [Install and start Nginx on an Azure virtual machine] **********

    TASK [Gathering Facts] **********
    ok: [ansible-inventory-test-vm1]

    TASK [install nginx] **********
    changed: [ansible-inventory-test-vm1]

    RUNNING HANDLER [start nginx] **********
    ok: [ansible-inventory-test-vm1]

    PLAY RECAP **********
    ansible-inventory-test-vm1 : ok=3    changed=1    unreachable=0    failed=0
    ```

## <a name="test-nginx-installation"></a>測試 Nginx 安裝
本節說明的一項技術，可測試 Nginx 已安裝在您的虛擬機器上。

1. 使用 [az vm list-ip-addresses](https://docs.microsoft.com/cli/azure/vm?view=azure-cli-latest#az-vm-list-ip-addresses) 命令來取出 `ansible-inventory-test-vm1` 虛擬機器的 IP 位址。 接著，傳回值 (虛擬機器的 IP 位址) 會用來作為連線到虛擬機器之 SSH 命令的參數。

    ```azurecli-interactive
    ssh `az vm list-ip-addresses \
    -n ansible-inventory-test-vm1 \
    --query [0].virtualMachine.network.publicIpAddresses[0].ipAddress -o tsv`
    ```

1. 在連線至 `ansible-inventory-test-vm1` 虛擬機器後，執行 [nginx-v](https://nginx.org/en/docs/switches.html) 命令以確認 Nginx 是否已安裝。

    ```azurecli-interactive
    nginx -v
    ```

1. 一旦您執行 `nginx -v` 命令後，會看到 Nginx 版本 (第二行)，指出已安裝 Nginx。

    ```Output
    tom@ansible-inventory-test-vm1:~$ nginx -v

    nginx version: nginx/1.10.3 (Ubuntu)
    
    tom@ansible-inventory-test-vm1:~$
    ```

1. 按 **&lt;Ctrl>D** 鍵盤組合，將 SSH 工作階段中斷連線。

1. 執行上述 `ansible-inventory-test-vm2` 虛擬機器的步驟時，會產生參考用訊息，指出您可以取得 Nginx (表示您此時未安裝) 的位置：

    ```Output
    tom@ansible-inventory-test-vm2:~$ nginx -v
    The program 'nginx' can be found in the following packages:
    * nginx-core
    * nginx-extras
    * nginx-full
    * nginx-lightTry: sudo apt install <selected package>
    tom@ansible-inventory-test-vm2:~$
    ```

## <a name="next-steps"></a>後續步驟
> [!div class="nextstepaction"] 
> [使用 Ansible 在 Azure 中建立基本虛擬機器](/azure/virtual-machines/linux/ansible-create-vm)
