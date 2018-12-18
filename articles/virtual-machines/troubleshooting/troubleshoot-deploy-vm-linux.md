---
title: 針對 Azure 中的 Linux 虛擬機器部署問題進行疑難排解 | Microsoft Docs
description: 針對 Azure Resource Manager 部署模型中的 Linux 虛擬機器部署問題進行疑難排解。
services: virtual-machines-windows
documentationcenter: ''
author: genlin
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: 4e383427-4aff-4bf3-a0f4-dbff5c6f0c81
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: troubleshooting
ms.date: 05/11/2018
ms.author: genli
ms.openlocfilehash: b14b09a919757ad2fc64d69e1749ebbe4d9dda5c
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47411717"
---
# <a name="troubleshoot-deploying-linux-virtual-machine-issues-in-azure"></a>針對 Azure 中的 Linux 虛擬機器部署問題進行疑難排解

若要針對 Azure 中的虛擬機器 (VM) 部署問題進行疑難排解，請檢閱[常見問題](#top-issues)以了解常見的失敗和解決方式。

如果您在本文中有任何需要協助的地方，您可以連絡 [MSDN Azure 和 Stack Overflow 論壇](https://azure.microsoft.com/support/forums/)上的 Azure 專家。 或者，您可以提出 Azure 支援事件。 請移至 [Azure 支援網站](https://azure.microsoft.com/support/options/)，然後選取 [取得支援]。

## <a name="top-issues"></a>常見問題
[!INCLUDE [virtual-machines-linux-troubleshoot-deploy-vm-top](../../../includes/virtual-machines-linux-troubleshoot-deploy-vm-top.md)]

## <a name="the-cluster-cannot-support-the-requested-vm-size"></a>叢集無法支援要求的 VM 大小
<properties
supportTopicIds="123456789"
resourceTags="windows"
productPesIds="1234, 5678"
/>
- 以較小的 VM 大小重試要求。
- 如果無法變更要求的 VM 的大小︰
    - 停止可用性設定組中的所有 VM。 按一下 [資源群組] > 您的資源群組 > [資源] > 您的可用性設定組 > [虛擬機器] > 您的虛擬機器 > [停止]。
    - 在所有 VM 都停止後，建立所需大小的 VM。
    - 先啟動新的 VM，然後選取每個已停止的 VM 並按一下 [啟動]。


## <a name="the-cluster-does-not-have-free-resources"></a>叢集沒有可用的資源
<properties
supportTopicIds="123456789"
resourceTags="windows"
productPesIds="1234, 5678"
/>
- 稍後再重試要求。
- 如果新的 VM 可以屬於不同的可用性設定組
    - 在不同的可用性設定組 (位於相同區域) 中建立 VM。
    - 將新的 VM 加入相同的虛擬網路。

## <a name="how-do-i-activate-my-monthly-credit-for-visual-studio-enterprise-bizspark"></a>我要如何啟用我的 Visual Studio Enterprise (BizSpark) 每月點數

若要啟用您的每月點數，請參閱這篇[文章](https://azure.microsoft.com/offers/ms-azr-0064p/)。

## <a name="why-can-i-not-install-the-gpu-driver-for-an-ubuntu-nv-vm"></a>為什麼我可以不安裝適用於 Ubuntu NV VM 的 GPU 驅動程式？

目前，Linux GPU 支援僅適用於執行 Ubuntu Server 16.04 LTS 的 Azure NC VM。 如需詳細資訊，請參閱[為執行 Linux 的 N 系列 VM 設定 GPU 驅動程式](../linux/n-series-driver-setup.md)。

## <a name="my-drivers-are-missing-for-my-linux-n-series-vm"></a>我的 Linux N 系列 VM 遺失驅動程式

Linux 型 VM 的驅動程式位於[這裡](../linux/n-series-driver-setup.md)。 

## <a name="i-cant-find-a-gpu-instance-within-my-n-series-vm"></a>在我的 N 系列 VM 內找不到 GPU 執行個體

若要利用 Azure N 系列 VM (執行 Windows Server 2016 或 Windows Server 2012 R2) 的 GPU 功能，您必須在部署之後於每個 VM 上安裝 NVIDIA 圖形驅動程式。 您可以取得 [Windows VM](../windows/n-series-driver-setup.md) 和 [Linux VM](../linux/n-series-driver-setup.md) 的驅動程式設定資訊。

## <a name="is-n-series-vms-available-in-my-region"></a>我的區域是否有提供 N 系列 VM？

您可以從[依區域提供的產品表](https://azure.microsoft.com/regions/services)查看是否有提供該產品，以及從[這裡](https://azure.microsoft.com/pricing/details/virtual-machines/series/#n-series)查看定價。

## <a name="i-am-not-able-to-see-vm-size-family-that-i-want-when-resizing-my-vm"></a>我在調整 VM 大小時，看不到所要的 VM 大小系列。

執行 VM 時，會將 VM 部署到實體伺服器。 Azure 區域中的實體伺服器會群組成共同的實體硬體叢集。 調整 VM 大小時如果需要將 VM 移到不同的硬體叢集，將會依部署 VM 時所用的部署模型而有不同的做法。

- 如果 VM 是以「傳統」部署模型部署的，就必須移除雲端服務部署後再重新部署，才能將 VM 變更成另一個大小系列中的大小。

- 如果 VM 是以 Resource Manager 部署模型部署的，您就必須先停止可用性設定組中的所有 VM，才能變更該可用性設定組中任何 VM 的大小。

## <a name="the-listed-vm-size-is-not-supported-while-deploying-in-availability-set"></a>部署在可用性設定組中時，不支援所列的 VM 大小。

請選擇可用性設定組叢集上支援的大小。 建議您在建立可用性設定組時，選擇您認為所需的最大 VM 大小，並且以它作為您的第一個可用性設定組部署項目。

## <a name="what-linux-distributionsversions-are-supported-on-azure"></a>Azure 上支援哪些 Linux 散發套件/版本？

您可以在[經 Azure 背書的散發套件](../linux/endorsed-distros.md)上找到 Linux 的清單。

## <a name="can-i-add-an-existing-classic-vm-to-an-availability-set"></a>我是否可以將現有的傳統 VM 新增到可用性設定組？

是。 您可以將現有的傳統 VM 新增到新的或現有的「可用性設定組」。 如需詳細資訊，請參閱[將現有的虛擬機器新增至可用性設定組](../windows/classic/configure-availability-classic.md#addmachine)。


## <a name="next-steps"></a>後續步驟
如果您在本文中有任何需要協助的地方，您可以連絡 [MSDN Azure 和 Stack Overflow 論壇](https://azure.microsoft.com/support/forums/)上的 Azure 專家。

或者，您可以提出 Azure 支援事件。 請移至 [Azure 支援網站](https://azure.microsoft.com/support/options/)，然後選取 [取得支援]。
