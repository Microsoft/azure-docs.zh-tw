---
title: 詳細步驟 - Azure Linux VM 的 SSH 金鑰組 | Microsoft Docs
description: 了解在 Azure 中建立和管理 Linux VM 的 SSH 公開和私密金鑰組的詳細步驟。
services: virtual-machines-linux
documentationcenter: ''
author: dlepow
manager: jeconnoc
editor: ''
tags: ''
ms.assetid: ''
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 04/17/2018
ms.author: danlep
ms.openlocfilehash: 804b7c0ff31575e6d62497fd5166e1a38a273076
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46965569"
---
# <a name="detailed-steps-create-and-manage-ssh-keys-for-authentication-to-a-linux-vm-in-azure"></a>詳細步驟：在 Azure 中建立和管理對 Linux VM 進行驗證所需的 SSH 金鑰 
您可以利用安全殼層 (SSH) 金鑰組，在 Azure 上建立依預設使用 SSH 金鑰進行驗證的 Linux 虛擬機器，而免除登入密碼的需求。 使用 Azure 入口網站、Azure CLI、Resource Manager 範本或其他工具建立的 VM，可以將 SSH 公開金鑰納入部署中，以設定 SSH 連線的 SSH 金鑰驗證。 

本文將詳細說明為 SSH 用戶端連線建立及管理 SSH RSA 公開/私密金鑰檔案組的背景和步驟。 如果您需要快速命令，請參閱[如何在 Azure 中建立 Linux VM 的 SSH 公開/私密金鑰組](mac-create-ssh-keys.md)。

如需在 Windows 電腦上產生及使用 SSH 金鑰的其他方式，請參閱[如何在 Azure 上搭配 Windows 使用 SSH 金鑰](ssh-from-windows.md)。

[!INCLUDE [virtual-machines-common-ssh-overview](../../../includes/virtual-machines-common-ssh-overview.md)]

### <a name="private-key-passphrase"></a>私密金鑰複雜密碼
SSH 私密金鑰應有非常安全的複雜密碼來保護金鑰。 此複雜密碼只能用來存取 SSH 私密金鑰檔，而*不是*使用者帳戶密碼。 當您將複雜密碼新增至 SSH 金鑰時，便會使用 128位元的 AES 來加密私密金鑰，而未利用複雜密碼解密的私密金鑰將沒有用處。 如果攻擊者竊取您的私密金鑰，而且該金鑰沒有複雜密碼，他們就能使用該私密金鑰來登入有對應公開金鑰的伺服器。 如果私密金鑰受到複雜密碼的保護，攻擊者就無法使用該金鑰，進而為您在 Azure 上的基礎結構提供額外的安全性。

[!INCLUDE [virtual-machines-common-ssh-support](../../../includes/virtual-machines-common-ssh-support.md)]

## <a name="ssh-keys-use-and-benefits"></a>SSH 金鑰的使用和好處

當您指定公開金鑰以建立 Azure VM 時，Azure 會將此公開金鑰 (以 `.pub` 的格式) 複製到 VM 上的 `~/.ssh/authorized_keys` 資料夾。 `~/.ssh/authorized_keys` 中的 SSH 金鑰用於挑戰用戶端，以符合 SSH 登入連線上的對應私密金鑰。 在使用 SSH 金鑰進行驗證的 Azure Linux VM 中，Azure 會將 SSHD 伺服器設定為不允許密碼登入，而僅允許以 SSH 金鑰登入。 因此，建立具 SSH 金鑰的 Azure Linux VM，即可協助保護 VM 部署的安全，並免除在 `sshd_config` 檔中停用密碼的標準部署後設定步驟。

如果您不想使用 SSH 金鑰，您可以將 Linux VM 設定為使用密碼驗證。 如果您的 VM 並未公開至網際網路，則使用密碼可能就以足夠。 不過，您仍然需要管理每個 Linux VM 的密碼，以及維護狀況良好的密碼原則和作法，例如最小密碼長度和定期更新。 使用 SSH 金鑰可降低橫跨多個 VM 管理個別認證的複雜度。

## <a name="generate-keys-with-ssh-keygen"></a>利用 ssh-keygen 產生金鑰

若要建立金鑰，一般常會使用 `ssh-keygen` 命令；您可以在 Azure Cloud Shell、macOS 或 Linux 主機、[Windows Subsystem for Linux](https://docs.microsoft.com/windows/wsl/about) 和其他工具中，透過 OpenSSH 公用程式來使用此命令。 `ssh-keygen` 在詢問一系列問題後，便會撰寫私密金鑰和對應的公開金鑰。 

依預設，SSH 金鑰會保留在 `~/.ssh` 目錄中。  如果您沒有 `~/.ssh` 目錄，`ssh-keygen` 命令會使用正確的權限為您建立。

### <a name="basic-example"></a>基本範例

下列 `ssh-keygen` 命令依預設會在 `~/.ssh` 目錄中產生 2048 位元的 SSH RSA 公開和私密金鑰檔案。 如果有 SSH 金鑰組存在於目前的位置，系統將會覆寫這些檔案。

```bash
ssh-keygen -t rsa -b 2048
```

### <a name="detailed-example"></a>詳細範例
下列範例說明可建立 SSH RSA 金鑰組的其他命令選項。 如果有 SSH 金鑰組存在於目前的位置，系統將會覆寫這些檔案。 

```bash
ssh-keygen \
    -t rsa \
    -b 4096 \
    -C "azureuser@myserver" \
    -f ~/.ssh/mykeys/myprivatekey \
    -N mypassphrase
```

**命令的說明**

`ssh-keygen` = 用來建立金鑰的程式

`-t rsa` = 要建立的金鑰類型，在此案例中採用 RSA 格式

`-b 4096` = 金鑰中的位元數，在此案例中為 4096

`-C "azureuser@myserver"` = 附加至公開金鑰檔案結尾以便輕鬆識別的註解。 通常會以電子郵件地址作為註解，但您可以使用任何最適合您的基礎結構的項目。

`-f ~/.ssh/mykeys/myprivatekey` = 私密金鑰檔案的檔案名稱 (如果您選擇不使用預設名稱)。 加上 `.pub` 的對應公開金鑰檔案會產生在相同的目錄中。 此目錄必須已存在。

`-N mypassphrase` = 用來存取私密金鑰檔案的其他複雜密碼。 

### <a name="example-of-ssh-keygen"></a>ssh-keygen 的範例

```bash
ssh-keygen -t rsa -b 2048 -C "azureuser@myserver"
Generating public/private rsa key pair.
Enter file in which to save the key (/home/azureuser/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/azureuser/.ssh/id_rsa.
Your public key has been saved in /home/azureuser/.ssh/id_rsa.pub.
The key fingerprint is:
14:a3:cb:3e:78:ad:25:cc:55:e9:0c:08:e5:d1:a9:08 azureuser@myserver
The keys randomart image is:
+--[ RSA 2048]----+
|        o o. .   |
|      E. = .o    |
|      ..o...     |
|     . o....     |
|      o S =      |
|     . + O       |
|      + = =      |
|       o +       |
|        .        |
+-----------------+
```

#### <a name="saved-key-files"></a>已儲存的金鑰檔案

`Enter file in which to save the key (/home/azureuser/.ssh/id_rsa): ~/.ssh/id_rsa`

本文中的金鑰組名稱。 依預設會有名為 `id_rsa` 的金鑰組；有些工具可能會預期私密金鑰的檔案名稱為 `id_rsa`，所以最好有此金鑰組。 `~/.ssh/` 目錄是 SSH 金鑰組和 SSH 組態檔的預設位置。 如果未指定完整路徑，則 `ssh-keygen` 會在目前的工作目錄 (而不是預設 `~/.ssh`) 中建立金鑰。

#### <a name="list-of-the-ssh-directory"></a>`~/.ssh` 目錄的清單

```bash
ls -al ~/.ssh
-rw------- 1 azureuser staff  1675 Aug 25 18:04 id_rsa
-rw-r--r-- 1 azureuser staff   410 Aug 25 18:04 id_rsa.pub
```

#### <a name="key-passphrase"></a>金鑰複雜密碼

`Enter passphrase (empty for no passphrase):`

「強烈」建議您為私密金鑰加上複雜密碼。 若未使用複雜密碼來保護金鑰檔案，任何人只要擁有該檔案，就可以用它登入具有對應公開金鑰的任何伺服器。 新增複雜密碼可提升防護能力，以防有人能夠取得您的私密金鑰檔案，讓您有時間變更金鑰。

## <a name="generate-keys-automatically-during-deployment"></a>在部署期間自動產生金鑰

如果您使用 [Azure CLI](/cli/azure) 來建立您的 VM，便可以執行 [az vm create](/cli/azure/vm#az_vm_create) 命令加上 `--generate-ssh-keys` 選項，以選擇性地產生 SSH 公開金鑰與私密金鑰檔案。 金鑰儲存在 ~/.ssh 目錄中。 請注意，如果金鑰已經存在於該位置，則此命令選項不會覆寫金鑰。

## <a name="provide-ssh-public-key-when-deploying-a-vm"></a>部署 VM 時，提供 SSH 公開金鑰

若要建立使用 SSH 金鑰進行驗證的 Linux VM，請在使用 Azure 入口網站、CLI、Resource Manager 範本或其他方法建立 VM 時，提供您的 SSH 公開金鑰。 使用入口網站時，您必須輸入公用金鑰本身。 如果您使用 [Azure CLI](/cli/azure) 搭配現有公開金鑰來建立 VM，可執行 [az vm create](/cli/azure/vm#az_vm_create) 命令加上 `--ssh-key-value` 選項，以指定這個公開金鑰的值或位置。 

如果您不熟悉 SSH 公開金鑰的格式，則可以如下所示執行 `cat`，並以自己的公開金鑰檔案位置取代 `~/.ssh/id_rsa.pub`，即可看到公開金鑰：

```bash
cat ~/.ssh/id_rsa.pub
```

輸出如下所示 (以下是擬定的輸出)：

```
ssh-rsa XXXXXXXXXXc2EAAAADAXABAAABAXC5Am7+fGZ+5zXBGgXS6GUvmsXCLGc7tX7/rViXk3+eShZzaXnt75gUmT1I2f75zFn2hlAIDGKWf4g12KWcZxy81TniUOTjUsVlwPymXUXxESL/UfJKfbdstBhTOdy5EG9rYWA0K43SJmwPhH28BpoLfXXXXXG+/ilsXXXXXKgRLiJ2W19MzXHp8z3Lxw7r9wx3HaVlP4XiFv9U4hGcp8RMI1MP1nNesFlOBpG4pV2bJRBTXNXeY4l6F8WZ3C4kuf8XxOo08mXaTpvZ3T1841altmNTZCcPkXuMrBjYSJbA8npoXAXNwiivyoe3X2KMXXXXXdXXXXXXXXXXCXXXXX/ azureuser@myserver
```

如果您複製公開金鑰檔案的內容，並將其貼到 Azure 入口網站或 Resource Manager 範本中，請確定您並未複製任何額外的空白字元或導入額外的分行符號。 例如，如果您使用 macOS，您可以透過管道將公開金鑰檔案 (預設為 `~/.ssh/id_rsa.pub`) 傳送至 **pbcopy** 以複製內容 (另有其他 Linux 程式可執行相同作業，例如 **xclip**)。

如果您想要使用多行格式的公開金鑰，您可以從您先前建立的公開金鑰在 pem 容器中產生 RFC4716 格式化金鑰。

若要從現有的 SSH 公開金鑰建立 RFC4716 格式的金鑰︰

```bash
ssh-keygen \
-f ~/.ssh/id_rsa.pub \
-e \
-m RFC4716 > ~/.ssh/id_ssh2.pem
```

## <a name="ssh-to-your-vm-with-an-ssh-client"></a>從 SSH 用戶端透過 SSH 連線至您的 VM
公開金鑰已部署到您的 Azure VM 且私密金鑰儲存在本機系統中，此時可以使用 VM 的 IP 位址或 DNS 名稱，透過 SSH 連線到您的 VM。 將下列命令中的 *azureuser* 和 *myvm.westus.cloudapp.azure.com* 換成系統管理員使用者名稱和完整網域名稱 (或 IP 位址)：

```bash
ssh azureuser@myvm.westus.cloudapp.azure.com
```

如果您在建立金鑰組時提供了複雜密碼，請在登入程序期間出現提示時輸入複雜密碼  (伺服器會新增至 `~/.ssh/known_hosts` 資料夾，而且系統不會要求您重新連線，除非 Azure VM 上的公開金鑰有變更或伺服器名稱已從 `~/.ssh/known_hosts` 中移除)。

## <a name="use-ssh-agent-to-store-your-private-key-passphrase"></a>使用 ssh-agent 儲存您的私密金鑰複雜密碼

若要避免在每次 SSH 登入時輸入私密金鑰檔案複雜密碼，您可以使用 `ssh-agent` 來快取您的私密金鑰檔案複雜密碼。 如果您使用 Mac，則 macOS 金鑰鏈會在您叫用 `ssh-agent`時安全地儲存私密金鑰複雜密碼。

確認並使用 `ssh-agent` 和 `ssh-add` 為 SSH 系統提供金鑰檔案的相關資訊，您即無須以互動方式使用複雜密碼。

```bash
eval "$(ssh-agent -s)"
```

現在使用 `ssh-add` 命令將私密金鑰加入至 `ssh-agent`。

```bash
ssh-add ~/.ssh/id_rsa
```

私密金鑰複雜密碼現已儲存在 `ssh-agent` 中。

## <a name="use-ssh-copy-id-to-copy-the-key-to-an-existing-vm"></a>使用 ssh-copy-id 將金鑰複製到現有 VM
如果您已建立 VM，您可以使用如下的命令，將新的 SSH 公開金鑰安裝到 Linux VM：

```bash
ssh-copy-id -i ~/.ssh/id_rsa.pub azureuser@myserver
```

## <a name="create-and-configure-an-ssh-config-file"></a>建立和設定 SSH 組態檔

您可以建立及設定 SSH 組態檔 (`~/.ssh/config`) 以加速登入，並最佳化您的 SSH 用戶端行為。 

下列範例說明可讓您以使用者身分使用預設的 SSH 私密金鑰快速登入特定 VM 的簡易設定。 

### <a name="create-the-file"></a>建立檔案

```bash
touch ~/.ssh/config
```

### <a name="edit-the-file-to-add-the-new-ssh-configuration"></a>編輯檔案以新增 SSH 組態

```bash
vim ~/.ssh/config
```

### <a name="example-configuration"></a>設定範例

為您的主機 VM 新增適當的組態設定。

```bash
# Azure Keys
Host myvm
  Hostname 102.160.203.241
  User azureuser
# ./Azure Keys
```

您可以新增其他主機的組態，讓每個主機使用其本身的專用金鑰組。 如需進階組態選項，請參閱 [SSH 組態檔](https://www.ssh.com/ssh/config/)。

您現在已擁有 SSH 金鑰組和設定好的 SSH 組態檔，因此能夠快速且安全地登入 Linux VM。 當您執行下列命令時，SSH 會從 SSH 組態檔的 `Host myvm` 區塊中找出並載入任何設定。

```bash
ssh myvm
```

第一次使用 SSH 金鑰登入伺服器時，命令會提示您輸入該金鑰檔案的複雜密碼。

## <a name="next-steps"></a>後續步驟

接下來是使用新的 SSH 公開金鑰建立 Azure Linux VM。 相較於以預設登入方法密碼建立的 VM，以 SSH 公開金鑰作為登入所建立的 Azure VM 比較安全。

* [使用 Azure 入口網站建立 Linux 虛擬機器](quick-create-portal.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
* [使用 Azure CLI 建立 Linux 虛擬機器](quick-create-cli.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
* [使用 Azure 範本建立 Linux VM](create-ssh-secured-vm-from-template.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
