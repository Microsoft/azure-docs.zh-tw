---
title: 使用 Azure DNS 和流量管理員進行災害復原 | Microsoft Docs
description: 使用 Azure DNS 和流量管理員的災害復原解決方案概觀。
services: dns
documentationcenter: na
author: KumudD
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: dns
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 06/08/2018
ms.author: kumud
ms.openlocfilehash: ce3e8f31c7fee6afdeabf931485a49934e98f81b
ms.sourcegitcommit: 794bfae2ae34263772d1f214a5a62ac29dcec3d2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44391346"
---
# <a name="disaster-recovery-using-azure-dns-and-traffic-manager"></a>使用 Azure DNS 和流量管理員進行災害復原

災害復原著重在從應用程式功能的嚴重損失中復原。 為了選擇災害復原解決方案，企業和技術擁有者必須先決定災害發生時所需的功能運作層級，例如，不需運作、儘需部分運作 (透過降低功能或延遲可用性來實現)、或是需完全運作。
大部分的企業客戶會選擇多區域架構，藉由應用程式或基礎結構層級的容錯移轉來恢復功能。 客戶可選擇多種方法來透過備援架構達到容錯移轉和高可用性。 以下是一些常見的方法：

- **搭配冷待命的主動-被動**：在此容錯移轉解決方案中，在待命區域中執行的 VM 及其他設備，會等到需要進行容錯移轉時才開始運作。 不過，生產環境會以備份、VM 映像或 Resource Manager 範本的形式來複寫到不同區域。 此容錯移轉機制符合成本效益，但須耗費較長時間來進行完整的容錯移轉。
 
    ![搭配冷待命的主動/被動](./media/disaster-recovery-dns-traffic-manager/active-passive-with-cold-standby.png)
    
    圖 - 搭配冷待命的主動/被動災害復原組態

- **搭配指示燈的主動/被動**：在此容錯移轉解決方案中，待命環境會以最低限度的組態來加以設定。 此設定只會執行必要的服務，來支援最低限度和最重要的一組應用程式。 在其原生形式中，此方案只能執行最少的功能，但可以相應增加和繁衍其他服務，以在發生容錯移轉時承受大量的生產負載。
    
    ![搭配指示燈的主動/被動](./media/disaster-recovery-dns-traffic-manager/active-passive-with-pilot-light.png)
    
    圖：搭配指示燈的主動/被動災害復原組態

- **搭配暖待命的主動/被動**：在此容錯移轉解決方案中，待命區域會預先作準備並可執行基本負載、自動調整會開啟，以及所有執行個體都會啟動並執行。 此解決方案不會調整到可執行完整的生產負載，但可運作，且所有服務都會啟動並執行。 此解決方案是指示燈方法的增強型版本。
    
    ![搭配暖待命的主動/被動](./media/disaster-recovery-dns-traffic-manager/active-passive-with-warm-standby.png)
    
    圖：搭配暖待命的主動/被動災害復原組態
    
如需有關容錯移轉和高可用性的詳細資訊，請參閱 [Azure 應用程式的災害復原](https://docs.microsoft.com/azure/architecture/resiliency/disaster-recovery-azure-applications)。


## <a name="planning-your-disaster-recovery-architecture"></a>規劃災害復原架構

設定災害復原架構涉及到兩個技術層面：
-  使用部署機制在主要和待命環境之間複寫執行個體、資料和組態。 您可以藉由 Microsoft Azure 合作夥伴設備/服務，例如 Veritas 或 NetApp 等，透過 Azure Site-Recovery 以原生方式來完成此類災害復原。 
- 開發解決方案來將網路/ Web 流量從主要站台轉移至待命站台。 此類災害復原可以透過 Azure DNS、Azure 流量管理員 (DNS) 或第三方全域負載平衡器來完成。

本文所介紹的方法，僅能透過網路與 Web 流量的重新導向來實現。 如需設定 Azure Site Recovery 的指示，請參閱 [Azure Site Recovery 文件](https://docs.microsoft.com/azure/site-recovery/)。
DNS 是轉移網路流量最有效率的機制之一，因為 DNS 通常是全球性的，且位於資料中心外部，不會受到任何區域性故障或可用性區域 (AZ) 層級的故障所影響。 除了能使用 DNS 架構的容錯移轉機制外，在 Azure 中還有另外兩個 DNS 服務能在某些形式上達到相同成效 - Azure DNS (權威式 DNS) 和 Azure 流量管理員 (DNS 架構的智慧流量路由)。 

請務必了解 DNS 的一些概念，我們會廣泛地使用這些概念來討論本文中提供的解決方案：
- **DNS A 記錄** – A 記錄是將網域指向 IPv4 位址的指標。 
- **CNAME 或 Canonical 名稱** - 此記錄類型用來指向其他 DNS 記錄。 CNAME 不會以 IP 位址作為回應，而是以指標，其指向包含 IP 位址的記錄。 
- **加權路由** – 您可以選擇為服務端點賦予特定的權重，然後再根據指派的權重分配流量。
 此路由方法是流量管理員中四個可用流量路由機制的其中一個。 如需詳細資訊，請參閱 [加權路由方法](../traffic-manager/traffic-manager-routing-methods.md#weighted)。
- **優先順序路由** – 優先順序路由會以端點的健康情況檢查為基礎。 根據預設，Azure 流量管理員會將所有流量傳送到優先順序最高的端點，而在發生失敗或損毀時，流量管理員會將流量路由至次要端點。 如需詳細資訊，請參閱 [優先順序路由方法](../traffic-manager/traffic-manager-routing-methods.md#priority)。

## <a name="manual-failover-using-azure-dns"></a>使用 Azure DNS 進行手動容錯移轉
災害復原的 Azure DNS 手動容錯移轉解決方案會使用標準 DNS 機制來容錯移轉至備份站台。 Azure DNS 的手動選項最適合與冷待命或指示燈方法搭配使用。 

![使用 Azure DNS 進行手動容錯移轉](./media/disaster-recovery-dns-traffic-manager/manual-failover-using-dns.png)

圖 - 使用 Azure DNS 進行手動容錯移轉

解決方案所做的假設是：
-   主要和次要端點都有不常變更的靜態 IP。 假設主要站台的 IP 是 100.168.124.44，而次要站台的 IP 是 100.168.124.43。
-   主要和次要站台都有 Azure DNS 區域。 假設主要站台的端點是 prod.contoso.com，而備份站台的端點是 dr.contoso.com。 主應用程式的 DNS 記錄 (稱為 www.contoso.com) 也存在。   
-   TTL 會等於或低於組織中設定的 RTO SLA。 例如，如果企業將應用程式災害回應的 RTO 設定為 60 分鐘，則 TTL 應不超過 60 分鐘，而且愈低愈好。 您可以設定 Azure DNS 的手動容錯移轉，如下所示：
1. 建立 DNS 區域
2. 建立 DNS 區域記錄
3. 更新 CNAME 記錄

### <a name="step-1-create-a-dns"></a>步驟 1 - 建立 DNS
建立 DNS 區域 (例如 www.contoso.com)，如下所示：

![在 Azure 中建立 DNS 區域](./media/disaster-recovery-dns-traffic-manager/create-dns-zone.png)

圖 - 在 Azure 中建立 DNS 區域

### <a name="step-2-create-dns-zone-records"></a>步驟 2：建立 DNS 區域記錄

在此區域內建立三個記錄 (例如 www.contoso.com、prod.contoso.com 和 dr.consoto.com)，如下所示。

![建立 DNS 區域記錄](./media/disaster-recovery-dns-traffic-manager/create-dns-zone-records.png)

圖 - 在 Azure 中建立 DNS 區域記錄

在此案例中，www.contoso.com 站台有 30 分鐘的 TTL (低於指定的 RTO)，且指向生產站台 prod.contoso.com。 這是正常營運時的組態。 prod.contoso.com 和 dr.contoso.com 的 TTL 已設定為 300 秒或 5 分鐘。 您可以使用 Azure 監視器或 Azure App Insights 這類 Azure 監視服務，或 Dynatrace 等任何合作夥伴監視解決方案，甚至可以使用自有的解決方案，來監視或偵測應用程式或虛擬基礎結構層級的失敗。

### <a name="step-3-update-the-cname-record"></a>步驟 3：更新 CNAME 記錄

一旦偵測到失敗時，記錄值將變更為指向 dr.contoso.com，如下所示：
       
![更新 CNAME 記錄](./media/disaster-recovery-dns-traffic-manager/update-cname-record.png)

圖 - 更新 Azure 中的 CNAME 記錄

在 30 分鐘內，大部分的解析程式將會重新整理快取的區域檔案，任何針對 www.contoso.com 的查詢會重新導向至 dr.contoso.com。
您也可以執行下列 Azure CLI 命令來變更 CNAME 值：
 ```azurecli
   az network dns record-set cname set-record \
   --resource-group 123 \
   --zone-name contoso.com \
   --record-set-name www \
   --cname dr.contoso.com
```
此步驟可以手動執行或透過自動化來執行。 手動執行可透過主控台或 Azure CLI。 Azure SDK 和 API 可用來自動化 CNAME 更新，如此則不需要手動介入。 您可以透過 Azure 函式或第三方監視應用程式，或甚至是從內部部署來建立自動化。

### <a name="how-manual-failover-works-using-azure-dns"></a>使用 Azure DNS 進行手動容錯移轉的運作方式
由於 DNS 伺服器位在容錯移轉或災害區域外，因此不受任何停機狀況所影響。 這可讓使用者建構符合成本效益的簡單容錯移轉案例，並在發生災害時持續運作 (假設運算子有網路連線)，而且可反轉為主要站台。 如果此解決方案以指令碼編寫，務必將執行指令碼的伺服器或服務與影響生產環境的問題隔離。 此外，請記住要針對區域設定低 TTL，如此一來，世界各地的解析程式就不會長期保留快取的端點，而客戶可在 RTO 內存取站台。 針對冷待命和指示燈方法，由於可能需要一些預先準備和其他系統管理活動，因此在進行反轉前應提供足夠的時間。

## <a name="automatic-failover-using-azure-traffic-manager"></a>使用 Azure 流量管理員進行自動容錯移轉
如果您有複雜的架構和能夠執行相同函式的多組資源，您可以設定 Azure 流量管理員 (以 DNS 為基礎) 來檢查您的資源健康情況，並將流量從狀況不好的資源路由到狀況良好的資源。 在下列範例中，主要區域和次要地區都有完整的部署。 此部署包括雲端服務和已同步處理的資料庫。 

![使用 Azure 流量管理員進行自動容錯移轉](./media/disaster-recovery-dns-traffic-manager/automatic-failover-using-traffic-manager.png)

圖 - 使用 Azure 流量管理員進行自動容錯移轉

不過，只有主要區域會主動處理來自使用者的網路要求。 只有當主要區域發生服務中斷時，次要地區才會變成主動。 在此情況下，所有新的網路要求會路由傳送到次要地區。 由於資料庫備份幾乎是瞬間完成、兩個負載平衡器的 IP 狀況都會受到檢查，以及執行個體永遠是啟動且正在執行中，因此此拓撲提供以低 RTO 運作及無須任何手動介入的容錯移轉選項。 次要的容錯移轉區域必須準備好在主要區域失敗後立即開始運作。
此案例非常適合使用 Azure 流量管理員，其內建探查可處理各種類型的健康情況檢查，包括 http / https 和 TCP。 Azure 流量管理員也有規則引擎，可設定為在發生失敗時進行容錯移轉，如下所述。 讓我們使用流量管理員來設想下列解決方案：
- 客戶都有稱為 prod.contoso.com 的區域 #1 端點，且靜態 IP 為 100.168.124.44，以及稱為 dr.contoso.com 的區域 #2 端點，靜態 IP 為 100.168.124.43。 
-   這些環境的每一個都透過公眾對應屬性 (例如負載平衡器) 而置於前方。 負載平衡器可以設定為有 DNS 架構的端點或完整網域名稱 (FQDN)，如上所示。
-   區域 2 中的所有執行個體幾乎是及時地對區域 1 進行複寫。 此外，機器映像處於最新狀態，而且所有軟體/組態資料接已修補，並且與區域 1 一致。  
-   已預先設定自動調整。 

使用 Azure 流量管理員設定容錯移轉的步驟如下：
1. 建立新的 Azure 流量管理員設定檔
2. 在流量管理員設定檔中建立端點
3. 設定健康情況檢查和容錯移轉組態

### <a name="step-1-create-a-new-azure-traffic-manager-profile"></a>步驟 1：建立新的 Azure 流量管理員設定檔
使用名稱 contoso123 建立新的 Azure 流量管理員設定檔，並選取 [優先順序] 作為 [路由方式]。 如果您有想與之建立關聯的現有資源群組，則您可以選取現有的資源群組，否則請建立新的資源群組。

![建立流量管理員設定檔](./media/disaster-recovery-dns-traffic-manager/create-traffic-manager-profile.png)
圖 - 建立流量管理員設定檔**

### <a name="step-2-create-endpoints-within-the-traffic-manager-profile"></a>步驟 2：在流量管理員設定檔中建立端點

在此步驟中，您會建立指向生產環境和災害復原站台的端點。 在這裡，選擇 [外部端點] 作為 [類型]，但如果資源裝載在 Azure 中，則也可以選擇 [Azure 端點]。 如果您選擇 [Azure 端點]，則選取 [App Service] 或 Azure 配置的 [公用 IP] **作為 [目標資源]**。 優先順序會設為 **1**，因為它是區域 1 的主要服務。
同樣地，也在流量管理員中建立災害復原端點。

![建立災害復原端點](./media/disaster-recovery-dns-traffic-manager/create-disaster-recovery-endpoint.png)

圖 - 建立災害復原端點

### <a name="step-3-set-up-health-check-and-failover-configuration"></a>步驟 3：設定健康情況檢查和容錯移轉組態

在此步驟中，您會將 DNS TTL 設為 10 秒，這適用於大部分與網際網路對應的遞迴解析程式。 此設定表示沒有 DNS 解析程式可快取資訊超過 10 秒。 針對端點監視設定，路徑目前設定在 / 或根目錄，但是您可以自訂端點設定來評估路徑，例如 prod.contoso.com/index。 下方範例顯示的探查通訊協定是 **https**。 不過，您也可以選擇 **http** 或 **tcp**。 通訊協定的選擇取決於後端應用程式。 探查間隔設定為 10 秒，以便進行快速探查，而重試設定為 3。 如此一來，如果三個連續的探查間隔顯示為失敗，流量管理員就會容錯移轉至第二個端點。 下列公式用來定義自動容錯移轉的時間總計：容錯移轉時間 = TTL + 重試 * 探查間隔，而在此案例中，這個值會是 10 + 3 * 10 = 40 秒 (最大值)。
如果重試設為 1 且 TTL 設定為 10 秒，則容錯移轉時間是 10 + 1 * 10 = 20 秒。 將重試值設為大於 **1**，可排除因為誤判或網路短暫中斷而造成容錯移轉的機會。 


![設定健康情況檢查](./media/disaster-recovery-dns-traffic-manager/set-up-health-check.png)

圖 - 設定健康情況檢查和容錯移轉組態

### <a name="how-automatic-failover-works-using-traffic-manager"></a>使用流量管理員進行自動容錯移轉的運作方式

發生損毀時，系統會探查主要端點，而狀態會變更為**降級**，然後災害復原站台會保持**連線**。 根據預設，流量管理員會將所有流量傳送至主要 (最高優先順序) 端點。 如果主要端點出現降級，流量管理員會將流量路由至第二個端點，前提是第二個端點的狀況良好。 您可以選擇在流量管理員中設定多個端點，讓這些端點可作為額外的容錯移轉端點，或是作為可分擔各端點負載的負載平衡器。

## <a name="next-steps"></a>後續步驟
- 深入了解 [Azure 流量管理員](../traffic-manager/traffic-manager-overview.md)。
- 深入了解 [ Azure DNS](../dns/dns-overview.md)。









