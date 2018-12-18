---
title: Azure Site Recovery 中的實體伺服器至 Azure 複寫架構 | Microsoft Docs
description: 本文概述使用 Azure Site Recovery 服務將內部部署實體伺服器複寫至 Azure 時，所使用的元件和架構
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 10/10/2018
ms.author: raynew
ms.openlocfilehash: 15e8e9ce7b12585f7ea89a0440fdb93cfb171feb
ms.sourcegitcommit: 4b1083fa9c78cd03633f11abb7a69fdbc740afd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/10/2018
ms.locfileid: "49077034"
---
# <a name="physical-server-to-azure-replication-architecture"></a>實體伺服器至 Azure 複寫架構

本文說明透過 [Azure Site Recovery](site-recovery-overview.md) 服務，在內部部署網站與 Azure 之間複寫、容錯移轉和復原 Windows 和 Linux 實體伺服器時所使用的架構和程序。


## <a name="architectural-components"></a>架構元件

下表和圖形提供將實體伺服器複寫至 Azure 時所用元件的高層級檢視。  

**元件** | **需求** | **詳細資料**
--- | --- | ---
**Azure** | Azure 訂用帳戶、Azure 儲存體帳戶及 Azure 網路。 | 從內部部署 VM 複寫的資料會儲存在儲存體帳戶中。 從內部部署環境容錯移轉至 Azure 時，便會以複寫的資料建立 Azure VM。 Azure VM 在建立後會連線到 Azure 虛擬網路。
**組態伺服器** | 系統會部署單一內部部署實體機器或 VMware VM，以執行所有的內部部署 Site Recovery 元件。 此 VM 會執行組態伺服器、處理序伺服器和主要目標伺服器。 | 組態伺服器會協調內部部署與 Azure 之間的通訊，以及管理資料複寫。
 **處理序伺服器**：  | 預設會與組態伺服器一起安裝。 | 會做為複寫閘道器。 接收複寫資料，以快取、壓縮和加密進行最佳化，然後將複寫資料傳送至 Azure 儲存體。<br/><br/> 處理序伺服器也會在您要複寫的伺服器上安裝行動服務。<br/><br/> 隨著部署規模擴大，您可以新增額外的個別處理序伺服器，以處理日較大的複寫流量。
 **主要目標伺服器** | 預設會與組態伺服器一起安裝。 | 在從 Azure 容錯回復期間，處理複寫資料。<br/><br/> 針對大型部署，您可以新增額外的個別主要目標伺服器進行容錯回復。
**複寫的伺服器** | 行動服務會安裝在您複寫的每部伺服器上。 | 我們建議您允許從處理序伺服器自動安裝。 或者，您可以手動安裝服務，或使用 System Center Configuration Manager 等自動化部署方法。

**實體至 Azure 架構**

![元件](./media/physical-azure-architecture/arch-enhanced.png)

## <a name="replication-process"></a>複寫程序

1. 您需設定部署，包括內部部署和 Azure 元件。 在 Recovery Services 保存庫中，您需指定複寫來源和目標、設定組態伺服器、建立複寫原則，以及啟用複寫。
2. 機器會根據複寫原則進行複寫，並將伺服器資料的初始複本複寫到 Azure 儲存體。
3. 初始複寫完成之後，就會開始將差異變更複寫到 Azure。 機器的追蹤變更會保存在 .hrl 檔案中。
    - 機器會在輸入連接埠 HTTPS 443 上與組態伺服器進行通訊，以便進行複寫管理。
    - 機器會透過輸入連接埠 HTTPS 9443 (可修改) 將複寫資料傳送到處理序伺服器。
    - 組態伺服器會透過輸出連接埠 HTTPS 443 與 Azure 協調複寫管理。
    - 處理序伺服器會透過輸出連接埠 443，接收來源機器所傳來的資料、將其最佳化並加密，再將它傳送至 Azure 儲存體。
    - 如果您啟用多部 VM 一致性，則複寫群組中的機器會透過連接埠 20004 彼此通訊。 如果您將多部機器群組為幾個共用當機時保持一致復原點和應用程式一致復原點的複寫群組，當這些群組在進行容錯移轉時，便會使用多部 VM。 如果機器執行的是相同的工作負載，且需要保持一致，此功能就很實用。
4. 流量透過網際網路複寫到 Azure 儲存體的公用端點。 或者，您可以使用 Azure ExpressRoute [公用對等](../expressroute/expressroute-circuit-peerings.md#azure-public-peering)。 不支援從內部部署網站透過站台對站台 VPN 將流量複寫至 Azure。


**實體到 Azure 複寫程序**

![複寫程序](./media/physical-azure-architecture/v2a-architecture-henry.png)

## <a name="failover-and-failback-process"></a>容錯移轉和容錯回復程序

設定複寫並已執行災害復原演練 (測試容錯移轉) 來檢查一切如預期般運作之後，您可以視需要執行容錯移轉和容錯回復。 請注意：

- 不支援有計劃的容錯移轉。
- 您必須容錯回復到內部部署 VMware VM。 這表示您需要內部部署 VMware 基礎結構，即使是在將內部部署實體伺服器複寫到 Azure 時。
- 您可容錯移轉單一機器，或建立復原方案，同時容錯移轉多部機器。
- 當您執行容錯移轉時，系統會從 Azure 儲存體中複寫的資料建立 Azure VM。
- 觸發初始容錯移轉之後，您要認可讓它開始從 Azure VM 存取工作負載。
- 當主要的內部部署網站恢復可用狀態時，您就可以容錯回復。
- 您需要設定容錯回復基礎結構，包括：
    - **Azure 中的暫時處理序伺服器**︰若要從 Azure 容錯回復，您可設定 Azure VM 來作為處理序伺服器，以處理來自 Azure 的複寫。 容錯回復完成後，您可以刪除此 VM。
    - **VPN 連線**：若要容錯回復，您需要有從 Azure 網路到內部部署網站的 VPN 連線 (或 Azure ExpressRoute)。
    - **個別的主要目標伺服器**︰根據預設，隨著組態伺服器安裝的主要目標伺服器 (在內部部署 VMware VM 上) 會處理容錯回復。 不過，如果您需要容錯回復大量流量，您應該針對此目的設定個別的內部部署主要目標伺服器。
    - **容錯回復原則**︰若要複寫回到內部部署網站，您需要容錯回復原則。 此原則會在您建立從內部部署環境到 Azure 的複寫原則時自動建立。
    - **VMware 基礎結構**：您需要 VMware 基礎結構進行容錯回復。 您無法容錯回復到實體伺服器。
- 備妥元件後，容錯回復分三個階段進行：
    - 第 1 階段：重新保護 Azure VM，使其能從 Azure 複寫回到內部部署 VMware VM。
    - 第 2 階段：執行容錯移轉至內部部署網站。
    - 第 3 階段：容錯回復工作負載之後，您會重新啟用複寫。

**從 Azure 容錯回復 VMware**

![容錯回復](./media/physical-azure-architecture/enhanced-failback.png)


## <a name="next-steps"></a>後續步驟

請遵循[此教學課程](physical-azure-disaster-recovery.md)，以啟用實體伺服器至 Azure 的複寫。
