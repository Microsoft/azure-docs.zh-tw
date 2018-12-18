---
title: 在 Azure Stack 中使用 API 版本設定檔搭配 Java | Microsoft Docs
description: 了解如何在 Azure Stack 中使用 API 版本設定檔搭配 Java。
services: azure-stack
documentationcenter: ''
author: sethmanheim
manager: femila
editor: ''
ms.assetid: ''
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/28/2018
ms.author: sethm
ms.reviewer: sijuman
ms.openlocfilehash: ffd22f3612d55258737cb9c004b2b0f4e9326f07
ms.sourcegitcommit: f31bfb398430ed7d66a85c7ca1f1cc9943656678
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47452508"
---
# <a name="use-api-version-profiles-with-java-in-azure-stack"></a>在 Azure Stack 中使用 API 版本設定檔搭配 Java

*適用於：Azure Stack 整合系統和 Azure Stack 開發套件*

適用於 Azure Stack Resource Manager 的 Java SDK 會提供工具，以協助您建置及管理您的基礎結構。 SDK 中的資源提供者包括計算、網路、儲存體、應用程式服務和 [KeyVault](../../key-vault/key-vault-whatis.md)。 Java SDK 會藉由在 Pom.xml 檔案中包含相依性來合併 API 設定檔，以將正確模組載入 .java 檔案。 不過，您可以將多個設定檔新增為相依性，例如以 **2018-03-01-hybrid** 或**最新**作為 Azure 設定檔。 使用這些相依性可載入正確的模組，讓您可以在建立資源類型時，從這些設定檔中選取想要使用的 API 版本。 這可讓您在 Azure 中使用最新版本，同時憑藉 Azure Stack 最新的 API 版本進行開發。 使用 Java SDK 可使真正的混合式雲端開發人員體驗發揮功能。 Java SDK 中的 API 設定檔透過協助您在全域 Azure 資源和 Azure Stack 的資源來回切換，提供混合式雲端開發功能。

## <a name="java-and-api-version-profiles"></a>Java 和 API 版本設定檔

API 設定檔是資源提供者和 API 版本的組合。 您可以使用 API 設定檔，取得資源提供者套件中每個資源類型的最新、最穩定版本。

- 若要使用所有服務的最新版本，請將**最新**設定檔當作相依性使用。
    
   - 若要使用最新的設定檔，相依性為 **com.microsoft.azure**。

   - 若要使用與 Azure Stack 相容的服務，請使用 **com.microsoft.azure.profile\_2018\_03\_01\_hybrid** 設定檔。
    
      - 這會在 Pom.xml 檔案中指定為相依性，並在您從下拉式清單中選擇正確類別時 (與使用 .NET 時一樣) 自動載入模組。
        
          - 每個模組的頂端會出現類似下列的內容：         
           `Import com.microsoft.azure.management.resources.v2018_03_01.ResourceGroup`
             

  - 相依性如下所示：
     ```xml
     <dependency>
     <groupId>com.microsoft.azure.profile_2018_03_01_hybrid</groupId>
     <artifactId>azure</artifactId>
     <version>1.0.0-beta</version>
     </dependency>
     ```

  - 若要使用特定資源提供者中某個資源類型的特定 API 版本，請使用透過 intellisense 所定義的特定 API 版本。

請注意，您可以組合相同應用程式中的所有選項。

## <a name="install-the-azure-java-sdk"></a>安裝 Azure Java SDK

使用下列步驟來安裝 Java SDK：

1.  依照官方指示安裝 Git。 如需相關指示，請參閱[使用者入門 - 安裝 Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)。

2.  請依照官方指示安裝 [Java SDK](http://zulu.org/download/) 和 [Maven](https://maven.apache.org/)。 Java Developer Kit 的正確版本是第 8 版。 Apache Maven 的正確版本是 3.0 版或更新版本。 JAVA_HOME 環境變數必須設定為 Java Development Kit 的安裝位置，才能完成快速入門。 如需詳細資訊，請參閱[使用 Java 和 Maven 建立您的第一個函式](../../azure-functions/functions-create-first-java-maven.md)。

3.  若要安裝正確的相依性套件，請在您的 Java 應用程式中開啟 Pom.xml 檔案。 新增相依性，如以下程式碼所示：

   ```xml  
   <dependency>
   <groupId>com.microsoft.azure.profile_2018_03_01_hybrid</groupId>
   <artifactId>azure</artifactId>
   <version>1.0.0-beta</version>
   </dependency>
   ```

4.  需要安裝的套件取決於您想要使用的設定檔版本。 設定檔版本的套件名稱是：
    
   - **com.microsoft.azure.profile\_2018\_03\_01\_hybrid**
   - **com.microsoft.azure**
      - **最新**

5.  如果無法使用，請建立訂用帳戶，並儲存訂用帳戶識別碼以供後續使用。 如需如何建立訂用帳戶的指示，請參閱[在 Azure Stack 中建立供應項目的訂用帳戶](../azure-stack-subscribe-plan-provision-vm.md)。

6.  建立服務主體，並儲存用戶端識別碼和用戶端密碼。 如需如何建立 Azure Stack 服務主體的指示，請參閱[為 Azure Stack 提供應用程式存取](../azure-stack-create-service-principals.md)。 請注意，在建立服務主體時，用戶端識別碼也稱為應用程式識別碼。

7.  確保服務主體在訂用帳戶中擁有參與者/擁有者角色。 如需如何對服務主體指派角色的指示，請參閱[為 Azure Stack 提供應用程式存取](../azure-stack-create-service-principals.md)。

## <a name="prerequisites"></a>必要條件

若要搭配使用 Azure Java SDK 與 Azure Stack，您必須提供下列值，然後以環境變數設定值。 若要設定環境變數，請參閱資料表後面適用於作業系統的指示。

| 值                     | 環境變數 | 說明                                                                                                                                                                                                          |
| ------------------------- | --------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 租用戶識別碼                 | TENANT_ID            | 您 Azure Stack [<span class="underline">租用戶識別碼</span>](../azure-stack-identity-overview.md)的值。                                                          |
| 用戶端識別碼                 | CLIENT_ID             | 您在本文件上一節中建立服務主體時儲存的服務主體應用程式識別碼。                                                                                              |
| 訂用帳戶識別碼           | SUBSCRIPTION_ID      | [<span class="underline">訂用帳戶識別碼</span>](../azure-stack-plan-offer-quota-overview.md#subscriptions)是您存取 Azure Stack 中供應項目的方式。                |
| 用戶端密碼             | CLIENT_SECRET        | 服務主體建立時儲存的服務主體應用程式祕密。                                                                                                                                   |
| Resource Manager 端點 | 端點              | 請參閱 [<span class="underline">Azure Stack Resource Manager 端點</span>](../user/azure-stack-version-profiles-ruby.md#the-azure-stack-resource-manager-endpoint)。 |
| 位置                  | RESOURCE_LOCATION    | Azure Stack 的位置                                                                                                                                                                                                |

若要尋找 Azure Stack 的租用戶識別碼，請遵循[此處](../azure-stack-csp-ref-operations.md)所提供的指示。 若要設定環境變數，請執行下列作業：

### <a name="microsoft-windows"></a>Microsoft Windows

若要在 Windows 命令提示字元中設定環境變數，請使用下列格式：

```shell
Set Azure_Tenant_ID=<Your_Tenant_ID>
```

### <a name="macos-linux-and-unix-based-systems"></a>MacOS、Linux 和以 Unix 為基礎的系統

在以 Unix 為基礎的系統中，您可以使用下列命令：

```shell
Export Azure_Tenant_ID=<Your_Tenant_ID>
```

### <a name="the-azure-stack-resource-manager-endpoint"></a>Azure Stack Resource Manager 端點

Microsoft Azure Resource Manager 是可讓系統管理員進行部署、管理及監視 Azure 資源的管理架構。 Azure Resource Manager 能夠以群組方式處理這些工作，而非個別單獨作業的方式。

您可以從 Resource Manager 端點取得中繼資料資訊。 此端點會傳回 JSON 檔案，該檔案中有執行您程式碼所需的資訊。

請注意下列考量：

- Azure Stack 開發套件 (ASDK) 中的 **ResourceManagerUrl** 為： https://management.local.azurestack.external/

- 整合系統中的 **ResourceManagerUrl** 為：`https://management.<location>.ext-<machine-name>.masd.stbtest.microsoft.com/`

若要擷取中繼資料，必須要有：`<ResourceManagerUrl>/metadata/endpoints?api-version=1.0`。

範例 JSON 檔案：

```json
{ 
   "galleryEndpoint": "https://portal.local.azurestack.external:30015/",
   "graphEndpoint": "https://graph.windows.net/",
   "portal Endpoint": "https://portal.local.azurestack.external/",
   "authentication": 
      {
      "loginEndpoint": "https://login.windows.net/",
      "audiences": ["https://management.<yourtenant>.onmicrosoft.com/3cc5febd-e4b7-4a85-a2ed-1d730e2f5928"]
      }
}
```

## <a name="existing-api-profiles"></a>現有的 API 設定檔

1.  **com.microsoft.azure.profile\_2018\_03\_01\_hybrid**：為 Azure Stack 建置的最新設定檔。 只要您是在 1808 以上的戳記上，使用此設定檔就能讓服務與 Azure Stack 有最高的相容性。

2.  **com.microsoft.azure**：包含所有服務最新版本的設定檔。 使用所有服務的最新版本。

如需有關 Azure Stack 和 API 設定檔的詳細資訊，請參閱 [API 設定檔的摘要](../user/azure-stack-version-profiles.md#summary-of-api-profiles)。

## <a name="azure-java-sdk-api-profile-usage"></a>Azure Java SDK API 設定檔使用方式

下列程式碼會在 Azure Stack 上驗證服務主體。 其會依據租用戶識別碼和驗證基底 (專屬於 Azure Stack) 來建立權杖：

```java
AzureTokenCredentials credentials = new ApplicationTokenCredentials(client, tenant, key, AZURE_STACK)
                    .withDefaultSubscriptionId(subscriptionId);
            Azure azureStack = Azure.configure().withLogLevel(com.microsoft.rest.LogLevel.BASIC)
                    .authenticate(credentials, credentials.defaultSubscriptionId());
```

這可讓您使用 API 設定檔相依性，將應用程式成功地部署至 Azure Stack。

## <a name="define-azure-stack-environment-setting-functions"></a>定義 Azure Stack 環境設定功能

若要在正確端點上註冊 Azure Stack 雲端，請使用下列程式碼：

```java
AzureEnvironment AZURE_STACK = new AzureEnvironment(new HashMap<String, String>() {
                {
                    put("managementEndpointUrl", settings.get("audience"));
                    put("resourceManagerEndpointUrl", armEndpoint);
                    put("galleryEndpointUrl", settings.get("galleryEndpoint"));
                    put("activeDirectoryEndpointUrl", settings.get("login_endpoint"));
                    put("activeDirectoryResourceId", settings.get("audience"));
                    put("activeDirectoryGraphResourceId", settings.get("graphEndpoint"));
                    put("storageEndpointSuffix", armEndpoint.substring(armEndpoint.indexOf('.')));
                    put("keyVaultDnsSuffix", ".adminvault" + armEndpoint.substring(armEndpoint.indexOf('.')));
                }
            });
```

下列程式碼中的 `getActiveDirectorySettings` 呼叫會從中繼資料端點中擷取端點。 它會從發出的呼叫中指出環境變數：

```java
public static HashMap<String, String>
getActiveDirectorySettings(String armEndpoint) {

HashMap<String, String> adSettings = new HashMap<String, String>();

try {

// create HTTP Client
HttpClient httpClient = HttpClientBuilder.create().build();

// Create new getRequest with below mentioned URL
HttpGet getRequest = new
HttpGet(String.format("%s/metadata/endpoints?api-version=1.0",
armEndpoint));

// Add additional header to getRequest which accepts application/xml
data
getRequest.addHeader("accept", "application/xml");

// Execute request and catch response
HttpResponse response = httpClient.execute(getRequest);
```

## <a name="samples-using-api-profiles"></a>使用 API 設定檔的範例

您可以使用下列 GitHub 範例，作為使用 .NET 和 Azure Stack API 設定檔來建立解決方案的參考：

  - [管理資源群組](https://github.com/viananth/resources-java-manage-resource-group/tree/stack/Hybrid)

  - [管理儲存體帳戶](https://github.com/viananth/storage-java-manage-storage-accounts/tree/stack/Hybrid)

  - [管理虛擬機器](https://github.com/viananth/compute-java-manage-vm/tree/stack/Hybrid)

### <a name="sample-unit-test-project"></a>單元測試專案範例 

1.  使用下列命令來複製存放庫：
    
    `git clone https://github.com/viananth/resources-java-manage-resource-group/tree/stack/Hybrid`

2.  建立 Azure 服務主體並指派用以存取訂用帳戶的角色。 如需建立服務主體的指示，請參閱[使用 Azure PowerShell 建立具有憑證的服務主體](../azure-stack-create-service-principals.md)。

3.  擷取下列必要的環境變數值：
    
   1.  TENANT_ID
   2.  CLIENT_ID
   3.  CLIENT_SECRET
   4.  SUBSCRIPTION_ID
   5.  ARM_ENDPOINT
   6.  RESOURCE_LOCATION

4.  在使用命令提示字元所建立的服務主體中，以從中擷取的資訊設定下列環境變數：
    
   1. export TENANT_ID={您的租用戶識別碼}
   2. export CLIENT_ID={您的用戶端識別碼}
   3. export CLIENT_SECRET={您的用戶端密碼}
   4. export SUBSCRIPTION_ID={您的訂用帳戶識別碼}
   5. export ARM_ENDPOINT={您的 Azure Stack Resource Manager URL}
   6. export RESOURCE_LOCATION={Azure Stack 的位置}

   在 Windows 上，請使用 **set** 而非 **export**。

5.  使用 `getactivedirectorysettings` 程式碼擷取 arm 中繼資料端點，並使用 HTTP 用戶端設定端點資訊。

   ```java
   public static HashMap<String, String> getActiveDirectorySettings(String armEndpoint) {
   HashMap<String, String> adSettings = new HashMap<String,> String>();

   try {

   // create HTTP Client
   HttpClient httpClient = HttpClientBuilder.create().build();

   // Create new getRequest with below mentioned URL
   HttpGet getRequest = new
   HttpGet(String.format("%s/metadata/endpoints?api-version=1.0", armEndpoint));

   // Add additional header to getRequest which accepts application/xml data
   getRequest.addHeader("accept", "application/xml");

   // Execute request and catch response
   HttpResponse response = httpClient.execute(getRequest);
   ```

6.  在 pom.xml 檔案中，新增下列相依性以使用 Azure Stack 的 2018-03-01-hybrid 設定檔。 此相依性會安裝與此設定檔相關聯的計算、網路、儲存體、KeyVault 和 App Services 資源提供者適用模組。
      
   ```xml
   <dependency>
   <groupId>com.microsoft.azure.profile_2018_03_01_hybrid</groupId>
   <artifactId>azure</artifactId>
   <version>1.0.0-beta</version>
   </dependency>
   ```

8.  在為設定環境變數而開啟的命令提示字元中，輸入下面這一行：
    
   ```shell
   mvn clean compile exec:java
   ```

## <a name="next-steps"></a>後續步驟

如需 API 設定檔的詳細資訊，請參閱：

- [在 Azure Stack 中管理 API 版本設定檔](azure-stack-version-profiles.md)
- [設定檔所支援的資源提供者 API 版本](azure-stack-profiles-azure-resource-manager-versions.md)
