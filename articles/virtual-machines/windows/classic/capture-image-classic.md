---
title: 擷取 Azure Windows VM 的映像 | Microsoft Docs
description: 擷取以傳統部署模型建立之 Azure Windows 虛擬機器的映像。
services: virtual-machines-windows
documentationcenter: ''
author: cynthn
manager: jeconnoc
editor: tysonn
tags: azure-service-management
ROBOTS: NOINDEX
ms.assetid: a5986eac-4cf3-40bd-9b79-7c811806b880
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.devlang: na
ms.topic: article
ms.date: 05/30/2017
ms.author: cynthn
ms.openlocfilehash: 24366d3adf63fb6fafb8afa9f1a9fb359e88d8ed
ms.sourcegitcommit: 5b2ac9e6d8539c11ab0891b686b8afa12441a8f3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/06/2018
ms.locfileid: "30914050"
---
# <a name="capture-an-image-of-an-azure-windows-virtual-machine-created-with-the-classic-deployment-model"></a>擷取以傳統部署模型建立之 Azure Windows 虛擬機器的映像
> [!IMPORTANT]
> Azure 建立和處理資源的部署模型有二種： [Resource Manager 和傳統](../../../resource-manager-deployment-model.md)。 本文涵蓋之內容包括使用傳統部署模型。 Microsoft 建議讓大部分的新部署使用 Resource Manager 模式。 如需 Resource Manager 模型的資訊，請參閱[在 Azure 中擷取一般化 VM 的受控映像](../capture-image-resource.md)。
> [!INCLUDE [virtual-machines-common-classic-createportal](../../../../includes/virtual-machines-classic-portal.md)]

本文說明如何擷取執行 Windows 的 Azure 虛擬機器，以便用它作為映像來建立其他虛擬機器。 此映像包括作業系統磁碟和任何連結至虛擬機器的資料磁碟。 它不包含網路組態，因此，當您建立其他使用此映像的虛擬機器時，將需要設定網路組態。

Azure 會將映像儲存於 [VM 映像 (傳統)] 下方，當您檢視所有 Azure 服務時會列出的這個 [計算] 服務。 這個位置與您已上傳映像的任何儲存位置相同。 如需映像的詳細資訊，請參閱 [有關虛擬機器的映像](about-images.md?toc=%2fazure%2fvirtual-machines%2fWindows%2fclassic%2ftoc.json)。

## <a name="before-you-begin"></a>開始之前
這些步驟假設您已建立 Azure 虛擬機器且設定好作業系統，包括連接任何資料磁碟。 如果您尚未完成這個動作，請參閱下列文章，以取得建立和準備虛擬機器的相關資訊：

* [從映像建立虛擬機器](createportal.md)
* [如何將資料磁碟連結至虛擬機器](attach-disk.md)
* 請確定 Sysprep 支援伺服器角色。 如需詳細資訊，請參閱 [Sysprep Support for Server Roles (伺服器角色的 Sysprep 支援)](https://msdn.microsoft.com/windows/hardware/commercialize/manufacture/desktop/sysprep-support-for-server-roles)。

> [!WARNING]
> 此程序會在擷取原始虛擬機器後將其刪除。
>
>

擷取 Azure 虛擬機器映像之前，建議先備份目標虛擬機器。 Azure 虛擬機器可使用 Azure 備份進行備份。 如需詳細資訊，請參閱 [備份 Azure 虛擬機器](../../../backup/backup-azure-arm-vms.md)。 來自認證合作夥伴的其他解決方案也可供使用。 若要了解目前可用的項目，請搜尋 Azure Marketplace。

## <a name="capture-the-virtual-machine"></a>擷取虛擬機器
1. 在 [Azure 入口網站](http://portal.azure.com)中，**連接**到虛擬機器。 如需指示，請參閱[如何登入執行 Windows Server 的虛擬機器][How to sign in to a virtual machine running Windows Server]。
2. 以系統管理員身分開啟 [命令提示字元] 視窗。
3. 切換至 `%windir%\system32\sysprep`目錄，然後執行 sysprep.exe。
4. [系統準備工具]  對話方塊隨即出現。 執行下列動作：

   * 在 [系統清理動作] 中選取 [Enter System Out-of-Box Experience (OOBE)]，並確認 [一般化] 已勾選。 如需 Sysprep 的詳細用法，請參閱[如何使用 Sysprep：簡介][How to Use Sysprep: An Introduction]。
   * 在 [關機選項] 中選取 [關機]。
   * 按一下 [SERVICEPRINCIPAL] 。

   ![執行 Sysprep](./media/capture-image/SysprepGeneral.png)
5. Sysprep 會將虛擬機器關機，並會在 Azure 入口網站中，將虛擬機器的狀態變更為 [ **已停止**]。
6. 在 Azure 入口網站中，按一下 [虛擬機器 (傳統)]，然後選取您想要擷取的虛擬機器。 [VM 映像 (傳統)] 群組會在您檢視 [所有服務] 時列於 [計算] 下方。

7. 按一下命令列上的 [擷取] 。

   ![擷取虛擬機器](./media/capture-image/CaptureVM.png)

   [擷取虛擬機器]  對話方塊隨即出現。

8. 在 [映像名稱] 中，輸入新映像的名稱。 在 [映像標籤] 中，輸入新映像的標籤。

9. 按一下 [我已在虛擬機器上執行 Sysprep]。 此核取方塊指的是在步驟 3-5 中使用 Sysprep 的動作。 將 Windows Server 映像新增到自訂映像組合之前，「必須」先執行 Sysprep，將映像一般化。

10. 完成擷取時，新映像會在 **[Marketplace]** \(位於 [計算] 、[VM 映像 (傳統)] 容器) 中變成可供使用狀態。

    ![成功擷取映像](./media/capture-image/VMCapturedImageAvailable.png)

## <a name="next-steps"></a>後續步驟
映像已準備好用來建立虛擬機器。 若要這樣做，您將建立虛擬機器，方法是選取服務功能表底部的 [所有服務] 功能表項目，然後選取 [計算] 群組中的 [VM 映像 (傳統)]。 如需指示，請參閱 [從映像建立虛擬機器](createportal.md)。

[How to sign in to a virtual machine running Windows Server]:connect-logon.md
[How to Use Sysprep: An Introduction]: http://technet.microsoft.com/library/bb457073.aspx
[Run Sysprep.exe]: ./media/virtual-machines-capture-image-windows-server/SysprepCommand.png
[Enter Sysprep.exe options]: ./media/capture-image/SysprepGeneral.png
[The virtual machine is stopped]: ./media/virtual-machines-capture-image-windows-server/SysprepStopped.png
[Capture an image of the virtual machine]: ./media/capture-image/CaptureVM.png
[Enter the image name]: ./media/virtual-machines-capture-image-windows-server/Capture.png
[Image capture successful]: ./media/virtual-machines-capture-image-windows-server/CaptureSuccess.png
[Use the captured image]: ./media/virtual-machines-capture-image-windows-server/MyImagesWindows.png
