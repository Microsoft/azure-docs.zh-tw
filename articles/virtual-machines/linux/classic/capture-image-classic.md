---
title: 擷取 Linux VM 的映像 | Microsoft Docs
description: 了解如何對以傳統部署模型建立的 Linux Azure 虛擬機器 (VM) 擷取映像。
services: virtual-machines-linux
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: tysonn
tags: azure-service-management
ROBOTS: NOINDEX
ms.assetid: 17d7ffee-a58e-4290-9de1-64c3cf1ddc05
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 03/14/2017
ms.author: cynthn
ms.openlocfilehash: ae87af45ffc442c0de6c7f703694a994e536cdb8
ms.sourcegitcommit: aa988666476c05787afc84db94cfa50bc6852520
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/10/2018
ms.locfileid: "37929207"
---
# <a name="how-to-capture-a-classic-linux-virtual-machine-as-an-image"></a>如何將傳統 Linux 虛擬機器擷取成映像
> [!IMPORTANT]
> Azure 建立和處理資源的部署模型有二種： [Resource Manager 和傳統](../../../resource-manager-deployment-model.md)。 本文涵蓋之內容包括使用傳統部署模型。 Microsoft 建議讓大部分的新部署使用 Resource Manager 模式。 了解如何[使用 Resource Manager 模型執行這些步驟](../capture-image.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。
> [!INCLUDE [virtual-machines-common-classic-createportal](../../../../includes/virtual-machines-classic-portal.md)]

本文說明如何將執行 Linux 的傳統 Azure 虛擬機器 (VM) 擷取成映像，以建立其他虛擬機器。 此映像包含作業系統磁碟和連結至虛擬機器的資料磁碟。 它並不包含網路組態，因此當您從此映像建立其他 VM 時，將需要設定該組態。

Azure 會將映像儲存在 [映像] 底下，連同您已上傳的任何映像。 如需映像的詳細資訊，請參閱[關於 Azure 中的虛擬機器映像][About Virtual Machine Images in Azure]。

## <a name="before-you-begin"></a>開始之前
這些步驟假設您已使用傳統部署模型建立 Azure VM，並設定好作業系統，包括連接任何資料磁碟。 如果您需要建立 VM，請閱讀[如何建立 Linux 虛擬機器][How to Create a Linux Virtual Machine]。

## <a name="capture-the-virtual-machine"></a>擷取虛擬機器
1. 使用您所選擇的 SSH 用戶端來[連接到 VM](../mac-create-ssh-keys.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。
2. 在 SSH 視窗中，輸入下列命令。 來自 `waagent` 的輸出可能會依此公用程式的版本而稍有不同：

    ```bash
    sudo waagent -deprovision+user
    ```

    上述命令會嘗試清除系統，使之適合重新佈建。 這項作業會執行下列工作：

   * 移除 SSH 主機金鑰 (如果組態檔中的 Provisioning.RegenerateSshHostKeyPair 是 'y')
   * 清除 /etc/resolv.conf 中的名稱伺服器設定
   * 移除 /etc/shadow 中的 `root` 使用者密碼 (如果組態檔中的 Provisioning.DeleteRootPassword 是 'y')
   * 移除快取的 DHCP 用戶端租用
   * 將主機名稱重設為 localhost.localdomain
   * 刪除最後佈建的使用者帳戶 (取自於 /var/lib/waagent) **和相關聯的資料**。

     > [!NOTE]
     > 解除佈建會將檔案和資料刪除以將映像「一般化」。 請只在您想要擷取做為新映像範本的 VM 上執行這個命令。 這不能保證映像檔中的所有機密資訊都會清除完畢或適合轉散發給第三方。

3. 輸入 **y** 繼續。 您可以新增 `-force` 參數，便不用進行此確認步驟。
4. 輸入 **Exit** 關閉 SSH 用戶端。

   > [!NOTE]
   > 其餘步驟會假設您已經在用戶端電腦上[安裝 Azure CLI](../../../cli-install-nodejs.md) 。 您也可以在 [Azure 入口網站](http://portal.azure.com)中完成接下來的所有步驟。

5. 從用戶端電腦，開啟 Azure CLI 並登入您的 Azure 訂用帳戶。 如需詳細資料，請閱讀[從 Azure CLI 連接到 Azure 訂用帳戶](/cli/azure/authenticate-azure-cli)。

   > [!NOTE]
   > 在 Azure 入口網站中，登入入口網站。

6. 請確定您是處於服務管理模式中：

    ```azurecli
    azure config mode asm
    ```

7. 關閉已取消佈建的 VM。 下列範例會關閉名為 `myVM` 的 VM：

    ```azurecli
    azure vm shutdown myVM
    ```
   如有需要，您可以使用 `azure vm list`，檢視在您的訂用帳戶中建立的所有 VM

   > [!NOTE]
   > 如果您使用 Azure 入口網站，請選取 VM，然後按一下 [停止] 來關閉 VM。

8. 當 VM 停止時，擷取映像。 下列範例會擷取名為 `myVM` 的 VM，並建立名為 `myNewVM` 的一般化映像：

    ```azurecli
    azure vm capture -t myVM myNewVM
    ```

    `-t` 子命令會刪除原本的虛擬機器。

    > [!NOTE]
    > 在 Azure 入口網站中，您可以從中樞功能表選取 [映像] 來擷取映像。 您需要提供映像的下列資訊：名稱、資源群組、位置、作業系統類型和儲存體 Blob 路徑。

9. 新的映像現在可從映像清單中取得，且可用來設定任何新的 VM。 您可以使用下列命令進行檢視：

   ```azurecli
   azure vm image list
   ```

   在 [Azure 入口網站](http://portal.azure.com) 上，新的映像會出現在屬於 [計算] 服務的 [VM 映像 (傳統)] 中。 按一下 Azure 服務清單底部的 [所有服務]，接著查看 [計算] 服務，即可存取 [VM 映像 (傳統)]。   

   ![成功擷取映像](./media/capture-image/VMCapturedImageAvailable.png)

## <a name="next-steps"></a>後續步驟
映像已準備好用來建立 VM。 您可以使用 Azure CLI 命令 `azure vm create`，並提供您已建立的映像名稱。 如需詳細資訊，請參閱[搭配使用 Azure CLI 與傳統部署模型](https://docs.microsoft.com/cli/azure/get-started-with-az-cli2)。

或者，您也可以透過 [Azure 入口網站](http://portal.azure.com)，使用 [映像] 方法並選取您已建立的映像來建立自訂 VM。 如需詳細資訊，請參閱[如何建立自訂 VM][How to Create a Custom Virtual Machine]。

**另請參閱：** [Azure Linux 代理程式使用者指南](../../extensions/agent-linux.md)

[About Virtual Machine Images in Azure]:../../virtual-machines-linux-classic-about-images.md
[How to Create a Custom Virtual Machine]:create-custom-classic.md
[How to Attach a Data Disk to a Virtual Machine]:attach-disk-classic.md
[How to Create a Linux Virtual Machine]:create-custom-classic.md
