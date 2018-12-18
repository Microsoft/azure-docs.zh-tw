---
title: 關於使用 Azure Site Recovery 進行 VMware VM 至 Azure 的災害復原 | Microsoft Docs
description: 本文概述如何使用 Azure Site Recovery 進行 VMware VM 至 Azure 的災害復原。
author: rayne-wiselman
ms.service: site-recovery
ms.topic: conceptual
services: site-recovery
ms.date: 08/29/2018
ms.author: raynew
ms.openlocfilehash: 105ad94b442790c82af0f6b9533f195d93c5cd1d
ms.sourcegitcommit: 0c64460a345c89a6b579b1d7e273435a5ab4157a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/31/2018
ms.locfileid: "43339144"
---
# <a name="about-disaster-recovery-of-vmware-vms-to-azure"></a>關於 VMware VM 至 Azure 的災害復原

本文概述如何使用 [Azure Site Recovery](site-recovery-overview.md) 服務進行內部部署 VMware VM 至 Azure 的災害復原。

## <a name="what-is-bcdr"></a>什麼是 BCDR？

商務持續性和災害復原 (BCDR) 策略有助於讓您的企業持續保持運作。 在計劃性停機和非預期中斷期間，BCDR 會保持資料的安全性和可用性，並確保應用程式繼續執行。 除了平台 BCDR 功能 (例如區域配對和高可用性儲存)，Azure 會提供復原服務作為 BCDR 解決方案不可或缺的一部分。 復原服務包括： 

- [Azure 備份](https://docs.microsoft.com/azure/backup/backup-introduction-to-azure-backup)會備份您的內部部署和 Azure VM 資料。 您可以備份檔案和資料夾、特定工作負載，或整個 VM。 
- [Azure Site Recovery](site-recovery-overview.md) 可為在內部部署機器或 Azure IaaS VM 上執行的應用程式和工作負載，提供復原能力和災害復原。 Site Recovery 會發生中斷時協調複寫，以及處理容錯移轉至 Azure。 它也會處理從 Azure 至主要網站的復原。 

## <a name="how-does-site-recovery-do-disaster-recovery"></a>Site Recovery 如何進行災害復原？

1. 準備好 Azure 和內部部署網站之後，您可為內部部署機器設定和啟用複寫功能。
2. Site Recovery 會根據您的原則設定來協調機器的初始複寫。
3. 初始複寫之後，Site Recovery 就會將差異變更複寫到 Azure。 
4. 當一切如預期般複寫時，您可執行災害復原演練。
    - 此演練有助於確保當真實需求出現時，容錯移轉會如預期般運作。
    - 此演練會在不影響生產環境的情況下，執行測試容錯移轉。
5. 如果發生中斷，您可執行完整容錯移轉至 Azure。 您可以容錯移轉單一機器，或建立可同時容錯移轉多部機器的復原方案。
6. 在容錯移轉時，系統會從 Azure 儲存體中的 VM 資料建立 Azure VM。 使用者可以繼續從 Azure VM 存取應用程式和工作負載
7. 當內部部署網站恢復可用狀態時，您就可以從 Azure 進行容錯回復。
8. 容錯回復後，當您再度從主要網站作業時，您就會開始再次將內部部署 VM 複寫至 Azure。


## <a name="how-do-i-know-if-my-environment-is-suitable-for-disaster-recovery-to-azure"></a>如何得知我的環境是否適用於災害復原至 Azure？

Site Recovery 可複寫在支援的 VMware VM 或實體伺服器上執行的任何工作負載。 以下是您需要在您的環境中檢查的項目：

- 如果您正在複寫 VMware VM，您執行的是正確的 VMware 虛擬化伺服器版本嗎？ [請查看這裡](vmware-physical-azure-support-matrix.md#on-premises-virtualization-servers)。
- 您要複寫的機器是否支援支援的作業系統？ [請查看這裡](vmware-physical-azure-support-matrix.md#replicated-machines)。
- 針對 Linux 災害復原，機器是否執行支援的檔案系統/客體儲存體？ [請查看這裡](vmware-physical-azure-support-matrix.md#linux-file-systemsguest-storage)
- 您想要複寫的機器是否符合 Azure 需求？ [請查看這裡](vmware-physical-azure-support-matrix.md#azure-vm-requirements)。
- 是否支援您的網路組態？ [請查看這裡](vmware-physical-azure-support-matrix.md#network)。
- 是否支援您的儲存體組態？ [請查看這裡](vmware-physical-azure-support-matrix.md#storage)。


## <a name="what-do-i-need-to-set-up-in-azure-before-i-start"></a>開始之前，我需要在 Azure 中設定哪些項目？

您需要在 Azure 中準備下列各項：

1. 確認您的 Azure 帳戶具有在 Azure 中建立 VM 的權限。
2. 建立儲存體帳戶以保存所複寫機器的映像。
3. 建立 Azure 網路，而在容錯移轉後從儲存體建立的 Azure VM 會加入該網路。
4. 設定 Site Recovery 的 Azure 復原服務保存庫。 此保存庫位於 Azure 入口網站中，用於部署、設定、協調、監視您的 Site Recovery 部署，以及進行其疑難排解。

*需要其他協助？*

了解如何藉由[驗證您的帳戶](tutorial-prepare-azure.md#verify-account-permissions)、建立[儲存體帳戶](tutorial-prepare-azure.md#create-a-storage-account)和[網路](tutorial-prepare-azure.md#set-up-an-azure-network)，以及[設定保存庫](tutorial-prepare-azure.md#create-a-recovery-services-vault)來設定 Azure。



## <a name="what-do-i-need-to-set-up-on-premises-before-i-start"></a>開始之前，我需要在內部部署環境中設定哪些項目？

以下是您需要在內部部署環境執行的作業：

1. 您需要設定幾個帳戶：

    - 如果您正在複寫 VMware VM，則需要一個帳戶，以供 Site Recovery 存取 vCenter Server 或 vSphere ESXi 主機自動探索 VM。
    - 需要一個帳戶，以便在您要複寫的每部實體機器或 VM 上安裝 Site Recovery 行動服務代理程式。

2. 您需要檢查 VMware 基礎結構的相容性 (如果您先前未這麼做)。
3. 確定您可以在容錯移轉後連線到 Azure VM。 在內部部署 Windows 機器上設定 RDP，或在 Linux 機器上設定 SSH。

*需要其他協助？*
- 準備用於[自動探索](vmware-azure-tutorial-prepare-on-premises.md#prepare-an-account-for-automatic-discovery)以及[安裝行動服務](vmware-azure-tutorial-prepare-on-premises.md#prepare-an-account-for-mobility-service-installation)的帳戶。
- [確認](vmware-azure-tutorial-prepare-on-premises.md#check-vmware-requirements) VMware 設定相容。
- 做好[準備](vmware-azure-tutorial-prepare-on-premises.md#prepare-to-connect-to-azure-vms-after-failover)，以便在容錯移轉後於 Azure 中連線。
- 如果您需要更多有關在容錯移轉後為 Azure VM 設定 IP 位址的深入說明，[請閱讀這篇文章](concepts-on-premises-to-azure-networking.md)。

## <a name="how-do-i-set-up-disaster-recovery"></a>如何設定災害復原？

在您備妥 Azure 和內部部署基礎結構之後，您可以設定災害復原。

1. 若要了解您需要部署的元件，請檢閱 [VMware 至 Azure 架構](vmware-azure-architecture.md)，以及[實體至 Azure 架構](physical-azure-architecture.md)。 有多個元件，因此務必了解這些元件如何相互配合。
2. **來源環境**：部署的第一個步驟，就是設定複寫來源環境。 指定您要複寫的項目，以及您要複寫到的位置。
3. **組態伺服器**︰您必須在內部部署來源環境中設定組態伺服器：
    - 組態伺服器是單一內部部署機器。 針對 VMware 災害復原，建議您將其當作可從下載的 OVF 範本部署的 VMware VM 來部署。
    - 設定伺服器會協調內部部署與 Azure 之間的通訊。
    - 還有一些其他元件會在組態伺服器機器上執行。
        - 處理序伺服器會接收、最佳化複寫資料，並將此資料傳送至 Azure 儲存體。 它也會在您要複寫的機器上處理行動服務的自動安裝，並且在 VMware 伺服器上執行 VM 的自動探索。
        - 主要目標伺服器會在從 Azure 容錯回復期間，處理複寫資料。
    - 設定包括在保存庫中註冊組態伺服器、下載 MySQL 伺服器和 VMware PowerCLI，以及指定針對自動探索和行動服務安裝所建立的帳戶。
4. **目標環境**：藉由指定您的 Azure 訂用帳戶、儲存體和網路設定，來設定您的目標 Azure 環境。
5. **複寫原則**：您可指定應如何進行複寫。 設定包含建立和儲存復原點的頻率，以及是否應該建立應用程式一致的快照集。
6. **啟用複寫**。 您可為內部部署機器啟用複寫。 如果您建立了用以安裝行動服務的帳戶，則該服務會在您為機器啟用複寫時安裝。 

*需要其他協助？*

- 如需這些步驟的快速逐步解說，您可以試試看 [VMware 教學課程](vmware-azure-tutorial.md)，以及[實體伺服器逐步解說](physical-azure-disaster-recovery.md)。
- [深入了解](vmware-azure-set-up-source.md)如何設定來源環境。
- [深入了解](vmware-azure-deploy-configuration-server.md)組態伺服器的需求，以及使用 VMware 複寫的 OVF 範本來設定組態伺服器。 如果基於某些原因而無法使用範本，或您正在複寫實體伺服器，[請使用這些指示](physical-azure-set-up-source.md#set-up-the-source-environment)。
- [深入了解](vmware-azure-set-up-target.md)目標設定。
- [取得更多關於設定複寫原則的資訊](vmware-azure-set-up-replication.md)。
- [了解](vmware-azure-enable-replication.md)如何啟用複寫，以及[排除](vmware-azure-exclude-disk.md)磁碟不要複寫。


## <a name="something-went-wrong-how-do-i-troubleshoot"></a>發生錯誤，如何進行疑難排解？

- 第一個步驟，嘗試[監視您的部署](site-recovery-monitor-and-troubleshoot.md)來確認所複寫項目、作業和基礎結構問題的狀態，並找出任何錯誤。
- 如果您無法完成初始複寫，或進行中複寫的運作不如預期，[請檢閱這篇文章](vmware-azure-troubleshoot-replication.md)中的常見錯誤和疑難排解提示。
- 如果在您要複寫的機器上自動安裝行動服務時遇到問題，請檢閱[這篇文章](vmware-azure-troubleshoot-push-install.md)中的常見錯誤。
- 如果容錯移轉的運作不如預期，請查看[這篇文章](site-recovery-failover-to-azure-troubleshoot.md)中的常見錯誤。
- 如果容錯回復無法運作，請查看您的問題是否出現於[這篇文章](vmware-azure-troubleshoot-failback-reprotect.md)中。



## <a name="next-steps"></a>後續步驟

複寫到位後，您應該[執行災害復原演練](tutorial-dr-drill-azure.md)，確保容錯移轉能如預期般運作。 
