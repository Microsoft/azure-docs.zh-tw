---
title: 單一 Azure 區域中的 SAP HANA 可用性 | Microsoft Docs
description: 說明單一 Azure 區域中 Azure 原生 VM 上的 SAP HANA 作業。
services: virtual-machines-linux,virtual-machines-windows
documentationcenter: ''
author: msjuergent
manager: patfilot
editor: ''
tags: azure-resource-manager
keywords: ''
ms.service: virtual-machines-linux
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 07/27/2018
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 4f2defd60ec6b835ec856c9253a92f1d6817e861
ms.sourcegitcommit: 7ad9db3d5f5fd35cfaa9f0735e8c0187b9c32ab1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/27/2018
ms.locfileid: "39325998"
---
# <a name="sap-hana-availability-within-one-azure-region"></a>單一 Azure 區域中的 SAP HANA 可用性
本文說明單一 Azure 區域中的幾種可用性案例。 Azure 有許多分散在世界各地的區域。 如需 Azure 區域的清單，請參閱 [Azure 區域](https://azure.microsoft.com/regions/)。 為了讓您能夠在單一 Azure 區域內的 VM 上部署 SAP HANA，Microsoft 提供了含一個 HANA 執行個體的單一 VM 部署。 如需提升可用性，您可以在 [Azure 可用性設定組](https://docs.microsoft.com/azure/virtual-machines/windows/tutorial-availability-sets)中部署含兩個 HANA 執行個體的兩個 VM，以便使用 HANA 系統複寫來獲得可用性。 

目前，Azure 提供的是 [Azure 可用性區域](https://docs.microsoft.com/azure/availability-zones/az-overview)。 本文不會詳細說明可用性區域。 但文章中會包含有關使用可用性設定組與可用性區域的一般討論。

有提供可用性區域的 Azure 區域會有多個資料中心。 這些資料中心有獨立供應的電源、冷卻系統和網路。 之所以在單一 Azure 區域 (region) 中提供不同區域 (zone)，是要讓您可以跨兩到三個所提供的可用性區域 (zone) 來部署應用程式。 透過跨區域 (zone) 部署，電力來源和網路中的問題就只會影響一個 Azure 可用性區域基礎結構，Azure 區域 (region) 內的應用程式部署仍然可完全正常運作。 但容量可能會降低。 例如，您可能會失去某個區域中的 VM，但另外兩個區域中的 VM 仍保持啟動並執行。 
 
Azure 可用性設定組是一種邏輯群組功能，有助於確保您放置在可用性設定組內的 VM 資源若是部署在 Azure 資料中心內，則不會因為失敗而對彼此造成影響。 Azure 可確保您在可用性設定組中所放置的 VM，會橫跨多部實體伺服器、計算機架、儲存體單位和網路交換器來執行。 在某些 Azure 文件中，這個設定是指不同[更新網域和容錯網域](https://docs.microsoft.com/azure/virtual-machines/windows/manage-availability)中的配置。 這些配置通常在 Azure 資料中心內。 假使電源和網路問題會影響您要部署的資料中心，則也會影響到您在單一 Azure 區域中的所有容量。

配置代表 Azure 可用性區域的資料中心，是在服務 (部署在不同區域) 之間傳遞可接受網路延遲與在資料中心之間有一定距離的折衷辦法。 最理想的情況是，此區域內所有可用性區域的電力、網路供應與基礎結構，都不會受到自然災害的影響。 不過，從歷史上的重大自然災害看來，可用性區域不一定能永遠提供您想要在單一區域內獲得的可用性。 例如 2017 年 9 月 20 日侵襲波多黎各島嶼的瑪莉亞 (Maria) 颶風。 這個颶風至少造成島上方圓 90 英里內的地區幾乎完全停電。

## <a name="single-vm-scenario"></a>單一 VM 案例

在單一 VM 案例中，您會建立 Azure 虛擬機器來取得 SAP HANA 執行個體。 您會使用 Azure 進階儲存體來裝載作業系統磁碟和所有資料磁碟。 Azure 99.9% 的執行時間 SLA 及其他 Azure 元件的 SLA，足以實現您要提供給客戶的可用性 SLA。 在此案例中，您不需要針對執行 DBMS 層的 VM 使用 Azure 可用性設定組。 在此案例中，您會依賴兩個不同的功能：

- Azure VM 自動重新啟動 (也稱為 Azure 服務修復)
- SAP HANA 自動重新啟動

Azure VM 自動重新啟動 (又稱服務修復) 是在以下兩個層級上運作的 Azure 功能：

- Azure 伺服器主機會檢查伺服器主機上所裝載 VM 的健康情況。
- Azure 網狀架構控制器會監視伺服器主機的健康情況和可用性。

健康情況檢查功能會針對裝載在 Azure 伺服器主機上的每個 VM，監視其健康情況。 如果 VM 落到健康情況不良的狀態，檢查 VM 健康情況的 Azure 主機代理程式就會起始 VM 的重新開機作業。 網狀架構控制器會藉由檢查許多可指出主機硬體問題的不同參數，來檢查主機的健康情況。 此外，也會檢查是否可透過網路存取主機。 若跡象表明主機有問題，則可能導致下列事件：

- 如果主機發出健康狀態不良的訊號，主機會重新開機，而且會觸發主機上執行的 VM 重新啟動。
- 如果成功重新開機後，主機不是處於狀況良好的狀態，系統就會開始將 VM 從原本所在但現在狀況不良的節點中，重新部署到狀況良好的主機伺服器上。 在此情況下，原本的主機會標示為狀況不良。 它無法再用來部署，直到加以清除或取代為止。
- 如果狀態不良的主機在重新開機程序中發生問題，則會觸發 VM 在狀態良好的主機上立即重新啟動。 

Azure VM 有 Azure 提供的主機和 VM 監視功能，因此在遇到主機問題時會自動於狀態良好的 Azure 主機上重新啟動。 

>[!IMPORTANT]
>Azure 服務修復並不會使客體作業系統處於核心異常狀態的 Linux Vm 重新啟動。 在常用的 Linux 發行版本中，預設設定不會自動重新啟動 Linux 核心處於異常狀態的 VM 或伺服器。 相反地，預設設定會讓作業系統維持在核心異常的狀態，以便連結核心偵錯來進行分析。 Azure 會接受該行為，不會自動重新啟動客體作業系統處於這類狀態的 VM。 我們假設這種情況極為罕見。 您也可以覆寫此預設行為，以啟用重新啟動 VM 的功能。 若要變更預設行為，在 /etc/sysctl.conf 中啟用 'kernel.panic' 參數。 設定此參數的時間大約是幾秒鐘。 常見的建議值約需要等候 20-30 秒，才會透過此參數觸發重新開機。 另請參閱 <https://gitlab.com/procps-ng/procps/blob/master/sysctl.conf>。

您在此案例中依賴的第二個功能，就是在重新啟動的 VM 上執行的 HANA 服務會在 VM 重新開機後自動啟動。 您可以透過不同 HANA 服務的監視程式服務來設定 [HANA 服務自動重新啟動](https://help.sap.com/viewer/6b94445c94ae495c83a19646e7c3fd56/2.0.01/en-US/cf10efba8bea4e81b1dc1907ecc652d3.html)。

您可藉由將冷容錯移轉的節點新增至 SAP HANA 設定，來改善這個單一 VM 的案例。 在 SAP HANA 文件中，此設定稱為[主機自動容錯移轉](https://help.sap.com/viewer/6b94445c94ae495c83a19646e7c3fd56/2.0.01/en-US/ae60cab98173431c97e8724856641207.html)。 此設定適用於內部部署的情況，其中伺服器硬體受到限制，而您可以將單一伺服器節點作為一組生產主機的主機自動容錯移轉節點。 但在 Azure 中，Azure 的基礎結構會提供狀態良好的目標伺服器來成功重新啟動 VM，因此，部署 SAP HANA 主機自動容錯移轉就毫無意義。 因為 Azure 服務的修復功能，我們沒有參考架構可用來預測 HANA 主機自動容錯移轉的待用節點。 

### <a name="special-case-of-sap-hana-scale-out-configurations-in-azure"></a>Azure 中的 SAP HANA 延展組態特殊案例
高可用性的 SAP HANA 延展組態依賴 Azure VM 的服務修復，以及依賴在 VM 再次啟動並執行時，SAP HANA 執行個體的重新啟動。 以 HANA 系統複寫為基礎的高可用性架構將在稍後介紹。 


## <a name="availability-scenarios-for-two-different-vms"></a>兩種不同 VM 的可用性案例

如果您在 Azure 可用性設定組中使用兩個 Azure VM，則可增加這兩個 VM 之間的運作時間，但前提是這些 VM 要放在單一 Azure 區域內的 Azure 可用性設定組中。 Azure 中的基底設定看起來會像這樣：

![兩個 VM 和所有層級的圖表](./media/sap-hana-availability-one-region/two_vm_all_shell.PNG)

為了說明不同的可用性情況，我們會省略圖表中的幾層。 下圖只會顯示描述了 VM、主機、可用性設定組及 Azure 區域的幾層。 本節所說明的案例中不會用到 Azure 虛擬網路執行個體、資源群組及訂用帳戶。

### <a name="replicate-backups-to-a-second-virtual-machine"></a>將備份複寫到第二個虛擬機器

最基本的設定之一是使用備份。 特別是，您可以將交易記錄備份從某個 VM 送到另一個 Azure VM。 您可以選擇 Azure 儲存體類型。 在此設定中，您要負責使用指令碼將第一個 VM 上所進行的排定備份複製到第二個 VM。 如果您需要使用第二個 VM 的執行個體，您必須將完整備份、增量/差異備份與交易記錄備份還原為所需的時間點。 

此架構如下所示：

![兩個 VM 與儲存體複寫的圖表](./media/sap-hana-availability-one-region/two_vm_storage_replication.PNG) 

此設定不適合用來實現絕佳的復原點目標 (RPO) 和復原時間目標 (RTO) 時間。 特別是 RTO 時間會因為需使用複製的備份來完整還原整個資料庫而受到影響。 不過，此設定適合用來復原不小心從主要執行個體上刪除的資料。 透過此設定，您可以隨時還原至特定時間點、擷取資料，以及將已刪除的資料匯入至您的主要執行個體。 因此，將備份複製方法與其他高可用性功能結合是很合理的行為。 

在複製備份時，您可以使用比 SAP HANA 執行個體執行所在的主要 VM 還要小的 VM。 請注意，您可以將更少量的 VHD 連結至較小的 VM。 如需個別 VM 類型限制的相關資訊，請參閱 [Azure 中的 Linux 虛擬機器大小](https://docs.microsoft.com/azure/virtual-machines/linux/sizes)。

### <a name="sap-hana-system-replication-without-automatic-failover"></a>不含自動容錯移轉的 SAP HANA 系統複寫

本節所述案例會使用 SAP HANA 系統複寫。 如需 SAP 文件，請參閱[系統複寫](https://help.sap.com/viewer/6b94445c94ae495c83a19646e7c3fd56/2.0.01/en-US/b74e16a9e09541749a745f41246a065e.html)。 不具有自動容錯移轉的案例，在單一 Azure 區域中的組態上並不常見。 組態上若沒有自動容錯移轉 (雖然可避免安裝 Pacemaker)，會造成您必須手動進行監視和容錯移轉。 由於這也是費時費力，大部分的客戶還是會依賴 Azure 服務修復。 在某些邊緣案例中，這種設定可能對失敗情況有所幫助。 或者，在某些案例中，客戶可以實現更大的效率。

#### <a name="sap-hana-system-replication-without-auto-failover-and-without-data-preload"></a>不具有自動容錯移轉和資料預先載入的 SAP HANA 系統複寫

在此案例中，您會使用 SAP HANA 系統複寫來同步移動資料，以實現值為 0 的 RPO。 另一方面，您有夠長的 RTO，因此不需要容錯移轉或預先將資料載入 HANA 執行個體快取。 在此情況下，您可以採取下列動作讓您的設定更具經濟效益：

- 在第二個 VM 中執行另一個 SAP HANA 執行個體。 第二個 VM 中的 SAP HANA 執行個體會耗用虛擬機器的大部分記憶體。 如果容錯移轉至第二個 VM，您需要關閉已將資料完全載入第二個 VM 的執行中 SAP HANA 執行個體，以便複寫資料可以載入到第二個 VM 中的目標 HANA 執行個體快取。
- 在第二個 VM 上使用較小的 VM 大小。 如果發生容錯移轉，您必須再執行一個步驟，才能進行手動容錯移轉。 在此步驟中，您可以將 VM 的大小調整為來源 VM 的大小。 
 
該案例將如下所示：

![兩個 VM 與儲存體複寫的圖表](./media/sap-hana-availability-one-region/two_vm_HSR_sync_nopreload.PNG)

> [!NOTE]
> 即使不在 HANA 系統複寫目標中使用資料預先載入，也至少需要 64 GB 的記憶體。 除了 64 GB 外，您還要有足夠記憶體以保存目標執行個體記憶體中的資料列存放區資料。

#### <a name="sap-hana-system-replication-without-auto-failover-and-with-data-preload"></a>不具有自動容錯移轉但具有資料預先載入的 SAP HANA 系統複寫

在此案例中，複寫到第二 VM 中 HANA 執行個體的資料會預先載入。 這會讓未預先載入資料的兩個優點消失。 在此情況下，您無法在第二個 VM 上執行另一個 SAP HANA 系統。 您也不能使用較小的 VM 大小。 因此，客戶很少會實作此案例。

### <a name="sap-hana-system-replication-with-automatic-failover"></a>具有自動容錯移轉的 SAP HANA 系統複寫

在單一 Azure 區域內最常見的標準可用性設定中，兩個執行 SLES Linux 的 Azure VM 會定義容錯移轉叢集。 SLES Linux 叢集會以 [Pacemaker](http://www.linux-ha.org/wiki/Pacemaker) 架構為基礎，並搭配 [STONITH](http://linux-ha.org/wiki/STONITH) 裝置。 

如果從 SAP HANA 的角度來看，所使用的複寫模式已同步處理，且自動容錯移轉也已設定。 在第二個 VM 中，SAP HANA 執行個體會作為熱待命節點。 待命節點會接收與主要 SAP HANA 執行個體同步的變更記錄資料流。 HANA 主要節點上的應用程式認可交易後，主要 HANA 節點會等候向應用程式確認該認可，直到次要 SAP HANA 節點確認接收到認可記錄為止。 SAP HANA 提供兩種同步複寫模式。 如需詳細資料以及這兩種同步複寫模式差異的說明，請參閱 SAP 的[SAP HANA 系統複寫的複寫模式](https://help.sap.com/viewer/6b94445c94ae495c83a19646e7c3fd56/2.0.02/en-US/c039a1a5b8824ecfa754b55e0caffc01.html)一文。

整體組態如下所示：

![兩個 VM 與儲存體複寫及容錯移轉的圖表](./media/sap-hana-availability-one-region/two_vm_HSR_sync_auto_pre_preload.PNG)

您可以選擇此解決方案，因為它可讓您實現 RPO = 0 和低 RTO。 透過 SAP HANA 用戶端使用虛擬 IP 位址來連線至 HANA 系統複寫組態的方式，設定 SAP HANA 用戶端連線。 藉由這類組態，在發生容錯移轉至第二個節點的情形時，就不需要重新設定應用程式。 在此解決方案中，主要和次要 VM 的 Azure VM SKU 必須相同。

## <a name="next-steps"></a>後續步驟

如需在 Azure 中設定這些設定的逐步指引，請參閱：

- [在 Azure 虛擬機器中設定 SAP Hana 系統複寫](sap-hana-high-availability.md)
- [使用系統複寫的 SAP HANA 高可用性](https://blogs.sap.com/2018/01/08/your-sap-on-azure-part-4-high-availability-for-sap-hana-using-system-replication/)

如需跨 Azure 區域的 SAP HANA 可用性詳細資訊，請參閱：

- [跨 Azure 區域的 SAP HANA 可用性](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/sap-hana-availability-across-regions) 

