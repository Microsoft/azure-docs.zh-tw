---
title: 監視 Azure DC/OS 叢集 - ELK 堆疊
description: 使用 ELK (Elasticsearch、Logstash 與 Kibana) 監視 Azure Container Service 中的 DC/OS 叢集。
services: container-service
author: sauryadas
manager: jeconnoc
ms.service: container-service
ms.topic: article
ms.date: 03/27/2017
ms.author: saudas
ms.custom: mvc
ms.openlocfilehash: dc863894d8846e066c90bdf7b309f141d32a1186
ms.sourcegitcommit: e2adef58c03b0a780173df2d988907b5cb809c82
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/28/2018
ms.locfileid: "32163175"
---
# <a name="monitor-an-azure-container-service-cluster-with-elk"></a>使用 ELK 監視 Azure Container Service 叢集

在此文章中，我們示範如何在 Azure Container Service 中的 DC/OS 叢集上部署 ELK (Elasticsearch、Logstash、Kibana) 堆疊。 

## <a name="prerequisites"></a>先決條件
[部署](container-service-deployment.md)並[連接](../container-service-connect.md) Azure Container Service 所設定的 DC/OS 叢集。 在[這裡](container-service-mesos-marathon-ui.md)探索探索 DC/OS 儀表板和 Marathon 服務。 同時也請安裝 [Marathon Load Balancer](container-service-load-balancing.md)。


## <a name="elk-elasticsearch-logstash-kibana"></a>ELK (Elasticsearch、Logstash、Kibana)
ELK 堆疊是 Elasticsearch、Logstash 和 Kibana 的組合，提供的端對端堆疊可以監視及分析叢集中的記錄。

## <a name="configure-the-elk-stack-on-a-dcos-cluster"></a>在 DC/OS 叢集上設定 ELK 堆疊
由 [http://localhost:80/](http://localhost:80/) 存取 DC/OS UI。開啟 DC/OS UI 之後，瀏覽到 [Universe]。 請在 DC/OS Universe 搜尋並依序安裝 Elasticsearch、Logstash 及 Kibana。 如需組態的詳細資訊，您可以移至 [Advanced Installation (進階安裝)]連結。

![ELK1](./media/container-service-monitoring-elk/elk1.PNG) ![ELK2](./media/container-service-monitoring-elk/elk2.PNG) ![ELK3](./media/container-service-monitoring-elk/elk3.PNG) 

ELK 容器開始運作之後，您需要讓 Marathon-LB 能夠存取 Kibana。 瀏覽到 [Services (服務)] > [kibana]，然後按一下 [Edit (編輯)]，如下圖。

![ELK4](./media/container-service-monitoring-elk/elk4.PNG)


切換成 [JSON mode (JSON 模式)]，並捲動到 labels 區段。
您需要新增一個 `"HAPROXY_GROUP": "external"` 項目，如下圖。
按一下 [Deploy changes (部署變更)] 之後，容器會重新啟動。

![ELK5](./media/container-service-monitoring-elk/elk5.PNG)


如果您想確認 Kibana 已經在 HAPROXY 儀表板中註冊為服務，您需要開啟代理程式叢集上的連接埠 9090，因為 HAPROXY 在連接埠 9090 執行。
根據預設，我們開啟 DC/OS 叢集中的接埠 80、8080 和 443。
[這裡](container-service-enable-public-access.md)有開啟連接埠並提供公開存取的指示。

若要存取 HAPROXY 儀表板，請開啟 Marathon-LB 系統管理員介面，網址是：`http://$PUBLIC_NODE_IP_ADDRESS:9090/haproxy?stats`。
瀏覽到該 URL 之後，您會看到如下圖的 HAPROXY 儀表板，也會看到 Kibana 的服務項目。

![ELK6](./media/container-service-monitoring-elk/elk6.PNG)


因為 Kibana 部署在連接埠 5601，所以您需要開啟連接埠 5601 才能存取 Kibana 儀表板。 遵循[這裡](container-service-enable-public-access.md)的指示。 然後開啟 Kibana 儀表板，網址是：`http://localhost:5601`。

## <a name="next-steps"></a>後續步驟

* 如需系統和應用程式記錄檔轉送與設定的相關資訊，請參閱 [DC/OS 中使用 ELK 的記錄檔管理 (英文)](https://docs.mesosphere.com/1.8/administration/logging/elk/)。

* 若要篩選記錄檔，請參閱[使用 ELK 篩選記錄檔 (英文)](https://docs.mesosphere.com/1.8/administration/logging/filter-elk/)。 

 

