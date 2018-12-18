---
title: 使用 Azure 儲存體安全存取雲端中的應用程式資料 | Microsoft Docs
description: 使用 SAS 權杖、加密和 HTTPS 保護雲端中的應用程式資料。
services: storage
author: tamram
ms.service: storage
ms.topic: tutorial
ms.date: 05/30/2018
ms.author: tamram
ms.custom: mvc
ms.openlocfilehash: 71eec62f4468b6b74a15a30be3e472b41c4d45b0
ms.sourcegitcommit: d4c076beea3a8d9e09c9d2f4a63428dc72dd9806
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/01/2018
ms.locfileid: "39397766"
---
# <a name="secure-access-to-an-applications-data-in-the-cloud"></a>安全存取雲端中的應用程式資料

本教學課程是一個系列的第三部分。 您了解如何安全存取儲存體帳戶。 

在系列的第三部分中，您將了解如何：

> [!div class="checklist"]
> * 使用 SAS 權杖存取縮圖影像
> * 開啟伺服器端加密
> * 只啟用 HTTPS 進行傳輸

[Azure Blob 儲存體](../common/storage-introduction.md#blob-storage)提供強固的服務來儲存應用程式的檔案。 本教學課程延伸[上一個主題][previous-tutorial]，說明如何從 Web 應用程式安全存取您的儲存體帳戶。 當您完成時，影像會經過加密，而 Web 應用程式會使用安全的 SAS 權杖來存取縮圖影像。

## <a name="prerequisites"></a>必要條件

若要完成本教學課程，您必須已完成上一個儲存體教學課程：[使用 Event Grid 自動調整已上傳影像的大小][previous-tutorial]。 

## <a name="set-container-public-access"></a>設定容器的公用存取

在教學課程系列的這個部分中，使用 SAS 權杖來存取縮圖。 在此步驟中，您會將 _thumbnails_ 容器的公用存取設為 `off`。

```azurecli-interactive 
blobStorageAccount=<blob_storage_account>

blobStorageAccountKey=$(az storage account keys list -g myResourceGroup \
-n $blobStorageAccount --query [0].value --output tsv) 

az storage container set-permission \ --account-name $blobStorageAccount \ --account-key $blobStorageAccountKey \ --name thumbnails  \
--public-access off
``` 

## <a name="configure-sas-tokens-for-thumbnails"></a>設定縮圖的 SAS 權杖

在此教學課程系列的第一部分中，Web 應用程式會顯示公用容器中的影像。 在系列的這個部分中，您會使用[共用存取簽章 (SAS)](../common/storage-dotnet-shared-access-signature-part-1.md#what-is-a-shared-access-signature) 權杖來擷取縮圖影像。 SAS 權杖可讓您根據 IP、通訊協定、時間間隔或允許的權限，來限制存取容器或 Blob。

在此範例中，原始程式碼存放庫使用具有已更新程式碼範例的 `sasTokens` 分支。 使用 [az webapp deployment source delete](/cli/azure/webapp/deployment/source#az_webapp_deployment_source_delete) 刪除現有的 GitHub 部署。 接著，使用 [az webapp deployment source config](/cli/azure/webapp/deployment/source#az_webapp_deployment_source_config) 命令設定 Web 應用程式的 GitHub 部署。  

在下列命令中，`<web-app>` 是您 Web 應用程式的名稱。  

```azurecli-interactive 
az webapp deployment source delete --name <web-app> --resource-group myResourceGroup

az webapp deployment source config --name <web_app> \
--resource-group myResourceGroup --branch sasTokens --manual-integration \
--repo-url https://github.com/Azure-Samples/storage-blob-upload-from-webapp
``` 

存放庫的 `sasTokens` 分支會更新 `StorageHelper.cs` 檔案。 它會將 `GetThumbNailUrls` 工作取代為下列程式碼範例。 更新的工作會藉由設定 [SharedAccessBlobPolicy](/dotnet/api/microsoft.windowsazure.storage.blob.sharedaccessblobpolicy?view=azure-dotnet) 來指定 SAS 權杖的開始時間、到期時間和權限，以擷取縮圖 URL。 部署後，Web 應用程式現在會使用 SAS 權杖擷取具有 URL 的縮圖。 下列範例顯示更新的工作：
    
```csharp
public static async Task<List<string>> GetThumbNailUrls(AzureStorageConfig _storageConfig)
{
    List<string> thumbnailUrls = new List<string>();

    // Create storagecredentials object by reading the values from the configuration (appsettings.json)
    StorageCredentials storageCredentials = new StorageCredentials(_storageConfig.AccountName, _storageConfig.AccountKey);

    // Create cloudstorage account by passing the storagecredentials
    CloudStorageAccount storageAccount = new CloudStorageAccount(storageCredentials, true);

    // Create blob client
    CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

    // Get reference to the container
    CloudBlobContainer container = blobClient.GetContainerReference(_storageConfig.ThumbnailContainer);

    BlobContinuationToken continuationToken = null;

    BlobResultSegment resultSegment = null;

    //Call ListBlobsSegmentedAsync and enumerate the result segment returned, while the continuation token is non-null.
    //When the continuation token is null, the last page has been returned and execution can exit the loop.
    do
    {
        //This overload allows control of the page size. You can return all remaining results by passing null for the maxResults parameter,
        //or by calling a different overload.
        resultSegment = await container.ListBlobsSegmentedAsync("", true, BlobListingDetails.All, 10, continuationToken, null, null);

        foreach (var blobItem in resultSegment.Results)
        {
            CloudBlockBlob blob = blobItem as CloudBlockBlob;
            //Set the expiry time and permissions for the blob.
            //In this case, the start time is specified as a few minutes in the past, to mitigate clock skew.
            //The shared access signature will be valid immediately.
            SharedAccessBlobPolicy sasConstraints = new SharedAccessBlobPolicy();

            sasConstraints.SharedAccessStartTime = DateTimeOffset.UtcNow.AddMinutes(-5);

            sasConstraints.SharedAccessExpiryTime = DateTimeOffset.UtcNow.AddHours(24);

            sasConstraints.Permissions = SharedAccessBlobPermissions.Read;

            //Generate the shared access signature on the blob, setting the constraints directly on the signature.
            string sasBlobToken = blob.GetSharedAccessSignature(sasConstraints);

            //Return the URI string for the container, including the SAS token.
            thumbnailUrls.Add(blob.Uri + sasBlobToken);

        }

        //Get the continuation token.
        continuationToken = resultSegment.ContinuationToken;
    }

    while (continuationToken != null);

    return await Task.FromResult(thumbnailUrls);
}
```

以下是之前的工作所使用的類別、屬性和方法：

|類別  |properties| 方法  |
|---------|---------|---------|
|[StorageCredentials](/dotnet/api/microsoft.windowsazure.storage.auth.storagecredentials?view=azure-dotnet)    |         |
|[CloudStorageAccount](/dotnet/api/microsoft.windowsazure.storage.cloudstorageaccount?view=azure-dotnet)     | |[CreateCloudBlobClient](/dotnet/api/microsoft.windowsazure.storage.cloudstorageaccount.createcloudblobclient?view=azure-dotnet#Microsoft_WindowsAzure_Storage_CloudStorageAccount_CreateCloudBlobClient)        |
|[CloudBlobClient](/dotnet/api/microsoft.windowsazure.storage.blob.cloudblobclient?view=azure-dotnet)     | |[GetContainerReference](/dotnet/api/microsoft.windowsazure.storage.blob.cloudblobclient.getcontainerreference?view=azure-dotnet#Microsoft_WindowsAzure_Storage_Blob_CloudBlobClient_GetContainerReference_System_String_)         |
|[CloudBlobContainer](/dotnet/api/microsoft.windowsazure.storage.blob.cloudblobcontainer?view=azure-dotnet)     | |[SetPermissionsAsync](/dotnet/api/microsoft.windowsazure.storage.blob.cloudblobcontainer.setpermissionsasync?view=azure-dotnet#Microsoft_WindowsAzure_Storage_Blob_CloudBlobContainer_SetPermissionsAsync_Microsoft_WindowsAzure_Storage_Blob_BlobContainerPermissions_) <br> [ListBlobsSegmentedAsync](/dotnet/api/microsoft.windowsazure.storage.blob.cloudblobcontainer.listblobssegmentedasync?view=azure-dotnet#Microsoft_WindowsAzure_Storage_Blob_CloudBlobContainer_ListBlobsSegmentedAsync_System_String_System_Boolean_Microsoft_WindowsAzure_Storage_Blob_BlobListingDetails_System_Nullable_System_Int32__Microsoft_WindowsAzure_Storage_Blob_BlobContinuationToken_Microsoft_WindowsAzure_Storage_Blob_BlobRequestOptions_Microsoft_WindowsAzure_Storage_OperationContext_)       |
|[BlobContinuationToken](/dotnet/api/microsoft.windowsazure.storage.blob.blobcontinuationtoken?view=azure-dotnet)     |         |
|[BlobResultSegment](/dotnet/api/microsoft.windowsazure.storage.blob.blobresultsegment?view=azure-dotnet)    | [結果](/dotnet/api/microsoft.windowsazure.storage.blob.blobresultsegment.results?view=azure-dotnet#Microsoft_WindowsAzure_Storage_Blob_BlobResultSegment_Results)         |
|[CloudBlockBlob](/dotnet/api/microsoft.windowsazure.storage.blob.cloudblockblob?view=azure-dotnet)    |         | [GetSharedAccessSignature](/dotnet/api/microsoft.windowsazure.storage.blob.cloudblob.getsharedaccesssignature?view=azure-dotnet#Microsoft_WindowsAzure_Storage_Blob_CloudBlob_GetSharedAccessSignature_Microsoft_WindowsAzure_Storage_Blob_SharedAccessBlobPolicy_)
|[SharedAccessBlobPolicy](/dotnet/api/microsoft.windowsazure.storage.blob.sharedaccessblobpolicy?view=azure-dotnet)     | [SharedAccessStartTime](/dotnet/api/microsoft.windowsazure.storage.blob.sharedaccessblobpolicy.sharedaccessstarttime?view=azure-dotnet#Microsoft_WindowsAzure_Storage_Blob_SharedAccessBlobPolicy_SharedAccessStartTime)<br>[SharedAccessExpiryTime](/dotnet/api/microsoft.windowsazure.storage.blob.sharedaccessblobpolicy.sharedaccessexpirytime?view=azure-dotnet#Microsoft_WindowsAzure_Storage_Blob_SharedAccessBlobPolicy_SharedAccessExpiryTime)<br>[權限](/dotnet/api/microsoft.windowsazure.storage.blob.sharedaccessblobpolicy.permissions?view=azure-dotnet#Microsoft_WindowsAzure_Storage_Blob_SharedAccessBlobPolicy_Permissions) |        |

## <a name="server-side-encryption"></a>伺服器端加密

[Azure 儲存體服務加密 (SSE)](../common/storage-service-encryption.md) 可協助您保護及防護資料。 SSE 會加密待用資料，並處理加密、解密和金鑰管理。 所有資料都使用 256 位元 [AES 加密](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard) (可用的最強區塊加密方式之一) 進行加密。

SSE 會自動將所有效能層 (標準和進階)、所有部署模型 (Azure Resource Manager 和傳統) 以及所有「Azure 儲存體」服務 (Blob、佇列、資料表和檔案) 中的資料加密。 

## <a name="enable-https-only"></a>只啟用 HTTPS

為了確保在儲存體帳戶之間傳輸資料的要求安全，您可以僅限 HTTPS 要求。 使用 [az storage account update](/cli/azure/storage/account#az_storage_account_update) 命令更新儲存體帳戶所需的通訊協定。

```azurecli-interactive
az storage account update --resource-group myresourcegroup --name <storage-account-name> --https-only true
```

利用 `curl` 測試使用 `HTTP` 通訊協定的連線。

```azurecli-interactive
curl http://<storage-account-name>.blob.core.windows.net/<container>/<blob-name> -I
```

要求安全傳輸之後，您會收到下列訊息：

```
HTTP/1.1 400 The account being accessed does not support http.
```

## <a name="next-steps"></a>後續步驟

在系列的第三部分中，您已了解如何安全存取儲存體帳戶，像是如何：

> [!div class="checklist"]
> * 使用 SAS 權杖存取縮圖影像
> * 開啟伺服器端加密
> * 只啟用 HTTPS 進行傳輸

請繼續進行系列的第四部分，以了解如何監視雲端儲存體應用程式並對其進行疑難排解。

> [!div class="nextstepaction"]
> [監視雲端儲存體應用程式並對其進行疑難排解](storage-monitor-troubleshoot-storage-application.md)

[previous-tutorial]: ../../event-grid/resize-images-on-storage-blob-upload-event.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json