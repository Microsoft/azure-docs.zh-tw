---
title: 在 Azure 入口網站中建立 VM | Microsoft Docs
description: 在 Azure 入口網站中建立 Windows 虛擬機器。
services: virtual-machines-windows
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: ''
tags: azure-service-management
ROBOTS: NOINDEX
ms.assetid: 1871f823-ebd7-4eff-9a22-8e2411555595
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.devlang: na
ms.topic: article
ms.date: 05/30/2017
ms.author: cynthn
ms.openlocfilehash: 5fd2128ff436d3211f41c7dfdcc4c2b8aabd0eb0
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38232321"
---
# <a name="create-a-virtual-machine-running-windows-in-the-azure-portal"></a>在 Azure 入口網站中建立執行 Windows 的虛擬機器
> [!div class="op_single_selector"]
> * [Azure 入口網站](tutorial.md)
> * [PowerShell：傳統部署](create-powershell.md)
>
>

<br>

> [!IMPORTANT]
> Azure 建立和處理資源的部署模型有二種： [Resource Manager 和傳統](../../../resource-manager-deployment-model.md)。 本文涵蓋之內容包括使用傳統部署模型。 Microsoft 建議讓大部分的新部署使用 Resource Manager 模式。 了解如何透過 **Azure 入口網站**，[使用 Resource Manager 部署模型執行這些步驟](../../virtual-machines-windows-hero-tutorial.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。
> [!INCLUDE [virtual-machines-common-classic-createportal](../../../../includes/virtual-machines-classic-portal.md)]

本教學課程示範如何在 Azure 入口網站中建立執行 Windows 的 Azure 虛擬機器 (VM)。 我們將使用 Windows Server 映像做為範例，這只是 Azure 提供眾多映像中的一種。 請注意，您可以選擇何種映像取決於您的訂用帳戶。 例如，Windows 桌面映像可能可供 MSDN 訂閱者使用。

本節說明如何使用 Azure 入口網站中的**儀表板**以選擇並建立虛擬機器。

您也可以使用 [您自己的映像](createupload-vhd.md)來建立 VM。 若要了解上述方法與其他方法，請參閱 [建立 Windows 虛擬機器的其他方式](../../virtual-machines-windows-creation-choices.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。

## <a id="createvirtualmachine"> </a>建立虛擬機器
[!INCLUDE [virtual-machines-create-WindowsVM](../../../../includes/virtual-machines-create-windowsvm.md)]

## <a name="next-steps"></a>後續步驟
* 深入了解如何在 Azure 入口網站，[使用 Resource Manager 部署模型建立 VM](../../virtual-machines-windows-hero-tutorial.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。
* 登入虛擬機器。 如需相關指示，請參閱 [登入執行 Windows Server 的虛擬機器](connect-logon.md)。
* 附加磁碟來儲存資料。 您可以附加空的磁碟和含有資料的磁碟。 如需相關指示，請參閱 [將資料磁碟連接至以傳統部署模型建立的 Windows 虛擬機器](attach-disk.md)。
