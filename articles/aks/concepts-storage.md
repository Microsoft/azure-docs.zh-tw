---
title: 概念 - Azure Kubernetes Service (AKS) 中的儲存體
description: 了解 Azure Kubernetes Service (AKS) 中的儲存體，包括磁碟區、永續性磁碟區、儲存體類別和宣告
services: container-service
ms.topic: conceptual
ms.date: 08/17/2020
ms.openlocfilehash: bf910c66694a62505f259c0a95a88f7dfed05d19
ms.sourcegitcommit: 02b1179dff399c1aa3210b5b73bf805791d45ca2
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/12/2021
ms.locfileid: "98127952"
---
# <a name="storage-options-for-applications-in-azure-kubernetes-service-aks"></a>Azure Kubernetes Service (AKS) 中的應用程式適用的儲存體選項

在 Azure Kubernetes Service (AKS) 中執行的應用程式可能需要儲存和擷取資料。 對於某些應用程式工作負載，此類資料儲存可以使用節點上在 Pod 刪除後即不再需要的本機快速儲存體。 其他應用程式工作負載則可能需要會在 Azure 平台內較一般的資料磁碟區上持續保存的儲存體。 多個 Pod 可能需要共用相同的資料磁碟區，或者，若 Pod 重新排程於不同節點上，則需要重新連結資料磁碟區。 最後，您可能需要將敏感性資料或應用程式設定資訊插入 Pod 中。

![Azure Kubernetes Service (AKS) 叢集中的應用程式適用的儲存體選項](media/concepts-storage/aks-storage-options.png)

本文將介紹為 AKS 中的應用程式提供儲存體的核心概念：

- [磁碟區](#volumes)
- [持續性磁片區](#persistent-volumes)
- [儲存類別](#storage-classes)
- [永續性磁碟區宣告](#persistent-volume-claims)

## <a name="volumes"></a>磁碟區

應用程式通常必須能夠儲存和擷取資料。 由於 Kubernetes 通常會將個別的 Pod 視為可處置的暫時性資源，因此可視需要以不同的方法讓應用程式使用和保存資料。 *磁碟區* 就是可跨 Pod 和應用程式生命週期儲存、擷取和保存資料的方式之一。

用來儲存和擷取資料的傳統磁碟區，會建立為受 Azure 儲存體支援的 Kubernetes 資源。 您可以手動建立這些資料磁碟區並直接將其指派給 Pod，或是由 Kubernetes 自動加以建立。 這些資料磁碟區可使用 Azure 磁碟或 Azure 檔案：

- *Azure 磁碟* 可用來建立 Kubernetes *DataDisk* 資源。 Azure 磁碟可以使用採用高效能 SSD 的 Azure 進階儲存體，或是採用一般 HDD 的 Azure 標準儲存體。 對於大部分的生產和開發工作負載，請使用進階儲存體。 Azure 磁片會掛接為 *>readwriteonce*，因此僅適用于單一 pod。 針對可同時由多個 pod 存取的儲存體磁片區，請使用 Azure 檔案儲存體。
- *Azure 檔案* 可用來將 Azure 儲存體帳戶所支援的 SMB 3.0 共用掛接至 Pod。 Azure 檔案可讓您在多個節點和 Pod 之間共用資料。 檔案可以使用標準 Hdd 所支援的 Azure 標準儲存體，或是由高效能 Ssd 支援的 Azure Premium 儲存體。

在 Kubernetes 中，磁碟區所代表的不只是可供儲存和擷取資訊的傳統磁碟。 Kubernetes 磁碟區也可用來將資料插入 Pod 中，供容器使用。 Kubernetes 中常見的其他磁碟區類型包括：

- *emptyDir* - 此磁碟區通常作為 Pod 的暫存空間。 Pod 內的所有容器都可存取磁碟區上的資料。 寫入此磁碟區類型的資料只會在 Pod 的存留期內保存 - 當 Pod 遭刪除後，磁碟區集會刪除。 此磁片區通常會使用基礎本機節點磁片儲存體，但它也只能存在於節點的記憶體中。
- *secret* - 此磁碟區可用來將敏感性資料插入 Pod 中，例如密碼。 首先，您必須使用 Kubernetes API 建立祕密。 您在定義 Pod 或部署時，系統可能會要求特定秘密。 對於有已排程的 Pod 需要秘密的節點，才會提供秘密，且秘密會儲存在 *tmpfs* 中，不會寫入至磁碟。 當節點上最後一個需要祕密的 Pod 遭刪除時，即會從該節點的 tmpfs 中刪除秘密。 祕密儲存在指定的命名空間內，且僅供相同命名空間中的 Pod 存取。
- *configMap* - 此磁碟區類型可用來將索引鍵/值組屬性插入 Pod 中，例如應用程式設定資訊。 您無須將應用程式設定資訊定義於容器映像內，而可以將其定義為 Kubernetes 資源，以便在 Pod 的新執行個體部署時為其更新和套用該資源。 和使用祕密一樣，您必須先使用 Kubernetes API 建立 ConfigMap。 隨後當您定義 Pod 或部署，即可要求此 ConfigMap。 ConfigMap 會儲存在指定的命名空間內，且僅供相同命名空間中的 Pod 存取。

## <a name="persistent-volumes"></a>永續性磁碟區

磁碟區會定義並建立為 Pod 生命週期的一部分，且在 Pod 刪除之後即不存在。 如果 Pod 在維護事件期間 (尤其是在 StatefulSet 中) 重新排程於不同的主機上，Pod 通常會預期其儲存體能持續保存。 *永續性磁碟區* (PV) 是由 Kubernetes API 建立和管理的儲存體資源，可跨個別 Pod 的存留期持續保存。

Azure 磁碟或檔案可用來提供 PersistentVolume。 如先前關於磁碟區的章節所說明，應選擇磁碟還是檔案，通常取決於資料的並行存取或效能層級的需求。

![Azure Kubernetes Service (AKS) 叢集中的永續性磁碟區](media/concepts-storage/persistent-volumes.png)

PersistentVolume 可由叢集管理員 *靜態* 建立，或由 Kubernetes API 伺服器 *動態* 建立。 如果 Pod 在排程後要求了目前無法使用的儲存體，Kubernetes 可以建立基礎 Azure 磁碟或檔案儲存體，並將其連結至 Pod。 動態佈建會使用 *StorageClass* 來識別需要建立的 Azure 儲存體類型。

## <a name="storage-classes"></a>儲存類別

若要定義不同層級的儲存體 (例如進階和標準)，您可以建立 *StorageClass*。 StorageClass 也會定義 *reclaimPolicy*。 此 reclaimPolicy 可控制基礎 Azure 儲存體資源在 Pod 刪除後和不再需要永續性磁碟區時的行為。 基礎儲存體資源可以刪除或保留供未來的 Pod 使用。

在 AKS 中， `StorageClasses` 會使用樹狀結構內儲存體外掛程式為叢集建立四個初始：

- `default` -使用 Azure StandardSSD 儲存體來建立受控磁片。 回收原則可確保在使用該磁片區的永久磁片區被刪除時，會刪除基礎的 Azure 磁片。
- `managed-premium` -使用 Azure Premium 儲存體來建立受控磁片。 再次回收原則可確保在使用該磁片區的永久磁片區被刪除時，會刪除基礎的 Azure 磁片。
- `azurefile` -使用 Azure 標準儲存體來建立 Azure 檔案共用。 回收原則可確保在刪除使用的永久磁片區時，會刪除基礎的 Azure 檔案共用。
- `azurefile-premium` -使用 Azure Premium 儲存體來建立 Azure 檔案共用。 回收原則可確保在刪除使用的永久磁片區時，會刪除基礎的 Azure 檔案共用。

針對使用新容器存放裝置介面的叢集 (CSI) external 外掛程式 (preview) 會建立下列其他項 `StorageClasses` ：
- `managed-csi` -使用 Azure StandardSSD 在本機重複的儲存體 (LRS) 來建立受控磁片。 回收原則可確保在使用該磁片區的永久磁片區被刪除時，會刪除基礎的 Azure 磁片。 儲存類別也會將持續性磁片區設定為可展開，您只需要以新的大小編輯持續性磁片區宣告。
- `managed-csi-premium` -使用 Azure Premium 本地區域冗余儲存體 (LRS) 來建立受控磁片。 再次回收原則可確保在使用該磁片區的永久磁片區被刪除時，會刪除基礎的 Azure 磁片。 同樣地，這個儲存類別可讓持續性磁片區展開。
- `azurefile-csi` -使用 Azure 標準儲存體來建立 Azure 檔案共用。 回收原則可確保在刪除使用的永久磁片區時，會刪除基礎的 Azure 檔案共用。
- `azurefile-csi-premium` -使用 Azure Premium 儲存體來建立 Azure 檔案共用。 回收原則可確保在刪除使用的永久磁片區時，會刪除基礎的 Azure 檔案共用。

若未指定永續性磁碟區的 StorageClass，將會使用預設 StorageClass。 要求永續性磁碟區時請多加留意，讓磁碟區使用您所需的適當儲存體。 您可以使用 `kubectl` 建立 StorageClass，以因應其他需求。 下列範例會使用進階受控磁碟，並指定在 Pod 刪除後應 *保留* 基礎 Azure 磁碟：

```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: managed-premium-retain
provisioner: kubernetes.io/azure-disk
reclaimPolicy: Retain
parameters:
  storageaccounttype: Premium_LRS
  kind: Managed
```

> [!NOTE]
> AKS 會協調預設的儲存類別，並覆寫您對這些儲存類別所做的任何變更。

## <a name="persistent-volume-claims"></a>永續性磁碟區宣告

PersistentVolumeClaim 會要求特定 StorageClass、存取模式和大小的磁碟或檔案儲存體。 若根據 StorageClass 中的定義並沒有現有的資源可用來履行宣告，Kubernetes API 伺服器可在 Azure 中動態佈建基礎儲存體資源。 在磁碟區連線至 Pod 後，Pod 定義即會包含磁碟區掛接。

![Azure Kubernetes Service (AKS) 叢集中的永續性磁碟區宣告](media/concepts-storage/persistent-volume-claims.png)

可用的儲存體資源指派給發出要求的 Pod 後，PersistentVolume 就會 *繫結* 至 PersistentVolumeClaim。 永續性磁碟區與宣告之間有 1:1 的對應。

下列範例 YAML 資訊清單顯示使用 *managed-premium* StorageClass 並要求 *5Gi* 磁碟的持續性磁碟區宣告：

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: azure-managed-disk
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: managed-premium
  resources:
    requests:
      storage: 5Gi
```

當您建立 Pod 定義時，您會指定永續性磁碟區宣告應要求所需的儲存體。 接著，您也會指定供應用程式用來讀取和寫入資料的 *volumeMount*。 下列範例 YAML 資訊清單說明如何使用先前的永續性磁碟區宣告在 */mnt/azure* 上掛階磁碟區：

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: nginx
spec:
  containers:
    - name: myfrontend
      image: mcr.microsoft.com/oss/nginx/nginx:1.15.5-alpine
      volumeMounts:
      - mountPath: "/mnt/azure"
        name: volume
  volumes:
    - name: volume
      persistentVolumeClaim:
        claimName: azure-managed-disk
```

若要在 Windows 容器中裝載磁片區，請指定磁碟機號和路徑。 例如：

```yaml
...      
       volumeMounts:
        - mountPath: "d:"
          name: volume
        - mountPath: "c:\k"
          name: k-dir
...
```

## <a name="next-steps"></a>後續步驟

如需相關的最佳做法，請參閱 [AKS 中的儲存和備份最佳做法][operator-best-practices-storage]。

若要了解如何建立使用 Azure 磁碟或 Azure 檔案的動態和靜態磁碟區，請參閱下列操作說明文章：

- [建立使用 Azure 磁碟的靜態磁碟區][aks-static-disks]
- [建立使用 Azure 檔案的靜態磁碟區][aks-static-files]
- [建立使用 Azure 磁碟的動態磁碟區][aks-dynamic-disks]
- [建立使用 Azure 檔案的動態磁碟區][aks-dynamic-files]

如需關於 Kubernetes 及 AKS 核心概念的詳細資訊，請參閱下列文章：

- [Kubernetes / AKS 叢集和工作負載][aks-concepts-clusters-workloads]
- [Kubernetes / AKS 身分識別][aks-concepts-identity]
- [Kubernetes / AKS 安全性][aks-concepts-security]
- [Kubernetes / AKS 虛擬網路][aks-concepts-network]
- [Kubernetes / AKS 調整][aks-concepts-scale]

<!-- EXTERNAL LINKS -->

<!-- INTERNAL LINKS -->
[aks-static-disks]: azure-disk-volume.md
[aks-static-files]: azure-files-volume.md
[aks-dynamic-disks]: azure-disks-dynamic-pv.md
[aks-dynamic-files]: azure-files-dynamic-pv.md
[aks-concepts-clusters-workloads]: concepts-clusters-workloads.md
[aks-concepts-identity]: concepts-identity.md
[aks-concepts-scale]: concepts-scale.md
[aks-concepts-security]: concepts-security.md
[aks-concepts-network]: concepts-network.md
[operator-best-practices-storage]: operator-best-practices-storage.md
