---
title: 包含檔案
description: 包含檔案
services: container-registry
author: dlepow
ms.service: container-registry
ms.topic: include
ms.date: 08/03/2018
ms.author: danlep
ms.custom: include file
ms.openlocfilehash: a76f0205a34b106cf04b61938b1b576db9325c40
ms.sourcegitcommit: 67abaa44871ab98770b22b29d899ff2f396bdae3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/08/2018
ms.locfileid: "48858038"
---
## <a name="create-a-service-principal"></a>建立服務主體

若要建立具容器登錄存取權的服務主體，請在 [Azure Cloud Shell](../articles/cloud-shell/overview.md) 或本機的 [Azure CLI](/cli/azure/install-azure-cli) 安裝中執行下列指令碼。 此指令碼會針對 Bash 殼層加以格式化。

執行指令碼之前，請使用容器登錄的名稱更新 `ACR_NAME` 變數。 `SERVICE_PRINCIPAL_NAME` 值在 Azure Active Directory 租用戶內必須是唯一的。 如果您收到「`'http://acr-service-principal' already exists.`」錯誤，請為服務主體指定不同的名稱。

(選擇性) 如果您要授與不同權限，則可以修改 [az ad sp create-for-rbac][az-ad-sp-create-for-rbac] 命令中的 `--role` 值。

執行指令碼之後，請記下的服務主體的**識別碼**和**密碼**。 一旦擁有其認證，便可以將您的應用程式和服務設定為以服務主體向您的容器登錄進行驗證。

<!-- https://github.com/Azure-Samples/azure-cli-samples/blob/master/container-registry/service-principal-create/service-principal-create.sh --> [!code-azurecli-interactive[acr-sp-create](~/cli_scripts/container-registry/service-principal-create/service-principal-create.sh)]

## <a name="use-an-existing-service-principal"></a>使用現有的服務主體

若要授與現有服務主體登錄存取權，您必須為服務主體指派新的角色。 如同建立新的服務主體，您可以授與提取、推送和提取，以及擁有者存取權。

以下指令碼使用 [az role assignment create][az-role-assignment-create] 命令，以授與您在 `SERVICE_PRINCIPAL_ID` 變數中所指定的服務主體「提取」權限。 如果您要授與不同層級的存取權，請調整 `--role` 值。

<!-- https://github.com/Azure-Samples/azure-cli-samples/blob/master/container-registry/service-principal-assign-role/service-principal-assign-role.sh --> [!code-azurecli-interactive[acr-sp-role-assign](~/cli_scripts/container-registry/service-principal-assign-role/service-principal-assign-role.sh)]

<!-- LINKS - Internal -->
[az-ad-sp-create-for-rbac]: /cli/azure/ad/sp#az_ad_sp_create_for_rbac
[az-role-assignment-create]: /cli/azure/role/assignment#az_role_assignment_create
