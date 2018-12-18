---
title: 如何使用 Azure CLI 將 MSI 存取權指派給 Azure 資源
description: 使用 Azure CLI 在一個資源上指派 MSI，並存取另一個資源的逐步解說指示。
services: active-directory
documentationcenter: ''
author: daveba
manager: mtillman
editor: ''
ms.service: active-directory
ms.component: msi
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 09/25/2017
ms.author: daveba
ms.openlocfilehash: 2e3b85251b9dabd6efd23e5b41372703a237d227
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46949072"
---
# <a name="assign-a-managed-service-identity-msi-access-to-a-resource-using-azure-cli"></a>使用 Azure CLI 將受控服務識別 (MSI) 存取權指派給資源

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

一旦已透過 MSI 設定 Azure 資源，您便可以將 MSI 存取權提供給另一個資源，就像任何安全性主體。 此範例將示範如何使用 Azure CLI 將 Azure 虛擬機器或虛擬機器擴展集的 MSI 存取權提供給 Azure 儲存體帳戶。

## <a name="prerequisites"></a>必要條件

[!INCLUDE [msi-qs-configure-prereqs](../../../includes/active-directory-msi-qs-configure-prereqs.md)]

若要執行 CLI 指令碼範例，您有三個選項：

- 從 Azure 入口網站使用 [Azure Cloud Shell](../../cloud-shell/overview.md) (請參閱下一節)。
- 請透過每個程式碼區塊右上角的 [立即試用] 按鈕，使用內嵌的 Azure Cloud Shell。
- 如果您偏好使用本機 CLI 主控台，請[安裝最新版的 Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli)。 

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="use-rbac-to-assign-the-msi-access-to-another-resource"></a>使用 RBAC 將 MSI 存取權指派給另一個資源

在 Azure 資源 ([Azure 虛擬機器](qs-configure-cli-windows-vm.md)或 [Azure 虛擬機器擴展集](qs-configure-cli-windows-vmss.md)) 上啟用 MSI 之後： 

1. 如果您要在本機主控台中使用 Azure CLI，請先使用 [az login](/cli/azure/reference-index#az-login) 登入 Azure。 使用您想部署 VM 或虛擬機器擴展集且已與 Azure 訂用帳戶相關聯的帳戶：

   ```azurecli-interactive
   az login
   ```

2. 在此範例中，我們會將 Azure 虛擬機器存取權給予儲存體帳戶。 首先我們使用 [az resource list](/cli/azure/resource/#az-resource-list)取得名為 "myVM" 的虛擬機器服務主體：

   ```azurecli-interactive
   spID=$(az resource list -n myVM --query [*].identity.principalId --out tsv)
   ```
   用於 Azure 虛擬機器擴展集的命令是相同的，不同的是您會取得名為 "DevTestVMSS" 的虛擬機器擴展集服務主體：
   
   ```azurecli-interactive
   spID=$(az resource list -n DevTestVMSS --query [*].identity.principalId --out tsv)
   ```

3. 一旦您有服務主體 ID 後，請使用 [az role assignment create](/cli/azure/role/assignment#az-role-assignment-create) 將虛擬機器或虛擬機器擴展集「讀取」存取權提供給稱為 "myStorageAcct" 的儲存體帳戶：

   ```azurecli-interactive
   az role assignment create --assignee $spID --role 'Reader' --scope /subscriptions/<mySubscriptionID>/resourceGroups/<myResourceGroup>/providers/Microsoft.Storage/storageAccounts/myStorageAcct
   ```

## <a name="troubleshooting"></a>疑難排解

如果資源的 MSI 沒有顯示在可用的身分識別清單中，請確認已正確啟用 MSI。 在我們的案例中，我們可以返回 [Azure 入口網站](https://portal.azure.com)中的 Azure 虛擬機器或虛擬機器擴展集：

- 查看 [組態] 頁面，並確定已啟用的 MSI = "Yes"。
- 查看 [擴充功能] 頁面，並確定已成功部署 MSI 擴充功能 (**擴充功能**頁面不適用於 Azure 虛擬機器擴展集)。

如果其中一者不正確，您可能需要再次在資源上重新部署 MSI，或針對部署失敗進行疑難排解。

## <a name="related-content"></a>相關內容

- 如需 MSI 的概觀，請參閱[受控服務識別概觀](overview.md)。
- 若要在 Azure 虛擬機器上啟用 MSI，請參閱[使用 Azure CLI 設定 Azure VM 受控服務身分識別 (MSI)](qs-configure-cli-windows-vm.md)。
- 若要在 Azure 虛擬機器擴展集上啟用 MSI，請參閱[使用 Azure 入口網站設定 Azure 虛擬機器擴展集的受控服務身分識別 (MSI)](qs-configure-portal-windows-vmss.md)

使用下列意見區段來提供意見反應，並協助我們改善及設計我們的內容。

