---
title: Azure DC/OS 叢集上具備 Vamp 的 Canary 版本
description: 如何使用 Vamp 進行 Canary 版本服務，並在 Azure Container Service DC/OS 叢集上套用智慧型流量篩選
services: container-service
author: gggina
manager: jeconnoc
ms.service: container-service
ms.topic: article
ms.date: 04/17/2017
ms.author: rasquill
ms.custom: mvc
ms.openlocfilehash: 0f6c0e9206d9e056ee0496b6cc515625b08b1e4a
ms.sourcegitcommit: 0408c7d1b6dd7ffd376a2241936167cc95cfe10f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/26/2018
ms.locfileid: "36959911"
---
# <a name="canary-release-microservices-with-vamp-on-an-azure-container-service-dcos-cluster"></a>Azure Container Service DC/OS 叢集上具備 Vamp 的 Canary 版本微服務

在本逐步解說中，我們會在具備 DC/OS 叢集的 Azure Container Service 上設定 Vamp。 我們在 Canary 版本中的 Vamp 示範服務為 "sava"，並套用智慧型流量篩選，解決了與 Firefox 服務不相容的問題。 

> [!TIP] 
> 在本逐步解說中，Vamp 是在 DC/OS 叢集上執行，但您也可以使用 Vamp 搭配 Kubernetes 來作為協調者。
>

## <a name="about-canary-releases-and-vamp"></a>關於 Canary 版本與 Vamp


[Canary 版本](https://martinfowler.com/bliki/CanaryRelease.html)是諸如 Netflix、Facebook 和 Spotify 等創新組織所採用的智慧型部署策略。 它是個很適切的方法，因為它能減少問題、引入網路安全性，並提升創新技術。 那為什麼並非所有公司都使用它？ 擴充 CI/CD 管線以包含 Canary 策略會增加複雜度，並且需要廣泛的 DevOps 知識和經驗。 這甚至足以使較小型公司以及企業在還沒開始進行之前就望之卻步。 

[Vamp](http://vamp.io/) 是開放原始碼系統，旨在簡化這項轉換，並將 Canary 版本功能引進您慣用的容器排程器。 Vamp 的 Canary 功能並不僅是以百分比為基礎的發行。 流量可在各種不同的條件 (例如目標特定使用者、IP 範圍或裝置) 下加以篩選及分割。 Vamp 會追蹤和分析效能計量，允許以實際資料作為基礎進行自動化。 您可以設定錯誤的自動復原，或以負載或延遲作為基礎，將個別服務變化進行調整。

## <a name="set-up-azure-container-service-with-dcos"></a>使用 DC/OS 設定 Azure Container Service



1. 使用一個主機和兩個預設大小的代理程式來[部署 DC/OS 叢集](container-service-deployment.md)。 

2. [建立 SSH 通道](../container-service-connect.md)以連線至 DC/OS 叢集。 本文假設您建立至本機連接埠 80 上叢集的通道。


## <a name="set-up-vamp"></a>設定 Vamp

現在您已擁有執行中的 DC/OS 叢集，可以從 DC/OS UI (http://localhost:80) 安裝 Vamp。 

![DC/OS UI](./media/container-service-dcos-vamp-canary-release/01_set_up_vamp.png)

會在兩個階段中完成安裝︰

1. **部署 Elasticsearch**。

2. 然後安裝 Vamp DC/OS Universe 套件來**部署 Vamp**。

### <a name="deploy-elasticsearch"></a>部署 Elasticsearch

Vamp 需要 Elasticsearch 來進行計量收集和彙總。 您可以使用 [magneticio Docker 映像](https://hub.docker.com/r/magneticio/elastic/)來部署相容的 Vamp Elasticsearch 堆疊。

1. 在 DC/OS UI 中，請移至 [服務]，然後按一下 [部署服務]。

2. 從 [部署新服務] 快顯中選取 [JSON 模式]。

  ![選取 JSON 模式](./media/container-service-dcos-vamp-canary-release/02_deploy_service_json_mode.png)

3. 貼入下列 JSON。 這個設定會使用 1 GB 的 RAM 和 Elasticsearch 連接埠上的基本健康情況檢查來執行容器。
  
  ```JSON
  {
    "id": "elasticsearch",
    "instances": 1,
    "cpus": 0.2,
    "mem": 1024.0,
    "container": {
      "docker": {
        "image": "magneticio/elastic:2.2",
        "network": "HOST",
        "forcePullImage": true
      }
    },
    "healthChecks": [
      {
        "protocol": "TCP",
        "gracePeriodSeconds": 30,
        "intervalSeconds": 10,
        "timeoutSeconds": 5,
        "port": 9200,
        "maxConsecutiveFailures": 0
      }
    ]
  }
  ```
  

3. 按一下 [ **部署**]。

  DC/OS 會部署 Elasticsearch 容器。 您可以在 [服務] 頁面上追蹤進度。  

  ![部署 e?Elasticsearch](./media/container-service-dcos-vamp-canary-release/03_deply_elasticsearch.png)

### <a name="deploy-vamp"></a>部署 Vamp

一旦 Elasticsearch 報告為**執行**後，您就可以新增 Vamp DC/OS Universe 套件。 

1. 移至 **Universe**，並搜尋 **vamp**。 
  ![DC/OS Universe 上的 Vamp](./media/container-service-dcos-vamp-canary-release/04_universe_deploy_vamp.png)

2. 按一下 Vamp 套件旁的 [安裝]，然後選擇 [進階安裝]。

3. 向下捲動並輸入下列 elasticsearch-url：`http://elasticsearch.marathon.mesos:9200`。 

  ![輸入 Elasticsearch URL](./media/container-service-dcos-vamp-canary-release/05_universe_elasticsearch_url.png)

4. 依序按一下 [檢閱並安裝]、[安裝] 可開始進行部署。  

  DC/OS 會部署所有必要的 Vamp 元件。 您可以在 [服務] 頁面上追蹤進度。
  
  ![部署 Vamp 作為 Universe 套件](./media/container-service-dcos-vamp-canary-release/06_deploy_vamp.png)
  
5. 一旦部署完成後，您就可以存取 Vamp UI：

  ![DC/OS 上的 Vamp 服務](./media/container-service-dcos-vamp-canary-release/07_deploy_vamp_complete.png)
  
  ![Vamp UI](./media/container-service-dcos-vamp-canary-release/08_vamp_ui.png)


## <a name="deploy-your-first-service"></a>部署您的第一個服務

現在該 Vamp 已啟動並且執行中，請從藍圖部署服務。 

簡單來說，[Vamp 藍圖](http://vamp.io/documentation/using-vamp/blueprints/)會描述要部署的端點 (閘道)、叢集和服務。 Vamp 會使用叢集，將相同服務的不同變化群組為邏輯群組，以供 Canary 版本或 A/B 進行測試。  

此案例中使用的範例整合型應用程式稱為 [**sava**](https://github.com/magneticio/sava)，為 1.0 版。 整合是封裝在一個 Docker 容器中，位於 magneticio/sava:1.0.0 下的 Docker 中樞。 應用程式通常會在連接埠 8080 上執行，但在此情況下，您需要在連接埠 9050 下將它公開。 使用簡單的藍圖，透過 Vamp 將應用程式進行部署。

1. 前往**部署**。

2. 按一下 [新增] 。

3. 貼入下列藍圖 YAML。 此藍圖所包含的一個叢集只有一個服務變化，我們將在稍後步驟中進行變更︰

  ```YAML
  name: sava                        # deployment name
  gateways:
    9050: sava_cluster/webport      # stable endpoint
  clusters:
    sava_cluster:               # cluster to create
     services:
        -
          breed:
            name: sava:1.0.0        # service variant name
            deployable: magneticio/sava:1.0.0
            ports:
              webport: 8080/http # cluster endpoint, used for canary releasing
  ```

4. 按一下 [檔案] 。 Vamp 會起始部署。

部署會列在 [部署] 頁面上。 按一下 [部署] 可監視其狀態。

![Vamp UI - 部署 sava](./media/container-service-dcos-vamp-canary-release/09_sava100.png)

![Vamp UI 中的 sava 服務](./media/container-service-dcos-vamp-canary-release/09a_sava100.png)

會建立兩個閘道，並列於 [閘道] 頁面︰

* 存取執行中服務的穩定端點 (連接埠 9050) 
* 受 Vamp 管理的內部閘道 (稍後將詳細說明此閘道)。 

![Vamp UI - sava 閘道](./media/container-service-dcos-vamp-canary-release/10_vamp_sava_gateways.png)

現在已部署 sava 服務，但您無法在外部存取，因為 Azure Load Balancer 還不知道要將流量向它轉送。 若要存取服務，請更新 Azure 網路設定。


## <a name="update-the-azure-network-configuration"></a>更新 Azure 網路設定

Vamp 已在 DC/OS 代理程式節點上部署 sava 服務，在連接埠 9050 中將穩定端點公開。 若要從 DC/OS 叢集外部存取服務，在叢集部署中，請對 Azure 網路設定進行下列變更︰ 

1. 使用健康情況探查和規則，將連接埠 9050 上的流量轉送至 sava 執行個體，從而**設定代理程式的 Azure Load Balancer** (名為 **dcos-agent-lb-xxxx** 的資源)。 

2. **更新公用代理程式的網路安全性群組** (名為 **XXXX-agent-public-nsg-XXXX** 的資源)，以允許連接埠 9050 上的流量。

如需使用 Azure 入口網站完成這些工作的詳細步驟，請參閱[啟用 Azure Container Service 應用程式的公用存取](container-service-enable-public-access.md)。 針對所有連接埠設定指定連接埠 9050。


一旦建立所有項目之後，請移至 DC/OS 代理程式負載平衡器的 [概觀] 刀鋒視窗 (名為 **dcos-agent-lb-xxxx** 的資源)。 尋找**公用 IP 位址**，並使用該位址來存取連接埠 9050 上的 sava。

![Azure 入口網站 - 取得公用 IP 位址](./media/container-service-dcos-vamp-canary-release/18_public_ip_address.png)

![sava](./media/container-service-dcos-vamp-canary-release/19_sava100.png)


## <a name="run-a-canary-release"></a>執行 Canary 版本

假設您有這個應用程式的新版本，而您想要將 Canary 版本用於實際執行環境。 您將它容器化為 magneticio/sava:1.1.0 並準備就緒。 Vamp 可讓您輕鬆地將新服務新增至執行中的部署。 這些「合併」的服務會與叢集中現有的服務一起部署，並將權重指派為 0%。 在您將流量散發進行調整之前，流量不會路由傳送至新合併的服務。 Vamp UI 中的權數滑桿可讓您完全控制散發，以便進行增量調整 (Canary 版本) 或立即復原。

### <a name="merge-a-new-service-variant"></a>合併新的服務變化

若要將新的 sava 1.1 服務與執行中的部署合併︰

1. 在 Vamp UI 中，按一下 [藍圖]。

2. 按一下 [新增]，並貼入下列藍圖 YAML︰這個藍圖會描述要在現有叢集 (sava_cluster) 內部署的新服務變化 (sava:1.1.0)。

  ```YAML
  name: sava:1.1.0      # blueprint name
  clusters:
    sava_cluster:       # cluster to update
      services:
        -
          breed:
            name: sava:1.1.0    # service variant name
            deployable: magneticio/sava:1.1.0    
            ports:
              webport: 8080/http # cluster endpoint to update
  ```
  
3. 按一下 [檔案] 。 藍圖會儲存在 [藍圖] 頁面上並加以列出。

4. 將 sava:1.1 藍圖上的 [動作] 功能表開啟，然後按一下 [合併至]。

  ![Vamp UI - 藍圖](./media/container-service-dcos-vamp-canary-release/20_sava110_mergeto.png)

5. 選取 **sava** 部署，然後按一下 [合併]。

  ![Vamp UI - 將藍圖合併至部署](./media/container-service-dcos-vamp-canary-release/21_sava110_merge.png)

Vamp 會部署藍圖中所述的新 sava:1.1.0 服務變化，連同執行中部署的 **sava_cluster** 中的 sava:1.0.0。 

![Vamp UI - 更新的 sava 部署](./media/container-service-dcos-vamp-canary-release/22_sava_cluster.png)

**sava/sava_cluster/webport** 閘道 (叢集端點) 也會加以更新，將路由新增至新部署的 sava:1.1.0。 此時，沒有流量會路由傳送至這裡 (**權數**設定為 0%)。

![Vamp UI - 叢集閘道](./media/container-service-dcos-vamp-canary-release/23_sava_cluster_webport.png)

### <a name="canary-release"></a>Canary 版本

將兩個版本的 sava 部署在相同叢集中，移動**權數**滑桿來調整這兩者間的流量散發。

1. 按一下 [權數] 旁的 [Vamp UI - 編輯]![](./media/container-service-dcos-vamp-canary-release/vamp_ui_edit.png)。

2. 將權數散發設定為 50%/50%，然後按一下 [儲存]。

  ![Vamp UI - 閘道權數滑桿](./media/container-service-dcos-vamp-canary-release/24_sava_cluster_webport_weight.png)

3. 返回您的瀏覽器，並將 sava 頁面重新整理數次。 sava 應用程式現在會在 sava:1.0 頁面和 sava:1.1 頁面之間進行切換。

  ![將 sava1.0 和 sava1.1 服務進行交替](./media/container-service-dcos-vamp-canary-release/25_sava_100_101.png)


  > [!NOTE]
  > 此頁面交替會快取靜態資產，因此最適用於瀏覽器的 "Incognito" 或 “Anonymous” 模式。
  >

### <a name="filter-traffic"></a>篩選流量

假設您在部署之後發現 sava:1.1.0 中發生不相容，而造成 Firefox 瀏覽器中的顯示問題。 您可以將 Vamp 設為篩選傳入流量，並將所有 Firefox 使用者導向回到已知的穩定 sava:1.0.0。 此篩選會立即解決 Firefox 使用者的中斷問題，而其他人還是能繼續享有 sava:1.1.0 提升的優點。

Vamp 會使用**條件**來篩選閘道路由之間的流量。 流量會以套用至每個路由的條件作為基礎，先行加以篩選及導向。 所有剩餘的流量會根據閘道權重設定加以散發。

您可以建立條件來篩選所有的 Firefox 使用者，並將他們導向至舊的 sava:1.0.0︰

1. 在 sava/sava_cluster/webport [閘道] 頁面上，按一下 [Vamp UI - 編輯]![](./media/container-service-dcos-vamp-canary-release/vamp_ui_edit.png)，將 [條件] 新增至路由 sava/sava_cluster/sava:1.0.0/webport。 

2. 輸入條件 **user-agent == Firefox**，然後按一下 [Vamp UI - 儲存]![](./media/container-service-dcos-vamp-canary-release/vamp_ui_save.png)。

  Vamp 會以 0% 的預設強度來新增條件。 若您要啟動篩選流量，需要調整條件強度。

3. 按一下 [Vamp UI - 編輯]![](./media/container-service-dcos-vamp-canary-release/vamp_ui_edit.png) 可將套用至條件的**強度**進行變更。
 
4. 將 [強度]設定為 100%，然後按一下 [Vamp UI - 儲存]![](./media/container-service-dcos-vamp-canary-release/vamp_ui_save.png) 來加以儲存。

  Vamp 現在會將符合條件 (所有 Firefox 使用者) 的所有流量傳送至 sava:1.0.0。

  ![Vamp UI - 將條件套用至閘道](./media/container-service-dcos-vamp-canary-release/26_apply_condition.png)

5. 最後，調整閘道加權，將所有其他流量 (所有非 Firefox 使用者) 傳送至新的 sava:1.1.0。 按一下 [權數] 旁的 [Vamp UI - 編輯]![](./media/container-service-dcos-vamp-canary-release/vamp_ui_edit.png) 並設定權數散發，讓 100% 導向路由 sava/sava_cluster/sava:1.1.0/webport。

  非經條件篩選的所有流量現在都會導向至新的 sava:1.1.0。

6. 若要查看動作中的篩選，請開啟兩個不同的瀏覽器 (一個為 Firefox 及一個其他瀏覽器)，並同時從兩個瀏覽器存取 sava 服務。 所有 Firefox 的要求都會傳送至 sava:1.0.0，而所有其他瀏覽器的要求則會導向至 sava:1.1.0。

  ![Vamp UI - 篩選流量](./media/container-service-dcos-vamp-canary-release/27_filter_traffic.png)

## <a name="summing-up"></a>總結

本文是 DC/OS 叢集上的 Vamp 快速簡介。 對於新手，您會啟動 Vamp 並在 Azure Container Service DC/OS 叢集上加以執行、使用 Vamp 藍圖部署服務，並在公開的端點 (閘道) 加以存取。

我們也提及 Vamp 的一些強大功能︰將新的服務變化合併至執行中的部署，並以增量方式將它引入，然後篩選流量來解決已知的不相容問題。


## <a name="next-steps"></a>後續步驟

* 了解如何透過 [Vamp REST API](http://vamp.io/documentation/api/api-reference/) 來管理 Vamp 動作。

* 在 Node.js 中建置 Vamp 自動化指令碼，並以 [Vamp 工作流程](http://vamp.io/documentation/tutorials/create-a-workflow/)來執行這些指令碼。

* 請參閱其他 [VAMP 教學課程](http://vamp.io/documentation/tutorials/)。

