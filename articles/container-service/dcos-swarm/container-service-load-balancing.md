---
title: Azure DC/OS 叢集中的負載平衡容器
description: Azure Container Service DC/OS 叢集中多個容器的負載平衡。
services: container-service
author: rgardler
manager: jeconnoc
ms.service: container-service
ms.topic: tutorial
ms.date: 06/02/2017
ms.author: rogardle
ms.custom: mvc
ms.openlocfilehash: 8e8ad913c6e5b8e947b17ed8920285de3fcfd8ae
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46964355"
---
# <a name="load-balance-containers-in-an-azure-container-service-dcos-cluster"></a>Azure Container Service DC/OS 叢集中容器的負載平衡

在此文章中，我們將探討如何在 DC/OS 管理的 Azure Container Service 中使用 Marathon-LB 建立內部負載平衡器。 這項設定可讓您以水平方式調整應用程式。 也可讓您在將負載平衡器放在公用叢集上，並將應用程式容器放在私用叢集上，藉此利用公用和私用代理程式叢集。 在本教學課程中，您：

> [!div class="checklist"]
> * 設定 Marathon 負載平衡器
> * 使用負載平衡器部署應用程式
> * 設定 Azure 負載平衡器

您需要有 ACS DC/OS 叢集，才能完成本教學課程中的步驟。 如有需要，[此指令碼範例](./../kubernetes/scripts/container-service-cli-deploy-dcos.md)可為您建立一個叢集。

本教學課程需要 Azure CLI 2.0.4 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要升級，請參閱[安裝 Azure CLI]( /cli/azure/install-azure-cli)。 

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="load-balancing-overview"></a>負載平衡概觀

在 Azure Container Service DC/OS 叢集中有兩個負載平衡層： 

**Azure 負載平衡器**會提供公用進入點 (一般使用者存取的點)。 Azure Container Service 會自動提供 Azure LB，而且 Azure LB 會預設為公開連接埠 80、443 和 8080。

**Marathon 負載平衡器 (marathon-lb)** 會將輸入要求傳送至為這些要求提供服務的容器執行個體。 當我們調整提供 Web 服務的容器時，marathon-lb 會跟著動態調整。 根據預設，您的 Container Service 中不會提供此負載平衡器，但進行安裝非常容易。

## <a name="configure-marathon-load-balancer"></a>設定 Marathon 負載平衡器

Marathon Load Balancer 會根據您所部署的容器以動態方式重新設定本身。 它也比較容易從遺失容器或代理程式中復原 - 如果發生這種情況，Apache Mesos 會重新啟動別處的容器且 marathon-lb 會調整。

執行下列命令，在公用代理程式叢集上安裝 marathon 負載平衡器。

```azurecli-interactive
dcos package install marathon-lb
```

## <a name="deploy-load-balanced-application"></a>部署負載平衡應用程式

我們現在有 marathon-lb 套件，可以部署希望平衡負載的應用程式容器。 

首先，取得已公開代理程式的 FQDN。

```azurecli-interactive
az acs list --resource-group myResourceGroup --query "[0].agentPoolProfiles[0].fqdn" --output tsv
```

接下來，建立名為 *hello-web.json* 的檔案，並複製下列內容。 `HAPROXY_0_VHOST` 標籤需要使用 DC/OS 代理程式的 FQDN 進行更新。 

```json
{
  "id": "web",
  "container": {
    "type": "DOCKER",
    "docker": {
      "image": "yeasy/simple-web",
      "network": "BRIDGE",
      "portMappings": [
        { "hostPort": 0, "containerPort": 80, "servicePort": 10000 }
      ],
      "forcePullImage":true
    }
  },
  "instances": 3,
  "cpus": 0.1,
  "mem": 65,
  "healthChecks": [{
      "protocol": "HTTP",
      "path": "/",
      "portIndex": 0,
      "timeoutSeconds": 10,
      "gracePeriodSeconds": 10,
      "intervalSeconds": 2,
      "maxConsecutiveFailures": 10
  }],
  "labels":{
    "HAPROXY_GROUP":"external",
    "HAPROXY_0_VHOST":"YOUR FQDN",
    "HAPROXY_0_MODE":"http"
  }
}
```

請使用 DC/OS CLI 來執行應用程式。 Marathon 預設會將應用程式部署到私人叢集。 這表示上述部署只能透過您的負載平衡器進行存取，此舉通常是所需的行為。

```azurecli-interactive
dcos marathon app add hello-web.json
```

部署應用程式之後，請瀏覽至代理程式叢集的 FQDN，以檢視負載平衡應用程式。

![負載平衡應用程式的影像](./media/container-service-load-balancing/lb-app.png)

## <a name="configure-azure-load-balancer"></a>設定 Azure 負載平衡器

根據預設，Azure Load Balancer 會公開連接埠 80、8080 和 443。 如果您使用上述其中一個連接埠 (如我們在上述範例中所為)，則不需要執行任何作業。 您應該可以叫用代理程式負載平衡器的 FQDN，而每次重新整理，您就會以循環配置方式叫用三個 Web 伺服器之一。 

不過，如果您使用不同的連接埠，則需要在負載平衡器上針對您所使用的連接埠新增循環備置規則和探查。 您可以從 [Azure CLI](../../azure-resource-manager/xplat-cli-azure-resource-manager.md) 中使用 `azure network lb rule create` 和 `azure network lb probe create` 命令來執行此動作。

## <a name="next-steps"></a>後續步驟

在此教學課程中，您已了解在 ACS 中使用 Marathon 和 Azure 這兩個負載平衡器來進行負載平衡包括下列動作：

> [!div class="checklist"]
> * 設定 Marathon 負載平衡器
> * 使用負載平衡器部署應用程式
> * 設定 Azure 負載平衡器

請前往下一個教學課程，了解如何在 Azure 中將 Azure 儲存體與 DC/OS 進行整合。

> [!div class="nextstepaction"]
> [在 DC/OS 叢集中裝載 Azure 檔案共用](container-service-dcos-fileshare.md)