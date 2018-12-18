---
title: 使用 Draft 搭配 Azure Container Service 與 Azure Container Registry
description: 建立 ACS Kubernetes 叢集和 Azure Container Registry，可使用 Draft 在 Azure 中建立第一個應用程式。
services: container-service
author: squillace
manager: jeconnoc
ms.service: container-service
ms.topic: article
ms.date: 09/14/2017
ms.author: rasquill
ms.custom: mvc
ms.openlocfilehash: c635a869506918ab7ee032df349eb307987c1284
ms.sourcegitcommit: 1d850f6cae47261eacdb7604a9f17edc6626ae4b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39432274"
---
# <a name="use-draft-with-azure-container-service-and-azure-container-registry-to-build-and-deploy-an-application-to-kubernetes"></a>使用 Draft 搭配 Azure Container Service 與 Azure Container Registry，可將應用程式建置及部署至 Kubernetes

[!INCLUDE [aks-preview-redirect.md](../../../includes/aks-preview-redirect.md)]

[Draft](https://aka.ms/draft) 是新的開放原始碼工具，可讓您輕鬆地開發以容器作為基礎的應用程式，並將其部署至 Kubernetes 叢集，而無需深入了解 Docker 和 Kubernetes，或甚至進行安裝。 使用諸如 Draft 等工具可讓您和小組專注於使用 Kubernetes 來建置應用程式，無須投入過多注意力在基礎結構。

您可以使用 Draft 搭配任何 Docker 映像登錄與任何 Kubernetes 叢集，包括本機。 本教學課程會說明如何使用 ACS 搭配 Kubernetes 和 ACR，在 Kubernetes 中使用 Draft 建立即時但安全的開發人員管線，並說明如何使用 Azure DNS 在網域公開該開發人員管線以供其他人查看。


## <a name="create-an-azure-container-registry"></a>建立 Azure Container Registry
您可以輕鬆地[建立新的 Azure Container Registry](../../container-registry/container-registry-get-started-azure-cli.md)，步驟如下所示：

1. 建立 Azure 資源群組可在 ACS 中管理您的 ACR 登錄和 Kubernetes 叢集。
      ```azurecli
      az group create --name draft --location eastus
      ```

2. 使用 [az acr create](/cli/azure/acr#az-acr-create) 建立 ACR 映像登錄，並確定 `--admin-enabled` 選項已設定為 `true`。
      ```azurecli
      az acr create --resource-group draft --name draftacs --sku Basic
      ```


## <a name="create-an-azure-container-service-with-kubernetes"></a>使用 Kubernetes 建立 Azure Container Service

現在您準備好使用 [az acs create](/cli/azure/acs#az-acs-create)，利用 Kubernetes 作為 `--orchestrator-type` 值來建立 ACS 叢集。
```azurecli
az acs create --resource-group draft --name draft-kube-acs --dns-prefix draft-cluster --orchestrator-type kubernetes --generate-ssh-keys
```

> [!NOTE]
> 因為 Kubernetes 不是預設的 Orchestrator 類型，請確定您使用 `--orchestrator-type kubernetes` 參數。

成功時的輸出大致如下所示。

```json
waiting for AAD role to propagate.done
{
  "id": "/subscriptions/<guid>/resourceGroups/draft/providers/Microsoft.Resources/deployments/azurecli14904.93snip09",
  "name": "azurecli1496227204.9323909",
  "properties": {
    "correlationId": "<guid>",
    "debugSetting": null,
    "dependencies": [],
    "mode": "Incremental",
    "outputs": null,
    "parameters": {
      "clientSecret": {
        "type": "SecureString"
      }
    },
    "parametersLink": null,
    "providers": [
      {
        "id": null,
        "namespace": "Microsoft.ContainerService",
        "registrationState": null,
        "resourceTypes": [
          {
            "aliases": null,
            "apiVersions": null,
            "locations": [
              "westus"
            ],
            "properties": null,
            "resourceType": "containerServices"
          }
        ]
      }
    ],
    "provisioningState": "Succeeded",
    "template": null,
    "templateLink": null,
    "timestamp": "2017-05-31T10:46:29.434095+00:00"
  },
  "resourceGroup": "draft"
}
```

現在，您有一個叢集，可以使用 [az acs kubernetes get-credentials](/cli/azure/acs/kubernetes#get-credentials) 命令將認證匯入。 現在您有叢集的本機組態檔，這是 Helm 和 Draft 完成其工作所需要的項目。

## <a name="install-and-configure-draft"></a>安裝及設定草稿


1. 在 https://github.com/Azure/draft/releases \(英文\) 下載適用於您環境的 Draft ，並將它安裝到 PATH 中以使用該命令。
2. 在 https://github.com/kubernetes/helm/releases \(英文\) 下載適用於您環境的 Helm，並將它[安裝到 PATH 中以使用該命令](https://github.com/kubernetes/helm/blob/master/docs/install.md#installing-the-helm-client) \(英文\)。
3. 設定 Draft 以使用您的登錄，並針對它所建立的每個 Helm 圖表建立子網域。 若要設定 Draft，您需要：
  - 您的 Azure Container Registry 名稱 (在此範例中為 `draftacsdemo`)
  - 您的登錄機碼或密碼，從 `az acr credential show -n <registry name> --output tsv --query "passwords[0].value"`。

  呼叫 `draft init`，設定程序會提示您輸入上述的值。請注意，登錄 URL 的 URL 格式是登錄名稱 (在本範例中為 `draftacsdemo`) 加上 `.azurecr.io`。 您的使用者名稱是登錄名稱本身。 第一次執行此程序時，它看起來如下所示。
 ```bash
    $ draft init
    Creating /home/ralph/.draft 
    Creating /home/ralph/.draft/plugins 
    Creating /home/ralph/.draft/packs 
    Creating pack go...
    Creating pack python...
    Creating pack ruby...
    Creating pack javascript...
    Creating pack gradle...
    Creating pack java...
    Creating pack php...
    Creating pack csharp...
    $DRAFT_HOME has been configured at /home/ralph/.draft.

    In order to configure Draft, we need a bit more information...

    1. Enter your Docker registry URL (e.g. docker.io/myuser, quay.io/myuser, myregistry.azurecr.io): draftacsdemo.azurecr.io
    2. Enter your username: draftacsdemo
    3. Enter your password: 
    Draft has been installed into your Kubernetes Cluster.
    Happy Sailing!
```

您現在已準備好要部署應用程式。


## <a name="build-and-deploy-an-application"></a>建置和部署應用程式

在 Draft 存放庫中，有[六個簡單的範例應用程式](https://github.com/Azure/draft/tree/master/examples)。 複製存放庫，讓我們使用 [Java 範例](https://github.com/Azure/draft/tree/master/examples/java) \(英文\)。 變更到 examples/java 目錄，並輸入 `draft create` 來建置應用程式。 它看起來會如下範例所示。
```bash
$ draft create
--> Draft detected the primary language as Java with 91.228814% certainty.
--> Ready to sail
```

輸出包含 Dockerfile 和 Helm 圖表。 若要建置和部署，您只要輸入 `draft up`。 輸出量會很大，但看起來應該會如下列範例。
```bash
$ draft up
Draft Up Started: 'handy-labradoodle'
handy-labradoodle: Building Docker Image: SUCCESS ⚓  (35.0232s)
handy-labradoodle: Pushing Docker Image: SUCCESS ⚓  (17.0062s)
handy-labradoodle: Releasing Application: SUCCESS ⚓  (3.8903s)
handy-labradoodle: Build ID: 01BT0ZJ87NWCD7BBPK4Y3BTTPB
```

## <a name="securely-view-your-application"></a>安全地檢視應用程式

您的容器現已在 ACS 中執行。 若要檢視它，請使用 `draft connect` 命令，這會和目標叢集 IP 建立安全連線，並針對您的應用程式提供特定的連接埠，使您能夠於本機檢視它。 如果成功，請在第一行的 **SUCCESS** 指標之後尋找用來連線至應用程式的 URL。

> [!NOTE]
> 如果您收到訊息指出沒有任何 Pod 準備就緒，請等候一段時間再重試，或您可以使用 `kubectl get pods -w` 來查看 Pod 的就緒進度，並於它們準備好後重試。

```bash
draft connect
Connecting to your app...SUCCESS...Connect to your app on localhost:46143
Starting log streaming...
SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
SLF4J: Defaulting to no-operation (NOP) logger implementation
SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for further details.
== Spark has ignited ...
>> Listening on 0.0.0.0:4567
```

在上述範例中，您可以輸入 `curl -s http://localhost:46143` 來接收回覆 `Hello World, I'm Java!`。 當您按 CTRL+C 或 CMD+C (取決於您的作業系統環境) 時，安全通道就會終止，您可以繼續反覆執行。

## <a name="sharing-your-application-by-configuring-a-deployment-domain-with-azure-dns"></a>使用 Azure DNS 設定部署網域來共用應用程式

您已經執行 Draft 在上述步驟中建立的開發人員反覆運算迴圈。 不過，您可以透過下列方法於網際網路上共用應用程式：
1. 在您的 ACS 叢集中安裝輸入 (以提供要顯示應用程式的公用 IP 位址)
2. 將您的自訂網域委派給 Azure DNS，並將您的網域對應到 ACS 指派給您輸入控制器的 IP 位址

### <a name="use-helm-to-install-the-ingress-controller"></a>使用 helm 安裝輸入控制器。
使用 **helm** 來搜尋並安裝 `stable/traefik` 輸入控制器，以針對您的組建啟用輸入要求。
```bash
$ helm search traefik
NAME            VERSION DESCRIPTION
stable/traefik  1.3.0   A Traefik based Kubernetes ingress controller w...

$ helm install stable/traefik --name ingress
```
現在，請在 `ingress` 控制站上設定監看，以在部署外部 IP 值時加以擷取。 此 IP 位址會是下一節中[對應到您部署網域](#wire-up-deployment-domain)的 IP 位址。

```bash
$ kubectl get svc -w
NAME                          CLUSTER-IP     EXTERNAL-IP     PORT(S)                      AGE
ingress-traefik               10.0.248.104   13.64.108.240   80:31046/TCP,443:32556/TCP   1h
kubernetes                    10.0.0.1       <none>          443/TCP                      7h
```

在此案例中，部署網域的外部 IP 是 `13.64.108.240`。 現在您可以將網域對應至該 IP。

### <a name="map-the-ingress-ip-to-a-custom-subdomain"></a>將輸入 IP 對應至自訂子網域

Draft 會針對其所建立的每個 Helm 圖表，以及您在使用每個應用程式建立一個版本。 每個版本都會取得一個供 **draft** 使用的產生名稱，作為您所控制之根「部署網域」上的「子網域」。 (在此範例中，我們使用 `squillace.io` 作為部署網域。)若要啟用此子網域行為，您必須針對部署網域，在 DNS 項目中建立 `'*.draft'` 的 A 記錄，以便每個產生的子網域會路由傳送至 Kubernetes 叢集的輸入控制器。 

您自己的網域提供者都有其各自的方法可指派 DNS 伺服器；若要[將您的 nameservers 網域委派給 Azure DNS](../../dns/dns-delegate-domain-azure-dns.md)，請採取下列步驟：

1. 建立您區域的資源群組。
    ```azurecli
    az group create --name squillace.io --location eastus
    {
      "id": "/subscriptions/<guid>/resourceGroups/squillace.io",
      "location": "eastus",
      "managedBy": null,
      "name": "zones",
      "properties": {
        "provisioningState": "Succeeded"
      },
      "tags": null
    }
    ```

2. 建立您網域的 DNS 區域。
使用 [az network dns zone create](/cli/azure/network/dns/zone#az-network-dns-zone-create) 命令來取得 nameservers，將 DNS 控制項委派給網域的 Azure DNS。
    ```azurecli
    az network dns zone create --resource-group squillace.io --name squillace.io
    {
      "etag": "<guid>",
      "id": "/subscriptions/<guid>/resourceGroups/zones/providers/Microsoft.Network/dnszones/squillace.io",
      "location": "global",
      "maxNumberOfRecordSets": 5000,
      "name": "squillace.io",
      "nameServers": [
        "ns1-09.azure-dns.com.",
        "ns2-09.azure-dns.net.",
        "ns3-09.azure-dns.org.",
        "ns4-09.azure-dns.info."
      ],
      "numberOfRecordSets": 2,
      "resourceGroup": "squillace.io",
      "tags": {},
      "type": "Microsoft.Network/dnszones"
    }
    ```
3. 將您所取得的 DNS 伺服器新增至您部署網域的網域提供者，可讓您視需要使用 Azure DNS 重新指向您的網域。 執行這項作業的方式會隨網域提供者而有所不同，[將您的網域 NameServer 委派給 Azure DNS](../../dns/dns-delegate-domain-azure-dns.md) 中包含一些您應該知道的詳細資料。 
4. 一旦您將網域委派給 Azure DNS 之後，請建立針對您部署網域的 A 記錄集項目，對應至來自上一節步驟 2 中的 `ingress` IP。
  ```azurecli
  az network dns record-set a add-record --ipv4-address 13.64.108.240 --record-set-name '*.draft' -g squillace.io -z squillace.io
  ```
輸出看起來會類似於：
  ```json
  {
    "arecords": [
      {
        "ipv4Address": "13.64.108.240"
      }
    ],
    "etag": "<guid>",
    "id": "/subscriptions/<guid>/resourceGroups/squillace.io/providers/Microsoft.Network/dnszones/squillace.io/A/*",
    "metadata": null,
    "name": "*.draft",
    "resourceGroup": "squillace.io",
    "ttl": 3600,
    "type": "Microsoft.Network/dnszones/A"
  }
  ```
5. 重新安裝 **draft**

   1. 輸入 `helm delete --purge draft` 以從叢集中移除 **draft**。 
   2. 使用相同的 `draft-init` 命令 (但加上 `--ingress-enabled` 選項) 來重新安裝 **draft**：
    ```bash
    draft init --ingress-enabled
    ```
   以您先前於上述步驟中的回應方式來回應提示。 不過，您有額外一個問題需要回應，這必須使用您利用 Azure DNS 設定的完整網域路徑。

6. 輸入您用於輸入的最上層網域 (例如 draft.example.com)：draft.squillace.io
7. 當您這次呼叫 `draft up` 時，您將能在以 `<appname>.draft.<domain>.<top-level-domain>` 作為格式的 URL 上看到您的應用程式 (或對它進行 `curl`)。 在此範例中，URL 將會是 `http://handy-labradoodle.draft.squillace.io`。 
```bash
curl -s http://handy-labradoodle.draft.squillace.io
Hello World, I'm Java!
```


## <a name="next-steps"></a>後續步驟

您有了 ACS Kubernetes 叢集之後，可以使用 [Azure Container Registry](../../container-registry/container-registry-intro.md) 進行調查，建立更多這種案例與不同的部署。 例如，您可以建立 draft._basedomain.toplevel_ 網域 DNS 記錄集，可針對特定 ACS 部署，控制項目移出更深入的子網域。






