---
title: 在 Azure 中使用負載平衡服務 | Microsoft Docs
description: 本教學課程會示範如何使用 Azure 負載平衡組合建立案例︰流量管理員、應用程式閘道和負載平衡器。
services: traffic-manager
documentationcenter: ''
author: liumichelle
manager: vitinnan
editor: ''
ms.assetid: f89be3be-a16f-4d47-bcae-db2ab72ade17
ms.service: traffic-manager
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/27/2016
ms.author: limichel
ms.openlocfilehash: 86867a9d6d2c43e6505b1a06672546a017172bfe
ms.sourcegitcommit: d87b039e13a5f8df1ee9d82a727e6bc04715c341
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/21/2018
ms.locfileid: "29401101"
---
# <a name="using-load-balancing-services-in-azure"></a>在 Azure 中使用負載平衡服務

## <a name="introduction"></a>簡介

Microsoft Azure 提供多個服務，可管理分配網路流量和負載平衡的方式。 根據您的需求，您可以個別使用這些服務或者結合其方法來建立最佳的解決方案。

在本教學課程中，我們首先定義客戶使用案例，看看如何使用下列的 Azure 負載平衡組合讓它更強大且高效能︰流量管理員、應用程式閘道和負載平衡器。 然後，我們會提供逐步說明如何建立地理備援、分散流量至 VM，並協助您管理不同要求類型功能的部署。

在概念層級中，這些服務每一個都在負載平衡階層架構中扮演不同的角色。

* **流量管理員**提供全域 DNS 負載平衡。 它會查看傳入 DNS 要求並根據客戶所選取的路由原則，以狀況良好的端點回應。 路由方法的選項如下︰
  * 效能路由，根據延遲將要求者傳送到最接近的端點。
  * 優先順序路由，可將所有流量導向端點，而其他端點做為備份。
  * 加權循環配置資源路由，可根據指派給各端點的加權分散流量。

  用戶端直接連線至該端點。 Azure 流量管理員會在偵測到端點狀況不良時，將用戶端重新導向至另一個狀況良好的執行個體。 若要了解服務的相關資訊，請參閱 [Azure 流量管理員文件](traffic-manager-overview.md)。
* **應用程式閘道**會以服務形式提供應用程式傳遞控制器 (ADC)，為您的應用程式提供各種第 7 層負載平衡功能。 它會將 CPU 密集 SSL 終止卸載至應用程式閘道，讓客戶最佳化 Web 伺服陣列的產能。 其他第 7 層路由功能包括循環配置連入流量、以 Cookie 為基礎的工作階段同質、URL 路徑型路由，以及在單一應用程式閘道背後代管多個網站的能力。 應用程式閘道可以設定為連結網際網路的閘道、內部專用閘道或兩者混合。 應用程式閘道完全由 Azure 管理、可調整且可用性極高。 它提供一組豐富的診斷和記錄功能，很好管理。
* **負載平衡器**是 Azure SDN 堆疊不可或缺的一部分，針對所有 UDP 和 TCP 通訊協定提供高效能、低延遲的第 4 層負載平衡服務。 它會管理輸入及輸出連線。 您可以設定公用和內部負載平衡端點，並使用 TCP 和 HTTP 健全狀況探查選項定義規則，將輸入連線對應至後端集區目的地，以管理服務可用性。

## <a name="scenario"></a>案例

在此範例案例中，我們使用簡單的網站提供兩種類型的內容︰影像及動態呈現的網頁。 網站必須是地理備援，且應從最接近 (最低延遲) 的位置將服務提供給其使用者。 應用程式開發人員決定任何符合模式/影像/* 的 URL 都是從不同於 Web 伺服陣列之其餘部分的專用 VM 集區提供。

此外，提供動態內容的預設 VM 集區需要與高可用性叢集上裝載的後端資料庫聯繫。 整個部署是透過 Azure Resource Manager 來設定。

使用流量管理員、應用程式閘道和負載平衡器可讓這個網站達成這些設計目標︰

* **多重地理區域備援**：如果某個區域故障，流量管理員會將流量順暢地路由到最近的區域，而不需要應用程式擁有者介入。
* **降低延遲**：因為流量管理員會自動將客戶導向到最近的區域，所以當客戶要求網頁內容時會體驗到較低的延遲。
* **獨立擴充**︰因為 Web 應用程式工作負載是透過內容類型分隔，應用程式擁有者可以各自獨立調整要求工作負載。 應用程式閘道可根據指定的規則和應用程式的健全狀況，確保流量路由傳送至適當的集區。
* **內部負載平衡**︰因為高可用性叢集前面具有負載平衡器，只有作用中且狀況良好的資料庫端點會公開給應用程式。 此外，資料庫管理員可以獨立將主動和被動複本分散到前端應用程式的叢集來最佳化工作負載。 負載平衡器會提供對高可用性叢集的連線，並確定只有狀況良好的資料庫會收到連線要求。

下圖顯示此案例的架構︰

![負載平衡架構的圖表](./media/traffic-manager-load-balancing-azure/scenario-diagram.png)

> [!NOTE]
> 這個範例只是 Azure 所提供的許多可能的負載平衡服務設定的其中一個。 Azure 流量管理員、應用程式閘道和負載平衡器可以混合使用及配對，以提供最符合您負載平衡的需求。 例如，如果 SSL 卸載或第 7 層處理並非必要，則負載平衡器可用以取代應用程式閘道。

## <a name="setting-up-the-load-balancing-stack"></a>設定負載平衡堆疊

### <a name="step-1-create-a-traffic-manager-profile"></a>步驟 1︰建立流量管理員設定檔

1. 在 Azure 入口網站中，按一下 [建立資源] > [網路] > [流量管理員設定檔] > [建立]。
2. 輸入下列基本資訊：

  * **名稱**：為您的流量管理員設定檔提供一個 DNS 首碼名稱。
  * **路由方法**：選取流量路由方法原則。 如需方法的詳細資訊，請參閱[關於流量管理員流量路由方法](traffic-manager-routing-methods.md)。
  * **訂閱**：選取包含設定檔的訂用帳戶。
  * **資源群組**：選取包含設定檔的資源群組。 可以是新的或現有的資源群組。
  * **資源群組位置**：流量管理員服務為全域服務，無法繫結至某個位置。 不過，您必須針對群組指定一個與流量管理員設定檔相關聯之中繼資料所在的區域。 這個位置對於設定檔的執行階段可用性沒有任何影響。

3. 按一下 [建立] 以產生流量管理員設定檔。

  ![[建立流量管理員] 刀鋒視窗](./media/traffic-manager-load-balancing-azure/s1-create-tm-blade.png)

### <a name="step-2-create-the-application-gateways"></a>步驟 2：建立應用程式閘道

1. 在 Azure 入口網站的左窗格中，按一下 [建立資源] > [網路] > [應用程式閘道]。
2. 輸入下列關於應用程式閘道的基本資訊︰

  * **名稱**：應用程式閘道的名稱。
  * **SKU 大小**：應用程式閘道的大小，有「小型」、「中型」或「大型」。
  * **執行個體計數**：執行個體的數目，從 2 到 10 的值。
  * **資源群組**：保存應用程式閘道的資源群組。 它可以是現有的或新的資源群組。
  * **位置**：應用程式閘道的區域，它是與資源群組所在相同的位置。 位置相當重要，因為虛擬網路和公用 IP 必須與閘道位於相同位置。
3. 按一下 [SERVICEPRINCIPAL] 。
4. 接下來，針對應用程式閘道定義虛擬網路、子網路、前端 IP 與接聽程式設定。 在此案例中，前端 IP 位址是**公用**，以允許它稍後做為端點新增至流量管理員設定檔。
5. 使用下列其中一個選項設定接聽程式︰
    * 如果您使用 HTTP，則無需設定。 按一下 [SERVICEPRINCIPAL] 。
    * 如果您使用 HTTPS，則需要設定進一步的設定。 請參閱[建立應用程式閘道](../application-gateway/application-gateway-create-gateway-portal.md)，由步驟 9 開始。 完成設定後，按一下 [確定]。

#### <a name="configure-url-routing-for-application-gateways"></a>設定應用程式閘道的 URL 路由

當您選擇後端集區時，使用路徑型規則設定的應用程式閘道會採用除循環配置資源散發以外之要求 URL 的路徑模式。 在此案例中，我們要將導向任何具有「/images/\*」之 URL 的路徑型規則新增到映像伺服器集區。 如需設定應用程式閘道的 URL 路徑型路由相關資訊，請參閱[建立應用程式閘道的路徑型規則](../application-gateway/application-gateway-create-url-route-portal.md)。

![應用程式閘道 Web 層圖表](./media/traffic-manager-load-balancing-azure/web-tier-diagram.png)

1. 從資源群組中，移至您在上一節所建立之應用程式閘道的執行個體。
2. 在 [設定] 下選取 [後端集區]，然後選取 [新增] 以新增您要與 Web 層後端集區相關聯的 VM。
3. 輸入後端集區的名稱和存放於集區中所有機器的 IP 位址。 在此案例中，我們會連接兩個虛擬機器的後端伺服器集區。

  ![應用程式閘道「新增後端集區」](./media/traffic-manager-load-balancing-azure/s2-appgw-add-bepool.png)

4. 在應用程式閘道的 [設定] 下選取 [規則]，然後按一下 [路徑型] 按鈕以新增規則。

  ![應用程式閘道規則「路徑型」按鈕](./media/traffic-manager-load-balancing-azure/s2-appgw-add-pathrule.png)

5. 提供下列資訊來設定規則。

   基本設定：

   + **名稱**：入口網站中可存取之規則的易記名稱。
   + **接聽程式**：用於規則的接聽程式。
   + **預設後端集區**：搭配預設規則使用的後端集區。
   + **預設 HTTP 設定**：要搭配預設規則使用的 HTTP 設定。

   路徑型規則：

   + **名稱**：路徑型規則的易記名稱。
   + **路徑**：用於轉送流量的路徑規則。
   + **後端集區**：要搭配此規則使用的後端集區。
   + **HTTP 設定**：要搭配此規則使用的 HTTP 設定。

   > [!IMPORTANT]
   > 路徑︰有效的路徑必須以 "/" 開頭。 萬用字元 "\*" 僅允許在結尾使用。 有效範例包括 /xyz、/xyz\* 或 /xyz/\*。

   ![應用程式閘道器「新增路徑型規則」刀鋒視窗](./media/traffic-manager-load-balancing-azure/s2-appgw-pathrule-blade.png)

### <a name="step-3-add-application-gateways-to-the-traffic-manager-endpoints"></a>步驟 3︰將應用程式閘道新增至流量管理員端點

在此案例中，流量管理員會連結到位於不同區域中的應用程式閘道 (如前述步驟中所設定)。 現在已設定應用程式閘道，下一步是將它們連接至您的流量管理員設定檔。

1. 開啟流量管理員設定檔。 若要這樣做，請從 [所有資源] 查看您的資源群組，或搜尋流量管理員設定檔的名稱。
2. 在左窗格中，選取 [端點]，然後按一下 [新增] 以新增端點。

  ![流量管理員端點「新增」按鈕](./media/traffic-manager-load-balancing-azure/s3-tm-add-endpoint.png)

3. 輸入下列資訊來建立端點︰

  * **類型**：選取進行負載平衡之端點的類型。 在此案例中，選取 [Azure 端點]，因為我們正在將它連接到先前已設定的應用程式閘道執行個體。
  * **名稱**：輸入端點的名稱。
  * **目標資源類型**：選取 [公用 IP 位址]，然後在 [目標資源] 下，選取先前已設定的應用程式閘道公用 IP。

   ![流量管理員「新增端點」](./media/traffic-manager-load-balancing-azure/s3-tm-add-endpoint-blade.png)

4. 現在您可以測試您的安裝程式，方法為使用流量管理員設定檔的 DNS (此範例中為︰TrafficManagerScenario.trafficmanager.net) 來存取它。 您可以重送要求、啟動或關閉在不同區域中建立的 VM 和 Web 伺服器，以及變更流量管理員設定檔設定來測試您的設定。

### <a name="step-4-create-a-load-balancer"></a>步驟 4：建立負載平衡器

在此案例中，負載平衡器會在高可用性叢集內將連接從 Web 層分配至資料庫。

如果您的高可用性資料庫叢集是使用 SQL Server AlwaysOn，請參閱[設定一或多個 Always On 可用性群組接聽程式](../virtual-machines/windows/sql/virtual-machines-windows-portal-sql-ps-alwayson-int-listener.md)以取得逐步指示。

如需有關如何設定內部負載平衡器的詳細資訊，請參閱[在 Azure 入口網站中建立內部負載平衡器](../load-balancer/load-balancer-get-started-ilb-arm-portal.md)。

1. 在 Azure 入口網站的左窗格中，按一下 [建立資源] > [網路] > [負載平衡器]。
2. 為負載平衡器選擇名稱。
3. 將 [類型] 設定為 [內部]，並選擇適當的虛擬網路和子網路以供負載平衡器存放。
4. 在 [IP 位址指派] 下，選取 [動態] 或 [靜態]。
5. 在 [資源群組] 下，選擇負載平衡器的資源群組。
6. 在 [位置] 下，選擇負載平衡器適用的地區。
7. 按一下 [建立] 以產生負載平衡器。

#### <a name="connect-a-back-end-database-tier-to-the-load-balancer"></a>將後端資料庫層連接到負載平衡器

1. 從資源群組中，尋找先前步驟中所建立的負載平衡器。
2. 在 [設定] 下，按一下 [後端集區]，然後按一下 [新增] 來新增後端集區。

  ![負載平衡器「新增後端集區」](./media/traffic-manager-load-balancing-azure/s4-ilb-add-bepool.png)

3. 輸入後端集區的名稱。
4. 將個別機器或可用性設定組新增至後端集區。

#### <a name="configure-a-probe"></a>設定探查

1. 在您的負載平衡器中，於 [設定] 下選取 [探查]，然後按一下 [新增] 來新增探查。

 ![負載平衡器「新增探查」](./media/traffic-manager-load-balancing-azure/s4-ilb-add-probe.png)

2. 輸入探查的名稱。
3. 選取探查的**通訊協定**。 針對資料庫，您可能想要 TCP 探查，而不是 HTTP 探查。 若要了解有關負載平衡器探查的詳細資訊，請參閱[了解負載平衡器探查](../load-balancer/load-balancer-custom-probe-overview.md)。
4. 輸入您要用來存取探查的資料庫**連接埠**。
5. 在 [間隔] 下，指定探查應用程式的頻率。
6. 在 [狀況不良閾值] 下，針對視為狀況不良的後端 VM 指定必須發生的連續探查失敗次數。
7. 按一下 [確定] 以建立探查。

#### <a name="configure-the-load-balancing-rules"></a>設定負載平衡規則

1. 在負載平衡器的 [設定] 下，選取 [負載平衡規則]，然後按一下 [新增] 以建立規則。
2. 輸入負載平衡規則的 [名稱]。
3. 依序選擇負載平衡器的 [前端 IP 位址]、[通訊協定] 和 [連接埠]。
4. 在 [後端連接埠] 下，指定要在後端集區中使用的連接埠。
5. 選取**後端集區**和在先前步驟中所建立以對其套用規則的**探查**。
6. 在 [工作階段持續性] 下，選擇要保存工作階段的方式。
7. 在 [閒置逾時] 下，指定閒置逾時之前的分鐘數。
8. 在 [浮點 IP] 下，選取 [停用] 或 [啟用]。
9. 按一下 [確定] 以建立規則。

### <a name="step-5-connect-web-tier-vms-to-the-load-balancer"></a>步驟 5︰將 Web 層 VM 連線到負載平衡器

現在我們針對任何資料庫連接，在 Web 層 VM 上執行的應用程式中設定 IP 位址和負載平衡器前端連接埠。 此設定是在這些 VM 上執行之應用程式所特有的。 若要設定目的地 IP 位址和連接埠，請參閱應用程式文件。 若要尋找前端 IP 位址，請在 Azure 入口網站中，移至 [負載平衡器設定] 上的前端 IP 集區。

![負載平衡器「前端 IP 集區」瀏覽窗格](./media/traffic-manager-load-balancing-azure/s5-ilb-frontend-ippool.png)

## <a name="next-steps"></a>後續步驟

* [流量管理員概觀](traffic-manager-overview.md)
* [應用程式閘道概觀](../application-gateway/application-gateway-introduction.md)
* [Azure 負載平衡器概觀](../load-balancer/load-balancer-overview.md)
