---
title: Azure 上的 FreeBSD 簡介 | Microsoft Docs
description: 了解如何使用 Azure 上的 FreeBSD 虛擬機器
services: virtual-machines-linux
documentationcenter: ''
author: thomas1206
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: 32b87a5f-d024-4da0-8bf0-77e233d1422b
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 09/13/2017
ms.author: huishao
ms.openlocfilehash: 0eab96d85aa9cc8bea8d30c5a36c8489b8ea652e
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46974165"
---
# <a name="introduction-to-freebsd-on-azure"></a>Azure 上的 FreeBSD 簡介
本文提供在 Azure 中執行 FreeBSD 虛擬機器的概觀。

## <a name="overview"></a>概觀
適用於 Microsoft Azure 的 FreeBSD 是一套先進的電腦作業系統，可用來為新式伺服器、桌上型電腦及內嵌平台提供技術支援。

Microsoft Corporation 目前在 Azure 上提供已預先設定 [Azure VM 客體代理程式](https://github.com/Azure/WALinuxAgent/)的 FreeBSD 映像。 目前，Microsoft 以映像形式提供下列 FreeBSD 版本：

- FreeBSD 10.3-RELEASE
- FreeBSD 10.4-RELEASE
- FreeBSD 11.1-RELEASE

此代理程式會負責 FreeBSD VM 與 Azure 網狀架構之間作業的通訊，例如在第一次使用 VM 時佈建 VM (使用者名稱、密碼或 SSH 金鑰、主機名稱等)，以及啟用選擇性 VM 延伸模組的功能。

至於未來的 FreeBSD 版本，策略是維持最新狀態，在 FreeBSD 版本工程小組發佈最新版本後不久，便立即提供最新版本。

## <a name="deploying-a-freebsd-virtual-machine"></a>部署 FreeBSD 虛擬機器
使用來自 Azure Marketplace 的映像從 Azure 入口網站部署 FreeBSD 虛擬機器相當簡單：

- [Azure Marketplace 上的 FreeBSD 10.3](https://azure.microsoft.com/marketplace/partners/microsoft/freebsd103)
- [Azure Marketplace 上的 FreeBSD 10.4](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/Microsoft.FreeBSD104)
- [Azure Marketplace 上的 FreeBSD 11.1](https://azuremarketplace.microsoft.com/marketplace/apps/Microsoft.FreeBSD111)

### <a name="create-a-freebsd-vm-through-azure-cli-on-freebsd"></a>透過 Azure CLI 在 FreeBSD 上建立 FreeBSD VM
首先，您必須透過下列命令在 FreeBSD 電腦上安裝 [Azure CLI](https://docs.microsoft.com/cli/azure/get-started-with-azure-cli)。

```bash 
curl -L https://aka.ms/InstallAzureCli | bash
```

如果您的 FreeBSD 電腦上未安裝 Bash，請先執行下列命令，再進行安裝。 

```bash
sudo pkg install bash
```

如果您的 FreeBSD 電腦上未安裝 Python，請先執行下列命令，再進行安裝。 

```bash
sudo pkg install python35
cd /usr/local/bin 
sudo rm /usr/local/bin/python 
sudo ln -s /usr/local/bin/python3.5 /usr/local/bin/python
```

進行安裝時，系統會向您提出下列問題：`Modify profile to update your $PATH and enable shell/tab completion now? (Y/n)`。 如果您回答 `y` 並輸入 `/etc/rc.conf` 作為 `a path to an rc file to update`，您可能會遇到下列問題：`ERROR: [Errno 13] Permission denied`。 若要解決此問題，您應該將檔案 `etc/rc.conf` 的寫入權限授與目前的使用者。

現在您可以登入 Azure 並建立您的 FreeBSD VM。 以下是一個建立 FreeBSD 11.0 VM 的範例。 您也可以新增 `--public-ip-address-dns-name` 參數，其中含有新建立之公用 IP 的全域唯一 DNS 名稱。 

```azurecli
az login 
az group create --name myResourceGroup --location eastus
az vm create --name myFreeBSD11 \
    --resource-group myResourceGroup \
    --image MicrosoftOSTC:FreeBSD:11.0:latest \
    --admin-username azureuser \
    --generate-ssh-keys
```

接著，您便可以透過上述部署作業輸出中所列印的 IP 位址來登入您的 FreeBSD VM。 

```bash
ssh azureuser@xx.xx.xx.xx -i /etc/ssh/ssh_host_rsa_key
```   

## <a name="vm-extensions-for-freebsd"></a>適用於 FreeBSD 的 VM 延伸模組
以下為 FreeBSD VM 中支援的 VM 延伸模組。

### <a name="vmaccess"></a>VMAccess
[VMAccess](https://github.com/Azure/azure-linux-extensions/tree/master/VMAccess) 延伸模組可以：

* 重設原始 sudo 使用者的密碼。
* 建立新的 sudo 使用者並指定密碼。
* 使用指定的金鑰來設定公開主機金鑰。
* 重設 VM 佈建期間所提供的公開主機金鑰 (如果未提供主機金鑰)。
* 開啟 SSH 連接埠 (22)，而且如果已將 reset_ssh 設定為 true，則還原 sshd_config。
* 移除現有的使用者。
* 檢查磁碟。
* 修復新增的磁碟。

### <a name="customscript"></a>CustomScript
[CustomScript](https://github.com/Azure/azure-linux-extensions/tree/master/CustomScript) 延伸模組可以：

* 如果提供，可從 Azure 儲存體或外部公用儲存體 (例如 GitHub) 下載自訂的指令碼。
* 執行進入點指令碼。
* 支援內嵌命令。
* 自動轉換 Shell 和 Python 指令碼中的 Windows 樣式新行字元。
* 自動移除 Shell 和 Python 指令碼中的 BOM。
* 保護 CommandToExecute 中的機密資料。

> [!NOTE]
> FreeBSD VM 目前僅支援 CustomScript 1.x 版。  

## <a name="authentication-user-names-passwords-and-ssh-keys"></a>驗證：使用者名稱、密碼和 SSH 金鑰
使用 Azure 入口網站來建立 FreeBSD 虛擬機器時，您必須提供使用者名稱、密碼或 SSH 公開金鑰。
在 Azure 上部署 FreeBSD 虛擬機器時所使用的使用者名稱，不可以和虛擬機器中已存在的系統帳戶 (UID < 100) 名稱相同 (例如 "root")。
目前只支援 RSA SSH 金鑰。 多行 SSH 金鑰的開頭必須為 `---- BEGIN SSH2 PUBLIC KEY ----`，結尾為 `---- END SSH2 PUBLIC KEY ----`。

## <a name="obtaining-superuser-privileges"></a>取得超級使用者權限
在 Azure 上部署虛擬機器執行個體時所指定的使用者帳戶是特殊權限帳戶。 sudo 的封裝已安裝於所發佈的 FreeBSD 映像中。
當您使用此使用者帳戶登入之後，您便能以 root 身分，使用命令語法來執行命令。

```
$ sudo <COMMAND>
```

您可以視需要使用 `sudo -s` 來取得 root shell。

## <a name="known-issues"></a>已知問題
[Azure VM 客體代理程式](https://github.com/Azure/WALinuxAgent/) 2.2.2 版有一個[已知問題] (https://github.com/Azure/WALinuxAgent/pull/517)，此問題會導致 Azure 上的 FreeBSD VM 佈建失敗。 [Azure VM 客體代理程式](https://github.com/Azure/WALinuxAgent/) 2.2.3 版和更新版本已包含此修正。 

## <a name="next-steps"></a>後續步驟
* 前往 [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/Microsoft.FreeBSD111) 以建立 FreeBSD VM。
