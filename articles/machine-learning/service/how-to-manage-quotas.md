---
title: 如何管理及要求 Azure Machine Learning 服務的配額
description: 此操作指南說明 Azure Machine Learning 資源上的各種配額，以及如何檢視及要求更多配額。
services: machine-learning
ms.service: machine-learning
ms.component: core
ms.topic: conceptual
ms.reviewer: jmartens
author: nishankgu
ms.author: nigup
ms.date: 9/24/2018
ms.openlocfilehash: 0ee0a29be6a313e09cd77051dc29bc0131b57a40
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46997806"
---
# <a name="manage-and-request-quotas-for-azure-resources"></a>管理及要求 Azure 資源的配額

如同使用其他 Azure 服務，對於與 Azure Machine Learning 服務相關聯的特定資源有一些限制。 這些限制的範圍，包括您可建立的工作區數目上限，到用於定型或推斷您的模型之實際基礎計算上的限制。 此文章針對您訂用帳戶之各種 Azure 資源所預先設定的限制提供更詳細的資料，另外還包含針對各種資源類型要求配額增強功能的方便連結。

當您設計及相應增加您的 Azure ML 資源時，請記住這些配額。 例如，如果您的叢集未達到您所指定的目標節點數目，您可能已達到訂用帳戶的 Batch AI 核心限制。 如果您計劃在 Batch AI 中執行生產工作負載，您可能需要增加一或多個高於預設值的配額。 如果您想要將限制或配額提升到預設限制以上，您可以免費提出線上客戶支援要求。 您無法將限制提升到高於下表中所示的「上限」值。 如果沒有上限欄，資源即沒有可調整的限制。 

## <a name="special-considerations"></a>特殊考量

+ 配額是一種信用限制，不是容量保證。 如果您有大規模的容量需求，請連絡 Azure 支援。

+ 您的配額是由您訂用帳戶中的所有服務 (除了 Batch AI) 一起共用的，Azure Machine Learning 就是其中之一。 當評估您的容量需求時，請務必計算所有服務的配額使用量。

+ 預設限制會因供應項目類別類型 (例如免費試用、預付型方案) 和系列 (例如 Dv2、F、G 等) 而有所差異。

## <a name="default-resource-quotas"></a>預設資源配額

以下是 Azure 訂用帳戶中各種資源類型的配額限制細目。 

> [!Important]
> 限制日後有可能會變更。 最新的限制隨時可在適用於全 Azure 的服務層級配額[文件](https://docs.microsoft.com/azure/azure-subscription-service-limits/)中找到。

### <a name="virtual-machines"></a>虛擬機器 
可在 Azure 訂用帳戶上跨服務佈建，或以獨立方式佈建的虛擬機器數目有所限制。 此限制是區域層級限制，並同時針對總核心數量以及每個系列進行限制。

請務必注意，虛擬機器核心具有個別強制執行的區域總數限制，以及區域每個大小系列 (Dv2、F 等等) 的限制。 例如，請考慮美國東部訂用帳戶的總計 VM 核心限制為 30、A 系列核心限制為 30，和 D 系列核心限制為 30。 此訂用帳戶會允許部署 30 個 A1 VM、30 個 D1 VM，或是兩個的組合，總計不超過 30 個核心 (例如 10 個 A1 VM 和 20 個 D1 VM)。

[!INCLUDE [azure-subscription-limits-azure-resource-manager](../../../includes/azure-subscription-limits-azure-resource-manager.md)]

如需配額限制詳細資訊與最新清單，請查看[這裡](https://docs.microsoft.com/azure/azure-subscription-service-limits#subscription-limits-1)的全 Azure 配額文章。

### <a name="batch-ai-clusters"></a>Batch AI 叢集
在 Batch AI 中，針對訂用帳戶中每個區域允許的核心數目與叢集數目有預設的配額限制。 Batch AI 配額不同於上述 VM 核心配額，而且目前在兩個資源類型之間不共用核心限制。

可用的資源：
+ 每個區域的專用核心預設限制為 10 - 24 個。  您可以增加每個 Batch AI 訂用帳戶的專用核心數目。 請連絡 Azure 支援以討論增加選項。

+ 每個區域的低優先順序核心預設限制為 10 - 24 個。  您可以增加每個 Batch AI 訂用帳戶的低優先順序核心數目。 請連絡 Azure 支援以討論增加選項。

+ 每個區域的叢集都有預設的限制 (20 個) 和上限 (200 個)。 如果您想要要求增加到超過此限制，請連絡 Azure 支援。

如需配額限制的詳細資訊與最新清單，請查看[這裡](https://docs.microsoft.com/azure/azure-subscription-service-limits#batch-ai-limits)的全 Azure 配額文章。

### <a name="container-instances"></a>容器執行個體

針對您在指定期間 (以每小時為範圍) 或整個訂用帳戶中可以啟動的容器執行個體數目也有限制。

[!INCLUDE [container-instances-limits](../../../includes/container-instances-limits.md)]

如需配額限制的詳細資訊與最新清單，請查看[這裡](https://docs.microsoft.com/azure/azure-subscription-service-limits#container-instances-limits)的全 Azure 配額文章。

### <a name="storage"></a>儲存體
針對每個區域以及指定的訂用帳戶中的儲存體帳戶數目有限制。 預設限制為 200 個，包括標準與進階儲存體帳戶兩者。 如果您在指定區域需要超過 200 個儲存體帳戶，請透過 [Azure 支援](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest/)提出要求。 Azure 儲存體小組將會檢閱您的商務案例，而且可以針對指定區域核准多達 250 個儲存體帳戶。


## <a name="find-your-quotas"></a>找出您的配額

透過 Azure 入口網站檢視各種資源 (例如虛擬機器、儲存體、網路) 的配額非常簡單。

1. 在左窗格中，選取 [所有服務]，然後選取 [一般] 類別底下的 [訂用帳戶]。

1. 從訂用帳戶清單中，選取您要尋找其配額的訂用帳戶。

   **有一點需要注意**，具體而言是檢視 Batch AI 配額時。 如上所述，該配額與您訂用帳戶上的計算配額是分開的。 
   針對 Batch AI，在選取 [所有服務] 之後，請搜尋 Batch AI 並開啟該服務。

1. 在 [設定] 下，選取 [使用量 + 配額] 以檢視目前配額限制與使用量。

1. 選取您的訂用帳戶以檢視配額限制。 請記得篩選出您感興趣的服務與區域。


## <a name="request-quota-increases"></a>要求增加配額

如果您想要將限制或配額提升到預設限制以上，您可以免費[建立線上客戶支援要求](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest/)。 

您無法將限制提升到高於下表中所示的上限值。 如果沒有上限，資源即沒有可調整的限制。 [這篇](https://docs.microsoft.com/azure/azure-resource-manager/resource-manager-quota-errors)文章涵蓋配額增加程序的詳細資料。

當要求增加配額時，您需要選取正在要求提升配額的服務 (可以是 AzureML 配額、Batch AI 配額或儲存體配額等服務)。 

> [!NOTE]
> [免費試用訂用帳戶](https://azure.microsoft.com/offers/ms-azr-0044p)無法增加限制或配額。 如果您有[免費試用訂用帳戶](https://azure.microsoft.com/offers/ms-azr-0044p)，則可以升級到[預付型方案](https://azure.microsoft.com/offers/ms-azr-0003p/)訂用帳戶。 如需詳細資訊，請參閱[將 Azure 免費試用升級至預付型方案](../../billing/billing-upgrade-azure-subscription.md)和[免費試用訂用帳戶常見問題集](https://azure.microsoft.com/free/free-account-faq)。
