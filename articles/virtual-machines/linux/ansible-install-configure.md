---
title: 在 Azure 虛擬機器上安裝 Ansible
description: 了解如何安裝及設定 Ansible，以便管理 Ubuntu、CentOS 和 SLES 上的 Azure 資源
ms.service: ansible
keywords: ansible、azure、devops、bash、cloudshell、腳本、bash
author: tomarcher
manager: jeconnoc
ms.author: tarcher
ms.topic: quickstart
ms.date: 08/21/2018
ms.openlocfilehash: b714470cd12bb7a0cd2d2a00b4f09467726f505d
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46987045"
---
# <a name="install-ansible-on-azure-virtual-machines"></a>在 Azure 虛擬機器上安裝 Ansible

Ansible 可讓您將環境中的資源部署和設定自動化。 您可以使用 Ansible 在 Azure 中管理虛擬機器 (VM)，就像是任何其他資源一樣。 本文詳細說明如何針對某些最常見的 Linux 發行版，安裝 Ansible 和必要的 Azure Python SDK 模組。 您可以配合特定的平台調整安裝的套件，來將 Ansible 安裝在其他發行版上。 為了以安全的方式建立 Azure 資源，您也將了解如何建立及定義 Ansible 所要使用的認證。 如需可在 Cloud Shell 中使用的其他工具清單，請參閱 [Azure Cloud Shell 中的 Bash 功能和工具](../../cloud-shell/features.md#tools)。

## <a name="prerequisites"></a>必要條件

- **Azure 訂用帳戶** - 如果您沒有 Azure 訂用帳戶，請建立[免費帳戶](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio)。

- **Linux 或 Linux 虛擬機器的存取權** - 如果您沒有 Linux 機器，請建立 [Linux 虛擬機器](https://docs.microsoft.com/azure/virtual-network/quick-create-cli)。

- **Azure 服務主體**：請遵循[使用 Azure CLI 2.0 建立 Azure 服務主體](/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest#create-the-service-principal)一文中**建立服務主體**一節的指示。 記下 **appId**、**displayName**、**密碼**和**租用戶**的值。

## <a name="install-ansible-on-an-azure-linux-virtual-machine"></a>在 Azure Linux 虛擬機器上安裝 Ansible

登入您的 Linux 機器，並選取下列其中一個散發版本，以執行相關步驟來了解如何安裝 Ansible：

- [CentOS 7.4](#centos-74)
- [Ubuntu 16.04 LTS](#ubuntu1604-lts)
- [SLES 12 SP2](#sles-12-sp2)

### <a name="centos-74"></a>CentOS 7.4

在終端機或 Bash 視窗中輸入下列命令，以安裝 Azure Python SDK 模組和 Ansible 所需的套件：

```bash
## Install pre-requisite packages
sudo yum check-update; sudo yum install -y gcc libffi-devel python-devel openssl-devel epel-release
sudo yum install -y python-pip python-wheel

## Install Ansible and Azure SDKs via pip
sudo pip install ansible[azure]
```

請依照[建立 Azure 認證](#create-azure-credentials)一節中所列的指示操作。

### <a name="ubuntu-1604-lts"></a>Ubuntu 16.04 LTS

在終端機或 Bash 視窗中輸入下列命令，以安裝 Azure Python SDK 模組和 Ansible 所需的套件：


```bash
## Install pre-requisite packages
sudo apt-get update && sudo apt-get install -y libssl-dev libffi-dev python-dev python-pip

## Install Ansible and Azure SDKs via pip
sudo pip install ansible[azure]
```

請依照[建立 Azure 認證](#create-azure-credentials)一節中所列的指示操作。

### <a name="sles-12-sp2"></a>SLES 12 SP2

在終端機或 Bash 視窗中輸入下列命令，以安裝 Azure Python SDK 模組和 Ansible 所需的套件：

```bash
## Install pre-requisite packages
sudo zypper refresh && sudo zypper --non-interactive install gcc libffi-devel-gcc5 make \
    python-devel libopenssl-devel libtool python-pip python-setuptools

## Install Ansible and Azure SDKs via pip
sudo pip install ansible[azure]

# Remove conflicting Python cryptography package
sudo pip uninstall -y cryptography
```

請依照[建立 Azure 認證](#create-azure-credentials)一節中所列的指示操作。

## <a name="create-azure-credentials"></a>建立 Azure 認證

訂用帳戶識別碼可與建立服務主體時傳回的資訊搭配使用，以透過兩種方式之一來設定 Ansible 認證：

- [建立 Ansible 認證檔案](#file-credentials)
- [使用 Ansible 環境變數](#env-credentials)

如果您想要使用 Ansible Tower 或 Jenkins 等工具，您必須使用將服務主體值宣告為環境變數的選項。

### <a name="span-idfile-credentials-create-ansible-credentials-file"></a><span id="file-credentials"/>建立 Ansible 認證檔案

本節說明如何建立為 Ansible 提供認證的本機認證檔案。 如需如何定義 Ansible 認證的詳細資訊，請參閱 [Providing Credentials to Azure Modules](https://docs.ansible.com/ansible/guide_azure.html#providing-credentials-to-azure-modules) (提供認證給 Azure 模組)。

針對開發環境，請在您的主機虛擬機器上建立 Ansible 的「認證」檔案，如下所示：

```bash
mkdir ~/.azure
vi ~/.azure/credentials
```

將以下幾行插入「認證」檔案中 - 請將預留位置取代為建立服務主體時產生的資訊。

```bash
[default]
subscription_id=<your-subscription_id>
client_id=<security-principal-appid>
secret=<security-principal-password>
tenant=<security-principal-tenant>
```

儲存並關閉檔案。

### <a name="span-idenv-credentialsuse-ansible-environment-variables"></a><span id="env-credentials"/>使用 Ansible 環境變數

本節說明如何將 Ansible 認證匯出為環境變數，以設定您的認證。

在終端機或 Bash 視窗中，輸入下列命令：

```bash
export AZURE_SUBSCRIPTION_ID=<your-subscription_id>
export AZURE_CLIENT_ID=<security-principal-appid>
export AZURE_SECRET=<security-principal-password>
export AZURE_TENANT=<security-principal-tenant>
```

## <a name="verify-the-configuration"></a>驗證組態
若要確認設定成功，您現在可以使用 Ansible 建立資源群組。

[!INCLUDE [create-resource-group-with-ansible.md](../../../includes/ansible-create-resource-group.md)]

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"] 
> [使用 Ansible 在 Azure 中建立 Linux 虛擬機器](./ansible-create-vm.md)
