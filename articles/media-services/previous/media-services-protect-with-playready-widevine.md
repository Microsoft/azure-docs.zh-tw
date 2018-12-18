---
title: 使用 PlayReady 和/或 Widevine 動態一般加密 | Microsoft Docs
description: 您可以使用 Azure 媒體服務來傳遞受到 Microsoft PlayReady DRM 保護的 MPEG-DASH、Smooth Streaming 和 HTTP 即時串流 (HLS) 串流。 您也可以用它來傳遞利用 Widevine DRM 加密的 DASH。 本主題展示如何利用 PlayReady 和 Widevine DRM 動態加密。
services: media-services
documentationcenter: ''
author: juliako
manager: femila
editor: ''
ms.assetid: 548d1a12-e2cb-45fe-9307-4ec0320567a2
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 09/18/2018
ms.author: juliako
ms.openlocfilehash: ddcd93a82c3eea6266275f79f7a34df5a6557ebf
ms.sourcegitcommit: ce526d13cd826b6f3e2d80558ea2e289d034d48f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46364749"
---
# <a name="use-playready-andor-widevine-dynamic-common-encryption"></a>使用 PlayReady 和/或 Widevine 動態一般加密

> [!div class="op_single_selector"]
> * [.NET](media-services-protect-with-playready-widevine.md)
> * [Java](https://github.com/southworkscom/azure-sdk-for-media-services-java-samples)
> * [PHP](https://github.com/Azure/azure-sdk-for-php/tree/master/examples/MediaServices)
>

> [!NOTE]
> 若要取得最新的 Java SDK 版本並開始使用 Java 進行開發，請參閱[開始使用適用於 Azure 媒體服務的 Java 用戶端 SDK](https://docs.microsoft.com/azure/media-services/media-services-java-how-to-use)。 <br/>
> 若要下載適用於媒體服務的最新 PHP SDK，請在 [Packagist 存放庫](https://packagist.org/packages/microsoft/windowsazure#v0.5.7)中尋找 0.5.7 版的 Microsoft/WindowsAzure 套件。 

## <a name="overview"></a>概觀

 您可以使用媒體服務來傳遞受到 [PlayReady 數位版權管理 (DRM)](https://www.microsoft.com/playready/overview/) 保護的 MPEG-DASH、Smooth Streaming 和 HTTP 即時串流 (HLS) 串流。 您也可傳遞包含 Widevine DRM 授權的加密 DASH 資料流。 PlayReady 和 Widevine 是依照一般加密 (ISO/IEC 23001-7 CENC) 規格加密。 您可以使用[媒體服務 .NET SDK](https://www.nuget.org/packages/windowsazure.mediaservices/) (從 3.5.1 版開始) 或 REST API 設定 AssetDeliveryConfiguration，以使用 Widevine。

媒體服務提供傳遞 PlayReady 和 Widevine DRM 授權的服務。 媒體服務也提供 API，供您用來設定需要 PlayReady DRM 或 Widevine DRM 執行階段在使用者播放受保護內容時，強制執行的權限和限制。 當使用者要求受 DRM 保護的內容時，播放器應用程式會向媒體服務授權服務要求授權。 如果播放器應用程式已獲得授權，媒體服務授權服務就會發出授權給播放器。 PlayReady 或 Widevine 授權包含解密金鑰，可被用戶端播放器用來解密和串流處理內容。

您也可以使用下列媒體服務合作夥伴，協助您傳遞 Widevine 授權： 

* [Axinom](http://www.axinom.com/press/ibc-axinom-drm-6/) 
* [EZDRM](http://ezdrm.com/) 
* [castLabs](http://castlabs.com/company/partners/azure/) 

如需詳細資訊，請參閱整合 [Axinom](media-services-axinom-integration.md) 和 [castLabs](media-services-castlabs-integration.md)。

媒體服務支援多種方式來授權提出金鑰要求的使用者。 內容金鑰授權原則可能有一個或多個授權限制，可能是 Open 或權杖限制。 權杖限制原則必須伴隨 Security Token Service (STS) 所發出的權杖。 媒體服務支援使用[簡單 Web 權杖](https://msdn.microsoft.com/library/gg185950.aspx#BKMK_2) (SWT) 和 [JSON Web 權杖](https://msdn.microsoft.com/library/gg185950.aspx#BKMK_3) (JWT) 格式的權杖。 

如需詳細資訊，請參閱[設定內容金鑰的授權原則](media-services-protect-with-aes128.md#configure_key_auth_policy)。

若要利用動態加密，您需有一個資產，其中包含一組多位元速率 MP4 檔案或多位元速率 Smooth Streaming 來源檔案。 您也需要設定資產的傳遞原則 (本主題稍後會加以描述)。 然後，根據串流 URL 中指定的格式，隨選資料流處理伺服器確保會以您選取的通訊協定傳遞串流。 如此一來，您就能只以單一儲存格式儲存及支付檔案。 媒體服務會根據用戶端的每個要求，建置及傳遞適當的 HTTP 回應。

本文可協助開發人員處理傳遞多重 DRM (如 PlayReady 和 Widevine) 保護之媒體的應用程式。 本文將示範如何利用授權原則設定 PlayReady 授權傳遞服務，這樣只有授權的用戶端才會收到 PlayReady 或 Widevine 授權。 其中也示範如何搭配 PlayReady 或透過 DASH 的 Widevine DRM 來使用動態加密。

>[!NOTE]
>建立媒體服務帳戶時，預設串流端點會新增至 [已停止] 狀態的帳戶。 若要開始串流內容並利用動態封裝和動態加密功能，您需要串流內容的串流端點就必須處於 [執行中] 狀態。 

## <a name="download-the-sample"></a>下載範例
您可以從 [GitHub 上的 Azure 範例](https://github.com/Azure-Samples/media-services-dotnet-dynamic-encryption-with-drm)下載本文所述的範例。

## <a name="configure-dynamic-common-encryption-and-drm-license-delivery-services"></a>設定動態一般加密和 DRM 授權傳遞服務

當您利用 PlayReady 並使用媒體服務授權傳遞服務以及使用動態加密來保護資產時，請執行下列一般步驟：

1. 建立資產，並將檔案上傳到資產。

2. 將包含檔案的資產編碼為自適性位元速率 MP4 集。

3. 建立內容金鑰並將它與編碼的資產產生關聯。 在媒體服務中，內容金鑰包含資產的加密金鑰。

4. 設定內容金鑰的授權原則。 您必須設定內容金鑰授權原則。 用戶端必須先符合原則，系統才會將內容金鑰傳遞給用戶端。

    建立內容金鑰授權原則時，必須指定傳遞方法 (PlayReady 或 Widevine) 和限制 (Open 或 Token)。 您還必須指定金鑰傳遞類型的特定資訊，以定義如何將金鑰傳遞至用戶端 ([PlayReady](media-services-playready-license-template-overview.md) 或 [Widevine](media-services-widevine-license-template-overview.md) 授權範本)。

5. 設定資產的傳遞原則。 傳遞原則設定包含傳遞通訊協定 (例如 MPEG-DASH、HLS、Smooth Streaming 或全部)。 設定還包含動態加密的類型 (例如，一般加密) 和 PlayReady 或 Widevine 授權取得 URL。

    您可以將不同的原則套用至相同資產上的每一個通訊協定。 例如，您可以將 PlayReady 加密套用到 Smooth/DASH，以及將 AES 信封加密套用到 HLS。 傳遞原則中未定義的任何通訊協定 (例如，若您新增單一原則，而它僅指定 HLS 作為通訊協定) 都會遭到封鎖無法串流。 例外情形是當您完全未定義資產傳遞原則時。 接著，允許所有通訊協定，不受阻礙。

6. 建立隨選定位器以取得串流 URL。

您可以在本文結尾處找到完整的 .NET 範例。

下圖示範上述的工作流程。 這裡的權杖用於驗證。

![利用 PlayReady 保護](media/media-services-content-protection-overview/media-services-content-protection-with-drm.png)

本文的其餘部分會提供詳細的說明、程式碼範例，以及示範如何達成上述工作之主題的連結。

## <a name="current-limitations"></a>目前的限制
如果您新增或更新資產傳遞原則，就必須刪除相關聯的定位器，並建立新的定位器。

目前，當您透過媒體服務使用 Widevine 加密時，不支援多個內容金鑰。 

## <a name="create-an-asset-and-upload-files-into-the-asset"></a>建立資產並將檔案上傳到資產
如需管理、編碼及串流處理您的視訊，您必須先將內容上傳到媒體服務。 在上傳後，您的內容就會安全地儲存在雲端，以進一步進行處理和串流處理。

如需詳細資訊，請參閱[將檔案上傳到媒體服務帳戶](media-services-dotnet-upload-files.md)。

## <a name="encode-the-asset-that-contains-the-file-to-the-adaptive-bitrate-mp4-set"></a>將包含檔案的資產編碼為自適性位元速率 MP4 集
使用動態加密時，您要建立一個資產，其中包含一組多位元速率 MP4 檔案，或多位元速率 Smooth Streaming 來源檔案。 然後，隨選資料流處理伺服器會以資訊清單和片段要求中的指定格式作為基礎，確保您以自己選取的通訊協定接收串流。 然後，您就能只以單一儲存格式儲存及支付檔案。 媒體服務會根據用戶單的要求建置及傳遞適當的回應。 如需詳細資訊，請參閱[動態封裝概觀](media-services-dynamic-packaging-overview.md)。

如需編碼方式的指示，請參閱[使用媒體編碼器標準將資產編碼](media-services-dotnet-encode-with-media-encoder-standard.md)。

## <a id="create_contentkey"></a>建立內容金鑰並將它與編碼的資產產生關聯
在媒體服務中，內容金鑰包含您要加密資產時使用的金鑰。

如需詳細資訊，請參閱[建立內容金鑰](media-services-dotnet-create-contentkey.md)。

## <a id="configure_key_auth_policy"></a>設定內容金鑰的授權原則
媒體服務支援多種方式來驗證提出金鑰要求的使用者。 您必須設定內容金鑰授權原則。 用戶端 (播放器) 必須先符合原則，系統才會將金鑰傳遞給用戶端。 內容金鑰授權原則可能有一個或多個授權限制，可能是 Open 或權杖限制。

如需詳細資訊，請參閱[設定內容金鑰授權原則](media-services-dotnet-configure-content-key-auth-policy.md#playready-dynamic-encryption)。

## <a id="configure_asset_delivery_policy"></a>設定資產傳遞原則
設定資產的傳遞原則。 資產傳遞原則組態包括：

* DRM 授權取得 URL。
* 資產傳遞通訊協定 (例如，MPEG DASH、HLS、Smooth Streaming 或全部)。
* 動態加密類型 (在此案例中是一般加密)。

如需詳細資訊，請參閱[設定資產傳遞原則](media-services-dotnet-configure-asset-delivery-policy.md)。

## <a id="create_locator"></a>建立隨選串流定位器以取得串流 URL
您必須為使用者提供 Smooth Streaming、DASH 或 HLS 的串流 URL。

> [!NOTE]
> 如果您新增或更新資產的傳遞原則，就必須刪除現有的定位器，並建立新的定位器。
>
>

如需有關如何發佈資產，並建置串流 URL 的指示，請參閱 [建置串流 URL](media-services-deliver-streaming-content.md)。

## <a name="get-a-test-token"></a>取得測試權杖
根據用於金鑰授權原則的權杖限制取得測試權杖。

```csharp
// Deserializes a string containing an XML representation of a TokenRestrictionTemplate
// back into a TokenRestrictionTemplate class instance.
TokenRestrictionTemplate tokenTemplate =
TokenRestrictionTemplateSerializer.Deserialize(tokenTemplateString);

// Generate a test token based on the data in the given TokenRestrictionTemplate.
//The GenerateTestToken method returns the token without the word "Bearer" in front,
//so you have to add it in front of the token string.
string testToken = TokenRestrictionTemplateSerializer.GenerateTestToken(tokenTemplate);
Console.WriteLine("The authorization token is:\nBearer {0}", testToken);
```

您可以使用 [Azure 媒體服務播放器](http://amsplayer.azurewebsites.net/azuremediaplayer.html)來測試您的串流。

## <a name="create-and-configure-a-visual-studio-project"></a>建立和設定 Visual Studio 專案

1. 設定您的開發環境，並在 app.config 檔案中填入連線資訊，如[使用 .NET 進行 Media Services 開發](media-services-dotnet-how-to-use.md)所述。

2. 將下列項目新增至 app.config 檔案中定義的 **appSettings**：

    ```xml
    <add key="Issuer" value="http://testissuer.com"/>
    <add key="Audience" value="urn:test"/>
    ```

## <a name="example"></a>範例

下列範例會示範 Media Services SDK for .NET 3.5.2 版所推出的功能。 (具體來說，它所包括的能力能夠定義 Widevine 授權範本，並向媒體服務要求 Widevine 授權。)

以本章節中所顯示的程式碼覆寫 Program.cs 檔案中的程式碼。

>[!NOTE]
>對於不同的媒體服務原則 (例如定位器原則或 ContentKeyAuthorizationPolicy) 有 100 萬個原則的限制。 如果您一律使用相同的天數/存取權限，請使用相同的原則識別碼。 例如，為預定要長時間維持就地 (非上傳原則) 的定位器原則。 

如需詳細資訊，請參閱[使用媒體服務 .NET SDK 管理資產和相關的實體](media-services-dotnet-manage-entities.md#limit-access-policies)。

請務必更新變數，以指向您的輸入檔案所在的資料夾。

```csharp
using System;
using System.Collections.Generic;
using System.Configuration;
using System.IO;
using System.Linq;
using System.Threading;
using Microsoft.WindowsAzure.MediaServices.Client;
using Microsoft.WindowsAzure.MediaServices.Client.ContentKeyAuthorization;
using Microsoft.WindowsAzure.MediaServices.Client.DynamicEncryption;
using Microsoft.WindowsAzure.MediaServices.Client.Widevine;
using Newtonsoft.Json;

namespace DynamicEncryptionWithDRM
{
    class Program
    {
        // Read values from the App.config file.
        private static readonly string _AADTenantDomain =
            ConfigurationManager.AppSettings["AMSAADTenantDomain"];
        private static readonly string _RESTAPIEndpoint =
            ConfigurationManager.AppSettings["AMSRESTAPIEndpoint"];
        private static readonly string _AMSClientId =
            ConfigurationManager.AppSettings["AMSClientId"];
        private static readonly string _AMSClientSecret =
            ConfigurationManager.AppSettings["AMSClientSecret"];

        private static readonly Uri _sampleIssuer =
            new Uri(ConfigurationManager.AppSettings["Issuer"]);
        private static readonly Uri _sampleAudience =
            new Uri(ConfigurationManager.AppSettings["Audience"]);

        // Field for service context.
        private static CloudMediaContext _context = null;

        private static readonly string _mediaFiles =
            Path.GetFullPath(@"../..\Media");

        private static readonly string _singleMP4File =
            Path.Combine(_mediaFiles, @"BigBuckBunny.mp4");

        static void Main(string[] args)
        {
            AzureAdTokenCredentials tokenCredentials =
                new AzureAdTokenCredentials(_AADTenantDomain,
                    new AzureAdClientSymmetricKey(_AMSClientId, _AMSClientSecret),
                    AzureEnvironments.AzureCloudEnvironment);

            var tokenProvider = new AzureAdTokenProvider(tokenCredentials);

            _context = new CloudMediaContext(new Uri(_RESTAPIEndpoint), tokenProvider);

            bool tokenRestriction = false;
            string tokenTemplateString = null;

            IAsset asset = UploadFileAndCreateAsset(_singleMP4File);
            Console.WriteLine("Uploaded asset: {0}", asset.Id);

            IAsset encodedAsset = EncodeToAdaptiveBitrateMP4Set(asset);
            Console.WriteLine("Encoded asset: {0}", encodedAsset.Id);

            IContentKey key = CreateCommonTypeContentKey(encodedAsset);
            Console.WriteLine("Created key {0} for the asset {1} ", key.Id, encodedAsset.Id);
            Console.WriteLine("PlayReady License Key delivery URL: {0}", key.GetKeyDeliveryUrl(ContentKeyDeliveryType.PlayReadyLicense));
            Console.WriteLine();

            if (tokenRestriction)
                tokenTemplateString = AddTokenRestrictedAuthorizationPolicy(key);
            else
                AddOpenAuthorizationPolicy(key);

            Console.WriteLine("Added authorization policy: {0}", key.AuthorizationPolicyId);
            Console.WriteLine();

            CreateAssetDeliveryPolicy(encodedAsset, key);
            Console.WriteLine("Created asset delivery policy. \n");
            Console.WriteLine();

            if (tokenRestriction && !String.IsNullOrEmpty(tokenTemplateString))
            {
                // Deserializes a string containing an XML representation of a TokenRestrictionTemplate
                // back into a TokenRestrictionTemplate class instance.
                TokenRestrictionTemplate tokenTemplate =
                    TokenRestrictionTemplateSerializer.Deserialize(tokenTemplateString);

                // Generate a test token based on the data in the given TokenRestrictionTemplate.
                // Note that you need to pass the key ID GUID because 
                // TokenClaim.ContentKeyIdentifierClaim was specified during the creation of TokenRestrictionTemplate.
                Guid rawkey = EncryptionUtils.GetKeyIdAsGuid(key.Id);
                string testToken = TokenRestrictionTemplateSerializer.GenerateTestToken(tokenTemplate, null, rawkey,
                                            DateTime.UtcNow.AddDays(365));
                Console.WriteLine("The authorization token is:\nBearer {0}", testToken);
                Console.WriteLine();
            }

            // You can use the http://amsplayer.azurewebsites.net/azuremediaplayer.html player to test streams.
            // Note that DASH works on Internet Explorer 11 (via PlayReady), Edge (via PlayReady), and Chrome (via Widevine).

            string url = GetStreamingOriginLocator(encodedAsset);
            Console.WriteLine("Encrypted DASH URL: {0}/manifest(format=mpd-time-csf)", url);

            Console.ReadLine();
        }

        static public IAsset UploadFileAndCreateAsset(string singleFilePath)
        {
            if (!File.Exists(singleFilePath))
            {
                Console.WriteLine("File does not exist.");
                return null;
            }

            var assetName = Path.GetFileNameWithoutExtension(singleFilePath);
            IAsset inputAsset = _context.Assets.Create(assetName, AssetCreationOptions.None);

            var assetFile = inputAsset.AssetFiles.Create(Path.GetFileName(singleFilePath));

            Console.WriteLine("Created assetFile {0}", assetFile.Name);

            Console.WriteLine("Upload {0}", assetFile.Name);

            assetFile.Upload(singleFilePath);
            Console.WriteLine("Done uploading {0}", assetFile.Name);

            return inputAsset;
        }

        static public IAsset EncodeToAdaptiveBitrateMP4Set(IAsset inputAsset)
        {
            var encodingPreset = "Adaptive Streaming";

            IJob job = _context.Jobs.Create(String.Format("Encoding into Mp4 {0} to {1}",
                        inputAsset.Name,
                        encodingPreset));

            var mediaProcessors =
            _context.MediaProcessors.Where(p => p.Name.Contains("Media Encoder Standard")).ToList();

            var latestMediaProcessor =
            mediaProcessors.OrderBy(mp => new Version(mp.Version)).LastOrDefault();

            ITask encodeTask = job.Tasks.AddNew("Encoding", latestMediaProcessor, encodingPreset, TaskOptions.None);
            encodeTask.InputAssets.Add(inputAsset);
            encodeTask.OutputAssets.AddNew(String.Format("{0} as {1}", inputAsset.Name, encodingPreset), AssetCreationOptions.StorageEncrypted);

            job.StateChanged += new EventHandler<JobStateChangedEventArgs>(JobStateChanged);
            job.Submit();
            job.GetExecutionProgressTask(CancellationToken.None).Wait();

            return job.OutputMediaAssets[0];
        }


        static public IContentKey CreateCommonTypeContentKey(IAsset asset)
        {

            Guid keyId = Guid.NewGuid();
            byte[] contentKey = GetRandomBuffer(16);

            IContentKey key = _context.ContentKeys.Create(
                        keyId,
                        contentKey,
                        "ContentKey",
                        ContentKeyType.CommonEncryption);

            // Associate the key with the asset.
            asset.ContentKeys.Add(key);

            return key;
        }

        static public void AddOpenAuthorizationPolicy(IContentKey contentKey)
        {

            // Create ContentKeyAuthorizationPolicy with open restrictions
            // and create an authorization policy.         

            List<ContentKeyAuthorizationPolicyRestriction> restrictions = new List<ContentKeyAuthorizationPolicyRestriction>
                {
                new ContentKeyAuthorizationPolicyRestriction
                {
                    Name = "Open",
                    KeyRestrictionType = (int)ContentKeyRestrictionType.Open,
                    Requirements = null
                }
                };

            // Configure PlayReady and Widevine license templates.
            string PlayReadyLicenseTemplate = ConfigurePlayReadyLicenseTemplate();

            string WidevineLicenseTemplate = ConfigureWidevineLicenseTemplate();

            IContentKeyAuthorizationPolicyOption PlayReadyPolicy =
            _context.ContentKeyAuthorizationPolicyOptions.Create("",
                ContentKeyDeliveryType.PlayReadyLicense,
                restrictions, PlayReadyLicenseTemplate);

            IContentKeyAuthorizationPolicyOption WidevinePolicy =
            _context.ContentKeyAuthorizationPolicyOptions.Create("",
                ContentKeyDeliveryType.Widevine,
                restrictions, WidevineLicenseTemplate);

            IContentKeyAuthorizationPolicy contentKeyAuthorizationPolicy = _context.
                ContentKeyAuthorizationPolicies.
                CreateAsync("Deliver Common Content Key with no restrictions").
                Result;


            contentKeyAuthorizationPolicy.Options.Add(PlayReadyPolicy);
            contentKeyAuthorizationPolicy.Options.Add(WidevinePolicy);
            // Associate the content key authorization policy with the content key.
            contentKey.AuthorizationPolicyId = contentKeyAuthorizationPolicy.Id;
            contentKey = contentKey.UpdateAsync().Result;
        }

        public static string AddTokenRestrictedAuthorizationPolicy(IContentKey contentKey)
        {
            string tokenTemplateString = GenerateTokenRequirements();

            List<ContentKeyAuthorizationPolicyRestriction> restrictions = new List<ContentKeyAuthorizationPolicyRestriction>
                {
                new ContentKeyAuthorizationPolicyRestriction
                {
                    Name = "Token Authorization Policy",
                    KeyRestrictionType = (int)ContentKeyRestrictionType.TokenRestricted,
                    Requirements = tokenTemplateString,
                }
                };

            // Configure PlayReady and Widevine license templates.
            string PlayReadyLicenseTemplate = ConfigurePlayReadyLicenseTemplate();

            string WidevineLicenseTemplate = ConfigureWidevineLicenseTemplate();

            IContentKeyAuthorizationPolicyOption PlayReadyPolicy =
            _context.ContentKeyAuthorizationPolicyOptions.Create("Token option",
                ContentKeyDeliveryType.PlayReadyLicense,
                restrictions, PlayReadyLicenseTemplate);

            IContentKeyAuthorizationPolicyOption WidevinePolicy =
            _context.ContentKeyAuthorizationPolicyOptions.Create("Token option",
                ContentKeyDeliveryType.Widevine,
                restrictions, WidevineLicenseTemplate);

            IContentKeyAuthorizationPolicy contentKeyAuthorizationPolicy = _context.
                ContentKeyAuthorizationPolicies.
                CreateAsync("Deliver Common Content Key with token restrictions").
                Result;

            contentKeyAuthorizationPolicy.Options.Add(PlayReadyPolicy);
            contentKeyAuthorizationPolicy.Options.Add(WidevinePolicy);

            // Associate the content key authorization policy with the content key.
            contentKey.AuthorizationPolicyId = contentKeyAuthorizationPolicy.Id;
            contentKey = contentKey.UpdateAsync().Result;

            return tokenTemplateString;
        }

        static private string GenerateTokenRequirements()
        {
            TokenRestrictionTemplate template = new TokenRestrictionTemplate(TokenType.SWT);

            template.PrimaryVerificationKey = new SymmetricVerificationKey();
            template.AlternateVerificationKeys.Add(new SymmetricVerificationKey());
            template.Audience = _sampleAudience.ToString();
            template.Issuer = _sampleIssuer.ToString();
            template.RequiredClaims.Add(TokenClaim.ContentKeyIdentifierClaim);

            return TokenRestrictionTemplateSerializer.Serialize(template);
        }

        static private string ConfigurePlayReadyLicenseTemplate()
        {
            // The following code configures the PlayReady license template by using .NET classes
            // and returns the XML string.

            //The PlayReadyLicenseResponseTemplate class represents the template for the response sent back to the end user.
            //It contains a field for a custom data string between the license server and the application
            //(which might be useful for custom app logic) as well as a list of one or more license templates.
            PlayReadyLicenseResponseTemplate responseTemplate = new PlayReadyLicenseResponseTemplate();

            // The PlayReadyLicenseTemplate class represents a license template you can use to create PlayReady licenses
            // to be returned to end users.
            //It contains the data on the content key in the license and any rights or restrictions to be
            //enforced by the PlayReady DRM runtime when you use the content key.
            PlayReadyLicenseTemplate licenseTemplate = new PlayReadyLicenseTemplate();
            //Configure whether the license is persistent (saved in persistent storage on the client)
            //or nonpersistent (held in memory only while the player uses the license).  
            licenseTemplate.LicenseType = PlayReadyLicenseType.Nonpersistent;

            // AllowTestDevices controls whether test devices can use the license or not.  
            // If true, the MinimumSecurityLevel property of the license
            // is set to 150. If false (the default), the MinimumSecurityLevel property of the license is set to 2,000.
            licenseTemplate.AllowTestDevices = true;

            // You also can configure the PlayRight in the PlayReady license by using the PlayReadyPlayRight class.
            // It grants the user the ability to play back the content subject to the zero or more restrictions
            // configured in the license and on the PlayRight itself (for playback-specific policy).
            // Much of the policy on the PlayRight has to do with output restrictions,
            // which control the types of outputs that the content can be played over and
            // any restrictions that must be put in place when you use a given output.
            // For example, if DigitalVideoOnlyContentRestriction is enabled,
            // the DRM runtime allows the video to be displayed only over digital outputs
            //(analog video outputs aren't allowed to pass the content).

            //IMPORTANT: These types of restrictions can be very powerful but also can affect the consumer experience.
            // If output protections are too restrictive, 
            // content might be unplayable on some clients. For more information, see the PlayReady Compliance Rules document.

            // For example:
            //licenseTemplate.PlayRight.AgcAndColorStripeRestriction = new AgcAndColorStripeRestriction(1);

            responseTemplate.LicenseTemplates.Add(licenseTemplate);

            return MediaServicesLicenseTemplateSerializer.Serialize(responseTemplate);
        }

        private static string ConfigureWidevineLicenseTemplate()
        {
            var template = new WidevineMessage
            {
                allowed_track_types = AllowedTrackTypes.SD_HD,
                content_key_specs = new[]
            {
                    new ContentKeySpecs
                    {
                    required_output_protection = new RequiredOutputProtection { hdcp = Hdcp.HDCP_NONE},
                    security_level = 1,
                    track_type = "SD"
                    }
                },
                policy_overrides = new
                {
                    can_play = true,
                    can_persist = true,
                    can_renew = false
                }
            };

            string configuration = JsonConvert.SerializeObject(template);
            return configuration;
        }

        static public void CreateAssetDeliveryPolicy(IAsset asset, IContentKey key)
        {
            // Get the PlayReady license service URL.
            Uri acquisitionUrl = key.GetKeyDeliveryUrl(ContentKeyDeliveryType.PlayReadyLicense);

            // GetKeyDeliveryUrl for Widevine attaches the KID to the URL.
            // For example: https://amsaccount1.keydelivery.mediaservices.windows.net/Widevine/?KID=268a6dcb-18c8-4648-8c95-f46429e4927c.  
            // WidevineBaseLicenseAcquisitionUrl (used in the following) also tells dynamic encryption
            // to append /? KID =< keyId > to the end of the URL when you create the manifest.
            // As a result, the Widevine license acquisition URL has the KID appended twice,
            // so you need to remove the KID in the URL when you call GetKeyDeliveryUrl.

            Uri widevineUrl = key.GetKeyDeliveryUrl(ContentKeyDeliveryType.Widevine);
            UriBuilder uriBuilder = new UriBuilder(widevineUrl);
            uriBuilder.Query = String.Empty;
            widevineUrl = uriBuilder.Uri;

            Dictionary<AssetDeliveryPolicyConfigurationKey, string> assetDeliveryPolicyConfiguration =
            new Dictionary<AssetDeliveryPolicyConfigurationKey, string>
            {
                    {AssetDeliveryPolicyConfigurationKey.PlayReadyLicenseAcquisitionUrl, acquisitionUrl.ToString()},
                    {AssetDeliveryPolicyConfigurationKey.WidevineBaseLicenseAcquisitionUrl, widevineUrl.ToString()}

            };

            // In this case, we specify only the DASH streaming protocol in the delivery policy.
            // All other protocols are blocked from streaming.
            var assetDeliveryPolicy = _context.AssetDeliveryPolicies.Create(
                "AssetDeliveryPolicy",
            AssetDeliveryPolicyType.DynamicCommonEncryption,
            AssetDeliveryProtocol.Dash,
            assetDeliveryPolicyConfiguration);


            // Add AssetDelivery Policy to the asset.
            asset.DeliveryPolicies.Add(assetDeliveryPolicy);

        }

        /// <summary>
        /// Gets the streaming origin locator.
        /// </summary>
        /// <param name="assets"></param>
        /// <returns></returns>
        static public string GetStreamingOriginLocator(IAsset asset)
        {

            // Get a reference to the streaming manifest file from the 
            // collection of files in the asset.

            var assetFile = asset.AssetFiles.Where(f => f.Name.ToLower().
                         EndsWith(".ism")).
                         FirstOrDefault();

            // Create a 30-day read-only access policy.
            IAccessPolicy policy = _context.AccessPolicies.Create("Streaming policy",
            TimeSpan.FromDays(30),
            AccessPermissions.Read);

            // Create a locator to the streaming content on an origin.
            ILocator originLocator = _context.Locators.CreateLocator(LocatorType.OnDemandOrigin, asset,
            policy,
            DateTime.UtcNow.AddMinutes(-5));

            // Create a URL to the manifest file.
            return originLocator.Path + assetFile.Name;
        }

        static private void JobStateChanged(object sender, JobStateChangedEventArgs e)
        {
            Console.WriteLine(string.Format("{0}\n  State: {1}\n  Time: {2}\n\n",
            ((IJob)sender).Name,
            e.CurrentState,
            DateTime.UtcNow.ToString(@"yyyy_M_d__hh_mm_ss")));
        }

        static private byte[] GetRandomBuffer(int length)
        {
            var returnValue = new byte[length];

            using (var rng =
            new System.Security.Cryptography.RNGCryptoServiceProvider())
            {
                rng.GetBytes(returnValue);
            }

            return returnValue;
        }
    }
}
```

## <a name="next-steps"></a>後續步驟

[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="see-also"></a>另請參閱
* [使用具有多重 DRM 及存取控制的 CENC](media-services-cenc-with-multidrm-access-control.md)
* [使用媒體服務設定 Widevine 套件](http://mingfeiy.com/how-to-configure-widevine-packaging-with-azure-media-services)
* [宣布在 Azure 媒體服務中推出 Google Widevine 授權傳遞服務](https://azure.microsoft.com/blog/announcing-general-availability-of-google-widevine-license-services/)
