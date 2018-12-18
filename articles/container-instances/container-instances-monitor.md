---
title: 在 Azure 容器執行個體中監視容器
description: 有關如何在 Azure 容器執行個體中依照您的容器，監視計算資源 (例如 CPU 和記憶體) 耗用量的詳細資訊。
services: container-instances
author: mmacy
manager: jeconnoc
ms.service: container-instances
ms.topic: overview
ms.date: 04/24/2018
ms.author: marsma
ms.openlocfilehash: 6107aee0a4d7497e90186e0ea4bda6cd2712820d
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46979602"
---
# <a name="monitor-container-resources-in-azure-container-instances"></a>在 Azure 容器執行個體中監視容器資源

Azure 監視器可提供您的容器執行個體所用計算資源的見解。 使用 Azure 監視器來追蹤容器群組及其容器的 CPU 和記憶體使用率。 此資源使用量資料可協助您判斷容器群組的最佳 CPU 和記憶體設定。

本文件詳述如何使用 Azure 入口網站和 Azure CLI，蒐集容器執行個體的 CPU 和記憶體使用量。

> [!IMPORTANT]
> 在這個階段，資源使用量計量只適用於 Linux 容器。
>

## <a name="available-metrics"></a>可用的計量

Azure 監視器會提供 Azure 容器執行個體的 **CPU** 和**記憶體** 使用量計量。 這兩個計量均適用於容器群組和個別容器。

CPU 計量是以 **Millicore** 表示。 一個 Millicore 為一個 CPU 核心的 1/1000，因此 500 Millicore (或 500 m) 表示 CPU 核心的 50% 使用率。

記憶體計量是以**位元組**表示。

## <a name="get-metrics---azure-portal"></a>取得計量 - Azure 入口網站

建立容器群組後，即可在 Azure 入口網站中使用 Azure 監視資料。 若要查看容器群組的計量，請選取資源群組，然後選取容器群組。 您可以在此查看 CPU 和記憶體使用量的預建圖表。

![雙重圖表][dual-chart]

如果您有包含多個容器的容器群組，請使用[維度][monitor-dimension]來呈現每個個別容器的計量。 若要使用個別容器計量來建立圖表，請執行下列步驟：

1. 從左側導覽功能表中選取 [監視]。
2. 選取容器群組和計量 (CPU 或記憶體)。
3. 選取綠色維度按鈕，然後選取 [容器名稱]。

![維度][dimension]

## <a name="get-metrics---azure-cli"></a>取得計量 - Azure CLI

使用 Azure CLI 也可以蒐集容器執行個體 CPU 和記憶體使用量。 首先，使用下列命令來取得容器群組的識別碼。 以您的資源群組名稱取代 `<resource-group>`，並以您的容器群組名稱取代 `<container-group>`。


```console
CONTAINER_GROUP=$(az container show --resource-group <resource-group> --name <container-group> --query id --output tsv)
```

使用下列命令來取得 **CPU** 使用量計量。

```console
$ az monitor metrics list --resource $CONTAINER_GROUP --metric CPUUsage --output table

Timestamp            Name              Average
-------------------  ------------  -----------
2018-04-22 04:39:00  CPU Usage
2018-04-22 04:40:00  CPU Usage
2018-04-22 04:41:00  CPU Usage
2018-04-22 04:42:00  CPU Usage
2018-04-22 04:43:00  CPU Usage      0.375
2018-04-22 04:44:00  CPU Usage      0.875
2018-04-22 04:45:00  CPU Usage      1
2018-04-22 04:46:00  CPU Usage      3.625
2018-04-22 04:47:00  CPU Usage      1.5
2018-04-22 04:48:00  CPU Usage      2.75
2018-04-22 04:49:00  CPU Usage      1.625
2018-04-22 04:50:00  CPU Usage      0.625
2018-04-22 04:51:00  CPU Usage      0.5
2018-04-22 04:52:00  CPU Usage      0.5
2018-04-22 04:53:00  CPU Usage      0.5
```

然後，使用下列命令來取得**記憶體**使用量計量。

```console
$ az monitor metrics list --resource $CONTAINER_GROUP --metric MemoryUsage --output table

Timestamp            Name              Average
-------------------  ------------  -----------
2018-04-22 04:38:00  Memory Usage
2018-04-22 04:39:00  Memory Usage
2018-04-22 04:40:00  Memory Usage
2018-04-22 04:41:00  Memory Usage
2018-04-22 04:42:00  Memory Usage  6.76915e+06
2018-04-22 04:43:00  Memory Usage  9.22061e+06
2018-04-22 04:44:00  Memory Usage  9.83552e+06
2018-04-22 04:45:00  Memory Usage  8.42906e+06
2018-04-22 04:46:00  Memory Usage  8.39526e+06
2018-04-22 04:47:00  Memory Usage  8.88013e+06
2018-04-22 04:48:00  Memory Usage  8.89293e+06
2018-04-22 04:49:00  Memory Usage  9.2073e+06
2018-04-22 04:50:00  Memory Usage  9.36243e+06
2018-04-22 04:51:00  Memory Usage  9.30509e+06
2018-04-22 04:52:00  Memory Usage  9.2416e+06
2018-04-22 04:53:00  Memory Usage  9.1008e+06
```

對於多容器群組，可以新增 `containerName` 維度以針對每個容器傳回此資料。

```console
$ az monitor metrics list --resource $CONTAINER_GROUP --metric CPUUsage --dimension containerName --output table

Timestamp            Name          Containername             Average
-------------------  ------------  --------------------  -----------
2018-04-22 17:03:00  Memory Usage  aci-tutorial-app      1.95338e+07
2018-04-22 17:04:00  Memory Usage  aci-tutorial-app      1.93096e+07
2018-04-22 17:05:00  Memory Usage  aci-tutorial-app      1.91488e+07
2018-04-22 17:06:00  Memory Usage  aci-tutorial-app      1.94335e+07
2018-04-22 17:07:00  Memory Usage  aci-tutorial-app      1.97714e+07
2018-04-22 17:08:00  Memory Usage  aci-tutorial-app      1.96178e+07
2018-04-22 17:09:00  Memory Usage  aci-tutorial-app      1.93434e+07
2018-04-22 17:10:00  Memory Usage  aci-tutorial-app      1.92614e+07
2018-04-22 17:11:00  Memory Usage  aci-tutorial-app      1.90659e+07
2018-04-22 16:12:00  Memory Usage  aci-tutorial-sidecar  1.35373e+06
2018-04-22 16:13:00  Memory Usage  aci-tutorial-sidecar  1.28614e+06
2018-04-22 16:14:00  Memory Usage  aci-tutorial-sidecar  1.31379e+06
2018-04-22 16:15:00  Memory Usage  aci-tutorial-sidecar  1.29536e+06
2018-04-22 16:16:00  Memory Usage  aci-tutorial-sidecar  1.38138e+06
2018-04-22 16:17:00  Memory Usage  aci-tutorial-sidecar  1.41312e+06
2018-04-22 16:18:00  Memory Usage  aci-tutorial-sidecar  1.49914e+06
2018-04-22 16:19:00  Memory Usage  aci-tutorial-sidecar  1.43565e+06
2018-04-22 16:20:00  Memory Usage  aci-tutorial-sidecar  1.408e+06
```

## <a name="next-steps"></a>後續步驟

在 [Azure 監視概觀][azure-monitoring]深入了解 Azure 監視。

<!-- IMAGES -->
[cpu-chart]: ./media/container-instances-monitor/cpu-multi.png
[dimension]: ./media/container-instances-monitor/dimension.png
[dual-chart]: ./media/container-instances-monitor/metrics.png
[memory-chart]: ./media/container-instances-monitor/memory-multi.png

<!-- LINKS - Internal -->
[azure-monitoring]: ../monitoring-and-diagnostics/monitoring-overview.md
[monitor-dimension]: ../monitoring/monitoring-data-collection.md#multi-dimensional-metrics
