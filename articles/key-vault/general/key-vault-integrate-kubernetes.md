---
title: 整合 Azure Key Vault 與 Kubernetes
description: 在本教學課程中，您將使用祕密存放區容器儲存體介面 (CSI) 驅動程式從 Azure 金鑰保存庫存取及擷取祕密，以便掛接至 Kubernetes Pod。
author: msmbaldwin
ms.author: mbaldwin
ms.service: key-vault
ms.subservice: general
ms.topic: tutorial
ms.date: 09/25/2020
ms.openlocfilehash: f0699ed065da4c63bc88945d75a866abcfbb9053
ms.sourcegitcommit: aacbf77e4e40266e497b6073679642d97d110cda
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/12/2021
ms.locfileid: "98121357"
---
# <a name="tutorial-configure-and-run-the-azure-key-vault-provider-for-the-secrets-store-csi-driver-on-kubernetes"></a>教學課程：在 Kubernetes 上，為祕密存放區 CSI 驅動程式設定及執行 Azure Key Vault 提供者

> [!IMPORTANT]
> CSI Driver 是一項開放原始碼專案，不受 Azure 技術支援。 請在[這裡](https://github.com/Azure/secrets-store-csi-driver-provider-azure/issues)的 github 連結中，回報有關 CSI Driver Key Vault 整合的所有意見反應和問題。 此工具可供使用者自行安裝到叢集，並從社群收集意見反應。


在本教學課程中，您將使用祕密存放區容器儲存體介面 (CSI) 驅動程式從 Azure 金鑰保存庫存取及擷取祕密，以便將祕密掛接至 Kubernetes Pod。

在本教學課程中，您會了解如何：

> [!div class="checklist"]
> * 使用受控識別。
> * 使用 Azure CLI 部署 Azure Kubernetes Service (AKS) 叢集。
> * 安裝 Helm 與祕密存放區 CSI 驅動程式。
> * 建立 Azure 金鑰保存庫並設定祕密。
> * 建立您自己的 SecretProviderClass 物件。
> * 使用從金鑰保存庫掛接而來的祕密部署您的 Pod。

## <a name="prerequisites"></a>必要條件

* 如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

* 開始本教學課程之前，請先安裝 [Azure CLI](/cli/azure/install-azure-cli-windows?view=azure-cli-latest)。

## <a name="use-managed-identities"></a>使用受控識別

下圖說明受控識別的 AKS–Key Vault 整合流程：

![圖表說明受控識別的 AKS–Key Vault 整合流程](../media/aks-key-vault-integration-flow.png)

## <a name="deploy-an-azure-kubernetes-service-aks-cluster-by-using-the-azure-cli"></a>使用 Azure CLI 部署 Azure Kubernetes Service (AKS) 叢集

您不需要使用 Azure Cloud Shell。 已安裝 Azure CLI 的命令提示字元 (終端機) 就已足夠。 

完成[使用 Azure CLI 部署 Azure Kubernetes Service 叢集](../../aks/kubernetes-walkthrough.md)中的＜建立資源群組＞、＜建立 AKS 叢集＞和＜連線到叢集＞等節。 

> [!NOTE] 
> 如果您打算使用 Pod 身分識別，請務必在建立 Kubernetes 叢集時加以啟用，如下列命令所示：
>
> ```azurecli
> az aks create -n contosoAKSCluster -g contosoResourceGroup --kubernetes-version 1.16.9 --node-count 1 --enable-managed-identity
> ```

1. [將 PATH 環境變數設定](https://www.java.com/en/download/help/path.xml)為所下載的 *kubectl.exe* 檔案。
1. 使用下列命令來檢查您的 Kubernetes 版本，這會輸出用戶端和伺服器版本。 用戶端版本是您安裝的 *kubectl.exe* 檔案，而伺服器版本則是叢集執行所在的 Azure Kubernetes Services (AKS)。
    ```azurecli
    kubectl version
    ```
1. 確定您的 Kubernetes 版本是 1.16.0 或更新版本。 若為 Windows 叢集，請確定您的 Kubernetes 版本是 1.18.0 或更新版本。 下列命令會同時升級 Kubernetes 叢集和節點集區。 命令可能需要幾分鐘的時間來執行。 在此範例中，資源群組為 *contosoResourceGroup*，Kubernetes 叢集為 *contosoAKSCluster*。
    ```azurecli
    az aks upgrade --kubernetes-version 1.16.9 --name contosoAKSCluster --resource-group contosoResourceGroup
    ```
1. 若要顯示所建立 AKS 叢集的中繼資料，請使用下列命令。 複製 **principalId**、**clientId**、**subscriptionId** 與 **nodeResourceGroup** 以供稍後使用。 如果未建立啟用受控識別的 AKS 叢集，則 **principalId** 和 **clientId** 會是 Null。 

    ```azurecli
    az aks show --name contosoAKSCluster --resource-group contosoResourceGroup
    ```

    所顯示的輸出會醒目提示兩個參數：
    
    ![Azure CLI 的螢幕擷取畫面，其中醒目提示了 principalId 和 clientId 值](../media/kubernetes-key-vault-2.png) ![Azure CLI 的螢幕擷取畫面，其中醒目提示了 subscriptionId 和 nodeResourceGroup 值](../media/kubernetes-key-vault-3.png)
    
## <a name="install-helm-and-the-secrets-store-csi-driver"></a>安裝 Helm 與祕密存放區 CSI 驅動程式

若要安裝祕密存放區 CSI 驅動程式，您必須先安裝 [Helm](https://helm.sh/docs/intro/install/)。

透過[秘密存放區 CSI](https://github.com/Azure/secrets-store-csi-driver-provider-azure/blob/master/charts/csi-secrets-store-provider-azure/README.md) 驅動程式介面，您可以取得儲存在 Azure 金鑰保存庫執行個體中的秘密，然後使用驅動程式介面將秘密內容掛接到 Kubernetes Pod。

1. 檢查以確定 Helm 版本為 v3 或更新版本：
    ```azurecli
    helm version
    ```
1. 安裝密碼存放區 CSI 驅動程式及其 Azure Key Vault 提供者：
    ```azurecli
    helm repo add csi-secrets-store-provider-azure https://raw.githubusercontent.com/Azure/secrets-store-csi-driver-provider-azure/master/charts

    helm install csi-secrets-store-provider-azure/csi-secrets-store-provider-azure --generate-name
    ```

## <a name="create-an-azure-key-vault-and-set-your-secrets"></a>建立 Azure 金鑰保存庫並設定祕密

若要建立您自己的金鑰保存庫並設定祕密，請遵循 [使用 Azure CLI 從 Azure Key Vault 設定和擷取秘密](../secrets/quick-create-cli.md)中的指示。

> [!NOTE] 
> 您無須使用 Azure Cloud Shell，也無須建立新的資源群組， 您可以使用先前為 Kubernetes 叢集建立的資源群組。

## <a name="create-your-own-secretproviderclass-object"></a>建立您自己的 SecretProviderClass 物件

若要使用祕密存放區 CSI 驅動程式的提供者專用參數來建立您自己的自訂 SecretProviderClass 物件，請[使用此範本](https://github.com/Azure/secrets-store-csi-driver-provider-azure/blob/master/examples/v1alpha1_secretproviderclass_service_principal.yaml)。 此物件將提供存取您金鑰保存庫所需的身分識別。

在範例 SecretProviderClass YAML 檔案中，填入遺漏的參數。 必要參數如下：

* **userAssignedIdentityID**: # [REQUIRED] 如果值是空的，則預設為在 VM 上使用系統指派的身分識別 
* **keyvaultName**：金鑰保存庫的名稱
* **objects**：您想要掛接的所有秘密內容所屬的容器
    * **objectName**：秘密內容的名稱
    * **objectType**：物件類型 (祕密、金鑰、憑證)
* **resourceGroup**：資源群組的名稱  # [版本 < 0.0.4 為必要] KeyVault 的資源群組
* **subscriptionId**：金鑰保存庫的訂用帳戶識別碼 # [版本 < 0.0.4 為必要] KeyVault 的訂用帳戶識別碼
* **tenantID**：金鑰保存庫的租用戶識別碼 (或目錄識別碼)

所有必要欄位的文件可在這裡取得：[連結](https://github.com/Azure/secrets-store-csi-driver-provider-azure#create-a-new-azure-key-vault-resource-or-use-an-existing-one)

已更新的範本會顯示在下列程式碼中。 請將其下載為 YAML 檔案，並填入必要欄位。 在此範例中，金鑰保存庫是 **contosoKeyVault5**。 其具有兩個秘密，分別是 **secret1** 和 **secret2**。

> [!NOTE] 
> 如果您要使用受控識別，請將 **usePodIdentity** 值設定為 *true*，並將 **userAssignedIdentityID** 值設定為一對引號 ( **""** )。 

```yaml
apiVersion: secrets-store.csi.x-k8s.io/v1alpha1
kind: SecretProviderClass
metadata:
  name: azure-kvname
spec:
  provider: azure
  parameters:
    usePodIdentity: "false"                   # [REQUIRED] Set to "true" if using managed identities
    useVMManagedIdentity: "false"             # [OPTIONAL] if not provided, will default to "false"
    userAssignedIdentityID: "servicePrincipalClientID"       # [REQUIRED]  If you're using a user-assigned identity as the VM's managed identity, specify the identity's client id. If the value is empty, it defaults to use the system-assigned identity on the VM
                                                         
    keyvaultName: "contosoKeyVault5"          # [REQUIRED] the name of the key vault
                                              #     az keyvault show --name contosoKeyVault5
                                              #     the preceding command will display the key vault metadata, which includes the subscription ID, resource group name, key vault 
    cloudName: ""                                # [OPTIONAL for Azure] if not provided, Azure environment will default to AzurePublicCloud
    objects:  |
      array:
        - |
          objectName: secret1                 # [REQUIRED] object name
                                              #     az keyvault secret list --vault-name "contosoKeyVault5"
                                              #     the above command will display a list of secret names from your key vault
          objectType: secret                  # [REQUIRED] object types: secret, key, or cert
          objectVersion: ""                   # [OPTIONAL] object versions, default to latest if empty
        - |
          objectName: secret2
          objectType: secret
          objectVersion: ""
    resourceGroup: "contosoResourceGroup"     # [REQUIRED] the resource group name of the key vault
    subscriptionId: "subscriptionID"          # [REQUIRED] the subscription ID of the key vault
    tenantId: "tenantID"                      # [REQUIRED] the tenant ID of the key vault
```
下圖顯示 **az keyvault show --name contosoKeyVault5** 的主控台輸出，以及已醒目提示的相關中繼資料：

![顯示 "az keyvault show --name contosoKeyVault5" 主控台輸出的螢幕擷取畫面](../media/kubernetes-key-vault-4.png)

## <a name="assign-managed-identity"></a>指派受控識別

請為您建立的 AKS 叢集，指派特定的角色。 

1. 若要建立、列出或讀取使用者指派的受控識別，您必須為 AKS 叢集指派[受控識別操作員](../../role-based-access-control/built-in-roles.md#managed-identity-operator)角色。 請確定 **$clientId** 是 Kubernetes 叢集的 clientId。 至於範圍，其會在您的 Azure 訂用帳戶服務下，特別是建立 AKS 叢集時所建立的節點資源群組。 此範圍會確保只有該群組內的資源會受到下列指派角色所影響。 

    ```azurecli
    RESOURCE_GROUP=contosoResourceGroup
    
    az role assignment create --role "Managed Identity Operator" --assignee $clientId --scope /subscriptions/<SUBID>/resourcegroups/$RESOURCE_GROUP
    
    az role assignment create --role "Virtual Machine Contributor" --assignee $clientId --scope /subscriptions/<SUBID>/resourcegroups/$RESOURCE_GROUP
    ```

1. 為 AKS 安裝 Azure Active Directory (Azure AD) 身分識別。
    ```azurecli
    helm repo add aad-pod-identity https://raw.githubusercontent.com/Azure/aad-pod-identity/master/charts

    helm install pod-identity aad-pod-identity/aad-pod-identity
    ```

1. 建立 Azure AD 身分識別。 在輸出中，複製 **clientId** 和 **principalId** 以供稍後使用。
    ```azurecli
    az identity create -g $resourceGroupName -n $identityName
    ```

1. 將 *讀者* 角色指派給您在上一個步驟中為金鑰保存庫建立的 Azure AD 身分識別，然後向身分識別授與可從金鑰保存庫取得秘密的權限。 使用 Azure AD 身分識別中的 **clientId** 和 **principalId**。
    ```azurecli
    az role assignment create --role "Reader" --assignee $principalId --scope /subscriptions/<SUBID>/resourceGroups/contosoResourceGroup/providers/Microsoft.KeyVault/vaults/contosoKeyVault5

    az keyvault set-policy -n contosoKeyVault5 --secret-permissions get --spn $clientId
    az keyvault set-policy -n contosoKeyVault5 --key-permissions get --spn $clientId
    ```

## <a name="deploy-your-pod-with-mounted-secrets-from-your-key-vault"></a>使用從金鑰保存庫掛接而來的祕密部署您的 Pod

若要設定您的 SecretProviderClass 物件，請執行下列命令：
```azurecli
kubectl apply -f secretProviderClass.yaml
```

### <a name="use-managed-identities"></a>使用受控識別

如果您使用的是受控識別，請在叢集中建立會參考稍早所建立身分識別的 *AzureIdentity*。 接著，請建立會參考所建立 AzureIdentity 的 *AzureIdentityBinding*。 填寫下列範本中的參數，然後將其儲存為 *podIdentityAndBinding.yaml*。  

```yml
apiVersion: aadpodidentity.k8s.io/v1
kind: AzureIdentity
metadata:
    name: "azureIdentityName"               # The name of your Azure identity
spec:
    type: 0                                 # Set type: 0 for managed service identity
    resourceID: /subscriptions/<SUBID>/resourcegroups/<RESOURCEGROUP>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<AZUREIDENTITYNAME>
    clientID: "managedIdentityClientId"     # The clientId of the Azure AD identity that you created earlier
---
apiVersion: aadpodidentity.k8s.io/v1
kind: AzureIdentityBinding
metadata:
    name: azure-pod-identity-binding
spec:
    azureIdentity: "azureIdentityName"      # The name of your Azure identity
    selector: azure-pod-identity-binding-selector
```
    
執行下列命令執行繫結：

```azurecli
kubectl apply -f podIdentityAndBinding.yaml
```

接下來，您要部署 Pod。 下列程式碼是部署 YAML 檔案，其會使用上一個步驟中的 Pod 身分識別繫結。 將此檔案儲存為 *podBindingDeployment.yaml*。

```yml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-secrets-store-inline
  labels:
    aadpodidbinding: azure-pod-identity-binding-selector
spec:
  containers:
    - name: nginx
      image: nginx
      volumeMounts:
        - name: secrets-store-inline
          mountPath: "/mnt/secrets-store"
          readOnly: true
  volumes:
    - name: secrets-store-inline
      csi:
        driver: secrets-store.csi.k8s.io
        readOnly: true
        volumeAttributes:
          secretProviderClass: azure-kvname
```

執行下列命令，以部署您的 Pod：

```azurecli
kubectl apply -f podBindingDeployment.yaml
```

### <a name="check-the-pod-status-and-secret-content"></a>檢查 Pod 狀態和祕密內容 

若要顯示您已部署的 Pod，請執行下列命令：
```azurecli
kubectl get pods
```

若要檢查 Pod 的狀態，請執行下列命令：
```azurecli
kubectl describe pod/nginx-secrets-store-inline
```

![Azure CLI 輸出的螢幕擷取畫面，其中顯示 Pod 的「執行中」狀態，且所有事件均顯示為「正常」 ](../media/kubernetes-key-vault-6.png)

在輸出視窗中，已部署的 Pod 應該會處於 *執行中* 狀態。 在底部的 [事件] 區段中，所有事件類型均顯示為 *正常*。

確認 Pod 正在執行之後，您可以確認 Pod 是否包含金鑰保存庫中的秘密。

若要顯示 Pod 中包含的所有秘密，請執行下列命令：
```azurecli
kubectl exec -it nginx-secrets-store-inline -- ls /mnt/secrets-store/
```

若要顯示特定祕密的內容，請執行下列命令：
```azurecli
kubectl exec -it nginx-secrets-store-inline -- cat /mnt/secrets-store/secret1
```

確認已顯示祕密的內容。

## <a name="next-steps"></a>後續步驟

若要協助確保金鑰保存庫可以復原，請參閱：
> [!div class="nextstepaction"]
> [開啟虛刪除](./key-vault-recovery.md)
