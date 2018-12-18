---
title: 將 Linux 映像新增到 Azure Stack
description: 了解如何將 Linux 映像新增到 Azure Stack。
services: azure-stack
documentationcenter: ''
author: brenduns
manager: femila
editor: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/08/2018
ms.author: brenduns
ms.reviewer: jeffgo
ms.openlocfilehash: 64a860bc925b9c7499363c1fe39d03df88a9a51d
ms.sourcegitcommit: d98d99567d0383bb8d7cbe2d767ec15ebf2daeb2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/10/2018
ms.locfileid: "33935703"
---
# <a name="add-linux-images-to-azure-stack"></a>將 Linux 映像新增到 Azure Stack

*適用於：Azure Stack 整合系統和 Azure Stack 開發套件*

您可以透過將 Linux 型映像新增到「Azure Stack 市集」中，在 Azure Stack 上部署 Linux 虛擬機器 (VM)。 將 Linux 映像新增到 Azure Stack 的最簡單方式就是透過「市集管理」。 這些映像已備妥，並已針對與 Azure Stack 的相容性進行測試。

## <a name="marketplace-management"></a>市集管理

若要從 Azure Marketplace 下載 Linux 映像，請使用下列文章中的程序。 選取您想要在 Azure Stack 上提供給使用者的 Linux 映像。 

[將市集項目從 Azure 下載到 Azure Stack](azure-stack-download-azure-marketplace-item.md).

請注意，這些映像會不時更新，因此請經常查看市集管理以保持在最新狀態。

## <a name="prepare-your-own-image"></a>準備您自己的映像

 請盡可能下載透過市集管理提供並專為 Azure Stack 準備及測試的映像。 
 
 需要 Azure Linux 代理程式 (通常稱為 `WALinuxAgent` 或 `walinuxagent`)，而且並非所有代理程式版本都適用於 Azure Stack。 如果您建立自己的映像，則應該使用 2.2.18 版或更新版本。 請注意，Azure Stack 目前不支援 [cloud-init](https://cloud-init.io/)。

 您可以使用下列指示來準備自己的 Linux 映像：

   * [CentOS 型發行版本](../virtual-machines/linux/create-upload-centos.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
   * [Debian Linux](../virtual-machines/linux/debian-create-upload-vhd.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
   * [Red Hat Enterprise Linux](azure-stack-redhat-create-upload-vhd.md)
   * [SLES 和 openSUSE](../virtual-machines/linux/suse-create-upload-vhd.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
   * [Ubuntu Server](../virtual-machines/linux/create-upload-ubuntu.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)

    
## <a name="add-your-image-to-the-marketplace"></a>將您的映像新增至 Marketplace
 
遵循[將映像新增至 Marketplace](azure-stack-add-vm-image.md)。 請確定已將 `OSType` 參數設定為 `Linux`。

將映像新增到「市集」之後，便會建立「市集」項目，使用者就可以部署 Linux 虛擬機器。
