---
title: Azure Cloud Shell 中的 Bash 快速入門 | Microsoft Docs
description: Cloud Shell 中 Bash 的快速入門
services: ''
documentationcenter: ''
author: jluk
manager: timlt
tags: azure-resource-manager
ms.assetid: ''
ms.service: azure
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 03/12/2018
ms.author: juluk
ms.openlocfilehash: 0b3616a723e793ab1ce8d7bcca1f53ca10ec4f96
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46970648"
---
# <a name="quickstart-for-bash-in-azure-cloud-shell"></a>Azure Cloud Shell 中 Bash 的快速入門

本文件會詳細說明如何在 [Azure 入口網站](https://ms.portal.azure.com/)中使用 Azure Cloud Shell 中的 Bash。

> [!NOTE]
> 也提供 [Azure Cloud Shell 中的 PowerShell](quickstart-powershell.md) 快速入門。

## <a name="start-cloud-shell"></a>啟動 Cloud Shell
1. 從 Azure 入口網站的頂端導覽啟動 **Cloud Shell**。 <br>
![](media/quickstart/shell-icon.png)

2. 選取用來建立儲存體帳戶和 Microsoft Azure 檔案共用的訂用帳戶。
3. 選取 [建立儲存體]

> [!TIP]
> 在每個工作階段會自動驗證您以使用 Azure CLI。

### <a name="select-the-bash-environment"></a>選取 Bash 環境
確認殼層視窗左側的環境下拉式清單顯示為 `Bash`。 <br>
![](media/quickstart/env-selector.png)

### <a name="set-your-subscription"></a>設定您的訂用帳戶
1. 列出您可存取的訂用帳戶。
```azurecli-interactive
az account list
```

2. 設定您慣用的訂用帳戶： <br>
```azurecli-interactive
az account set --subscription my-subscription-name`
```

> [!TIP]
> 系統將使用 `/home/<user>/.azure/azureProfile.json` 來記住您的訂用帳戶，以供未來工作階段使用。

### <a name="create-a-resource-group"></a>建立資源群組
在 WestUS 中建立名為 "MyRG" 的新資源群組。
```azurecli-interactive
az group create --location westus --name MyRG
```

### <a name="create-a-linux-vm"></a>建立 Linux VM
在您的新資源群組中建立 Ubuntu VM。 Azure CLI 會建立 SSH 金鑰，並使用它們來設定 VM。 <br>

```azurecli-interactive
az vm create -n myVM -g MyRG --image UbuntuLTS --generate-ssh-keys
```

> [!NOTE]
> 使用 `--generate-ssh-keys` 會指示 Azure CLI 在您的 VM 和 `$Home` 目錄中建立並設定公用和私密金鑰。 根據預設，金鑰會放在 Cloud Shell 的 `/home/<user>/.ssh/id_rsa` 和 `/home/<user>/.ssh/id_rsa.pub` 上。 您的 `.ssh` 資料夾會保存在已連接之檔案共用的 5-GB 映像中，用以保存 `$Home`。

您在此 VM 上的使用者名稱，將會是用於 Cloud Shell 中的使用者名稱 ($User@Azure:)。

### <a name="ssh-into-your-linux-vm"></a>透過 SSH 連線到您的 Linux VM
1. 在 Azure 入口網站搜尋列中搜尋您的 VM 名稱。
2. 按一下 [連接] 以取得您的 VM 名稱和公用 IP 位址。 <br>
![](media/quickstart/sshcmd-copy.png)

3. 使用 `ssh` CMD，透過 SSH 連線到您的 VM。
```
ssh username@ipaddress
```

建立 SSH 連線時，應該會看到 Ubuntu 歡迎提示。 <br>
![](media/quickstart/ubuntu-welcome.png)

## <a name="cleaning-up"></a>清除 
1. 結束 SSH 工作階段。
```azurecli-interactive
exit
```

2. 刪除您的資源群組以及其中的任何資源。
```azurecli-interactive
az group delete -n MyRG
```

## <a name="next-steps"></a>後續步驟
[了解保存 Cloud Shell 中 Bash 的檔案](persisting-shell-storage.md) <br>
[了解 Azure CLI](https://docs.microsoft.com/cli/azure/) <br>
[了解 Azure 檔案儲存體](../storage/files/storage-files-introduction.md) <br>
