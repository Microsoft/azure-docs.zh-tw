---
title: 如何調整 Machine Learning 的 Azure Container Service 叢集大小 | Microsoft Docs
description: 調整 ACS 叢集 - 自動縮放比例和靜態縮放比例；調整叢集中的節點數目
services: machine-learning
author: aashishb
ms.author: aashishb
manager: mwinkle
ms.reviewer: jmartens, jasonwhowell, mldocs
ms.service: machine-learning
ms.component: core
ms.workload: data-services
ms.custom: mvc
ms.topic: article
ms.date: 10/04/2017
ROBOTS: NOINDEX
ms.openlocfilehash: 718f351ad4ce325f3585ebcbe7adfc4b5905e734
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46998605"
---
# <a name="scaling-the-cluster-to-manage-web-service-throughput"></a>調整叢集以管理 web 服務輸送量

[!INCLUDE [workbench-deprecated](../../../includes/aml-deprecating-preview-2017.md)]


## <a name="why-scale-the-cluster"></a>為什麼要調整叢集？

調整 Azure Container Service (ACS) 叢集是將服務輸送量最佳化，同時保持最低的叢集大小以降低成本的有效方式。 

若要進一步了解自動縮放比例，請考慮下列範例執行三個 web 服務的叢集：

![範例：叢集上的三個服務](media/how-to-scale-clusters/three-services.png)

服務具備各種尖峰需求：服務 1 (藍線) 在尖峰要求時需要 40 個 Pod，服務 2 (橘線) 在尖峰時需要 38 個，而服務 3 (灰線) 在尖峰時需要 50 個。 如果您個別保留每個服務所需的尖峰容量，此叢集總共就需要至少 40 + 38 + 50 = 128 個 Pod。

但是，請考慮任何時間點的實際 Pod 使用量，在圖表中以黑色的虛線表示。 在此情況下，在任何一個時間使用的最大 Pod 數目為 64，發生於服務 3 的尖峰 20:00 時。 此時，服務 3 會使用 50 個 Pod，但服務 2 只會使用 9 個 Pod，而服務 1 只使用 5 個 Pod。 請記住，這是此叢集的尖峰使用量。 這表示，叢集絕不會使用超過 64 個 Pod - 128 個 pod 的一半導出需求，用於針對尖峰使用量獨立縮放的三種服務。

藉由重新指派叢集中的 Pod -- 也就是重新縮放比例 -- 來符合每個服務的目前要求，而不只是要求所有服務在尖峰需求時的足夠資源，您就可以減少叢集大小。 在這個簡單的範例中，自動縮放比例會將所需的 Pod 數目從 128 減少為 64，將必要的叢集大小縮減為一半。

調整 Pod 的數目是相當快速的作業，需要不到一分鐘，因此不會嚴重影響服務的回應性。

> [!NOTE]
> 調整叢集對於要求延遲問題沒有幫助。 基於作業目的，相應增加會增加成功的次數，並降低服務無法使用的錯誤。 

## <a name="how-to-scale-web-services-on-your-acs-cluster"></a>如何調整 ACS 叢集上的 web 服務

模型管理 CLI 的叢集安裝選項依預設會設定您環境中的兩個代理程式和一個 Pod。 它也會安裝 Kubernetes CLI。

您可以將已部署的 web 服務調整為 ACS，方法為調整：

* 叢集中的代理程式節點數目
* 代理程式節點上執行的 Kubernetes Pod 複本數目

### <a name="scaling-the-number-of-nodes-in-the-cluster"></a>調整叢集中的節點數目

下列命令會設定叢集中的代理程式節點計數：

```
az acs scale -g <resource group> -n <cluster name> --new-agent-count <new scale>
```

這需要幾分鐘才能完成。 如需有關調整叢集中節點數目的詳細資訊，請參閱[調整 Container Service 叢集中的代理程式節點](https://docs.microsoft.com/azure/container-service/container-service-scale)。

### <a name="scaling-the-number-of-kubernetes-pod-replicas-in-a-cluster"></a>調整叢集中 Kubernetes Pod 複本的數目
 
您可以使用 Azure Machine Learning CLI 或 [Kubernetes 儀表板] (https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/)來調整指派給叢集的 Pod 複本數目 。

如需有關 Kubernetes 複本 Pod 的詳細資訊，請參閱 [Kubernetes Pod](https://kubernetes.io/docs/concepts/workloads/pods/pod/) 文件。

#### <a name="scaling-a-cluster-with-the-azure-machine-learning-cli"></a>使用 Azure Machine Learning CLI 調整叢集

有兩種方式可以使用 CLI 來調整叢集：

- Autoscale
- 靜態縮放比例

服務建立時，自動縮放比例預設為作用中，且在大部分情況下是偏好的縮放比例方法。

##### <a name="autoscale"></a>Autoscale

下列命令會啟用自動縮放比例，並設定服務複本的最小和最大數目。

```
az ml service update realtime -i <service id> --autoscale-enabled true --autoscale-min-replicas <positive number> --autoscale-max-replicas <positive number>
```

例如，將 `autoscale-min-replicas` 設定為 5 會建立五個複本。 若要達到 Web 服務的最佳數目，將值設定為例如 10 的數目，並監視 503 錯誤訊息的數目。 然後據以調整數字。


| 參數名稱 | 類型 | 說明 |
|--------------------|--------------------|--------------------|
| `autoscale-enabled` | 布林值 | 指定是否啟用自動縮放比例。 預設值：True |
| `autoscale-min-replicas` | integer | 指定 Pod 的最小數目。 必須是 0 或更大。 預設值：1 |
| `autoscale-max-replicas` | integer | 指定 Pod 的最大數目。 必須是 1 或更大。 如果 autoscale-max-replicas 小於 autoscale-min-replicas，將會忽略 autoscale-max-replicas。 預設值：10 |
| `autoscale-refresh-period-seconds` | integer | 指定自動縮放比例重新整理之間的期間 (以秒計)。 預設值：1 |
| `autoscale-target-utilization` | integer | 指定自動縮放比例作為目標的使用量百分比 (1 到 100 之間)。 預設值：70 |

自動縮放比例的運作可確保下列兩種情況：

1. 符合目標使用率
2. 縮放比例一律不得超過最小和最大設定

叢集中的服務會競用叢集資源。 自動縮放比例服務會隨著其每秒的需求 (RPS) 增加而增加其叢集資源使用方式，並隨著 RPS 減少而緩慢釋放資源。 只要這類要取得服務的資源存在，就會視需要取得叢集資源。

如需有關如何使用自動縮放比例參數的詳細資訊，請參閱[模型管理命令列介面參考](model-management-cli-reference.md)文件。

##### <a name="static-scale"></a>靜態縮放比例

一般而言，應該避免靜態縮放比例，因為它不允許自動縮放比例的叢集大小縮減。 即便如此，在某些情況下，可能會建議使用靜態縮放比例。 例如，當叢集專屬於單一服務時，自動縮放比例無法提供任何好處；所有叢集資源都應該指派給該服務。

若要以靜態方式調整叢集，必須關閉自動縮放比例。 使用下列命令來停用自動縮放比例︰

```
az ml service update realtime -i <service id> --autoscale-enabled false
```

關閉自動縮放比例後，下列命令會直接縮放服務複本的數目。

```
az ml service update realtime -i <service id> -z <replica count>
```
 
如需有關調整叢集中節點數目的詳細資訊，請參閱調整 Container Service 叢集中的代理程式節點。

#### <a name="scaling-number-of-replicas-using-the-kubernetes-dashboard"></a>使用 Kubernetes 儀表板來調整複本數目

在命令列中，輸入：

```
kubectl proxy
```

在 Windows 中，Kubernetes 安裝位置不會自動新增至路徑。 請先瀏覽至安裝資料夾：

```
c:\users\<user name>\bin
```

一旦執行命令後，您應該會看到下列資訊的訊息：

```
Starting to serve on 127.0.0.1:8001
```

如果連接埠已在使用中，您會看到類似下列範例的訊息：

```
F0612 21:49:22.459111   59621 proxy.go:137] listen tcp 127.0.0.1:8001: bind: address already in use
```

您可以使用 --port 參數來指定替代連接埠號碼。

```
kubectl proxy --port=8010
Starting to serve on 127.0.0.1:8010
```

一旦您啟動儀表板伺服器後，開啟瀏覽器，並輸入下列 URL：

```
127.0.0.1:<port number>/ui
```

從儀表板的主畫面上，按一下左側導覽列上的 [部署]。 如果未顯示瀏覽窗格，請在左上方選取 ![包含三個短水平線的功能表](media/how-to-scale-clusters/icon-hamburger.png) 這個圖示。

找出要修改的部署，然後按一下右邊的這個圖示 ![包含三個垂直點的功能表圖示](media/how-to-scale-clusters/icon-kebab.png)，然後按一下 [View/Edit YAML] \(檢視/編輯 YAML\)。

在 [編輯部署] 畫面上，找出規格節點、修改複本值，並按一下 [更新]。
