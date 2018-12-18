---
title: 在 Azure DevTest Labs 中調整實驗室的配額和限制 | Microsoft Docs
description: 了解如何在 Azure DevTest Labs 中調整實驗室
services: devtest-lab,virtual-machines,lab-services
documentationcenter: na
author: spelluru
manager: femila
ms.assetid: ae624155-9181-45fa-bd2b-1983339b7e0e
ms.service: lab-services
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/17/2018
ms.author: spelluru
ms.openlocfilehash: 5a179c0c6b777ee6b2afdd0f2e9267d247665d8d
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38675960"
---
# <a name="scale-quotas-and-limits-in-devtest-labs"></a>在 DevTest Labs 中調整配額和限制
當您在 DevTest Labs 中工作時，您可能會發現某些 Azure 資源有特定預設限制，會影響 DevTest Labs 服務。 這些限制稱為**配額**。

> [!NOTE]
> DevTest Labs 服務不會造成任何配額。 您可能遇到的任何配額是整體 Azure 訂用帳戶的預設條件約束。

您可以使用每個 Azure 資源直到您達到其配額。 每個訂用帳戶都有個別配額，並且會追蹤每個訂用帳戶的使用量。

例如，每個訂用帳戶都有預設配額 20 個核心。 因此，如果您在具有四個核心的實驗室中建立 VM，則您只能建立五個 VM。 

[Azure 訂用帳戶和服務限制](https://docs.microsoft.com/azure/azure-subscription-service-limits)列出一些 Azure 資源最常見的配額。 在實驗室中最常使用的資源 (且您可能會遇到其配額) 包括 VM 核心、公用 IP 位址、網路介面、受控磁碟、RBAC 角色指派，以及 ExpressRoute 線路。

## <a name="view-your-usage-and-quotas"></a>檢視使用量和配額
下列步驟將示範如何檢視訂用帳戶中 Azure 資源的目前配額，以及查看您使用的每個配額百分比。

1. 登入 [Azure 入口網站](http://go.microsoft.com/fwlink/p/?LinkID=525040)。
1. 選取 [更多服務]，然後從清單中選取 [計費]。
1. 在 [計費] 刀鋒視窗中選取訂用帳戶。
4. 選取 [使用量 + 配額]。

   ![使用量和配額按鈕](./media/devtest-lab-scale-lab/devtestlab-usage-and-quotas.png)

   [使用量 + 配額] 刀鋒視窗隨即出現，列出該訂用帳戶中可用的不同資源，以及每個資源使用的配額百分比。

   ![配額和使用量](./media/devtest-lab-scale-lab/devtestlab-view-quotas.png)

## <a name="requesting-more-resources-in-your-subscription"></a>要求訂用帳戶中的更多資源
如果達到配額上限，訂用帳戶中資源的預設限制可以增加到最大限制，如 [Azure 訂用帳戶和服務限制](https://docs.microsoft.com/azure/azure-subscription-service-limits)中所述。

這些步驟為您示範如何透過 [Azure 入口網站](http://go.microsoft.com/fwlink/p/?LinkID=525040)要求配額增加。

1. 選取 [更多服務]，選取 [計費]，然後選取 [使用量 + 配額]。
1. 在 [使用量 + 配額] 刀鋒視窗中，選取 [要求增加] 按鈕。

   ![要求增加按鈕](./media/devtest-lab-scale-lab/devtestlab-request-increase.png)

1. 若要完成要求並且提交，請在 [新增支援要求] 表單的全部三個索引標籤上填寫必要資訊。

   ![要求增加表單](./media/devtest-lab-scale-lab/devtestlab-support-form.png)

[了解 Azure 限制和增加](https://azure.microsoft.com/blog/azure-limits-quotas-increase-requests/)提供連絡 Azure 支援以要求配額增加的詳細資訊。



[!INCLUDE [devtest-lab-try-it-out](../../includes/devtest-lab-try-it-out.md)]

### <a name="next-steps"></a>後續步驟
* 瀏覽 [DevTest Labs Azure Resource Manager 快速入門範本資源庫 (英文)](https://github.com/Azure/azure-devtestlab/tree/master/Samples)。
