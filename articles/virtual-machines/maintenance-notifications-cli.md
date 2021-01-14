---
title: 使用 CLI 取得維護通知
description: 使用 Azure CLI 來查看在 Azure 中執行之虛擬機器的維修通知，並開始自助式維護。
author: shants123
ms.service: virtual-machines
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 11/19/2019
ms.author: shants
ms.openlocfilehash: 4bd5c8ae7b4f2ba2d057f61712ce799814e19050
ms.sourcegitcommit: 2bd0a039be8126c969a795cea3b60ce8e4ce64fc
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/14/2021
ms.locfileid: "98202142"
---
# <a name="handling-planned-maintenance-notifications-using-the-azure-cli"></a>使用 Azure CLI 處理預定的維修通知

**本文適用于執行 Linux 和 Windows 的虛擬機器。**

您可以使用 CLI 來查看 Vm 何時排程進行 [維護](maintenance-notifications.md)。 您可以從 [az vm get-instance view 取得](/cli/azure/vm#az-vm-get-instance-view)預定的維修資訊。
 
只在有預定進行的維修作業時，才會傳回維修資訊。 

```azurecli-interactive
az vm get-instance-view -n myVM -g myResourceGroup --query instanceView.maintenanceRedeployStatus
```

輸出
```
      "maintenanceRedeployStatus": {
      "additionalProperties": {},
      "isCustomerInitiatedMaintenanceAllowed": true,
      "lastOperationMessage": null,
      "lastOperationResultCode": "None",
      "maintenanceWindowEndTime": "2018-06-04T16:30:00+00:00",
      "maintenanceWindowStartTime": "2018-05-21T16:30:00+00:00",
      "preMaintenanceWindowEndTime": "2018-05-19T12:30:00+00:00",
      "preMaintenanceWindowStartTime": "2018-05-14T12:30:00+00:00"
```

## <a name="start-maintenance"></a>開始維護

如果設定為 true，則下列呼叫會在 VM 上啟動維護 `IsCustomerInitiatedMaintenanceAllowed` 。

```azurecli-interactive
az vm perform-maintenance -g myResourceGroup -n myVM 
```

## <a name="classic-deployments"></a>傳統部署

[!INCLUDE [classic-vm-deprecation](../../includes/classic-vm-deprecation.md)]

如果您仍有使用傳統部署模型部署的舊版 VM，可以使用 Azure 傳統 CLI 查詢 VM 並起始維護。

請鍵入下列項目，以確認您使用傳統 VM 的模式正確：

```
azure config mode asm
```

若要取得名為 *myVM* 的 VM 維護狀態，請鍵入：

```
azure vm show myVM 
``` 

若要開始維護 *myService* 服務和 *myDeployment* 部署中名為 *myVM* 的傳統虛擬機器，請鍵入：

```
azure compute virtual-machine initiate-maintenance --service-name myService --name myDeployment --virtual-machine-name myVM
```

## <a name="next-steps"></a>後續步驟

您也可以使用 [Azure PowerShell](maintenance-notifications-powershell.md) 或 [入口網站](maintenance-notifications-portal.md)來處理預定的維護。
