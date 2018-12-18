---
title: Azure 上的 SAP HANA (大型執行個體) 災害復原準則和準備工作 | Microsoft Docs
description: Azure 上的 SAP HANA (大型執行個體) 災害復原準則和準備工作
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
ms.openlocfilehash: ff214460d919eff5c3c1a2e608958673867ddc55
ms.sourcegitcommit: 794bfae2ae34263772d1f214a5a62ac29dcec3d2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44492811"
---
# <a name="disaster-recovery-principles"></a>災害復原原則

HANA 大型執行個體提供了能在不同 Azure 區域的 HANA 大型執行個體戳記間執行的災害復原功能。 例如，如果您將 HANA 大型執行個體單位部署在 Azure 的美國西部區域，即可使用美國東部區域的 HANA 大型執行個體單位來作為災害復原單位。 如先前所述，災害復原功能不會自動進行設定，因為如果要使用此功能，您還必須支付 DR 區域中的另一個 HANA 大型執行個體單位。 災害復原設定適用於相應增加以及相應放大設定。 

在截至目前為止所部署的案例中，客戶會使用 DR 區域中的單位來執行非生產系統，這些系統會使用已安裝的 HANA 執行個體。 HANA 大型執行個體單位的 SKU 必須與生產用途所使用的 SKU 相同。 Azure 生產區域與災害復原區域中的伺服器單位之間的磁碟組態如下圖所示：

![從磁碟觀點來看的 DR 設定組態](./media/hana-overview-high-availability-disaster-recovery/disaster_recovery_setup.PNG)

如此概觀圖所示，您接著需要排序第二組的磁碟區。 對災害復原單位中的生產執行個體而言，目標磁碟區與生產磁碟區的大小相同。 這些磁碟區會與災害復原網站中的 HANA 大型執行個體伺服器單位產生關聯。 下列磁碟區會從生產區域複寫至 DR 網站：

- /hana/data
- /hana/logbackups 
- /hana/shared (包括 /usr/sap)

因為從這些磁碟區進行還原時不需要 SAP HANA 交易記錄，所以不會複寫 /hana/log 磁碟區。 

所提供的災害復原功能基礎，是 HANA 大型執行個體基礎結構所提供的儲存體複寫功能。 儲存體端所使用的這項功能不會在存放磁碟區發生變更時，以非同步方式連續不斷地複寫變更。 相反地，這項機制要仰賴「系統會定期對這些磁碟區建立快照集」的事實。 已複寫的快照集和尚未複寫的新快照集之間的差異，接著會傳送至災害復原網站的目標磁碟區。  這些快照集會儲存在磁碟區上，如果出現災害復原移轉，就需要在這些磁碟區上還原快照集。  

您必須在快照集之間的差異資料量變小之前，第一次將完整的磁碟區資料傳送出去。 因此，DR 網站中的磁碟區會包含生產網站中所執行之磁碟區快照集的每個人。 最終，您可以使用該 DR 系統來取得先前的狀態，以復原遺失的資料，而又不必復原生產系統。

如果在單一 HANA 大型執行個體單位上有多個獨立 SAP HANA 執行個體進行 MCOD 部署，則所有的 SAP HANA 執行個體皆會將儲存體複寫至 DR 網站。

如果您在生產網站中使用 HANA 系統複寫作為高可用性功能，並且對 DR 網站使用以儲存體為基礎的複寫，則兩個節點的磁碟區都會從主要網站複寫到 DR 執行個體。 您必須在 DR 網站購買額外的儲存體 (大小與主要節點相同)，以容納從主要和次要節點複寫到 DR 的資料。 



>[!NOTE]
>HANA 大型執行個體儲存體複寫功能會鏡像處理及複寫儲存體快照集。 如果您未依照本文的[備份與還原](#backup-and-restore)章節所說的方式來執行儲存體快照集，系統就完全不會複寫到災害復原網站。 若要執行儲存體複寫到災害復原網站的作業，就必須先執行儲存體快照集。



## <a name="preparation-of-the-disaster-recovery-scenario"></a>災害復原案例的準備工作
在本節中，您具有在生產 Azure 區域中的 HANA 大型執行個體上執行的生產系統。 針對下面步驟，讓我們假設 HANA 系統的 SID 是 "PRD"，以及您有個非生產系統執行於 DR Azure 區域中的 HANA 大型執行個體上。 對於後者，讓我們假設其 SID 就是 "TST"。 下圖顯示此組態：

![DR 設定的開頭](./media/hana-overview-high-availability-disaster-recovery/disaster_recovery_start1.PNG)

如果伺服器執行個體尚未與額外的存放磁碟區組一起排序，則「SAP HANA on Azure 服務管理」會將一組額外的磁碟區作為生產複本的目標，連結至執行 TST HANA 執行個體所在的 HANA 大型執行個體單位。 基於這個目的，您需要提供生產 HANA 執行個體的 SID。 在「SAP HANA on Azure 服務管理」確認這些磁碟區的連結之後，您需要將這些磁碟區掛接到 HANA 大型執行個體單位。

![DR 設定的下一個步驟](./media/hana-overview-high-availability-disaster-recovery/disaster_recovery_start2.PNG)

下一個步驟是在 DR Azure 區域 (您在其中執行 TST HANA 執行個體) 中的 HANA 大型執行個體單位上，安裝第二個 SAP HANA 執行個體。 新安裝的 SAP HANA 執行個體必須具有相同的 SID。 所建立的使用者必須具有與生產執行個體相同的 UID 和群組識別碼。 如需詳細資訊，請參閱[備份和還原](hana-backup-restore.md)。 如果安裝成功，您必須：

- 執行[備份與還原](hana-backup-restore.md)中所述的儲存體快照集準備工作步驟 2。
- 如果您未執行此步驟，請針對 HANA 大型執行個體單位的 DR 單位建立公開金鑰。 請參閱[備份與還原](hana-backup-restore.md)一文中所述的儲存體快照集準備工作步驟 3。
- 使用新的 HANA 執行個體維護 *HANABackupCustomerDetails.txt*，並測試儲存體連線是否正常運作。  
- 在 DR Azure 區域中的 HANA 大型執行個體單位上，停止新安裝的 SAP HANA 執行個體。
- 卸載這些 PRD 磁碟區並連絡「SAP HANA on Azure 服務管理」。 磁碟區無法一直掛接至此單位，因為它們在當作儲存體複寫目標時無法加以存取。  

![建立複寫之前的 DR 設定步驟](./media/hana-overview-high-availability-disaster-recovery/disaster_recovery_start3.PNG)

營運小組會建立生產 Azure 區域中的 PRD 磁碟區與 DR Azure 區域中的 PRD 磁碟區之間的複寫關聯性。

>[!IMPORTANT]
>不會複寫 /hana/log 磁碟區，因為不必將所複寫的 SAP HANA 資料庫還原到災害復原網站中的一致狀態。

下一步，設定或調整儲存體快照集備份排程，以在災害情況下取得您的 RTO 和 RPO。 若要盡量縮減復原點目標，請在 HANA 大型執行個體服務中設定下列複寫間隔：
- 針對合併快照集 (快照集類型為 **hana**) 所涵蓋的磁碟區，設定為每隔 15 分鐘就會複寫到災害復原網站中對等的存放磁碟區目標。
- 針對交易記錄備份磁碟區 (快照集類型為 **logs**)，設定為每隔 3 分鐘就會複寫到災害復原網站中對等的存放磁碟區目標。

若要盡量縮減復原點目標，請設定下列各項：
- 每隔 30 分鐘到 1 小時執行一次 **hana** 類型的儲存體快照集 (請參閱「步驟 7：執行快照集」)。
- 每隔 5 分鐘執行一次 SAP HANA 交易記錄備份。
- 每隔 5-15 分鐘執行一次 **logs** 類型的儲存體快照集。 使用此間隔期間，您會達到約 15-25 分鐘的 RPO。

若您使用此設定，HANA 交易記錄備份磁碟區、/hana/data 和 /hana/shared (包括 /usr/sap) 的交易記錄備份、儲存體快照集和複寫的順序會如下圖所示：

 ![時間軸上交易記錄備份快照集與快照鏡像之間的關聯性](./media/hana-overview-high-availability-disaster-recovery/snapmirror.PNG)

若要在災害復原案例中達成更好的 RPO，您可以從 SAP HANA on Azure (大型執行個體) 將 HANA 交易記錄備份複製到其他 Azure 區域。 若要再縮減此 RPO，請執行下列步驟：

1. 儘可能經常將 HANA 交易記錄複製到 /hana/logbackups。
1. 使用 rsync 將交易記錄備份複製到 NFS 共用裝載的 Azure 虛擬機器。 VM 位於 Azure 生產區域和 DR 區域中的 Azure 虛擬網路。 您需要將這兩個 Azure 虛擬網路連線到可將生產 HANA 大型執行個體連線到 Azure 的線路。 請參閱[使用 HANA 大型執行個體進行災害復原的網路考量](#Network-considerations-for-disaster recovery-with-HANA-Large-Instances)一節中的圖形。 
1. 將交易記錄備份保留在該區域內 VM 所連結的 NFS 匯出儲存體中。
1. 在災害容錯移轉案例中，為您在 /hana/logbackups 磁碟區所找到的交易記錄備份，補充您在災害復原網站的 NFS 共用上所擷取的較新交易記錄備份。 
1. 可以開始將交易記錄備份還原至可能儲存到 DR 區域的最新備份。

當 HANA 大型執行個體作業確認已設定複寫關聯性並開始執行儲存體快照集備份時，資料就會開始進行複寫。

![建立複寫之前的 DR 設定步驟](./media/hana-overview-high-availability-disaster-recovery/disaster_recovery_start4.PNG)

進行複寫時，不會還原 DR Azure 區域中 PRD 磁碟區上的快照集。 只會儲存這些快照集。 如果磁碟區是在此種狀態下掛接，則代表在 PRD SAP HANA 執行個體安裝於 DR Azure 區域的伺服器單位之後，您卸載這些磁碟區的狀態。 它們也代表尚未還原的儲存體備份。

如果出現容錯移轉，您也可以選擇還原到較舊的儲存體快照集，而不是最新的儲存體快照集。

## <a name="next-steps"></a>後續步驟

- 請參閱[災害復原容錯移轉程序](hana-failover-procedure.md) \(英文\)。