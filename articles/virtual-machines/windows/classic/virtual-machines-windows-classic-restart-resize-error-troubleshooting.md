---
title: VM 重新啟動或調整大小的問題 | Microsoft 文件
description: 針對在 Azure 中重新啟動或調整現有 Windows 虛擬機器大小的傳統部署問題進行疑難排解
services: virtual-machines-windows
documentationcenter: ''
author: Deland-Han
manager: felixwu
editor: ''
tags: top-support-issue
ms.assetid: aa854fff-c057-4b8e-ad77-e4dbc39648cc
ms.service: virtual-machines-windows
ms.topic: troubleshooting
ms.tgt_pltfrm: vm-windows
ms.workload: required
ms.date: 06/15/2018
ms.devlang: na
ms.author: delhan
ms.openlocfilehash: 400b20e474257650a22e04c89ddde581bf0552f4
ms.sourcegitcommit: e8f443ac09eaa6ef1d56a60cd6ac7d351d9271b9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/12/2018
ms.locfileid: "35772344"
---
# <a name="troubleshoot-classic-deployment-issues-with-restarting-or-resizing-an-existing-windows-virtual-machine-in-azure"></a>針對在 Azure 中重新啟動或調整現有 Windows 虛擬機器大小的傳統部署問題進行疑難排解
> [!div class="op_single_selector"]
> * [傳統](virtual-machines-windows-classic-restart-resize-error-troubleshooting.md)
> * [Resource Manager](../restart-resize-error-troubleshooting.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
> 
> 

當您嘗試啟動已停止的 Azure 虛擬機器 (VM)，或調整現有 Azure VM 的大小時，常會遇到的錯誤是配置失敗。 當叢集或區域沒有可用的資源或無法支援所要求的 VM 大小，就會產生此錯誤。

> [!IMPORTANT]
> Azure 建立和處理資源的部署模型有二種：[Resource Manager 和傳統](../../../azure-resource-manager/resource-manager-deployment-model.md)。  本文涵蓋之內容包括使用傳統部署模型。 Microsoft 建議讓大部分的新部署使用 Resource Manager 模式。
> 
> 

[!INCLUDE [support-disclaimer](../../../../includes/support-disclaimer.md)]

## <a name="collect-audit-logs"></a>收集稽核記錄檔
若要開始進行排解疑難，請收集稽核記錄，識別與問題相關的錯誤。

在 Azure 入口網站中，依序按一下 [瀏覽] > [虛擬機器] > 您的 Windows 虛擬機器 > [設定] > [稽核記錄檔]。

## <a name="issue-error-when-starting-a-stopped-vm"></a>問題：啟動已停止的 VM 時發生錯誤
您嘗試啟動已停止的 VM，但是發現配置失敗。

### <a name="cause"></a>原因
必須在架設雲端服務的原始叢集上嘗試提出啟動已停止的 VM 要求。 不過，叢集沒有足夠空間可完成要求。

### <a name="resolution"></a>解決方案
* 建立新的雲端服務，並將它和區域或以區域為基礎的虛擬網路相關聯，但不是和同質群組。
* 刪除已停止的 VM。
* 使用磁碟在新的雲端服務中重新建立 VM。
* 啟動重新建立的 VM。

如果您在嘗試建立新的雲端服務時收到錯誤，請稍後再試一次，或變更雲端服務的區域。

> [!IMPORTANT]
> 新的雲端服務將會有新的名稱和 VIP，因此您需要為所有將資訊用於現有雲端服務的相依性變更該資訊。
> 
> 

## <a name="issue-error-when-resizing-an-existing-vm"></a>問題：調整現有 VM 的大小時發生錯誤
您嘗試調整現有 VM 的大小，但是發現配置失敗。

### <a name="cause"></a>原因
必須在架設雲端服務的原始叢集上嘗試提出調整 VM 大小的要求。 不過，叢集不支援要求的 VM 大小。

### <a name="resolution"></a>解決方案
減少要求的 VM 大小，並再試一次調整大小要求。

* 按一下 [全部瀏覽]  >  [虛擬機器 (傳統)]  >  *您的虛擬機器*  >  [設定]  >  [大小]。 如需詳細步驟，請參閱 [調整虛擬機器的大小](https://msdn.microsoft.com/library/dn168976.aspx)。

如果您無法減少 VM 大小，請遵循下列步驟︰

* 建立新的雲端服務，確保不會連結至同質群組，以及未與連結至同質群組的虛擬網路相關聯。
* 在其中建立新的、較大的 VM。

您可以在相同的雲端服務中合併所有 VM。 如果現有的雲端服務和以區域為基礎的虛擬網路相關聯，您可以將新的雲端服務連接到現有的虛擬網路。

如果現有的雲端服務未和以區域為基礎的虛擬網路相關聯，您必須刪除現有雲端服務中的 VM，並從其磁碟在新的雲端服務中將其重新建立。 然而，請務必記得新的雲端服務將會有新的名稱和 VIP，因此您需要為所有目前將此資訊用於現有雲端服務的相依性更新該資訊。

## <a name="next-steps"></a>後續步驟
如果您在 Azure 中建立 Windows VM 時遇到問題，請參閱[針對在 Azure 中建立 Windows 虛擬機器的部署問題進行疑難排解](../troubleshoot-deployment-new-vm.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。

