---
title: 開始在 Azure 媒體服務上使用 Java SDK |Microsoft 文件
description: 本教學課程會逐步引導您使用 Java 實作含 Azure 媒體服務 (AMS) 應用程式的基本點播視訊 (VoD) 內容傳遞服務。
services: media-services
documentationcenter: java
author: juliako
manager: femila
editor: johndeu
ms.assetid: b884bd61-dbdb-42ea-b170-8fb02e7fded7
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: java
ms.topic: get-started-article
ms.date: 09/18/2018
ms.author: juliako
ms.openlocfilehash: f53efde38c4f37a6f92aa71d9ab7eb4976c048f4
ms.sourcegitcommit: ce526d13cd826b6f3e2d80558ea2e289d034d48f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46365412"
---
# <a name="get-started-with-the-java-client-sdk-for-azure-media-services"></a>開始在 Azure 媒體服務上使用 Java 用戶端 SDK
[!INCLUDE [media-services-selector-get-started](../../../includes/media-services-selector-get-started.md)]

本教學課程會逐步引導您使用 Java 用戶端 SDK，來實作搭配 Azure 媒體服務的基本視訊內容傳遞服務。

## <a name="prerequisites"></a>必要條件

需要有下列項目，才能完成本教學課程：

* 一個 Azure 帳戶。 如需詳細資訊，請參閱 [Azure 免費試用](https://azure.microsoft.com/pricing/free-trial/)。
* 媒體服務帳戶。 若要建立媒體服務帳戶，請參閱[如何建立媒體服務帳戶](media-services-portal-create-account.md)。
* 目前的 [Azure 媒體服務 Java SDK](https://mvnrepository.com/artifact/com.microsoft.azure/azure-media/latest)

## <a name="how-to-import-the-azure-media-services-java-client-sdk-package"></a>作法：匯入 Azure 媒體服務 Java 用戶端 SDK 套件

若要開始使用適用 Java 的媒體服務 SDK，請從 [Azure 媒體服務 Java SDK](https://mvnrepository.com/artifact/com.microsoft.azure/azure-media/latest)，將參考新增至 `azure-media` 套件的目前版本 (0.9.8)

例如，如果您的建置工具是 `gradle`，請將下列相依項目新增至您的 `build.gradle` 檔案：

    compile group: 'com.microsoft.azure', name: 'azure-media', version: '0.9.8'

>[!IMPORTANT]
>從 `0.9.8` 版的 `azure-media` 套件開始，SDK 新增了 Azure Active Directory (AAD) 驗證的支援，並移除了 Azure 存取控制服務 (ACS) 驗證的支援。 建議您儘速移轉至 Azure AD 驗證模型。 如需有關移轉的詳細資訊，請閱讀[使用 Azure AD 驗證存取 Azure 媒體服務 API](media-services-use-aad-auth-to-access-ams-api.md)一文。

>[!NOTE]
>您可以在我們的 [GitHub 存放庫](https://github.com/Azure/azure-sdk-for-java/tree/0.9/services/azure-media)中找到 Azure 媒體服務 Java SDK 的原始程式碼。 請確定切換至 0.9 分支，而不是主要分支。 

## <a name="how-to-use-azure-media-services-with-java"></a>作法：搭配使用 Azure 媒體服務和 Java

>[!NOTE]
>建立媒體服務帳戶時，**預設**串流端點會新增至 [已停止] 狀態的帳戶。 若要開始串流內容並利用動態封裝和動態加密功能，您想要串流內容的串流端點必須處於 [執行中] 狀態。

下列程式碼將示範如何建立資產、上傳媒體檔案到資產、使用工作 (Task) 執行作業 (Job) 來轉換此資產，以及建立定位器以便串流您的影片。

使用此程式碼前，請先設定媒體服務帳戶。 如需設定帳戶的相關資訊，請參閱[如何建立媒體服務帳戶](media-services-portal-create-account.md)。

透過使用 Azure AD 服務主體驗證，程式碼會連線到 Azure 媒體服務 API。 建立 Azure AD 應用程式，並在程式碼中指定下列變數的值：
* `tenant`：Azure AD 應用程式所在的 Azure AD 租用戶網域
* `clientId`：Azure AD 應用程式的用戶端識別碼
* `clientKey`：Azure AD 應用程式的用戶端金鑰
* `restApiEndpoint`：Azure 媒體服務帳戶的 REST API 端點

您可以建立 Azure AD 應用程式，並從 Azure 入口網站取得上述設定值。 如需詳細資訊，請參閱[利用 Azure 入口網站開始使用 Azure AD 驗證](https://docs.microsoft.com/azure/media-services/media-services-portal-get-started-with-aad)的**服務主體驗證**區段。

此程式碼還需要用到儲存在本機的視訊檔案。 您必須編輯程式碼，以上傳您自己的本機檔案。

```java
    import java.io.*;
    import java.net.URI;
    import java.security.NoSuchAlgorithmException;
    import java.util.EnumSet;
    import java.util.concurrent.ExecutorService;
    import java.util.concurrent.Executors;

    import com.microsoft.windowsazure.Configuration;
    import com.microsoft.windowsazure.exception.ServiceException;
    import com.microsoft.windowsazure.services.media.MediaConfiguration;
    import com.microsoft.windowsazure.services.media.MediaContract;
    import com.microsoft.windowsazure.services.media.MediaService;
    import com.microsoft.windowsazure.services.media.WritableBlobContainerContract;
    import com.microsoft.windowsazure.services.media.authentication.AzureAdClientSymmetricKey;
    import com.microsoft.windowsazure.services.media.authentication.AzureAdTokenCredentials;
    import com.microsoft.windowsazure.services.media.authentication.AzureAdTokenProvider;
    import com.microsoft.windowsazure.services.media.authentication.AzureEnvironments;
    import com.microsoft.windowsazure.services.media.models.AccessPolicy;
    import com.microsoft.windowsazure.services.media.models.AccessPolicyInfo;
    import com.microsoft.windowsazure.services.media.models.AccessPolicyPermission;
    import com.microsoft.windowsazure.services.media.models.Asset;
    import com.microsoft.windowsazure.services.media.models.AssetFile;
    import com.microsoft.windowsazure.services.media.models.AssetFileInfo;
    import com.microsoft.windowsazure.services.media.models.AssetInfo;
    import com.microsoft.windowsazure.services.media.models.Job;
    import com.microsoft.windowsazure.services.media.models.JobInfo;
    import com.microsoft.windowsazure.services.media.models.JobState;
    import com.microsoft.windowsazure.services.media.models.ListResult;
    import com.microsoft.windowsazure.services.media.models.Locator;
    import com.microsoft.windowsazure.services.media.models.LocatorInfo;
    import com.microsoft.windowsazure.services.media.models.LocatorType;
    import com.microsoft.windowsazure.services.media.models.MediaProcessor;
    import com.microsoft.windowsazure.services.media.models.MediaProcessorInfo;
    import com.microsoft.windowsazure.services.media.models.Task;

    public class Program
    {
        // Media Services account credentials configuration
        private static String tenant = "tenant.domain.com";
        private static String clientId = "<client id>";
        private static String clientKey = "<client key>";
        private static String restApiEndpoint = "https://account_name.restv2.region_name.media.azure.net/api/";

        // Media Services API
        private static MediaContract mediaService;

        // Encoder configuration
        // This is using the default Adaptive Streaming encoding preset. 
        // You can choose to use a custom preset, or any other sample defined preset. 
        // In addition you can use other processors, like Speech Analyzer, or Redactor if desired.
        private static String preferedEncoder = "Media Encoder Standard";
        private static String encodingPreset = "Adaptive Streaming";

        public static void main(String[] args)
        {
            ExecutorService executorService = Executors.newFixedThreadPool(1);

            try {
                // Setup Azure AD Service Principal Symmetric Key Credentials
                AzureAdTokenCredentials credentials = new AzureAdTokenCredentials(
                        tenant,
                        new AzureAdClientSymmetricKey(clientId, clientKey),
                        AzureEnvironments.AZURE_CLOUD_ENVIRONMENT);

                AzureAdTokenProvider provider = new AzureAdTokenProvider(credentials, executorService);

                // Create a new configuration with the credentials
                Configuration configuration = MediaConfiguration.configureWithAzureAdTokenProvider(
                        new URI(restApiEndpoint),
                        provider);

                // Create the media service provisioned with the new configuration
                mediaService = MediaService.create(configuration);

                // Upload a local file to an Asset
                AssetInfo uploadAsset = uploadFileAndCreateAsset("Video Name", "C:/path/to/video.mp4");
                System.out.println("Uploaded Asset Id: " + uploadAsset.getId());

                // Transform the Asset
                AssetInfo encodedAsset = encode(uploadAsset);
                System.out.println("Encoded Asset Id: " + encodedAsset.getId());

                // Create the Streaming Origin Locator
                String url = getStreamingOriginLocator(encodedAsset);

                System.out.println("Origin Locator URL: " + url);
                System.out.println("Sample completed!");

            } catch (ServiceException se) {
                System.out.println("ServiceException encountered.");
                System.out.println(se.toString());
            } catch (Exception e) {
                System.out.println("Exception encountered.");
                System.out.println(e.toString());
            } finally {
                executorService.shutdown();
            }
        }

        private static AssetInfo uploadFileAndCreateAsset(String assetName, String fileName)
            throws ServiceException, FileNotFoundException, NoSuchAlgorithmException {

            WritableBlobContainerContract uploader;
            AssetInfo resultAsset;
            AccessPolicyInfo uploadAccessPolicy;
            LocatorInfo uploadLocator = null;

            // Create an Asset
            resultAsset = mediaService.create(Asset.create().setName(assetName).setAlternateId("altId"));
            System.out.println("Created Asset " + fileName);

            // Create an AccessPolicy that provides Write access for 15 minutes
            uploadAccessPolicy = mediaService
                .create(AccessPolicy.create("uploadAccessPolicy", 15.0, EnumSet.of(AccessPolicyPermission.WRITE)));

            // Create a Locator using the AccessPolicy and Asset
            uploadLocator = mediaService
                .create(Locator.create(uploadAccessPolicy.getId(), resultAsset.getId(), LocatorType.SAS));

            // Create the Blob Writer using the Locator
            uploader = mediaService.createBlobWriter(uploadLocator);

            File file = new File(fileName);

            // The local file that will be uploaded to your Media Services account
            InputStream input = new FileInputStream(file);

            System.out.println("Uploading " + fileName);

            // Upload the local file to the media asset
            uploader.createBlockBlob(file.getName(), input);

            // Inform Media Services about the uploaded files
            mediaService.action(AssetFile.createFileInfos(resultAsset.getId()));
            System.out.println("Uploaded Asset File " + fileName);

            mediaService.delete(Locator.delete(uploadLocator.getId()));
            mediaService.delete(AccessPolicy.delete(uploadAccessPolicy.getId()));

            return resultAsset;
        }

        // Create a Job that contains a Task to transform the Asset
        private static AssetInfo encode(AssetInfo assetToEncode)
            throws ServiceException, InterruptedException {

            // Retrieve the list of Media Processors that match the name
            ListResult<MediaProcessorInfo> mediaProcessors = mediaService
                            .list(MediaProcessor.list().set("$filter", String.format("Name eq '%s'", preferedEncoder)));

            // Use the latest version of the Media Processor
            MediaProcessorInfo mediaProcessor = null;
            for (MediaProcessorInfo info : mediaProcessors) {
                if (null == mediaProcessor || info.getVersion().compareTo(mediaProcessor.getVersion()) > 0) {
                    mediaProcessor = info;
                }
            }

            System.out.println("Using Media Processor: " + mediaProcessor.getName() + " " + mediaProcessor.getVersion());

            // Create a task with the specified Media Processor
            String outputAssetName = String.format("%s as %s", assetToEncode.getName(), encodingPreset);
            String taskXml = "<taskBody><inputAsset>JobInputAsset(0)</inputAsset>"
                    + "<outputAsset assetCreationOptions=\"0\"" // AssetCreationOptions.None
                    + " assetName=\"" + outputAssetName + "\">JobOutputAsset(0)</outputAsset></taskBody>";

            Task.CreateBatchOperation task = Task.create(mediaProcessor.getId(), taskXml)
                    .setConfiguration(encodingPreset).setName("Encoding");

            // Create the Job; this automatically schedules and runs it.
            Job.Creator jobCreator = Job.create()
                    .setName(String.format("Encoding %s to %s", assetToEncode.getName(), encodingPreset))
                    .addInputMediaAsset(assetToEncode.getId()).setPriority(2).addTaskCreator(task);
            JobInfo job = mediaService.create(jobCreator);

            String jobId = job.getId();
            System.out.println("Created Job with Id: " + jobId);

            // Check to see if the Job has completed
            checkJobStatus(jobId);
            // Done with the Job

            // Retrieve the output Asset
            ListResult<AssetInfo> outputAssets = mediaService.list(Asset.list(job.getOutputAssetsLink()));
            return outputAssets.get(0);
        }


        public static String getStreamingOriginLocator(AssetInfo asset) throws ServiceException {
            // Get the .ISM AssetFile
            ListResult<AssetFileInfo> assetFiles = mediaService.list(AssetFile.list(asset.getAssetFilesLink()));
            AssetFileInfo streamingAssetFile = null;
            for (AssetFileInfo file : assetFiles) {
                if (file.getName().toLowerCase().endsWith(".ism")) {
                    streamingAssetFile = file;
                    break;
                }
            }

            AccessPolicyInfo originAccessPolicy;
            LocatorInfo originLocator = null;

            // Create a 30-day read only AccessPolicy
            double durationInMinutes = 60 * 24 * 30;
            originAccessPolicy = mediaService.create(
                    AccessPolicy.create("Streaming policy", durationInMinutes, EnumSet.of(AccessPolicyPermission.READ)));

            // Create a Locator using the AccessPolicy and Asset
            originLocator = mediaService
                    .create(Locator.create(originAccessPolicy.getId(), asset.getId(), LocatorType.OnDemandOrigin));

            // Create a Smooth Streaming base URL
            return originLocator.getPath() + streamingAssetFile.getName() + "/manifest";
        }

        private static void checkJobStatus(String jobId) throws InterruptedException, ServiceException {
            boolean done = false;
            JobState jobState = null;
            while (!done) {
                // Sleep for 5 seconds
                Thread.sleep(5000);

                // Query the updated Job state
                jobState = mediaService.get(Job.get(jobId)).getState();
                System.out.println("Job state: " + jobState);

                if (jobState == JobState.Finished || jobState == JobState.Canceled || jobState == JobState.Error) {
                    done = true;
                }
            }
        }
    }
```

## <a name="media-services-learning-paths"></a>媒體服務學習路徑
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="additional-resources"></a>其他資源
如需在 Azure 中開發 Java 應用程式的詳細資訊，請參閱 [Azure Java 開發人員中心][Azure Java Developer Center]和[適用於 Java 開發人員的 Azure][Azure for Java developers]。


如需媒體服務 Javadoc 文件，請參閱 [Azure Libraries for Java 文件][Azure Libraries for Java 文件]。

<!-- URLs. -->

[Azure Media Services SDK Maven Package]: https://mvnrepository.com/artifact/com.microsoft.azure/azure-media/latest
[Azure Java Developer Center]: http://azure.microsoft.com/develop/java/
[Azure for Java developers]: https://docs.microsoft.com/java/azure/
[Media Services Client Development]: http://msdn.microsoft.com/library/windowsazure/dn223283.aspx

