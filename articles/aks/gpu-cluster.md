---
title: Azure Kubernetes Service (AKS) 上的 GPU
description: 在 Azure Kubernetes Service (AKS) 上使用 GPU
services: container-service
author: lachie83
manager: jeconnoc
ms.service: container-service
ms.topic: article
ms.date: 04/05/2018
ms.author: laevenso
ms.custom: mvc
ms.openlocfilehash: 7ee5198b070fee6b6ce04d9fc2639ba23ae93296
ms.sourcegitcommit: c52123364e2ba086722bc860f2972642115316ef
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/11/2018
ms.locfileid: "34070561"
---
# <a name="using-gpus-on-aks"></a>在 AKS 上使用 GPU

AKS 支援建立已啟用 GPU 的節點集區。 Azure 目前提供單一或多個已啟用 GPU 的虛擬機器。 已啟用 GPU 的虛擬機器是專門針對計算密集型、圖形密集型及視覺效果的工作負載所設計。 已啟用 GPU 的虛擬機器清單可以在[這裡](https://docs.microsoft.com/azure/virtual-machines/windows/sizes-gpu)找到。

## <a name="create-an-aks-cluster"></a>建立 AKS 叢集

計算密集型工作負載 (例如圖形密集型和視覺效果工作負載) 通常需要 GPU。 請參考下列[文件](https://docs.microsoft.com/azure/virtual-machines/windows/sizes-gpu)以判斷您工作負載的正確虛擬機器大小。
針對您的 Azure Kubernetes Service (AKS) 節點，我們建議使用 `Standard_NC6` 大小的最小值。

> [!NOTE]
> 已啟用 GPU 的虛擬機器包含特定硬體，該特定硬體受限於較高的定價和區域可用性。 如需詳細資訊，請參閱[定價](https://azure.microsoft.com/pricing/)工具和[區域可用性](https://azure.microsoft.com/global-infrastructure/services/)網站。


如果您需要符合此最小值建議的 AKS 叢集，請執行下列命令。

建立叢集的資源群組。

```azurecli
az group create --name myGPUCluster --location eastus
```

以大小為 `Standard_NC6` 的節點建立 AKS 叢集。

```azurecli
az aks create --resource-group myGPUCluster --name myGPUCluster --node-vm-size Standard_NC6
```

連線到 AKS 叢集。

```azurecli
az aks get-credentials --resource-group myGPUCluster --name myGPUCluster
```

## <a name="confirm-gpus-are-schedulable"></a>確認 GPU 可排程

執行下列命令以確認 GPU 可以透過 Kubernetes 進行排程。

取得目前的節點清單。

```
$ kubectl get nodes
NAME                       STATUS    ROLES     AGE       VERSION
aks-nodepool1-22139053-0   Ready     agent     10h       v1.9.6
aks-nodepool1-22139053-1   Ready     agent     10h       v1.9.6
aks-nodepool1-22139053-2   Ready     agent     10h       v1.9.6
```

說明其中一個節點，以確認 GPU 可排程。 這可以在 `Capacity` 區段底下找到。 例如： `alpha.kubernetes.io/nvidia-gpu:  1`。

```
$ kubectl describe node aks-nodepool1-22139053-0
Name:               aks-nodepool1-22139053-0
Roles:              agent
Labels:             agentpool=nodepool1
                    beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/instance-type=Standard_NC6
                    beta.kubernetes.io/os=linux
                    failure-domain.beta.kubernetes.io/region=eastus
                    failure-domain.beta.kubernetes.io/zone=1
                    kubernetes.azure.com/cluster=MC_myGPUCluster_myGPUCluster
                    kubernetes.io/hostname=aks-nodepool1-22139053-0
                    kubernetes.io/role=agent
                    storageprofile=managed
                    storagetier=Standard_LRS
Annotations:        node.alpha.kubernetes.io/ttl=0
                    volumes.kubernetes.io/controller-managed-attach-detach=true
Taints:             <none>
CreationTimestamp:  Thu, 05 Apr 2018 12:13:20 -0700
Conditions:
  Type                 Status  LastHeartbeatTime                 LastTransitionTime                Reason                       Message
  ----                 ------  -----------------                 ------------------                ------                       -------
  NetworkUnavailable   False   Thu, 05 Apr 2018 12:15:07 -0700   Thu, 05 Apr 2018 12:15:07 -0700   RouteCreated                 RouteController created a route
  OutOfDisk            False   Thu, 05 Apr 2018 22:14:33 -0700   Thu, 05 Apr 2018 12:13:20 -0700   KubeletHasSufficientDisk     kubelet has sufficient disk space available
  MemoryPressure       False   Thu, 05 Apr 2018 22:14:33 -0700   Thu, 05 Apr 2018 12:13:20 -0700   KubeletHasSufficientMemory   kubelet has sufficient memory available
  DiskPressure         False   Thu, 05 Apr 2018 22:14:33 -0700   Thu, 05 Apr 2018 12:13:20 -0700   KubeletHasNoDiskPressure     kubelet has no disk pressure
  Ready                True    Thu, 05 Apr 2018 22:14:33 -0700   Thu, 05 Apr 2018 12:15:10 -0700   KubeletReady                 kubelet is posting ready status. AppArmor enabled
Addresses:
  InternalIP:  10.240.0.4
  Hostname:    aks-nodepool1-22139053-0
Capacity:
 alpha.kubernetes.io/nvidia-gpu:  1
 cpu:                             6
 memory:                          57691688Ki
 pods:                            110
Allocatable:
 alpha.kubernetes.io/nvidia-gpu:  1
 cpu:                             6
 memory:                          57589288Ki
 pods:                            110
System Info:
 Machine ID:                 2eb0e90bd1fe450ba3cf83479443a511
 System UUID:                CFB485B6-CB49-A545-A2C9-8E4C592C3273
 Boot ID:                    fea24544-596d-4246-b8c3-610fc7ac7280
 Kernel Version:             4.13.0-1011-azure
 OS Image:                   Debian GNU/Linux 9 (stretch)
 Operating System:           linux
 Architecture:               amd64
 Container Runtime Version:  docker://1.13.1
 Kubelet Version:            v1.9.6
 Kube-Proxy Version:         v1.9.6
PodCIDR:                     10.244.1.0/24
ExternalID:                  /subscriptions/8ecadfc9-d1a3-4ea4-b844-0d9f87e4d7c8/resourceGroups/MC_myGPUCluster_myGPUCluster/providers/Microsoft.Compute/virtualMachines/aks-nodepool1-22139053-0
Non-terminated Pods:         (2 in total)
  Namespace                  Name                       CPU Requests  CPU Limits  Memory Requests  Memory Limits
  ---------                  ----                       ------------  ----------  ---------------  -------------
  kube-system                kube-proxy-pwffr           100m (1%)     0 (0%)      0 (0%)           0 (0%)
  kube-system                kube-svc-redirect-mkpf4    0 (0%)        0 (0%)      0 (0%)           0 (0%)
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  CPU Requests  CPU Limits  Memory Requests  Memory Limits
  ------------  ----------  ---------------  -------------
  100m (1%)     0 (0%)      0 (0%)           0 (0%)
Events:         <none>
```

## <a name="run-a-gpu-enabled-workload"></a>執行已啟用 GPU 的工作負載

若要示範 GPU 確實在運作中，請使用適當的資源要求對已啟用 GPU 的工作負載進行排程。 這個範例會針對 [MNIST 資料集](http://yann.lecun.com/exdb/mnist/)執行 [Tensorflow](https://www.tensorflow.org/versions/r1.1/get_started/mnist/beginners) 作業。

下列作業資訊清單包含 `alpha.kubernetes.io/nvidia-gpu: 1` 的資源限制。 適當的 CUDA 程式庫和偵錯工具可以在位於 `/usr/local/nvidia` 的節點上取得，且必須使用如下所見的適當磁碟區規格裝載到 Pod。

複製資訊清單並且儲存為 **samples-tf-mnist-demo.yaml**。
```
apiVersion: batch/v1
kind: Job
metadata:
  labels:
    app: samples-tf-mnist-demo
  name: samples-tf-mnist-demo
spec:
  template:
    metadata:
      labels:
        app: samples-tf-mnist-demo
    spec:
      containers:
      - name: samples-tf-mnist-demo
        image: microsoft/samples-tf-mnist-demo:gpu
        args: ["--max_steps", "500"]
        imagePullPolicy: IfNotPresent
        resources:
          limits:
            alpha.kubernetes.io/nvidia-gpu: 1
        volumeMounts:
        - name: nvidia
          mountPath: /usr/local/nvidia
      restartPolicy: OnFailure
      volumes:
        - name: nvidia
          hostPath:
            path: /usr/local/nvidia
```

使用 [kubectl apply][kubectl-apply] 命令來執行作業。 此命令會剖析資訊清單檔，並建立已定義的 Kubernetes 物件。
```
$ kubectl apply -f samples-tf-mnist-demo.yaml
job "samples-tf-mnist-demo" created
```

使用 [kubectl get jobs][kubectl-get] 命令和 `--watch` 引數監視作業進度，直到成功完成為止。
```
$ kubectl get jobs samples-tf-mnist-demo --watch
NAME                    DESIRED   SUCCESSFUL   AGE
samples-tf-mnist-demo   1         0            8s
samples-tf-mnist-demo   1         1            35s
```

藉由顯示已完成的 Pod 來決定要檢視記錄的 Pod 名稱。
```
$ kubectl get pods --selector app=samples-tf-mnist-demo --show-all
NAME                          READY     STATUS      RESTARTS   AGE
samples-tf-mnist-demo-smnr6   0/1       Completed   0          4m
```

使用上述命令輸出中的 Pod 名稱來參考 Pod 記錄，以確認在此案例中已探索到適當的 GPU 裝置 (`Tesla K80`)。
```
$ kubectl logs samples-tf-mnist-demo-smnr6
2018-04-13 04:11:08.710863: I tensorflow/core/platform/cpu_feature_guard.cc:137] Your CPU supports instructions that this TensorFlow binary was not compiled to use: SSE4.1 SSE4.2 AVX AVX2 FMA
2018-04-13 04:11:15.824349: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1030] Found device 0 with properties:
name: Tesla K80 major: 3 minor: 7 memoryClockRate(GHz): 0.8235
pciBusID: 04e1:00:00.0
totalMemory: 11.17GiB freeMemory: 11.10GiB
2018-04-13 04:11:15.824394: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1120] Creating TensorFlow device (/device:GPU:0) -> (device: 0, name: Tesla K80, pci bus id: 04e1:00:00.0, compute capability: 3.7)
2018-04-13 04:11:20.891910: I tensorflow/stream_executor/dso_loader.cc:139] successfully opened CUDA library libcupti.so.8.0 locally
Successfully downloaded train-images-idx3-ubyte.gz 9912422 bytes.
Extracting /tmp/tensorflow/input_data/train-images-idx3-ubyte.gz
Successfully downloaded train-labels-idx1-ubyte.gz 28881 bytes.
Extracting /tmp/tensorflow/input_data/train-labels-idx1-ubyte.gz
Successfully downloaded t10k-images-idx3-ubyte.gz 1648877 bytes.
Extracting /tmp/tensorflow/input_data/t10k-images-idx3-ubyte.gz
Successfully downloaded t10k-labels-idx1-ubyte.gz 4542 bytes.
Extracting /tmp/tensorflow/input_data/t10k-labels-idx1-ubyte.gz
Accuracy at step 0: 0.0487
Accuracy at step 10: 0.6571
Accuracy at step 20: 0.8111
Accuracy at step 30: 0.8562
Accuracy at step 40: 0.8786
Accuracy at step 50: 0.8911
Accuracy at step 60: 0.8986
Accuracy at step 70: 0.9017
Accuracy at step 80: 0.9049
Accuracy at step 90: 0.9114
Adding run metadata for 99
Accuracy at step 100: 0.9109
Accuracy at step 110: 0.9143
Accuracy at step 120: 0.9188
Accuracy at step 130: 0.9194
Accuracy at step 140: 0.9237
Accuracy at step 150: 0.9231
Accuracy at step 160: 0.9158
Accuracy at step 170: 0.9259
Accuracy at step 180: 0.9303
Accuracy at step 190: 0.9315
Adding run metadata for 199
Accuracy at step 200: 0.9334
Accuracy at step 210: 0.9342
Accuracy at step 220: 0.9359
Accuracy at step 230: 0.9353
Accuracy at step 240: 0.933
Accuracy at step 250: 0.9353
Accuracy at step 260: 0.9408
Accuracy at step 270: 0.9396
Accuracy at step 280: 0.9406
Accuracy at step 290: 0.9444
Adding run metadata for 299
Accuracy at step 300: 0.9453
Accuracy at step 310: 0.946
Accuracy at step 320: 0.9464
Accuracy at step 330: 0.9472
Accuracy at step 340: 0.9516
Accuracy at step 350: 0.9473
Accuracy at step 360: 0.9502
Accuracy at step 370: 0.9483
Accuracy at step 380: 0.9481
Accuracy at step 390: 0.9467
Adding run metadata for 399
Accuracy at step 400: 0.9477
Accuracy at step 410: 0.948
Accuracy at step 420: 0.9496
Accuracy at step 430: 0.9501
Accuracy at step 440: 0.9534
Accuracy at step 450: 0.9551
Accuracy at step 460: 0.9518
Accuracy at step 470: 0.9562
Accuracy at step 480: 0.9583
Accuracy at step 490: 0.9575
Adding run metadata for 499
```

## <a name="cleanup"></a>清除
移除在這個步驟中建立的相關聯 Kubernetes 物件。
```
$ kubectl delete jobs samples-tf-mnist-demo
job "samples-tf-mnist-demo" deleted
```

## <a name="next-steps"></a>後續步驟

對於在 Kubernetes 上執行機器學習工作負載感到興趣嗎？ 如需詳細資訊，請參閱 Kubeflow labs。

> [!div class="nextstepaction"]
> [Kubeflow Labs][kubeflow-labs]

<!-- LINKS - external -->
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[kubeflow-labs]: https://github.com/Azure/kubeflow-labs
