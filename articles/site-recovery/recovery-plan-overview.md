---
title: 在 Azure Site Recovery 中使用復原方案 | Microsoft Docs
description: 深入了解 Azure Site Recovery 中的復原方案。
services: site-recovery
documentationcenter: ''
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.devlang: na
ms.topic: article
ms.date: 07/06/2018
ms.author: raynew
ms.openlocfilehash: 011c9acb5d34e15f65b64d59867e7501f0720a08
ms.sourcegitcommit: a06c4177068aafc8387ddcd54e3071099faf659d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/09/2018
ms.locfileid: "37920104"
---
# <a name="about-recovery-plans"></a>關於復原方案

本文說明 [Azure Site Recovery](site-recovery-overview.md) 中的復原方案。

復原方案會將機器收集至復原群組中。 您可以將順序、指示及工作新增至方案來加以自訂。 定義方案之後，您可以在該方案上執行容錯移轉。





## <a name="why-use-a-recovery-plan"></a>為何使用復原方案？

復原方案可藉由建立您可容錯移轉的小型獨立單位，協助您定義系統性復原程序。 單位通常代表您環境中的應用程式。 復原方案會定義機器容錯移轉的方式，以及機器在容錯移轉之後啟動的順序。 使用復原方案來：

* 根據應用程式的相依性建立模型。
* 將復原工作自動化以縮短 RTO。
- 請確認您的應用程式屬於復原方案，以準備好進行移轉或災害復原。
* 請在復原方案上執行測試容錯移轉，以確保災害復原或移轉如預期般運作。


## <a name="model-apps"></a>建立應用程式模型

您可以計劃並建立復原群組，以擷取應用程式特有的屬性。 例如，我們來看一個包含 SQL Server 後端、中介軟體及 Web 前端的一般三層式應用程式。 一般而言，您要自訂復原方案，讓每一層的機器在容錯移轉之後，會以正確的順序啟動。

    - 應該先啟動 SQL 後端中，接著是中介軟體，最後再啟動 Web 前端。
    - 這個啟動順序可確保應用程式在最後一部機器啟動時已在運作。
    - 此順序可確保當中介軟體啟動並嘗試連線到 SQL Server 層時，SQL Server 層已經在執行。 
    - 此順序還可確保前端伺服器最後啟動，以便在所有元件都已啟動並執行，且應用程式已準備好接受要求之前，使用者不會連線到應用程式 URL。

若要建立此順序，請將群組新增至復原群組，並將機器新增至群組。 
    - 在指定順序之處，就會使用排序。 應用程式在適用時會以平行方式執行，以改善應用程式復原 RTO。
    - 單一群組中的機器會以平行方式容錯移轉。
    - 不同群組中的機器會依順序群組進行容錯移轉，因此只有在群組 1 中所有機器容錯移轉並啟動之後，群組 2 的機器才會開始其容錯移轉。

    ![範例復原方案](./media/recovery-plan-overview/rp.png)

在進行此自訂後，以下是您在復原方案上執行容錯移轉時所發生的情況： 

1. 關閉步驟會嘗試關閉內部部署機器。 例外狀況是如果您執行測試容錯移轉，在此情況下，主要站台會繼續執行。 
2. 關閉會在復原方案中觸發所有機器的平行容錯移轉。
3. 容錯移轉會使用複寫的資料來準備虛擬機器磁碟。
4. 啟動群組會依順序執行，並啟動每個群組中的機器。 首先，群組 1 執行、然後群組 2 執行，最後群組 3 才會執行。 如果任何群組中有多部機器，則所有機器會以平行方式啟動。


## <a name="automate-tasks"></a>自動化工作

復原大型應用程式可能是相當複雜的工作。 手動步驟會讓程序容易發生錯誤，且執行容錯移轉的人員可能不會察覺所有應用程式的複雜性。 您可以使用復原方案強制加上順序，並使用 Azure 自動化 Runbook 容錯移轉至 Azure 或使用指令碼，自動執行每個步驟所需的動作。 對於無法自動化的工作，您可以將手動動作的暫停插入復原方案中。 有幾個工作類型可供您設定：

* **容錯移轉後在 Azure VM 上的工作**：當您要容錯移轉至 Azure 時，通常必須執行動作，以便您可在容錯移轉之後連線至 VM。 例如︰ 
    * 在 Azure VM 上建立公用 IP 位址。
    * 將網路安全性群組指派給 Azure VM 的網路介面卡。
    * 將負載平衡器新增至可用性設定組。
* **容錯移轉後 VM 內的工作**：這些工作通常會重新設定機器上執行的應用程式，以便讓應用程式在新環境中繼續正常運作。 例如︰
    * 修改機器內的資料庫連接字串。
    * 變更 Web 伺服器組態或規則。


## <a name="test-failover"></a>測試容錯移轉

您可以使用復原方案來觸發測試容錯移轉。 使用下列最佳做法：

- 執行完整的容錯移轉之前，請一律完成應用程式上的測試容錯移轉。 測試容錯移轉可協助您檢查應用程式是否會在復原網站上啟動。
- 如果您發現遺漏了項目，請觸發清除，然後重新執行測試容錯移轉。 
- 請執行測試容錯移轉多次，直到您確定應用程式可順暢復原為止。
- 因為每個應用程式都是唯一的，所以您必須建置針對每個應用程式自訂的復原方案，並在每個復原方案上執行測試容錯移轉。
- 應用程式和其相依性會經常變更。 若要確保是最新的復原方案，請每一季針對每個應用程式執行測試容錯移轉。

    ![Site Recovery 方案中範例復原方案的螢幕擷取畫面](./media/recovery-plan-overview/rptest.png)

## <a name="watch-the-video"></a>觀賞影片

請觀看簡短的範例影片，示範兩層式 WordPress 應用程式的按一下容錯移轉。
    
> [!VIDEO https://channel9.msdn.com/Series/Azure-Site-Recovery/One-click-failover-of-a-2-tier-WordPress-application-using-Azure-Site-Recovery/player]



## <a name="next-steps"></a>後續步驟

- [建立](site-recovery-create-recovery-plans.md)復原方案。
* 深入了解[執行容錯移轉](site-recovery-failover.md)。  
