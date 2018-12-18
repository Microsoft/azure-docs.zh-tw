---
title: Azure 成本管理概觀 | Microsoft Docs
description: 「Azure 成本管理」是一個成本管理解決方案，有助於監視及控制 Azure 支出並最佳化資源使用狀況。
services: cost-management
keywords: ''
author: bandersmsft
ms.author: banders
ms.date: 09/21/2018
ms.topic: overview
ms.service: cost-management
manager: dougeby
ms.custom: ''
ms.openlocfilehash: 470802c1119c1c367dcd72f782dfc1036f75f6db
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47035941"
---
# <a name="what-is-azure-cost-management"></a>什麼是 Azure 成本管理？

成本管理是您有效規劃及控制商務相關成本的程序。 成本管理工作通常是由財務、管理和應用程式小組執行。 Azure 成本管理可協助組織與 Microsoft Enterprise 合約 (EA) 在規劃時考量成本。 它也有助於有效地分析成本，並採取動作來將雲端費用最佳化。 若要深入了解如何以組織身分進行成本管理，請查看 [Azure 成本管理最佳做法](cost-mgt-best-practices.md)一文。

雖然計費與成本管理相關，則實則不同。 計費是涉及向客戶開立商品或服務發票以及管理商業關係的程序。  採購和財務小組通常會進行計費工作。

成本管理會利用進階分析來顯示組織的成本和使用量模式。 成本管理中的報告會顯示 Azure 成本、使用量、保留執行個體及 Azure Hybrid Benefit 使用情況。 整體而言，這些報告會針對使用量和 Azure Marketplace 費用，顯示您的內部和外部成本。 其他費用，例如保留購買、支援和稅金還不會顯示在報告中。 這些報告可協助您了解您的費用和資源使用情況，並可協助您找出異常的費用。 也可提供預測性分析。 成本管理會使用 Azure 管理群組、預算及建議，清楚地說明如何組織您的費用以及如何降低成本。

您可以使用 Azure 入口網站或適用於匯出自動化的各種 API ，來與外部系統和程序整合。 也可使用自動化計費資料匯出和排程定報告。

## <a name="plan-and-control-expenses"></a>計劃和控制費用

成本管理協助您規劃和控制成本的方式包括：成本分析、預算、建議，以及匯出成本管理資料。

您可以使用成本分析來探索及分析組織成本。 您可以檢視組織所彙總的成本，以了解成本是在何處累算，以及識別費用趨勢。 此外，您可以查看經過一段時間累積的成本，對照預算來預估每月、每季，或甚至每年的成本趨勢。

預算可協助您進行規劃並符合貴組織的財務責任歸屬。 其有助於避免超過成本閾值或限制。 預算也可以協助您通知其他人其費用的相關資訊以主動管理成本。 透過預算，您可以查看費用在經過一段時間的進展方式。

建議顯示如何透過識別閒置和使用量過低的資源來最佳化及改善效率。 或者，其可顯示費用較低的資源選項。 當您依照建議採取動作時，您會變更您使用資源的方式來節省成本。 若要採取行動，請先檢視成本最佳化建議以檢視潛在的使用效率不足。 接下來，您可依照建議採取動作，將 Azure 資源修改為使用更符合成本效益的選項。 然後，您可驗證此動作，以確保您所進行的變更成功。

如果您使用外部系統來存取或檢閱成本管理資料，即可輕鬆地從 Azure 匯出資料。 而且，您可以設定 CSV 格式的每日排程匯出，並且在 Azure 儲存體中儲存資料檔案。 然後，您可以從外部系統存取資料。

## <a name="consider-cloudyn"></a>考慮 Cloudyn

[Cloudyn](overview.md) 是一項與成本管理相關的 Azure 服務。 透過 Cloudyn，您可以追蹤 Azure 資源的雲端使用量和支出。 它也支援其他雲端提供者，包括 AWS 和 Google。 淺顯易懂的儀表板報告有助於成本配置和回報 (showback)/計費 (chargeback)。 成本管理目前不支援回報/退款或其他雲端服務提供者。 不過，Cloudyn 是「不」支援雲端提供者的選項。 成本管理目前僅支援 Azure EA 帳戶。 雖然它不支援個別或預付型帳戶或 Microsoft 雲端服務提供者帳戶，但 Cloudyn 提供支援。 如果您有上述其中一個帳戶，您可以使用 Cloudyn 協助管理您的成本。

## <a name="additional-azure-tools"></a>其他 Azure 工具

Azure 有其他不屬於 Azure 成本管理功能集的工具。 不過，它們在成本管理流程中扮演重要角色。 若要深入了解這些工具，請參閱下列連結。

- [Azure 定價計算機](https://azure.microsoft.com/pricing/calculator/) - 使用此工具來評估您的前期雲端成本。
- [Azure Migrate](../migrate/migrate-overview.md) - 評估您目前的資料中心工作負載，以取得 Azure 替代解決方案需求的見解。
- [Azure Advisor](../advisor/advisor-overview.md) - 識別未使用的 VM，並接收有關 Azure 保留執行個體購買的建議。
- [Azure Hybrid Benefit](https://azure.microsoft.com/pricing/hybrid-benefit/) - 將您目前的內部部署 Windows Server 或 SQL Server 授權用於 Azure 中的 VM 以節省成本。


## <a name="next-steps"></a>後續步驟

您現在已經熟悉成本管理，下一步是開始使用此服務。

- 開始使用 Azure 成本管理來[分析成本](quick-acm-cost-analysis.md)。
- 您也可以進一步閱讀 [Azure 成本管理最佳做法](cost-mgt-best-practices.md)。
