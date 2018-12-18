---
title: 在 Azure 中建立及上傳 Ubuntu Linux VHD
description: 了解如何建立及上傳包含 Ubuntu Linux 作業系統的 Azure 虛擬硬碟 (VHD)。
services: virtual-machines-linux
documentationcenter: ''
author: szarkos
manager: jeconnoc
editor: tysonn
tags: azure-resource-manager,azure-service-management
ms.assetid: 3e097959-84fc-4f6a-8cc8-35e087fd1542
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 03/12/2018
ms.author: szark
ms.openlocfilehash: 5b2f902389b96136e0ea0c4c58f5e8be8144a248
ms.sourcegitcommit: 8ebcecb837bbfb989728e4667d74e42f7a3a9352
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/21/2018
ms.locfileid: "42146199"
---
# <a name="prepare-an-ubuntu-virtual-machine-for-azure"></a>準備適用於 Azure 的 Ubuntu 虛擬機器
[!INCLUDE [learn-about-deployment-models](../../../includes/learn-about-deployment-models-both-include.md)]

## <a name="official-ubuntu-cloud-images"></a>官方 Ubuntu 雲端映像
Ubuntu 現在發佈官方 Azure VHD 提供下載，位於：[http://cloud-images.ubuntu.com/](http://cloud-images.ubuntu.com/)。 如果您需要針對 Azure 建置您專用的 Ubuntu 映像，而不使用下面的手動程序，建議您視需要從使用這些已知可運作的 VHD 並加以自訂來開始建置。 最新的映像版本一律可以在下列位置找到︰

* Ubuntu 12.04/Precise︰ [ubuntu-12.04-server-cloudimg-amd64-disk1.vhd.zip](https://cloud-images.ubuntu.com/precise/current/precise-server-cloudimg-amd64-disk1.vhd.zip)
* Ubuntu 14.04/Trusty︰ [ubuntu-14.04-server-cloudimg-amd64-disk1.vhd.zip](http://cloud-images.ubuntu.com/releases/trusty/release/ubuntu-14.04-server-cloudimg-amd64-disk1.vhd.zip)
* Ubuntu 16.04/Xenial︰ [ubuntu-16.04-server-cloudimg-amd64-disk1.vhd.zip](http://cloud-images.ubuntu.com/releases/xenial/release/ubuntu-16.04-server-cloudimg-amd64-disk1.vhd.zip)

## <a name="prerequisites"></a>必要條件
本文假設您已將 Ubuntu Linux 作業系統安裝到虛擬硬碟。 有多個工具可用來建立 .vhd 檔案，例如，像是 Hyper-V 的虛擬化解決方案。 如需指示，請參閱 [安裝 Hyper-V 角色及設定虛擬機器](http://technet.microsoft.com/library/hh846766.aspx)。

**Ubuntu 安裝注意事項**

* 如需有關準備 Azure 之 Linux 的更多秘訣，另請參閱 [一般 Linux 安裝注意事項](create-upload-generic.md#general-linux-installation-notes) 。
* Azure 不支援 VHDX 格式，只支援 **固定 VHD**。  您可以使用 Hyper-V 管理員或 convert-vhd Cmdlet，將磁碟轉換為 VHD 格式。
* 安裝 Linux 系統時，建議您使用標準磁碟分割而不是 LVM (常是許多安裝的預設設定)。 這可避免 LVM 與複製之虛擬機器的名稱衝突，特別是為了疑難排解而需要將作業系統磁碟連接至其他虛擬機器時。 如果願意，您可以在資料磁碟上使用 [LVM](configure-lvm.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) 或 [RAID](configure-raid.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。
* 請勿在作業系統磁碟上設定交換磁碟分割。 您可以設定 Linux 代理程式在暫存資源磁碟上建立交換檔。  您可以在以下步驟中找到與此有關的詳細資訊。
* Azure 上的所有 VHD 必須具有與 1 MB 對應的虛擬大小。 從未經處理的磁碟轉換成 VHD 時，您必須確定未經處理的磁碟大小在轉換前是 1 MB 的倍數。 如需詳細資訊，請參閱 [Linux 安裝注意事項](create-upload-generic.md#general-linux-installation-notes)。

## <a name="manual-steps"></a>手動步驟
> [!NOTE]
> 在嘗試為 Azure 建立您的自訂 Ubuntu 映像之前，請考慮改為使用來自 [http://cloud-images.ubuntu.com/](http://cloud-images.ubuntu.com/) 的已預先建置並已測試的映像。
> 
> 

1. 在 Hyper-V 管理員的中間窗格中，選取虛擬機器。

2. 按一下 **[連接]** ，以開啟虛擬機器的視窗。

3. 取代映像中的目前儲存機制，以使用 Ubuntu 的 Azure 儲存機制。 此步驟會隨著 Ubuntu 版本而略有不同。
   
    建議您在編輯 `/etc/apt/sources.list` 之前先進行備份：
   
        # sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak

    Ubuntu 12.04：
   
        # sudo sed -i 's/[a-z][a-z].archive.ubuntu.com/azure.archive.ubuntu.com/g' /etc/apt/sources.list
        # sudo apt-get update

    Ubuntu 14.04：
   
        # sudo sed -i 's/[a-z][a-z].archive.ubuntu.com/azure.archive.ubuntu.com/g' /etc/apt/sources.list
        # sudo apt-get update

    Ubuntu 16.04：
   
        # sudo sed -i 's/[a-z][a-z].archive.ubuntu.com/azure.archive.ubuntu.com/g' /etc/apt/sources.list
        # sudo apt-get update

4. Ubuntu 的 Azure 映像現在遵循「硬體啟用」  (HWE) 核心。 執行下列命令，將作業系統更新為最新的核心：

    Ubuntu 12.04：
   
        # sudo apt-get update
        # sudo apt-get install linux-image-generic-lts-trusty linux-cloud-tools-generic-lts-trusty
        # sudo apt-get install hv-kvp-daemon-init
        (recommended) sudo apt-get dist-upgrade
   
        # sudo reboot
   
    Ubuntu 14.04：
   
        # sudo apt-get update
        # sudo apt-get install linux-image-virtual-lts-vivid linux-lts-vivid-tools-common
        # sudo apt-get install hv-kvp-daemon-init
        (recommended) sudo apt-get dist-upgrade
   
        # sudo reboot

    Ubuntu 16.04：
   
        # sudo apt-get update
        # sudo apt-get install linux-generic-hwe-16.04 linux-cloud-tools-generic-hwe-16.04
        (recommended) sudo apt-get dist-upgrade

        # sudo reboot

    **另請參閱：**
    - [https://wiki.ubuntu.com/Kernel/LTSEnablementStack](https://wiki.ubuntu.com/Kernel/LTSEnablementStack)
    - [https://wiki.ubuntu.com/Kernel/RollingLTSEnablementStack](https://wiki.ubuntu.com/Kernel/RollingLTSEnablementStack)


5. 修改 Grub 的核心開機程式行，使其額外包含用於 Azure 的核心參數。 若要這樣做，請在文字編輯器中開啟 `/etc/default/grub`，找到名為 `GRUB_CMDLINE_LINUX_DEFAULT` 的變數 (或視需要加入它)，然後進行編輯以使其包含以下參數：
   
        GRUB_CMDLINE_LINUX_DEFAULT="console=tty1 console=ttyS0,115200n8 earlyprintk=ttyS0,115200 rootdelay=300"

    儲存並關閉此檔案，然後執行 `sudo update-grub`。 這將確保所有主控台訊息都會傳送給第一個序列埠，有助於 Azure 技術支援團隊進行問題偵錯程序。

6. 確定您已安裝 SSH 伺服器，並已設定為在開機時啟動。  這通常是預設值。

7. 安裝 Azure Linux 代理程式：
   
        # sudo apt-get update
        # sudo apt-get install walinuxagent

    >[!Note]
    若已安裝 `NetworkManager` 和 `NetworkManager-gnome` 套件，則 `walinuxagent` 套件可能會將它們移除。

8. 執行下列命令，以取消佈建虛擬機器，並準備將它佈建於 Azure 上：
   
        # sudo waagent -force -deprovision
        # export HISTSIZE=0
        # logout

9. 在 Hyper-V 管理員中，依序按一下 [動作] -> [關閉]。 您現在可以將 Linux VHD 上傳至 Azure。

## <a name="references"></a>參考
[Ubuntu 硬體啟用 (HWE) 核心](https://wiki.ubuntu.com/Kernel/LTSEnablementStack)

## <a name="next-steps"></a>後續步驟
您現在可以開始使用您的 Ubuntu Linux 虛擬硬碟在 Azure 建立新的虛擬機器。 如果您是第一次將 .vhd 檔案上傳至 Azure，請參閱[從自訂磁碟建立 Linux VM](upload-vhd.md#option-1-upload-a-vhd)。




