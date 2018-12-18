---
title: 將存取重設為 Azure Linux VM | Microsoft Docs
description: 如何使用 VMAccess 擴充功能和 Azure CLI 在 Linux VM 上管理系統管理使用者及重設存取
services: virtual-machines-linux
documentationcenter: ''
author: roiyz-msft
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: 261a9646-1f93-407e-951e-0be7226b3064
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: azurecli
ms.topic: article
ms.date: 05/10/2018
ms.author: roiyz
ms.openlocfilehash: 638ca5d1b1b68896ff5dcad70fedf27261ae96cb
ms.sourcegitcommit: f31bfb398430ed7d66a85c7ca1f1cc9943656678
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47452049"
---
# <a name="manage-administrative-users-ssh-and-check-or-repair-disks-on-linux-vms-using-the-vmaccess-extension-with-the-azure-cli"></a>使用 VMAccess 擴充功能搭配 Azure CLI 在 Linux VM 上管理系統管理使用者、SSH 及檢查或修復磁碟
## <a name="overview"></a>概觀
Linux VM 的磁碟顯示錯誤。 您不知怎麼重設 Linux VM的根密碼，或不小心刪除了 SSH 私密金鑰。 如果是過去資料中心的時代發生此狀況，您必須親赴現場，然後再開啟 KVM 才能存取伺服器主控台。 請將 Azure VMAccess 擴充功能想成 KVM 交換器，在此可以存取主控台重設 Linux 存取或執行磁碟等級維護。

本文將說明在這些項目以 Azure Resource Manager 虛擬機器的形式執行時，如何使用 Azure VMAccess 擴充功能來檢查或修復磁碟、重設使用者存取、管理系統管理使用者帳戶或更新 Linux 上的 SSH 組態。 如果您需要管理傳統虛擬機器，您可以依照[傳統 VM 文件](../linux/classic/reset-access-classic.md)中的指示操作。 
 
> [!NOTE]
> 如果您在安裝 AAD 登入擴充功能之後，使用 VMAccess 擴充功能重設 VM 的密碼，則您必須重新執行 AAD 登入擴充功能以便為您的機器重新啟用 AAD 登入。

## <a name="prerequisites"></a>必要條件
### <a name="operating-system"></a>作業系統

您可以對這類 Linux 發行版執行 VM 存取擴充功能。

| 配送映像 | 版本 |
|---|---|
| Ubuntu | 16.04 LTS、14.04 LTS 和 12.04 LTS |
| Debian | Debian 7.9+、8.2+ |
| Red Hat | RHEL 6.7+、7.1+ |
| Oracle Linux | 6.4+、7.0+ |
| Suse | 11 和 12 |
| OpenSuse | openSUSE Leap 42.2+ |
| CentOS | CentOS 6.3+、7.0+ |
| CoreOS | 494.4.0+ |

## <a name="ways-to-use-the-vmaccess-extension"></a>使用 VMAccess 擴充功能的方式
您可以透過兩種方式在 Linux VM 上使用「VMAccess 擴充功能」：

* 使用 Azure CLI 和必要的參數。
* [使用 VMAccess 擴充功能會處理然後作為行動依據的原始 JSON 檔案](#use-json-files-and-the-vmaccess-extension)。

下列範例會使用 [az vm user](/cli/azure/vm/user) 命令。 若要執行這些步驟，您需要安裝最新的 [Azure CLI](/cli/azure/install-az-cli2)，並且使用 [az login](/cli/azure/reference-index#az_login) 來登入 Azure 帳戶。

## <a name="update-ssh-key"></a>更新 SSH 金鑰
下列範例會更新名為 `myVM` 的 VM 上使用者 `azureuser` 的 SSH 金鑰：

```azurecli-interactive
az vm user update \
  --resource-group myResourceGroup \
  --name myVM \
  --username azureuser \
  --ssh-key-value ~/.ssh/id_rsa.pub
```

> **注意：** 若為 VM 上的系統管理使用者，`az vm user update` 命令會將新的公用金鑰文字附加至 `~/.ssh/authorized_keys` 檔案。 這並非取代或移除任何現有的 SSH 金鑰。 並不會移除先前在部署期間所設定的金鑰，也不會移除 VMAccess 擴充功能提供的後續更新。

## <a name="reset-password"></a>重設密碼
下列範例會重設名為 `myVM` 的 VM 上使用者 `azureuser` 的密碼：

```azurecli-interactive
az vm user update \
  --resource-group myResourceGroup \
  --name myVM \
  --username azureuser \
  --password myNewPassword
```

## <a name="restart-ssh"></a>重新啟動 SSH
下列範例會重新啟動 SSH 精靈，並將 SSH 組態重設成名為 `myVM` 的 VM 上之預設值：

```azurecli-interactive
az vm user reset-ssh \
  --resource-group myResourceGroup \
  --name myVM
```

## <a name="create-an-administrativesudo-user"></a>建立系統管理/sudo 使用者
下列範例會建立名為 `myNewUser` 且具有 **sudo** 權限的使用者。 此帳戶會使用 SSH 金鑰在名為 `myVM` 的 VM 上進行驗證。 之所以設計此方法，是為了協助您在遺失或忘記目前的認證時能夠重新取得 VM 的存取權。 最佳做法是限制具有 **sudo** 權限的帳戶。

```azurecli-interactive
az vm user update \
  --resource-group myResourceGroup \
  --name myVM \
  --username myNewUser \
  --ssh-key-value ~/.ssh/id_rsa.pub
```

## <a name="delete-a-user"></a>刪除使用者
下列範例會刪除名為 `myVM` 之 VM 上名為 `myNewUser` 的使用者：

```azurecli-interactive
az vm user delete \
  --resource-group myResourceGroup \
  --name myVM \
  --username myNewUser
```

## <a name="use-json-files-and-the-vmaccess-extension"></a>使用 JSON 檔案和 VMAccess 擴充功能
下列範例會使用原始 JSON 檔案。 請使用 [az vm extension set](/cli/azure/vm/extension#az_vm_extension_set) 來接著呼叫您的 JSON 檔案。 您也可以從 Azure 範本呼叫這些 JSON 檔案。 

### <a name="reset-user-access"></a>重設使用者存取
如果您已無法存取 Linux VM 上的根，可以啟動 VMAccess 指令碼來更新使用者的 SSH 金鑰或密碼。

若要更新使用者的 SSH 公開金鑰，請建立名為 `update_ssh_key.json` 的檔案，並以下列格式新增設定。 將您自己的值取代為 `username` 和 `ssh_key` 參數：

```json
{
  "username":"azureuser",
  "ssh_key":"ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQCZ3S7gGp3rcbKmG2Y4vGZFMuMZCwoUzZNGxxxxxx2XV2x9FfAhy8iGD+lF8UdjFX3t5ebMm6BnnMh8fHwkTRdOt3LDQq8o8ElTBrZaKPxZN2thMZnODs5Hlemb2UX0oRIGRcvWqsd4oJmxsXa/Si98Wa6RHWbc9QZhw80KAcOVhmndZAZAGR+Wq6yslNo5TMOr1/ZyQAook5C4FtcSGn3Y+WczaoGWIxG4ZaWk128g79VIeJcIQqOjPodHvQAhll7qDlItVvBfMOben3GyhYTm7k4YwlEdkONm4yV/UIW0la1rmyztSBQIm9sZmSq44XXgjVmDHNF8UfCZ1ToE4r2SdwTmZv00T2i5faeYnHzxiLPA3Enub7xxxxxxwFArnqad7MO1SY1kLemhX9eFjLWN4mJe56Fu4NiWJkR9APSZQrYeKaqru4KUC68QpVasNJHbuxPSf/PcjF3cjO1+X+4x6L1H5HTPuqUkyZGgDO4ynUHbko4dhlanALcriF7tIfQR9i2r2xOyv5gxJEW/zztGqWma/d4rBoPjnf6tO7rLFHXMt/DVTkAfn5wxxtLDwkn5FMyvThRmex3BDf0gujoI1y6cOWLe9Y5geNX0oj+MXg/W0cXAtzSFocstV1PoVqy883hNoeQZ3mIGB3Q0rIUm5d9MA2bMMt31m1g3Sin6EQ== azureuser@myVM"
}
```

執行 VMAccess 指令碼搭配︰

```azurecli-interactive
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVM \
  --name VMAccessForLinux \
  --publisher Microsoft.OSTCExtensions \
  --version 1.4 \
  --protected-settings update_ssh_key.json
```

若要重設使用者密碼，請建立名為 `reset_user_password.json` 的檔案，並以下列格式新增設定。 將您自己的值取代為 `username` 和 `password` 參數：

```json
{
  "username":"azureuser",
  "password":"myNewPassword" 
}
```

執行 VMAccess 指令碼搭配︰

```azurecli-interactive
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVM \
  --name VMAccessForLinux \
  --publisher Microsoft.OSTCExtensions \
  --version 1.4 \
  --protected-settings reset_user_password.json
```

### <a name="restart-ssh"></a>重新啟動 SSH
若要重新啟動 SSH 精靈並將 SSH 組態重設為預設值，請建立名為 `reset_sshd.json` 的檔案。 新增下列內容：

```json
{
  "reset_ssh": true
}
```

執行 VMAccess 指令碼搭配︰

```azurecli-interactive
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVM \
  --name VMAccessForLinux \
  --publisher Microsoft.OSTCExtensions \
  --version 1.4 \
  --protected-settings reset_sshd.json
```

### <a name="manage-administrative-users"></a>管理系統管理使用者

若要建立具有 **sudo** 權限的使用者以使用 SSH 金鑰來進行驗證，請建立名為 `create_new_user.json` 的檔案，並以下列格式新增設定。 將您自己的值取代為 `username` 和 `ssh_key` 參數。 之所以設計此方法，是為了協助您在遺失或忘記目前的認證時能夠重新取得 VM 的存取權。 最佳做法是限制具有 **sudo** 權限的帳戶。

```json
{
  "username":"myNewUser",
  "ssh_key":"ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQCZ3S7gGp3rcbKmG2Y4vGZFMuMZCwoUzZNG1vHY7P2XV2x9FfAhy8iGD+lF8UdjFX3t5ebMm6BnnMh8fHwkTRdOt3LDQq8o8ElTBrZaKPxZN2thMZnODs5Hlemb2UX0oRIGRcvWqsd4oJmxsXa/Si98Wa6RHWbc9QZhw80KAcOVhmndZAZAGR+Wq6yslNo5TMOr1/ZyQAook5C4FtcSGn3Y+WczaoGWIxG4ZaWk128g79VIeJcIQqOjPodHvQAhll7qDlItVvBfMOben3GyhYTm7k4YwlEdkONm4yV/UIW0la1rmyztSBQIm9sZmSq44XXgjVmDHNF8UfCZ1ToE4r2SdwTmZv00T2i5faeYnHzxiLPA3Enub7iUo5IdwFArnqad7MO1SY1kLemhX9eFjLWN4mJe56Fu4NiWJkR9APSZQrYeKaqru4KUC68QpVasNJHbuxPSf/PcjF3cjO1+X+4x6L1H5HTPuqUkyZGgDO4ynUHbko4dhlanALcriF7tIfQR9i2r2xOyv5gxJEW/zztGqWma/d4rBoPjnf6tO7rLFHXMt/DVTkAfn5woYtLDwkn5FMyvThRmex3BDf0gujoI1y6cOWLe9Y5geNX0oj+MXg/W0cXAtzSFocstV1PoVqy883hNoeQZ3mIGB3Q0rIUm5d9MA2bMMt31m1g3Sin6EQ== myNewUser@myVM",
  "password":"myNewUserPassword"
}
```

執行 VMAccess 指令碼搭配︰

```azurecli-interactive
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVM \
  --name VMAccessForLinux \
  --publisher Microsoft.OSTCExtensions \
  --version 1.4 \
  --protected-settings create_new_user.json
```

若要刪除使用者，請建立名為 `delete_user.json` 的檔案並新增下列內容。 將您自己的值取代為 `remove_user` 參數：

```json
{
  "remove_user":"myNewUser"
}
```

執行 VMAccess 指令碼搭配︰

```azurecli-interactive
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVM \
  --name VMAccessForLinux \
  --publisher Microsoft.OSTCExtensions \
  --version 1.4 \
  --protected-settings delete_user.json
```

### <a name="check-or-repair-the-disk"></a>檢查或修復磁碟
您也可以使用 VMAccess 來檢查並修復您新增至 Linux VM 的磁碟。

若要檢查然後修復磁碟，請建立名為 `disk_check_repair.json` 的檔案，並以下列格式新增設定。 將您自己的值取代為 `repair_disk` 的名稱：

```json
{
  "check_disk": "true",
  "repair_disk": "true, mydiskname"
}
```

執行 VMAccess 指令碼搭配︰

```azurecli-interactive
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVM \
  --name VMAccessForLinux \
  --publisher Microsoft.OSTCExtensions \
  --version 1.4 \
  --protected-settings disk_check_repair.json
```
## <a name="troubleshoot-and-support"></a>疑難排解與支援

### <a name="troubleshoot"></a>疑難排解

使用 Azure CLI，就可以從 Azure 入口網站擷取有關擴充功能部署狀態的資料。 若要查看指定 VM 的擴充功能部署狀態，請使用 Azure CLI 執行下列命令。

```azurecli
az vm extension list --resource-group myResourceGroup --vm-name myVM -o table
```

### <a name="support"></a>支援

如果您在本文中有任何需要協助的地方，您可以連絡 [MSDN Azure 和 Stack Overflow 論壇](https://azure.microsoft.com/support/forums/)上的 Azure 專家。 或者，您可以提出 Azure 支援事件。 請移至 [Azure 支援網站](https://azure.microsoft.com/support/options/)，然後選取 [取得支援]。 如需使用 Azure 支援的資訊，請參閱 [Microsoft Azure 支援常見問題集](https://azure.microsoft.com/support/faq/)。
