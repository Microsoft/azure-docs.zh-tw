---
title: 在 Azure Cloud Shell 中使用 Bash 執行 Ansible
description: 了解如何在 Azure Cloud Shell 中使用 Bash 執行各種 Ansible 工作
ms.service: ansible
keywords: ansible、azure、devops、bash、cloudshell、腳本、bash
author: tomarcher
manager: jeconnoc
ms.author: tarcher
ms.date: 08/07/2018
ms.topic: article
ms.openlocfilehash: 9928f646905dd0da4b15166ec55e5d8a183cb210
ms.sourcegitcommit: 1af4bceb45a0b4edcdb1079fc279f9f2f448140b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/09/2018
ms.locfileid: "42140614"
---
# <a name="run-ansible-with-bash-in-azure-cloud-shell"></a>在 Azure Cloud Shell 中使用 Bash 執行 Ansible

在本教學課程中，您會了解如何在 Cloud Shell 內使用 Bash，以將 Azure 訂用帳戶設定為 Ansible 工作區。 

## <a name="prerequisites"></a>必要條件

- **Azure 訂用帳戶** - 如果您沒有 Azure 訂用帳戶，請建立[免費帳戶](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio)。

- **設定 Azure Cloud Shell** - 如果您不熟悉 Azure Cloud Shell，[Azure Cloud Shell 中 Bash 的快速入門](https://docs.microsoft.com/azure/cloud-shell/quickstart)一文會說明如何啟動及設定 Cloud Shell。 

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="automatic-credential-configuration"></a>自動認證設定

登入 Cloud Shell 時，Ansible 會向 Azure 進行驗證以管理結礎結構，而不必進行任何額外設定。 如果您有多個訂用帳戶，您可以匯出 `AZURE_SUBSCRIPTION_ID` 環境變數，來選擇 Ansible 應該與哪個訂用帳戶搭配運作。 若要列出所有 Azure 訂用帳戶，請執行下列命令：

```azurecli-interactive
az account list
```

使用所要搭配運作的訂用帳戶**識別碼**，來設定 **AZURE_SUBSCRIPTION_ID**，如下所示：

```azurecli-interactive
export AZURE_SUBSCRIPTION_ID=<your-subscription-id>
```

## <a name="verify-the-configuration"></a>驗證組態
若要確認設定成功，請使用 Ansible 建立資源群組。

[!INCLUDE [create-resource-group-with-ansible.md](../../includes/ansible-create-resource-group.md)]

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"] 
> [使用 Ansible 在 Azure 中建立基本虛擬機器](/azure/virtual-machines/linux/ansible-create-vm)