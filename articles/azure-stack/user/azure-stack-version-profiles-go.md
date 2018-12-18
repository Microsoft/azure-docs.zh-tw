---
title: 在 Azure Stack 中使用 API 版本設定檔與 GO | Microsoft Docs
description: 了解如何在 Azure Stack 中使用 API 版本設定檔與 GO。
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/10/2018
ms.author: mabrigg
ms.reviewer: sijuman
ms.openlocfilehash: dd2d0c46c0829a73d32c96b506b9f2111eda3c84
ms.sourcegitcommit: 909469bf17211be40ea24a981c3e0331ea182996
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/10/2018
ms.locfileid: "34010059"
---
# <a name="use-api-version-profiles-with-go-in-azure-stack"></a>在 Azure Stack 中使用 API 版本設定檔與 GO

*適用於：Azure Stack 整合系統和 Azure Stack 開發套件*

## <a name="go-and-version-profiles"></a>GO 與版本設定檔

設定檔結合了不同服務之不同版本的不同資源類型。 使用設定檔可協助您混合和比對不同的資源類型。 設定檔可提供：

 - 藉由鎖定至特定 API 版本所帶來的應用程式穩定性。
 - 您的應用程式與 Azure Stack 和區域 Azure 資料中心的相容性。

在 Go SDK 中，設定檔可於 profiles/ 路徑下取得，而其版本採用 **YYYY-MM-DD** 格式。 現在，最新的 Azure Stack 設定檔版本為 **2017-03-09**。 若要從設定檔匯入指定的服務，您必須從設定檔匯入其對應的模組。 例如，若要從 **2017-03-09** 設定檔匯入**計算**服務：

````go
import "github.com/Azure/azure-sdk-for-go/profiles/2017-03-09/compute/mgmt/compute" 
````

## <a name="install-azure-sdk-for-go"></a>安裝 Azure SDK for Go

  1. 安裝 GIT。 如需相關指示，請參閱[使用者入門 - 安裝 Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)。
  2. 安裝 [Go 程式設計語言](https://golang.org/dl)。  
  Azure 的 API 設定檔將需要 Go 1.9 版或更新版本。
  3. 執行下列 bash 命令，以安裝 GO Azure SDK 及其相依性：
  ```
    go get -u -d github.com/Azure/azure-sdk-for-go/...
  ```

### <a name="the-go-sdk"></a>GO SDK

您可以在下列位置找到更多關於 Azure GO SDK 的資訊：
- [安裝 Azure SDK for Go](https://docs.microsoft.com/go/azure/azure-sdk-go-install) 上的 Azure Go SDK。
- Azure Go SDK 可從 [azure-sdk-for-go](https://github.com/Azure/azure-sdk-for-go) 上的 GitHub 公開取得。

### <a name="go-autorest-dependencies"></a>Go-AutoRest 相依性

GO SDK 需倚賴 Azure Go-AutoRest 模組將 REST 要求傳送至 Azure Resource Manager 端點。 您必須從 [GitHub 上的 Azure Go-AutoRest](https://github.com/Azure/go-autorest) 匯入 Azure Go-AutoRest 模組相依性。 您可以在**安裝**一節中找到安裝 bash 命令。

## <a name="how-to-use-go-sdk-profiles-on-azure-stack"></a>如何在 Azure Stack 上使用 GO SDK 設定檔

若要在 Azure Stack 上執行 GO 程式碼的範例：
  1. 安裝 Azure SDK for Go 及其相依性。 如需相關指示，請參閱上一節：[安裝 Azure SDK for Go](#install-azure-sdk-for-go)。
  2. 從資源管理員端點取得中繼資料資訊。 此端點會傳回 JSON 檔案，以及執行 GO 程式碼所需的資訊。

  > [!Note]  
  > Azure Stack 開發套件 (ASDK) 中的 **ResourceManagerUrl** 為：`https://management.local.azurestack.external/`  
  > 整合系統中的 **ResourceManagerUrl** 為：`https://management.<location>.ext-<machine-name>.masd.stbtest.microsoft.com/`  
  > 若要擷取中繼資料，必須要有：`<ResourceManagerUrl>/metadata/endpoints?api-version=1.0`
  
  範例 JSON 檔案：

  ```json
  { "galleryEndpoint": "https://portal.local.azurestack.external:30015/",  
    "graphEndpoint": "https://graph.windows.net/",  
    "portal Endpoint": "https://portal.local.azurestack.external/", 
    "authentication": {
      "loginEndpoint": "https://login.windows.net/", 
      "audiences": ["https://management.<yourtenant>.onmicrosoft.com/3cc5febd-e4b7-4a85-a2ed-1d730e2f5928"]
    }
  }
  ```

  3. 如果無法使用，請建立訂用帳戶，並儲存訂用帳戶識別碼供後續使用。 如需建立訂用帳戶的相關資訊，請參閱[在 Azure Stack 中建立供應項目的訂用帳戶](https://docs.microsoft.com/azure/azure-stack/azure-stack-subscribe-plan-provision-vm)。 
  4. 建立具有「訂用帳戶」範圍和**擁有者**角色的服務主體。 儲存服務主體的識別碼和秘密。 如需為 Azure Stack 建立服務主體的相關資訊，請參閱[建立服務主體](https://docs.microsoft.com/azure/azure-stack/azure-stack-create-service-principals#create-service-principal-for-azure-ad)。 您的 Azure Stack 環境已設定完成。
  5. 使用程式碼從 GO SDK 設定檔匯入服務模組。 Azure Stack 設定檔目前的版本是 **2017-03-09**。 例如，若要從 **2017-03-09** 設定檔類型匯入網路模組： 

  ````go
    package main 
    import "github.com/Azure/azure-sdk-for-go/profiles/2017-03-09/network/mgmt/network"
  ````
  
  6. 在您的函式中，使用**新增**用戶端函式呼叫建立並驗證用戶端。 若要建立虛擬網路用戶端，您可以使用下列程式碼：  

  ````go
  package main 

  import "github.com/Azure/azure-sdk-for-go/profiles/2017-03-09/network/mgmt/network" 

  func main() { 
      vnetClient := network.NewVirtualNetworksClientWithBaseURI("<baseURI>", "(subscriptionID>")
      vnetClient.Authorizer = autorest.NewBearerAuthorizer(token)
  ````

  將 `<baseURI>` 設為在步驟二中使用的 **ResourceManagerUrl** 值。
  將 `<subscriptionID>` 設為儲存自步驟三的 **SubscriptionID** 值。
  若要建立權杖，請參閱下方的「驗證」一節。  

  7. 使用您在上一個步驟中建立的用戶端，叫用 API 方法。 例如，若要使用我們上一個步驟中的用戶端建立虛擬網路： 
  
````go
package main

import "github.com/Azure/azure-sdk-for-go/profiles/2017-03-09/network/mgmt/network" 
func main() { 
  vnetC1ient := network.NewVirtualNetworksClientWithBaseURI("<baseURI>", "(subscriptionID>")
  vnetClient .Authorizer = autorest.NewBearerAuthorizer(token)

  vnetClient .CreateOrUpdate( ) 
````
  
  如需使用 Go SDK 設定檔在 Azure Stack 上建立虛擬網路的完整範例，請參閱[範例](#example)。

## <a name="authentication"></a>驗證

若要使用 Go SDK 從 Azure Active Directory 取得授權者屬性，請安裝 Go-AutoRest 模組。 這些模組應已使用 "Go SDK" 安裝完成安裝；若尚未安裝，請安裝 [GitHub 上的驗證套件](https://github.com/Azure/go-autorest/tree/master/autorest/adal)。

[授權者] 必須設為資源用戶端的授權者。 您可以透過不同的方法取得授權者；如需完整清單，請參閱此處。

本節介紹使用用戶端認證在 Azure Stack 上取得授權者權杖的常見方法：

  1. 如果訂用帳戶上有具備擁有者角色的服務主體可供使用，請略過此步驟。 否則，請建立服務主體[指示]( https://docs.microsoft.com/azure/azure-stack/azure-stack-create-service-principals)，並為其指派範圍限定於您的訂用帳戶[指示]( https://docs.microsoft.com/azure/azure-stack/azure-stack-create-service-principals#assign-role-to-service-principal)的「擁有者」角色。 儲存服務主體應用程式識別碼和密碼。 

  2. 使用程式碼從 Go-AutoRest 匯入 **adal** 套件。 
  
  ````go
  package main
  import "github.com/Azure/go-autorest/autorest/adal" 
  ````

  3. 使用 NewOAuthConfig 方法從 **adal** 模組建立 oauthConfig。 
  
  ````go
  package main 

  import "github.com/Azure/go-autorest/autorest/ada1" 

  func CreateToken() (adal.OAuthTokenProvider, error) {
      var token adal.OAuthTokenProvider
      oauthConfig, err := adal.NewOAuthConfig(activeDirectoryEndpoint, tenantID)
  ````
   
  將 `<activeDirectoryEndpoint>` 設為在本文上一節中擷取之 ResourceManagerUrl 中繼資料中的 "loginEndpoint" 屬性值。
  將 `<tenantID>` 值設為您的 Azure Stack 租用戶識別碼。 

  4. 最後，使用 NewServicePrincipalToken 方法從 adal 模組建立服務主體權杖。 

  ````go
  package main 

  import "github.com/Azure/go-autorest/autorest/adal" 

  func CreateToken() (adal.OAuthTokenProvider, error) {
      var token adal.OAuthTokenProvider
      oauthConfig, err := adal.NewOAuthConfig(activeDirectoryEndpoint, tenantID)
      token, err = adal.NewServicePrincipalToken(
          *oauthConfig,
          clientID,
          clientSecret,
          activeDirectoryResourceID)
      return token, err
  ````
  
  將 `<activeDirectoryResourceID>` 設為在本文上一節中擷取之 ResourceManagerUrl 中繼資料中的「對象」清單值之一。  
  將 `<clientID>` 設為您在本文上一節中建立服務主體時儲存的服務主體應用程式識別碼。  
  將 `<clientSecret>` 設為您在本文上一節中建立服務主體時儲存的服務主體應用程式密碼。  

## <a name="example"></a>範例

本節說明在 Azure Stack 上建立虛擬網路的 GO 程式碼範例。 如需 Go SDK 的完整範例，請參閱 [Azure Go SDk 範例存放庫](https://github.com/Azure-Samples/azure-sdk-for-go-samples)。 Azure Stack 範例可從該存放庫之服務資料夾內的 hybrid/ 路徑下取得。

> [!Note]  
> 若要執行此範例中的程式碼，請確認使用的訂用帳戶具有列示為**已註冊**的**網路**資源提供者。 若要加以確認，請在 Azure Stack 入口網站中找出訂用帳戶，然後按一下 [資源提供者]。

1. 使用程式碼匯入所需的套件。 您應使用 Azure Stack 上最新的可用設定檔來匯入網路模組。 
  
  ````go
  package main

  import (
      "context"
      "fmt"
      "github.com/Azure/azure-sdk-for-go/profiles/2017-03-09/network/mgmt/network"
      "github.com/Azure/go-autorest/autorest"
      "github.com/Azure/go-autorest/autorest/adal"
      "github.com/Azure/go-autorest/autorest/to"
  )
  ````

2. 定義您的環境變數。 若要要建立虛擬網路，您必須具有資源群組。 

  ````go
  var (
      activeDirectoryEndpoint = "yourLoginEndpointFromResourceManagerUrlMetadata"
      tenantID = "yourAzureStackTenantID"
      clientID = "yourServicePrincipalApplicationID"
      clientSecret = "yourServicePrincipalSecret"
      activeDirectoryResourceID = "yourAudienceFromResourceManagerUrlMetadata"
      subscriptionID = "yourSubscriptionID"
      baseURI = "yourResourceManagerURL"
      resourceGroupName = "existingResourceGroupName"
  )
  ````

3. 您已定義環境變數，現在請使用 **adal** 套件新增建立驗證權杖的方法。 請參閱上一節中關於驗證的詳細資料。
  
  ````go
  //CreateToken creates a service principal token
  func CreateToken() (adal.OAuthTokenProvider, error) {
      var token adal.OAuthTokenProvider
      oauthConfig, err := adal.NewOAuthConfig(activeDirectoryEndpoint, tenantID)
      token, err = adal.NewServicePrincipalToken(
          *oauthConfig,
          clientID,
          clientSecret,
          activeDirectoryResourceID)
      return token, err
  }
  ````

4. 新增 Main 方法。 Main 方法會先使用上一個步驟中定義的方法取得權杖。 然後，它會使用設定檔中的網路模組建立用戶端。 最後，它會建立虛擬網路。 
  
````go
package main

import (
    "context"
    "fmt"
    "github.com/Azure/azure-sdk-for-go/profiles/2017-03-09/network/mgmt/network"
    "github.com/Azure/go-autorest/autorest"
    "github.com/Azure/go-autorest/autorest/adal"
    "github.com/Azure/go-autorest/autorest/to"
)

var (
    activeDirectoryEndpoint = "yourLoginEndpointFromResourceManagerUrlMetadata"
    tenantID = "yourAzureStackTenantID"
    clientID = "yourServicePrincipalApplicationID"
    clientSecret = "yourServicePrincipalSecret"
    activeDirectoryResourceID = "yourAudienceFromResourceManagerUrlMetadata"
    subscriptionID = "yourSubscriptionID"
    baseURI = "yourResourceManagerURL"
    resourceGroupName = "existingResourceGroupName"
)

//CreateToken creates a service principal token
func CreateToken() (adal.OAuthTokenProvider, error) {
    var token adal.OAuthTokenProvider
    oauthConfig, err := adal.NewOAuthConfig(activeDirectoryEndpoint, tenantID)
    token, err = adal.NewServicePrincipalToken(
        *oauthConfig,
        clientID,
        clientSecret,
        activeDirectoryResourceID)
    return token, err
}

func main() {
    token, _ := CreateToken()
    vnetClient := network.NewVirtualNetworksClientWithBaseURI(baseURI, subscriptionID)
    vnetClient.Authorizer = autorest.NewBearerAuthorizer(token)
    future, _ := vnetClient.CreateOrUpdate(
        context.Background(),
        resourceGroupName,
        "sampleVnetName",
        network.VirtualNetwork{
            Location: to.StringPtr("local"),
            VirtualNetworkPropertiesFormat: &network.VirtualNetworkPropertiesFormat{
                AddressSpace: &network.AddressSpace{
                    AddressPrefixes: &[]string{"10.0.0.0/8"},
                },
                Subnets: &[]network.Subnet{
                    {
                        Name: to.StringPtr("subnetName"),
                        SubnetPropertiesFormat: &network.SubnetPropertiesFormat{
                            AddressPrefix: to.StringPtr("10.0.0.0/16"),
                        },
                    },
                },
            },
        })
    err := future.WaitForCompletion(context.Background(), vnetClient.Client)
    if err != nil {
        fmt.Printf(err.Error())
        return
    }
}

````

## <a name="next-steps"></a>後續步驟
* [安裝 Azure Stack 適用的 PowerShell](azure-stack-powershell-install.md)
* [設定 Azure Stack 使用者的 PowerShell 環境](azure-stack-powershell-configure-user.md)  
