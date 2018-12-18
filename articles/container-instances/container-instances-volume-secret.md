---
title: 在 Azure 容器執行個體中掛接秘密磁碟區
description: 了解如何掛接秘密磁碟區，以儲存供您的容器執行個體存取的機密資訊
services: container-instances
author: mmacy
manager: jeconnoc
ms.service: container-instances
ms.topic: article
ms.date: 07/19/2018
ms.author: marsma
ms.openlocfilehash: 572e6701bbe69bbb07c76d468a309030fc37d984
ms.sourcegitcommit: 1478591671a0d5f73e75aa3fb1143e59f4b04e6a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/19/2018
ms.locfileid: "39159884"
---
# <a name="mount-a-secret-volume-in-azure-container-instances"></a>在 Azure 容器執行個體中掛接秘密磁碟區

使用「秘密」磁碟區，將敏感性資訊提供給容器群組中的容器。 「秘密」磁碟區會將您的祕密儲存在磁碟區內的檔案中，容器群組中的容器即可存取這些檔案。 藉由在「秘密」磁碟區中儲存密碼，您可以避免將 SSH 金鑰或資料庫認證等敏感性資料新增至您的應用程式程式碼。

所有秘密磁碟區都由 RAM 型檔案系統 [tmpfs][tmpfs] 支援；其內容永遠不會寫入靜態儲存區。

> [!NOTE]
> 「祕密」磁碟需目前僅限於 Linux 容器。 了解如何在[設定環境變數](container-instances-environment-variables.md)中，為 Windows 和 Linux 容器傳遞安全的環境變數。 在我們致力於將所有功能帶入 Windows 容器的同時，您可以在 [Azure 容器執行個體配額和區域可用性](container-instances-quotas.md)中找到目前的平台差異。

## <a name="mount-secret-volume---azure-cli"></a>掛接秘密磁碟區 - Azure CLI

若要使用 Azure CLI 部署具有一或多個祕密的容器，請在 [az container create][az-container-create] 命令中包和 `--secrets` 和 `--secrets-mount-path` 參數。 此範例會在 `/mnt/secrets` 掛接由兩個祕密 ("mysecret1" 和 "mysecret2") 所組成的「秘密」磁碟區：

```azurecli-interactive
az container create \
    --resource-group myResourceGroup \
    --name secret-volume-demo \
    --image microsoft/aci-helloworld \
    --secrets mysecret1="My first secret FOO" mysecret2="My second secret BAR" \
    --secrets-mount-path /mnt/secrets
```

下列 [az container exec][az-container-exec] 輸出顯示如何在執行中的容器中開啟殼層、列出祕密磁碟區內的檔案，然後顯示其內容：

```console
$ az container exec --resource-group myResourceGroup --name secret-volume-demo --exec-command "/bin/sh"
/usr/src/app # ls -1 /mnt/secrets
mysecret1
mysecret2
/usr/src/app # cat /mnt/secrets/mysecret1
My first secret FOO
/usr/src/app # cat /mnt/secrets/mysecret2
My second secret BAR
/usr/src/app # exit
Bye.
```

## <a name="mount-secret-volume---yaml"></a>掛接秘密磁碟區 - YAML

您也可以使用 Azure CLI 和 [YAML 範本](container-instances-multi-container-yaml.md)部署容器群組。 部署由多個容器所組成的容器群組時，偏好經由 YAML 範本進行部署。

當您透過 YAML 範本進行部署時，範本中的祕密值必須為 **Base64 編碼**。 不過，祕密值會出現於容器中檔案內的純文字。

下列 YAML 範本可定義含有一個容器的容器群組，而該容器會在 `/mnt/secrets` 掛接「祕密」磁碟區。 此祕密磁碟區有兩個祕密 "mysecret1" 和 "mysecret2"。

```yaml
apiVersion: '2018-06-01'
location: eastus
name: secret-volume-demo
properties:
  containers:
  - name: aci-tutorial-app
    properties:
      environmentVariables: []
      image: microsoft/aci-helloworld:latest
      ports: []
      resources:
        requests:
          cpu: 1.0
          memoryInGB: 1.5
      volumeMounts:
      - mountPath: /mnt/secrets
        name: secretvolume1
  osType: Linux
  restartPolicy: Always
  volumes:
  - name: secretvolume1
    secret:
      mysecret1: TXkgZmlyc3Qgc2VjcmV0IEZPTwo=
      mysecret2: TXkgc2Vjb25kIHNlY3JldCBCQVIK
tags: {}
type: Microsoft.ContainerInstance/containerGroups
```

若要透過 YAML 範本進行部署，請將上述 YAML 儲存到名為 `deploy-aci.yaml` 的檔案，然後使用 `--file` 參數執行 [az container create][az-container-create] 命令：

```azurecli-interactive
# Deploy with YAML template
az container create --resource-group myResourceGroup --file deploy-aci.yaml
```

## <a name="mount-secret-volume---resource-manager"></a>掛接秘密磁碟區 - Resource Manager

除了 CLI 和 YAML 部署，您可以使用 Azure [Resource Manager 範例](/azure/templates/microsoft.containerinstance/containergroups)來部署容器群組。

首先，填入範本的容器群組 `properties` 區段中的 `volumes` 陣列。 當您透過 Resource Manager 範本進行部署時，範本中的祕密值必須為 **Base64 編碼**。 不過，祕密值會出現於容器中檔案內的純文字。

接下來，針對您想要掛接祕密磁碟區所在容器群組中的每個容器，填入容器定義之 `properties` 區段中的 `volumeMounts` 陣列。

下列 Resource Manager 範本可定義含有一個容器的容器群組，而該容器會在 `/mnt/secrets` 掛接「祕密」磁碟區。 此祕密磁碟區有兩個祕密 "mysecret1" 和 "mysecret2"。

<!-- https://github.com/Azure/azure-docs-json-samples/blob/master/container-instances/aci-deploy-volume-secret.json --> [!code-json[volume-secret](~/azure-docs-json-samples/container-instances/aci-deploy-volume-secret.json)]

若要透過 Resource Manager 範本進行部署，請將上述 JSON 儲存到名為 `deploy-aci.json` 的檔案，然後使用 `--template-file` 參數執行 [az group deployment create][az-group-deployment-create] 命令：

```azurecli-interactive
# Deploy with Resource Manager template
az group deployment create --resource-group myResourceGroup --template-file deploy-aci.json
```

## <a name="next-steps"></a>後續步驟

### <a name="volumes"></a>磁碟區

了解如何在 Azure 容器執行個體中掛接其他類型的磁碟區：

* [在 Azure 容器執行個體中掛接 Azure 檔案共用](container-instances-volume-azure-files.md)
* [在 Azure 容器執行個體中掛接 emptyDir 磁碟區](container-instances-volume-emptydir.md)
* [在 Azure 容器執行個體中掛接 gitRepo 磁碟區](container-instances-volume-gitrepo.md)

### <a name="secure-environment-variables"></a>安全的環境變數

將敏感性資訊提供給容器 (包括 Windows 容器) 的另一種方法是使用[安全的環境變數](container-instances-environment-variables.md#secure-values)。

<!-- LINKS - External -->
[tmpfs]: https://wikipedia.org/wiki/Tmpfs

<!-- LINKS - Internal -->
[az-container-create]: /cli/azure/container#az-container-create
[az-container-exec]: /cli/azure/container#az-container-exec
[az-group-deployment-create]: /cli/azure/group/deployment#az-group-deployment-create
