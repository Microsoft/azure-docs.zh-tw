---
title: '在啟用 Arc 的 Kubernetes 叢集 (預覽版上使用 Gitops) 將部署設定) '
services: azure-arc
ms.service: azure-arc
ms.date: 05/19/2020
ms.topic: article
author: mlearned
ms.author: mlearned
description: 針對已啟用 Azure Arc 的叢集設定使用 GitOps (預覽)
keywords: GitOps, Kubernetes, K8s, Azure, Arc, Azure Kubernetes Service, 容器
ms.openlocfilehash: ce6c754c308d2979db9b1b8eb36e7858e8a91c3c
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 11/17/2020
ms.locfileid: "94659789"
---
# <a name="deploy-configurations-using-gitops-on-arc-enabled-kubernetes-cluster-preview"></a>在啟用 Arc 的 Kubernetes 叢集 (預覽版上使用 Gitops) 將部署設定) 

Gitops 是在 Git 存放庫中宣告 Kubernetes 設定的預期狀態 (部署、命名空間等) ，然後操作員使用設定好的輪詢和提取，部署到叢集的做法。 本檔涵蓋 Azure Arc 啟用的 Kubernetes 叢集上的工作流程設定。

在您的叢集有一或多個 Git 存放庫的連線，這些連線，會在 Azure Resource Manager 中當成 `sourceControlConfiguration` 延伸模組資源進行追蹤。 `sourceControlConfiguration` 資源屬性代表 Kubernetes 資源從 Git 流向您叢集的位置和方式。 `sourceControlConfiguration`資料會以加密儲存在 Azure Cosmos DB 資料庫中，以確保資料機密性。

在您的叢集中執行的 `config-agent` 會負責在 `sourceControlConfiguration` 啟用 Azure Arc 在 Kubernetes 資源上監看新的或更新的延伸模組資源、部署 flux 操作員以監看 Git 存放庫，以及傳播所對 `sourceControlConfiguration` 的任何更新。 您甚至可以在 `namespace` 範圍位相同的 Azure Arc 啟用的 Kubernetes 叢集來建立多個 `sourceControlConfiguration` 資源，以達成多租使用者。在這種情況下，每個運算子只能將設定部署到其各自的命名空間。

Git 存放庫可以包含任何有效的 Kubernetes 資源，包括命名空間、ConfigMaps、部署、Daemonset 等。其也可能包含用來部署應用程式的 Helm 圖表。 常見的一組案例是包括定義組織的基準設定，其中可能包括常見的 Azure 角色和系結、監視或記錄代理程式，或全叢集服務。

您可以使用相同的模式來管理較大的叢集集合，這些叢集可能會跨不同的環境進行部署。 例如，您可能有一個存放庫會定義貴組織的基準設定，並一次將該設定套用至數十個 Kubernetes 叢集。 [Azure 原則可](use-azure-policy.md) 在 `sourceControlConfiguration` 訂用帳戶或資源群組的範圍下，以一組特定 Azure Arc 參數自動建立。

此快速入門手冊將逐步引導您在叢集管理範圍內套用一組設定。

## <a name="before-you-begin"></a>開始之前

本文假設您具有已啟用 Azure Arc 的 Kubernetes 連線叢集。 如果您需要已連線的叢集，請參閱[連線叢集快速入門](./connect-cluster.md)。

## <a name="create-a-configuration"></a>建立設定

本檔中使用的 [範例存放庫](https://github.com/Azure/arc-k8s-demo) 是以叢集操作員的角色為結構，而該角色需要布建一些命名空間、部署一般工作負載，並提供一些小組專屬的設定。 使用此存放庫，會在您的叢集上建立下列資源：

**命名空間：** `cluster-config`、`team-a`、`team-b`
**部署：** `cluster-config/azure-vote`
**ConfigMap：** `team-a/endpoints`

`config-agent`會每隔30秒輪詢 Azure 以進行新增或更新 `sourceControlConfiguration` ，這是 `config-agent` 挑選新的或更新的設定所花費的時間上限。
如果您要將私人存放庫與建立關聯 `sourceControlConfiguration` ，請確定您也已完成 [從私用 git 存放庫](#apply-configuration-from-a-private-git-repository)套用設定中的步驟。

### <a name="using-azure-cli"></a>使用 Azure CLI

使用的 Azure CLI 擴充功能 `k8sconfiguration` ，讓我們將連線的叢集連結至 [範例 git 存放庫](https://github.com/Azure/arc-k8s-demo)。 我們會將此設定命名為 `cluster-config`、指示代理程式在 `cluster-config` 命名空間中部署操作員，並為該操作員授與 `cluster-admin` 權限。

```console
az k8sconfiguration create --name cluster-config --cluster-name AzureArcTest1 --resource-group AzureArcTest --operator-instance-name cluster-config --operator-namespace cluster-config --repository-url https://github.com/Azure/arc-k8s-demo --scope cluster --cluster-type connectedClusters
```

**輸出：**

```console
Command group 'k8sconfiguration' is in preview. It may be changed/removed in a future release.
{
  "complianceStatus": {
    "ComplianceStatus": 1,
    "clientAppliedTime": null,
    "level": 3,
    "message": "{\"OperatorMessage\":null,\"ClusterState\":null}"
  },
  "configKind": "Git",
  "configName": "cluster-config",
  "configOperator": {
    "operatorInstanceName": "cluster-config",
    "operatorNamespace": "cluster-config",
    "operatorParams": "--git-readonly",
    "operatorScope": "cluster",
    "operatorType": "flux"
  },
  "configType": "",
  "id": null,
  "name": null,
  "operatorInstanceName": null,
  "operatorNamespace": null,
  "operatorParams": null,
  "operatorScope": null,
  "operatorType": null,
  "providerName": "ConnectedClusters",
  "provisioningState": "Succeeded",
  "repositoryPublicKey": null,
  "repositoryUrl": null,
  "sourceControlConfiguration": {
    "repositoryPublicKey": "",
    "repositoryUrl": "git://github.com/Azure/arc-k8s-demo.git"
  },
  "type": null
}
```

#### <a name="repository-url-parameter"></a>repository-url 參數

以下是 --repository-url 參數值的支援案例。

| 狀況 | [格式] | 描述 |
| ------------- | ------------- | ------------- |
| 公用 Git 存放庫 | HTTP [s]：//server/repo.git 或 git://server/repo.git   | 公用 Git 存放庫  |
| 私人 Git 存放庫– SSH – Flux 建立的金鑰 | ssh：//[user@] server/存放庫。 git 或 [user@] 伺服器：存放庫 git | Flux 產生的公開金鑰必須加入至您 Git 服務提供者中的使用者帳戶。 如果將部署金鑰新增至存放庫而非使用者帳戶，請使用 `git@` 取代 `user@` 。 在 [這裡](#apply-configuration-from-a-private-git-repository) |

Flux 支援這些案例，但尚未由 sourceControlConfiguration 支援。

| 狀況 | [格式] | 描述 |
| ------------- | ------------- | ------------- |
| 私人 Git 存放庫-HTTPS | https://server/repo.git | 即將推出 (將支援使用者名稱/密碼、使用者名稱/權杖、憑證)  |
| 私人 Git 存放庫-SSH –使用者提供的金鑰 | ssh：//[user@] server/存放庫。 git 或 [user@] 伺服器：存放庫 git | 即將推出 |
| 私人 Git 主機– SSH –自訂 known_hosts | ssh：//[user@] server/存放庫。 git 或 [user@] 伺服器：存放庫 git | 即將推出 |

#### <a name="additional-parameters"></a>其他參數

若要自訂設定的建立，以下是一些額外參數：

`--enable-helm-operator`：「選擇性」參數，可啟用對 Helm 圖表部署的支援。

`--helm-operator-chart-values`：適用於 Helm 操作員 (如果已啟用) 的「選擇性」圖表值。  例如，'--set helm.versions=v3'。

`--helm-operator-chart-version`：適用於 Helm 操作員 (如果已啟用) 的「選擇性」圖表版本。 預設值：'0.6.0'。

`--operator-namespace`：操作員命名空間的「選擇性」名稱。 預設值：'default'

`--operator-params`：適用於操作員的「選擇性」參數。 必須在單引號中提供。 例如， ```--operator-params='--git-readonly --git-path=releases' ```

--operator-params 中支援的選項

| 選項 | 描述 |
| ------------- | ------------- |
| --git-branch  | 要用於 Kubernetes 資訊清單的 Git 存放庫分支。 預設值是 'master'。 |
| --git-path  | 適用於 Flux 之 Git 存放庫內的相對路徑，可尋找 Kubernetes 資訊清單。 |
| --git-readonly | Git 存放庫將被視為唯讀；Flux 將不會嘗試寫入至其中。 |
| --manifest-generation  | 啟用時，Flux 將會尋找 .flux.yaml，並執行 Kustomize 或其他資訊清單產生器。 |
| --git-poll-interval  | 輪詢 Git 存放庫以取得新認可的期間。 預設值為 '5m' (5 分鐘)。 |
| --sync-garbage-collection  | 啟用時，Flux 將刪除其已建立但不再出現於 Git 中的資源。 |
| --git-label  | 用來追蹤同步處理進度的標籤，可用來標記 Git 分支。  預設值為 'flux-sync'。 |
| --git-user  | 適用於 Git 認可的使用者名稱。 |
| --git-email  | 用於 Git 認可的電子郵件。 |

* 如果未設定 '--git-user' 或 '--git-email' (這表示您不希望 Flux 寫入至存放庫)，將會自動設定 --git-readonly (如果您尚未設定)。

* 如果 enableHelmOperator 為 True，則 operatorInstanceName + operatorNamespace 字串組合不能超過 47 個字元。  如果您無法遵守這項限制，您將會收到下列錯誤：

   ```console
   {"OperatorMessage":"Error: {failed to install chart from path [helm-operator] for release [<operatorInstanceName>-helm-<operatorNamespace>]: err [release name \"<operatorInstanceName>-helm-<operatorNamespace>\" exceeds max length of 53]} occurred while doing the operation : {Installing the operator} on the config","ClusterState":"Installing the operator"}
   ```

如需詳細資訊，請參閱 [Flux 檔](https://aka.ms/FluxcdReadme)。

> [!TIP]
> 您也可以在 Azure 入口網站上 **的 [設定] 索引** 標籤上建立 sourceControlConfiguration，也就是 Azure Arc enabled Kubernetes resource blade。

## <a name="validate-the-sourcecontrolconfiguration"></a>驗證 sourceControlConfiguration

使用 Azure CLI 來驗證是否已成功建立 `sourceControlConfiguration`。

```console
az k8sconfiguration show --name cluster-config --cluster-name AzureArcTest1 --resource-group AzureArcTest --cluster-type connectedClusters
```

請注意，`sourceControlConfiguration` 資源會利用合規性狀態、訊息和偵錯資訊進行更新。

**輸出：**

```console
Command group 'k8sconfiguration' is in preview. It may be changed/removed in a future release.
{
  "complianceStatus": {
    "complianceState": "Installed",
    "lastConfigApplied": "2019-12-05T05:34:41.481000",
    "message": "...",
    "messageLevel": "3"
  },
  "id": "/subscriptions/57ac26cf-a9f0-4908-b300-9a4e9a0fb205/resourceGroups/AzureArcTest/providers/Microsoft.Kubernetes/connectedClusters/AzureArcTest1/providers/Microsoft.KubernetesConfiguration/sourceControlConfigurations/cluster-config",
  "name": "cluster-config",
  "operatorInstanceName": "cluster-config",
  "operatorNamespace": "cluster-config",
  "operatorParams": "--git-readonly",
  "operatorScope": "cluster",
  "operatorType": "Flux",
  "provisioningState": "Succeeded",
  "repositoryPublicKey": "...",
  "repositoryUrl": "git://github.com/Azure/arc-k8s-demo.git",
  "resourceGroup": "AzureArcTest",
  "type": "Microsoft.KubernetesConfiguration/sourceControlConfigurations"
}
```

建立 `sourceControlConfiguration` 時，會在幕後發生下列動作：

1. Azure Arc `config-agent` 會監視 Azure Resource Manager 的新設定或更新設定 (`Microsoft.KubernetesConfiguration/sourceControlConfiguration`)
1. `config-agent` 會注意到新的 `Pending` 設定
1. `config-agent` 會讀取設定屬性，並準備部署 `flux` 的受控執行個體
    * `config-agent` 會建立目的地命名空間
    * `config-agent` 會準備一個具有適當權限 (`cluster` 或 `namespace` 範圍) 的 Kubernetes Service 帳戶
    * `config-agent` 會部署 `flux` 的執行個體
    * `flux` 產生 SSH 金鑰並記錄公開金鑰
1. `config-agent` 會向 `sourceControlConfiguration` 回報狀態

當佈建程序發生時，`sourceControlConfiguration` 將歷經數個狀態變更。 使用上述 `az k8sconfiguration show ...` 命令來監視進度：

1. `complianceStatus` -> `Pending`：代表初始和進行中狀態
1. `complianceStatus` -> `Installed`：`config-agent` 能夠順利設定叢集並部署 `flux`，而不會發生錯誤
1. `complianceStatus` -> `Failed`：`config-agent` 在部署 `flux` 時發生錯誤，`complianceStatus.message` 回應本文中應該會提供詳細資料

## <a name="apply-configuration-from-a-private-git-repository"></a>套用來自私人 Git 存放庫的設定

如果您使用私用 git 存放庫，則需要執行另一個工作來關閉迴圈：將產生的公開金鑰 `flux` 做為存放庫中的 **部署金鑰** 。

**使用 Azure CLI 取得公開金鑰**

```console
$ az k8sconfiguration show --resource-group <resource group name> --cluster-name <connected cluster name> --name <configuration name> --query 'repositoryPublicKey'
Command group 'k8sconfiguration' is in preview. It may be changed/removed in a future release.
"ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAREDACTED"
```

**從 Azure 入口網站取得公開金鑰**

1. 在 Azure 入口網站中，瀏覽至連線的叢集資源。
2. 在資源頁面中選取 [設定]，並查看此叢集的設定清單。
3. 選取使用私人 Git 存放庫的設定。
4. 在開啟的內容視窗中，複製視窗底部的 **存放庫公開金鑰**。

如果您使用的是 GitHub，請使用下列2個選項的其中一個：

**選項1：將公開金鑰加入您的使用者帳戶**

1. 開啟 GitHub，按一下頁面右上角的設定檔圖示。
2. 按一下 [**設定**]
3. 按一下 **SSH 和 GPG 金鑰**
4. 按一下 [**新增 SSH 金鑰**]
5. 提供標題
6. 貼上公開金鑰 (不包括周圍的所有引號)
7. 按一下 [**新增 SSH 金鑰**]

**選項2：將公開金鑰做為部署金鑰加入至 git 存放庫**

1. 開啟 GitHub、流覽至您的存放庫、至 **設定**，然後 **部署金鑰**
2. 按一下 [**新增部署金鑰**]
3. 提供標題
4. 檢查 **允許寫入權限**
5. 貼上公開金鑰 (不包括周圍的所有引號)
6. 按一下 [**新增金鑰**]

**如果您使用 Azure DevOps 存放庫，請將金鑰新增至您的 SSH 金鑰**

1. 在右上角的 [使用者設定] 底下 (設定檔影像旁邊)，按一下 [SSH 公開金鑰]
1. 選取 [+ 新增金鑰]。
1. 提供名稱
1. 貼上周圍不含任何引號的公開金鑰
1. 按一下 [新增]

## <a name="validate-the-kubernetes-configuration"></a>驗證 Kubernetes 設定

當 `config-agent` 安裝 `flux` 執行個體之後，Git 存放庫中保留的資源應該會開始流向叢集。 檢查以查看是否已建立命名空間、部署和資源：

```console
kubectl get ns --show-labels
```

**輸出：**

```console
NAME              STATUS   AGE    LABELS
azure-arc         Active   24h    <none>
cluster-config    Active   177m   <none>
default           Active   29h    <none>
itops             Active   177m   fluxcd.io/sync-gc-mark=sha256.9oYk8yEsRwWkR09n8eJCRNafckASgghAsUWgXWEQ9es,name=itops
kube-node-lease   Active   29h    <none>
kube-public       Active   29h    <none>
kube-system       Active   29h    <none>
team-a            Active   177m   fluxcd.io/sync-gc-mark=sha256.CS5boSi8kg_vyxfAeu7Das5harSy1i0gc2fodD7YDqA,name=team-a
team-b            Active   177m   fluxcd.io/sync-gc-mark=sha256.vF36thDIFnDDI2VEttBp5jgdxvEuaLmm7yT_cuA2UEw,name=team-b
```

我們可以看到已建立 `team-a`、`team-b`、`itops` 和 `cluster-config` 命名空間。

`flux` 操作員已部署至 `cluster-config` 命名空間，如我們的 `sourceControlConfig` 所指示：

```console
kubectl -n cluster-config get deploy  -o wide
```

**輸出：**

```console
NAME             READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES                         SELECTOR
cluster-config   1/1     1            1           3h    flux         docker.io/fluxcd/flux:1.16.0   instanceName=cluster-config,name=flux
memcached        1/1     1            1           3h    memcached    memcached:1.5.15               name=memcached
```

## <a name="further-exploration"></a>進一步探索

您可以瀏覽其他部署為設定存放庫一部分的資源：

```console
kubectl -n team-a get cm -o yaml
kubectl -n itops get all
```

## <a name="delete-a-configuration"></a>刪除設定

`sourceControlConfiguration`使用 Azure CLI 或 Azure 入口網站刪除。  在您起始 delete 命令之後， `sourceControlConfiguration` 資源將會立即在 Azure 中刪除，但最多可能需要1小時的時間，才能從叢集中完整刪除相關聯的物件 (我們有一個待處理專案可減少此時間延遲) 。

> [!NOTE]
> 建立命名空間範圍的 sourceControlConfiguration 之後，使用者可以 `edit` 在命名空間上進行角色系結，以在此命名空間上部署工作負載。 當 `sourceControlConfiguration` 刪除具有命名空間範圍的此項時，命名空間會保持不變，且不會刪除，以避免破壞這些其他工作負載。
> 當刪除時，不會刪除從追蹤的 git 存放庫部署所產生之叢集的任何變更 `sourceControlConfiguration` 。

```console
az k8sconfiguration delete --name cluster-config --cluster-name AzureArcTest1 --resource-group AzureArcTest --cluster-type connectedClusters
```

**輸出：**

```console
Command group 'k8sconfiguration' is in preview. It may be changed/removed in a future release.
```

## <a name="next-steps"></a>後續步驟

- [使用 Helm 搭配原始檔控制設定](./use-gitops-with-helm.md)
- [使用 Azure 原則來控管叢集設定](./use-azure-policy.md)
