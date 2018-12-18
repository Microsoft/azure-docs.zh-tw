---
title: 將 blob 從儲存體帳戶複製到 Azure 媒體服務資產 | Microsoft Docs
description: 本主題說明如何將現有的 blob 複製到媒體服務資產。 下列範例會使用 Azure 媒體服務 .NET SDK 延伸模組。
services: media-services
documentationcenter: ''
author: Juliako
manager: cfowler
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: ne
ms.topic: article
ms.date: 12/09/2017
ms.author: juliako
ms.openlocfilehash: 9305b3cb810af9f0653d980328c46e41a540bf1a
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33783387"
---
# <a name="copying-existing-blobs-into-a-media-services-asset"></a>將現有的 blob 複製到媒體服務資產
本文章說明如何使用 [Azure 媒體服務 .NET SDK 延伸模組](https://github.com/Azure/azure-sdk-for-media-services-extensions/)將 Blob 從儲存體帳戶複製到新的 Azure 媒體服務 (AMS) 資產。

此擴充方法適用於：

- 一般資產。
- 即時封存資產 (FragBlob 格式)。
- 屬於不同媒體服務帳戶 (甚至跨不同資料中心) 的來源和目的地資產。 不過，這麼做可能會向您索取費用。 如需有關定價的詳細資訊，請參閱 [資料傳輸](https://azure.microsoft.com/pricing/#header-11)。

> [!NOTE]
> 請勿在不使用媒體服務 API 的情況下，嘗試變更由媒體服務所產生的 blob 容器內容。
> 

此文章顯示兩個程式碼範例：

1. 將 Blob 從一個 AMS 帳戶中的資產複製到另一個 AMS 帳戶中的新資產。
2. 將 Blob 從某些儲存體帳戶複製到 AMS 帳戶中的新資產。

## <a name="copy-blobs-between-two-ams-accounts"></a>在兩個 AMS 帳戶之間複製 Blob  

### <a name="prerequisites"></a>先決條件

兩個媒體服務帳戶。 請參閱文章[如何建立媒體服務帳戶](media-services-portal-create-account.md)。

### <a name="download-sample"></a>下載範例
您可以遵循本文中的步驟，或者您可以從[這裡](https://azure.microsoft.com/documentation/samples/media-services-dotnet-copy-blob-into-asset/)下載包含本文所述的程式碼範例。

### <a name="set-up-your-project"></a>設定專案

1. 設定您的開發環境，如[使用 .NET 進行媒體服務開發](media-services-dotnet-how-to-use.md)中所述。 
2. 將 appSettings 區段新增至 .config 檔中，並根據媒體服務帳戶、目的地儲存體帳戶及來源資產識別碼將值更新。  

```xml
<appSettings>
    <add key="AMSSourceAADTenantDomain" value="tenant"/>
    <add key="AMSSourceRESTAPIEndpoint" value="endpoint"/>

    <add key="SourceAMSClientId" value="clientID"/>
    <add key="SourceAMSClientSecret" value="clientSecret"/>

    <add key="SourceAssetID" value="nb:cid:UUID:<id>"/>

    <add key="AMSDestAADTenantDomain" value="tenant"/>
    <add key="AMSDestRESTAPIEndpoint" value="endpoint"/>

    <add key="DestAMSClientId" value="clientID"/>
    <add key="DestAMSClientSecret" value="clientSecret"/>

    <add key="DestStorageAccountName" value="name"/>
    <add key="DestStorageAccountKey" value="key"/>

</appSettings>
```

### <a name="copy-blobs-from-an-asset-in-one-ams-account-into-an-asset-in-another-ams-account"></a>將 Blob 從一個 AMS 帳戶中的資產複製到另一個 AMS 帳戶中的資產

下列程式碼使用延伸模組 **IAsset.Copy**方法，將來源資產中的所有檔案都複製到使用單個延伸模組的目的地資產。

```csharp
using System;
using Microsoft.WindowsAzure.MediaServices.Client;
using System.Linq;
using System.Configuration;
using Microsoft.WindowsAzure.Storage.Auth;

namespace CopyExistingBlobsIntoAsset
{
    class Program
    {
        static string _sourceAADTenantDomain = 
            ConfigurationManager.AppSettings["AMSSourceAADTenantDomain"];
        static string _sourceRESTAPIEndpoint = 
            ConfigurationManager.AppSettings["AMSSourceRESTAPIEndpoint"];
        static string _sourceClientId = 
            ConfigurationManager.AppSettings["SourceAMSClientId"];
        static string _sourceClientSecret = 
            ConfigurationManager.AppSettings["SourceAMSClientSecret"];

        static string _destAADTenantDomain = 
            ConfigurationManager.AppSettings["AMSDestAADTenantDomain"];
        static string _destRESTAPIEndpoint = 
            ConfigurationManager.AppSettings["AMSDestRESTAPIEndpoint"];
        static string _destClientId = 
            ConfigurationManager.AppSettings["DestAMSClientId"];
        static string _destClientSecret = 
            ConfigurationManager.AppSettings["DestAMSClientSecret"];

        static string _destStorageAccountName = 
            ConfigurationManager.AppSettings["DestStorageAccountName"];
        static string _destStorageAccountKey = 
            ConfigurationManager.AppSettings["DestStorageAccountKey"];
        static string _sourceAssetID = 
            ConfigurationManager.AppSettings["SourceAssetID"];

        private static CloudMediaContext _sourceContext = null;
        private static CloudMediaContext _destContext = null;

        static void Main(string[] args)
        {
            AzureAdTokenCredentials tokenCredentials1 = new AzureAdTokenCredentials(_sourceAADTenantDomain,
                   new AzureAdClientSymmetricKey(_sourceClientId, _sourceClientSecret),
                   AzureEnvironments.AzureCloudEnvironment);

            AzureAdTokenCredentials tokenCredentials2 = new AzureAdTokenCredentials(_destAADTenantDomain,
                   new AzureAdClientSymmetricKey(_destClientId, _destClientSecret),
                   AzureEnvironments.AzureCloudEnvironment);

            var tokenProvider1 = new AzureAdTokenProvider(tokenCredentials1);
            var tokenProvider2 = new AzureAdTokenProvider(tokenCredentials2);

            // Create the context for your source Media Services account.
            _sourceContext = new CloudMediaContext(new Uri(_sourceRESTAPIEndpoint), tokenProvider1);

            // Create the context for your destination Media Services account.
            _destContext = new CloudMediaContext(new Uri(_destRESTAPIEndpoint), tokenProvider2);

            // Get the credentials of the default Storage account bound to your destination Media Services account.
            StorageCredentials destinationStorageCredentials =
                new StorageCredentials(_destStorageAccountName, _destStorageAccountKey);

            // Get a reference to the source asset in the source context.
            IAsset sourceAsset = _sourceContext.Assets.Where(asset => asset.Id == _sourceAssetID).First();

            // Create an empty destination asset in the destination context.
            IAsset destinationAsset = _destContext.Assets.Create(sourceAsset.Name, AssetCreationOptions.None);

            // Copy the files in the source asset instance into the destination asset instance.
            // There is an additional overload with async support.
            sourceAsset.Copy(destinationAsset, destinationStorageCredentials);

            Console.WriteLine("Done");
        }
    }
}
```

## <a name="copy-blobs-from-a-storage-account-into-an-ams-account"></a>從儲存體帳戶將 Blob 複製到 AMS 帳戶 

### <a name="prerequisites"></a>先決條件

- 您從中複製 Blob 的來源儲存體帳戶。
- 您要將 Blob 複製到其中的 AMS 帳戶。

### <a name="set-up-your-project"></a>設定專案

1. 設定您的開發環境，如[使用 .NET 進行媒體服務開發](media-services-dotnet-how-to-use.md)中所述。 
2. 將 appSettings 區段新增至 .config 檔案，並根據您的來源儲存體和目的地 AMS 帳戶將值更新。

```xml
<appSettings>
    <add key="SourceStorageAccountName" value="name" />
    <add key="SourceStorageAccountKey" value="key" />
    <add key="NameOfBlobContainerYouWantToCopy" value="BlobContainerName"/>
    
    <add key="AMSAADTenantDomain" value="tenant"/>
    <add key="AMSRESTAPIEndpoint" value="endpoint"/>
    <add key="AMSClientId" value="clientID"/>
    <add key="AMSClientSecret" value="clientSecret"/>
    <add key="AMSStorageAccountName" value="name"/>
    <add key="AMSStorageAccountKey" value="key"/>
</appSettings>
```

### <a name="copy-blobs-from-some-storage-account-into-a-new-asset-in-an-ams-account"></a>將 Blob 從某些儲存體帳戶複製到 AMS 帳戶中的新資產

下列程式碼會將 Blob 從儲存體帳戶複製到媒體服務資產。 

>[!NOTE]
>對於不同的 AMS 原則 (例如 Locator 原則或 ContentKeyAuthorizationPolicy) 有 1,000,000 個原則的限制。 如果您一律使用相同的日期 / 存取權限，例如，要長時間維持就地 (非上載原則) 的定位器原則，您應該使用相同的原則識別碼。 如需詳細資訊，請參閱[本篇文章](media-services-dotnet-manage-entities.md#limit-access-policies)。

```csharp
using System;
using System.Configuration;
using System.Linq;
using Microsoft.WindowsAzure.MediaServices.Client;
using Microsoft.WindowsAzure.Storage.Auth;
using Microsoft.WindowsAzure.Storage;
using Microsoft.WindowsAzure.Storage.Blob;
    
namespace CopyExistingBlobsIntoAsset
{
    class Program
    {
        // Read values from the App.config file.
        private static readonly string _sourceStorageAccountName =
            ConfigurationManager.AppSettings["SourceStorageAccountName"];
        private static readonly string _sourceStorageAccountKey =
            ConfigurationManager.AppSettings["SourceStorageAccountKey"];
        private static readonly string _NameOfBlobContainerYouWantToCopy =
            ConfigurationManager.AppSettings["NameOfBlobContainerYouWantToCopy"];

        private static readonly string _AMSAADTenantDomain =
            ConfigurationManager.AppSettings["AMSAADTenantDomain"];
        private static readonly string _AMSRESTAPIEndpoint =
            ConfigurationManager.AppSettings["AMSRESTAPIEndpoint"];
        private static readonly string _AMSClientId =
            ConfigurationManager.AppSettings["AMSClientId"];
        private static readonly string _AMSClientSecret =
            ConfigurationManager.AppSettings["AMSClientSecret"];
        private static readonly string _AMSStorageAccountName =
            ConfigurationManager.AppSettings["AMSStorageAccountName"];
        private static readonly string _AMSStorageAccountKey =
            ConfigurationManager.AppSettings["AMSStorageAccountKey"];

        // Field for service context.
        private static CloudMediaContext _context = null;
        private static CloudStorageAccount _sourceStorageAccount = null;
        private static CloudStorageAccount _destinationStorageAccount = null;

        static void Main(string[] args)
        {
            AzureAdTokenCredentials tokenCredentials = new AzureAdTokenCredentials(_AMSAADTenantDomain,
               new AzureAdClientSymmetricKey(_AMSClientId, _AMSClientSecret),
               AzureEnvironments.AzureCloudEnvironment);
            
            var tokenProvider = new AzureAdTokenProvider(tokenCredentials);

            // Create the context for your source Media Services account.
            _context = new CloudMediaContext(new Uri(_AMSRESTAPIEndpoint), tokenProvider);

            _sourceStorageAccount =
                new CloudStorageAccount(new StorageCredentials(_sourceStorageAccountName,
                    _sourceStorageAccountKey), true);

            _destinationStorageAccount =
                new CloudStorageAccount(new StorageCredentials(_AMSStorageAccountName,
                    _AMSStorageAccountKey), true);

            CloudBlobClient sourceCloudBlobClient =
                _sourceStorageAccount.CreateCloudBlobClient();
            CloudBlobContainer sourceContainer =
                sourceCloudBlobClient.GetContainerReference(_NameOfBlobContainerYouWantToCopy);

            CreateAssetFromExistingBlobs(sourceContainer);

            Console.WriteLine("Done");
        }

        static private IAsset CreateAssetFromExistingBlobs(CloudBlobContainer sourceBlobContainer)
        {
            CloudBlobClient destBlobStorage = _destinationStorageAccount.CreateCloudBlobClient();

            // Create a new asset. 
            IAsset asset = _context.Assets.Create("NewAsset_" + Guid.NewGuid(), AssetCreationOptions.None);

            IAccessPolicy writePolicy = _context.AccessPolicies.Create("writePolicy",
                TimeSpan.FromHours(24), AccessPermissions.Write);

            ILocator destinationLocator =
                _context.Locators.CreateLocator(LocatorType.Sas, asset, writePolicy);

            // Get the asset container URI and Blob copy from mediaContainer to assetContainer. 
            CloudBlobContainer destAssetContainer =
                destBlobStorage.GetContainerReference((new Uri(destinationLocator.Path)).Segments[1]);

            if (destAssetContainer.CreateIfNotExists())
            {
                destAssetContainer.SetPermissions(new BlobContainerPermissions
                {
                    PublicAccess = BlobContainerPublicAccessType.Blob
                });
            }

            var blobList = sourceBlobContainer.ListBlobs();

            foreach (CloudBlockBlob sourceBlob in blobList)
            {
                var assetFile = asset.AssetFiles.Create((sourceBlob as ICloudBlob).Name);

                ICloudBlob destinationBlob = destAssetContainer.GetBlockBlobReference(assetFile.Name);

                CopyBlob(sourceBlob, destAssetContainer);
                
                sourceBlob.FetchAttributes();
                assetFile.ContentFileSize = (sourceBlob as ICloudBlob).Properties.Length;
                assetFile.Update();
                Console.WriteLine("File {0} is of {1} size", assetFile.Name, assetFile.ContentFileSize);
            }

            asset.Update();

            destinationLocator.Delete();
            writePolicy.Delete();

            // Set the primary asset file.
            // If, for example, we copied a set of Smooth Streaming files, 
            // set the .ism file to be the primary file. 
            // If we, for example, copied an .mp4, then the mp4 would be the primary file. 
            var ismAssetFile = asset.AssetFiles.ToList().
                Where(f => f.Name.EndsWith(".ism", StringComparison.OrdinalIgnoreCase)).ToArray().FirstOrDefault();

            // The following code assigns the first .ism file as the primary file in the asset.
            // An asset should have one .ism file.  
            if (ismAssetFile != null)
            {
                ismAssetFile.IsPrimary = true;
                ismAssetFile.Update();
            }

            return asset;
        }

        /// <summary>
        /// Copies the specified blob into the specified container.
        /// </summary>
        /// <param name="sourceBlob">The source container.</param>
        /// <param name="destinationContainer">The destination container.</param>
        static private void CopyBlob(ICloudBlob sourceBlob, CloudBlobContainer destinationContainer)
        {
            var signature = sourceBlob.GetSharedAccessSignature(new SharedAccessBlobPolicy
            {
                Permissions = SharedAccessBlobPermissions.Read,
                SharedAccessExpiryTime = DateTime.UtcNow.AddHours(24)
            });

            var destinationBlob = destinationContainer.GetBlockBlobReference(sourceBlob.Name);

            if (destinationBlob.Exists())
            {
                Console.WriteLine(string.Format("Destination blob '{0}' already exists. Skipping.", destinationBlob.Uri));
            }
            else
            {

                // Display the size of the source blob.
                Console.WriteLine(sourceBlob.Properties.Length);

                Console.WriteLine(string.Format("Copy blob '{0}' to '{1}'", sourceBlob.Uri, destinationBlob.Uri));
                destinationBlob.StartCopy(new Uri(sourceBlob.Uri.AbsoluteUri + signature));

                while (true)
                {
                    // The StartCopyFromBlob is an async operation, 
                    // so we want to check if the copy operation is completed before proceeding. 
                    // To do that, we call FetchAttributes on the blob and check the CopyStatus. 
                    destinationBlob.FetchAttributes();
                    if (destinationBlob.CopyState.Status != CopyStatus.Pending)
                    {
                        break;
                    }
                    //It's still not completed. So wait for some time.
                    System.Threading.Thread.Sleep(1000);
                }

                // Display the size of the destination blob.
                Console.WriteLine(destinationBlob.Properties.Length);

            }
        }
    }
}
```

## <a name="next-steps"></a>後續步驟

您現在可以將上傳的資產編碼。 如需詳細資訊，請參閱 [為資產編碼](media-services-portal-encode.md)。

您也可以使用 Azure Functions，以根據在所設定容器到達的檔案來觸發編碼作業。 如需詳細資訊，請參閱[此範例](https://azure.microsoft.com/resources/samples/media-services-dotnet-functions-integration/ )。

## <a name="media-services-learning-paths"></a>媒體服務學習路徑
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

