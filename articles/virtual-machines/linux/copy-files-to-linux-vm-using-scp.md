---
title: 使用 SCP 將檔案移入和移出 Azure Linux VM | Microsoft Docs
description: 使用 SCP 和 SSH 金鑰組將檔案安全地移入和移出 Azure 中的 Linux VM。
services: virtual-machines-linux
documentationcenter: virtual-machines
author: dlepow
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-linux
ms.workload: infrastructure
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 07/12/2017
ms.author: danlep
ms.openlocfilehash: 0231e402848e617a46ca70470ba4d3272ace59f7
ms.sourcegitcommit: 5b2ac9e6d8539c11ab0891b686b8afa12441a8f3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/06/2018
ms.locfileid: "30904500"
---
# <a name="move-files-to-and-from-a-linux-vm-using-scp"></a>使用 SCP 將檔案移入和移出 Linux VM

本文說明如何使用安全複製 (SCP) 從您的工作站將檔案上移至 Azure Linux VM，或從 Azure Linux VM 下移至您的工作站。 在您的工作站與 Linux VM 之間快速又安全地移動檔案，對於管理您的 Azure 基礎結構來說相當重要。 

針對本文，您需要一個使用 [SSH 公開和私密金鑰檔案](mac-create-ssh-keys.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)在 Azure 中部署的 Linux VM。 此外，還需要一個用於您本機電腦的 SCP 用戶端。 此工具建置在 SSH 之上，並包含在大多數 Linux 和 Mac 電腦的預設 Bash 殼層中，以及部分 Windows 殼層中。

## <a name="quick-commands"></a>快速命令

將檔案向上複製到 Linux VM

```bash
scp file azureuser@azurehost:directory/targetfile
```

從 Linux VM 向下複製檔案

```bash
scp azureuser@azurehost:directory/file targetfile
```

## <a name="detailed-walkthrough"></a>詳細的逐步解說

作為範例，我們會將 Azure 組態檔上移至 Linux VM，以及將記錄檔目錄下拉，兩者皆使用 SCP 和 SSH 金鑰來完成。   

## <a name="ssh-key-pair-authentication"></a>SSH 金鑰組驗證

SCP 會針對傳輸層使用 SSH。 SSH 會處理目的地主機上的驗證，它會在 SSH 預設提供的加密通道中移動檔案。 針對 SSH 驗證，可以使用使用者名稱和密碼。 不過，建議的安全性最佳做法是使用 SSH 公用和私密金鑰驗證。 在 SSH 驗證連線之後，SCP 就會開始複製檔案。 藉由使用已正確設定的 `~/.ssh/config` 和 SSH 公用與私密金鑰，只要使用伺服器名稱 (或 IP 位址)，即可建立 SCP 連線。 如果您只有一個 SSH 金鑰，SCP 會在 `~/.ssh/` 目錄中尋找它，並預設使用它來登入 VM。

如需有關設定您的 `~/.ssh/config` 和 SSH 公用與私密金鑰的詳細資訊，請參閱[建立 SSH 金鑰](mac-create-ssh-keys.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。

## <a name="scp-a-file-to-a-linux-vm"></a>SCP 檔案至 Linux VM

針對第一個範例，我們會將用來部署自動化的 Azure 組態檔向上複製到 Linux VM。 由於這個檔案包含 Azure API 認證，其中包含祕密，因此安全性很重要。 SSH 提供的加密通道可保護此檔案的內容。

下列命令會將本機 *.azure/config* 檔案複製到 FQDN 為 *myserver.eastus.cloudapp.azure.com* 的 Azure VM。Azure VM 上的管理使用者名稱是 *azureuser*。 此檔案的目標為 */home/azureuser/* 目錄。 請在此命令中使用您自己的值來替代。

```bash
scp ~/.azure/config azureuser@myserver.eastus.cloudapp.com:/home/azureuser/config
```

## <a name="scp-a-directory-from-a-linux-vm"></a>從 Linux VM SCP 目錄

針對此範例，我們會從 Linux VM 將記錄檔目錄向下複製到您的工作站。 記錄檔可能包含也可能不包含敏感或祕密資料。 不過，使用 SCP 可確保將記錄檔的內容加密。 使用 SCP 來傳輸檔案，不僅是將記錄檔目錄和檔案向下移到您工作站的最簡單方式，同時也很安全。

下列命令會將 Azure VM 上 */home/azureuser/logs/* 目錄中的檔案複製到本機 /tmp 目錄：

```bash
scp -r azureuser@myserver.eastus.cloudapp.com:/home/azureuser/logs/. /tmp/
```

`-r` CLI 旗標指示 SCP 以遞迴方式從命令中列出的目錄點複製檔案和目錄。  另請注意，命令列語法類似 `cp` 複製命令。

## <a name="next-steps"></a>後續步驟

* [管理使用者、SSH，並使用 VMAccess 擴充功能檢查或修復 Azure Linux VM 上的磁碟](using-vmaccess-extension.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)