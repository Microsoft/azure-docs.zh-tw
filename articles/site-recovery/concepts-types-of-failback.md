---
title: Azure Site Recovery 中的容錯回復 | Microsoft Docs
description: 本文概述使用 Azure Site Recovery 服務容錯回復至內部部署時要考慮之各種類型的容錯回復和警告。
services: site-recovery
author: rajani-janaki-ram
manager: guaravd
ms.service: site-recovery
ms.topic: article
ms.date: 07/06/2018
ms.author: rajanki
ms.openlocfilehash: 2a9ee380fc16c4088d98875dd465509c4023d037
ms.sourcegitcommit: a06c4177068aafc8387ddcd54e3071099faf659d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/09/2018
ms.locfileid: "37920384"
---
# <a name="overview-of-failback"></a>容錯回復的概觀

當您容錯移轉至 Azure 之後，您可以容錯回復至內部部署網站。 有兩種不同類型的容錯回復能與 Azure Site Recovery 搭配使用： 

- 容錯回復至原始位置 
- 容錯回復至其他位置

如果您已容錯移轉 VMware 虛擬機器，在它仍存在的前提下，您可以將它容錯回復到相同的來源內部部署虛擬機器。 在此案例中，系統只會將變更複寫回來。 此案例稱為**原始位置復原**。 如果內部部署虛擬機器不存在，則此案例為**替代位置復原**。

> [!NOTE]
> 您只能容錯回復至原始的 vCenter 和設定伺服器。 您無法部署新的設定伺服器，並使用它進行容錯回復。 此外，您無法將新的 vCenter 新增至現有設定伺服器並容錯回復至新的 vCenter。

## <a name="original-location-recovery-olr"></a>原始位置復原 (OLR)
如果您選擇容錯回復至原始虛擬機器，就必須符合下列條件：

* 如果虛擬機器是由 vCenter 伺服器管理，主要目標的 ESX 主機應該可以存取虛擬機器資料存放區。
* 如果虛擬機器位於 ESX 主機上，但不是由 vCenter 管理，則虛擬機器的硬碟必須位於主要目標的主機可以存取的資料存放區中。
* 如果虛擬機器位於 ESX 主機上，但未使用 vCenter，則應該先完成主要目標之 ESX 主機的探索後，才能重新保護。 如果您要容錯回復實體伺服器，則也適用這一條件。
* 您可以容錯回復至虛擬存放區域網路 (vSAN) 或以原始裝置對應 (RDM) 為基礎的磁碟 (如果磁碟已存在並連線到內部部署虛擬機器)。

> [!IMPORTANT]
> 請務必啟用 disk.enableUUID= TRUE，以便在容錯回復期間，讓 Azure Site Recovery 服務能夠在將寫入暫止變更的虛擬機器上識別原始 VMDK。 如果未將此值設為 TRUE，則服務會盡可能地嘗試識別對應的內部部署 VMDK。 如果找不到正確的 VMDK，它就會建立額外的磁碟，並將資料寫入至其中。

## <a name="alternate-location-recovery-alr"></a>替代位置復原 (ALR)
如果在重新保護虛擬機器之前，內部部署虛擬機器不存在，則此案例稱為替代位置復原。 重新保護工作流程會重新建立內部部署虛擬機器。 這也會導致下載完整資料。

* 當您容錯回復至替代位置時，就會將虛擬機器復原到主要目標伺服器部署所在的相同 ESX 主機上。 用於建立磁碟的資料存放區將會是重新保護虛擬機器時選取的相同資料存放區。
* 您只能容錯回復到虛擬機器檔案系統 (VMFS) 或 vSAN 資料存放區。 如果您有 RDM，重新保護與容錯回復將無法運作。
* 重新保護程序涉及一項大型的初始資料傳輸作業以及緊接著的變更作業。 之所以會有此程序，是因為虛擬機器未存在於內部部署環境。 資料必須完整地複寫回來。 此重新保護作業也需要比原始位置復原更長的時間。
* 您無法容錯回復到以 RDM 為基礎的磁碟。 只能在 VMFS/vSAN 資料存放區上建立新的虛擬機器磁碟 (VMDK)。

> [!NOTE]
> 將實體機器容錯回復至 Azure 時，只能容錯回復為 VMware 虛擬機器。 這會遵循與替代位置復原相同的工作流程。 請確定您至少探索一部主要目標伺服器以及需要容錯回復到的必要 ESX/ESXi 主機。

## <a name="next-steps"></a>後續步驟

依照步驟執行[容錯回復作業](vmware-azure-failback.md)。

