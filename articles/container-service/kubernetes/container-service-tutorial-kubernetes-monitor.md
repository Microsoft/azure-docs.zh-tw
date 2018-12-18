---
title: Azure Container Service 教學課程 - 監視 Kubernetes
description: Azure Container Service 教學課程 - 使用 Log Analytics 監視 Kubernetes
services: container-service
author: neilpeterson
manager: jeconnoc
ms.service: container-service
ms.topic: tutorial
ms.date: 04/05/2018
ms.author: nepeters
ms.custom: mvc
ms.openlocfilehash: 6d178fe3caa121c51ef081708ca440113871938e
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46996905"
---
# <a name="monitor-a-kubernetes-cluster-with-log-analytics"></a>使用 Log Analytics 監視 Kubernetes 叢集

[!INCLUDE [aks-preview-redirect.md](../../../includes/aks-preview-redirect.md)]

監視 Kubernetes 叢集和容器很重要，尤其在您使用多個應用程式大規模管理生產叢集時。

您可以利用來自 Microsoft 或其他提供者的數個 Kubernetes 監視解決方案。 在本教學課程中，您會使用 [Log Analytics](../../operations-management-suite/operations-management-suite-overview.md) 中的容器解決方案 (Microsoft 的雲端式 IT 管理解決方案) 來監視 Kubernetes 叢集。 (容器解決方案處於預覽狀態。)

本教學課程 (七個章節的第七部分) 涵蓋下列工作：

> [!div class="checklist"]
> * 取得 Log Analytics 工作區設定
> * 在 Kubernetes 節點上設定 Log Analytics 代理程式
> * 在 Log Analytics 入口網站或 Azure 入口網站中存取監視資訊

## <a name="before-you-begin"></a>開始之前

在上一個教學課程中，應用程式已封裝成容器映像、這些映像已上傳至 Azure Container Registry，並已建立 Kubernetes 叢集。

如果您尚未完成這些步驟，而想要跟著做，請回到[教學課程 1 – 建立容器映像](./container-service-tutorial-kubernetes-prepare-app.md)。

## <a name="get-workspace-settings"></a>取得工作區設定

可以存取 [Log Analytics 入口網站](https://mms.microsoft.com)時，請移至 [設定] >  [已連線的服務] >  [Linux 伺服器]。 您可以在那裡找到「工作區識別碼」和主要或次要「工作區金鑰」。 請記下這些值，您在叢集上設定 Log Analytics 代理程式時需要用到這些值。

## <a name="create-kubernetes-secret"></a>建立 Kubernetes 祕密

使用 [kubectl create secret][kubectl-create-secret] 命令，將 Log Analytics 工作區設定儲存在名為 `omsagent-secret` 的 Kubernetes 祕密中。 以您的 Log Analytics 工作區識別碼更新 `WORKSPACE_ID`，並以工作區金鑰更新 `WORKSPACE_KEY`。

```console
kubectl create secret generic omsagent-secret --from-literal=WSID=WORKSPACE_ID --from-literal=KEY=WORKSPACE_KEY
```

## <a name="set-up-log-analytics-agents"></a>設定 Log Analytics 代理程式

下列 Kubernetes 資訊清單檔案可用於在 Kubernetes 叢集上設定容器監視代理程式。 它會建立 Kubernetes [DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)，以在每個叢集節點上執行單一相同的 pod。

將下列文字儲存到名為 `oms-daemonset.yaml` 的檔案。

```YAML
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
 name: omsagent
spec:
 template:
  metadata:
   labels:
    app: omsagent
    agentVersion: 1.4.3-174
    dockerProviderVersion: 1.0.0-30
  spec:
   containers:
     - name: omsagent
       image: "microsoft/oms"
       imagePullPolicy: Always
       securityContext:
         privileged: true
       ports:
       - containerPort: 25225
         protocol: TCP
       - containerPort: 25224
         protocol: UDP
       volumeMounts:
        - mountPath: /var/run/docker.sock
          name: docker-sock
        - mountPath: /var/log
          name: host-log
        - mountPath: /etc/omsagent-secret
          name: omsagent-secret
          readOnly: true
        - mountPath: /var/lib/docker/containers
          name: containerlog-path
       livenessProbe:
        exec:
         command:
         - /bin/bash
         - -c
         - ps -ef | grep omsagent | grep -v "grep"
        initialDelaySeconds: 60
        periodSeconds: 60
   nodeSelector:
    beta.kubernetes.io/os: linux
   # Tolerate a NoSchedule taint on master that ACS Engine sets.
   tolerations:
    - key: "node-role.kubernetes.io/master"
      operator: "Equal"
      value: "true"
      effect: "NoSchedule"
   volumes:
    - name: docker-sock
      hostPath:
       path: /var/run/docker.sock
    - name: host-log
      hostPath:
       path: /var/log
    - name: omsagent-secret
      secret:
       secretName: omsagent-secret
    - name: containerlog-path
      hostPath:
       path: /var/lib/docker/containers
```

使用下列命令建立 DaemonSet：

```azurecli-interactive
kubectl create -f oms-daemonset.yaml
```

若要查看是否已建立 DaemonSet，請執行：

```azurecli-interactive
kubectl get daemonset
```

輸出大致如下：

```azurecli-interactive
NAME       DESIRED   CURRENT   READY     UP-TO-DATE   AVAILABLE   NODE-SELECTOR   AGE
omsagent   3         3         3         0            3           <none>          5m
```

執行代理程式之後，Log Analytics 需要幾分鐘的時間來擷取及處理資料。

## <a name="access-monitoring-data"></a>存取監視資料

在 Log Analytics 入口網站或 Azure 入口網站中，使用[容器解決方案](../../log-analytics/log-analytics-containers.md)來檢視和分析容器監視資料。

若要使用 [Log Analytics 入口網站](https://mms.microsoft.com)來安裝容器解決方案，請移至**方案庫**。 然後新增 [Container 解決方案]。 或者，從 [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/microsoft.containersoms?tab=Overview) 新增 Containers 解決方案。

在 Log Analytics 入口網站中，尋找儀表板上的 [容器] 摘要圖格。 按一下此圖格以取得下列詳細資訊：容器事件、錯誤、狀態、映像清查，以及 CPU 和記憶體使用量。 如需更細微的資訊，請按一下任何圖格上的資料列，或執行[記錄搜尋](../../log-analytics/log-analytics-log-searches.md)。

![OMS 入口網站中的 Containers 儀表板](./media/container-service-tutorial-kubernetes-monitor/oms-containers-dashboard.png)

同樣地，在 Azure 入口網站中，移至 [Log Analytics] 並選取您的工作區名稱。 若要查看 [Containers] 摘要圖格，請按一下 [解決方案] > [Containers]。 若要查看詳細資料，按一下圖格。

請參閱 [Azure Log Analytics 文件](../../log-analytics/log-analytics-queries.md)，以取得查詢及分析監視資料的詳細指引。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已使用 Log Analytics 監視 Kubernetes 叢集。 涵蓋的工作包括：

> [!div class="checklist"]
> * 取得 Log Analytics 工作區設定
> * 在 Kubernetes 節點上設定 Log Analytics 代理程式
> * 在 Log Analytics 入口網站或 Azure 入口網站中存取監視資訊


用以下連結查看針對 Container Service 預先建立的指令碼範例。

> [!div class="nextstepaction"]
> [Azure Container Service 指令碼範例](cli-samples.md)
