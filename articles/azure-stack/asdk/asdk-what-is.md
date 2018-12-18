---
title: Azure Stack 開發套件 (ASDK) 簡介 | Microsoft Docs
description: 說明 ASDK 是什麼以及用於評估 Microsoft Azure Stack 的常見使用案例。
services: azure-stack
documentationcenter: ''
author: jeffgilb
manager: femila
editor: ''
ms.assetid: ''
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: overview
ms.custom: mvc
ms.date: 10/15/2018
ms.author: jeffgilb
ms.reviewer: misainat
ms.openlocfilehash: fa20f746e55f784e02244355c96ac273b9906acc
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49339537"
---
# <a name="what-is-the-azure-stack-development-kit"></a>何謂 Azure Stack 開發套件？
[Microsoft Azure Stack 整合系統](.\.\azure-stack-poc.md)的大小範圍為 4 到 12 個節點，並且由硬體合作夥伴與 Microsoft 共同支援。 您可以使用 Azure Stack 整合系統，來為生產環境工作負載啟用新案例。 如果您是管理整合系統基礎結構並提供服務的 Azure Stack 操作員，請參閱我們的[操作員文件](https://docs.microsoft.com/azure/azure-stack)。

Azure Stack 開發套件 (ASDK) 是 Azure Stack 的單一節點部署，供您**免費**下載並使用。 所有的 ASDK 元件會安裝在單一主機伺服器電腦上所執行的虛擬機器中，且此電腦必須符合或超過[最低硬體需求](asdk-deploy-considerations.md#hardware)。 ASDK 旨在提供一個環境，讓您評估 Azure Stack 並使用 API 以及與「非生產」環境中相同的 Azure 工具來開發現代化的應用程式。 

> [!IMPORTANT]
> ASDK 的目的並非要在生產環境中使用或支援。

所有 ASDK 元件都會部署至單一開發套件主機電腦，因此可用的實體資源有限。 使用 ASDK 部署時，Azure Stack 基礎結構 VM 和租用戶 VM 會共存於同一部伺服器電腦上。 此設定不適合進行規模或效能評估。

ASDK 的設計目的是要為下列人員提供 Azure 一致的混合式雲端體驗：
- **系統管理員** (Azure Stack 操作員)。 ASDK 是用來評估和了解可用 Azure Stack 服務的絕佳資源。
- **開發人員**。 ASDK 可用來在內部部署環境 (開發/測試環境) 中開發混合式或現代應用程式。 這可在 Azure Stack 生產部署之前或同時間提供重複的開發體驗。 

請觀看這部簡短的影片以深入了解 ASDK：

> [!VIDEO https://www.youtube.com/embed/dbVWDrl00MM]


## <a name="asdk-and-multi-node-azure-stack-differences"></a>ASDK 和多節點 Azure Stack 的差異
請務必了解，單一節點 ASDK 部署與多節點 Azure Stack 部署會在幾個重要方面有所不同。

|說明|ASDK|多節點 Azure Stack|
|-----|-----|-----|
|**調整**|所有元件都安裝在單一節點伺服器電腦上。|其大小範圍可為 4 到 12 個節點。|
|**恢復功能**|單一節點組態未提供高可用性|支援[高可用性](.\.\azure-stack-key-features.md#high-availability-for-azure-stack)功能。|
|**網路功能**|ASDK 使用名為 AzS-BGPNAT01 的 VM 來路由傳送所有 ASDK 網路流量。 不需要任何其他參數。|AzS-BGPNAT01 VM 不存在於多節點部署中。 需要更複雜的[網路路由基礎結構](.\.\azure-stack-network.md#network-infrastructure)，包括機架頂端 (TOR)、基礎板管理控制器 (BMC) 和界限 (資料中心網路) 交換器。|
|**修補和更新程序**|若要移至新版 ASDK，您必須在開發套件主機電腦上重新部署 ASDK。|[修補和更新](.\.\azure-stack-updates.md)程序可用來更新已安裝的 Azure Stack 版本。|
|**支援**|MSDN Azure Stack 論壇。 非生產環境「不」支援 Microsoft 客戶服務及支援 (CSS)。|[MSDN Azure Stack 論壇](https://social.msdn.microsoft.com/Forums/en-US/home?forum=AzureStack)和完整的 CSS 支援。|
| | |

## <a name="learn-about-available-services"></a>了解可用的服務
身為 Azure Stack 操作員，您需要知道有哪些服務可提供給使用者。 Azure Stack 支援一部分的 Azure 服務，所支援服務的清單會持續隨著時間演變。

### <a name="foundational-services"></a>基礎服務
依預設，當您部署 ASDK 時，Azure Stack 會包含下列的「基礎服務」：
- 計算
- 儲存體
- 網路功能
- Key Vault

利用這些基礎服務，您可用最少的設定，將基礎結構即服務 (IaaS) 提供給您的使用者。

### <a name="additional-services"></a>其他服務
目前所支援的其他平台即服務 (PaaS) 服務如下：
- App Service 方案
- Azure Functions
- SQL 和 MySQL 資料庫

> [!NOTE]
> 必須先對這些服務進行其他設定，才能提供給使用者使用，而且在您安裝 ASDK 時，預設無法提供這些服務。

## <a name="service-roadmap"></a>服務藍圖
Azure Stack 會持續新增對其他 Azure 服務的支援。 若要了解 Azure Stack 的後續發展，請參閱 [Azure Stack 藍圖](https://azure.microsoft.com/roadmap/?tag=azure-stack)。 


## <a name="next-steps"></a>後續步驟
若要開始評估 Azure Stack，您必須先[下載最新的 ASDK](asdk-download.md) 並準備 ASDK 主機電腦。 準備好開發套件主機後，您就可以安裝 ASDK，並登入系統管理員和使用者入口網站以開始使用 Azure Stack。