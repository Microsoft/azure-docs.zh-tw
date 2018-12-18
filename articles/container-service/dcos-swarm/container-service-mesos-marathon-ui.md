---
title: 使用 Marathon UI 管理 Azure DC/OS 叢集
description: 使用 Marathon Web UI 將容器部署到 Azure 容器服務叢集服務。
services: container-service
author: iainfoulds
manager: jeconnoc
ms.service: container-service
ms.topic: article
ms.date: 04/04/2017
ms.author: iainfou
ms.custom: mvc
ms.openlocfilehash: a22bddf48f97d961d481e2aedb42f7d645f3e678
ms.sourcegitcommit: d551ddf8d6c0fd3a884c9852bc4443c1a1485899
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/07/2018
ms.locfileid: "37903076"
---
# <a name="manage-an-azure-container-service-dcos-cluster-through-the-marathon-web-ui"></a>透過 Marathon Web UI 管理 Azure Container Service DC/OS 叢集

DC/OS 提供環境來部署及調整叢集工作負載，同時將基礎硬體抽象化。 在 DC/OS 之上有架構會管理排程和執行計算工作負載。

雖然許多常見的工作負載都有可用的架構，但此文件只說明如何使用 Marathon 來開始部署容器。 


## <a name="prerequisites"></a>先決條件
在練習這些範例之前，您需要 Azure 容器服務中設定的 DC/OS 叢集。 您也需要有此叢集的遠端連線。 如需這些項目的詳細資訊，請參閱下列文章。

* [部署 Azure 容器服務叢集](container-service-deployment.md)
* [連接到 Azure Container Service 叢集](../container-service-connect.md)

> [!NOTE]
> 此文章假設您透過本機連接埠 80 以通道方式連線到 DC/OS 叢集。
>

## <a name="explore-the-dcos-ui"></a>探索 DC/OS UI
在[建立](../container-service-connect.md)安全殼層 (SSH) 通道之後，瀏覽至 http://localhost/。 這會載入 DC/OS Web UI 並顯示叢集的相關資訊，例如使用的資源、作用中代理程式和執行中的服務。

![DC/OS UI](./media/container-service-mesos-marathon-ui/dcos2.png)

## <a name="explore-the-marathon-ui"></a>瀏覽 Marathon UI
若要查看 Marathon UI，請瀏覽至 http://localhost/marathon。 在此畫面中，您可以啟動 Azure 容器服務 DC/OS 叢集上的新容器或其他應用程式。 您也可以看到有關執行容器和應用程式的資訊。  

![Marathon UI](./media/container-service-mesos-marathon-ui/dcos3.png)

## <a name="deploy-a-docker-formatted-container"></a>部署 Docker 格式化容器
若要使用 Marathon 部署新容器，請按一下 [建立應用程式]，並在表單索引標籤中輸入下列資訊：

| 欄位 | 值 |
| --- | --- |
| ID |nginx |
| 記憶體 | 32 |
| 映像 |nginx |
| 網路 |橋接 |
| 主機連接埠 |80 |
| 通訊協定 |TCP |

![新增應用程式 UI--一般](./media/container-service-mesos-marathon-ui/dcos4.png)

![新增應用程式 UI--Docker 容器](./media/container-service-mesos-marathon-ui/dcos5.png)

![新增應用程式 UI--連接埠和服務探索](./media/container-service-mesos-marathon-ui/dcos6.png)

如果您想要以靜態方式將容器的連接埠對應至代理程式上的連接埠，您需要使用 JSON 模式。 若要這樣做，請使用切換功能將「新增應用程式」精靈切換為 **JSON 模式** 。 然後在應用程式定義的 `portMappings` 區段底下輸入下列設定。 此範例會將容器的連接埠 80 繫結至 DC/OS 代理程式的連接埠 80。 在進行這項變更之後，您可以將此精靈切換離開 JSON 模式。

```none
"hostPort": 80,
```

![新增應用程式 UI--連接埠 80 範例](./media/container-service-mesos-marathon-ui/dcos13.png)

如果您想要啟用健康狀態檢查，請在 [健康狀態檢查] 索引標籤上設定路徑。

![新的應用程式 UI--健康狀態檢查](./media/container-service-mesos-marathon-ui/dcos_healthcheck.png)

DC/OS 叢集會使用一組私人和公用代理程式來進行部署。 若要讓叢集能夠從網際網路存取應用程式，您需要將應用程式部署至公用代理程式。 若要這樣做，請選取「新增應用程式」精靈的 [選擇性] 索引標籤，並在 [接受的資源角色] 中輸入 **slave_public**。

然後按一下 [建立應用程式]。

![新增應用程式 UI--公用代理程式設定](./media/container-service-mesos-marathon-ui/dcos14.png)

回到 Marathon 主頁面，您可以看到容器的部署狀態。 一開始您看到的狀態為 [部署中]。 部署成功之後，狀態會變更為 [執行中]。

![Marathon 主頁面 UI--容器部署狀態](./media/container-service-mesos-marathon-ui/dcos7.png)

當您切換回 DC/OS Web UI (http://localhost/)，便會看到 DC/OS 叢集上正在執行一項工作 (在此案例中是 Docker 格式化容器)。

![DC/OS Web UI--在叢集上執行的工作](./media/container-service-mesos-marathon-ui/dcos8.png)

若要查看工作執行所在的叢集節點，請按一下 [節點] 索引標籤。

![DC/OS Web UI--工作叢集節點](./media/container-service-mesos-marathon-ui/dcos9.png)

## <a name="reach-the-container"></a>存取容器

在此範例中，應用程式是在公用代理程式節點上執行。 從網際網路存取此應用程式的方式是瀏覽到叢集的代理程式 FQDN：`http://[DNSPREFIX]agents.[REGION].cloudapp.azure.com`，其中：

* **DNSPREFIX** 是您部署叢集時提供的 DNS 首碼。
* **REGION** 是資源群組所在的區域。

    ![來自網際網路的 Nginx](./media/container-service-mesos-marathon-ui/nginx.png)


## <a name="next-steps"></a>後續步驟
* [使用 DC/OS 和 Marathon API](container-service-mesos-marathon-rest.md)

* 透過 Mesos 在 Azure Container Service 上深入探索

    > [!VIDEO https://channel9.msdn.com/Events/Microsoft-Azure/AzureCon-2015/ACON203/player]
    > 
