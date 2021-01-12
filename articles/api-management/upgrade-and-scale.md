---
title: 升級和調整 Azure API 管理執行個體的規模 | Microsoft Docs
description: 本主題描述如何升級和調整 Azure API 管理執行個體的規模。
services: api-management
documentationcenter: ''
author: mikebudzynski
manager: anneta
editor: ''
ms.service: api-management
ms.workload: integration
ms.topic: article
ms.date: 04/20/2020
ms.author: apimpm
ms.openlocfilehash: bd36434bbfe435a53567c46728610627f99f987f
ms.sourcegitcommit: 02b1179dff399c1aa3210b5b73bf805791d45ca2
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/12/2021
ms.locfileid: "98127782"
---
# <a name="upgrade-and-scale-an-azure-api-management-instance"></a>升級和調整 Azure API 管理執行個體的規模  

客戶可以藉由新增和移除單位來調整 Azure API 管理實例。 **單位** 由專用的 Azure 資源組成，具有一定的承載容量 (以每月 API 呼叫次數表示)。 此數字不代表呼叫限制，而不是允許粗略產能規劃的最大輸送量值。 受到一些因素的影響，實際的輸送量和延遲會大不相同，例如並行連線數和速率、設定的原則種類和數目、要求和回應的大小，以及後端延遲。

每個單位的容量和價格取決於該單位所在的 **階層**。 有四層供您選擇：**開發人員**、**基本**、**標準**、**進階**。 如果您需要為階層內的服務增加容量，請新增單位。 如果您的 API 管理實例中目前選取的階層不允許增加更多單位，則您需要升級至較高層級的層級。

每個單位的價格和可用功能 (例如，多重區域部署) 取決於您為 API 管理實例所選擇的層級。 [定價詳細資料](https://azure.microsoft.com/pricing/details/api-management/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio)一文說明每一層的每單位價格和功能。 

>[!NOTE]
>[定價詳細資料](https://azure.microsoft.com/pricing/details/api-management/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio)一文顯示每一層單位容量的大約數字。 若要取得更精確的數字，則需要查看 API 的實際情節。 請參閱 [Azure API 管理執行個體的容量](api-management-capacity.md)一文。

## <a name="prerequisites"></a>Prerequisites

若要依照本文中的步驟進行，您必須：

+ 擁有有效的 Azure 訂用帳戶。

    [!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

+ 擁有 API 管理實例。 如需詳細資訊，請參閱[建立 Azure API 管理執行個體](get-started-create-service-instance.md)。

+ 了解 [Azure API 管理執行個體的容量](api-management-capacity.md)的概念。

[!INCLUDE [premium-dev-standard-basic.md](../../includes/api-management-availability-premium-dev-standard-basic.md)]

## <a name="upgrade-and-scale"></a>升級和調整規模  

您可以選擇四個層級： **Developer**、 **Basic**、  **Standard** 和 **Premium**。 **開發人員** 層應該用來評估服務，不適用於生產環境。 **開發人員** 層沒有 SLA，您無法調整此階層的規模 (新增/移除單位)。 

**基本**、 **標準** 和高階 **是具有** SLA 且可調整規模的生產層。 **基本** 層是具有 SLA 的最便宜層，且最多可相應增加為兩個單位，**標準** 層可以調整為最多四個單位。 可新增至 **進階** 層的單位數沒有限制。

**進階** 層可讓您將單一 Azure API 管理執行個體分散到您想要的 Azure 區域，而且數目不限。 最初建立 Azure API 管理服務時，執行個體只包含一個單位，而且位於單一 Azure 區域中。 初始區域會指定為 **主要** 區域。 您可以輕鬆新增更多區域。 新增區域時，您需要指定想要配置的單位數。 例如，您在 **主要** 區域中可以有一個單位，而在其他某些區域中有五個單位。 您可以依每個區域中的流量來調整單位數。 如需詳細資訊，請參閱[如何將 Azure API 管理服務執行個體部署到多個 Azure 區域](api-management-howto-deploy-multi-region.md)。

您可以升級到任何階層，也能從任何階層降級。 升級或降級可以移除某些功能（例如，Vnet 或多區域部署），從進階層降級為 Standard 或 Basic。

> [!NOTE]
> 升級或調整程序需要 15 到 45 分鐘才會生效。 完成時，您會收到通知。

> [!NOTE]
> **使用量** 層中的 API 管理服務會根據流量自動調整。

## <a name="scale-your-api-management-service"></a>調整您的 API 管理服務

![調整 Azure 入口網站中的 API 管理服務](./media/upgrade-and-scale/portal-scale.png)

1. 在 [Azure 入口網站](https://portal.azure.com/)中，流覽至您的 API 管理服務。
2. 從功能表選取 [ **位置** ]。
3. 按一下包含您要調整之位置的資料列。
4. 指定新數目的 **單位** ，請使用滑杆或輸入數位。
5. 按一下 [套用]。

## <a name="change-your-api-management-service-tier"></a>變更您的 API 管理服務層級

1. 在 [Azure 入口網站](https://portal.azure.com/)中，流覽至您的 API 管理服務。
2. 按一下功能表中的 **定價層** 。
3. 從下拉式清單中選取所需的服務層級。 使用滑杆來指定您的 API 管理服務在變更之後的規模。
4. 按一下 [檔案] 。

## <a name="downtime-during-scaling-up-and-down"></a>相應增加和減少期間的停機時間
如果您要從或升級至開發人員層，將會有停機時間。 否則，就不會有停機時間。 

## <a name="compute-isolation"></a>計算隔離
如果您的安全性需求包含 [計算隔離](../azure-government/azure-secure-isolation-guidance.md#compute-isolation)，則可以使用 **隔離** 的定價層。 這一層可確保 API 管理服務實例的計算資源會取用整個實體主機，並提供支援所需的隔離層級，例如美國國防部影響等級 5 (IL5) 工作負載。 若要取得隔離層的存取權，請 [建立支援票證](../azure-portal/supportability/how-to-create-azure-support-request.md)。 



## <a name="next-steps"></a>後續步驟

- [如何將 Azure API 管理服務執行個體部署到多個 Azure 區域](api-management-howto-deploy-multi-region.md)
- [如何自動調整 Azure API 管理服務執行個體](api-management-howto-autoscale.md)
- [規劃和管理 API 管理的成本](plan-manage-costs.md)
- [API 管理限制](../azure-resource-manager/management/azure-subscription-service-limits.md#api-management-limits)