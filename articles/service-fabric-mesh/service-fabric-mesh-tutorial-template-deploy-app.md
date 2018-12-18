---
title: 教學課程 - 將應用程式部署到 Azure Service Fabric Mesh | Microsoft Docs
description: 在本教學課程中，您會了解如何使用範本將應用程式部署到 Service Fabric Mesh。
services: service-fabric-mesh
documentationcenter: .net
author: rwike77
manager: jeconnoc
editor: ''
ms.assetid: ''
ms.service: service-fabric-mesh
ms.devlang: dotNet
ms.topic: tutorial
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 09/18/2018
ms.author: ryanwi
ms.custom: mvc, devcenter
ms.openlocfilehash: cca18b2aa5cb6f27df45e4b63e55251bea058625
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46968844"
---
# <a name="tutorial-deploy-an-application-to-service-fabric-mesh-using-a-template"></a>教學課程：使用範本將應用程式部署到 Service Fabric Mesh

本教學課程是一個系列的第一部分。 您將了解如何使用範本部署 Azure Service Fabric Mesh 應用程式。  應用程式是由 ASP.NET Web 前端服務和 ASP.NET Core Web API 後端服務所組成，這些服務可在 Docker Hub 中找到。  從 Docker Hub 提取兩個容器映像，然後將其推送到您自己的私人登錄。 接著，為應用程式建立 Azure RM 範本，並將容器登錄的應用程式部署到 Service Fabric Mesh。 完成時，您將會擁有一個在 Service Fabric Mesh 中執行之簡單的待辦事項清單應用程式。

在系列的第一部分中，您將了解如何：

> [!div class="checklist"]
> * 建立私人 Azure Container Registry 執行個體
> * 將容器映像推送至登錄
> * 建立 RM 範本和參數檔案
> * 將應用程式部署到 Service Fabric Mesh

在本教學課程系列中，您將了解如何：
> [!div class="checklist"]
> * 使用範本將應用程式部署到 Service Fabric Mesh
> * [調整 Service Fabric Mesh 上所執行應用程式中的服務](service-fabric-mesh-tutorial-template-scale-services.md)
> * [升級 Service Fabric Mesh 上所執行的應用程式](service-fabric-mesh-tutorial-template-upgrade-app.md)
> * [移除應用程式](service-fabric-mesh-tutorial-template-remove-app.md)

[!INCLUDE [preview note](./includes/include-preview-note.md)]

## <a name="prerequisites"></a>必要條件

開始進行本教學課程之前：

* 如果您沒有 Azure 訂用帳戶，您可以在開始前[建立免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

* [安裝 Docker](service-fabric-mesh-howto-setup-developer-environment-sdk.md#install-docker)

* [本機安裝 Azure CLI 和 Service Fabric Mesh CLI](service-fabric-mesh-howto-setup-cli.md#install-the-service-fabric-mesh-cli-locally)。

## <a name="create-a-container-registry"></a>建立容器登錄庫

與 Service Fabric Mesh 應用程式中的服務相關聯的容器映像必須儲存在容器登錄中。  本教學課程使用的是私人 Azure Container Registry (ACR) 執行個體。 

請使用下列步驟來建立 ACR 執行個體。  如果您已經設定 ACR 執行個體，可以跳過此步驟。

### <a name="sign-in-to-azure"></a>登入 Azure

登入 Azure，並設定使用中的訂用帳戶。

```azurecli
az login
az account set --subscription "<subscriptionName>"
```

### <a name="create-a-resource-group"></a>建立資源群組

Azure 資源群組是在其中部署與管理 Azure 資源的邏輯容器。 使用下列命令，在 *eastus* 位置中，建立名為 *myResourceGroup* 的資源群組。

```azurecli
az group create --name myResourceGroup --location eastus
```

### <a name="create-the-container-registry"></a>建立容器登錄

使用 `az acr create` 命令建立 ACR 執行個體。 登錄名稱在 Azure 內必須是唯一的，且包含 5-50 個英數字元。 下列範例中使用名稱 *myContainerRegistry*。 如果您收到錯誤指出登錄名稱已在使用中，請選擇其他名稱。

```azurecli
az acr create --resource-group myResourceGroup --name myContainerRegistry --sku Basic
```

建立登錄後，您將看到輸出如下：

```json
{
  "adminUserEnabled": false,
  "creationDate": "2018-09-13T19:43:57.388203+00:00",
  "id": "/subscriptions/<subscription>/resourceGroups/myResourceGroup/providers/Microsoft.ContainerRegistry/registries/myContainerRegistry",
  "location": "eastus",
  "loginServer": "mycontainerregistry.azurecr.io",
  "name": "myContainerRegistry",
  "provisioningState": "Succeeded",
  "resourceGroup": "myResourceGroup",
  "sku": {
    "name": "Basic",
    "tier": "Basic"
  },
  "status": null,
  "storageAccount": null,
  "tags": {},
  "type": "Microsoft.ContainerRegistry/registries"
}
```

## <a name="push-the-images-to-azure-container-registry"></a>將映像推送至 Azure Container Registry

本教學課程使用待辦事項清單範例應用程式作為範例。  [WebFrontEnd](https://hub.docker.com/r/seabreeze/azure-mesh-todo-webfrontend/) 與 [ToDoService](https://hub.docker.com/r/seabreeze/azure-mesh-todo-service/) 服務的容器映像可在 Docker Hub 中找到。 如需如何在 Visual Studio 中建立應用程式的相關資訊，請參閱[建立 Servic Fabric Mesh Web 應用程式](service-fabric-mesh-tutorial-create-dotnetcore.md)。 Service Fabric Mesh 可以執行 Windows 或 Linux Docker 容器。  如果您正在使用 Linux 容器，請在 Docker 中選取**切換至 Linux 容器**。  如果您正在使用 Windows 容器，請在 Docker 中選取**切換至 Windows 容器**。

若要推送映像到 ACR 執行個體，您必須先有容器映像。 如果您還沒有任何本機容器映像，請使用 [docker pull](https://docs.docker.com/engine/reference/commandline/pull/) 命令，以從 Docker Hub 中提取 [WebFrontEnd](https://hub.docker.com/r/seabreeze/azure-mesh-todo-webfrontend/) 與 [ToDoService](https://hub.docker.com/r/seabreeze/azure-mesh-todo-service/) 映像。

推送 Windows 映像：

```bash
docker pull seabreeze/azure-mesh-todo-webfrontend:1.0-nanoserver-1709
docker pull seabreeze/azure-mesh-todo-service:1.0-nanoserver-1709
```

您必須使用 ACR 登入伺服器的完整名稱來標記映像，才能將映像推送至您的登錄。

執行下列命令，取得 ACR 執行個體的完整登入伺服器名稱。

```azurecli
az acr list --resource-group myResourceGroup --query "[].{acrLoginServer:loginServer}" --output table

AcrLoginServer
---------------------------------
mycontainerregistry.azurecr.io
```

接下來，使用 [docker tag](https://docs.docker.com/engine/reference/commandline/tag/) 命令來標記 Docker 映像。 下列範例會標記 seabreeze/azure-mesh-todo-service:1.0-nanoserver-1709 與 seabreeze/azure-mesh-todo-webfrontend:1.0-nanoserver-1709 映像。 如果使用不同的映像，請使用下列命令更換這些映像名稱。

```bash
docker tag seabreeze/azure-mesh-todo-webfrontend:1.0-nanoserver-1709 mycontainerregistry.azurecr.io/seabreeze/azure-mesh-todo-webfrontend:1.0-nanoserver-1709
docker tag seabreeze/azure-mesh-todo-service:1.0-nanoserver-1709 mycontainerregistry.azurecr.io/seabreeze/azure-mesh-todo-service:1.0-nanoserver-1709
```

登入 Azure Container Registry。

```azurecli
az acr login -n myContainerRegistry
```

使用 [docker push](https://docs.docker.com/engine/reference/commandline/push/) 命令，將映像推送到 ACR 執行個體：

```bash
docker push mycontainerregistry.azurecr.io/seabreeze/azure-mesh-todo-webfrontend:1.0-nanoserver-1709
docker push mycontainerregistry.azurecr.io/seabreeze/azure-mesh-todo-service:1.0-nanoserver-1709
```

### <a name="list-container-images"></a>列出容器映像

若要查看 ACR 執行個體中的存放庫，請執行下列命令：

```azurecli
az acr repository list --name myContainerRegistry --output table

Result
-------------------------------
seabreeze/azure-mesh-todo-webfrontend
seabreeze/azure-mesh-todo-service
```

以下範例會列出在 **azure-mesh-todo-service** 存放庫上的標籤。

```azurecli
az acr repository show-tags --name myContainerRegistry --repository seabreeze/azure-mesh-todo-service --output table

Result
--------
1.0-nanoserver-1709
```

上述輸出會確認私人容器登錄中的 `azure-mesh-todo-service:1.0-nanoserver-1709` 是否存在。  此外，請確認是否存在 `azure-mesh-todo-webfrontend:1.0-nanoserver-1709`。

## <a name="retrieve-credentials-for-the-registry"></a>擷取登錄的認證

> [!IMPORTANT]
> 建議不要在生產案例中啟用 ACR 執行個體上的管理使用者。 此處做法是為了方便起見。 對於生產案例，請在生產案例中使用[服務主體](https://docs.microsoft.com/azure/container-registry/container-registry-auth-service-principal)，以進行使用者與系統驗證。

若要使用範本從已建立的登錄中部署容器執行個體，必須在部署期間提供登錄認證。 首先，使用下列命令，啟用您登錄上的管理使用者：

```azurecli
az acr update --name myContainerRegistry --admin-enabled true
```

接著，使用下列命令，取得登錄的登入伺服器名稱、使用者名稱和密碼：

```azurecli
az acr list --resource-group myResourceGroup --query "[].{acrLoginServer:loginServer}" --output table
az acr credential show --name myContainerRegistry --query username
az acr credential show --name myContainerRegistry --query "passwords[0].value"
```

在下一節中建立 RM 範本和參數檔案時，請使用傳回的 ACR 登入伺服器名稱、使用者名稱和密碼值。

## <a name="download-and-explore-the-template-and-parameters-files"></a>下載並瀏覽範本及參數檔案

Service Fabric Mesh 應用程式是一種 Azure 資源，您可以使用 Azure Resource Manager (RM) 範本部署和管理。 如果您不熟悉部署和管理 Azure 解決方案的相關概念，請參閱 [Azure Resource Manager 概觀](/azure/azure-resource-manager/resource-group-overview)與[了解 RM 範本的結構和語法](/azure/azure-resource-manager/resource-group-authoring-templates)。

本教學課程使用待辦事項清單範例作為範例。  不會建立新的範本和參數檔案，而是下載 [mesh_rp.windows.json 部署範本](https://github.com/Azure-Samples/service-fabric-mesh/blob/master/templates/todolist/mesh_rp.windows.json)與 [mesh_rp.windows.parameter.json 參數](https://github.com/Azure-Samples/service-fabric-mesh/blob/master/templates/todolist/mesh_rp.windows.parameters.json)檔案。

### <a name="parameters"></a>參數
當您範本中的值是您要在部署時變更的值，或希望可以選擇在每個部署基礎上進行變更的值 (如果您打算將此範本重新用於其他部署)，最佳做法是將值參數化。 執行此操作的正確方法是在部署範本的頂端建立「參數」區段，您可以在該區段中指定參數名稱和屬性，以供之後在部署範本中引用。 每個參數定義包括 *type*、*defaultValue*，以及具有*描述*之選用性*中繼資料*區段。

參數區段定義於部署範本的頂端，就在*資源*區段之前：

```json
{
    "$schema": "http://schema.management.azure.com/schemas/2014-04-01-preview/deploymentTemplate.json",
    "contentVersion": "1.0.0.0",
    "parameters": {
      ...
    },
    "resources": [
```

在 [mesh_rp.windows.json 部署範本](https://github.com/Azure-Samples/service-fabric-mesh/blob/master/templates/todolist/mesh_rp.windows.json)中，宣告下列參數：location、registryPassword、registryUserName、registryServer、frontEndImage、serviceImage、frontEndCpu、serviceCpu、frontEndMemory、serviceMemory、frontEndReplicaCount、serviceReplicaCount。  在部署範本檔案中，可以找到每個參數的描述。

這些參數用於 [mesh_rp.windows.parameter.json 參數](https://github.com/Azure-Samples/service-fabric-mesh/blob/master/templates/todolist/mesh_rp.windows.parameters.json)檔案。  使用不同的參數檔案，可讓您變更部署之間的參數值，而不需要更新部署範本本身。

### <a name="overview-of-the-application-and-services"></a>應用程式與服務概觀

服務在範本中指定為應用程式資源的屬性。  應用程式會部署到私人網路，宣告為範本中的資源。  服務可以使用磁碟區以保存在範本中宣告為資源的資料。  對於每個服務，作業系統類型、程式碼封裝、複本數目，以及網路皆指定為服務的屬性。  對於每個程式碼封裝，指定容器映像、端點、記憶體和 CPU 資源，以及映像存放庫認證。 整體而言，Service Fabric Mesh 應用程式與多個服務的範本所示如下：

```json
{
  "$schema": "http://schema.management.azure.com/schemas/2014-04-01-preview/deploymentTemplate.json",
  "contentVersion": "1.0.0.0",
  "parameters": {
    ...
  },
  "resources": [
    {
      "apiVersion": "2018-07-01-preview",
      "name": "MyMeshApplication",
      "type": "Microsoft.ServiceFabricMesh/applications",
      "location": "[parameters('location')]",
      "dependsOn": [
        "Microsoft.ServiceFabricMesh/networks/MeshAppNetwork",
        "Microsoft.ServiceFabricMesh/volumes/ServiceAVolume"
      ],
      "properties": {
        "services": [
          {
            "name": "ServiceA",
            "properties": {
              "description": "ServiceA description.",
              "osType": "Linux",
              "codePackages": [
                {
                  "name": "ServiceA",
                  "image": "[parameters('frontEndImage')]",
                  "volumeRefs": [
                    {
                      "name": "[resourceId('Microsoft.ServiceFabricMesh/volumes', 'ServiceAVolume')]",
                      "destinationPath": "/app/data"
                    }
                  ],
                  "endpoints": [
                    {
                      "name": "ServiceAListener",
                      "port": 20001
                    }
                  ],
                  "resources": {
                    "requests": {
                      "cpu": "[parameters('frontEndCpu')]",
                      "memoryInGB": "[parameters('frontEndMemory')]"
                    }
                  },
                  "imageRegistryCredential": {
                    "server": "[parameters('registryServer')]",
                    "username": "[parameters('registryUserName')]",
                    "password": "[parameters('registryPassword')]"
                  }
                }
              ],
              "replicaCount": "[parameters('frontEndReplicaCount')]",
              "networkRefs": [
                {
                  "name": "[resourceId('Microsoft.ServiceFabricMesh/networks', 'MeshAppNetwork')]"
                }
              ]
            }
          },
          {
            "name": "ServiceB",
            ...
          }
        ],
        "description": "Application description."
      }
    },
    {
      "apiVersion": "2018-07-01-preview",
      "name": "MeshAppNetwork",
      "type": "Microsoft.ServiceFabricMesh/networks",
      "location": "[parameters('location')]",
      "dependsOn": [],
      "properties": {
        "description": "MeshAppNetwork description.",
        "addressPrefix": "10.0.0.4/22",
        "ingressConfig": {
          "layer4": [
            {
              "name": "ServiceAIngress",
              "publicPort": "20001",
              "applicationName": "MyMeshApplication",
              "serviceName": "ServiceA",
              "endpointName": "ServiceAListener"
            }
          ]
        }
      }
    },
    {
      "apiVersion": "2018-07-01-preview",
      "name": "ServiceAVolume",
      "type": "Microsoft.ServiceFabricMesh/volumes",
      "location": "[parameters('location')]",
      "dependsOn": [],
      "properties": {
        "description": "Azure Files storage volume for counter App.",
        "provider": "SFAzureFile",
        "azureFileParameters": {
          "shareName": "[parameters('fileShareName')]",
          "accountName": "[parameters('storageAccountName')]",
          "accountKey": "[parameters('storageAccountKey')]"
        }
      }
    }
  ]
}
```

如需了解待辦事項清單應用程式的細節，請參閱 [mesh_rp.windows.json 部署範本](https://github.com/Azure-Samples/service-fabric-mesh/blob/master/templates/todolist/mesh_rp.windows.json)。

## <a name="deploy-the-application-to-service-fabric-mesh"></a>將應用程式部署到 Service Fabric Mesh
請使用下列命令建立應用程式和相關的資源，並提供先前[擷取登錄的認證](#retrieve-credentials-for-the-registry)步驟中的認證。

在參數檔案中，更新下列參數值：
|參數|值|
|---|---|
|location|部署應用程式的目標區域。  例如，「eastus」。|
|registryPassword|您之前在[擷取登錄的認證](#retrieve-credentials-for-the-registry)所取得的密碼。 此參數在範本中是安全字串，而且將不會顯示在部屬狀態或 `az mesh service show` 命令中。|
|registryUserName|您之前在[擷取登錄的認證](#retrieve-credentials-for-the-registry)中取得的使用者名稱。|
|registryServer|您之前在[擷取登錄的認證](#retrieve-credentials-for-the-registry)中取得的登錄伺服器名稱。|
|frontEndImage|前端服務的容器映像。  例如，「<myregistry>.azurecr.io/seabreeze/azure-mesh-todo-webfrontend:1.0-nanoserver-1709」。|
|serviceImage|後端服務的容器映像。  例如，「<myregistry>.azurecr.io/seabreeze/azure-mesh-todo-service:1.0-nanoserver-1709」。|

若要部署應用程式，請執行下列動作：

```azurecli
az mesh deployment create --resource-group myResourceGroup --template-file c:\temp\mesh_rp.windows.json --parameters c:\temp\mesh_rp.windows.parameters.json
```

幾分鐘後，您會看到︰

`todolistappNetwork has been deployed successfully on todolistappNetwork with public ip address <IP Address>`

## <a name="open-the-application"></a>開啟應用程式

在應用程式成功部署後，請取得服務端點的公用 IP 位址。 部署命令會傳回服務端點的公用 IP 位址。 或者，您也可以查詢網路資源，來尋找服務端點的公用 IP 位址。 此應用程式的網路資源名稱是 `todolistappNetwork`，請使用下列命令擷取其相關資訊。 

```azurecli
az mesh network show --resource-group myResourceGroup --name todolistappNetwork
```

請在網頁瀏覽器中導覽至該 IP 位址。

## <a name="check-application-status"></a>檢查應用程式狀態

您可以使用 app show 命令，檢查應用程式狀態。 已部署的應用程式名稱是「todolistapp」，因此請擷取其詳細資料。

```azurecli
az mesh app show --resource-group myResourceGroup --name todolistapp
```

使用 `az mesh code-package-log get` 命令，查看已部署應用程式的記錄：
```azurecli
az mesh code-package-log get --resource-group myResourceGroup --application-name todolistapp --service-name WebFrontEnd --replica-name 0 --code-package-name WebFrontEnd
```

## <a name="next-steps"></a>後續步驟

在教學課程的這個部分中，您已了解如何：

> [!div class="checklist"]
> * 建立私人容器登錄
> * 將容器映像推送至登錄
> * 建立範本和參數檔案
> * 將應用程式部署到 Service Fabric Mesh

前進到下一個教學課程：
> [!div class="nextstepaction"]
> [調整 Service Fabric Mesh 中所執行的應用程式](service-fabric-mesh-tutorial-template-scale-services.md)
