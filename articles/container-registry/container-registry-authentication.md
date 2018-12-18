---
title: 向 Azure Container Registry 進行驗證 | Microsoft Docs
description: Azure Container Registry 的驗證選項，包括 Azure Active Directory 服務主體的直接和登錄登入。
services: container-registry
author: stevelas
manager: jeconnoc
ms.service: container-registry
ms.topic: article
ms.date: 01/23/2018
ms.author: stevelas
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: be8adf9779c2d168c0ac7a0ed7dbc3e85935df68
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44293723"
---
# <a name="authenticate-with-a-private-docker-container-registry"></a>向私用 Docker 容器登錄進行驗證

向 Azure Container Registry 進行驗證的方式有數種，每一種方式都適用於一或多個登錄使用案例。

您可以直接透過[個人登入](#individual-login-with-azure-ad)來登入登錄，而您的應用程式和容器協調器則可以使用 Azure Active Directory (Azure AD) [服務主體](#service-principal)來執行自動 (或「遠端控制」) 驗證。

Azure Container Registry 不支援未經驗證的 Docker 作業或匿名存取。 針對公用映像，您可以使用 [Docker Hub](https://docs.docker.com/docker-hub/) \(英文\)。

## <a name="individual-login-with-azure-ad"></a>使用 Azure AD 進行個人登入

直接與您的登錄搭配運作時 (例如將映像拖曳至開發工作站或從開發工作站推送映像)，請在 [Azure CLI](/cli/azure/install-azure-cli) 中使用 [az acr login](/cli/azure/acr?view=azure-cli-latest#az-acr-login) 命令來進行驗證：

```azurecli
az acr login --name <acrName>
```

當您使用 `az acr login` 來進行登入時，CLI 會使用您執行 `az login` 時所建立的權杖，以順暢地向登錄驗證您的工作階段。 在您以此方式登入之後，系統會快取您的認證，而後續的 `docker` 命令就不會要求提供使用者名稱和密碼。 如果您的權杖過期，您可以再次使用 `az acr login` 命令進行重新驗證來重新整理該權杖。 使用 `az acr login` 搭配 Azure 身分識別可提供[角色型存取](../role-based-access-control/role-assignments-portal.md)功能。

## <a name="service-principal"></a>服務主體

您可以將[服務主體](../active-directory/develop/app-objects-and-service-principals.md)指派給登錄，而您的應用程式或服務便可以使用它來進行遠端控制驗證。 服務主體可允許對登錄進行[角色型存取](../role-based-access-control/role-assignments-portal.md)，而您可以將多個服務主體指派給登錄。 多個服務主體可讓您為不同的應用程式定義不同的存取權。

可用的角色包括：

  * **讀取者**：提取
  * **參與者**：提取和推送
  * **擁有者**：提取、推送，以及指派角色給其他使用者

服務主體可在如下的推送及提取案例中，對登錄啟用遠端控制連線：

  * *讀取者*：從登錄到協調流程系統 (包括 Kubernetes、DC/OS 及 Docker Swarm) 的容器部署。 您也可以從容器登錄提取到相關的 Azure 服務，例如 [AKS](../aks/index.yml)、[App Service](../app-service/index.yml)、[Batch](../batch/index.yml)、[Service Fabric](/azure/service-fabric/) 等。

  * *參與者*：建立容器映像並將其推送到登錄的持續整合和部署解決方案 (例如 Azure DevOps 或 Jenkins)。

> [!TIP]
> 您可以執行 [az ad sp reset-credentials](/cli/azure/ad/sp?view=azure-cli-latest#az-ad-sp-reset-credentials) 命令來重新產生服務主體的密碼。
>

您也可以使用服務主體來直接登入。 請將應用程式識別碼和服務主體的密碼提供給 `docker login` 命令：

```
docker login myregistry.azurecr.io -u xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx -p myPassword
```

登入後，Docker 會快取認證，因此您不需要記憶應用程式識別碼。

視您已安裝的 Docker 版本而定，您可能會看到安全性警告，其中會建議您使用 `--password-stdin` 參數。 其使用超出本文的範圍時，建議您遵循此最佳做法。 如需詳細資訊，請參閱 [docker login](https://docs.docker.com/engine/reference/commandline/login/) 命令參考。

如需使用服務主體對 ACR 進行無周邊驗證的詳細資訊，請參閱[使用服務主體進行 Azure Container Registry 驗證](container-registry-auth-service-principal.md)。

## <a name="admin-account"></a>管理帳戶

每個容器登錄都包含一個管理使用者帳戶，且預設為停用。 您可以在 [Azure 入口網站](container-registry-get-started-portal.md#create-a-container-registry)中或藉由使用 Azure CLI，啟用此管理使用者並管理其認證。

> [!IMPORTANT]
> 管理帳戶是專為讓單一使用者存取登錄而設計，主要用於測試。 不建議將管理帳戶認證與其他使用者共用。 所有使用管理帳戶進行驗證的使用者會顯示為單一使用者，此使用者具備登錄的推送和提取存取權。 變更或停用此帳戶時，會將所有使用其認證之使用者的登錄存取權都停用。 針對遠端控制案例的使用者和服務主體，建議使用個人身分識別。
>

管理帳戶隨附兩個密碼，兩個密碼都可以重新產生。 兩個密碼可讓您在重新產生其中一個密碼時，使用另一個密碼來維持與登錄的連線。 如果已啟用管理帳戶，您便可以將使用者名稱和其中一個密碼傳遞給 `docker login` 命令，來向登錄進行基本驗證。 例如︰

```
docker login myregistry.azurecr.io -u myAdminName -p myPassword1
```

同樣地，為了提升安全性，Docker 會建議您使用 `--password-stdin` 參數，而不要在命令列上提供密碼。 您也可以只指定您的使用者名稱 (不搭配 `-p`)，然後在出現提示時輸入您的密碼。

若要為現有的登錄啟用管理使用者，您可以在 Azure CLI 中使用 [az acr update](/cli/azure/acr?view=azure-cli-latest#az-acr-update) 命令的 `--admin-enabled` 參數：

```azurecli
az acr update -n <acrName> --admin-enabled true
```

您可以在 Azure 入口網站中啟用管理使用者，方法是瀏覽至您的登錄，選取 [設定]底下的 [存取金鑰]，然後選取 [管理使用者]底下的 [啟用]。

![Azure 入口網站中的啟用管理使用者 UI][auth-portal-01]

## <a name="next-steps"></a>後續步驟

* [使用 Azure CLI 推送您的第一個映像](container-registry-get-started-azure-cli.md)

<!-- IMAGES -->
[auth-portal-01]: ./media/container-registry-authentication/auth-portal-01.png
