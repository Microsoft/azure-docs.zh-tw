---
title: 使用 Azure Active Directory 認證登入 Linux VM | Microsoft Docs
description: 在此做法指示中，您將了解如何建立及設定要將 Azure Active Directory 驗證用於使用者登入的 Linux VM
services: virtual-machines-linux
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: ''
ms.assetid: ''
ms.service: virtual-machines-linux
ms.devlang: azurecli
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 06/17/2018
ms.author: cynthn
ms.openlocfilehash: 2ec712dcce1295a91f552176ddcf6572d3f23ecc
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46993556"
---
# <a name="log-in-to-a-linux-virtual-machine-in-azure-using-azure-active-directory-authentication-preview"></a>在 Azure 中使用 Azure Active Directory 驗證登入 Linux 虛擬機器 (預覽)

若要在 Azure 中改善 Linux 虛擬機器 (VM) 的安全性，您可以整合 Azure Active Directory (AD) 驗證。 當您將 Azure AD 驗證用於 Linux VM 時，您將可集中控制和強制執行允許或拒絕存取 VM 的原則。 本文說明如何建立及設定使用 Azure AD 驗證的 Linux VM。

> [!NOTE]
> 這項功能處於預覽狀態，不建議用於生產虛擬機器或工作負載。 請在您預期要在測試後捨棄的測試虛擬機器上使用此功能。

在 Azure 中使用 Azure AD 驗證登入 Linux VM 有許多優點，包括：

- **更高的安全性：**
  - 您可以使用公司 AD 認證來登入 Azure Linux VM。 不需要建立本機系統管理員帳戶及管理認證存留期。
  - 藉由減少您對本機系統管理員帳戶的依賴，您無須再擔心認證遺失/遭竊、使用者設定弱式認證等問題。
  - 針對 Azure AD 目錄而設定的密碼複雜性及密碼存留期原則，也有助於保護 Linux VM。
  - 若要進一步保護 Azure 虛擬機器的登入，您可以設定多因素驗證。
  - 使用 Azure Active Directory 登入 Linux 虛擬機器的能力，也適用於使用[同盟服務](../../active-directory/hybrid/how-to-connect-fed-whatis.md)的客戶。

- **無縫式共同作業：** 透過角色型存取控制 (RBAC)，您可以指定哪些人能夠以一般使用者的身分或系統管理員權限登入指定的 VM。 當使用者加入或退出您的小組時，您可以更新 VM 的 RBAC 原則，以授與適當的存取權。 這項體驗遠比經由刪除 VM 移除非必要的 SSH 公開金鑰來得容易。 當員工離開您的組織時，其使用者帳戶會從 Azure AD 中停用或移除，且他們將無法再存取您的資源。

### <a name="supported-azure-regions-and-linux-distributions"></a>支援的 Azure 區域和 Linux 發行版

以下是目前在這項功能的預覽期間支援的 Linux 發行版：

| 配送映像 | 版本 |
| --- | --- |
| CentOS | CentOS 6.9 和 CentOS 7.4 |
| Debian | Debian 9 |
| RedHat Enterprise Linux | RHEL 6、RHEL 7 | 
| Ubuntu Server | Ubuntu 14.04 LTS、Ubuntu Server 16.04、Ubuntu Server 17.10 和 Ubuntu Server 18.04 |

以下是目前在這項功能的預覽期間支援的 Azure 區域：

- 所有全域 Azure 區域

>[!IMPORTANT]
> 若要使用這項預覽功能，請您僅在支援的 Azure 區域中部署支援的 Linux 發行版。 Azure Government 或主權雲端不支援這項功能。

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝和使用 CLI，本教學課程會要求您執行 Azure CLI 2.0.31 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI]( /cli/azure/install-azure-cli)。

## <a name="create-a-linux-virtual-machine"></a>建立 Linux 虛擬機器

使用 [az group create](/cli/azure/group#az-group-create) 建立資源群組，然後透過 [az vm create](/cli/azure/vm#az-vm-create)，使用支援的發行版在支援的區域中建立 VM。 下列範例會將名為 *myVM*、使用 *Ubuntu 16.04 LTS* 的 VM 部署到 *southcentralus* 區域中名為 *myResourceGroup* 的資源群組中。 在下列範例中，您可以視需要提供自己的資源群組和 VM 名稱。

```azurecli-interactive
az group create --name myResourceGroup --location southcentralus

az vm create \
    --resource-group myResourceGroup \
    --name myVM \
    --image UbuntuLTS \
    --admin-username azureuser \
    --generate-ssh-keys
```

建立虛擬機器和支援資源需要幾分鐘的時間。

## <a name="install-the-azure-ad-login-vm-extension"></a>安裝 Azure AD 登入 VM 擴充功能。

若要使用 Azure AD 認證登入 Linux VM，請安裝 Azure Active Directory 登入 VM 擴充功能。 VM 擴充功能是小型的應用程式，可在「Azure 虛擬機器」上提供部署後設定及自動化工作。 使用 [az vm extension set](/cli/azure/vm/extension#az-vm-extension-set)，將 *AADLoginForLinux* 擴充功能安裝在 *myResourceGroup* 資源群組中名為 *myVM* 的 VM 上：

```azurecli-interactive
az vm extension set \
    --publisher Microsoft.Azure.ActiveDirectory.LinuxSSH \
    --name AADLoginForLinux \
    --resource-group myResourceGroup \
    --vm-name myVM
```

在 VM 上安裝擴充功能後，會顯示 [成功] 的 *ProvisioningState*。

## <a name="configure-role-assignments-for-the-vm"></a>設定 VM 的角色指派

Azure 角色型存取控制 (RBAC) 原則會決定哪些人可以登入 VM。 有兩個 RBAC 角色可用來授權 VM 登入：

- **虛擬機器系統管理員登入**：被指派此角色的使用者能夠以 Windows 系統管理員或 Linux 根使用者權限登入 Azure 虛擬機器。
- **虛擬機器使用者登入**：被指派此角色的使用者能夠以一般使用者權限登入 Azure 虛擬機器。

> [!NOTE]
> 若要讓使用者透過 SSH 登入 VM，您必須指派 [虛擬機器系統管理員登入] 或 [虛擬機器使用者登入] 角色。 被指派 VM 的 [擁有者] 或 [參與者] 角色的 Azure 使用者，並不會自動取得透過 SSH 登入 VM 的權限。

下列範例會使用 [az role assignment create](/cli/azure/role/assignment#az-role-assignment-create) 將 [虛擬機器系統管理員登入] 角色指派給您目前 Azure 使用者的 VM。 作用中 Azure 帳戶的使用者名稱可透過 [az account show](/cli/azure/account#az-account-show) 來取得，而*範圍*會設定為在上一個步驟中使用 [az vm show](/cli/azure/vm#az-vm-show) 建立的 VM。 範圍也可指派於資源群組或訂用帳戶層級上，並套用一般 RBAC 繼承權限。 如需詳細資訊，請參閱[角色型存取控制](../../azure-resource-manager/resource-group-overview.md#access-control)

```azurecli-interactive
username=$(az account show --query user.name --output tsv)
vm=$(az vm show --resource-group myResourceGroup --name myVM --query id -o tsv)

az role assignment create \
    --role "Virtual Machine Administrator Login" \
    --assignee $username \
    --scope $vm
```

> [!NOTE]
> 如果 AAD 網域和登入使用者名稱網域不相符，您必須以 --assignee-object-id 指定使用者帳戶的物件識別碼，而不只是以 -assignee 指定使用者名稱。 您可以使用 [az ad user list](/cli/azure/ad/user#az-ad-user-list) 取得使用者帳戶的物件識別碼。

如需關於如何使用 RBAC 來管理 Azure 訂用帳戶資源存取權的詳細資訊，請參閱使用 [Azure CLI](../../role-based-access-control/role-assignments-cli.md)、[Azure 入口網站](../../role-based-access-control/role-assignments-portal.md)或 [Azure PowerShell](../../role-based-access-control/role-assignments-powershell.md)。

您也可以設定 Azure AD，以要求特定使用者需要進行多因素驗證才能登入 Linux 虛擬機器。 如需詳細資訊，請參閱[開始在雲端中使用 Azure Multi-Factor Authentication](../../multi-factor-authentication/multi-factor-authentication-get-started-cloud.md)。

## <a name="log-in-to-the-linux-virtual-machine"></a>登入 Linux 虛擬機器

首先，請使用 [az vm show](/cli/azure/vm#az-vm-show) 檢視虛擬機器的 IP 位址：

```azurecli-interactive
az vm show --resource-group myResourceGroup --name myVM -d --query publicIps -o tsv
```

使用 Azure AD 認證登入 Azure Linux 虛擬機器。 `-l` 參數可讓您指定自己的 Azure AD 帳戶位址。 依照上述命令的輸出，指定 VM 的公用 IP 位址。

```azurecli-interactive
ssh -l azureuser@contoso.onmicrosoft.com publicIps
```

系統會提示您使用一次性使用碼在 [https://microsoft.com/devicelogin](https://microsoft.com/devicelogin) 上登入 Azure AD。 將一次性使用碼複製並貼到裝置的登入頁面中，如下列範例所示：

```bash
~$ ssh -l azureuser@contoso.onmicrosoft.com 13.65.237.247
To sign in, use a web browser to open the page https://microsoft.com/devicelogin and enter the code FJS3K6X4D to authenticate. Press ENTER when ready.
```

出現提示時，請在登入頁面上輸入您的 Azure AD 登入認證。 當您成功通過驗證時，網頁瀏覽器中會顯示下列訊息：

    You have signed in to the Microsoft Azure Linux Virtual Machine Sign-In application on your device.

關閉瀏覽器視窗，返回 SSH 提示字元，然後按 **Enter** 鍵。 您此時會已使用指派的角色權限 (例如 [VM 使用者]或 [VM 系統管理員]) 登入 Azure Linux 虛擬機器。 如果您的使用者帳戶已被指派 [虛擬機器系統管理員登入] 角色，您可以使用 `sudo` 執行需要根權限的命令。

## <a name="troubleshoot-sign-in-issues"></a>對登入問題進行疑難排解

您在嘗試使用 Azure AD 認證進行 SSH 連線的一些常見錯誤，包括未指派 RBAC 角色，以及重複出現登入提示。 請參考下列各節，以修正這些問題。

### <a name="access-denied-rbac-role-not-assigned"></a>拒絕存取：未指派 RBAC 角色

如果您在 SSH 提示字元中看到下列錯誤，請確認您已為 VM [設定 RBAC 原則](#configure-rbac-policy-for-the-virtual-machine)，而可為使用者授與 [虛擬機器系統管理員登入] 或 [虛擬機器使用者登入] 角色：

```bash
login as: azureuser@contoso.onmicrosoft.com
Using keyboard-interactive authentication.
To sign in, use a web browser to open the page https://microsoft.com/devicelogin and enter the code FJX327AXD to authenticate. Press ENTER when ready.
Using keyboard-interactive authentication.
Access denied:  to sign-in you be assigned a role with action 'Microsoft.Compute/virtualMachines/login/action', for example 'Virtual Machine User Login'
Access denied
```

### <a name="continued-ssh-sign-in-prompts"></a>持續出現 SSH 登入提示

如果您已在網頁瀏覽器中成功完成驗證步驟，系統可能會立即提示您再次登入以全新的使用碼登入。 之所以發生此錯誤，通常是因為您在 SSH 提示字元中指定的登入名稱與您用來登入 Azure AD 的帳戶不相符。 若要更正此問題：

- 確認在 SSH 提示字元中指定的登入名稱正確無誤。 登入名稱若有拼字錯誤，即可能導致您在 SSH 提示字元中指定的登入名稱與您用來登入 Azure AD 的帳戶不相符。 例如，如果您輸入 *azuresuer@contoso.onmicrosoft.com*，而不是 *azureuser@contoso.onmicrosoft.com*。
- 如果您有多個使用者帳戶，請確定您在登入 Azure AD 時，並未在瀏覽器視窗中提供不同的使用者帳戶。
- Linux 是區分大小寫的作業系統。 'Azureuser@contoso.onmicrosoft.com' 和 'azureuser@contoso.onmicrosoft.com' 之間的差異可能會導致不相符。 請確定您在 SSH 提示字元中指定 UPN 時使用的是正確的大小寫設定。

## <a name="preview-feedback"></a>預覽意見反應

在 [Azure AD 意見反應論壇](https://feedback.azure.com/forums/169401-azure-active-directory?category_id=166032)上分享您關於這個預覽功能的意見反應或報告它的使用問題

## <a name="next-steps"></a>後續步驟

如需有關 Azure Active Directory 的詳細資訊，請參閱[什麼是 Azure Active Directory](../../active-directory/fundamentals/active-directory-whatis.md)
