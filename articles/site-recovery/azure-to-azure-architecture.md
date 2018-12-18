---
title: Azure Site Recovery 中的 Azure 至 Azure 複寫架構 | Microsoft Docs
description: 本文概要說明使用 Azure Site Recovery 服務複寫 Azure 區域之間的 Azure VM 時所用的元件和架構。
services: site-recovery
author: rayne-wiselman
manager: carmonm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 10/10/2018
ms.author: raynew
ms.openlocfilehash: 20387c325497934ccb2e02188b24a0125f937e00
ms.sourcegitcommit: 4b1083fa9c78cd03633f11abb7a69fdbc740afd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/10/2018
ms.locfileid: "49078252"
---
# <a name="azure-to-azure-replication-architecture"></a>Azure 至 Azure 複寫架構


本文說明使用 [Azure Site Recovery](site-recovery-overview.md) 服務，在 Azure 區域之間對 Azure 虛擬機器 (VM) 進行複寫、容錯移轉和復原時所使用的架構。




## <a name="architectural-components"></a>架構元件

下圖提供特定區域 (在此範例中為美國東部位置) 之中 Azure VM 環境的高層級檢視。 在 Azure VM 的環境中：
- 應用程式可以在 VM 上執行，而受控和非受控磁碟分散於不同的儲存體帳戶。
- VM 可以包含在虛擬網路內的一個或多個子網路。


**Azure 至 Azure 複寫**

![客戶環境](./media/concepts-azure-to-azure-architecture/source-environment.png)

## <a name="replication-process"></a>複寫程序

### <a name="step-1"></a>步驟 1

當您啟用 Azure VM 複寫時，系統會根據來源區域設定，在目標區域中自動建立下列資源。 您可以視需要自訂目標來源設定。

![啟用複寫程序，步驟 1](./media/concepts-azure-to-azure-architecture/enable-replication-step-1.png)

**Resource** | **詳細資料**
--- | ---
**目標資源群組** | 在容錯移轉之後複寫的 VM 所屬的資源群組。 此資源群組的位置可以是任何 Azure 區域，但是裝載您來源虛擬機器所在的 Azure 區域除外。
**目標虛擬網路** | 在容錯移轉之後複寫的 VM 所在的虛擬網路。 在來源和目標的虛擬網路之間會建立網路對應，反之亦然。
**快取儲存體帳戶** | 將來源 VM 變更複寫到目標儲存體帳戶之前，系統會追蹤這些變更並傳送到來源位置中的快取儲存體帳戶。 此步驟確保對於 VM 上執行的生產應用程式所造成的影響會降到最低。
**目標儲存體帳戶 (如果來源 VM 不使用受控磁碟)**  | 將資料複寫至其中的目標位置儲存體帳戶。
** 複本受控磁碟 (如果來源 VM 位在受控磁碟)**  | 目標位置中將複寫資料的目的地受控磁碟。
**目標可用性設定組**  | 在容錯移轉之後複寫的 VM 所在的可用性設定組。

### <a name="step-2"></a>步驟 2

啟用複寫時，系統會在 VM 上自動安裝 Site Recovery 擴充行動服務：

1. 向 Site Recovery 註冊 VM。

2. 對於 VM 設定連續複寫。 VM 磁碟上的資料寫入會持續傳送到來源位置的快取儲存體帳戶中。

   ![啟用複寫程序，步驟 2](./media/concepts-azure-to-azure-architecture/enable-replication-step-2.png)


 Site Recovery 永遠不會需要 VM 的輸入連線能力。 下列各項僅需輸出連線能力。

 - Site Recovery 服務端點 URL/IP 位址
 - Office 365 驗證 URL/IP 位址
 - 快取儲存體帳戶 IP 位址

如果您啟用多部 VM 一致性，則複寫群組中的機器會透過連接埠 20004 彼此通訊。 請確定並沒有防火牆應用裝置封鎖了 VM 之間透過連接埠 20004 進行的內部通訊。

> [!IMPORTANT]
如果您想要讓 Linux VM 成為複寫群組的一部分，請確定已根據特定 Linux 版本的指引手動開啟連接埠 20004上 的輸出流量。

### <a name="step-3"></a>步驟 3

連續複寫進行之後，磁碟寫入會立即傳送至快取儲存體帳戶。 Site Recovery 會處理資料，並將資料傳送到目標儲存體帳戶或複本受控磁碟。 處理資料之後，目標儲存體帳戶會每隔幾分鐘產生復原點。

## <a name="failover-process"></a>容錯移轉程序

您起始容錯移轉時，系統會在目標資源群組、目標虛擬網路，目標子網路和目標可用性設定組中建立 VM。 在容錯移轉時，您可以使用任何復原點。

![容錯移轉程序](./media/concepts-azure-to-azure-architecture/failover.png)

## <a name="next-steps"></a>後續步驟

將 Azure VM [快速複寫](azure-to-azure-quickstart.md)到次要地區。
