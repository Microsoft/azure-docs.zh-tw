---
title: 透過 Azure REST API 建立 Linux 虛擬機器 | Microsoft Docs
description: 了解如何透過 Azure REST API 在 Azure 中建立使用受控磁碟與 SSH 驗證的 Linux 虛擬機器。
services: virtual-machines-linux
documentationcenter: virtual-machines
author: cynthn
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 06/05/2018
ms.author: cynthn
ms.openlocfilehash: 3eeaee9bc6320231f10aa85227e2f43756181806
ms.sourcegitcommit: 7c4fd6fe267f79e760dc9aa8b432caa03d34615d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47433475"
---
# <a name="create-a-linux-virtual-machine-that-uses-ssh-authentication-with-the-rest-api"></a>透過 REST API 建立使用 SSH 驗證的 Linux 虛擬機器

Azure 中的 Linux 虛擬機器 (VM) 包含各種資源，例如磁碟和網路介面，並且會定義參數，例如位置、大小和作業系統映象與驗證設定。

您可以透過 Azure 入口網站、Azure CLI 2.0、許多 Azure SDK、Azure Resource Manager 範本和許多第三方工具 (例如 Ansible 或 Terraform)，來建立 Linux VM。 這些工具最終都會使用 REST API 來建立 Linux VM。

本文說明如何使用 REST API 建立 Linux VM，來執行使用受控磁碟和 SSH 驗證的 Ubuntu 18.04-LTS。

## <a name="before-you-start"></a>開始之前

在建立及提交要求之前，您需要：

* 您訂用帳戶的 `{subscription-id}`
  * 如果您有多個訂用帳戶，請參閱[使用多個訂用帳戶](/cli/azure/manage-azure-subscriptions-azure-cli?view=azure-cli-latest#working-with-multiple-subscriptions)
* 您事先建立的 `{resourceGroupName}`
* 相同資源群組中的[虛擬網路介面](../../virtual-network/virtual-network-network-interface.md)
* SSH 金鑰組 (如果您還沒有的話，可以[產生新的金鑰組](mac-create-ssh-keys.md))

## <a name="request-basics"></a>要求基本

若要建立或更新虛擬機器，請使用下列 *PUT* 作業：

``` http
PUT https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachines/{vmName}?api-version=2017-12-01
```

除了 `{subscription-id}` 和 `{resourceGroupName}` 參數之外，您還必須指定 `{vmName}` (`api-version` 選用項目，但是本文是以 `api-version=2017-12-01` 進行測試)

以下是必要標頭：

| 要求標頭   | 說明 |
|------------------|-----------------|
| *Content-Type:*  | 必要。 設定為 `application/json`。 |
| *Authorization:* | 必要。 設定為無效的 `Bearer` [存取權杖](https://docs.microsoft.com/rest/api/azure/#authorization-code-grant-interactive-clients)。 |

如需使用 REST API 要求的一般資訊，請參閱 [REST API 要求/回應的元件](/rest/api/azure/#components-of-a-rest-api-requestresponse)。

## <a name="create-the-request-body"></a>建立要求本文

以下是用來建立要求本文的常用定義：

| 名稱                       | 必要 | 類型                                                                                | 說明  |
|----------------------------|----------|-------------------------------------------------------------------------------------|--------------|
| location                   | True     | 字串                                                                              | 資源位置。 |
| name                       |          | 字串                                                                              | 虛擬機器的名稱。 |
| properties.hardwareProfile |          | [HardwareProfile](/rest/api/compute/virtualmachines/createorupdate#hardwareprofile) | 指定虛擬機器的硬體設定。 |
| properties.storageProfile  |          | [StorageProfile](/rest/api/compute/virtualmachines/createorupdate#storageprofile)   | 指定虛擬機器磁碟的儲存體設定。 |
| properties.osProfile       |          | [OSProfile](/rest/api/compute/virtualmachines/createorupdate#osprofile)             | 指定虛擬機器的作業系統設定。 |
| properties.networkProfile  |          | [NetworkProfile](/rest/api/compute/virtualmachines/createorupdate#networkprofile)   | 指定虛擬機器的網路介面。 |

範例要求本文如下所示。 請確定您在 `{computerName}` 和 `{name}` 參數中指定 VM 名稱、指定您已在 `networkInterfaces` 底下建立的網路介面名稱、在 `adminUsername` 和 `path` 中指定您的使用者名稱，以及在 `keyData` 中指定您 SSH 金鑰組的「公用」部分 (舉例來說，位於 `~/.ssh/id_rsa.pub`)。 您想要修改的其他參數包括 `location` 和 `vmSize`。  

```json
{
  "location": "eastus",
  "name": "{vmName}",
  "properties": {
    "hardwareProfile": {
      "vmSize": "Standard_DS1_v2"
    },
    "storageProfile": {
      "imageReference": {
        "sku": "18.04-LTS",
        "publisher": "Canonical",
        "version": "latest",
        "offer": "UbuntuServer"
      },
      "osDisk": {
        "caching": "ReadWrite",
        "managedDisk": {
          "storageAccountType": "Premium_LRS"
        },
        "name": "myVMosdisk",
        "createOption": "FromImage"
      }
    },
    "osProfile": {
      "adminUsername": "{your-username}",
      "computerName": "{vmName}",
      "linuxConfiguration": {
        "ssh": {
          "publicKeys": [
            {
              "path": "/home/{your-username}/.ssh/authorized_keys",
              "keyData": "ssh-rsa AAAAB3NzaC1{snip}mf69/J1"
            }
          ]
        },
        "disablePasswordAuthentication": true
      }
    },
    "networkProfile": {
      "networkInterfaces": [
        {
          "id": "/subscriptions/{subscription-id}/resourceGroups/{resourceGroupName}/providers/Microsoft.Network/networkInterfaces/{existing-nic-name}",
          "properties": {
            "primary": true
          }
        }
      ]
    }
  }
}
```

如需要求主文中可用定義的完整清單，請參閱[虛擬機器建立或更新要求本文定義](/rest/api/compute/virtualmachines/createorupdate#definitions)。

## <a name="sending-the-request"></a>傳送要求

您可以使用喜好的用戶端來傳送這個 HTTP 要求。 也可以按一下 [試用] 按鈕，使用[瀏覽器內工具](https://docs.microsoft.com/rest/api/compute/virtualmachines/createorupdate)。

### <a name="responses"></a>回應

建立或更新虛擬機器的作業會有兩個成功的回應：

| 名稱        | 類型                                                                              | 說明 |
|-------------|-----------------------------------------------------------------------------------|-------------|
| 200 確定      | [VirtualMachine](/rest/api/compute/virtualmachines/createorupdate#virtualmachine) | OK          |
| 201 Created | [VirtualMachine](/rest/api/compute/virtualmachines/createorupdate#virtualmachine) | 建立時間     |

先前建立 VM 的要求本文範例，所傳回的扼要 *201 Created* 回應顯示已指派 *vmId*，且 *provisioningState* 為 *Creating*：

```json
{
    "vmId": "e0de9b84-a506-4b95-9623-00a425d05c90",
    "provisioningState": "Creating"
}
```

如需 REST API 回應的詳細資訊，請參閱[處理回應訊息](/rest/api/azure/#process-the-response-message)。

## <a name="next-steps"></a>後續步驟

如需有關 Azure REST API 或諸如 Azure CLI 或 Azure PowerShell 等其他管理工具的詳細資訊，請參閱下列內容：

- [Azure 計算提供者 REST API](/rest/api/compute/)
- [Get started with Azure REST API](/rest/api/azure/) (開始使用 Azure REST API)
- [Azure CLI](/cli/azure/)
- [Azure PowerShell 模組](/powershell/azure/overview)
