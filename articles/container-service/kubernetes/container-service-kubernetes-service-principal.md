---
title: Azure Kubernetes 叢集的服務主體
description: 在 Azure Container Service 中建立和管理 Kubernetes 叢集的 Azure Active Directory 服務主體
services: container-service
author: neilpeterson
manager: jeconnoc
ms.service: container-service
ms.topic: get-started-article
ms.date: 02/26/2018
ms.author: nepeters
ms.custom: mvc
ms.openlocfilehash: e3e3a981daf1273b8b2387cb1c665317f860b1d2
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46974862"
---
# <a name="set-up-an-azure-ad-service-principal-for-a-kubernetes-cluster-in-container-service"></a>在 Container Service 中設定 Kubernetes 叢集的 Azure AD 服務主體

[!INCLUDE [aks-preview-redirect.md](../../../includes/aks-preview-redirect.md)]

在 Azure Container Service 中，Kubernetes 叢集需要 [Azure Active Directory 服務主體](../../active-directory/develop/app-objects-and-service-principals.md)，才能與 Azure API 進行互動。 需要服務主體，才能以動態方式管理資源，例如[使用者定義的路由](../../virtual-network/virtual-networks-udr-overview.md)及[第 4 層 Azure Load Balancer](../../load-balancer/load-balancer-overview.md)。


本文說明為 Kubernetes 叢集設定服務主體的各種選項。 例如，如果您已安裝並設定 [Azure CLI](/cli/azure/install-az-cli2)，您可以執行 [`az acs create`](/cli/azure/acs#az_acs_create) 命令，在同一時間建立 Kubernetes 叢集與服務主體。


## <a name="requirements-for-the-service-principal"></a>服務主體的需求

您可以使用符合下列需求的現有 Azure AD 服務主體，或建立一個新的服務主體。

* **範圍**：資源群組

* **角色**：參與者

* **用戶端密碼**：必須是密碼。 目前，您無法使用針對憑證驗證設定的服務主體。

> [!IMPORTANT]
> 若要建立服務主體，您必須有足夠權限向 Azure AD 租用戶註冊應用程式，並將應用程式指派給您訂用帳戶中的角色。 若要查看您是否有必要的權限，請[在入口網站中檢查](../../azure-resource-manager/resource-group-create-service-principal-portal.md#required-permissions)。
>

## <a name="option-1-create-a-service-principal-in-azure-ad"></a>選項 1：在 Azure AD 中建立服務主體

如果您想要在部署 Kubernetes 叢集之前建立 Azure AD 服務主體，Azure 會提供數種方法。

下列範例命令示範如何使用 [Azure CLI](../../azure-resource-manager/resource-group-authenticate-service-principal-cli.md) 執行這項作業。 或者，您也可以使用 [Azure PowerShell](../../azure-resource-manager/resource-group-authenticate-service-principal.md)、[入口網站](../../azure-resource-manager/resource-group-create-service-principal-portal.md)或其他方法來建立服務主體。

```azurecli
az login

az account set --subscription "mySubscriptionID"

az group create --name "myResourceGroup" --location "westus"

az ad sp create-for-rbac --role="Contributor" --scopes="/subscriptions/<subscriptionID>/resourceGroups/<resourceGroupName>"
```

輸出如下所示 (以下顯示擬定的輸出)：

![建立服務主體](./media/container-service-kubernetes-service-principal/service-principal-creds.png)

醒目提示的是您做為叢集部署之服務主體參數的**用戶端識別碼** (`appId`) 和**用戶端密碼** (`password`)。


### <a name="specify-service-principal-when-creating-the-kubernetes-cluster"></a>在建立 Kubernetes 叢集時指定服務主體

當您建立 Kubernetes 叢集時，提供現有服務主體的**用戶端識別碼** (也稱為`appId`，適用於應用程式識別碼) 和**用戶端密碼** (`password`) 做為參數。 確定服務主體符合本文開頭所述的需求。

使用 [Azure 命令列介面 (CLI)](container-service-kubernetes-walkthrough.md)、[Azure 入口網站](../dcos-swarm/container-service-deployment.md)或其他方法部署 Kubernetes 叢集時，您可以指定這些參數。

>[!TIP]
>指定**用戶端識別碼**時，務必使用服務主體的 `appId`，而非 `ObjectId`。
>

下列範例顯示使用 Azure CLI 傳遞參數的其中一種方式。 此範例使用 [Kubernetes 快速入門範本](https://github.com/Azure/azure-quickstart-templates/tree/master/101-acs-kubernetes)。

1. 從 GitHub [下載](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-acs-kubernetes/azuredeploy.parameters.json)範本參數檔案 `azuredeploy.parameters.json`。

2. 若要指定服務主體，在檔案中輸入 `servicePrincipalClientId` 和 `servicePrincipalClientSecret` 的值。 (您也必須提供自己的 `dnsNamePrefix` 和 `sshRSAPublicKey` 值。 後者是可存取叢集的 SSH 公開金鑰)。儲存檔案。

    ![傳遞服務主體參數](./media/container-service-kubernetes-service-principal/service-principal-params.png)

3. 執行下列命令，並使用 `--parameters` 來設定 azuredeploy.parameters.json 檔案的路徑。 此命令會在美國西部區域中名為 `myResourceGroup` 的資源群組中部署叢集。

    ```azurecli
    az login

    az account set --subscription "mySubscriptionID"

    az group create --name "myResourceGroup" --location "westus"

    az group deployment create -g "myResourceGroup" --template-uri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-acs-kubernetes/azuredeploy.json" --parameters @azuredeploy.parameters.json
    ```


## <a name="option-2-generate-a-service-principal-when-creating-the-cluster-with-az-acs-create"></a>選項 2︰在使用 `az acs create` 建立叢集時產生服務主體

如果您執行 [`az acs create`](/cli/azure/acs#az-acs-create) 命令來建立 Kubernetes 叢集，您可以選擇自動產生服務主體。

至於其他 Kubernetes 叢集建立選項，您可以在執行 `az acs create` 時指定現有服務主體的參數。 不過，當您省略這些參數時，Azure CLI 會自動建立一個參數以搭配 Container Service 使用。 這會在部署期間以透明方式發生。

下列命令會建立 Kubernetes 叢集並產生 SSH 金鑰和服務主體認證︰

```console
az acs create -n myClusterName -d myDNSPrefix -g myResourceGroup --generate-ssh-keys --orchestrator-type kubernetes
```

> [!IMPORTANT]
> 如果您的帳戶沒有 Azure AD 和訂用帳戶權限可建立服務主體，則命令會產生類似 `Insufficient privileges to complete the operation.` 的錯誤。
>

## <a name="additional-considerations"></a>其他考量

* 如果您沒有在訂用帳戶中建立服務主體的權限，您可能需要要求 Azure AD 或訂用帳戶管理員指派所需的權限，或要求他們提供服務主體以搭配 Azure Container Service 使用。

* Kubernetes 的服務主體是叢集組態的一部分。 不過，請勿使用身分識別來部署叢集。

* 每個服務主體都會與 Azure AD 應用程式相關聯。 Kubernetes 叢集的服務主體可與任何有效的 Azure AD 應用程式名稱相關聯 (例如：`https://www.contoso.org/example`)。 應用程式的 URL 不一定是實際端點。

* 指定服務主體的 [用戶端識別碼] 時，您可以使用 `appId` 的值 (如本文所示) 或對應的服務主體`name` (例如，`https://www.contoso.org/example`)。

* 在 Kubernetes 叢集中的主要和代理程式 VM 上，服務主體認證會儲存在 `/etc/kubernetes/azure.json` 檔案中。

* 當您使用 `az acs create` 命令自動產生服務主體時，服務主體認證會寫入用來執行命令之電腦上的 `~/.azure/acsServicePrincipal.json` 檔案。

* 當您使用 `az acs create` 命令自動產生服務主體時，服務主體也可以向在訂用帳戶中建立的 [Azure Container Registry](../../container-registry/container-registry-intro.md) 進行驗證。

* 服務主體認證可能會過期，導致您的叢集節點進入 **NotReady** 狀態。 請參閱[認證到期](#credential-expiration)一節，以取得風險降低資訊。

## <a name="credential-expiration"></a>認證到期

除非您在建立服務主體時以 `--years` 參數指定自訂的有效範圍，否則其認證的有效期為自建立起 1 年內。 當認證到期時，您的叢集節點可能會進入 **NotReady** 狀態。

若要檢查服務主體的到期日，請執行 [az ad app show](/cli/azure/ad/app#az-ad-app-show) 命令並搭配 `--debug` 參數，然後在輸出底部附近尋找 `passwordCredentials` 的 `endDate` 值：

```azurecli
az ad app show --id <appId> --debug
```

輸出 (此處顯示已截斷的內容)：

```json
...
"passwordCredentials":[{"customKeyIdentifier":null,"endDate":"2018-11-20T23:29:49.316176Z"
...
```

如果您的服務主體認證到期，請使用 [az ad sp reset-credentials](/cli/azure/ad/sp#az-ad-sp-reset-credentials) 命令更新認證：

```azurecli
az ad sp reset-credentials --name <appId>
```

輸出：

```json
{
  "appId": "4fd193b0-e6c6-408c-a21a-803441ad2851",
  "name": "4fd193b0-e6c6-408c-a21a-803441ad2851",
  "password": "404203c3-0000-0000-0000-d1d2956f3606",
  "tenant": "72f988bf-0000-0000-0000b-2d7cd011db47"
}
```

然後，使用所有叢集節點上的新認證更新 `/etc/kubernetes/azure.json`，並重新啟動節點。

## <a name="next-steps"></a>後續步驟

* [開始使用容器服務叢集中的 Kubernetes](container-service-kubernetes-walkthrough.md)。

* 若要針對 Kubernetes 的服務主體進行疑難排解，請參閱 [ACS 引擎文件](https://github.com/Azure/acs-engine/blob/master/docs/kubernetes.md#troubleshooting)。
