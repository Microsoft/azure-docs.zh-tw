---
title: 管理 Machine Learning 工作區 | Microsoft Docs
description: 管理 Azure 機器學習工作區的存取權，並部署和管理 ML API Web 服務
services: machine-learning
documentationcenter: ''
author: heatherbshapiro
ms.author: hshapiro
manager: hjerez
editor: cgronlun
ms.assetid: daf3d413-7a77-4beb-9a7a-6b4bdf717719
ms.service: machine-learning
ms.component: studio
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/27/2017
ms.openlocfilehash: 6e1febfff211dd92f8990d1880562e64382de5f0
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47095441"
---
# <a name="manage-an-azure-machine-learning-workspace"></a>管理 Azure Machine Learning 工作區

> [!NOTE]
> 如需在 Machine Learning Web 服務入口網站中管理 Web 服務的相關資訊，請參閱[使用 Azure Machine Learning Web 服務入口網站管理 Web 服務](manage-new-webservice.md)。
> 
> 

您可以在 Azure 入口網站中管理 Machine Learning 工作區。

[!INCLUDE [machine-learning-free-trial](../../../includes/machine-learning-free-trial.md)]

## <a name="use-the-azure-portal"></a>使用 Azure 入口網站

若要在 Azure 入口網站中管理工作區︰

1. 使用 Azure 訂用帳戶系統管理員帳戶登入 [Azure 入口網站](https://portal.azure.com/)。
2. 在頁面頂端的搜尋方塊中輸入「Machine Learning 工作區」，然後選取 [Machine Learning 工作區]。
3. 按一下您想要管理的工作區。

除了標準的資源管理資訊和可用的選項，您還可以︰

- 檢視**屬性** - 此頁面會顯示工作區和資源的資訊，而且您可以變更這個工作區所連線的訂用帳戶和資源群組。
- **重新同步儲存體金鑰** - 工作區會保有儲存體帳戶的金鑰。 如果儲存體帳戶變更了金鑰，則您可以按一下 [重新同步金鑰] 來與工作區同步處理金鑰。

若要管理與此工作區相關聯的 Web 服務，請使用 Machine Learning Web 服務入口網站。 如需詳細資訊，請參閱[使用 Azure Machine Learning Web 服務入口網站管理 Web 服務](manage-new-webservice.md)。

> [!NOTE]
> 若要部署或管理新的 Web 服務，您必須獲得下列角色的指派：要部署 Web 服務之訂用帳戶上的參與者或系統管理員角色。 如果您邀請另一位使用者到 Machine Learning 工作區，就必須為他們指派訂用帳戶上的參與者或管理員角色，然後他們才能部署或管理 Web 服務。 
> 
>如需如何設定存取權限的詳細資訊，請參閱[使用 RBAC 和 Azure 入口網站來管理存取權](../../role-based-access-control/role-assignments-portal.md)。

## <a name="next-steps"></a>後續步驟
* 深入了解[使用Azure Resource Manager 範本部署 Machine Learning](deploy-with-resource-manager-template.md)。 