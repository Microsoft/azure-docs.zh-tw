---
title: 使用 Azure Batch AI 叢集 | Microsoft Docs
description: 如何選擇 Batch AI 叢集組態，以及建立和管理叢集 AI
services: batch-ai
documentationcenter: ''
author: johnwu10
manager: jeconnoc
editor: ''
ms.service: batch-ai
ms.topic: article
ms.date: 08/14/2018
ms.author: danlep
ms.custom: mvc
ms.openlocfilehash: 61294d8b6b84b03b1e0c8d79b4d2855452c7f0e6
ms.sourcegitcommit: ebd06cee3e78674ba9e6764ddc889fc5948060c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44056705"
---
# <a name="work-with-batch-ai-clusters"></a>使用 Batch AI 叢集 

本文說明如何在 Azure Batch AI 中使用叢集。 文中會介紹叢集的概念、可行的組態類型，以及範例。 本文中用來建立和管理叢集的範例大多使用 Azure CLI。 不過，您也可以使用其他工具 (包括 Azure 入口網站和 Azure Batch AI SDK) 來使用叢集。

Batch AI 叢集是服務中數個資源的其中一個。 請參閱 [Batch AI 資源概觀](resource-concepts.md)以了解服務中叢集的範圍。

## <a name="introduction-to-clusters"></a>叢集簡介

Batch AI 中的叢集包含了用於執行機器學習和 AI 訓練作業的計算資源。 叢集中的所有節點都有相同的 VM 大小和 OS 映像。 Batch AI 提供許多選項供您建立自訂叢集以因應不同需求。 一般來說，您會對要完成專案所需的每個處理能力類別設定不同的叢集。 您可以根據需求和預算，來相應增加和減少叢集節點數目。 叢集可以在小組內佈建並共用，也可以每個人各自佈建自己的叢集。 

每個叢集都會存在於名為「工作區」的 Batch AI 資源底下。 在佈建任何叢集之前，您必須先設定好 Batch AI 工作區。 例如，如果您使用 Azure CLI，請使用 [az batchai workspace create](/cli/azure/batchai/workspace?view=azure-cli-latest#az-batchai-workspace-create) 命令來設定工作區。 

在建立叢集之後，您可以一次提交一個作業到佇列中。 然後，Batch AI 會處理叢集上用來執行作業的資源配置程序。 

## <a name="cluster-configuration-options"></a>叢集組態選項

在規劃叢集時，請先判斷計算需求。 Batch AI 提供了彈性的組態選項，讓您可以針對需求來量身打造叢集。 以下是佈建叢集時所要考量的主要選項：

* **VM 大小** - 選擇叢集節點的[支援區域](https://azure.microsoft.com/global-infrastructure/services/)中可用的任何 [VM 大小](../virtual-machines/linux/sizes.md)。 每個叢集只能包含一個 VM 大小，因此，如果工作需要用到多個 VM 類型，就必須為每種計算需求類型佈建個別的叢集。 若要對利用 GPU 的架構所開發的機器學習或 AI 模型進行訓練，請參閱 Azure 中的 [GPU 最佳化 VM 大小](../virtual-machines/linux/sizes-gpu.md)。
  
* **VM 優先順序** - Batch AI 可為叢集提供專用或低優先順序的 VM。 叢集內會保留專用 VM 供您使用。 低優先順序選項會配置未使用的 Azure VM 容量以便大幅節省成本，但代價是如果 Azure 回收 VM，作業可能會遭到優先佔用。 低優先順序 VM 上執行時間超過 24 小時的作業也會自動遭到優先佔用。 如果預算是個問題，請考慮對簡短的測試作業使用低優先順序 VM。 要執行時間較長的作業時，再改用專用 VM。

* **節點數目** - Batch AI 針對 叢集節點數目提供了手動和自動調整選項。 使用手動選項時，您可以控制何時要相應增加和減少叢集，以便管理您自己的計算成本。 自動調整選項可確保叢集一律會在不使用時縮小規模，以將計算成本降至最低。 

  如果您選擇手動調整叢集，則必須在叢集建立期間定義初始目標。 目標是 Batch AI 一開始會配置的節點數目。 之後，您可以手動調整節點數目大小。  

  如果您選擇自動調整選項，則必須在叢集建立期間定義最大和最小的目標節點數目。 目標的範圍可以小自 0，大到 [Batch AI 核心配額](quota-limits.md)所支援的最大節點數目。 目標為 0 可讓您既保有叢集組態，又不會產生計算費用。 如果所要求的目標高過配額限制的支援，則佈建會失敗。 

* **VM 映像** - Batch AI 預設會在佈建叢集 VM 時，使用支援容器工作負載的預設 Ubuntu Server 映像。 但您也可以從 Azure Marketplace 選擇其他已預先設定的 Linux 映像，例如[資料科學虛擬機器](../machine-learning/data-science-virtual-machine/overview.md)。 

* **儲存體** - Batch AI 會提供自動儲存體選項，您可以在使用 Azure CLI 建立叢集時選擇此選項。 此選項會自動在新的儲存體帳戶下建立 Azure 檔案共用和 Blob 容器。 在執行期間，這些儲存體資源會掛接到叢集中的每個節點，以便您能夠從本機路徑存取檔案。 儲存體帳戶名稱、檔案共用名稱、Blob 容器名稱和掛接路徑全都有預設值，並且也可以在叢集建立期間使用其他參數加以自訂。 

  如果未定義任何儲存體選項，則需要分開建立儲存體資源，並在提交作業時定義這些資源。 如果在組織層級管理叢集，卻在使用者層級管理儲存體，就很適合使用此選項。 如需如何將檔案上傳至 Azure 儲存體並在執行期間存取這些檔案的詳細資訊，請參閱[使用 Azure 儲存體儲存 Batch AI 作業的輸入和輸出](use-azure-storage.md)。

* **使用者存取** - Batch AI 可讓您在建立叢集時產生公用和私用的 SSH 金鑰檔案，但您也可以提供您自己的 SSH 金鑰，以便透過 SSH 連線到個別節點。 您也可以定義使用者名稱 (預設為目前的使用者) 和密碼。 這些認證可用來在執行期間存取節點，以便檢視各種計量或更加深入地解析作業。

* **設定工作** - Batch AI 可讓您定義要在配置時於每個節點上執行的命令列引數。 您也可以定義應該用來記錄輸出檔案的路徑。 當您有超過基礎映像的額外佈建步驟時，請使用此選項。

* **其他組態** - 有幾個較不常見的案例可能需要更特製化的組態。 在此情況下，您可以將[叢集組態檔](https://github.com/Azure/BatchAI/blob/master/documentation/using-azure-cli-20.md#using-cluster-configuration-file)連結到 Azure CLI 命令來建立叢集。 

## <a name="create-the-cluster"></a>建立叢集

決定好叢集組態選項之後，請使用 Azure CLI、Azure 入口網站或 Batch AI API 來建立叢集。 例如，若要使用 Azure CLI 建立叢集，您可以遵循 [az batchai cluster create](/cli/azure/batchai/cluster?view=azure-cli-latest#az-batchai-cluster-create) 文件，來建立可讓您獲得所需組態的確切命令。 

在最基本的組態中，下列命令可佈建具有一個節點和 SSH 存取權的 Standard_NC6 叢集。 根據預設，此叢集會包含執行最新預設 Ubuntu Server 映像的專用 VM。

```azurecli-interactive
az batchai cluster create \
    --name mycluster \
    --workspace <WORKSPACE> \
    --resource-group <RESOURCE GROUP> \
    --vm-size Standard_NC6 \
    --target 1 \
    --generate-ssh-keys
```

下列範例會佈建 Standard_NC6 叢集，此叢集可自動從 0 個節點調整到 4 個節點，並使用低優先順序節點和自動儲存體帳戶。 如果您需要低成本且容易管理的叢集，這項設定會是合適的組態。 

```azurecli-interactive
az batchai cluster create \
    --name mycluster \
    --workspace <WORKSPACE> \
    --resource-group <RESOURCE GROUP> \
    --vm-size Standard_NC6 \
    --vm-priority lowpriority \
    --max 4 \
    --min 0 \
    --use-auto-storage 
```

下列範例會佈建 Standard_NC6 叢集，此叢集包含資料科學虛擬機器映像、自訂的儲存體和掛接選項、自訂的 SSH 認證、會安裝 unzip 套件的設定工作，以及可供進行其他設定的叢集組態檔。 此組態可示範更加符合您自己需求的自訂叢集。

```azurecli-interactive
az batchai cluster create \
    --name mycluster \
    --workspace <WORKSPACE> \
    --resource-group <RESOURCE GROUP> \
    --vm-size Standard_NC6 \
    --image UbuntuDSVM \ 
    --config-file cluster.json \
    --setup-task 'apt install unzip -y'
    --storage-account-name <STORAGE ACCOUNT NAME> \
    --nfs-name nfsmount \
    --afs-name afsmount \
    --bfs-name bfsmount \
    --user-name adminuser \
    --ssh-key id_rsa.pub \
    --password secretpassword 
```

## <a name="monitor-the-cluster"></a>監視叢集

[az batchai cluster list](/cli/azure/batchai/cluster?view=azure-cli-latest#az-batchai-cluster-list)、[az batchai cluster show](/cli/azure/batchai/cluster?view=azure-cli-latest#az-batchai-cluster-show) 和 [az batchai cluster node list](/cli/azure/batchai/cluster/node?view=azure-cli-latest#az-batchai-cluster-node-list) 命令可用來監視每個叢集的各種資訊。

### <a name="list-all-clusters"></a>列出所有叢集

下列命令會列出工作區中的所有叢集。

```azurecli-interactive
az batchai cluster list \
    --workspace <WORKSPACE> \
    --resource-group <RESOURCE GROUP> 
```

### <a name="show-information-about-a-cluster"></a>顯示叢集的相關資訊

下列命令會以資料表格式顯示特定叢集的完整資訊。

```azurecli-interactive
az batchai cluster show \
    --name <CLUSTER NAME> \
    --workspace <WORKSPACE> \
    --resource-group <RESOURCE GROUP> \
    --output table
```

如果您使用自動儲存體選項佈建了叢集，建議您擷取要在上傳指令碼和訓練作業時使用的儲存體帳戶名稱。 使用下列命令：

```azurecli-interactive
az batchai cluster show \
    --name <CLUSTER NAME> \
    --workspace <WORKSPACE> \
    --resource-group <RESOURCE GROUP> \
    --query "nodeSetup.mountVolumes.azureFileShares[0].{storageAccountName:accountName}"
```

輸出應該類似如下範例。

```JSON
{
  "storageAccountName": "baixxxxxxxxx"
}
```

### <a name="list-cluster-nodes"></a>列出叢集節點

如果您需要連線到叢集節點，下列命令會擷取節點清單和連線資訊。  

```azurecli-interactive
az batchai cluster node list \
    --name <CLUSTER NAME> \
    --workspace <WORKSPACE> \
    --resource-group <RESOURCE GROUP> 
 ```

每個節點的輸出會如下所示：

```JSON
[
  {
    "ipAddress": "40.68.xxx.xxx",
    "nodeId": "tvm-xxxxxxxxxx-xxxxxxxx",
    "port": 50000.0
  }
]
```
您可以使用這項資訊，透過如下命令來對節點建立 SSH 連線。

```bash
ssh myusername@40.68.xxx.xxx -p 50000
``` 

## <a name="submit-jobs-to-the-cluster"></a>將作業提交至叢集

佈建叢集後，您便可以提交作業到節點上執行。 請參閱 [az batchai job](/cli/azure/batchai/job?view=azure-cli-latest) 命令，以了解各種使用 Azure CLI 來準備、提交和監視作業的方式。 

## <a name="downscale-cluster-for-later-use"></a>縮小叢集規模以供稍後使用

作業執行完成後，您可以縮小叢集規模。 建議您一律將未使用的叢集縮小規模，以節省計算成本。 將叢集規模縮小為 0 個節點，可讓您停止帳單費用，同時又不需要在未來需要再次擴大規模時重新佈建叢集。 如果在建立叢集時選取了自動調整，則叢集應該會自動將規模縮小為最小目標。 如果選取了手動調整，請使用下列命令縮小叢集規模。

```azurecli-interactive
az batchai cluster resize \
    --name <CLUSTER NAME> \
    --resource-group <RESOURCE GROUP> \
    --workspace <WORKSPACE> \
    --target 0
```

## <a name="delete-a-cluster"></a>刪除叢集

叢集使用完畢時，請使用 [az batchai cluster delete](/cli/azure/batchai/cluster?view=azure-cli-latest#az-batchai-cluster-delete) 命令來加以刪除。

```azurecli-interactive
az batchai cluster delete \
    --name <CLUSTER NAME> \
    --resource-group <RESOURCE GROUP> \
    --workspace <WORKSPACE>
```

刪除資源群組也會自動刪除該資源群組下的所有已佈建叢集。

```azurecli-interactive
az group delete --name <RESOURCE GROUP>
```

## <a name="next-steps"></a>後續步驟

如需更多建立 Batch AI 叢集的範例，請參閱[入口網站](quickstart-create-cluster-portal.md)或 [Azure CLI](quickstart-create-cluster-cli.md) 快速入門，或參閱 GitHub 上的 [Batch AI 配方](https://github.com/Azure/BatchAI/tree/master/recipes)。
