---
title: 在 Azure Stack 中使用 API 版本設定檔與 Python | Microsoft Docs
description: 了解如何在 Azure Stack 中使用 API 版本設定檔與 Python。
services: azure-stack
documentationcenter: ''
author: sethmanheim
manager: femila
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/15/2018
ms.author: sethm
ms.reviewer: sijuman
<!-- dev: viananth -->
ms.openlocfilehash: c55dcf0736642690f245f680db5cb1620c2175e7
ms.sourcegitcommit: d2f2356d8fe7845860b6cf6b6545f2a5036a3dd6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/16/2018
ms.locfileid: "41946603"
---
# <a name="use-api-version-profiles-with-python-in-azure-stack"></a>在 Azure Stack 中使用 API 版本設定檔與 Python

*適用於：Azure Stack 整合系統和 Azure Stack 開發套件*

## <a name="python-and-api-version-profiles"></a>Python 和 API 版本設定檔

Python SDK 支援 API 版本設定檔以不同雲端平台 (例如，Azure Stack 和全域 Azure) 作為目標。 您可以使用 API 設定檔來建立混合式雲端的解決方案。 Python SDK 支援下列 API 設定檔：

1. **最新**  
    設定檔會以 Azure 平台中所有服務提供者的最新 API 版本作為目標。
2.  **2017-03-09-profile**  
    **2017-03-09-profile**  
    設定檔會以 Azure Stack 所支援的資源提供者 API 版本 作為目標。

    如需 API 設定檔和 Azure Stack 的詳細資訊，請參閱[管理 Azure Stack 中的 API 版本設定檔](azure-stack-version-profiles.md)。

## <a name="install-azure-python-sdk"></a>安裝 Azure Python SDK

1.  從[官方網站](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)安裝 Git。
2.  如需 Python SDK 的安裝指示，請參閱[適用於 Python 開發人員的 Azure](https://docs.microsoft.com/python/azure/python-sdk-azure-install?view=azure-python)。
3.  如果無法使用，請建立訂用帳戶，並儲存訂用帳戶識別碼以供後續使用。 如需訂用帳戶的建立指示，請參閱[在 Azure Stack 中建立供應項目的訂用帳戶](../azure-stack-subscribe-plan-provision-vm.md)。 
4.  建立服務主體，並儲存其識別碼和祕密。 如需 Azure Stack 服務主體的建立指示，請參閱[為 Azure Stack 提供應用程式存取](../azure-stack-create-service-principals.md)。 
5.  確保您的服務主體在訂用帳戶中擁有參與者/擁有者角色。 如需如何對服務主體指派角色的指示，請參閱[為 Azure Stack 提供應用程式存取](../azure-stack-create-service-principals.md)。

## <a name="prerequisites"></a>必要條件

若要搭配 Azure Stack 使用 Python Azure SDK，您必須提供下列值，然後以環境變數設定值。 針對您的作業系統設定環境變數時，請參閱下表的指示。 

| 值 | 環境變數 | 說明 |
|---------------------------|-----------------------|-------------------------------------------------------------------------------------------------------------------------|
| 租用戶識別碼 | AZURE_TENANT_ID | 您的 Azure Stack [租用戶識別碼](../azure-stack-identity-overview.md)的值。 |
| 用戶端識別碼 | AZURE_CLIENT_ID | 您在本文件上一節中建立服務主體時儲存的服務主體應用程式識別碼。 |
| 訂用帳戶識別碼 | AZURE_SUBSCRIPTION_ID | [訂用帳戶識別碼](../azure-stack-plan-offer-quota-overview.md#subscriptions)是您存取 Azure Stack 中供應項目的方式。 |
| 用戶端密碼 | AZURE_CLIENT_SECRET | 服務主體建立時儲存的服務主體應用程式祕密。 |
| Resource Manager 端點 | ARM_ENDPOINT | 請參閱 [Azure Stack Resource Manager 端點](azure-stack-version-profiles-ruby.md#the-azure-stack-resource-manager-endpoint)。 |


## <a name="python-samples-for-azure-stack"></a>適用於 Azure Stack 的 Python 範例 

您可以使用下列程式碼範例，在 Azure Stack 中執行虛擬機器的一般管理工作。

程式碼範例會顯示您可以：

- 建立虛擬機器：
    - 建立 Linux 虛擬機器
    - 建立 Windows 虛擬機器
- 更新虛擬機器：
    - 展開磁碟機
    - 標記虛擬機器
    - 連接資料磁碟
    - 卸離資料磁碟
- 操作虛擬機器：
    - 啟動虛擬機器
    - 停止虛擬機器
    - 重新啟動虛擬機器
- 列出虛擬機器
- 刪除虛擬機器

若要檢閱程式碼來執行這些作業，請在 GitHub Repo [virtual-machines-python-manage](https://github.com/viananth/virtual-machines-python-manage/tree/8643ed4bec62aae6fdb81518f68d835452872f88) 中查看 Python 指令碼 **Hybrid/unmanaged-disks/example.py** 中的 **run_example()** 函式。

每個作業都會清楚地加上註解，並列印函式。
這些範例不一定依照上述清單中顯示的順序排列。


## <a name="run-the-python-sample"></a>執行 Python 範例

1.  如果還沒有 Python，請[安裝](https://www.python.org/downloads/)。

    此範例 (和 SDK) 與 Python 2.7、3.4、3.5 和 3.6 相容。

2.  Python 開發的一般建議是使用虛擬環境。 
    如需詳細資訊，請參閱 https://docs.python.org/3/tutorial/venv.html
    
    在 Python 3 上使用 "venv" 模組安裝和初始化虛擬環境 (您必須為 Python 2.7 安裝 [virtualenv](https://pypi.python.org/pypi/virtualenv))：

    ````bash
    python -m venv mytestenv # Might be "python3" or "py -3.6" depending on your Python installation
    cd mytestenv
    source bin/activate      # Linux shell (Bash, ZSH, etc.) only
    ./scripts/activate       # PowerShell only
    ./scripts/activate.bat   # Windows CMD only
    ````

3.  複製儲存機制。

    ````bash
    git clone https://github.com/Azure-Samples/virtual-machines-python-manage.git
    ````

4.  安裝使用 pip 的相依性。

    ````bash
    cd virtual-machines-python-manage\Hybrid
    pip install -r requirements.txt
    ````

5.  建立[服務主體](https://docs.microsoft.com/azure/azure-stack/azure-stack-create-service-principals)以搭配 Azure Stack 使用。 確保您的服務主體在訂用帳戶中擁有[參與者/擁有者角色](https://docs.microsoft.com/azure/azure-stack/azure-stack-create-service-principals#assign-role-to-service-principal)。

6.  設定下列變數，並將這些環境變數匯出到目前的殼層。 

    ```bash
    export AZURE_TENANT_ID={your tenant id}
    export AZURE_CLIENT_ID={your client id}
    export AZURE_CLIENT_SECRET={your client secret}
    export AZURE_SUBSCRIPTION_ID={your subscription id}
    export ARM_ENDPOINT={your AzureStack Resource Manager Endpoint}
    ```

7.  若要執行此範例，Azure Stack 市場中必須存在 Ubuntu 16.04-LTS 和 WindowsServer 2012-R2-Datacenter 映像。 這些可以是[從 Azure 下載](https://docs.microsoft.com/azure/azure-stack/azure-stack-download-azure-marketplace-item)或[加入平台映像儲存機制](https://docs.microsoft.com/azure/azure-stack/azure-stack-add-vm-image)。

8. 執行範例。

    ```
    python unmanaged-disks\example.py
    ```

## <a name="notes"></a>注意

您可能會想要再試一次使用 `virtual_machine.storage_profile.os_disk` 擷取 VM 的 OS 磁碟。
在某些情況下，如此可能會達成您的目的。但請注意，它會產生一個 `OSDisk` 物件。
為了更新 OS 磁碟的大小，與 `example.py` 一樣，您不需要 `OSDisk` 物件，而是 `Disk` 物件。
`example.py` 取得包含下列項目的 `Disk` 物件：

```python
os_disk_name = virtual_machine.storage_profile.os_disk.name
os_disk = compute_client.disks.get(GROUP_NAME, os_disk_name)
```

## <a name="next-steps"></a>後續步驟

- [Azure Python 開發中心](https://azure.microsoft.com/develop/python/)
- [Azure 虛擬機器文件](https://azure.microsoft.com/services/virtual-machines/)
- [虛擬機器的學習路徑](https://azure.microsoft.com/documentation/learning-paths/virtual-machines/)
- 如果您沒有 Microsoft Azure 訂用帳戶，則可以在[這裡](http://go.microsoft.com/fwlink/?LinkId=330212)建立免費的試用帳戶。
