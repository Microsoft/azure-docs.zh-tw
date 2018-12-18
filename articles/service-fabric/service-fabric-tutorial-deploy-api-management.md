---
title: 整合 API 管理與 Azure 中的 Service Fabric | Microsoft Docs
description: 了解如何快速開始使用 Azure API 管理以及將流量路由至 Service Fabric 中的後端服務。
services: service-fabric
documentationcenter: .net
author: rwike77
manager: timlt
editor: ''
ms.assetid: ''
ms.service: service-fabric
ms.devlang: dotNet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 9/26/2018
ms.author: ryanwi
ms.custom: mvc
ms.openlocfilehash: 572a4cd78fe60351babb9e86c604447f6848a866
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47408227"
---
# <a name="integrate-api-management-with-service-fabric-in-azure"></a>整合 API 管理與 Azure 中的 Service Fabric

使用 Service Fabric 部署 API 管理是進階案例。  當您需要為您的後端 Service Fabric 服務發佈具有豐富集合之路由規則的 API 時，API 管理很有用。 雲端應用程式通常需要前端閘道來為使用者、裝置或其他應用程式提供單一輸入點。 在 Service Fabric 中，閘道可以是為流量輸入設計的任何無狀態服務，例如 ASP.NET Core 應用程式、事件中樞、IoT 中樞或 Azure API 管理。

本文說明如何使用 Service Fabric 來設定 [Azure API 管理](../api-management/api-management-key-concepts.md)，以將流量路由至 Service Fabric 中的後端服務。  當您完成時，就已將 API 管理部署至 VNET，已設定 API 作業來將流量傳送到後端無狀態服務。 若要深入了解搭配 Service Fabric 的「Azure API 管理」案例，請參閱[概觀](service-fabric-api-management-overview.md)一文。

## <a name="prerequisites"></a>必要條件

開始之前：

* 如果您沒有 Azure 訂用帳戶，請建立[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)
* 安裝 [Azure PowerShell 模組 4.1 版或更新版本](https://docs.microsoft.com/powershell/azure/install-azurerm-ps)或 [Azure CLI](/cli/azure/install-azure-cli)。
* 在網路安全性群組中建立安全的 [Windows 叢集](service-fabric-tutorial-create-vnet-and-windows-cluster.md)。
* 如果您部署 Windows 叢集，請設定 Windows 開發環境。 安裝 [Visual Studio 2017](http://www.visualstudio.com) 和 **Azure 開發**、**ASP.NET 和 Web 開發**以及 **.NET Core 跨平台開發**工作負載。  然後設定 [.NET 開發環境](service-fabric-get-started.md)。

## <a name="network-topology"></a>網路拓撲

現在您在 Azure 上已有安全的 [Windows 叢集](service-fabric-tutorial-create-vnet-and-windows-cluster.md)，請將 API 管理部署至子網路中的虛擬網路 (VNET) 以及針對 API 管理指定的 NSG。 本文已將 API 管理 Resource Manager 範本預先設定為使用您在 [Windows 叢集教學課程](service-fabric-tutorial-create-vnet-and-windows-cluster.md)中設定的 VNET、子網路及 NSG 的名稱。本文將下列拓撲部署至 Azure，其中「API 管理」與 Service Fabric 位於相同「虛擬網路」的子網路中：

 ![圖片標題][sf-apim-topology-overview]

## <a name="sign-in-to-azure-and-select-your-subscription"></a>登入 Azure 帳戶並選取您的訂用帳戶

請先登入您的 Azure 帳戶並選取您的訂用帳戶，再執行 Azure 命令。

```powershell
Connect-AzureRmAccount
Get-AzureRmSubscription
Set-AzureRmContext -SubscriptionId <guid>
```

```azurecli
az login
az account set --subscription <guid>
```

## <a name="deploy-a-service-fabric-back-end-service"></a>部署 Service Fabric 後端服務

設定 API 管理以將流量路由傳送到 Service Fabric 後端服務之前，您需要執行中的服務以接受要求。  

使用預設的 Web API 專案範本來建立一個基本的無狀態「ASP.NET Core 可靠服務」。 這會為您的服務建立一個 HTTP 端點，而您會透過「Azure API 管理」來公開此服務。

請以「系統管理員」身分啟動 Visual Studio，然後建立 ASP.NET Core 服務：

 1. 在 Visual Studio 中，選取 [檔案] -> [新增專案]。
 2. 選取 [雲端] 底下的 [Service Fabric 應用程式] 範本，然後將它命名為 **"ApiApplication"**。
 3. 選取無狀態 ASP.NET Core 服務範本，然後將專案命名為 **"WebApiService"**。
 4. 選取 [Web API ASP.NET Core 2.0] 專案範本。
 5. 建立專案之後，開啟 `PackageRoot\ServiceManifest.xml`，然後從端點資源組態中移除 `Port` 屬性：

    ```xml
    <Resources>
      <Endpoints>
        <Endpoint Protocol="http" Name="ServiceEndpoint" Type="Input" />
      </Endpoints>
    </Resources>
    ```

    移除連接埠可讓 Service Fabric 從應用程式連接埠範圍動態指定連接埠，這些是透過叢集 Resource Manager 範本中的「網路安全性群組」開啟的連接埠，可允許流量從「API 管理」流到 Service Fabric。

 6. 在 Visual Studio 中按 F5 以確認本機有提供 Web API。

    開啟 Service Fabric Explorer，然後向下切入到特定的 ASP.NET Core 服務執行個體，以查看此服務所接聽的基底位址。 將 `/api/values` 新增至基底位址並且在瀏覽器中開啟它，這樣會在 Web API 範本中的 ValuesController 上叫用 Get 方法。 它會傳回範本所提供的預設回應，也就是包含兩個字串的 JSON 陣列：

    ```json
    ["value1", "value2"]`
    ```

    這是您會透過 Azure 中的「API 管理」公開的端點。

 7. 最後，將應用程式部署至您在 Azure 中的叢集。 在 Visual Studio 中的 [應用程式] 專案上按一下滑鼠右鍵，然後選取 [發佈]。 提供您的叢集端點 (例如 `mycluster.southcentralus.cloudapp.azure.com:19000`) 以將應用程式部署至您在 Azure 中的 Service Fabric 叢集。

一個名為 `fabric:/ApiApplication/WebApiService` 的 ASP.NET Core 無狀態服務現在應該正在 Azure 中的 Service Fabric 叢集內執行。

## <a name="download-and-understand-the-resource-manager-templates"></a>下載並了解 Resource Manager 範本

下載並儲存下列 Resource Manager 範本和參數檔：

* [network-apim.json][network-arm]
* [network-apim.parameters.json][network-parameters-arm]
* [apim.json][apim-arm]
* [apim.parameters.json][apim-parameters-arm]

*network-apim.json* 範本會在部署 Service Fabric 叢集的虛擬網路中，部署新的子網路與網路安全性群組。

下列各節描述 apim.json 範本所定義的資源。 如需詳細資訊，請遵循每個章節內範本參考文件的連結。 在 apim.parameters.json 參數檔中定義的可設定參數會在本文稍後設定。

### <a name="microsoftapimanagementservice"></a>Microsoft.ApiManagement/service

[Microsoft.ApiManagement/service](/azure/templates/microsoft.apimanagement/service) 描述「API 管理」服務執行個體：名稱、SKU 或階層、資源群組位置、發行者資訊及虛擬網路。

### <a name="microsoftapimanagementservicecertificates"></a>Microsoft.ApiManagement/service/certificates

[Microsoft.ApiManagement/service/certificates](/azure/templates/microsoft.apimanagement/service/certificates) 會設定「API 管理」安全性。 「API 管理」必須使用能夠存取您叢集的用戶端憑證來向 Service Fabric 叢集進行驗證，才能探索服務。 本文使用先前建立 [Windows 叢集](service-fabric-tutorial-create-vnet-and-windows-cluster.md#createvaultandcert_anchor)時所指定的相同憑證，此憑證依預設可用來存取您的叢集。

本文會將相同的憑證用於用戶端驗證和叢集節點對節點安全性。 如果您有一個已設定來存取 Service Fabric 叢集的個別用戶端憑證，則您可以使用該憑證。 當您建立 Service Fabric 叢集時，提供叢集憑證之私密金鑰檔案 (.pfx) 的**名稱**、**密碼**和**資料** (Base 64 編碼字串)。

### <a name="microsoftapimanagementservicebackends"></a>Microsoft.ApiManagement/service/backends

[Microsoft.ApiManagement/service/backends](/azure/templates/microsoft.apimanagement/service/backends) 描述流量轉送目標的後端服務。

就 Service Fabric 後端而言，作為後端的是 Service Fabric 叢集，而不是特定的 Service Fabric 服務。 這可讓單一原則路由傳送到叢集內的多個服務。 這裡的 **url** 欄位是當後端原則中未指定任何服務名稱時，您叢集內作為所有要求路由傳送目的地之服務的完整服務名稱。 如果您不打算有後援服務，則可以使用假的服務名稱 (例如 "fabric:/fake/service")。 **resourceId** 會指定叢集管理端點。  **clientCertificateThumbprint** 和 **serverCertificateThumbprints** 會識別用來驗證叢集的憑證。

### <a name="microsoftapimanagementserviceproducts"></a>Microsoft.ApiManagement/service/products

[Microsoft.ApiManagement/service/products](/azure/templates/microsoft.apimanagement/service/products) 會建立產品。 在 Azure API 管理中，產品包含一或多個 API，以及使用量配額與使用規定。 發行產品之後，開發人員便可訂閱產品，並開始使用產品的 API。

為產品輸入描述性 **displayName** 和 **description**。 在本文中需要訂用帳戶，但不是由管理員核准的訂用帳戶。  此產品**狀態**是「已發佈」且訂閱者可以看見。

### <a name="microsoftapimanagementserviceapis"></a>Microsoft.ApiManagement/service/apis

[Microsoft.ApiManagement/service/apis](/azure/templates/microsoft.apimanagement/service/apis) 會建立 API。 API 管理中的 API 代表可供用戶端應用程式叫用的一組作業。 加入操作之後，API 就可加入至產品，接著就可發佈。 API 發佈之後，就可供開發人員訂閱和使用。

* **displayName** 可以是 API 的任何名稱。 本文使用 "Service Fabric App"。
* **name** 提供 API 的唯一和描述性名稱，例如 "service-fabric-app"。 它會顯示在開發人員和發行者入口網站中。
* **serviceUrl** 會參考實作 API 的 HTTP 服務。 API 管理則將要求轉送至此位址。 就 Service Fabric 後端而言，並不使用此 URL 值。 您可以在這裡輸入任何值。 舉例來說，本文使用 "http://servicefabric"。
* **path** 會附加至 API 管理服務的基底 URL 後面。 基礎 URL 是 API 管理服務主控的所有 API 所共有。 API 管理依尾碼來區分 API，因此，特定發行者的每一個 API 必須有唯一的尾碼。
* **protocols** 會決定可使用哪些通訊協定來存取 API。 在本文中，請列出 **http** 和 **https**。
* **path** 是 API 的尾碼。 在本文中，請使用 "myapp"。

### <a name="microsoftapimanagementserviceapisoperations"></a>Microsoft.ApiManagement/service/apis/operations

[Microsoft.ApiManagement/service/apis/operations](/azure/templates/microsoft.apimanagement/service/apis/operations) 在「API 管理」中的 API 可以使用之前，必須將作業新增至 API。  外部用戶端使用作業來與在 Service Fabric 叢集內執行的 ASP.NET Core 無狀態服務進行通訊。

若要新增前端 API 作業，請填寫值：

* **displayName** 和 **description** 會描述作業。 在本文中，請使用 "Values"。
* **method** 會指定 HTTP 指令動詞。  在本文中，請指定 **GET**。
* **urlTemplate** 會附加至 API 的基底 URL，可識別單一 HTTP 作業。  在本文中，如果您新增 .NET 後端服務則使用 `/api/values`，或者，如果您新增 Java 後端服務則使用 `getMessage`。  根據預設，這裡指定的 URL 路徑是傳送到後端 Service Fabric 服務的 URL 路徑。 如果您在這裡使用與您服務所用相同的 URL 路徑 (例如 "/api/values")，則無須進一步修改，作業即可運作。 您也可以在這裡指定與您後端 Service Fabric 服務所用不同的 URL 路徑，在此情況下，您稍後也需要在作業原則中指定路徑重寫。

### <a name="microsoftapimanagementserviceapispolicies"></a>Microsoft.ApiManagement/service/apis/policies

[Microsoft.ApiManagement/service/apis/policies](/azure/templates/microsoft.apimanagement/service/apis/policies) 會建立後端原則，該原則會將所有項目繫結在一起。 您需在此原則中設定作為要求路由傳送目的地的後端 Service Fabric 服務。 您可以將此原則套用至任何 API 作業。  如需詳細資訊，請參閱[原則概觀](/azure/api-management/api-management-howto-policies)。

[Service Fabric 的後端組態](/azure/api-management/api-management-transformation-policies#SetBackendService)提供下列要求路由控制：

* 服務執行個體選取：方法是以硬式編碼 (例如 `"fabric:/myapp/myservice"`) 或從 HTTP 要求產生 (例如 `"fabric:/myapp/users/" + context.Request.MatchedParameters["name"]`) 來指定 Service Fabric 服務執行個體名稱。
* 分割區解析：方法是使用任何 Service Fabric 資料分割配置來產生分割區索引鍵。
* 無狀態服務的複本選取。
* 解析重試條件：可讓您指定重新解析服務位置及重新傳送要求的條件。

**policyContent** 是原則的 Json 逸出 XML 內容。  在本文中，請建立一個後端原則，以將要求直接路由傳送到稍早部署的 .NET 或 Java 無狀態服務。 在輸入原則底下新增 `set-backend-service` 原則。  如果您稍早已部署 .NET 後端服務，將 *sf-service-instance-name* 值替換為 `fabric:/ApiApplication/WebApiService`，或者如果您已部署 Java 服務，則替換為 `fabric:/EchoServerApplication/EchoServerService`。  backend-id 會參考後端資源，在此例中為 apim.json 範本中定義的 `Microsoft.ApiManagement/service/backends` 資源。 backend-id 也可以參考另一個使用「API 管理」API 所建立的後端資源。 在本文中，請將 backend-id 設定為 service_fabric_backend_name 參數的值。

```xml
<policies>
  <inbound>
    <base/>
    <set-backend-service
        backend-id="servicefabric"
        sf-service-instance-name="service-name"
        sf-resolve-condition="@(context.LastError?.Reason == 'BackendConnectionFailure')" />
  </inbound>
  <backend>
    <base/>
  </backend>
  <outbound>
    <base/>
  </outbound>
</policies>
```

如需完整的一組 Service Fabric 後端原則屬性，請參考 [API 管理後端文件](https://docs.microsoft.com/azure/api-management/api-management-transformation-policies#SetBackendService)

## <a name="set-parameters-and-deploy-api-management"></a>設定參數並且部署 API 管理

針對您的部署，在 apim.parameters.json 中填入下列空白參數。

|參數|值|
|---|---|
|apimInstanceName|sf-apim|
|apimPublisherEmail|myemail@contosos.com|
|apimSku|開發人員|
|serviceFabricCertificateName|sfclustertutorialgroup320171031144217|
|certificatePassword|q6D7nN%6ck@6|
|serviceFabricCertificateThumbprint|C4C1E541AD512B8065280292A8BA6079C3F26F10 |
|serviceFabricCertificate|&lt;base 64 編碼字串&gt;|
|url_path|/api/values|
|clusterHttpManagementEndpoint|https://mysfcluster.southcentralus.cloudapp.azure.com:19080|
|inbound_policy|&lt;XML 字串&gt;|

certificatePassword 和 serviceFabricCertificateThumbprint 必須符合用來設定叢集的叢集憑證。

serviceFabricCertificate 是 base 64 編碼字串的憑證，可以使用下列指令碼產生：

```powershell
$bytes = [System.IO.File]::ReadAllBytes("C:\mycertificates\sfclustertutorialgroup220171109113527.pfx");
$b64 = [System.Convert]::ToBase64String($bytes);
[System.Io.File]::WriteAllText("C:\mycertificates\sfclustertutorialgroup220171109113527.txt", $b64);
```

在 inbound_policy 中，如果您稍早已部署 .NET 後端服務，將 sf-service-instance-name 值替換為 `fabric:/ApiApplication/WebApiService`，或者如果您已部署 Java 服務，則替換為 `fabric:/EchoServerApplication/EchoServerService`。 backend-id 會參考後端資源，在此例中為 apim.json 範本中定義的 `Microsoft.ApiManagement/service/backends` 資源。 backend-id 也可以參考另一個使用「API 管理」API 所建立的後端資源。 在本文中，請將 backend-id 設定為 service_fabric_backend_name 參數的值。

```xml
<policies>
  <inbound>
    <base/>
    <set-backend-service
        backend-id="servicefabric"
        sf-service-instance-name="service-name"
        sf-resolve-condition="@(context.LastError?.Reason == 'BackendConnectionFailure')" />
  </inbound>
  <backend>
    <base/>
  </backend>
  <outbound>
    <base/>
  </outbound>
</policies>
```

使用下列指令碼來部署用於 API 管理的 Resource Manager 範本和參數檔：

```powershell
$groupname = "sfclustertutorialgroup"
$clusterloc="southcentralus"
$templatepath="C:\clustertemplates"

New-AzureRmResourceGroupDeployment -ResourceGroupName $groupname -TemplateFile "$templatepath\network-apim.json" -TemplateParameterFile "$templatepath\network-apim.parameters.json" -Verbose

New-AzureRmResourceGroupDeployment -ResourceGroupName $groupname -TemplateFile "$templatepath\apim.json" -TemplateParameterFile "$templatepath\apim.parameters.json" -Verbose
```

```azurecli
ResourceGroupName="sfclustertutorialgroup"
az group deployment create --name ApiMgmtNetworkDeployment --resource-group $ResourceGroupName --template-file network-apim.json --parameters @network-apim.parameters.json

az group deployment create --name ApiMgmtDeployment --resource-group $ResourceGroupName --template-file apim.json --parameters @apim.parameters.json
```

## <a name="test-it"></a>進行測試

您現在可以嘗試直接從 [Azure 入口網站](https://portal.azure.com)透過「API 管理」，將要求傳送到 Service Fabric 中的後端服務。

 1. 在 API 管理服務中，選取 [API]。
 2. 在您於先前步驟中建立的 **Service Fabric 應用程式** API 中，依序選取 [測試] 索引標籤和 [Values] 作業。
 3. 按一下 [傳送] 按鈕以將測試要求傳送到後端服務。  您應該會看到如下所示的 HTTP 回應：

    ```http
    HTTP/1.1 200 OK

    Transfer-Encoding: chunked

    Content-Type: application/json; charset=utf-8

    Vary: Origin

    Ocp-Apim-Trace-Location: https://apimgmtstodhwklpry2xgkdj.blob.core.windows.net/apiinspectorcontainer/PWSQOq_FCDjGcaI1rdMn8w2-2?sv=2015-07-08&sr=b&sig=MhQhzk%2FEKzE5odlLXRjyVsgzltWGF8OkNzAKaf0B1P0%3D&se=2018-01-28T01%3A04%3A44Z&sp=r&traceId=9f8f1892121e445ea1ae4d2bc8449ce4

    Date: Sat, 27 Jan 2018 01:04:44 GMT


    ["value1", "value2"]
    ```

## <a name="clean-up-resources"></a>清除資源

叢集是由叢集資源本身和其他 Azure 資源所構成。 刪除叢集及其取用之所有資源的最簡單方式，就是刪除資源群組。

登入 Azure 並選取您要移除叢集的訂用帳戶識別碼。  您可以登入[Azure 入口網站](http://portal.azure.com)找到您的訂用帳戶識別碼。 使用 [Remove-AzureRMResourceGroup Cmdlet](/en-us/powershell/module/azurerm.resources/remove-azurermresourcegroup) 刪除資源群組和所有叢集資源。

```powershell
$ResourceGroupName = "sfclustertutorialgroup"
Remove-AzureRmResourceGroup -Name $ResourceGroupName -Force
```

```azurecli
ResourceGroupName="sfclustertutorialgroup"
az group delete --name $ResourceGroupName
```

## <a name="next-steps"></a>後續步驟

深入了解如何使用 [API 管理](/azure/api-management/import-and-publish)。

[azure-powershell]: https://azure.microsoft.com/documentation/articles/powershell-install-configure/

[apim-arm]:https://github.com/Azure/service-fabric-scripts-and-templates/blob/master/templates/service-integration/apim.json
[apim-parameters-arm]:https://github.com/Azure/service-fabric-scripts-and-templates/blob/master/templates/service-integration/apim.parameters.json

[network-arm]: https://github.com/Azure/service-fabric-scripts-and-templates/blob/master/templates/service-integration/network-apim.json
[network-parameters-arm]: https://github.com/Azure/service-fabric-scripts-and-templates/blob/master/templates/service-integration/network-apim.parameters.json

<!-- pics -->
[sf-apim-topology-overview]: ./media/service-fabric-tutorial-deploy-api-management/sf-apim-topology-overview.png
