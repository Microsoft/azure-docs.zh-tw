---
title: 使用 CLI 和 YAML 在 Azure 容器執行個體中部署多個容器群組
description: 了解如何使用 Azure CLI 和 YAML 檔案在 Azure 容器執行個體中部署具有多個容器的容器群組。
services: container-instances
author: dlepow
ms.service: container-instances
ms.topic: article
ms.date: 07/17/2018
ms.author: danlep
ms.openlocfilehash: ffc9cf24e686924878a752b5d9df31160328ef0a
ms.sourcegitcommit: 67abaa44871ab98770b22b29d899ff2f396bdae3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/08/2018
ms.locfileid: "48854706"
---
# <a name="deploy-a-multi-container-container-group-with-yaml"></a>使用 YAML 部署多容器的容器群組

Azure 容器執行個體支援使用[容器群組](container-instances-container-groups.md)將多個容器部署至單一主機。 在建置應用程式 Sidecar 以便記錄、監視或進行服務需要第二個附加程序的任何其他設定時，多容器的容器群組非常有用。

使用 Azure CLI 來部署多容器群組有兩種方法：

* YAML 檔案部署 (本文)
* [Resource Manager 範本部署](container-instances-multi-container-group.md)

由於 YAML 格式的本質較簡潔，當您的部署「只」包含容器執行個體時，建議您使用 YAML 檔案進行部署。 如果您在部署容器執行個體時，需要部署其他 Azure 服務資源 (例如 Azure 檔案服務共用)，建議您使用 Resource Manager 範本部署。

> [!NOTE]
> 多容器群組目前僅限於 Linux 容器。 在我們致力於將所有功能帶入 Windows 容器的同時，您可以在 [Azure 容器執行個體配額和區域可用性](container-instances-quotas.md)中找到目前的平台差異。

## <a name="configure-the-yaml-file"></a>設定 YAML 檔案

若要在 Azure CLI 中使用 [az container create][az-container-create] 命令部署多容器的容器群組，您必須在 YAML 檔案中指定容器群組組態，然後以參數的形式將 YAML 檔案傳遞到命令中。

一開始先將下列 YAML 複製到名為 **deploy-aci.yaml** 的新檔案中。

此 YAML 檔案會定義名為 "myContainerGroup"，且含有兩個容器 (一個公用 IP 位址和兩個公開連接埠) 的容器群組。 群組中第一個容器會執行網際網路對向的 Web 應用程式。 第二個容器 sidecar 會透過容器群組的區域網路，定期對第一個容器中執行的 Web 應用程式執行 HTTP 要求。

```YAML
apiVersion: 2018-06-01
location: eastus
name: myContainerGroup
properties:
  containers:
  - name: aci-tutorial-app
    properties:
      image: microsoft/aci-helloworld:latest
      resources:
        requests:
          cpu: 1
          memoryInGb: 1.5
      ports:
      - port: 80
      - port: 8080
  - name: aci-tutorial-sidecar
    properties:
      image: microsoft/aci-tutorial-sidecar
      resources:
        requests:
          cpu: 1
          memoryInGb: 1.5
  osType: Linux
  ipAddress:
    type: Public
    ports:
    - protocol: tcp
      port: '80'
    - protocol: tcp
      port: '8080'
tags: null
type: Microsoft.ContainerInstance/containerGroups
```

## <a name="deploy-the-container-group"></a>部署容器群組

使用 [az group create][az-group-create] 命令來建立資源群組：

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

使用 [az container create][az-container-create] 命令部署容器群組，並以引數的形式傳遞 YAML 檔案：

```azurecli-interactive
az container create --resource-group myResourceGroup --file deploy-aci.yaml
```

在幾秒內，您應該會從 Azure 收到首次回應。

## <a name="view-deployment-state"></a>檢視部署狀態

若要檢視部署的狀態，請使用下列 [az container show][az-container-show] 命令：

```azurecli-interactive
az container show --resource-group myResourceGroup --name myContainerGroup --output table
```

若要檢視執行中的應用程式，請在瀏覽器中瀏覽至其公用 IP 位址。 例如，在此範例輸出中，IP 是 `52.168.26.124`：

```bash
Name              ResourceGroup    ProvisioningState    Image                                                           IP:ports               CPU/Memory       OsType    Location
----------------  ---------------  -------------------  --------------------------------------------------------------  ---------------------  ---------------  --------  ----------
myContainerGroup  myResourceGroup  Succeeded            microsoft/aci-helloworld:latest,microsoft/aci-tutorial-sidecar  52.168.26.124:80,8080  1.0 core/1.5 gb  Linux     eastus
```

## <a name="view-logs"></a>檢視記錄檔

使用 [az container logs][az-container-logs] 命令，檢視容器的記錄輸出。 `--container-name` 引數會指定要從中提取記錄的容器。 此範例所指定的會是第一個容器。

```azurecli-interactive
az container logs --resource-group myResourceGroup --name myContainerGroup --container-name aci-tutorial-app
```

輸出：

```console
listening on port 80
::1 - - [09/Jan/2018:23:17:48 +0000] "HEAD / HTTP/1.1" 200 1663 "-" "curl/7.54.0"
::1 - - [09/Jan/2018:23:17:51 +0000] "HEAD / HTTP/1.1" 200 1663 "-" "curl/7.54.0"
::1 - - [09/Jan/2018:23:17:54 +0000] "HEAD / HTTP/1.1" 200 1663 "-" "curl/7.54.0"
```

若要查看 Sidecar 容器的記錄，請執行相同命令來指定第二個容器名稱。

```azurecli-interactive
az container logs --resource-group myResourceGroup --name myContainerGroup --container-name aci-tutorial-sidecar
```

輸出：

```console
Every 3s: curl -I http://localhost                          2018-01-09 23:25:11

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0  1663    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
HTTP/1.1 200 OK
X-Powered-By: Express
Accept-Ranges: bytes
Cache-Control: public, max-age=0
Last-Modified: Wed, 29 Nov 2017 06:40:40 GMT
ETag: W/"67f-16006818640"
Content-Type: text/html; charset=UTF-8
Content-Length: 1663
Date: Tue, 09 Jan 2018 23:25:11 GMT
Connection: keep-alive
```

如您所見，Sidecar 會透過群組的區域網路定期地向主要 Web 應用程式提出 HTTP 要求，以確保它正在執行。 您可以擴充此 Sidecar 範例，以在其收到 200 OK 以外的 HTTP 回應碼時觸發警示。

## <a name="deploy-from-private-registry"></a>從私人登錄部署

若要使用私人容器映像登錄，請加入下列 YAML 和針對您環境修改的值：

```YAML
  imageRegistryCredentials:
  - server: imageRegistryLoginServer
    username: imageRegistryUsername
    password: imageRegistryPassword
```

例如，下列 YAML 部署的容器群組包含了單一容器，其映像就是取自名為"myregistry" 的私人 Azure 容器登錄：

```YAML
apiVersion: 2018-06-01
location: eastus
name: myContainerGroup2
properties:
  containers:
  - name: aci-tutorial-app
    properties:
      image: myregistry.azurecr.io/aci-helloworld:latest
      resources:
        requests:
          cpu: 1
          memoryInGb: 1.5
      ports:
      - port: 80
  osType: Linux
  ipAddress:
    type: Public
    ports:
    - protocol: tcp
      port: '80'
  imageRegistryCredentials:
  - server: myregistry.azurecr.io
    username: myregistry
    password: REGISTRY_PASSWORD
tags: null
type: Microsoft.ContainerInstance/containerGroups
```

## <a name="export-container-group-to-yaml"></a>將容器群組匯出至 YAML

您可以使用 Azure CLI 的 [az container export][az-container-export] 命令，將現有容器群組的組態匯出至 YAML 檔案。

為方便保留容器群組的組態，匯出可讓您將容器群組組態儲存在「組態即代碼 (configuration as code)」的版本控制中。 或者，在 YAML 中開發新組態時，您可以使用匯出的檔案作為起點。

發出下列 [az container export][az-container-export] 命令，即可匯出稍早建立的容器群群組組態：

```azurecli-interactive
az container export --resource-group myResourceGroup --name myContainerGroup --file deployed-aci.yaml
```

命令執行成功時不會顯示輸出，但您可以檢視檔案的內容以查看結果。 例如，`head` 後的前幾行：

```console
$ head deployed-aci.yaml
additional_properties: {}
apiVersion: '2018-06-01'
location: eastus
name: myContainerGroup
properties:
  containers:
  - name: aci-tutorial-app
    properties:
      environmentVariables: []
      image: microsoft/aci-helloworld:latest
```

## <a name="next-steps"></a>後續步驟

本文涵蓋部署多容器 Azure 容器執行個體所需的步驟。 如需端對端的 Azure 容器執行個體體驗 (包括使用私人 Azure 容器登錄)，請參閱 Azure 容器執行個體教學課程。

> [!div class="nextstepaction"]
> [Azure 容器執行個體教學課程][aci-tutorial]

<!-- LINKS - External -->
[cli-issue-6525]: https://github.com/Azure/azure-cli/issues/6525

<!-- LINKS - Internal -->
[aci-tutorial]: ./container-instances-tutorial-prepare-app.md
[az-container-create]: /cli/azure/container#az-container-create
[az-container-export]: /cli/azure/container#az-container-export
[az-container-logs]: /cli/azure/container#az-container-logs
[az-container-show]: /cli/azure/container#az-container-show
[az-group-create]: /cli/azure/group#az-group-create
[az-group-deployment-create]: /cli/azure/group/deployment#az-group-deployment-create
[template-reference]: https://docs.microsoft.com/azure/templates/microsoft.containerinstance/containergroups
