---
title: 在 Azure Stack 中使用 API 版本設定檔搭配 Ruby | Microsoft Docs
description: 了解如何在 Azure Stack 中使用 API 版本設定檔搭配 Ruby。
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila
editor: ''
ms.assetid: B82E4979-FB78-4522-B9A1-84222D4F854B
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/10/2018
ms.author: mabrigg
ms.reviewer: sijuman
ms.openlocfilehash: 4d62c192b4e74980fc8cd8a671a702ba2ddfdbcb
ms.sourcegitcommit: ab3b2482704758ed13cccafcf24345e833ceaff3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/06/2018
ms.locfileid: "37866588"
---
# <a name="use-api-version-profiles-with-ruby-in-azure-stack"></a>在 Azure Stack 中使用 API 版本設定檔搭配 Ruby

*適用於：Azure Stack 整合系統和 Azure Stack 開發套件*

## <a name="ruby-and-api-version-profiles"></a>Ruby 和 API 版本設定檔

適用於 Azure Stack Resource Manager 的 Ruby SDK 會提供工具，以協助您建置及管理您的基礎結構。 SDK 中的資源提供者包括計算、虛擬網路和具有 Ruby 語言的儲存體。 Ruby SDK 中的 API 設定檔透過協助您在全域 Azure 資源和 Azure Stack 的資源來回切換，提供混合式雲端開發功能。

API 設定檔是資源提供者和服務版本的組合。 您可以使用 API 設定檔來組合不同的資源類型。

 - 若要利用所有服務的最新版本，請使用 Azure SDK 彙總 Gem 的**最新**設定檔。
 - 若要使用與 Azure Stack 相容的服務，請使用 Azure SDK 彙總 Gem 的 **V2017_03_09** 設定檔。
 - 若要使用服務的最新 API 版本，請使用特定 Gem 的**最新**設定檔。 例如，如果您想要單獨使用計算服務的最新 API 版本，請使用**計算** Gem 的**最新**設定檔。
 - 若要使用服務的特定 API 版本，請使用 Gem 內定義的特定 API 版本。

> [!Note]   
> 您可以在相同應用程式中組合所有選項。

## <a name="install-the-azure-ruby-sdk"></a>安裝 Azure Ruby SDK

 - 依照官方指示安裝 [Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)。
 - 依照官方指示安裝 [Ruby](https://www.ruby-lang.org/en/documentation/installation/)。
    - 安裝時，請選擇 [Add Ruby to PATH variable] \(將 Ruby 新增到 PATH 變數\)
    - 在 Ruby 安裝過程中出現提示時，請安裝開發人員套件。
    - 接下來，使用下列命令安裝搭配程式：  
      `Gem install bundler`
 - 如果無法使用，請建立訂用帳戶，並儲存訂用帳戶識別碼以供後續使用。 您可以在[這裡](https://docs.microsoft.com/azure/azure-stack/azure-stack-subscribe-plan-provision-vm)找到建立訂用帳戶的指示。 
 - 建立服務主體，並儲存其識別碼和祕密。 您可以在[這裡](https://docs.microsoft.com/azure/azure-stack/azure-stack-create-service-principals)找到建立 Azure Stack 服務主體的指示。 
 - 確保您的服務主體在訂用帳戶中擁有參與者/擁有者角色。 您可以在[這裡](https://docs.microsoft.com/azure/azure-stack/azure-stack-create-service-principals)找到如何將角色指派給服務主體的指示。

## <a name="install-the-rubygem-packages"></a>安裝 rubygem 套件

您可以直接安裝 Azure Rubygem 套件。

````Ruby  
gem install azure_mgmt_compute
gem install azure_mgmt_storage
gem install azure_mgmt_resources
gem install azure_mgmt_network
Or use them in your Gemfile.
gem 'azure_mgmt_storage'
gem 'azure_mgmt_compute'
gem 'azure_mgmt_resources'
gem 'azure_mgmt_network'
````

請注意，Azure Resource Manager Ruby SDK 目前處於預覽狀態，而且即將發行的版本中可能會有重大介面變更。 次要版本中增加的數字，表示可能有重大變更。

## <a name="usage-of-the-azuresdk-gem"></a>azure_sdk Gem 的使用方式

azure_sdk Gem 是 Ruby SDK 中所有支援 Gem 的彙總。 這個 Gem 包含**最新**設定檔，可支援所有服務的最新版本。 它會導入版本為 **V2017_03_09** 的設定檔，這是專為 Azure Stack 所建置的設定檔。

您可以使用下列命令安裝 azure_sdk 彙總 Gem：  

````Ruby  
  gem install 'azure_sdk
````

## <a name="prerequisite"></a>必要條件

若要搭配 Azure Stack 使用 Ruby Azure SDK，您必須提供下列值，然後以環境變數設定值。 針對您的作業系統設定環境變數時，請參閱下表的指示。 

| 值 | 環境變數 | 說明 | 
| --- | --- | --- | --- |
| 租用戶識別碼 | AZURE_TENANT_ID | 您的 Azure Stack [租用戶識別碼](https://docs.microsoft.com/azure/azure-stack/azure-stack-identity-overview)的值。 |
| 用戶端識別碼 | AZURE_CLIENT_ID | 您在本文件上一節中建立服務主體時儲存的服務主體應用程式識別碼。  |
| 訂用帳戶識別碼 | AZURE_SUBSCRIPTION_ID | [訂用帳戶識別碼](https://docs.microsoft.com/azure/azure-stack/azure-stack-plan-offer-quota-overview#subscriptions)是您存取 Azure Stack 中供應項目的方式。 |
| 用戶端密碼 | AZURE_CLIENT_SECRET | 服務主體建立時儲存的服務主體應用程式祕密。 |
| Resource Manager 端點 | ARM_ENDPOINT | 請參閱 [Azure Stack Resource Manager 端點](#The-azure-stack-resource-manager-endpoint)。  |

### <a name="the-azure-stack-resource-manager-endpoint"></a>Azure Stack Resource Manager 端點

Microsoft Azure Resource Manager 是可讓系統管理員進行部署、管理及監視 Azure 資源的管理架構。 Azure Resource Manager 能夠以群組方式處理這些工作，而非個別單獨作業的方式。

您可以從 Resource Manager 端點取得中繼資料資訊。 此端點會傳回 JSON 檔案，該檔案中有執行您程式碼所需的資訊。

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

### <a name="set-environmental-variables"></a>設定環境變數

**Microsoft Windows**  
若要設定環境變數，在 Windows 命令提示字元中使用下列格式：  
`set AZURE_TENANT_ID=<YOUR_TENANT_ID>`

**macOS、Linux 和以 Unix 為基礎的系統**  
在以 Unix 為基礎的系統中，您可以使用如下命令：  
`export AZURE_TENANT_ID=<YOUR_TENANT_ID>`

## <a name="existing-api-profiles"></a>現有的 API 設定檔

azure_sdk 彙總 Gem 有下列兩個設定檔：

1. **V2017_03_09**  
  針對 Azure Stack 建置的設定檔。 使用此設定檔能讓服務與 Azure Stack 有最高的相容性。
2. **最新**  
  包含所有服務最新版本的設定檔。 使用所有服務的最新版本。

如需有關 Azure Stack 和 API 設定檔的詳細資訊，請參閱 [API 設定檔的摘要](azure-stack-version-profiles.md#summary-of-api-profiles)。

## <a name="azure-ruby-sdk-api-profile-usage"></a>Azure Ruby SDK API 設定檔使用方式

應該使用下列程式行將設定檔用戶端具現化。 只有 Azure Stack 或其他私人雲端才需要此參數。 全域 Azure 預設已經有這些設定。

````Ruby  
active_directory_settings = get_active_directory_settings(ENV['ARM_ENDPOINT'])

provider = MsRestAzure::ApplicationTokenProvider.new(
    ENV['AZURE_TENANT_ID'],
    ENV['AZURE_CLIENT_ID'],
    ENV['AZURE_CLIENT_SECRET'],
    active_directory_settings
)
credentials = MsRest::TokenCredentials.new(provider)
options = {
    credentials: credentials,
    subscription_id: subscription_id,
    active_directory_settings: active_directory_settings,
    base_url: ENV['ARM_ENDPOINT']
}

# Target profile built for Azure Stack
client = Azure::Resources::Profiles::V2017_03_09::Mgmt::Client.new(options)
````

設定檔用戶端可用來存取個別資源提供者，例如計算、儲存體和網路。

````Ruby  
# To access the operations associated with Compute
profile_client.compute.virtual_machines.get 'RESOURCE_GROUP_NAME', 'VIRTUAL_MACHINE_NAME'

# Option 1: To access the models associated with Compute
purchase_plan_obj = profile_client.compute.model_classes.purchase_plan.new

# Option 2: To access the models associated with Compute
# Notice Namespace: Azure::Profiles::<Profile Name>::<Service Name>::Mgmt::Models::<Model Name>
purchase_plan_obj = Azure::Profiles::V2017_03_09::Compute::Mgmt::Models::PurchasePlan.new
````

## <a name="define-azurestack-environment-setting-functions"></a>定義 AzureStack 環境設定函式

若要向 Azure Stack 環境驗證服務主體，請使用 **get_active_directory_settings()** 定義端點。 這個方法會使用您在建立環境變數時所設定的 **ARM_Endpoint** 環境變數。

````Ruby  
# Get Authentication endpoints using Arm Metadata Endpoints
def get_active_directory_settings(armEndpoint)
    settings = MsRestAzure::ActiveDirectoryServiceSettings.new
    response = Net::HTTP.get_response(URI("#{armEndpoint}/metadata/endpoints?api-version=1.0"))
    status_code = response.code
    response_content = response.body
    unless status_code == "200"
        error_model = JSON.load(response_content)
        fail MsRestAzure::AzureOperationError.new("Getting Azure Stack Metadata Endpoints", response, error_model)
    end
    result = JSON.load(response_content)
    settings.authentication_endpoint = result['authentication']['loginEndpoint'] unless result['authentication']['loginEndpoint'].nil?
    settings.token_audience = result['authentication']['audiences'][0] unless result['authentication']['audiences'][0].nil?
    settings
end
````

## <a name="samples-using-api-profiles"></a>使用 API 設定檔的範例

您可以使用下列在 GitHub 存放庫中找到的範例，作為使用 Ruby 和 Azure Stack API 設定檔建立解決方案的參考：

 - [使用 Ruby 管理 Azure 資源與資源群組](https://github.com/Azure-Samples/resource-manager-ruby-resources-and-groups/tree/master/Hybrid)
 - [使用 Ruby 管理虛擬機器](https://github.com/Azure-Samples/compute-ruby-manage-vm/tree/master/Hybrid)
 - [使用 Ruby 格式的範本部署已啟用 SSH 的 VM](https://github.com/Azure-Samples/resource-manager-ruby-template-deployment/tree/master/Hybrid)

### <a name="sample-resource-manager-and-groups"></a>範例 Resource Manager 與群組

若要執行範例，請確定您已安裝 Ruby。 如果您使用的是 Visual Studio Code，請一併下載 Ruby SDK 作為延伸模組。 

> [!Note]  
> 您可以在[使用 Ruby 管理 Azure 資源與資源群組](https://github.com/Azure-Samples/resource-manager-ruby-resources-and-groups/tree/master/Hybrid) \(英文\) 中取得範例的存放庫。

1. 複製存放庫。

    ````Bash
    git clone https://github.com/Azure-Samples/resource-manager-ruby-resources-and-groups.git
    ````

2. 使用 bundle 安裝相依性。

    ````Bash
    cd resource-manager-ruby-resources-and-groups\Hybrid\
    bundle install
    ````

3. 使用 PowerShell 建立 Azure 服務主體，並擷取需要的值。 

  如需建立服務主體的指示，請參閱[使用 Azure PowerShell 建立具有憑證的服務主體](https://docs.microsoft.com/azure/azure-stack/azure-stack-create-service-principals)。

  所需的值如下：
  - 租用戶識別碼
  - 用戶端識別碼
  - 用戶端密碼
  - 訂用帳戶識別碼
  - Resource Manager 端點

  使用從您所建立服務主體擷取的資訊，設定下列環境變數。

  - export AZURE_TENANT_ID={您的租用戶識別碼}
  - export AZURE_CLIENT_ID={您的用戶端識別碼}
  - export AZURE_CLIENT_SECRET={您的用戶端密碼}
  - export AZURE_SUBSCRIPTION_ID = {您的訂用帳戶識別碼}
  - export ARM_ENDPOINT = {您的 AzureStack Resource Manager URL}

  > [!Note]  
  > 在 Windows 上，請將 export 改為 set。

4. 確定位置變數設定為您的 AzureStack 位置。 例如 LOCAL="local"

5. 如果您使用 Azure Stack 或其他私人雲端，請加入下列程式行以將正確的 Active Directory 端點作為目標。

  ````Ruby  
  active_directory_settings = get_active_directory_settings(ENV['ARM_ENDPOINT'])
  ````

6. 在 options 變數中，加入 Active Directory 設定和基底 URL 以搭配 Azure Stack 使用。 

  ````Ruby  
  options = {
    credentials: credentials,
    subscription_id: subscription_id,
    active_directory_settings: active_directory_settings,
    base_url: ENV['ARM_ENDPOINT']
  }
  ````

7. 建立以 Azure Stack 設定檔為目標的設定檔用戶端：

  ````Ruby  
    client = Azure::Resources::Profiles::V2017_03_09::Mgmt::Client.new(options)
  ````

8. 若要向 Azure Stack 驗證服務主體，端點應使用 **get_active_directory_settings()** 來定義。 這個方法會使用您在建立環境變數時所設定的 **ARM_Endpoint** 環境變數。

  ````Ruby  
  def get_active_directory_settings(armEndpoint)
    settings = MsRestAzure::ActiveDirectoryServiceSettings.new
    response = Net::HTTP.get_response(URI("#{armEndpoint}/metadata/endpoints?api-version=1.0"))
    status_code = response.code
    response_content = response.body
    unless status_code == "200"
      error_model = JSON.load(response_content)
      fail MsRestAzure::AzureOperationError.new("Getting Azure Stack Metadata Endpoints", response, error_model)
    end
    result = JSON.load(response_content)
    settings.authentication_endpoint = result['authentication']['loginEndpoint'] unless result['authentication']['loginEndpoint'].nil?
    settings.token_audience = result['authentication']['audiences'][0] unless result['authentication']['audiences'][0].nil?
    settings
  end
  ````

9. 執行範例。

  ````Ruby
    bundle exec ruby example.rb
  ````

## 

## <a name="next-steps"></a>後續步驟

* [安裝 Azure Stack 適用的 PowerShell](azure-stack-powershell-install.md)
* [設定 Azure Stack 使用者的 PowerShell 環境](azure-stack-powershell-configure-user.md)  
