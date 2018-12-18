---
title: 整合 Azure Active Directory 與 Azure Kubernetes Service
description: 如何建立已啟用 Azure Active Directory 的 Azure Kubernetes Service (AKS) 叢集。
services: container-service
author: iainfoulds
ms.service: container-service
ms.topic: article
ms.date: 8/9/2018
ms.author: iainfou
ms.custom: mvc
ms.openlocfilehash: 9bbf7ad201a70a315b75ed5e1f35671e4a5604fc
ms.sourcegitcommit: 30c7f9994cf6fcdfb580616ea8d6d251364c0cd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/18/2018
ms.locfileid: "42140899"
---
# <a name="integrate-azure-active-directory-with-aks"></a>整合 Azure Active Directory 與 AKS

Azure Kubernetes Service (AKS) 可以設定為使用 Azure Active Directory (AD) 進行使用者驗證。 在此組態中，您可以使用您的 Azure Active Directory 驗證權杖來登入 AKS 叢集。 此外，叢集系統管理員能夠根據使用者身分識別或目錄群組成員資格，設定 Kubernetes 角色型存取控制 (RBAC)。

本文說明如何部署 AKS 和 Azure AD 的先決條件，然後說明如何部署已啟用 Azure AD 的叢集，以及在 AKS 叢集中建立簡單的 RBAC 角色。

套用下列限制：

- 目前無法更新現有之已啟用非 RBAC 的 AKS 叢集供 RBAC 使用。
- 不支援 Azure AD 中的來賓使用者 (例如，如果您從不同的目錄使用同盟登入)。

## <a name="authentication-details"></a>驗證詳細資料

透過 OpenID Connect 對 AKS 叢集提供 Azure AD 驗證。 OpenID Connect 是以 OAuth 2.0 通訊協定為建置基礎的身分識別層。 如需 OpenID Connect 的詳細資訊，請參閱 [OpenID Connect 文件][open-id-connect]。

從 Kubernetes 叢集內部，Webhook 權杖驗證用來確認驗證權杖。 Webhook 權杖驗證已設定並當作 AKS 叢集的一部分管理。 如需 Webhook 權杖驗證的詳細資訊，請參閱 [Webhook 驗證文件][kubernetes-webhook]。

> [!NOTE]
> 設定 Azure AD 進行 AKS 驗證時，會設定兩個 Azure AD 應用程式。 這項作業必須由 Azure 租用戶系統管理員完成。

## <a name="create-server-application"></a>建立伺服器應用程式

第一個 Azure AD 應用程式用來取得使用者的 Azure AD 群組成員資格。

1. 選取 [Azure Active Directory] > [應用程式註冊] > [新應用程式註冊]。

  提供應用程式名稱，針對應用程式類型選取 [Web 應用程式 / API]，然後針對 [登入 URL] 輸入任何 URI 格式化值。 完成後選取 [建立]。

  ![建立 Azure AD 註冊](media/aad-integration/app-registration.png)

2. 選取 [資訊清單]，並將 `groupMembershipClaims` 值編輯為 `"All"`。

  完成後儲存更新。

  ![將群組成員資格更新為全部](media/aad-integration/edit-manifest.png)

3. 回到 Azure AD 應用程式，選取 [建立] > [金鑰]。

  新增金鑰描述，選取到期期限，然後選取 [儲存]。 記下金鑰值。 部署已啟用 Azure AD 的 AKS 叢集時，這個值就是指 `Server application secret`。

  ![取得應用程式私密金鑰](media/aad-integration/application-key.png)

4. 返回 Azure AD 應用程式，選取 [設定] > [必要的權限] > [新增] > [選取 API] > [Microsoft Graph] > [選取]。

  ![請選取圖表 API](media/aad-integration/graph-api.png)

5. 在 [應用程式權限] 之下 [讀取目錄資料] 的旁邊標上核取記號。

  ![設定應用程式圖表權限](media/aad-integration/read-directory.png)

6. 在 [委派的權限] 之下 [登入並讀取使用者設定檔] 和 [讀取目錄資料] 的旁邊標上核取記號。 完成後儲存更新。

  ![設定應用程式圖表權限](media/aad-integration/delegated-permissions.png)

7. 選取 [完成]，從 API 清單中選擇 [Microsoft Graph]，然後選取 [授與權限]。 如果目前的帳戶不是租用戶系統管理員，此步驟將會失敗。

  ![設定應用程式圖表權限](media/aad-integration/grant-permissions.png)

  成功授與權限時，入口網站會顯示下列通知：

  ![成功授與權限的通知](media/aad-integration/permissions-granted.png)

8. 返回應用程式並記下 [應用程式識別碼]。 部署已啟用 Azure AD 的 AKS 叢集時，這個值就是指 `Server application ID`。

  ![取得應用程式識別碼](media/aad-integration/application-id.png)

## <a name="create-client-application"></a>建立用戶端應用程式

使用 Kubernetes CLI (kubectl) 登入時會使用第二個 Azure AD 應用程式。

1. 選取 [Azure Active Directory] > [應用程式註冊] > [新應用程式註冊]。

  提供應用程式名稱，針對應用程式類型選取 [原生]，然後針對 [重新導向 URI] 輸入任何 URI 格式化值。 完成後選取 [建立]。

  ![建立 AAD 註冊](media/aad-integration/app-registration-client.png)

2. 從 Azure AD 應用程式，選取 [設定] > [必要的權限] > [新增] > [選取 API]，並搜尋在本文件最後一個步驟中建立的伺服器應用程式名稱。

  ![設定應用程式權限](media/aad-integration/select-api.png)

3. 在應用程式的旁邊標上核取記號，然後按一下 [選取]。

  ![選取 AKS AAD 伺服器應用程式端點](media/aad-integration/select-server-app.png)

4. 選取 [完成] 和 [授與權限] 來完成此步驟。

  ![授與權限](media/aad-integration/grant-permissions-client.png)

5. 回到 AD 應用程式，記下 [應用程式識別碼]。 部署已啟用 Azure AD 的 AKS 叢集時，這個值就是指 `Client application ID`。

  ![取得應用程式識別碼](media/aad-integration/application-id-client.png)

## <a name="get-tenant-id"></a>取得租用戶識別碼

最後，取得您的 Azure 租用戶識別碼。 這個值也會在部署 AKS 叢集時使用。

從 Azure 入口網站，選取 [Azure Active Directory] > [屬性]，並記下 [目錄識別碼]。 部署已啟用 Azure AD 的 AKS 叢集時，這個值就是指 `Tenant ID`。

![取得 Azure 租用戶識別碼](media/aad-integration/tenant-id.png)

## <a name="deploy-cluster"></a>部署叢集

使用 [az group create][az-group-create] 命令來建立 AKS 叢集的資源群組。

```azurecli
az group create --name myAKSCluster --location eastus
```

使用 [az aks create][az-aks-create] 命令來部署叢集。 以建立 Azure AD 應用程式時收集的值，取代以下範例命令中的值。

```azurecli
az aks create --resource-group myAKSCluster --name myAKSCluster --generate-ssh-keys --enable-rbac \
  --aad-server-app-id 7ee598bb-0000-0000-0000-83692e2d717e \
  --aad-server-app-secret wHYomLe2i1mHR2B3/d4sFrooHwADZccKwfoQwK2QHg= \
  --aad-client-app-id 7ee598bb-0000-0000-0000-83692e2d717e \
  --aad-tenant-id 72f988bf-0000-0000-0000-2d7cd011db47
```

## <a name="create-rbac-binding"></a>建立 RBAC 繫結

必須先建立角色繫結或叢集角色繫結，Azure Active Directory 帳戶才能搭配 AKS 叢集使用。 「角色」會定義要授與的權限，而「繫結」會將角色套用至需要的使用者。 這些指派可以套用至指定的命名空間或在整個叢集中套用。 如需詳細資訊，請參閱[使用 RBAC 授權][rbac-authorization]。

首先，使用 [az aks get-credentials][az-aks-get-credentials] 命令搭配 `--admin` 引數，透過系統管理員存取權登入叢集。

```azurecli
az aks get-credentials --resource-group myAKSCluster --name myAKSCluster --admin
```

接下來，使用下列資訊清單來建立 Azure AD 帳戶的 ClusterRoleBinding。 以 Azure AD 租用戶中的使用者名稱來更新使用者名稱。 此範例讓帳戶可以完整存取叢集的所有命名空間：

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: contoso-cluster-admins
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: "user@contoso.com"
```

此外，也可以為 Azure AD 群組的所有成員建立角色繫結。 Azure AD 群組是使用群組物件識別碼所指定的，如下列範例所示：

 ```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: contoso-cluster-admins
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- apiGroup: rbac.authorization.k8s.io
   kind: Group
   name: "894656e1-39f8-4bfe-b16a-510f61af6f41"
```

如需有關使用 RBAC 保護 Kubernetes 叢集的詳細資訊，請參閱[使用 RBAC 授權][rbac-authorization]。

## <a name="access-cluster-with-azure-ad"></a>透過 Azure AD 存取叢集

接下來，使用 [az aks get-credentials][az-aks-get-credentials] 命令，提取非系統管理員使用者的內容。

```azurecli
az aks get-credentials --resource-group myAKSCluster --name myAKSCluster
```

執行任何 kubectl 命令之後，系統會提示您向 Azure 進行驗證。 依照螢幕上的指示進行。

```console
$ kubectl get nodes

To sign in, use a web browser to open the page https://microsoft.com/devicelogin and enter the code BUJHWDGNL to authenticate.

NAME                       STATUS    ROLES     AGE       VERSION
aks-nodepool1-42032720-0   Ready     agent     1h        v1.9.6
aks-nodepool1-42032720-1   Ready     agent     1h        v1.9.6
aks-nodepool1-42032720-2   Ready     agent     1h        v1.9.6
```

完成後，就會快取驗證權杖。 當權杖已過期或重新建立 Kubernetes 組態檔時，系統才會重新提示您登入。

如果成功登入後看到授權錯誤訊息，請檢查您在 Azure AD 中登入的使用者身分不是「來賓」(如果您使用來自其他目錄中的同盟登入，則通常會出現這種情況)。
```console
error: You must be logged in to the server (Unauthorized)
```


## <a name="next-steps"></a>後續步驟

請參閱[使用 RBAC 授權][rbac-authorization]文件，深入了解如何使用 RBAC 保護 Kubernetes 叢集。

<!-- LINKS - external -->
[kubernetes-webhook]:https://kubernetes.io/docs/reference/access-authn-authz/authentication/#webhook-token-authentication
[rbac-authorization]: https://kubernetes.io/docs/reference/access-authn-authz/rbac/

<!-- LINKS - internal -->
[az-aks-create]: /cli/azure/aks?view=azure-cli-latest#az-aks-create
[az-aks-get-credentials]: /cli/azure/aks?view=azure-cli-latest#az-aks-get-credentials
[az-group-create]: /cli/azure/group#az-group-create
[open-id-connect]:../active-directory/develop/v1-protocols-openid-connect-code.md
