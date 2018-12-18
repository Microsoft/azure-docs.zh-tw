---
title: 使用 Azure CLI 在 DevTest Labs 中建立和管理虛擬機器 | Microsoft Docs
description: 了解如何使用 Azure CLI 在 Azure DevTest Labs 中建立及管理虛擬機器
services: devtest-lab,virtual-machines,lab-services
documentationcenter: na
author: spelluru
manager: femila
editor: ''
ms.service: lab-services
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/17/2018
ms.author: spelluru
ms.openlocfilehash: ccf9b08856fcc652e3ad4b2b31587d43d7ef9cca
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46995942"
---
# <a name="create-and-manage-virtual-machines-with-devtest-labs-using-the-azure-cli"></a>使用 Azure CLI 在 DevTest Labs 中建立和管理虛擬機器
本快速入門會引導您完成建立、啟動、連線、更新和清理實驗室中的開發電腦。 

開始之前：

* 若尚未建立實驗室，請參閱[這裡](devtest-lab-create-lab.md)的指示。

* [安裝 Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli)。 若要開始，請執行 az login 建立 Azure 連線。 

## <a name="create-and-verify-the-virtual-machine"></a>建立並確認虛擬機器 
使用 SSH 驗證從 Marketplace 映像建立 VM。
```azurecli
az lab vm create --lab-name sampleLabName --resource-group sampleLabResourceGroup --name sampleVMName --image "Ubuntu Server 16.04 LTS" --image-type gallery --size Standard_DS1_v2 --authentication-type  ssh --generate-ssh-keys --ip-configuration public 
```
> [!NOTE]
> 在 --resource-group 參數中放入**實驗室的資源群組**名稱。
>

如果您想要使用公式建立 VM，請在 [az lab vm create](https://docs.microsoft.com/cli/azure/lab/vm#az-lab-vm-create) 中使用 --formula 參數。


確認有可用的 VM。
```azurecli
az lab vm show --lab-name sampleLabName --name sampleVMName --resource-group sampleResourceGroup --expand 'properties($expand=ComputeVm,NetworkInterface)' --query '{status: computeVm.statuses[0].displayStatus, fqdn: fqdn, ipAddress: networkInterface.publicIpAddress}'
```
```json
{
  "fqdn": "lisalabvm.southcentralus.cloudapp.azure.com",
  "ipAddress": "13.85.228.112",
  "status": "Provisioning succeeded"
}
```

## <a name="start-and-connect-to-the-virtual-machine"></a>啟動並連接至虛擬機器
啟動 VM。
```azurecli
az lab vm start --lab-name sampleLabName --name sampleVMName --resource-group sampleLabResourceGroup
```
> [!NOTE]
> 在 --resource-group 參數中放入**實驗室的資源群組**名稱。
>

連接到 VM：[SSH](../virtual-machines/linux/mac-create-ssh-keys.md) 或[遠端桌面](../virtual-machines/windows/connect-logon.md)。
```bash
ssh userName@ipAddressOrfqdn 
```

## <a name="update-the-virtual-machine"></a>更新虛擬機器
將構件套用至 VM。
```azurecli
az lab vm apply-artifacts --lab-name  sampleLabName --name sampleVMName  --resource-group sampleResourceGroup  --artifacts @/artifacts.json
```

```json
[
  {
    "artifactId": "/artifactSources/public repo/artifacts/linux-java",
    "parameters": []
  },
  {
    "artifactId": "/artifactSources/public repo/artifacts/linux-install-nodejs",
    "parameters": []
  },
  {
    "artifactId": "/artifactSources/public repo/artifacts/linux-apt-package",
    "parameters": [
      {
        "name": "packages",
        "value": "abcd"
      },
      {
        "name": "update",
        "value": "true"
      },
      {
        "name": "options",
        "value": ""
      }
    ]
  } 
]
```

列出實驗室中可用的構件。
```azurecli
az lab vm show --lab-name sampleLabName --name sampleVMName --resource-group sampleResourceGroup --expand "properties(\$expand=artifacts)" --query 'artifacts[].{artifactId: artifactId, status: status}'
```
```json
{
  "artifactId": "/subscriptions/abcdeftgh1213123/resourceGroups/lisalab123RG822645/providers/Microsoft.DevTestLab/labs/lisalab123/artifactSources/public repo/artifacts/linux-install-nodejs",
  "status": "Succeeded"
}
```

## <a name="stop-and-delete-the-virtual-machine"></a>停止並刪除虛擬機器    
停止 VM。
```azurecli
az lab vm stop --lab-name sampleLabName --name sampleVMName --resource-group sampleResourceGroup
```

刪除 VM。
```azurecli
az lab vm delete --lab-name sampleLabName --name sampleVMName --resource-group sampleResourceGroup
```

[!INCLUDE [devtest-lab-try-it-out](../../includes/devtest-lab-try-it-out.md)]
