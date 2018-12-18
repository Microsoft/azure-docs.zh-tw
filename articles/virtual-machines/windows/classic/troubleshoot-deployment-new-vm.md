---
title: 針對 Windows VM 部署-傳統進行疑難排解 | Microsoft Docs
description: 針對在 Azure 中建立新 Windows 虛擬機器的傳統部署問題進行疑難排解
services: virtual-machines-windows
documentationcenter: ''
author: JiangChen79
manager: felixwu
editor: ''
tags: top-support-issue
ms.assetid: 9f01d237-ba39-4c32-b72d-18f5f505d43a
ms.service: virtual-machines-windows
ms.workload: na
ms.tgt_pltfrm: vm-windows
ms.devlang: na
ms.topic: troubleshooting
ms.date: 11/03/2016
ms.author: cjiang
ms.openlocfilehash: 5a1ad33bd8c287910458d26a96858e85c689fb16
ms.sourcegitcommit: adf6a4c89364394931c1d29e4057a50799c90fc0
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/09/2017
ms.locfileid: "24056634"
---
# <a name="troubleshoot-classic-deployment-issues-with-creating-a-new-windows-virtual-machine-in-azure"></a>針對在 Azure 中建立新 Windows 虛擬機器的傳統部署問題進行疑難排解
[!INCLUDE [virtual-machines-troubleshoot-deployment-new-vm-selectors](../../../../includes/virtual-machines-windows-troubleshoot-deployment-new-vm-selectors-include.md)]

[!INCLUDE [virtual-machines-troubleshoot-deployment-new-vm-opening](../../../../includes/virtual-machines-troubleshoot-deployment-new-vm-opening-include.md)]

> [!IMPORTANT] 
> Azure 建立和處理資源的部署模型有二種： [Resource Manager 和傳統](../../../resource-manager-deployment-model.md)。 本文涵蓋之內容包括使用傳統部署模型。 Microsoft 建議讓大部分的新部署使用 Resource Manager 模式。 如需本文的 Resource Manager 版本，請參閱[這裡](../../virtual-machines-windows-troubleshoot-deployment-new-vm.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。
> [!INCLUDE [virtual-machines-common-classic-createportal](../../../../includes/virtual-machines-classic-portal.md)]

[!INCLUDE [support-disclaimer](../../../../includes/support-disclaimer.md)]

## <a name="collect-audit-logs"></a>收集稽核記錄檔
若要開始進行排解疑難，請收集稽核記錄，識別與問題相關的錯誤。

在 Azure 入口網站中，依序按一下 [瀏覽] > [虛擬機器] > 您的 Windows 虛擬機器 > [設定] > [稽核記錄檔]。

[!INCLUDE [virtual-machines-troubleshoot-deployment-new-vm-issue1](../../../../includes/virtual-machines-troubleshoot-deployment-new-vm-issue1-include.md)]

[!INCLUDE [virtual-machines-windows-troubleshoot-deployment-new-vm-table](../../../../includes/virtual-machines-windows-troubleshoot-deployment-new-vm-table.md)]

**Y：** 如果 OS 是一般化的 Windows，且上傳和 (或) 擷取它時使用的是一般化設定，就不會有任何錯誤。 同樣地，如果 OS 是特殊化的 Windows，且上傳和 (或) 擷取它時使用的是特殊化設定，就不會有任何錯誤。

**上傳錯誤：**

**N<sup>1</sup>：** 如果作業系統是一般化的 Windows，但是以特殊化被上傳，就會發生佈建逾時錯誤，VM 會卡在 OOBE 畫面。

**N<sup>2</sup>：** 如果作業系統是特殊化的 Windows，但是以一般化被上傳，就會發生佈建失敗錯誤，VM 會卡在 OOBE 畫面，因為新 VM 是以原始的電腦名稱、使用者名稱和密碼執行。

**解決方案：**

若要解決這兩個錯誤，請上傳原始 VHD、可用的內部部署、以及與該 OS (一般化/特殊化) 相同的設定。 若要以一般化上傳，請務必先執行 sysprep。 如需詳細資訊，請參閱 [建立 Windows Server VHD 並上傳至 Azure](createupload-vhd.md) 。

**擷取錯誤：**

**N<sup>3</sup>：** 如果作業系統是一般化的 Windows，但是以特殊化被擷取，就會發生佈建逾時錯誤，因為 VM 被標示為一般化而無法加以使用。

**N<sup>4</sup>：** 如果作業系統是特殊化的 Windows，但是以一般化的方式擷取，就會發生佈建失敗錯誤，因為新 VM 是以原始的電腦名稱、使用者名稱和密碼執行。 此外，原始 VM 會因被標示為特殊化而無法供使用。

**解決方案：**

若要解決這兩個錯誤，請從入口網站中刪除目前的映像，然後使用與作業系統相同的設定 (一般化/特殊化) [從目前的 VHD 重新擷取映像](capture-image.md) 。

## <a name="issue-custom-gallery-marketplace-image-allocation-failure"></a>問題︰自訂/資源庫/Marketplace 映像；配置失敗
當新的 VM 要求被傳送到沒有可用空間可處理要求、或不支援所要求的 VM 大小的叢集，便會發生此錯誤。 在相同的雲端服務中不可混合不同系列的 VM。 因此，如果您想要建立和您的雲端服務可支援大小不同的新 VM，計算要求將會失敗。

您可能會遇到因兩種情況造成的錯誤，取決於您用於建立新 VM 的雲端服務的條件約束。

**原因 1：** 雲端服務已釘選到特定叢集，或是連結到同質群組，因而釘選到所設計的特定叢集。 因此，系統會在裝載現有資源的相同叢集中，嘗試執行該同質群組中的新計算資源要求。 不過，同一叢集可能不支援要求的 VM 大小，或者可用空間不足，導致配置錯誤。 無論新的資源是透過新的雲端服務，或是透過現有的雲端服務來建立，都是如此。

**解決方式 1：**

* 建立新的雲端服務，並將它和區域或以區域為基礎的虛擬網路相關聯。
* 在新的雲端服務中建立新 VM。
  如果您在嘗試建立新的雲端服務時收到錯誤，請稍後再試一次，或變更雲端服務的區域。

> [!IMPORTANT]
> 如果您嘗試在現有的雲端服務中建立新的 VM，但無法建立，而您又必須為新的 VM 建立新的雲端服務，則可以選擇合併相同雲端服務中的所有 VM。 若要這樣做，請刪除現有雲端服務中的 VM，然後從它們位於新雲端服務中的磁碟重新擷取它們。 然而，請務必記得新的雲端服務將會有新的名稱和 VIP，因此您需要為所有目前將此資訊用於現有雲端服務的相依性更新該資訊。
> 
> 

**原因 2：** 雲端服務已經與連結到同質群組的虛擬網路關聯，因而釘選到所設計的特定叢集。 因此，系統會在裝載現有資源的相同叢集中，嘗試執行該同質群組中的所有新計算資源要求。 不過，同一叢集可能不支援要求的 VM 大小，或者可用空間不足，導致配置錯誤。 無論新的資源是透過新的雲端服務，或是透過現有的雲端服務來建立，都是如此。

**解決方式 2：**

* 建立新的區域虛擬網路。
* 在新的虛擬網路中建立新 VM。
* [連接您現有的虛擬網路](https://azure.microsoft.com/blog/vnet-to-vnet-connecting-virtual-networks-in-azure-across-different-regions/) 到新的虛擬網路。 深入了解 [區域虛擬網路](https://azure.microsoft.com/blog/2014/05/14/regional-virtual-networks/)。 此外，也可以 [將同質群組式虛擬網路移轉至區域虛擬網路](https://azure.microsoft.com/blog/2014/11/26/migrating-existing-services-to-regional-scope/)，然後建立新 VM。

## <a name="next-steps"></a>後續步驟
如果您在啟動已停止的 Windows VM，或重新調整 Azure 中現有的 Windows VM 大小時遇到問題，請參閱 [Troubleshoot classic deployment issues with restarting or resizing an existing Windows Virtual Machine in Azure (針對在 Azure 中重新啟動或調整現有 Windows 虛擬機器大小的傳統部署問題進行疑難排解)](virtual-machines-windows-classic-restart-resize-error-troubleshooting.md)。

