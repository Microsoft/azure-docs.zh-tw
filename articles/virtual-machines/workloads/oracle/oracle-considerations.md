---
title: Microsoft Azure 上的 Oracle 解決方案 | Microsoft Docs
description: 了解 Microsoft Azure 上支援的 Oracle 解決方案組態和限制。
services: virtual-machines-linux
documentationcenter: ''
author: romitgirdhar
manager: jeconnoc
tags: azure-resource-management
ms.assetid: 5d71886b-463a-43ae-b61f-35c6fc9bae25
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 08/02/2018
ms.author: rogirdh
ms.openlocfilehash: 6435c866f6cdf5abea3862a718579f3a6e4d7378
ms.sourcegitcommit: eaad191ede3510f07505b11e2d1bbfbaa7585dbd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39493059"
---
# <a name="oracle-solutions-and-their-deployment-on-microsoft-azure"></a>Microsoft Azure 上的 Oracle 解決方案及其部署
本文涵蓋在 Microsoft Azure 上成功部署各種 Oracle 解決方案的必要資訊。 這些解決方案以 Oracle 在 Azure Marketplace 中發佈的虛擬機器映像為基礎。 若要取得目前可用的映像清單，請執行下列命令：
```azurecli-interactive
az vm image list --publisher oracle -o table --all
```
下列為截至 2017 年 6 月 16 日的映像清單：
```bash
Offer                   Publisher    Sku                     Urn                                                          Version
----------------------  -----------  ----------------------  -----------------------------------------------------------  -------------
Oracle-Database-Ee      Oracle       12.1.0.2                Oracle:Oracle-Database-Ee:12.1.0.2:12.1.20170202             12.1.20170202
Oracle-Database-Se      Oracle       12.1.0.2                Oracle:Oracle-Database-Se:12.1.0.2:12.1.20170202             12.1.20170202
Oracle-Linux            Oracle       6.4                     Oracle:Oracle-Linux:6.4:6.4.20141206                         6.4.20141206
Oracle-Linux            Oracle       6.7                     Oracle:Oracle-Linux:6.7:6.7.20161007                         6.7.20161007
Oracle-Linux            Oracle       6.8                     Oracle:Oracle-Linux:6.8:6.8.20161020                         6.8.20161020
Oracle-Linux            Oracle       6.9                     Oracle:Oracle-Linux:6.9:6.9.20170406                         6.9.20170406
Oracle-Linux            Oracle       7.0                     Oracle:Oracle-Linux:7.0:7.0.20141217                         7.0.20141217
Oracle-Linux            Oracle       7.2                     Oracle:Oracle-Linux:7.2:7.2.20161020                         7.2.20161020
Oracle-Linux            Oracle       7.3                     Oracle:Oracle-Linux:7.3:7.3.20170320                         7.3.20170320
Oracle-WebLogic-Server  Oracle       Oracle-WebLogic-Server  Oracle:Oracle-WebLogic-Server:Oracle-WebLogic-Server:12.1.2  12.1.2
```

系統會將這些映像視為「自備授權」，因此您只會支付執行 VM 產生的計算成本、儲存成本和網路成本。  其假設您已取得使用 Oracle 軟體的適當授權，且與 Oracle 之間已擁有支援合約。 Oracle 已保證從內部部署至 Azure 的授權行動性。 如需有關授權行動性的詳細資訊，請參閱已發佈的 [Oracle 和 Microsoft](http://www.oracle.com/technetwork/topics/cloud/faq-1963009.html) (英文) 附註。 

個人也可以選擇將其在 Azure 中從頭建立的自訂映像，或從內部部署環境上傳的自訂映像作為解決方案的基礎。

## <a name="support-for-jd-edwards"></a>JD Edwards 的支援
根據 Oracle 支援附註 [Doc ID 2178595.1](https://support.oracle.com/epmos/faces/DocumentDisplay?_afrLoop=573435677515785&id=2178595.1&_afrWindowMode=0&_adf.ctrl-state=o852dw7d_4)，符合其特定 `Minimum Technical Requirements` (MTR) 的**任何公用雲端供應項目**都支援 JD Edwards EnterpriseOne 9.2 版和更新版本。  您必須建立符合其 OS 和軟體應用程式相容性之 MTR 規格的自訂映像。 

## <a name="oracle-database-virtual-machine-images"></a>Oracle 資料庫虛擬機器映像
Oracle 支援在以 Oracle Linux 為基礎的虛擬機器映像上，在 Azure 中執行 Oracle DB 12.1 Standard 和 Enterprise 版本。  若要讓 Azure 上的 Oracle DB 生產工作負載提供最佳效能，請務必適當地調整 VM 映像的大小，並使用進階儲存體支援的受控磁碟。 如需有關如何使用 Oracle 已發佈之 VM 映像在 Azure 中快速地讓 Oracle DB 啟動並執行的指示，[請嘗試 Oracle DB 快速入門逐步解說](oracle-database-quick-create.md)。

### <a name="attached-disk-configuration-options"></a>連接的磁碟組態選項

連接的磁碟依賴 Azure Blob 儲存體服務。 理論上每個標準磁碟最多都能達約每秒 500 輸入/輸出作業 (IOPS)。 我們的進階磁碟供應項目十分適用於高效能資料庫工作附載，且可達到每部磁碟最多 5000 IOPS。 如果一個磁碟就能符合您的效能需求，則您可以僅使用單一磁碟，然而，透過使用多個連結的磁碟，並將資料庫資料分散至其間，然後使用 Oracle Automatic Storage Management (ASM)，通常就能改善有效的 IOPS 效能。 如需有關 Oracle ASM 的特定詳細資訊，請參閱 [Oracle Automatic Storage 概觀](http://www.oracle.com/technetwork/database/index-100339.html) (英文)。 如需如何在 Linux Azure VM 上安裝和設定 Oracle ASM 的範例，您可以嘗試[安裝和設定 Oracle Automatic Storage Management](configure-oracle-asm.md) 教學課程。

## <a name="oracle-real-application-cluster-oracle-rac"></a>Oracle Real Application Cluster (Oracle RAC)
Oracle RAC 是用來在內部部署的多節點叢集組態中，減少發生單一節點錯誤的機率。 它依賴以下兩個內部部署技術，而這兩個技術並非大規模公用雲端環境的原生技術︰網路多點傳送和共用磁碟。 若您的資料庫解決方案需要位在 Azure 中的 Oracle RAC，則您會需要第三方軟體才能啟用這些技術。  在 Microsoft Azure Marketplace 中提供經 **Microsoft Azure 認證**，名為[適用於 Oracle RAC 的 FlashGrid 節點](https://azuremarketplace.microsoft.com/marketplace/apps/flashgrid-inc.flashgrid-racnode?tab=Overview)的供應項目，是由 FlashGrid Inc. 所發行。如需此解決方案的詳細資訊，及其在 Azure 中的運作方式，請參閱 [FlashGrid 解決方案頁面](https://www.flashgrid.io/oracle-rac-in-azure/)。

## <a name="high-availability-and-disaster-recovery-considerations"></a>高可用性和災害復原考量
在 Azure 中使用 Oracle 資料庫時，您必須負責實作高可用性和災害復原解決方案，以避免任何停機。 

在 Azure 上可以使用 [Data Guard、Active Data Guard](http://www.oracle.com/technetwork/articles/oem/dataguardoverview-083155.html) 或 [Oracle Golden Gate](http://www.oracle.com/technetwork/middleware/goldengate) 來達成 Oracle Database Enterprise Edition 的高可用性和災害復原 (不需倚賴 Oracle RAC)，且兩個資料庫位於兩個不同的虛擬機器上。 這兩個虛擬機器應該位於相同的[虛擬網路](https://azure.microsoft.com/documentation/services/virtual-network/)中，以確保它們可以透過永續私人 IP 位址互相存取。  此外，建議您將虛擬機器放置於相同的可用性設定組，讓 Azure 將其放置於不同的容錯網域和升級網域。  如果您想要擁有異地備援，您可以讓這兩個資料庫在兩個不同的區域之間進行複寫，並透過 VPN 閘道連接兩個執行個體。

我們的教學課程「[在 Azure 上實作 Oracle DataGuard](configure-oracle-dataguard.md)」會引導您完成在 Azure 上試用此實作的基本安裝程序。  

使用 Oracle Data Guard，可以藉由某個虛擬機器中的主要資料庫、另一個虛擬機器中的次要 (待命) 資料庫以及它們之間的單向複寫設定，達到高可用性。 這樣讀取作業存取的會是資料庫的複本。 使用 Oracle GoldenGate，您則可以設定兩個資料庫之間的雙向複寫。 若要了解如何使用這些工具為資料庫設定高可用性解決方案，請參閱 Oracle 網站上的 [Active Data Guard](http://www.oracle.com/technetwork/database/features/availability/data-guard-documentation-152848.html) 和 [GoldenGate](http://docs.oracle.com/goldengate/1212/gg-winux/index.html) 文件 (英文)。 如需資料庫複本的讀取-寫入存取權，您可以使用 [Oracle Active Data Guard](http://www.oracle.com/uk/products/database/options/active-data-guard/overview/index.html)。

我們的教學課程「[在 Azure 上實作 Oracle GoldenGate](configure-oracle-golden-gate.md)」會引導您完成在 Azure 上試用此實作的基本安裝程序。

儘管已在 Azure 中建構 HA 和 DR 解決方案架構，您仍需要確定擁有可用於還原資料庫的備份策略。  我們的教學課程「[備份及復原 Oracle 資料庫](oracle-backup-recovery.md)」會引導您完成建立一致性備份的基本程序。

## <a name="oracle-weblogic-server-virtual-machine-images"></a>Oracle WebLogic Server 虛擬機器映像
* **只有在 Enterprise Edition 上才支援叢集。** 只有當您使用的是 WebLogic Server Enterprise Edition 時，才會獲授權使用 WebLogic 叢集。 請勿在使用 WebLogic Server Standard Edition 時使用叢集。
* **不支援 UDP 多點傳送。** Azure 支援 UDP 單點傳播，但不支援多點傳送或廣播。 WebLogic Server 可以依賴 Azure 的 UDP 單點傳播功能。 若要依賴 UDP 單點傳播獲得最佳結果，建議 WebLogic 叢集大小保持靜態，或叢集中保持不超過 10 個受控伺服器。
* **WebLogic Server 預期針對 T3 存取的公用和私人連接埠是相同的 (例如，使用 Enterprise JavaBeans 時)。** 請考慮使用多層式案例，其中服務層 (EJB) 應用程式會在 WebLogic Server 叢集上執行，而該叢集包含兩個或多個 VM，且都位於名為 **SLWLS** 的 vNet 中。 用戶端層位於相同 vNet 中的不同子網路，並執行簡單的 Java 程式，嘗試呼叫服務層中的 EJB。 由於負載平衡服務層是必要動作，所以必須先為 WebLogic Server 叢集中的虛擬機器建立公用負載平衡端點。 如果您指定的私人連接埠與公用連接埠不同 (例如，7006:7008)，則會發生下列錯誤：

       [java] javax.naming.CommunicationException [Root exception is java.net.ConnectException: t3://example.cloudapp.net:7006:

       Bootstrap to: example.cloudapp.net/138.91.142.178:7006' over: 't3' got an error or timed out]

   這是因為對於任何遠端 T3 存取，WebLogic Server 預期負載平衡器連接埠和 WebLogic 受控伺服器連接埠是相同的。 在上述情況中，用戶端正在存取連接埠 7006 (負載平衡器連接埠)，而受控伺服器正在接聽 7008 (私人連接埠)。 這項限制只適用於 T3 存取，不適用於 HTTP。

   若要避免此問題，請使用下列其中一種因應措施：

  * 針對 T3 存取專用的負載平衡端點使用相同的私人和公用連接埠號碼。
  * 啟動 WebLogic Server 時包含下列 JVM 參數：

         -Dweblogic.rjvm.enableprotocolswitch=true

如需相關資訊，請參閱位於 <http://support.oracle.com> 的 KB 文章 **860340.1**。

* **動態叢集和負載平衡限制。** 假設您想要在 WebLogic Server 中使用動態叢集，並透過 Azure 中的單一、公用負載平衡端點將它公開。 做法是只要您使用各個受控伺服器的固定連接埠號碼 (不是從範圍動態指派)，且不要啟動比系統管理員正在追蹤之機器的數量還多的受控伺服器 (也就是每台虛擬機器不超過一個受控伺服器)。 若您的設定會導致啟動的 WebLogic 伺服器多於可用的虛擬機器 (也就是多個 WebLogic Server 執行個體共用相同的虛擬機器)，則那些 WebLogic 伺服器執行個體中只有一個可以繫結到指定的連接埠號碼，該虛擬機器上的其他執行個體則會失敗。

   若您設定管理伺服器自動將唯一的連接埠號碼指派給其受控伺服器，則無法進行負載平衡，因為 Azure 不支援從單一公用連接埠對應至多個私人連接埠，而此設定中需要這樣做。
* **虛擬機器上的多個 Weblogic Server 執行個體。** 根據您的部署需求，如果虛擬機器夠大，您可以考慮選擇在相同虛擬機器上執行多個 WebLogic Server 的執行個體。 例如，在中型大小虛擬機器上 (包含 2 個核心)，您可以選擇執行兩個 WebLogic Server 的執行個體。 不過請注意，仍然建議您避免將單一失敗點引入您的架構，如果您僅使用一個執行多個 WebLogic Server 執行個體的虛擬機器也是如此。 使用至少兩個虛擬機器可能是較好的方法，每個虛擬機器可以執行多個 WebLogic Server 執行個體。 每個 WebLogic Server 的執行個體仍可以是相同叢集的一部分。 但是請注意，目前無法使用 Azure 對相同虛擬機器內此類 WebLogic Server 部署所公開的端點進行負載平衡，因為 Azure 負載平衡器需要負載平衡伺服器在唯一的虛擬機器內散佈。

## <a name="oracle-jdk-virtual-machine-images"></a>Oracle JDK 虛擬機器映像
* **JDK 6 和 7 最新更新。** 雖然建議使用 Java 的最新公開支援版本 (目前為 Java 8)，Azure 也提供 JDK 6 和 7 映像。 這是針對尚未準備升級至 JDK 8 的舊版應用程式。 雖然舊版 JDK 映像的更新可能不再提供一般大眾使用，憑藉 Microsoft 與 Oracle 的合作夥伴關係，Azure 提供的 JDK 6 和 7 映像會包含最新的非公用更新，該更新通常是由 Oracle 僅提供給 Oracle 支援客戶的選取群組。 新版本的 JDK 映像與 JDK 6 和 7 的更新版本會在一段時間內供取用。

   此 JDK 6 和 7 映像中可用的 JDK，以及從中衍生的虛擬機器和映像，只能在 Azure 內使用。
* **64 位元 JDK。** Oracle WebLogic Server 虛擬機器映像和 Azure 所提供的 Oracle JDK 虛擬機器映像包含 Windows Server 和 JDK 的 64 位元版本。

## <a name="next-steps"></a>後續步驟
您現在已大致了解 Microsoft Azure 上目前的 Oracle 解決方案。 下一個步驟是在 Azure 上部署第一個 Oracle 資料庫。
- 請嘗試[在 Azure 上建立 Oracle 資料庫](oracle-database-quick-create.md)教學課程以開始進行。