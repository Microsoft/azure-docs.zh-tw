---
title: Azure 上的 Kubernetes - 叢集自動調整程式
description: 了解如何使用叢集自動調整程式與 Azure Kubernetes Service (AKS) 來根據需求自動縮放叢集。
services: container-service
author: sakthivetrivel
manager: jeconnoc
ms.service: container-service
ms.topic: article
ms.date: 07/19/18
ms.author: sakthivetrivel
ms.custom: mvc
ms.openlocfilehash: 3bac6534f43d62e6eb9381b8513025ba9117ed04
ms.sourcegitcommit: 67abaa44871ab98770b22b29d899ff2f396bdae3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/08/2018
ms.locfileid: "48857001"
---
# <a name="cluster-autoscaler-on-azure-kubernetes-service-aks---preview"></a>Azure Kubernetes Service (AKS) 上的叢集自動調整程式 - 預覽

Azure Kubernetes Service (AKS) 會提供可在 Azure 中部署受控 Kubernetes 叢集的靈活解決方案。 當資源需求增加時，叢集自動調整程式會根據您設定的條件，讓叢集擴大到符合該需求的大小。 叢集自動調整程式 (CA) 執行此作業的方式是，根據擱置中的 Pod 來縮放代理程式節點。 它會定期掃描叢集，以檢查擱置中的 Pod 或空白節點，然後在允許的情況下增加大小。 根據預設，CA 會每 10 秒掃描一次擱置中的 Pod，並將超過 10 分鐘沒用到的節點移除。 與[水平 Pod 自動調整程式](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/) (HPA) 搭配使用時，HPA 會根據需求更新 Pod 複本和資源。 如果沒有足夠的節點或非必要節點可配合此 Pod 縮放，則 CA 會回應並在一組新的節點上排定 Pod。

本文說明如何在代理程式節點上部署叢集自動調整程式。 不過，叢集自動調整程式已部署在 kube-system 命名空間中，因此自動調整程式將不會相應縮小執行該 pod 的節點。

> [!IMPORTANT]
> Azure Kubernetes Service (AKS) 叢集自動調整程式整合目前為**預覽**狀態。 若您同意[補充的使用規定](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)即可取得預覽。 在公開上市 (GA) 之前，此功能的某些領域可能會變更。
>

## <a name="prerequisites"></a>必要條件

本文件假設您有已啟用 RBAC 的 AKS 叢集。 如果您需要 AKS 叢集，請參閱 [Azure Kubernetes Service (AKS) 快速入門][aks-quick-start]。

 若要使用叢集自動調整程式，您的叢集必須使用 Kubernetes v1.10.X 或更高版本，而且必須啟用 RBAC。 若要升級您的叢集，請參閱[升級 AKS 叢集][aks-upgrade]上的文章。

## <a name="gather-information"></a>收集資訊

若要為叢集自動調整程式產生權限以在您的叢集中執行，請執行此 bash 指令碼：

```sh
#! /bin/bash
ID=`az account show --query id -o json`
SUBSCRIPTION_ID=`echo $ID | tr -d '"' `

TENANT=`az account show --query tenantId -o json`
TENANT_ID=`echo $TENANT | tr -d '"' | base64`

read -p "What's your cluster name? " cluster_name
read -p "Resource group name? " resource_group

CLUSTER_NAME=`echo $cluster_name | base64`
RESOURCE_GROUP=`echo $resource_group | base64`

PERMISSIONS=`az ad sp create-for-rbac --role="Contributor" --scopes="/subscriptions/$SUBSCRIPTION_ID" -o json`
CLIENT_ID=`echo $PERMISSIONS | sed -e 's/^.*"appId"[ ]*:[ ]*"//' -e 's/".*//' | base64`
CLIENT_SECRET=`echo $PERMISSIONS | sed -e 's/^.*"password"[ ]*:[ ]*"//' -e 's/".*//' | base64`

SUBSCRIPTION_ID=`echo $ID | tr -d '"' | base64 `

NODE_RESOURCE_GROUP=`az aks show --name $cluster_name  --resource-group $resource_group -o tsv --query 'nodeResourceGroup' | base64`

echo "---
apiVersion: v1
kind: Secret
metadata:
    name: cluster-autoscaler-azure
    namespace: kube-system
data:
    ClientID: $CLIENT_ID
    ClientSecret: $CLIENT_SECRET
    ResourceGroup: $RESOURCE_GROUP
    SubscriptionID: $SUBSCRIPTION_ID
    TenantID: $TENANT_ID
    VMType: QUtTCg==
    ClusterName: $CLUSTER_NAME
    NodeResourceGroup: $NODE_RESOURCE_GROUP
---"
```

在指令碼中依照步驟執行之後，指令碼會以祕密形式輸出您的詳細資料，如下所示：

```yaml
---
apiVersion: v1
kind: Secret
metadata:
  name: cluster-autoscaler-azure
  namespace: kube-system
data:
  ClientID: <base64-encoded-client-id>
  ClientSecret: <base64-encoded-client-secret>$
  ResourceGroup: <base64-encoded-resource-group>  SubscriptionID: <base64-encode-subscription-id>
  TenantID: <base64-encoded-tenant-id>
  VMType: QUtTCg==
  ClusterName: <base64-encoded-clustername>
  NodeResourceGroup: <base64-encoded-node-resource-group>
---
```

接下來，執行下列命令可取得您節點集區的名稱。 

```console
$ kubectl get nodes --show-labels
```

輸出：

```console
NAME                       STATUS    ROLES     AGE       VERSION   LABELS
aks-nodepool1-37756013-0   Ready     agent     1h        v1.10.3   agentpool=nodepool1,beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=Standard_DS1_v2,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/region=eastus,failure-domain.beta.kubernetes.io/zone=0,kubernetes.azure.com/cluster=MC_[resource-group]\_[cluster-name]_[location],kubernetes.io/hostname=aks-nodepool1-37756013-0,kubernetes.io/role=agent,storageprofile=managed,storagetier=Premium_LRS
 ```

然後，擷取 **agentpool** 標籤的值。 叢集的預設節點集區名稱是 "nodepool1"。

現在使用您的祕密和節點集區，即可建立部署圖表。

## <a name="create-a-deployment-chart"></a>建立部署圖表

建立名為 `aks-cluster-autoscaler.yaml` 的檔案，然後將下列 YAML 程式碼複製到其中。

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    k8s-addon: cluster-autoscaler.addons.k8s.io
    k8s-app: cluster-autoscaler
  name: cluster-autoscaler
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  name: cluster-autoscaler
  labels:
    k8s-addon: cluster-autoscaler.addons.k8s.io
    k8s-app: cluster-autoscaler
rules:
- apiGroups: [""]
  resources: ["events","endpoints"]
  verbs: ["create", "patch"]
- apiGroups: [""]
  resources: ["pods/eviction"]
  verbs: ["create"]
- apiGroups: [""]
  resources: ["pods/status"]
  verbs: ["update"]
- apiGroups: [""]
  resources: ["endpoints"]
  resourceNames: ["cluster-autoscaler"]
  verbs: ["get","update"]
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["watch","list","get","update"]
- apiGroups: [""]
  resources: ["pods","services","replicationcontrollers","persistentvolumeclaims","persistentvolumes"]
  verbs: ["watch","list","get"]
- apiGroups: ["extensions"]
  resources: ["replicasets","daemonsets"]
  verbs: ["watch","list","get"]
- apiGroups: ["policy"]
  resources: ["poddisruptionbudgets"]
  verbs: ["watch","list"]
- apiGroups: ["apps"]
  resources: ["statefulsets"]
  verbs: ["watch","list","get"]
- apiGroups: ["storage.k8s.io"]
  resources: ["storageclasses"]
  verbs: ["get", "list", "watch"]

---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: Role
metadata:
  name: cluster-autoscaler
  namespace: kube-system
  labels:
    k8s-addon: cluster-autoscaler.addons.k8s.io
    k8s-app: cluster-autoscaler
rules:
- apiGroups: [""]
  resources: ["configmaps"]
  verbs: ["create"]
- apiGroups: [""]
  resources: ["configmaps"]
  resourceNames: ["cluster-autoscaler-status"]
  verbs: ["delete","get","update"]

---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: cluster-autoscaler
  labels:
    k8s-addon: cluster-autoscaler.addons.k8s.io
    k8s-app: cluster-autoscaler
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-autoscaler
subjects:
  - kind: ServiceAccount
    name: cluster-autoscaler
    namespace: kube-system

---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: RoleBinding
metadata:
  name: cluster-autoscaler
  namespace: kube-system
  labels:
    k8s-addon: cluster-autoscaler.addons.k8s.io
    k8s-app: cluster-autoscaler
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: cluster-autoscaler
subjects:
  - kind: ServiceAccount
    name: cluster-autoscaler
    namespace: kube-system

---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  labels:
    app: cluster-autoscaler
  name: cluster-autoscaler
  namespace: kube-system
spec:
  replicas: 1
  selector:
    matchLabels:
      app: cluster-autoscaler
  template:
    metadata:
      labels:
        app: cluster-autoscaler
    spec:
      serviceAccountName: cluster-autoscaler
      containers:
      - image: gcr.io/google-containers/cluster-autoscaler:v1.2.2
        imagePullPolicy: Always
        name: cluster-autoscaler
        resources:
          limits:
            cpu: 100m
            memory: 300Mi
          requests:
            cpu: 100m
            memory: 300Mi
        command:
        - ./cluster-autoscaler
        - --v=3
        - --logtostderr=true
        - --cloud-provider=azure
        - --skip-nodes-with-local-storage=false
        - --nodes=1:10:nodepool1
        env:
        - name: ARM_SUBSCRIPTION_ID
          valueFrom:
            secretKeyRef:
              key: SubscriptionID
              name: cluster-autoscaler-azure
        - name: ARM_RESOURCE_GROUP
          valueFrom:
            secretKeyRef:
              key: ResourceGroup
              name: cluster-autoscaler-azure
        - name: ARM_TENANT_ID
          valueFrom:
            secretKeyRef:
              key: TenantID
              name: cluster-autoscaler-azure
        - name: ARM_CLIENT_ID
          valueFrom:
            secretKeyRef:
              key: ClientID
              name: cluster-autoscaler-azure
        - name: ARM_CLIENT_SECRET
          valueFrom:
            secretKeyRef:
              key: ClientSecret
              name: cluster-autoscaler-azure
        - name: ARM_VM_TYPE
          valueFrom:
            secretKeyRef:
              key: VMType
              name: cluster-autoscaler-azure
        - name: AZURE_CLUSTER_NAME
          valueFrom:
            secretKeyRef:
              key: ClusterName
              name: cluster-autoscaler-azure
        - name: AZURE_NODE_RESOURCE_GROUP
          valueFrom:
            secretKeyRef:
              key: NodeResourceGroup
              name: cluster-autoscaler-azure
      restartPolicy: Always
```

複製並貼上先前步驟中建立的秘密，然後將其插入檔案的開頭。

接下來，若要設定節點的範圍，請以 MIN:MAX:NODE_POOL_NAME 的形式填寫 `command` 底下的 `--nodes` 引數。 例如：`--nodes=3:10:nodepool1` 就是將最小節點數目設為 3，將最大節點數目設為 10，以及將節點集區名稱設為 nodepool1。

然後，在 **containers** 下的 [映像] 欄位中，填入要使用的叢集自動調整程式版本。 AKS 必須是 v1.2.2 版或更新版本。 此範例使用的是 v1.2.2 版。

## <a name="deployment"></a>部署

執行下列命令來部署叢集自動調整程式

```console
kubectl create -f aks-cluster-autoscaler.yaml
```

若要檢查叢集自動調整程式是否正在執行，請使用下列命令並檢查 Pod 清單。 應該要有前面加上 "cluster-autoscaler" 的 pod 執行中。 如果您看到此 Pod，您的叢集自動調整程式便已部署。

```console
kubectl -n kube-system get pods
```

若要檢視叢集自動調整程式的狀態，請執行

```console
kubectl -n kube-system describe configmap cluster-autoscaler-status
```

## <a name="interpreting-the-cluster-autoscaler-status"></a>解譯叢集自動調整程式狀態

```console
$ kubectl -n kube-system describe configmap cluster-autoscaler-status
Name:         cluster-autoscaler-status
Namespace:    kube-system
Labels:       <none>
Annotations:  cluster-autoscaler.kubernetes.io/last-updated=2018-07-25 22:59:22.661669494 +0000 UTC

Data
====
status:
----
Cluster-autoscaler status at 2018-07-25 22:59:22.661669494 +0000 UTC:
Cluster-wide:
  Health:      Healthy (ready=1 unready=0 notStarted=0 longNotStarted=0 registered=1 longUnregistered=0)
               LastProbeTime:      2018-07-25 22:59:22.067828801 +0000 UTC
               LastTransitionTime: 2018-07-25 00:38:36.41372897 +0000 UTC
  ScaleUp:     NoActivity (ready=1 registered=1)
               LastProbeTime:      2018-07-25 22:59:22.067828801 +0000 UTC
               LastTransitionTime: 2018-07-25 00:38:36.41372897 +0000 UTC
  ScaleDown:   NoCandidates (candidates=0)
               LastProbeTime:      2018-07-25 22:59:22.067828801 +0000 UTC
               LastTransitionTime: 2018-07-25 00:38:36.41372897 +0000 UTC

NodeGroups:
  Name:        nodepool1
  Health:      Healthy (ready=1 unready=0 notStarted=0 longNotStarted=0 registered=1 longUnregistered=0 cloudProviderTarget=1 (minSize=1, maxSize=5))
               LastProbeTime:      2018-07-25 22:59:22.067828801 +0000 UTC
               LastTransitionTime: 2018-07-25 00:38:36.41372897 +0000 UTC
  ScaleUp:     NoActivity (ready=1 cloudProviderTarget=1)
               LastProbeTime:      2018-07-25 22:59:22.067828801 +0000 UTC
               LastTransitionTime: 2018-07-25 00:38:36.41372897 +0000 UTC
  ScaleDown:   NoCandidates (candidates=0)
               LastProbeTime:      2018-07-25 22:59:22.067828801 +0000 UTC
               LastTransitionTime: 2018-07-25 00:38:36.41372897 +0000 UTC


Events:  <none>
```

叢集自動調整程式狀態可讓您查看在兩個不同層級的叢集自動調整程式狀態：整個叢集，以及每個節點群組內。 AKS 目前只支援一個節點集區，因此這些計量都相同。

* 健康情況表示節點的整體健康情況。 如果叢集自動調整程式努力建立或移除叢集中的節點，則此狀態會變更為「狀況不良」。 另外還有不同節點的狀態分解：
    * "Ready" 表示節點已準備好在其上排定 pod。
    * "Unready" 表示節點在啟動之後失效。
    * "NotStarted" 表示節點尚未完全啟動。
    * "LongNotStarted" 表示節點無法在合理的限制內啟動。
    * "Registered" 表示節點已在群組中註冊
    * "Unregistered" 表示節點存在於叢集提供者端，但無法在 Kubernetes 中註冊。
  
* ScaleUp 可讓您檢查叢集自動調整程式判斷叢集中何時應該發生相應增加。
    * 轉換就是當叢集中的節點數目變更時，或節點的狀態變更時。
    * 就緒節點數目就是叢集中可立即使用的節點數目。 
    * CloudProviderTarget 是叢集自動調整程式判定叢集必須處理其工作負載的節點數目。

* ScaleDown 可讓您檢查是否有要相應減少的候選項目。 
    * 相應減少的候選項目是叢集自動調整程式判定可以移除的節點，而且不會影響叢集處理其工作負載的能力。 
    * 所提供的時間顯示上次檢查叢集是否有相應減少候選項目及其上次轉換時間的時間。

最後，在 Events 之下，您可看見叢集自動調整程式已執行的所有相應增加或相應減少事件 (不論失敗或成功) 及其時間。

## <a name="next-steps"></a>後續步驟

若要搭配使用叢集自動調整程式與水平 Pod 自動調整程式，請參閱[縮放 Kubernetes 應用程式和基礎結構][aks-tutorial-scale]。

使用 AKS 教學課程深入了解部署和管理 AKS。

> [!div class="nextstepaction"]
> [AKS 教學課程][aks-tutorial-prepare-app]

<!-- LINKS - internal -->
[aks-quick-start]: ./kubernetes-walkthrough.md
[aks-tutorial-prepare-app]: ./tutorial-kubernetes-prepare-app.md
[aks-tutorial-scale]: ./tutorial-kubernetes-scale.md
[aks-upgrade]: ./upgrade-cluster.md

<!-- LINKS - external -->
[cluster-autoscale]: https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/FAQ.md
