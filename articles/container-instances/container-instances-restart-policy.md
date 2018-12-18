---
title: 在 Azure Container Instances 中使用重新啟動原則執行容器化工作
description: 了解如何使用 Azure Container Instances 來執行工作，該工作會執行到完成為止，例如建置、測試或映像轉譯作業。
services: container-instances
author: dlepow
ms.service: container-instances
ms.topic: article
ms.date: 07/26/2018
ms.author: danlep
ms.openlocfilehash: c9e3fadd5164ca0d770f36ba95c30db933efcd39
ms.sourcegitcommit: 67abaa44871ab98770b22b29d899ff2f396bdae3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/08/2018
ms.locfileid: "48853879"
---
# <a name="run-containerized-tasks-with-restart-policies"></a>使用重新啟動原則執行容器化工作

在 Azure Container Instances 中部署容器的簡易和快速，提供令人信服的平台，可以用來進行執行一次的工作，例如在容器執行個體中建置、測試和映像轉譯。

使用可設定的重新啟動原則，您可以指定當容器的程序完成時，停止容器。 因為容器執行個體是由第二個開始計費，所以您只需支付當執行您的工作之容器執行時所使用的計算資源。

本文所呈現的範例是使用 Azure CLI。 您必須[在本機安裝][azure-cli-install] Azure CLI 版本 2.0.21 或更新版本，或者使用 [Azure Cloud Shell](../cloud-shell/overview.md) 中的 CLI。

## <a name="container-restart-policy"></a>容器重新啟動原則

當您在 Azure Container Instances 中建立容器時，您可以指定三個重新啟動原則設定的其中一個。

| 重新啟動原則   | 說明 |
| ---------------- | :---------- |
| `Always` | 容器群組中的容器一律會重新啟動。 這是在建立容器之際未指定重新啟動原則時，套用的**預設**設定。 |
| `Never` | 容器群組中的容器永不重新啟動。 容器最多執行一次。 |
| `OnFailure` | 容器群組中的容器只會在容器中的程序執行失敗時 (當它以非零結束代碼終止時) 重新啟動。 容器執行至少一次。 |

## <a name="specify-a-restart-policy"></a>指定重新啟動原則

您指定重新啟動原則的方式取決於如何建立容器執行個體，例如使用 Azure CLI、Azure PowerShell Cmdlet，或是 Azure 入口網站。 在 Azure CLI 中，當您呼叫 [az container create][az-container-create] 時指定 `--restart-policy` 參數。

```azurecli-interactive
az container create \
    --resource-group myResourceGroup \
    --name mycontainer \
    --image mycontainerimage \
    --restart-policy OnFailure
```

## <a name="run-to-completion-example"></a>執行至完成範例

若要查看作用中的重新啟動原則，從 [microsoft/aci-wordcount][aci-wordcount-image] 映像建立容器執行個體，然後指定 `OnFailure` 重新啟動原則。 此範例容器會執行 Python 指令碼，根據預設，它會分析 Shakespeare 的 [Hamlet](http://shakespeare.mit.edu/hamlet/full.html) 的文字，將 10 個最常見的字詞寫入 STDOUT，然後結束。

使用下列 [az container create][az-container-create] 命令執行範例容器：

```azurecli-interactive
az container create \
    --resource-group myResourceGroup \
    --name mycontainer \
    --image microsoft/aci-wordcount:latest \
    --restart-policy OnFailure
```

Azure Container Instances 會啟動容器，然後在它的應用程式 (或者是這個案例中的指令碼) 結束時停止它。 當 Azure Container Instances 停止其重新啟動原則為 `Never` 或 `OnFailure` 的容器時，容器的狀態會設定為「已終止」。 您可以使用 [az container show][az-container-show] 命令來檢查容器的狀態：

```azurecli-interactive
az container show --resource-group myResourceGroup --name mycontainer --query containers[0].instanceView.currentState.state
```

範例輸出︰

```bash
"Terminated"
```

一旦範例容器的狀態顯示「已終止」，您可以藉由檢視容器記錄來查看其工作輸出。 執行 [az container logs][az-container-logs] 命令以檢視指令碼的輸出：

```azurecli-interactive
az container logs --resource-group myResourceGroup --name mycontainer
```

輸出：

```bash
[('the', 990),
 ('and', 702),
 ('of', 628),
 ('to', 610),
 ('I', 544),
 ('you', 495),
 ('a', 453),
 ('my', 441),
 ('in', 399),
 ('HAMLET', 386)]
```

這個範例會顯示指令碼傳送到 STDOUT 的輸出。 您容器化工作，不過，可能會改為將其輸出寫入永續性儲存體以供日後擷取。 例如，寫入至 [Azure 檔案共用](container-instances-mounting-azure-files-volume.md)。

## <a name="configure-containers-at-runtime"></a>在執行階段設定容器

當您建立容器執行個體時，您可以設定其**環境變數**，以及指定自訂**命令列**在容器啟動時執行。 您可以在批次工作中使用這些設定，使用工作特定設定來準備每個容器。

## <a name="environment-variables"></a>環境變數

在您的容器中設定環境變數，以提供由容器執行之應用程式或指令碼的動態設定。 這類似於 `--env` 命令列引數 `docker run`。

例如，您可以藉由在建立容器執行個體時指定下列環境變數，來修改範例容器中指令碼的行為：

*NumWords*：傳送至 STDOUT 的字詞數。

*MinLength*：列入計算之字詞中的字元數上限。 較高的數目會略過常用字詞，例如 "of" 和 "the"。

```azurecli-interactive
az container create \
    --resource-group myResourceGroup \
    --name mycontainer2 \
    --image microsoft/aci-wordcount:latest \
    --restart-policy OnFailure \
    --environment-variables NumWords=5 MinLength=8
```

藉由為容器的環境變數指定 `NumWords=5` 和 `MinLength=8`，容器記錄應該會顯示不同的輸出。 一旦容器狀態顯示為「已終止」 (使用 `az container show` 檢查其狀態)，顯示其記錄以查看新的輸出：

```azurecli-interactive
az container logs --resource-group myResourceGroup --name mycontainer2
```

輸出：

```bash
[('CLAUDIUS', 120),
 ('POLONIUS', 113),
 ('GERTRUDE', 82),
 ('ROSENCRANTZ', 69),
 ('GUILDENSTERN', 54)]
```

## <a name="command-line-override"></a>命令列覆寫

當您建立容器執行個體以將所製作的命令列覆寫至容器映像時，請指定命令列。 這類似於 `--entrypoint` 命令列引數 `docker run`。

例如，您可以藉由指定不同的命令列，讓範例容器分析 Hamlet 以外的文字。 Python 指令碼是由容器 wordcount.py 執行，接受 URL 作為引數，而且會處理該分頁的內容，而不是預設值。

例如，若要判斷 Romeo and Juliet 中前三名的五個字母字詞：

```azurecli-interactive
az container create \
    --resource-group myResourceGroup \
    --name mycontainer3 \
    --image microsoft/aci-wordcount:latest \
    --restart-policy OnFailure \
    --environment-variables NumWords=3 MinLength=5 \
    --command-line "python wordcount.py http://shakespeare.mit.edu/romeo_juliet/full.html"
```

同樣地，一旦容器是「已終止」，藉由顯示容器的記錄來檢視輸出：

```azurecli-interactive
az container logs --resource-group myResourceGroup --name mycontainer3
```

輸出：

```bash
[('ROMEO', 177), ('JULIET', 134), ('CAPULET', 119)]
```

## <a name="next-steps"></a>後續步驟

### <a name="persist-task-output"></a>持續工作輸出

如需如何保存執行至完成之容器的輸入的詳細資訊，請參閱[使用 Azure Container Instances 來掛接 Azure 檔案共用](container-instances-mounting-azure-files-volume.md)。

<!-- LINKS - External -->
[aci-wordcount-image]: https://hub.docker.com/r/microsoft/aci-wordcount/

<!-- LINKS - Internal -->
[az-container-create]: /cli/azure/container?view=azure-cli-latest#az-container-create
[az-container-logs]: /cli/azure/container?view=azure-cli-latest#az-container-logs
[az-container-show]: /cli/azure/container?view=azure-cli-latest#az-container-show
[azure-cli-install]: /cli/azure/install-azure-cli
