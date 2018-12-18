---
title: 使用 Open Service Broker for Azure (OSBA) 與 Azure 受控服務整合
description: 使用 Open Service Broker for Azure (OSBA) 與 Azure 受控服務整合
services: container-service
author: sozercan
manager: jeconnoc
ms.service: container-service
ms.topic: overview
ms.date: 12/05/2017
ms.author: seozerca
ms.openlocfilehash: d0b6fc1ebd08b29b9acc28cfb0107b815c7d7bad
ms.sourcegitcommit: 7b0778a1488e8fd70ee57e55bde783a69521c912
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/10/2018
ms.locfileid: "49068232"
---
# <a name="integrate-with-azure-managed-services-using-open-service-broker-for-azure-osba"></a>使用 Open Service Broker for Azure (OSBA) 與 Azure 受控服務整合

Open Service Broker for Azure (OSBA) 可以與 [Kubernetes 服務類別目錄][kubernetes-service-catalog]搭配使用，允許開發人員利用 Kubernetes 中的 Azure 受控服務。 本指南著重於部署 Kubernetes 服務類別目錄、Open Service Broker for Azure (OSBA)，以及利用 Kubernetes 使用 Azure 受控服務的應用程式。

## <a name="prerequisites"></a>必要條件
* Azure 訂用帳戶

* Azure CLI：[在本機進行安裝][azure-cli-install]，或用於 [Azure Cloud Shell][azure-cloud-shell]。

* Helm CLI 2.7+：[在本機進行安裝][helm-cli-install]，或用於 [Azure Cloud Shell][azure-cloud-shell]。

* 在 Azure 訂用帳戶上使用參與者角色建立服務主體的權限

* 現有的 Azure Kubernetes Service (AKS) 叢集。 如果您需要 AKS 叢集，請依照[建立 AKS 叢集][create-aks-cluster]快速入門進行。

## <a name="install-service-catalog"></a>安裝服務類別目錄

第一步是在 Kubernetes 叢集中，使用 Helm 圖表安裝服務類別目錄。 使用下列方式升級叢集中的 Tiller (Helm 伺服器) 安裝：

```azurecli-interactive
helm init --upgrade
```

現在，將服務類別目錄圖表新增至 Helm 存放庫：

```azurecli-interactive
helm repo add svc-cat https://svc-catalog-charts.storage.googleapis.com
```

最後，安裝包含 Helm 圖表的服務類別目錄。 如果您的叢集已啟用 RBAC，請執行此命令。

```azurecli-interactive
helm install svc-cat/catalog --name catalog --namespace catalog --set controllerManager.healthcheck.enabled=false
```

如果您的叢集未啟用 RBAC，請執行此命令。

```azurecli-interactive
helm install svc-cat/catalog --name catalog --namespace catalog --set rbacEnable=false --set controllerManager.healthcheck.enabled=false
```

執行 Helm 圖表之後，請確認 `servicecatalog` 出現在下列命令的輸出中：

```azurecli-interactive
kubectl get apiservice
```

例如，您應該會看到如下的輸出 (這裡顯示截斷的結果)：

```
NAME                                 AGE
v1.                                  10m
v1.authentication.k8s.io             10m
...
v1beta1.servicecatalog.k8s.io        34s
v1beta1.storage.k8s.io               10
```

## <a name="install-open-service-broker-for-azure"></a>安裝 Open Service Broker for Azure

下一步是安裝 [Open Service Broker for Azure][open-service-broker-azure]，其中包含用於 Azure 受控服務的類別目錄。 可用 Azure 服務的範例包括適用於 PostgreSQL 的 Azure 資料庫、適用於 MySQL 的 Azure 資料庫和 Azure SQL Database。

從新增 Open Service Broker for Azure Helm 存放庫開始：

```azurecli-interactive
helm repo add azure https://kubernetescharts.blob.core.windows.net/azure
```

使用下列 Azure CLI 命令建立[服務主體][create-service-principal]：

```azurecli-interactive
az ad sp create-for-rbac
```

輸出應該類似下面這樣。 請記下 `appId`、`password` 和 `tenant` 值，您在下一個步驟中將會用到這些值。

```JSON
{
  "appId": "7248f250-0000-0000-0000-dbdeb8400d85",
  "displayName": "azure-cli-2017-10-15-02-20-15",
  "name": "http://azure-cli-2017-10-15-02-20-15",
  "password": "77851d2c-0000-0000-0000-cb3ebc97975a",
  "tenant": "72f988bf-0000-0000-0000-2d7cd011db47"
}
```

使用上述的值設定下列環境變數：

```azurecli-interactive
AZURE_CLIENT_ID=<appId>
AZURE_CLIENT_SECRET=<password>
AZURE_TENANT_ID=<tenant>
```

現在，取得您的 Azure 訂用帳戶識別碼：

```azurecli-interactive
az account show --query id --output tsv
```

再次使用上述的值設定下列環境變數：

```azurecli-interactive
AZURE_SUBSCRIPTION_ID=[your Azure subscription ID from above]
```

既然您已填入這些環境變數，請執行下列命令，使用 Helm 圖表安裝 Open Service Broker for Azure：

```azurecli-interactive
helm install azure/open-service-broker-azure --name osba --namespace osba \
    --set azure.subscriptionId=$AZURE_SUBSCRIPTION_ID \
    --set azure.tenantId=$AZURE_TENANT_ID \
    --set azure.clientId=$AZURE_CLIENT_ID \
    --set azure.clientSecret=$AZURE_CLIENT_SECRET
```

一旦 OSBA 部署完成，請安裝 [Service Catalog CLI][service-catalog-cli]，這是一個易於使用的命令列介面，可用於查詢 Service Broker、服務類別、服務方案等等。

執行下列命令以安裝 Service Catalog CLI 二進位檔：

```azurecli-interactive
curl -sLO https://servicecatalogcli.blob.core.windows.net/cli/latest/$(uname -s)/$(uname -m)/svcat
chmod +x ./svcat
```

現在，列出已安裝的 Service Broker:

```azurecli-interactive
./svcat get brokers
```

您應該會看到如下所示的輸出：

```
  NAME                               URL                                STATUS
+------+--------------------------------------------------------------+--------+
  osba   http://osba-open-service-broker-azure.osba.svc.cluster.local   Ready
```

接下來，列出可用的服務類別。 所顯示的服務類別就是可用的 Azure 受控服務，您可以透過 Open Service Broker for Azure 佈建這些服務。

```azurecli-interactive
./svcat get classes
```

最後，列出所有可用的服務方案。 服務方案是 Azure 受控服務的服務層。 例如，適用於 MySQL 的 Azure 資料庫計劃的範圍是從包含 50 個資料庫交易單位 (DTU) 之基本層的 `basic50` 到包含 800 個 DTU 之標準層的 `standard800`。

```azurecli-interactive
./svcat get plans
```

## <a name="install-wordpress-from-helm-chart-using-azure-database-for-mysql"></a>使用適用於 MySQL 的 Azure 資料庫從 Helm 圖表安裝 WordPress

在此步驟中，您要使用 Helm 為 WordPress 安裝已更新的 Helm 圖表。 此圖表會佈建一個 WordPress 可以使用的外部適用於 MySQL 的 Azure 資料庫執行個體。 此程序需要數分鐘的時間。

```azurecli-interactive
helm install azure/wordpress --name wordpress --namespace wordpress --set resources.requests.cpu=0
```

若要確認安裝已佈建正確的資源，請列出已安裝的服務執行個體和繫結：

```azurecli-interactive
./svcat get instances -n wordpress
./svcat get bindings -n wordpress
```

列出已安裝的密碼：

```azurecli-interactive
kubectl get secrets -n wordpress -o yaml
```

## <a name="next-steps"></a>後續步驟

依照本文件，將 Service Catalog 部署至 Azure Kubernetes Service (AKS) 叢集。 您要使用 Open Service Broker for Azure，部署使用 Azure 受控服務的 WordPress 安裝，在此案例中為適用於 MySQL 的 Azure 資料庫。

若要存取其他更新的 OSBA 型 Helm 圖表，請參閱 [Azure/helm-charts][helm-charts] 存放庫。 如果您要建立使用 OSBA 的圖表，請參閱[建立新的圖表][helm-create-new-chart]。

<!-- LINKS - external -->
[helm-charts]: https://github.com/Azure/helm-charts
[helm-cli-install]: https://docs.helm.sh/helm/#helm-install
[helm-create-new-chart]: https://github.com/Azure/helm-charts#creating-a-new-chart
[kubernetes-service-catalog]: https://github.com/kubernetes-incubator/service-catalog
[open-service-broker-azure]: https://github.com/Azure/open-service-broker-azure
[service-catalog-cli]: https://github.com/Azure/service-catalog-cli

<!-- LINKS - internal -->
[azure-cli-install]: /cli/azure/install-azure-cli
[azure-cloud-shell]: ../cloud-shell/overview.md
[create-aks-cluster]: ./kubernetes-walkthrough.md
[create-service-principal]: ./kubernetes-service-principal.md
