---
title: Azure 中的 OpenShift 必要條件 | Microsoft Docs
description: 在 Azure 中部署 OpenShift 的必要條件。
services: virtual-machines-linux
documentationcenter: virtual-machines
author: haroldw
manager: najoshi
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: ''
ms.author: haroldw
ms.openlocfilehash: 36271116d697e5ee6c6ed08d5fdc6063a511e820
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46984327"
---
# <a name="common-prerequisites-for-deploying-openshift-in-azure"></a>在 Azure 中開發 OpenShift 的一般必要條件

本文說明在 Azure 中部署 OpenShift Origin 或 OpenShift 容器平台的一般必要條件。

OpenShift 安裝是使用 Ansible 腳本。 Ansible 使用安全殼層 (SSH) 連接到所有叢集主機，以完成安裝步驟。

當您起始遠端主機的 SSH 連線時，無法輸入密碼。 因此，私密金鑰不能有相關聯的密碼，否則部署將會失敗。

由於所有虛擬機器 (VM) 是透過 Azure Resource Manager 範本部署的，因此會使用相同的公開金鑰存取所有 VM。 您也必須將對應的私密金鑰插入將執行所有腳本的 VM。 若要安全地執行這項作業，您可以使用 Azure Key Vault，將私密金鑰傳遞至 VM。

如果容器有永續性儲存體的需求，則需要永久性磁碟區。 OpenShift 支援適用於這項功能的 Azure 虛擬硬碟 (VHD)，但必須先將 Azure 設定為雲端提供者。 

在此模型中，OpenShift 將會：

- 在 Microsoft Azure 儲存體帳戶中建立 VHD 物件。
- 將 VHD 掛接到 VM 並將磁碟區格式化。
- 將磁碟區掛接到 Pod。

為了使這個設定運作，OpenShift 需要在 Azure 中執行上述工作的權限。 您需要有服務主體以達到此目的。 服務主體是 Azure Active Directory 中，獲授與資源權限的安全性帳戶。

服務主體必須具備儲存體帳戶和構成叢集之 VM 的存取權。 如果將所有 OpenShift 叢集資源都部署到單一資源群組中，則可以授與服務主體該資源群組的權限。

本指南描述如何建立與必要條件相關聯的成品。

> [!div class="checklist"]
> * 建立金鑰保存庫來管理 OpenShift 叢集的 SSH 金鑰。
> * 建立可供 Azure 雲端解決方案提供者使用的服務主體。

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

## <a name="sign-in-to-azure"></a>登入 Azure 
使用 [az login](/cli/azure/reference-index#az_login) 命令來登入 Azure 訂用帳戶並遵循畫面上的指示進行，或按一下 [試用] 來使用 Cloud Shell。

```azurecli 
az login
```
## <a name="create-a-resource-group"></a>建立資源群組

使用 [az group create](/cli/azure/group#az_group_create) 命令來建立資源群組。 Azure 資源群組是在其中部署與管理 Azure 資源的邏輯容器。 使用專用的資源群組來裝載金鑰保存庫。 此群組與 OpenShift 叢集資源部署所在的資源群組分開。 

下列範例會在 *eastus* 位置，建立名為 *keyvaultrg* 的資源群組：

```azurecli 
az group create --name keyvaultrg --location eastus
```

## <a name="create-a-key-vault"></a>建立金鑰保存庫
使用 [az keyvault create](/cli/azure/keyvault#az_keyvault_create) 命令來建立要儲存叢集之 SSH 金鑰的金鑰保存庫。 金鑰保存庫名稱必須是全域唯一的。

下列範例會在 *keyvaultrg* 資源群組中，建立名為 *keyvault* 的金鑰保存庫：

```azurecli 
az keyvault create --resource-group keyvaultrg --name keyvault \
       --enabled-for-template-deployment true \
       --location eastus
```

## <a name="create-an-ssh-key"></a>建立 SSH 金鑰 
需要 SSH 金鑰才能安全存取 OpenShift Origin 叢集。 使用 `ssh-keygen` 命令建立 SSH 金鑰組 (在 Linux 或 macOS 上)：
 
 ```bash
ssh-keygen -f ~/.ssh/openshift_rsa -t rsa -N ''
```

> [!NOTE]
> 您的 SSH 金鑰組不能有密碼。

如需有關 Windows 上的 SSH 金鑰詳細資訊，請參閱[如何在 Windows 上建立 SSH 金鑰](/azure/virtual-machines/linux/ssh-from-windows)。

## <a name="store-the-ssh-private-key-in-azure-key-vault"></a>將 SSH 私密金鑰儲存在 Azure Key Vault 中
OpenShift 部署會使用您建立的 SSH 金鑰來安全存取 OpenShift 主機。 若要啟用部署以安全地擷取 SSH 金鑰，請使用下列命令將金鑰儲存在金鑰保存庫中：

```azurecli
az keyvault secret set --vault-name keyvault --name keysecret --file ~/.ssh/openshift_rsa
```

## <a name="create-a-service-principal"></a>建立服務主體 
OpenShift 會使用使用者名稱與密碼或服務主體與 Azure 進行通訊。 Azure 服務主體是安全性識別，可供您與應用程式、服務及諸如 OpenShift 等自動化工具搭配使用。 您可以控制和定義關於服務主體可以在 Azure 中執行哪些作業的權限。 為了提高只提供使用者名稱和密碼的安全性，此範例會建立基本的服務主體。

使用 [az ad sp create-for-rbac](/cli/azure/ad/sp#az_ad_sp_create_for_rbac) 建立服務主體，並輸出 OpenShift 所需的認證。

下列範例會建立服務主體，並將其參與者權限指派給名為 myResourceGroup 的資源群組。 如果您使用 Windows，請另外執行 ```az group show --name myResourceGroup --query id```，並使用輸出摘要 --scopes 選項。

```azurecli
az ad sp create-for-rbac --name openshiftsp \
          --role Contributor --password {Strong Password} \
          --scopes $(az group show --name myResourceGroup --query id)
```

記下命令傳回的 appId 屬性：
```json
{
  "appId": "11111111-abcd-1234-efgh-111111111111",            
  "displayName": "openshiftsp",
  "name": "http://openshiftsp",
  "password": {Strong Password},
  "tenant": "XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX"
}
```
 > [!WARNING] 
 > 請務必建立安全的密碼。 請依照 [Azure AD 密碼規則和限制](/azure/active-directory/active-directory-passwords-policy)指引。

如需有關服務主體的詳細資訊，請參閱[使用 Azure CLI 建立 Azure 服務主體](https://docs.microsoft.com/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest)。

## <a name="next-steps"></a>後續步驟

本文涵蓋下列主題：
> [!div class="checklist"]
> * 建立金鑰保存庫來管理 OpenShift 叢集的 SSH 金鑰。
> * 建立可供 Azure 雲端解決方案提供者使用的服務主體。

接下來，部署 OpenShift 叢集：

- [部署 OpenShift Origin](./openshift-origin.md)
- [部署 OpenShift 容器平台](./openshift-container-platform.md)

