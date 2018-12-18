---
title: 在 Azure 中建立和使用 Linux VM 的 SSH 金鑰組 | Microsoft Docs
description: 如何在 Azure 中建立和使用 Linux VM 的 SSH 公開和私密金鑰組，以改善驗證程序的安全性。
services: virtual-machines-linux
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: 34ae9482-da3e-4b2d-9d0d-9d672aa42498
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 09/11/2018
ms.author: cynthn
ms.openlocfilehash: 63a7602deee402bed056937c8465fd87c8256cd8
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46962840"
---
# <a name="quick-steps-create-and-use-an-ssh-public-private-key-pair-for-linux-vms-in-azure"></a>快速步驟：在 Azure 中建立和使用 Linux VM 的 SSH 公開和私密金鑰組

您可以利用安全殼層 (SSH) 金鑰組，在 Azure 上建立使用 SSH 金鑰來進行驗證的虛擬機器 (VM)，進而免除登入密碼的需求。 本文說明如何快速產生和使用 Linux VM 的 SSH 公開和私密金鑰檔案組。 您可以使用 Azure Cloud Shell、macOS 或 Linux 主機、Windows Subsystem for Linux 以及其他支援 OpenSSH 的工具，完成這些步驟。 

> [!NOTE]
> 使用 SSH 金鑰建立的 VM 預設會設定為停用密碼，這會大幅增加暴力密碼破解猜測嘗試的困難度。 

如需詳細的背景和範例，請參閱[建立 SSH 金鑰組的詳細步驟](create-ssh-keys-detailed.md)。

如需在 Windows 電腦上產生及使用 SSH 金鑰的其他方式，請參閱[如何在 Azure 上搭配 Windows 使用 SSH 金鑰](ssh-from-windows.md)。

[!INCLUDE [virtual-machines-common-ssh-support](../../../includes/virtual-machines-common-ssh-support.md)]

## <a name="create-an-ssh-key-pair"></a>建立 SSH 金鑰組

使用 `ssh-keygen` 命令來產生 SSH 公開和私密金鑰檔案。 根據預設，這些檔案會建立在 ~/.ssh 目錄中。 您可以指定不同位置和選擇性密碼 (「複雜密碼」) 來存取私密金鑰檔案。 如果指定位置中存在相同名稱的 SSH 金鑰組，則系統會覆寫那些檔案。

下列命令會使用 RSA 加密建立 SSH 金鑰組，位元長度為 2048：

```bash
ssh-keygen -t rsa -b 2048
```

如果您使用 [Azure CLI](/cli/azure) 搭配 [az vm create](/cli/azure/vm#az-vm-create) 命令來建立您的 VM，您即可選擇性地使用 `--generate-ssh-keys` 選項產生 SSH 公開金鑰和私密金鑰檔案。 金鑰檔案會儲存在 ~/.ssh 目錄中，除非使用 `--ssh-dest-key-path` 選項另行指定。 `--generate-ssh-keys` 選項不會覆寫現有的金鑰檔案，而會傳回錯誤。 在下列命令中，將 *VMname* 和 *RGname* 取代為您自己的值：

```azurecli
az vm create --name VMname --resource-group RGname --generate-ssh-keys 
```

## <a name="provide-an-ssh-public-key-when-deploying-a-vm"></a>部署 VM 時，提供 SSH 公開金鑰

若要建立使用 SSH 金鑰進行驗證的 Linux VM，請在使用 Azure 入口網站、Azure CLI、Azure Resource Manager 範本或其他方法建立 VM 時，指定您的 SSH 公開金鑰：

* [使用 Azure 入口網站建立 Linux 虛擬機器](quick-create-portal.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
* [使用 Azure CLI 建立 Linux 虛擬機器](quick-create-cli.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
* [使用 Azure 範本建立 Linux VM](create-ssh-secured-vm-from-template.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)

如果您不熟悉 SSH 公開金鑰的格式，可以使用下列 `cat` 命令顯示您的公開金鑰，如果需要，將 `~/.ssh/id_rsa.pub` 取代為您自己的公開金鑰檔案的路徑和檔案名稱：

```bash
cat ~/.ssh/id_rsa.pub
```

典型的公開金鑰值看起來像這樣：

```
ssh-rsa AAAAB3NzaC1yc2EAABADAQABAAACAQC1/KanayNr+Q7ogR5mKnGpKWRBQU7F3Jjhn7utdf7Z2iUFykaYx+MInSnT3XdnBRS8KhC0IP8ptbngIaNOWd6zM8hB6UrcRTlTpwk/SuGMw1Vb40xlEFphBkVEUgBolOoANIEXriAMvlDMZsgvnMFiQ12tD/u14cxy1WNEMAftey/vX3Fgp2vEq4zHXEliY/sFZLJUJzcRUI0MOfHXAuCjg/qyqqbIuTDFyfg8k0JTtyGFEMQhbXKcuP2yGx1uw0ice62LRzr8w0mszftXyMik1PnshRXbmE2xgINYg5xo/ra3mq2imwtOKJpfdtFoMiKhJmSNHBSkK7vFTeYgg0v2cQ2+vL38lcIFX4Oh+QCzvNF/AXoDVlQtVtSqfQxRVG79Zqio5p12gHFktlfV7reCBvVIhyxc2LlYUkrq4DHzkxNY5c9OGSHXSle9YsO3F1J5ip18f6gPq4xFmo6dVoJodZm9N0YMKCkZ4k1qJDESsJBk2ujDPmQQeMjJX3FnDXYYB182ZCGQzXfzlPDC29cWVgDZEXNHuYrOLmJTmYtLZ4WkdUhLLlt5XsdoKWqlWpbegyYtGZgeZNRtOOdN6ybOPJqmYFd2qRtb4sYPniGJDOGhx4VodXAjT09omhQJpE6wlZbRWDvKC55R2d/CSPHJscEiuudb+1SG2uA/oik/WQ== username@domainname
```

如果您在 Azure 入口網站或 Resource Manager 範本中複製和貼上要使用之公開金鑰檔案的內容，請確定您並未複製任何結尾空白字元。 若要在 macOS 中複製公開金鑰，您可以透過管道將公開金鑰檔案傳送至 **pbcopy**。 同樣地，在 Linux 中，您可以將公開金鑰檔案透過管道傳送至 **xclip** 等程式。

您放置在 Azure 中 Linux VM 上的公開金鑰預設會儲存在 ~/.ssh/id_rsa.pub 中，除非您在建立金鑰組時指定不同的位置。 若要使用 [Azure CLI 2.0](/cli/azure) 搭配現有公開金鑰來建立 VM，可以使用 [az vm create](/cli/azure/vm#az-vm-create) 命令加上 `--ssh-key-value` 選項，來指定這個公開金鑰的值並選擇性地指定位置。 在下列命令中，將 *VMname*、*RGname* 和 *keyFile* 取代為您自己的值：

```azurecli
az vm create --name VMname --resource-group RGname --ssh-key-value @keyFile
```

## <a name="ssh-into-your-vm"></a>透過 SSH 連線到您的 VM

在公開金鑰已部署到您的 Azure VM 且私密金鑰儲存在本機系統之後，可以使用 VM 的 IP 位址或 DNS 名稱，透過 SSH 連線到您的 VM。 在下列命令中，將 *azureuser* 和 *myvm.westus.cloudapp.azure.com* 取代為系統管理員使用者名稱和完整網域名稱 (或 IP 位址)：

```bash
ssh azureuser@myvm.westus.cloudapp.azure.com
```

如果您在建立金鑰組時指定了複雜密碼，請在登入程序期間出現提示時輸入複雜密碼。 VM 會新增至 ~/.ssh/known_hosts 檔案，而且系統不會要求您重新連線，除非 Azure VM 上的公開金鑰有變更或伺服器名稱已從 ~/.ssh/known_hosts 中移除。

## <a name="next-steps"></a>後續步驟

* 如需有關使用 SSH 金鑰組的詳細資訊，請參閱[建立及管理 SSH 金鑰組的詳細步驟](create-ssh-keys-detailed.md)。

* 如果您無法使用 SSH 連線到 Azure VM，請參閱[針對 SSH 連線至 Azure Linux VM 進行疑難排解](troubleshoot-ssh-connection.md)。
