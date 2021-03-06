---
title: 使用容器的 Azure 監視器設定 GPU 監視 |Microsoft Docs
description: 本文說明如何使用具有適用于容器 Azure 監視器的 NVIDIA 和 AMD GPU 啟用節點來設定監視 Kubernetes 叢集。
ms.topic: conceptual
ms.date: 03/27/2020
ms.openlocfilehash: e391117ab57211aa5d178d11c27b934b4ccd37f8
ms.sourcegitcommit: 80c1056113a9d65b6db69c06ca79fa531b9e3a00
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/09/2020
ms.locfileid: "96905582"
---
# <a name="configure-gpu-monitoring-with-azure-monitor-for-containers"></a>使用容器的 Azure 監視器設定 GPU 監視

從代理程式版本 *ciprod03022019* 開始，適用于容器的 Azure 監視器整合式代理程式現在支援監視 gpu (圖形處理單位) 在 gpu 感知的 Kubernetes 叢集節點上使用，以及監視要求和使用 gpu 資源的 pod/容器。

## <a name="supported-gpu-vendors"></a>支援的 GPU 廠商

適用于容器的 Azure 監視器支援監視下列 GPU 廠商的 GPU 叢集：

- [Nvidia](https://developer.nvidia.com/kubernetes-gpu)

- [AMD](https://github.com/RadeonOpenCompute/k8s-device-plugin)

適用于容器的 Azure 監視器會在60sec 間隔收集下列計量，並將其儲存在 **InsightMetrics** 資料表中，以自動開始監視節點的 gpu 使用量，以及要求 pod 和工作負載的 gpu。

>[!NOTE]
>布建具有 GPU 節點的叢集之後，請確定 AKS 需要安裝 [gpu 驅動程式](../../aks/gpu-cluster.md) ，才能執行 gpu 工作負載。 適用于容器的 Azure 監視器會透過在節點中執行的 GPU 驅動程式 pod 收集 GPU 計量。 

|度量名稱 |) 的計量維度 (標記 |描述 |
|------------|------------------------|------------|
|containerGpuDutyCycle |container.azm.ms/clusterId、container.azm.ms/clusterName、容器、gpuId、gpuModel、gpuVendor|過去取樣期間內的時間百分比 (60 秒) 在這段期間，GPU 忙於/主動處理容器的時間。 工作週期是介於1到100之間的數位。 |
|containerGpuLimits |container.azm.ms/clusterId、container.azm.ms/clusterName、容器 |每個容器可以將限制指定為一或多個 Gpu。 不可能要求或限制 GPU 的一部分。 |
|containerGpuRequests |container.azm.ms/clusterId、container.azm.ms/clusterName、容器 |每個容器都可以要求一或多個 Gpu。 不可能要求或限制 GPU 的一部分。|
|containerGpumemoryTotalBytes |container.azm.ms/clusterId、container.azm.ms/clusterName、容器、gpuId、gpuModel、gpuVendor |可用於特定容器的 GPU 記憶體量（以位元組為單位）。 |
|containerGpumemoryUsedBytes |container.azm.ms/clusterId、container.azm.ms/clusterName、容器、gpuId、gpuModel、gpuVendor |特定容器所使用的 GPU 記憶體量（以位元組為單位）。 |
|nodeGpuAllocatable |container.azm.ms/clusterId、container.azm.ms/clusterName、gpuVendor |節點中可供 Kubernetes 使用的 Gpu 數目。 |
|nodeGpuCapacity |container.azm.ms/clusterId、container.azm.ms/clusterName、gpuVendor |節點中的 Gpu 總數。 |

## <a name="gpu-performance-charts"></a>GPU 效能圖表 

容器的 Azure 監視器包括先前在資料表中為每個叢集的 GPU 活頁簿所列的計量預先設定的圖表。 請參閱 [容器 Azure 監視器中](container-insights-reports.md) 的活頁簿，以取得可供容器 Azure 監視器的活頁簿描述。

## <a name="next-steps"></a>後續步驟

- 若要瞭解如何部署包含已啟用 GPU 之節點的 AKS 叢集，請參閱在 Azure Kubernetes Service (AKS) [上使用 gpu 進行需要大量計算的工作負載](../../aks/gpu-cluster.md) 。

- 深入瞭解 [Microsoft Azure 中的 GPU 優化 VM sku](../../virtual-machines/sizes-gpu.md)。

- 查看 [Kubernetes 中的 gpu 支援](https://kubernetes.io/docs/tasks/manage-gpus/scheduling-gpus/) ，以深入瞭解如何在叢集中的一或多個節點上管理 Gpu 的 Kubernetes 實驗性支援。
