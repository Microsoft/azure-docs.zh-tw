---
title: Azure Site Recovery 可以保護哪些工作負載？ | Microsoft Docs
description: 描述可以透過 Azure Site Recovery 服務使用災害復原來保護的工作負載。
services: site-recovery
author: rayne-wiselman
ms.service: site-recovery
ms.topic: conceptual
ms.date: 07/22/2018
ms.author: raynew
ms.openlocfilehash: 5295002e823dc864f0f0dcc67594f49b5431bce7
ms.sourcegitcommit: 30221e77dd199ffe0f2e86f6e762df5a32cdbe5f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/23/2018
ms.locfileid: "39205947"
---
# <a name="what-workloads-can-you-protect-with-azure-site-recovery"></a>Azure Site Recovery 可以保護哪些工作負載？

本文說明您可以使用 [Azure Site Recovery](site-recovery-overview.md) 服務進行複寫的工作負載和應用程式。



## <a name="overview"></a>概觀

組織需要商務持續性和災害復原 (BCDR) 策略，以便讓工作負載和資料在計劃性停機與非計劃性停機期間維持安全，並儘速復原到一般運作的情況。

Site Recovery 是一項有助於建立 BCDR 策略的 Azure 服務。 使用 Site Recovery，您可以將應用程式感知的複寫部署到雲端，或部署到次要站台。 不論您的應用程式是 Windows 或 Linux 型、在實體伺服器、VMware 或 Hyper-V 上執行，您都可以使用 Site Recovery 來協調複寫、執行災害復原測試，以及執行容錯移轉和容錯回復。

Site Recovery 整合了 Microsoft 應用程式，包括 SharePoint、Exchange、Dynamics、SQL Server 和 Active Directory。 Microsoft 也與領導廠商密切合作，包括 Oracle、SAP 和 Red Hat。 您可以根據各應用程式自訂複寫解決方案。

## <a name="why-use-site-recovery-for-application-replication"></a>為何要使用 Site Recovery 進行應用程式複寫？

Site Recovery 可協助您實作應用程式層級的保護和復原，如下所示：

* 應用程式無從驗證，針對在受支援電腦上執行的任何工作負載提供複寫。
* 僅需 30 秒即可完成 PRO 近同步複寫，能滿足大多數重要商務應用程式的需求。
* 適用於單一或多層式架構應用程式的應用程式一致性快照。
* 整合 SQL Server AlwaysOn，並與其他應用程式層級的複寫技術合作，包括 AD 複寫、SQL AlwaysOn、Exchange 資料庫可用性群組 (DAG) 和 Oracle 資料保護。
* 彈性修復計劃，讓您只要按一下就能復原整個應用程式堆疊，並在方案中納入外部指令碼和手動動作。
* Site Recovery 和 Azure 中的進階網路管理可簡化應用程式網路需求，包括保留 IP 位址、設定負載平衡，以及與 Azure 流量管理員整合而達到低 RTO 網路轉換。
* 豐富的自動化程式庫，提供已可用於生產環境的應用程式特定指令碼，這些指令碼可供下載並與復原方案整合。

## <a name="workload-summary"></a>工作負載摘要
Site Recovery 可複寫在支援的機器上執行的任何應用程式。 此外，我們已經與產品團隊合作，合力執行額外的特定應用程式測試。

| **工作負載** |**將 Azure VM 複寫至 Azure** |**將 Hyper-V VM 複寫至次要網站** | **將 Hyper-V VM 複寫至 Azure** | **將 VMware VM 複寫到次要網站** | **將 VMware VM 複寫到 Azure** |
| --- | --- | --- | --- | --- |---|
| Active Directory、DNS |Y |Y |Y |Y |Y|
| Web 應用程式 (IIS、SQL) |Y |Y |Y |Y |Y|
| System Center Operations Manager |Y |Y |Y |Y |Y|
| Sharepoint |Y |Y |Y |Y |Y|
| SAP<br/><br/>將 SAP 網站複寫至非叢集的 Azure |Y (由 Microsoft 測試) |Y (由 Microsoft 測試) |Y (由 Microsoft 測試) |Y (由 Microsoft 測試) |Y (由 Microsoft 測試)|
| Exchange (非 DAG) |Y |Y |Y |Y |Y|
| 遠端桌面/VDI |Y |Y |Y |Y |Y|
| Linux (作業系統和應用程式) |Y (由 Microsoft 測試) |Y (由 Microsoft 測試) |Y (由 Microsoft 測試) |Y (由 Microsoft 測試) |Y (由 Microsoft 測試)|
| Dynamics AX |Y |Y |Y |Y |Y|
| Windows 檔案伺服器 |Y |Y |Y |Y |Y|
| Citrix XenApp 和 XenDesktop |Y|N/A |Y |N/A |Y |

## <a name="replicate-active-directory-and-dns"></a>複寫 Active Directory 和 DNS
Active Directory 和 DNS 基礎結構是大多數企業應用程式的必要項目。 在災害復原期間，您必須在復原工作負載和應用程式之前，先保護與復原這些基礎結構元件。

您可以使用 Site Recovery，對 Active Directory 和 DNS 建立一個完整的自動化災害復原方案。 例如，如果您想要將 SharePoint 和 SAP 從主要網站容錯移轉至次要網站，您可以先設定可容錯移轉 Active Directory 的復原方案，然後再設定額外的應用程式專屬方案，以便容錯移轉依賴 Active Directory 的其他應用程式。

[深入了解](site-recovery-active-directory.md) 如何保護 Active Directory 和 DNS。

## <a name="protect-sql-server"></a>保護 SQL Server
SQL Server 針對內部部署資料中心內許多商務應用程式，提供資料服務的資料服務基礎。  Site Recovery 可與 SQL Server HA/DR 技術一起用來保護採用 SQL Server 的多層式企業應用程式。 Site Recovery 提供：

* 針對 SQL Server 提供簡單且符合成本效益的災害復原解決方案。 將多個版本的 SQL Server 獨立伺服器和叢集複寫至 Azure 或次要網站。  
* 整合 SQL AlwaysOn 可用性群組，利用 Azure Site Recovery 復原方案管理容錯移轉和容錯回復。
* 應用程式中各層 (包括 SQL Server 資料庫) 的端對端復原方案。
* 藉由將 SQL Server「暴增」到 Azure 中較大的 IaaS 虛擬機器中，利用 Azure Site Recovery 調整 SQL Server 以因應尖峰負載。
* 簡單的 SQL Server 災害復原測試。 您可以測試容錯移轉，以分析資料並執行相容性檢查，但不會影響您的生產環境。

[深入了解](site-recovery-sql.md) 如何保護 SQL Server。

## <a name="protect-sharepoint"></a>保護 SharePoint
Azure Site Recovery 可協助保護 SharePoint 部署，如下所示：

* 排除對災害復原之待命伺服器陣列的需要和相關聯的基礎結構成本。 使用 Site Recovery，將整個伺服器陣列 (Web、應用程式和資料庫層) 複寫至 Azure 或次要網站。
* 簡化應用程式部署和管理作業。 部署至主要網站的更新會自動複寫，因此可在伺服器陣列於次要網站中容錯移轉和復原之後使用。 此外，還可降低讓待命伺服器陣列保持最新狀態的管理複雜度和相關成本。
* 藉由建立類似生產的複本隨選複本環境進行測試和偵錯，以簡化 SharePoint 應用程式開發和測試。
* 藉由使用 Site Recovery 將 SharePoint 部署移轉至 Azure，來簡化雲端轉換。

[深入了解](site-recovery-sharepoint.md) 如何保護 SharePoint。

## <a name="protect-dynamics-ax"></a>保護 Dynamics AX
Azure Site Recovery 可協助保護您的 Dynamics AX ERP 解決方案，方法如下：

* 協調將整個 Dynamics AX 環境 (Web 和 AOS 層、資料庫層、SharePoint) 複寫到 Azure，或是次要網站。
* 簡化 Dynamics AX 部署到雲端 (Azure) 的移轉。
* 藉由建立類似生產的複本隨選進行測試和偵錯，以簡化 Dynamics AX 應用程式開發和測試。

[深入了解](site-recovery-dynamicsax.md) 如何保護動態 Dynamic AX。

## <a name="protect-rds"></a>保護 RDS
遠端桌面服務 (RDS) 可啟用虛擬桌面基礎結構 (VDI)、以工作階段為基礎的桌面和應用程式，讓使用者可在任何地方工作。 使用 Azure Site Recovery，您可以：

* 將受控或非受控集區化虛擬桌面複寫至次要網站，以及將遠端應用程式和工作階段複寫至次要網站或 Azure。

* 以下是您可以複寫的項目︰

| **RDS** |**將 Azure VM 複寫至 Azure** | **將 Hyper-V VM 複寫至次要網站** | **將 Hyper-V VM 複寫至 Azure** | **將 VMware VM 複寫到次要網站** | **將 VMware VM 複寫到 Azure** | **將實體伺服器複寫到次要網站** | **將實體伺服器複寫到 Azure** |
|---| --- | --- | --- | --- | --- | --- | --- |
| **集區化虛擬桌面 (非受控)** |否|是 |否 |是 |否 |是 |否 |
| **集區化虛擬桌面 (受控但不含 UPD)** |否|是 |否 |是 |否 |是 |否 |
| **遠端應用程式和桌面工作階段 (不含 UPD)** |是|yes |yes |yes |yes |yes |是 |

[使用 Azure Site Recovery 設定適用於 RDS 的災害復原](https://docs.microsoft.com/windows-server/remote/remote-desktop-services/rds-disaster-recovery-with-azure)。

[深入了解](https://gallery.technet.microsoft.com/Remote-Desktop-DR-Solution-bdf6ddcb) 如何保護 RDS。

## <a name="protect-exchange"></a>保護 Exchange
Site Recovery 協助保護 Exchange 的方式如下所示：

* 對於小型 Exchange 部署 (例如單一或獨立式伺服器)，Site Recovery 可以複寫並容錯移轉至 Azure 或次要網站。
* 對於大型部署，Site Recovery 會與 Exchange DAG 整合。
* Exchange DAG 是企業中 Exchange 災害復原的建議解決方案。  Site Recovery 復原方案可以包含 DAG，以協調網站之間的 DAG 容錯移轉。

[深入了解](https://gallery.technet.microsoft.com/Exchange-DR-Solution-using-11a7dcb6) 如何保護 Exchange。

## <a name="protect-sap"></a>保護 SAP
使用 Site Recovery 保護您的 SAP 部署，如下所示：

* 藉由將元件複寫到 Azure，以啟用在內部部署環境執行之 SAP NetWeaver 和非 NetWeaver 生產應用程式的保護功能。
* 藉由將元件複寫到另一個 Azure 資料中心，啟用執行 Azure 之 SAP NetWeaver 和非 NetWeaver 生產應用程式的保護功能。
* 使用 Site Recovery 將您的 SAP 部署移轉至 Azure，來簡化雲端移轉。
* 藉由建立隨選生產複本來測試 SAP 應用程式，以簡化 SAP 專案升級、測試和原型設計。

[深入了解](site-recovery-sap.md) 如何保護 SAP。

## <a name="protect-iis"></a>保護 IIS
使用 Site Recovery 保護您的 IIS 部署，如下所示：

Azure Site Recovery 提供災害復原，方法為將環境中的重要元件複寫至冷遠端站台或類似 Microsoft Azure 的公用雲端。 具有 web 伺服器和資料庫的虛擬機器將複寫至復原站台，因此不需要分開備份組態檔或憑證。 取決於容錯移轉後之環境變數的應用程式對應和繫結，可以透過指令碼整合到災害復原計劃進行更新。 僅在容錯移轉時，虛擬機器才會帶出復原站台。 不只如此，Azure Site Recovery 也會提供下列功能，協助您協調端對端容錯移轉︰

-   排序各層中的虛擬機器啟動與關機。
-   新增指令碼，以在它們啟動後允許應用程式相依性更新以及虛擬機器上的繫結。 指令碼也可用來更新 DNS 伺服器指向復原站台。
-   在容錯移轉之前將 IP 位址配置給虛擬機器，方法為對應主要與復原網路，因此使用不需要在容錯移之後更新的指令碼。
-   能夠在 web 伺服器上一鍵容錯多個 web 應用程式，因此在災害發生時排除造成混淆的範圍。
-   能夠在 DR 鑽研的隔離環境中測試復原計劃。

[深入了解](https://aka.ms/asr-iis) 如何保護 IIS web 伺服陣列。

## <a name="protect-citrix-xenapp-and-xendesktop"></a>保護 Citrix XenApp 和 XenDesktop
使用 Site Recovery 保護您的 Citrix XenApp 和 XenDesktop 部署，如下所示：

* 啟用 Citrix XenApp 和 XenDesktop 部署的保護，做法是將不同的部署層 (包括 AD DNS 伺服器、SQL Database 伺服器、Citrix 傳遞控制站、StoreFront 伺服器，XenApp Master (VDA)、Citrix XenApp 授權伺服器) 複寫至 Azure。
* 使用 Site Recovery 將您的 Citrix XenApp 和 XenDesktop 部署移轉至 Azure，來簡化雲端移轉。
* 藉由建立類似生產的隨選複本來進行測試和偵錯，以簡化 Citrix XenApp/XenDesktop 測試。
* 此解決方案僅適用於 Windows Server 作業系統虛擬桌面，而不適用於用戶端虛擬桌面，因為尚未支援在 Azure 中進行用戶端虛擬桌面的授權。
[深入了解](https://azure.microsoft.com/pricing/licensing-faq/)如何在 Azure 中進行用戶端/伺服器桌面的授權。

[深入了解](site-recovery-citrix-xenapp-and-xendesktop.md)如何保護 Citrix XenApp 和 XenDesktop 部署。 或者，您可以參考詳述相同功能的 [Citrix 白皮書](https://aka.ms/citrix-xenapp-xendesktop-with-asr)。

## <a name="next-steps"></a>後續步驟

[開始使用](azure-to-azure-quickstart.md) Azure VM 複寫。
