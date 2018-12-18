---
title: SAP HANA on Azure (大型執行個體) 的高可用性和災害復原 | Microsoft Docs
description: 建立高可用性並為 SAP HANA on Azure (大型執行個體) 的災害復原做規劃
services: virtual-machines-linux
documentationcenter: ''
author: saghorpa
manager: jeconnoc
editor: ''
ms.service: virtual-machines-linux
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 09/10/2018
ms.author: saghorpa
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 61f536ee5eb27982bd63daf0b278e6c7a836fe08
ms.sourcegitcommit: 794bfae2ae34263772d1f214a5a62ac29dcec3d2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44390734"
---
# <a name="sap-hana-large-instances-high-availability-and-disaster-recovery-on-azure"></a>Azure 上 SAP Hana (大型執行個體) 的高可用性和災害復原 

>[!IMPORTANT]
>本說明文件不可用 SAP HANA 管理說明文件或 SAP 附註取代。 本文預期讀者對 SAP HANA 管理和操作有深入的了解和專業知識，尤其是關於備份、還原、高可用性和災害復原的主題。

請務必使用您的 HANA 版本和版次，在您的環境中執行步驟和程序。 為了能讓大眾更全面地了解，這份說明文件中所述的某些程序已簡化，但並非用做最終操作手冊的詳細步驟。 如果您想要為組態建立操作手冊，則需要測試及執行程序，並記錄與特定組態相關的處理程序。 


高可用性和災害復原 (DR) 對執行具任務關鍵性的 SAP HANA on Azure (大型執行個體) 伺服器來說，是極其重要的層面。 請務必與 SAP、您的系統整合者或 Microsoft 合作，以便正確建構及實作正確的高可用性和災害復原策略。 也務必考量您環境特有的復原點目標 (RPO) 和復原時間目標。

Microsoft 可透過 HANA 大型執行個體支援某些 SAP HANA 高可用性功能。 這些功能包括：

- **儲存體複寫：** 儲存體系統將所有資料複寫到另一個 Azure 區域中另一個 HANA 大型執行個體戳記的能力。 SAP HANA 獨立運作，不依賴此方法。 此功能是提供給 HANA 大型執行個體使用的預設災害復原機制。
- **HANA 系統複寫**：[將 SAP HANA 中的所有資料複寫](https://help.sap.com/viewer/6b94445c94ae495c83a19646e7c3fd56/2.0.01/en-US/b74e16a9e09541749a745f41246a065e.html)到個別的 SAP HANA 系統。 復原時間目標是透過定期的資料複寫而最小化。 SAP HANA 支援非同步、記憶體內同步及同步模式。 同步模式僅用於相同資料中心內或距離 100 公里內的 SAP HANA 系統。 以 HANA 大型執行個體戳記目前的設計而言，HANA 系統複寫只可用於單一區域內的高可用性。 HANA 系統複寫需要第三方反向 Proxy 或路由元件，以便在另一個 Azure 區域中進行災害復原設定。 
- **主機自動容錯移轉**︰SAP HANA 的本機錯誤復原解決方案，可作為 HANA 系統複寫的替代選項。 如果主要節點變得無法使用，您可以相應放大模式設定一或多個待命 SAP HANA 節點，而 SAP HANA 會自動容錯移轉到待命節點。

SAP HANA on Azure (大型執行個體) 會在四個地緣政治區域 (美國、澳洲、歐洲和日本) 中的兩個 Azure 區域提供。 地緣政治區域內裝載 HANA 大型執行個體戳記的兩個區域，會與個別的專用網路線路連線。 這會用來複寫儲存體快照集，以提供災害復原方法。 複寫不會在預設中建立，但會為訂購災害復原功能的客戶設定。 執行儲存體複寫需要使用 HANA 大型執行個體的儲存體快照集。 您無法選擇 Azure 區域作為 DR 區域，因為該區域位於不同的地緣政治地區。 

下表顯示目前支援的高可用性和災害復原方法以及兩者的組合：

| HANA 大型執行個體所支援的案例 | 高可用性選項 | 災害復原選項 | 註解 |
| --- | --- | --- | --- |
| 單一節點 | 無法使用。 | 專用 DR 設定。<br /> 多用途 DR 設定。 | |
| 主機自動容錯移轉：向外延展 (不一定有待命)<br /> 包括 1 + 1 | 可透過擔任作用中角色的待命節點來實現。<br /> HANA 會控制角色的切換。 | 專用 DR 設定。<br /> 多用途 DR 設定。<br /> 使用儲存體複寫進行的 DR 同步處理。 | HANA 磁碟區組會連接到所有節點。<br /> DR 網站必須具有相同數目的節點。 |
| HANA 系統複寫 | 可透過主要或次要設定來實現。<br /> 在容錯移轉的情況下，次要節點會轉而成為主要節點。<br /> HANA 系統複寫和 OS 控制容錯移轉。 | 專用 DR 設定。<br /> 多用途 DR 設定。<br /> 使用儲存體複寫進行的 DR 同步處理。<br /> 若沒有第三方元件，則還無法實現使用 HANA 系統複寫來進行的 DR。 | 另一組不同的磁碟區會連結至每個節點。<br /> 只有生產網站中次要複本的磁碟區會複寫到 DR 位置。<br /> DR 網站需要一組磁碟區。 | 

作為專用 DR 設定時，DR 網站中的 HANA 大型執行個體單位不會用於執行任何其他工作負載或非生產系統。 該單位是被動的，只會在災害容錯移轉執行時部署。 然而，此設定不是許多客戶偏好的選擇。

若要了解適用於您的架構的儲存配置和乙太網路詳細資料，請參閱 [HLI 支援案例](hana-supported-scenario.md)。

> [!NOTE]
> [SAP HANA MCOD 部署](https://launchpad.support.sap.com/#/notes/1681092) (一個單位上有多個 HANA 執行個體) 如表格中列出之 HA 與 DR 方法搭配使用的重疊情況所示。 根據 Pacemaker 將 HANA 系統複寫與自動容錯移轉叢集搭配使用的情況例外。 這種情況僅支援每個單位一個 HANA 執行個體。 對於 [SAP HANA MDC](https://launchpad.support.sap.com/#/notes/2096000) 部署，如果已部署多個租用戶，只有以非儲存體為基礎的 HA 和 DR 方法才會產生效用。 如果已部署一個租用戶，則所有列出的方法皆有效。  

作為多用途 DR 設定時，DR 網站上的 HANA 大型執行個體單位會執行非生產工作負載。 在災害情況下，關閉非生產系統，掛接儲存體複寫的 (其他) 磁碟區組，然後啟動生產 HANA 執行個體。 許多使用 HANA 大型執行個體災害復原功能的客戶都使用此組態。 


您可以在下列 SAP 文章中找到更多關於 SAP HANA 高可用性的資訊： 

- [SAP HANA 高可用性白皮書](http://go.sap.com/documents/2016/05/f8e5eeba-737c-0010-82c7-eda71af511fa.html)
- [SAP HANA 管理指南](http://help.sap.com/hana/SAP_HANA_Administration_Guide_en.pdf)
- [SAP HANA Academy 的 SAP HANA 系統複寫影片](http://scn.sap.com/community/hana-in-memory/blog/2015/05/19/sap-hana-system-replication)
- [SAP 支援附註 #1999880 - SAP HANA 系統複寫常見問題集](https://apps.support.sap.com/sap/support/knowledge/preview/en/1999880)
- [SAP 支援附註 #2165547 - SAP HANA 系統複寫環境內的 SAP HANA 備份與還原](https://websmp230.sap-ag.de/sap(bD1lbiZjPTAwMQ==)/bc/bsp/sno/ui_entry/entry.htm?param=69765F6D6F64653D3030312669765F7361706E6F7465735F6E756D6265723D3231363535343726)
- [SAP 支援附註 #1984882 - 使用 SAP HANA 系統複寫以最短/零停機時間的方式進行硬體交換](https://websmp230.sap-ag.de/sap(bD1lbiZjPTAwMQ==)/bc/bsp/sno/ui_entry/entry.htm?param=69765F6D6F64653D3030312669765F7361706E6F7465735F6E756D6265723D3139383438383226)

## <a name="network-considerations-for-disaster-recovery-with-hana-large-instances"></a>使用 HANA 大型執行個體之災害復原的網路考量

若要利用 HANA 大型執行個體的災害復原功能，您必須設計連到兩個 Azure 區域的網路連線。 您需要一條從主要 Azure 區域中的內部部署連接的 ExpressRoute 線路，還需要另一條從內部部署連接到災害復原區域的線路。 此措施可解決 Azure 區域 (包含 Microsoft Enterprise Edge (MSEE) 路由器位置) 發生問題的情況。

作為第二個措施，您可以將連線到一個區域中 SAP HANA on Azure (大型執行個體) 的所有 Azure 虛擬網路，都連線到與其他區域中的 HANA 大型執行個體連線的 ExpressRoute 線路。 透過該「交叉連線」，於區域 1 的 Azure 虛擬網路中執行的服務可以連線到區域 2 的 HANA 大型執行個體單位，反之亦然。 此措施可解決在 Azure 離線的情況下，只有一個 MSEE 位置會連線到內部部署位置的情形。

下圖說明災害復原的復原組態案例：

![災害復原的最佳組態](./media/hana-overview-high-availability-disaster-recovery/image1-optimal-configuration.png)



## <a name="other-requirements-with-hana-large-instances-storage-replication-for-disaster-recovery"></a>在災害復原中使用 HANA 大型執行個體儲存體複寫的其他需求

除了上述使用 HANA 大型執行個體之災害復原設定的需求，您必須：

- 訂購與生產 SKU 大小相同的 SAP HANA on Azure (大型執行個體) SKU，並部署在災害復原區域中。 在目前的客戶部署中，這些執行個體會用來執行非生產 HANA 執行個體。 這些設定稱為*多用途 DR 設定*。   
- 針對您在災害復原網站中想要復原的每個 SAP HANA on Azure (大型執行個體) SKU，在 DR 網站上為其訂購其他儲存體。 購買額外的儲存體，可讓您配置存放磁碟區。 您可以配置磁碟區，這些磁碟區是從生產 Azure 區域將儲存體複寫到災害復原 Azure 區域時的目標。
- 如果您在主要節點上設定 HSR，而且設定 DR 網站的儲存體式複寫，則必須在 DR 網站購買額外的儲存體，以便將主要和次要節點資料會複寫至 DR 網站。

 **後續步驟**
- 請參閱[備份與還原](hana-backup-restore.md)。













