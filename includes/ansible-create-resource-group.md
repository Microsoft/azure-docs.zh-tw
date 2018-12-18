---
author: tomarcher
ms.service: ansible
ms.topic: include
ms.date: 08/09/2018
ms.author: tarcher
ms.openlocfilehash: fa1f7fe0b4b70aae4f9165197d5d1463df1f2e3b
ms.sourcegitcommit: 1af4bceb45a0b4edcdb1079fc279f9f2f448140b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/09/2018
ms.locfileid: "40026714"
---
1. 在 Cloud Shell 中建立名稱為 `rg.yml` 的檔案。

    ```bash
    vi rg.yml
    ```

1. 選取 **I** 鍵輸入插入模式。

1. 將下列程式碼貼到編輯器中：

   ```yaml
   ---
   - hosts: localhost
     connection: local
     tasks:
       - name: Create resource group
         azure_rm_resourcegroup:
           name: ansible-rg
           location: eastus
         register: rg
       - debug:
           var: rg
   ```

1. 選取 **Esc** 鍵結束插入模式。

1. 輸入下列命令來儲存檔案及結束 vi 編輯器：

    ```bash
    :wq
    ```

1. 執行 `rg.yml` 腳本：

   ```bash
   ansible-playbook rg.yml
   ```

執行 Ansible 命令的結果應類似下列輸出：

```output
PLAY [localhost] *********************************************************************************

TASK [Gathering Facts] ***************************************************************************
ok: [localhost]

TASK [Create resource group] *********************************************************************
changed: [localhost]

TASK [debug] *************************************************************************************
ok: [localhost] => {
    "rg": {
        "changed": true,
        "contains_resources": false,
        "failed": false,
        "state": {
            "id": "/subscriptions/XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX/resourceGroups/ansible-rg",
            "location": "eastus",
            "name": "ansible-rg",
            "provisioning_state": "Succeeded",
            "tags": null
        }
    }
}

PLAY RECAP ***************************************************************************************
localhost                  : ok=3    changed=1    unreachable=0    failed=0
```