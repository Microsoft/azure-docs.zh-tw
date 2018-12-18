---
title: 針對 Linux VM 部署-傳統進行疑難排解 | Microsoft Docs
description: 針對在 Azure 中建立新 Linux 虛擬機器的傳統部署問題進行疑難排解
services: virtual-machines-linux
documentationcenter: ''
author: JiangChen79
manager: felixwu
editor: ''
tags: top-support-issue
ms.assetid: c8a963fa-6b2a-4c7a-a1f4-7793adb02b19
ms.service: virtual-machines-linux
ms.workload: na
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: troubleshooting
ms.date: 09/06/2016
ms.author: cjiang
ms.openlocfilehash: 581fbaa477bd603fea5fdc0ef77c6ef7498b7897
ms.sourcegitcommit: be9a42d7b321304d9a33786ed8e2b9b972a5977e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/19/2018
ms.locfileid: "27965963"
---
# <a name="troubleshoot-classic-deployment-issues-with-creating-a-new-linux-virtual-machine-in-azure"></a>針對在 Azure 中建立新 Linux 虛擬機器的傳統部署問題進行疑難排解
[!INCLUDE [virtual-machines-troubleshoot-deployment-new-vm-selectors](../../../../includes/virtual-machines-linux-troubleshoot-deployment-new-vm-selectors-include.md)]

[!INCLUDE [virtual-machines-troubleshoot-deployment-new-vm-opening](../../../../includes/virtual-machines-troubleshoot-deployment-new-vm-opening-include.md)]

> [!IMPORTANT] 
> Azure 建立和處理資源的部署模型有二種： [Resource Manager 和傳統](../../../resource-manager-deployment-model.md)。 本文涵蓋之內容包括使用傳統部署模型。 Microsoft 建議讓大部分的新部署使用 Resource Manager 模式。 如需本文的 Resource Manager 版本，請參閱[這裡](../troubleshoot-deployment-new-vm.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。

[!INCLUDE [support-disclaimer](../../../../includes/support-disclaimer.md)]

## <a name="collect-audit-logs"></a>收集稽核記錄檔
若要開始進行排解疑難，請收集稽核記錄，識別與問題相關的錯誤。

在 Azure 入口網站中，依序按一下 [瀏覽] > [虛擬機器] > 您的 Windows 虛擬機器 > [設定] > [稽核記錄檔]。

[!INCLUDE [virtual-machines-troubleshoot-deployment-new-vm-issue1](../../../../includes/virtual-machines-troubleshoot-deployment-new-vm-issue1-include.md)]

[!INCLUDE [virtual-machines-linux-troubleshoot-deployment-new-vm-table](../../../../includes/virtual-machines-linux-troubleshoot-deployment-new-vm-table.md)]

**Y：** 如果作業系統是一般化的 Linux，且上傳和 (或) 擷取它時使用的是一般化設定，就不會有任何錯誤。 同樣地，如果作業系統是特殊化的 Linux，且上傳和 (或) 擷取它時使用的是特殊化設定，就不會有任何錯誤。

**上傳錯誤：**

**N<sup>1</sup>：** 如果作業系統是一般化的 Linux，但是上傳它時是以特殊化形式上傳，就會發生佈建逾時錯誤，因為 VM 會卡在佈建階段。

**N<sup>2</sup>：** 如果作業系統是特殊化的 Linux，但是上傳它時是以一般化形式上傳，就會發生佈建失敗錯誤，因為新 VM 是以原始的電腦名稱、使用者名稱和密碼執行。

**解決方案：**

若要解決這兩個錯誤，請使用與作業系統相同的設定 (一般化/特殊化) 來上傳原始 VHD (可從內部部署環境取得)。 若要以一般化上傳，請務必先執行 -deprovision。 如需詳細資訊，請參閱 [建立及上傳包含 Linux 作業系統的虛擬硬碟](create-upload-vhd-classic.md) 。

**擷取錯誤：**

**N<sup>3</sup>：** 如果作業系統是一般化的 Linux，但是擷取它時是以特殊化形式擷取，就會發生佈建逾時錯誤，因為原始 VM 會因被標示為一般化而無法供使用。

**N<sup>4</sup>：** 如果作業系統是特殊化的 Linux，但是擷取它時是以一般化形式擷取，就會發生佈建失敗錯誤，因為新 VM 是以原始的電腦名稱、使用者名稱和密碼執行。 此外，原始 VM 會因被標示為特殊化而無法供使用。

**解決方案：**

若要解決這兩個錯誤，請從入口網站中刪除目前的映像，然後使用與作業系統相同的設定 (一般化/特殊化) [從目前的 VHD 重新擷取映像](capture-image-classic.md) 。

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
如果您在啟動已停止的 Linux VM，或重新調整 Azure 中現有 Linux VM 的大小時遇到問題，請參閱 [針對在 Azure 中重新啟動或調整現有 Linux 虛擬機器大小的傳統部署問題進行疑難排解](restart-resize-error-troubleshooting.md)。

