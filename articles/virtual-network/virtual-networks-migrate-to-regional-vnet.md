---
title: 將 Azure 虛擬網路 (傳統) 從同質群組移轉至區域 | Microsoft Docs
description: 了解如何將虛擬網路 (傳統) 從同質群組移轉至區域。
services: virtual-network
documentationcenter: na
author: genlin
manager: cshepard
editor: ''
tags: azure-service-management
ms.assetid: 84febcb9-bb8b-4e79-ab91-865ad9de41cb
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/15/2016
ms.author: genli
ms.openlocfilehash: 1fca7f6165998b95254f841638cf8bcbc1fb352d
ms.sourcegitcommit: fa493b66552af11260db48d89e3ddfcdcb5e3152
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/23/2018
ms.locfileid: "31792149"
---
# <a name="migrate-a-virtual-network-classic-from-an-affinity-group-to-a-region"></a>將虛擬網路 (傳統) 從同質群組移轉至區域

> [!IMPORTANT]
> Azure 建立和處理資源的部署模型有二種： [資源管理員和傳統](../resource-manager-deployment-model.md?toc=%2fazure%2fvirtual-network%2ftoc.json)。 本文涵蓋之內容包括使用傳統部署模型。 Microsoft 建議讓大部分的新部署都使用 Resource Manager 部署模型。

同質群組確保在相同同質群組內建立的資源均由伺服器所實際主控，這些伺服器彼此鄰近，可讓這些資源進行更快速的通訊。 在過去，建立虛擬網路 (傳統) 需要同質群組。 當時，管理虛擬網路 (傳統) 的網路管理員服務僅能在一組實體伺服器或縮放單位內工作。 架構改進則增加區域的網路管理範圍。

由於這些架構的改進，因而導致不再建議使用同質群組，或不再需要虛擬網路 (傳統)。 針對虛擬網路 (傳統) 使用同質群組，會由區域取代。 與區域相關聯的虛擬網路 (傳統) 稱為地區虛擬網路。

我們建議您在一般情況下不要使用同質群組。 除了虛擬網路的需求，過去也必須使用同質群組來確保資源，例如放置在彼此鄰近位置的計算 (傳統) 和儲存體 (傳統)。 不過，使用目前的 Azure 網路架構時，這些放置需求不再必要。

> [!IMPORTANT]
> 雖然技術上來說仍然可以建立與同質群組相關聯的虛擬網路，但是這樣做沒有令人信服的理由。 許多虛擬網路功能 (例如網路安全性群組) 只有在使用區域虛擬網路時才能使用，且不適用於與同質群組相關聯的虛擬網路。
> 
> 

## <a name="edit-the-network-configuration-file"></a>編輯網路組態檔

1. 匯出網路組態檔。 若要了解如何使用 PowerShell 或 Azure 命令列介面 (CLI) 1.0 匯出網路組態檔，請參閱[使用網路組態檔設定虛擬網路](virtual-networks-using-network-configuration-file.md#export)。
2. 編輯網路組態檔，使用 **Location** 取代 **AffinityGroup**。 您需針對 **Location** 指定 Azure [區域](https://azure.microsoft.com/regions)。
   
   > [!NOTE]
   > **Location** 是您所指定與您虛擬網路 (傳統) 相關聯之同質群組的區域。 例如，如果您的虛擬網路 (傳統) 與位於美國西部的同質群組相關聯，則當移轉時，您的 **Location** 必須指向 West US (美國西部)。 
   > 
   > 
   
    在網路組態檔中編輯下列幾行，並取代為您需要的值： 
   
    **舊值：**\<VirtualNetworkSitename="VNetUSWest" AffinityGroup="VNetDemoAG"\> 
   
    **新值：**\<VirtualNetworkSitename="VNetUSWest" Location="West US"\>
3. 儲存您的變更並網路組態 [匯入](virtual-networks-using-network-configuration-file.md#import) 至 Azure。

> [!NOTE]
> 這項移轉不會對您的服務造成任何停機時間。
> 
> 

## <a name="what-to-do-if-you-have-a-vm-classic-in-an-affinity-group"></a>如果您在同質群組中擁有 VM (傳統) 時該怎麼辦
目前在同質群組中的 VM (傳統) 不需要從同質群組中移除。 一旦部署 VM，其會部署至單一的縮放單位。 同質群組可針對新的 VM 部署限制可用的 VM 大小集合，但任何已部署的現有 VM 部署，已在 VM 所部署的位置限制縮放單位中可用的 VM 大小集合。 由於已經將 VM 部署至縮放單位，因此，從同質群組中移除 VM 在該 VM 上並無任何作用。
