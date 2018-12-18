---
title: 在 Azure 容器執行個體中掛接 gitRepo 磁碟區
description: 了解如何掛接 gitRepo 磁碟區，以將 Git 存放庫複製到您的容器執行個體中
services: container-instances
author: mmacy
manager: jeconnoc
ms.service: container-instances
ms.topic: article
ms.date: 06/15/2018
ms.author: marsma
ms.openlocfilehash: 34036c5ec9ccd8c502104ce862e4749c59be62b9
ms.sourcegitcommit: f6e2a03076679d53b550a24828141c4fb978dcf9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43105194"
---
# <a name="mount-a-gitrepo-volume-in-azure-container-instances"></a>在 Azure 容器執行個體中掛接 gitRepo 磁碟區

了解如何掛接 *gitRepo* 磁碟區，以將 Git 存放庫複製到您的容器執行個體中。

> [!NOTE]
> 目前只有 Linux 容器才能掛接 *gitRepo* 磁碟區。 雖然我們致力於將所有功能帶入 Windows 容器，但是您可以在 [Azure 容器執行個體配額和區域可用性](container-instances-quotas.md)中找到目前的平台差異。

## <a name="gitrepo-volume"></a>gitRepo 磁碟區

*gitRepo* 磁碟區會掛接目錄，並在容器啟動時將指定的 Git 存放庫複製到該目錄中。 透過在您的容器執行個體中使用 *gitRepo* 磁碟區，您可以避免在您的應用程式中加入程式碼。

當您掛接 *gitRepo* 磁碟區時，您可以設定三個屬性來設定磁碟區：

| 屬性 | 必要 | 說明 |
| -------- | -------- | ----------- |
| `repository` | 是 | 要複製之 Git 存放庫的完整 URL，包括 `http://` 或 `https://`。|
| `directory` | 否 | 要在其中複製存放庫的目錄。 路徑不能包含或開頭為 "`..`"。  如果您指定 "`.`"，存放庫會複製到磁碟區的目錄中。 否則，Git 存放庫會複製到磁碟區目錄內指定名稱的子目錄中。 |
| `revision` | 否 | 要複製之修訂的認可雜湊。 如果未指定，則 `HEAD` 修訂已複製。 |

## <a name="mount-gitrepo-volume-azure-cli"></a>掛接 gitRepo 磁碟區：Azure CLI

當您使用 [Azure CLI](/cli/azure) 部署容器執行個體時，若要掛接 gitRepo 磁碟區，請將 `--gitrepo-url` 和 `--gitrepo-mount-path` 參數提供給 [az container create][az-container-create] 命令。 您可以選擇性地指定磁碟區內要複製存放庫到其中的目錄 (`--gitrepo-dir`)，以及要複製的修訂認可雜湊 (`--gitrepo-revision`)。

此範例命令會將 [aci-helloworld][aci-helloworld] 範例應用程式複製到容器執行個體中的 `/mnt/aci-helloworld`：

```azurecli-interactive
az container create \
    --resource-group myResourceGroup \
    --name hellogitrepo \
    --image microsoft/aci-helloworld \
    --dns-name-label aci-demo \
    --ports 80 \
    --gitrepo-url https://github.com/Azure-Samples/aci-helloworld \
    --gitrepo-mount-path /mnt/aci-helloworld
```

若要確認已掛接 gitRepo 磁碟區，請使用 [az container exec][az-container-exec] 來啟動容器中的 Shell 並列出目錄：

```console
$ az container exec --resource-group myResourceGroup --name hellogitrepo --exec-command /bin/sh
/usr/src/app # ls -l /mnt/aci-helloworld/
total 16
-rw-r--r--    1 root     root           144 Apr 16 16:35 Dockerfile
-rw-r--r--    1 root     root          1162 Apr 16 16:35 LICENSE
-rw-r--r--    1 root     root          1237 Apr 16 16:35 README.md
drwxr-xr-x    2 root     root          4096 Apr 16 16:35 app
```

## <a name="mount-gitrepo-volume-resource-manager"></a>掛接 gitRepo 磁碟區：Resource Manager

當您使用 [Azure Resource Manager 範本](/azure/templates/microsoft.containerinstance/containergroups)部署容器執行個體時，若要掛接 gitRepo 磁碟區，請先在範本的容器群組 `properties` 區段中填入 `volumes` 陣列。 然後，針對您想要掛接 gitRepo 磁碟區所在容器群組中的每個容器，填入容器定義 `properties` 區段中的 `volumeMounts` 陣列。

例如，下列 Resource Manager 範本會建立一個由單一容器組成的容器群組。 容器會複製由 *gitRepo* 磁碟區區塊所指定的兩個 GitHub 存放庫。 第二個磁碟區包含指定要複製到其中之目錄的其他屬性，和要複製之指定修訂的認可雜湊。

<!-- https://github.com/Azure/azure-docs-json-samples/blob/master/container-instances/aci-deploy-volume-gitrepo.json --> [!code-json[volume-gitrepo](~/azure-docs-json-samples/container-instances/aci-deploy-volume-gitrepo.json)]

上述範本中定義的兩個已複製存放庫的結果目錄結構是：

```
/mnt/repo1/aci-helloworld
/mnt/repo2/my-custom-clone-directory
```

若要使用 Azure Resource Manager 範本來查看容器執行個體部署範例，請參閱[在 Azure 容器執行個體中部署多個容器群組](container-instances-multi-container-group.md)。

## <a name="private-git-repo-authentication"></a>私人 Git 存放庫驗證

若要為私人 Git 存放庫掛接 gitRepo 磁碟區，請在存放庫 URL 中指定認證。 通常，認證的形式是使用者名稱和個人存取權杖 (PAT)，可授與存放庫的有限範圍存取。

例如，私人 GitHub 存放庫的 Azure CLI `--gitrepo-url` 參數會類似下列項目 (其中 "gituser" 是 GitHub 使用者名稱，"abcdef1234fdsa4321abcdef" 是使用者的個人存取權杖)：

```azurecli
--gitrepo-url https://gituser:abcdef1234fdsa4321abcdef@github.com/GitUser/some-private-repository
```

針對 VSTS Git 存放庫，指定任何使用者名稱 (您可以如同下列範例一樣使用 "vstsuser") 與有效的 PAT 搭配使用：

```azurecli
--gitrepo-url https://vstsuser:abcdef1234fdsa4321abcdef@vstsaccountname.visualstudio.com/_git/some-private-repository
```

如需 GitHub 和 VSTS 的個人存取權杖詳細資訊，請參閱下列項目：

GitHub：[針對命令列建立個人存取權杖][pat-github]

VSTS：[建立個人存取權杖來驗證存取][pat-vsts]

## <a name="next-steps"></a>後續步驟

了解如何在 Azure 容器執行個體中掛接其他類型的磁碟區：

* [在 Azure 容器執行個體中掛接 Azure 檔案共用](container-instances-volume-azure-files.md)
* [在 Azure 容器執行個體中掛接 emptyDir 磁碟區](container-instances-volume-emptydir.md)
* [在 Azure 容器執行個體中掛接秘密磁碟區](container-instances-volume-secret.md)

<!-- LINKS - External -->
[aci-helloworld]: https://github.com/Azure-Samples/aci-helloworld
[pat-github]: https://help.github.com/articles/creating-a-personal-access-token-for-the-command-line/
[pat-vsts]: https://docs.microsoft.com/vsts/organizations/accounts/use-personal-access-tokens-to-authenticate

<!-- LINKS - Internal -->
[az-container-create]: /cli/azure/container#az-container-create
[az-container-exec]: /cli/azure/container#az-container-exec
