---
title: 從 Azure Data Lake Storage Gen1 來回複製資料 | Microsoft Docs
description: 了解如何使用 Azure Data Factory 從 Azure Data Lake Store 來回複製資料
services: data-factory
documentationcenter: ''
author: linda33wj
manager: craigg
ms.assetid: 25b1ff3c-b2fd-48e5-b759-bb2112122e30
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 01/22/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: 97bd2081df8c90f885996629862f25cbec8fd2c2
ms.sourcegitcommit: 0b4da003fc0063c6232f795d6b67fa8101695b61
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/05/2018
ms.locfileid: "37860226"
---
# <a name="copy-data-to-and-from-data-lake-storage-gen1-by-using-data-factory"></a>使用 Data Factory 從 Data Lake Storage Gen1 來回複製資料
> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [第 1 版](data-factory-azure-datalake-connector.md)
> * [第 2 版 (目前的版本)](../connector-azure-data-lake-store.md)

> [!NOTE]
> 本文適用於 Data Factory 第 1 版。 如果您使用目前的 Data Factory 服務版本，請參閱[第 2 版中的 Azure Data Lake Storage Gen1 連接器](../connector-azure-data-lake-store.md)。

本文說明如何使用 Azure Data Factory 中的「複製活動」，從 Azure Data Lake Storage Gen1 (先前稱為 Azure Data Lake Store) 來回移動資料。 本文是根據[資料移動活動](data-factory-data-movement-activities.md)一文，該文概述使用「複製活動」移動資料。

## <a name="supported-scenarios"></a>支援的案例
您可以將資料從 Azure Data Lake Store 複製到下列資料存放區：

[!INCLUDE [data-factory-supported-sinks](../../../includes/data-factory-supported-sinks.md)]

您可以從下列資料存放區將資料複製到 Azure Data Lake Store：

[!INCLUDE [data-factory-supported-sources](../../../includes/data-factory-supported-sources.md)]

> [!NOTE]
> 先建立 Data Lake Store 帳戶，再使用「複製活動」建立管線。 如需詳細資訊，請參閱[開始使用 Azure Data Lake Store](../../data-lake-store/data-lake-store-get-started-portal.md)。

## <a name="supported-authentication-types"></a>支援的驗證類型
Data Lake Store 連接器支援這些驗證類型：
* 服務主體驗證
* 使用者認證 (OAuth) 驗證 

我們建議您使用服務主體驗證，特別是針對排程的資料複本。 權杖到期行為會連同使用者認證驗證發生。 如需組態詳細資料，請參閱[連結服務屬性](#linked-service-properties)一節。

## <a name="get-started"></a>開始使用
您可以藉由使用不同的工具/API，建立內含複製活動的管線，以將資料移進/移出 Azure Data Lake Store。

建立管線的最簡單方式就是使用「複製精靈」。 如需使用「複製精靈」建立管線的教學課程，請參閱[教學課程︰使用複製精靈建立管線](data-factory-copy-data-wizard-tutorial.md)。

您也可以使用下列工具來建立管線︰**Azure 入口網站**、**Visual Studio**、**Azure PowerShell**、**Azure Resource Manager 範本**、**.NET API** 及 **REST API**。 如需建立內含複製活動之管線的逐步指示，請參閱[複製活動教學課程](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)。

不論您是使用工具還是 API，都需執行下列步驟來建立將資料從來源資料存放區移到接收資料存放區的管線：

1. 建立 **Data Factory**。 資料處理站可包含一或多個管線。 
2. 建立**連結服務**，將輸入和輸出資料存放區連結到資料處理站。 例如，如果您將資料從 Azure Blob 儲存體複製到 Azure Data Lake Store，您會建立兩個連結服務，可將 Azure 儲存體帳戶和 Azure Data Lake Store 連結至資料處理站。 針對 Azure Data Lake Store 專屬的連結服務屬性，請參閱[連結服務屬性](#linked-service-properties)一節。 
2. 建立**資料集**，代表複製作業的輸入和輸出資料。 在上一個步驟所述的範例中，您會建立資料集來指定 blob 容器和包含輸入資料的資料夾。 同時並建立另一個資料集，以指定在 Data Lake Store 中保存從 Blob 儲存體複製之資料的資料夾和檔案路徑。 如需 Azure Data Lake Store 專屬的資料集屬性，請參閱[資料集屬性](#dataset-properties)一節。
3. 建立**管線**，其中含有以一個資料集作為輸入、一個資料集作為輸出的複製活動。 在稍早所述的範例中，您使用 BlobSource 作為來源，以及使用 AzureDataLakeStoreSink 作為複製活動的接收器。 同樣地，如果您正從 Azure Data Lake Store 複製到 Azure Blob 儲存體，則會在複製活動中使用 AzureDataLakeStoreSource 與 BlobSink。 針對 Azure Data Lake Store 專屬的複製活動屬性，請參閱[複製活動屬性](#copy-activity-properties)一節。 如需有關如何使用資料存放區作為來源或接收器的詳細資訊，請按一下上一節中資料存放區的連結。  

使用精靈時，精靈會自動為您建立這些 Data Factory 實體 (已連結的服務、資料集及管線) 的 JSON 定義。 使用工具/API (.NET API 除外) 時，您需使用 JSON 格式來定義這些 Data Factory 實體。  如需相關範例，其中含有用來將資料複製到 Azure Data Lake Store (或從 Azure Data Lake Store 複製資料) 之 Data Factory 實體的 JSON 定義，請參閱本文的 [JSON 範例](#json-examples-for-copying-data-to-and-from-data-lake-store)一節。

下列各節提供 JSON 屬性的相關詳細資料，這些屬性是用來定義 Data Lake Store 特定的 Data Factory 實體。

## <a name="linked-service-properties"></a>連結服務屬性
已連結的服務會將資料存放區連結到 Data Factory。 您建立類型為 **AzureDataLakeStore** 的連結服務，以將 Data Lake Store 資料連結到資料處理站。 下表提供 Data Lake Store 連結服務專屬的 JSON 元素說明。 您可以在服務主體與使用者認證驗證之間選擇。

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| **type** | 類型屬性必須設定為 **AzureDataLakeStore**。 | yes |
| **dataLakeStoreUri** | Azure Data Lake Store 帳戶相關資訊。 此資訊會採用下列其中一種格式：`https://[accountname].azuredatalakestore.net/webhdfs/v1` 或 `adl://[accountname].azuredatalakestore.net/`。 | yes |
| **subscriptionId** | Data Lake Store 帳戶所屬的 Azure 訂用帳戶識別碼。 | 接收 (Sink) 的必要項目 |
| **resourceGroupName** | Data Lake Store 帳戶所屬的 Azure 資源群組名稱。 | 接收 (Sink) 的必要項目 |

### <a name="service-principal-authentication-recommended"></a>服務主體驗證 (建議)
若要使用服務主體驗證，請在 Azure Active Directory (Azure AD) 中註冊應用程式實體，並授與其 Data Lake Store 存取權。 如需詳細的步驟，請參閱[服務對服務驗證](../../data-lake-store/data-lake-store-authenticate-using-active-directory.md)。 請記下以下的值，您可以使用這些值來定義連結服務：
* 應用程式識別碼
* 應用程式金鑰 
* 租用戶識別碼

> [!IMPORTANT]
> 請確定您將 Azure Data Lake Store 中適當的權限授與服務主體：
>- **若要使用 Data Lake Store 作為來源**，請至少授與**讀取 + 執行**資料存取權限，以列出和複製資料夾的內容，或授與**讀取**權限，以複製單一檔案。 在帳戶層級存取控制上沒有任何要求。
>- **若要使用 Data Lake Store 作為接收器**，請至少授與**寫入 + 執行**資料存取權限，以在資料夾中建立子項目。 如果您使用 Azure IR 來授權複製 (來源和接收器都在雲端)，為了讓 Data Factory 偵測 Data Lake Store 的區域，請在帳戶存取控制 (IAM) 中至少授與**讀取者**角色。 如果您想要避免使用此 IAM 角色，請在複製活動中以您的 Data Lake Store 位置[指定 executionLocation](data-factory-data-movement-activities.md#global)。
>- 如果您**使用「複製精靈」來撰寫管線**，請在帳戶存取控制 (IAM) 中至少授與**讀取者**角色。 此外，請至少授與您 Data Lake Store 根目錄 ("/") 及其子系的**讀取 + 執行**權限。 否則，您可能會看到「提供的認證無效」訊息。

指定下列屬性以使用服務主體驗證：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| **servicePrincipalId** | 指定應用程式的用戶端識別碼。 | yes |
| **servicePrincipalKey** | 指定應用程式的金鑰。 | yes |
| **tenant** | 指定您的應用程式所在租用戶的資訊 (網域名稱或租用戶識別碼)。 將滑鼠游標暫留在 Azure 入口網站右上角，即可擷取它。 | yes |

**範例：服務主體驗證**
```json
{
    "name": "AzureDataLakeStoreLinkedService",
    "properties": {
        "type": "AzureDataLakeStore",
        "typeProperties": {
            "dataLakeStoreUri": "https://<accountname>.azuredatalakestore.net/webhdfs/v1",
            "servicePrincipalId": "<service principal id>",
            "servicePrincipalKey": "<service principal key>",
            "tenant": "<tenant info, e.g. microsoft.onmicrosoft.com>",
            "subscriptionId": "<subscription of ADLS>",
            "resourceGroupName": "<resource group of ADLS>"
        }
    }
}
```

### <a name="user-credential-authentication"></a>使用者認證驗證
或者，您也可以藉由指定下列屬性，使用使用者認證驗證從 Data Lake Store 複製，或是複製到 Data Lake Store：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| **authorization** | 按一下「資料處理站編輯器」中的 [授權] 按鈕，然後輸入您的認證，此動作會將自動產生的授權 URL 指派給此屬性。 | yes |
| **sessionId** | OAuth 授權工作階段的 OAuth 工作階段識別碼。 每個工作階段識別碼都是唯一的，只能使用一次。 當您使用「資料處理站編輯器」時便會自動產生此設定。 | yes |

> [!IMPORTANT]
> 請確定您將 Azure Data Lake Store 中適當的權限授與使用者：
>- **若要使用 Data Lake Store 作為來源**，請至少授與**讀取 + 執行**資料存取權限，以列出和複製資料夾的內容，或授與**讀取**權限，以複製單一檔案。 在帳戶層級存取控制上沒有任何要求。
>- **若要使用 Data Lake Store 作為接收器**，請至少授與**寫入 + 執行**資料存取權限，以在資料夾中建立子項目。 如果您使用 Azure IR 來授權複製 (來源和接收器都在雲端)，為了讓 Data Factory 偵測 Data Lake Store 的區域，請在帳戶存取控制 (IAM) 中至少授與**讀取者**角色。 如果您想要避免使用此 IAM 角色，請在複製活動中以您的 Data Lake Store 位置[指定 executionLocation](data-factory-data-movement-activities.md#global)。
>- 如果您**使用「複製精靈」來撰寫管線**，請在帳戶存取控制 (IAM) 中至少授與**讀取者**角色。 此外，請至少授與您 Data Lake Store 根目錄 ("/") 及其子系的**讀取 + 執行**權限。 否則，您可能會看到「提供的認證無效」訊息。

**範例：使用者認證授權**
```json
{
    "name": "AzureDataLakeStoreLinkedService",
    "properties": {
        "type": "AzureDataLakeStore",
        "typeProperties": {
            "dataLakeStoreUri": "https://<accountname>.azuredatalakestore.net/webhdfs/v1",
            "sessionId": "<session ID>",
            "authorization": "<authorization URL>",
            "subscriptionId": "<subscription of ADLS>",
            "resourceGroupName": "<resource group of ADLS>"
        }
    }
}
```

#### <a name="token-expiration"></a>權杖到期
您使用 [授權] 按鈕所產生的授權碼在一段時間後會到期。 下列訊息表示驗證權杖已過期：

認證作業錯誤：invalid_grant - AADSTS70002：驗證認證時發生錯誤。 AADSTS70008：提供的存取授權已過期或撤銷。 追蹤識別碼：d18629e8-af88-43c5-88e3-d8419eb1fca1 相互關連識別碼：fac30a0c-6be6-4e02-8d69-a776d2ffefd7 時間戳記：2015-12-15 21-09-31Z。

下表顯示不同類型使用者帳戶的到期時間：

| 使用者類型 | 到期時間 |
|:--- |:--- |
| 「不」受 Azure Active Directory 管理的使用者帳戶 (例如 @hotmail.com 或 @live.com) |12 小時 |
| 受 Azure Active Directory 管理的使用者帳戶 |最後一個配量執行後的 14 天 <br/><br/>如果以 OAuth 式連結服務為基礎的配量至少每 14 天執行一次，則為 90 天 |

如果您在此權杖的到期時間之前變更密碼，權杖會立即到期。 您將會看到本節稍早所提到的訊息。

當權杖到期時，您可以使用 [授權] 按鈕重新授權帳戶，以重新部署連結服務。 您也可以使用下列程式碼，以程式設計方式產生 sessionId  和 authorization 屬性的值：


```csharp
if (linkedService.Properties.TypeProperties is AzureDataLakeStoreLinkedService ||
    linkedService.Properties.TypeProperties is AzureDataLakeAnalyticsLinkedService)
{
    AuthorizationSessionGetResponse authorizationSession = this.Client.OAuth.Get(this.ResourceGroupName, this.DataFactoryName, linkedService.Properties.Type);

    WindowsFormsWebAuthenticationDialog authenticationDialog = new WindowsFormsWebAuthenticationDialog(null);
    string authorization = authenticationDialog.AuthenticateAAD(authorizationSession.AuthorizationSession.Endpoint, new Uri("urn:ietf:wg:oauth:2.0:oob"));

    AzureDataLakeStoreLinkedService azureDataLakeStoreProperties = linkedService.Properties.TypeProperties as AzureDataLakeStoreLinkedService;
    if (azureDataLakeStoreProperties != null)
    {
        azureDataLakeStoreProperties.SessionId = authorizationSession.AuthorizationSession.SessionId;
        azureDataLakeStoreProperties.Authorization = authorization;
    }

    AzureDataLakeAnalyticsLinkedService azureDataLakeAnalyticsProperties = linkedService.Properties.TypeProperties as AzureDataLakeAnalyticsLinkedService;
    if (azureDataLakeAnalyticsProperties != null)
    {
        azureDataLakeAnalyticsProperties.SessionId = authorizationSession.AuthorizationSession.SessionId;
        azureDataLakeAnalyticsProperties.Authorization = authorization;
    }
}
```
如需程式碼中所使用 Data Factory 類別的詳細資料，請參閱 [AzureDataLakeStoreLinkedService 類別](https://msdn.microsoft.com/library/microsoft.azure.management.datafactories.models.azuredatalakestorelinkedservice.aspx)、[AzureDataLakeAnalyticsLinkedService 類別](https://msdn.microsoft.com/library/microsoft.azure.management.datafactories.models.azuredatalakeanalyticslinkedservice.aspx)和 [AuthorizationSessionGetResponse 類別](https://msdn.microsoft.com/library/microsoft.azure.management.datafactories.models.authorizationsessiongetresponse.aspx)主題。 新增程式碼中所使用 `WindowsFormsWebAuthenticationDialog` 類別的 `2.9.10826.1824` 版 `Microsoft.IdentityModel.Clients.ActiveDirectory.WindowsForms.dll` 參考。

## <a name="troubleshooting-tips"></a>疑難排解秘訣

**徵兆：** 將資料複製**到** Azure Data Lake Store 時，如果複製活動因以下錯誤而發生失敗：

  ```
  Failed to detect the region for Azure Data Lake account {your account name}. Please make sure that the Resource Group name: {resource group name} and subscription ID: {subscription ID} of this Azure Data Lake Store resource are correct.
  ```

**根本原因：** 有 2 個可能的原因：

1. Azure Data Lake Store 已連結服務中指定的 `resourceGroupName` 和/或 `subscriptionId` 不正確；
2. 使用者或服務主體沒有所需的權限。

**解決方案：**

1. 確定您在已連結的服務 `typeProperties` 中指定的 `subscriptionId` 和 `resourceGroupName` 確實是您 Data Lake 帳戶所屬的項目。

2. 確定您至少將 Data Lake 帳戶上的 [讀取者] 角色授與使用者或服務主體。 以下是做法：

    1. 移至 Azure 入口網站 -> 您的 Data Lake Store 帳戶
    2. 按一下 Data Lake Store 刀鋒視窗上的 [存取控制 (IAM)]
    3. 按一下 [存取控制 (IAM)] 刀鋒視窗上的 [新增]
    4. 將 [角色] 設定為 [讀取者]，然後選取您用來執行複製的使用者或服務主體來授與存取權

3. 如果您不想將 [讀取者] 角色授與使用者或服務主體，替代方案是在複製活動中使用您 Data Lake Store 的位置來[明確指定執行位置](data-factory-data-movement-activities.md#global)。 範例：

    ```json
    {
      "name": "CopyToADLS",
      "type": "Copy",
      ......
      "typeProperties": {
        "source": {
          "type": "<source type>"
        },
        "sink": {
          "type": "AzureDataLakeStoreSink"
        },
        "exeuctionLocation": "West US"
      }
    }
    ```

## <a name="dataset-properties"></a>資料集屬性
若要指定資料集以代表 Data Lake Store 中的輸入資料，請將資料集的 type 屬性設定成 AzureDataLakeStore。 請將資料集的 linkedServiceName 屬性設定成 Data Lake Store 連結服務的名稱。 如需定義資料集的 JSON 區段和屬性完整清單，請參閱[建立資料集](data-factory-create-datasets.md)一文。 所有資料集類型 (例如 Azure SQL 資料庫、Azure Blob 及 Azure 資料表) 其 JSON 中如結構、可用性及原則等資料集區段都相似。 每個資料集類型的 typeProperties 區段都不同，並提供如資料存放區中資料位置與格式等相關資訊。 

類型為 AzureDataLakeStore  的資料集其 typeProperties 區段包含下列屬性：

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| **folderPath** |Data Lake Store 中容器與資料夾的路徑。 |yes |
| **fileName** |Azure Data Lake Store 中的檔案名稱。 fileName 屬性是選擇性的，而且區分大小寫。 <br/><br/>如果您指定 fileName，活動 (包括複製) 適用於特定的檔案。<br/><br/>如果您未指定 fileName，複製會在輸入資料集中包含 folderPath 中的所有檔案。<br/><br/>當未指定輸出資料集的 fileName，且在活動接收中未指定 preserveHierarchy 時，所產生檔案的名稱格式為 Data._Guid_.txt`。 例如：Data.0a405f8a-93ff-4c6f-b3be-f69616f1df7a.txt。 |否 |
| **partitionedBy** |partitionedBy 屬性為選擇性。 您可以用來指定時間序列資料的動態路徑與檔案名稱。 例如，folderPath 可針對每小時的資料進行參數化。 如需詳細資料和範例，請參閱 [partitionedBy 屬性](#using-partitionedby-property)一節。 |否 |
| **format** | 支援下列格式類型：TextFormat、JsonFormat、AvroFormat、OrcFormat 和 ParquetFormat。 將 [format] 下的 [type] 屬性設定為下列其中一個值。 如需詳細資訊，請參閱 [Azure Data Factory 支援的檔案與壓縮格式](data-factory-supported-file-and-compression-formats.md)一文中[文字格式](data-factory-supported-file-and-compression-formats.md#text-format)、[JSON 格式](data-factory-supported-file-and-compression-formats.md#json-format)、[Avro 格式](data-factory-supported-file-and-compression-formats.md#avro-format)、[ORC 格式](data-factory-supported-file-and-compression-formats.md#orc-format)及 [Parquet 格式](data-factory-supported-file-and-compression-formats.md#parquet-format)各節。 <br><br> 如果您想要在以檔案為基礎的存放區之間「依原樣」複製檔案 (二進位複本)，請略過輸入和輸出資料集定義中的 `format` 區段。 |否 |
| **compression** | 指定此資料的壓縮類型和層級。 支援的類型為：GZip、Deflate、BZip2 及 ZipDeflate。 支援的層級為 **Optimal** 和 **Fastest**。 如需詳細資訊，請參閱 [Azure Data Factory 支援的檔案與壓縮格式](data-factory-supported-file-and-compression-formats.md#compression-support)。 |否 |

### <a name="the-partitionedby-property"></a>partitionedBy 屬性
您可以使用 partitionedBy 屬性、Data Factory 函式及系統變數，來指定時間序列資料的動態 folderPath 和 fileName 屬性。 如需詳細資料，請參閱 [Azure Data Factory - 函式與系統變數](data-factory-functions-variables.md)一文。


在下列範例中，`{Slice}` 會取代成 Data Factory 系統變數 `SliceStart` 的值，並採用指定的格式 (`yyyyMMddHH`)。 `SliceStart` 名稱是指配量的開始時間。 每個配量的 `folderPath` 屬性皆不同，如 `wikidatagateway/wikisampledataout/2014100103` 或 `wikidatagateway/wikisampledataout/2014100104`。

```JSON
"folderPath": "wikidatagateway/wikisampledataout/{Slice}",
"partitionedBy":
[
    { "name": "Slice", "value": { "type": "DateTime", "date": "SliceStart", "format": "yyyyMMddHH" } },
],
```

在下列範例中，會將 `SliceStart` 的年、月、日和時間擷取至由 `folderPath` 與 `fileName` 屬性使用的個別變數：
```JSON
"folderPath": "wikidatagateway/wikisampledataout/{Year}/{Month}/{Day}",
"fileName": "{Hour}.csv",
"partitionedBy":
 [
    { "name": "Year", "value": { "type": "DateTime", "date": "SliceStart", "format": "yyyy" } },
    { "name": "Month", "value": { "type": "DateTime", "date": "SliceStart", "format": "MM" } },
    { "name": "Day", "value": { "type": "DateTime", "date": "SliceStart", "format": "dd" } },
    { "name": "Hour", "value": { "type": "DateTime", "date": "SliceStart", "format": "hh" } }
],
```
如需時間序列資料集、排程及配量的詳細資料，請參閱 [Azure Data Factory 中的資料集](data-factory-create-datasets.md)與 [Data Factory 排程與執行](data-factory-scheduling-and-execution.md)文章。 


## <a name="copy-activity-properties"></a>複製活動屬性
如需定義活動的可用區段和屬性完整清單，請參閱[建立管線](data-factory-create-pipelines.md)一文。 屬性 (例如名稱、描述、輸入和輸出資料表，以及原則) 適用於所有類型的活動。

活動之 [typeProperties] 區段中的可用屬性，會隨著每個活動類型而有所不同。 就複製活動而言，這些屬性會根據來源和接收器的類型而有所不同。

AzureDataLakeStoreSource 支援 [typeProperties] 區段中的下列屬性：

| 屬性 | 說明 | 允許的值 | 必要 |
| --- | --- | --- | --- |
| **recursive** |指出是否從子資料夾、或只有從指定的資料夾，以遞迴方式讀取資料。 |True (預設值)、False |否 |


AzureDataLakeStoreSink 支援 [typeProperties] 區段中的下列屬性：

| 屬性 | 說明 | 允許的值 | 必要 |
| --- | --- | --- | --- |
| **copyBehavior** |指定複製行為。 |<b>PreserveHierarchy</b>：保留目標資料夾中的檔案階層。 來源檔案到來源資料夾的相對路徑，與目標檔案到目標資料夾的相對路徑相同。<br/><br/><b>FlattenHierarchy</b>：來源資料夾的中所有檔案都會建立在目標資料夾的第一層中。 建立的目標檔案會具有自動產生的名稱。<br/><br/><b>MergeFiles</b>：將來源資料夾的所有檔案合併為一個檔案。 如果有指定檔案或 Blob 名稱，合併檔案的名稱會是指定的名稱。 否則，會自動產生檔案名稱。 |否 |

### <a name="recursive-and-copybehavior-examples"></a>遞迴和 copyBehavior 範例
本節說明遞迴和 copyBehavior 值在不同組合的情況下，複製作業所產生的行為。

| 遞迴 | copyBehavior | 產生的行為 |
| --- | --- | --- |
| true |preserveHierarchy |對於有下列結構的來源資料夾 Folder1： <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5<br/><br/>會以與來源相同的結構，建立目標資料夾 Folder1<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5 |
| true |flattenHierarchy |對於有下列結構的來源資料夾 Folder1： <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5<br/><br/>會以下列結構建立目標資料夾 1： <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1 有自動產生的名稱<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2 有自動產生的名稱<br/>&nbsp;&nbsp;&nbsp;&nbsp;File3 有自動產生的名稱<br/>&nbsp;&nbsp;&nbsp;&nbsp;File4 有自動產生的名稱<br/>&nbsp;&nbsp;&nbsp;&nbsp;File5 有自動產生的名稱 |
| true |mergeFiles |對於有下列結構的來源資料夾 Folder1： <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5<br/><br/>會以下列結構建立目標資料夾 1： <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1 + File2 + File3 + File4 + File 5 的內容會合併成一個檔案，並有自動產生的檔案名稱 |
| false |preserveHierarchy |對於有下列結構的來源資料夾 Folder1： <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5<br/><br/>會以下列結構建立目標資料夾 Folder1<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/><br/><br/>系統不會挑選含有 File3、File4 和 File5 的 Subfolder1。 |
| false |flattenHierarchy |對於有下列結構的來源資料夾 Folder1：<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5<br/><br/>會以下列結構建立目標資料夾 Folder1<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1 有自動產生的名稱<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2 有自動產生的名稱<br/><br/><br/>系統不會挑選含有 File3、File4 和 File5 的 Subfolder1。 |
| false |mergeFiles |對於有下列結構的來源資料夾 Folder1：<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5<br/><br/>會以下列結構建立目標資料夾 Folder1<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;File1 + File2 的內容會合併成一個檔案，並有自動產生的檔案名稱。 File1 有自動產生的名稱<br/><br/>系統不會挑選含有 File3、File4 和 File5 的 Subfolder1。 |

## <a name="supported-file-and-compression-formats"></a>支援的檔案和壓縮格式
如需詳細資料，請參閱 [Azure Data Factory 中的檔案和壓縮格式](data-factory-supported-file-and-compression-formats.md)一文。

## <a name="json-examples-for-copying-data-to-and-from-data-lake-store"></a>從 Data Lake Store 來回複製資料的 JSON 範例
下列範例提供範例 JSON 定義。 您可以使用這些範例定義，透過使用 [Azure 入口網站](data-factory-copy-activity-tutorial-using-azure-portal.md)、[Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md) 或 [Azure PowerShell](data-factory-copy-activity-tutorial-using-powershell.md) 來建立管線。 這些範例會示範如何從 Data Lake Store 與 Azure Blob 儲存體來回複製資料。 不過，您可以將資料從任何來源_直接_複製到任何支援的接收器。 如需詳細資訊，請參閱[使用複製活動來移動資料](data-factory-data-movement-activities.md)中的＜支援的資料存放區和格式＞一節。  

### <a name="example-copy-data-from-azure-blob-storage-to-azure-data-lake-store"></a>範例：將資料從 Azure Blob 儲存體複製到 Azure Data Lake Store
本節中的範例程式碼顯示︰

* [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties)類型的連結服務。
* [AzureDataLakeStore](#linked-service-properties)類型的連結服務。
* [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties) 類型的輸入[資料集](data-factory-create-datasets.md)。
* [AzureDataLakeStore](#dataset-properties) 類型的輸出[資料集](data-factory-create-datasets.md)。
* 具有使用 [BlobSource](data-factory-azure-blob-connector.md#copy-activity-properties) 和 [AzureDataLakeStoreSink](#copy-activity-properties) 之複製活動的[管線](data-factory-create-pipelines.md)。

此範例示範如何每小時將時間序列資料從 Azure Blob 儲存體複製到 Data Lake Store。 

**Azure 儲存體連結服務**

```JSON
{
  "name": "StorageLinkedService",
  "properties": {
    "type": "AzureStorage",
    "typeProperties": {
      "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
    }
  }
}
```

**Azure Data Lake Store 已連結的服務**

```JSON
{
    "name": "AzureDataLakeStoreLinkedService",
    "properties": {
        "type": "AzureDataLakeStore",
        "typeProperties": {
            "dataLakeStoreUri": "https://<accountname>.azuredatalakestore.net/webhdfs/v1",
            "servicePrincipalId": "<service principal id>",
            "servicePrincipalKey": "<service principal key>",
            "tenant": "<tenant info, e.g. microsoft.onmicrosoft.com>",
            "subscriptionId": "<subscription of ADLS>",
            "resourceGroupName": "<resource group of ADLS>"
        }
    }
}
```

> [!NOTE]
> 如需組態詳細資料，請參閱[連結服務屬性](#linked-service-properties)一節。
>

**Azure Blob 輸入資料集**

在下列範例中，每小時 (`"frequency": "Hour", "interval": 1`) 會從新的 Blob 中挑選資料。 根據正在處理之配量的開始時間，以動態方式評估 Blob 的資料夾路徑和檔案名稱。 資料夾路徑會使用開始時間的年、月和日部分。 檔案名稱會使用開始時間的小時部分。 `"external": true` 設定會通知 Data Factory 服務：這是資料處理站外部的資料表，且不是由資料處理站中的活動所產生。

```JSON
{
  "name": "AzureBlobInput",
  "properties": {
    "type": "AzureBlob",
    "linkedServiceName": "StorageLinkedService",
    "typeProperties": {
      "folderPath": "mycontainer/myfolder/yearno={Year}/monthno={Month}/dayno={Day}",
      "partitionedBy": [
        {
          "name": "Year",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "yyyy"
          }
        },
        {
          "name": "Month",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "MM"
          }
        },
        {
          "name": "Day",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "dd"
          }
        },
        {
          "name": "Hour",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "HH"
          }
        }
      ]
    },
    "external": true,
    "availability": {
      "frequency": "Hour",
      "interval": 1
    },
    "policy": {
      "externalData": {
        "retryInterval": "00:01:00",
        "retryTimeout": "00:10:00",
        "maximumRetry": 3
      }
    }
  }
}
```

**Azure Data Lake Store 輸出資料集**

下列範例會將資料複製到 Data Lake Store。 每個小時會將新資料複製到 Data Lake Store。

```JSON
{
    "name": "AzureDataLakeStoreOutput",
      "properties": {
        "type": "AzureDataLakeStore",
        "linkedServiceName": "AzureDataLakeStoreLinkedService",
        "typeProperties": {
            "folderPath": "datalake/output/"
        },
        "availability": {
              "frequency": "Hour",
              "interval": 1
        }
      }
}
```


**具有 Blob 來源和 Data Lake Store 接收器的管線中的複製活動**

在下列範例中，管線包含複製活動，該活動已設定為使用輸入和輸出資料集。 複製活動排程為每小時執行一次。 在管線 JSON 定義中，`source` 類型設為 `BlobSource`，`sink` 類型設為 `AzureDataLakeStoreSink`。

```json
{  
    "name":"SamplePipeline",
    "properties":
    {  
        "start":"2014-06-01T18:00:00",
        "end":"2014-06-01T19:00:00",
        "description":"pipeline with copy activity",
        "activities":
        [  
              {
                "name": "AzureBlobtoDataLake",
                "description": "Copy Activity",
                "type": "Copy",
                "inputs": [
                  {
                    "name": "AzureBlobInput"
                  }
                ],
                "outputs": [
                  {
                    "name": "AzureDataLakeStoreOutput"
                  }
                ],
                "typeProperties": {
                    "source": {
                        "type": "BlobSource"
                      },
                      "sink": {
                        "type": "AzureDataLakeStoreSink"
                      }
                },
                   "scheduler": {
                      "frequency": "Hour",
                      "interval": 1
                },
                "policy": {
                      "concurrency": 1,
                      "executionPriorityOrder": "OldestFirst",
                      "retry": 0,
                      "timeout": "01:00:00"
                }
              }
        ]
    }
}
```

### <a name="example-copy-data-from-azure-data-lake-store-to-an-azure-blob"></a>範例：將資料從 Azure Data Lake Store 複製到 Azure Blob
本節中的範例程式碼顯示︰

* [AzureDataLakeStore](#linked-service-properties)類型的連結服務。
* [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties)類型的連結服務。
* [AzureDataLakeStore](#dataset-properties) 類型的輸入[資料集](data-factory-create-datasets.md)。
* [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties) 類型的輸出[資料集](data-factory-create-datasets.md)。
* 具有使用 [AzureDataLakeStoreSource](#copy-activity-properties) 和 [BlobSink](data-factory-azure-blob-connector.md#copy-activity-properties) 之複製活動的[管線](data-factory-create-pipelines.md)。

該程式碼每小時會將時間序列資料從 Data Lake Store 複製到 Azure Blob。 

**Azure Data Lake Store 已連結的服務**

```json
{
    "name": "AzureDataLakeStoreLinkedService",
    "properties": {
        "type": "AzureDataLakeStore",
        "typeProperties": {
            "dataLakeStoreUri": "https://<accountname>.azuredatalakestore.net/webhdfs/v1",
            "servicePrincipalId": "<service principal id>",
            "servicePrincipalKey": "<service principal key>",
            "tenant": "<tenant info, e.g. microsoft.onmicrosoft.com>"
        }
    }
}
```

> [!NOTE]
> 如需組態詳細資料，請參閱[連結服務屬性](#linked-service-properties)一節。
>

**Azure 儲存體連結服務**

```JSON
{
  "name": "StorageLinkedService",
  "properties": {
    "type": "AzureStorage",
    "typeProperties": {
      "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
    }
  }
}
```
**Azure Data Lake 輸入資料集**

在此範例中，將 `"external"` 設定為 `true` 會通知 Data Factory 服務：這是資料處理站外部的資料表，而不是由資料處理站中的活動所產生。

```json
{
    "name": "AzureDataLakeStoreInput",
      "properties":
    {
        "type": "AzureDataLakeStore",
        "linkedServiceName": "AzureDataLakeStoreLinkedService",
        "typeProperties": {
            "folderPath": "datalake/input/",
            "fileName": "SearchLog.tsv",
            "format": {
                "type": "TextFormat",
                "rowDelimiter": "\n",
                "columnDelimiter": "\t"
            }
        },
        "external": true,
        "availability": {
            "frequency": "Hour",
              "interval": 1
        },
        "policy": {
              "externalData": {
                "retryInterval": "00:01:00",
                "retryTimeout": "00:10:00",
                "maximumRetry": 3
              }
        }
      }
}
```
**Azure Blob 輸出資料集**

在下列範例中，每小時 (`"frequency": "Hour", "interval": 1`) 會將資料寫入新的 Blob。 根據正在處理之配量的開始時間，以動態方式評估 Blob 的資料夾路徑。 此資料夾路徑會使用開始時間的年、月、日和小時部分。

```JSON
{
  "name": "AzureBlobOutput",
  "properties": {
    "type": "AzureBlob",
    "linkedServiceName": "StorageLinkedService",
    "typeProperties": {
      "folderPath": "mycontainer/myfolder/yearno={Year}/monthno={Month}/dayno={Day}/hourno={Hour}",
      "partitionedBy": [
        {
          "name": "Year",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "yyyy"
          }
        },
        {
          "name": "Month",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "MM"
          }
        },
        {
          "name": "Day",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "dd"
          }
        },
        {
          "name": "Hour",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "HH"
          }
        }
      ],
      "format": {
        "type": "TextFormat",
        "columnDelimiter": "\t",
        "rowDelimiter": "\n"
      }
    },
    "availability": {
      "frequency": "Hour",
      "interval": 1
    }
  }
}
```

**具有 Azure Data Lake Store 來源和 Blob 接收器的管線中的複製活動**

在下列範例中，管線包含複製活動，該活動已設定為使用輸入和輸出資料集。 複製活動排程為每小時執行一次。 在管線 JSON 定義中，`source` 類型設為 `AzureDataLakeStoreSource`，`sink` 類型設為 `BlobSink`。

```json
{  
    "name":"SamplePipeline",
    "properties":{  
        "start":"2014-06-01T18:00:00",
        "end":"2014-06-01T19:00:00",
        "description":"pipeline for copy activity",
        "activities":[  
              {
                "name": "AzureDakeLaketoBlob",
                "description": "copy activity",
                "type": "Copy",
                "inputs": [
                  {
                    "name": "AzureDataLakeStoreInput"
                  }
                ],
                "outputs": [
                  {
                    "name": "AzureBlobOutput"
                  }
                ],
                "typeProperties": {
                    "source": {
                        "type": "AzureDataLakeStoreSource",
                      },
                      "sink": {
                        "type": "BlobSink"
                      }
                },
                   "scheduler": {
                      "frequency": "Hour",
                      "interval": 1
                },
                "policy": {
                      "concurrency": 1,
                      "executionPriorityOrder": "OldestFirst",
                      "retry": 0,
                      "timeout": "01:00:00"
                }
              }
         ]
    }
}
```

在複製活動定義中，您也可以將來源資料集的資料行對應至接收資料集中的資料行。 如需詳細資料，請參閱[在 Azure Data Factory 中對應資料集資料行](data-factory-map-columns.md)。

## <a name="performance-and-tuning"></a>效能和微調
若要了解影響「複製活動」效能的因素，以及最佳化的方法，請參閱[複製活動的效能及微調指南](data-factory-copy-activity-performance.md)一文。
