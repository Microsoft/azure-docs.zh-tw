---
title: 使用 Microsoft PlayReady 或 Apple FairPlay 來保護 HLS 內容 | Microsoft Docs
description: 本主題提供概觀及顯示如何使用 Azure 媒體服務，以 Apple FairPlay 動態加密您的 HTTP 即時串流 (HLS) 內容。 它也會顯示如何使用媒體服務授權傳遞服務，傳遞 FairPlay 授權給用戶端。
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/18/2018
ms.author: juliako
ms.openlocfilehash: 32f3f4fd3f4f299c9b084ab8604b56ea70e639a4
ms.sourcegitcommit: ce526d13cd826b6f3e2d80558ea2e289d034d48f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46368218"
---
# <a name="protect-your-hls-content-with-apple-fairplay-or-microsoft-playready"></a>使用 Apple FairPlay 或 Microsoft PlayReady 保護 HLS 內容
Azure 媒體服務可讓您使用下列格式，動態加密您的 HTTP 即時串流 (HLS) 內容︰  

* **AES-128 信封清除金鑰**

    使用 **AES-128 CBC** 模式加密整個區塊。 iOS 和 OS X 播放程式原本就支援資料流解密。 如需詳細資訊，請參閱[使用 AES-128 動態加密和金鑰傳遞服務](media-services-protect-with-aes128.md)。
* **Apple FairPlay**

    使用 **AES-128 CBC** 模式加密個別的視訊和音訊範例。 **FairPlay 串流** (FPS) 已整合至裝置工作系統，在 iOS 和 Apple 電視上具有原生支援。 OS X 上的 Safari 使用加密媒體擴充功能 (EME) 介面支援來啟用 FPS。
* **Microsoft PlayReady (英文)**

下圖顯示 **HLS + FairPlay 或 PlayReady 動態加密** 工作流程。

![動態加密工作流程的圖表](./media/media-services-content-protection-overview/media-services-content-protection-with-FairPlay.png)

本文示範如何使用媒體服務，以 Apple FairPlay 動態加密 HLS 內容。 它也會顯示如何使用媒體服務授權傳遞服務，傳遞 FairPlay 授權給用戶端。

> [!NOTE]
> 如果您也想要以 PlayReady 加密 HLS 內容，必須建立一般內容金鑰，並將它與您的資產產生關聯。 您也必須設定內容金鑰的授權原則，如[使用 PlayReady 動態一般加密](media-services-protect-with-playready-widevine.md)中所述。
>
>

## <a name="requirements-and-considerations"></a>需求和考量

以下是使用媒體服務傳遞以 FairPlay 加密的 HLS，以及傳遞 FairPlay 授權時所需的項目：

  * 一個 Azure 帳戶。 如需詳細資訊，請參閱 [Azure 免費試用](https://azure.microsoft.com/pricing/free-trial/?WT.mc_id=A261C142F)。
  * 媒體服務帳戶。 若要建立一個，請參閱[使用 Azure 入口網站建立 Azure 媒體服務帳戶](media-services-portal-create-account.md)。
  * 註冊 [Apple Development Program](https://developer.apple.com/)。
  * Apple 要求內容擁有者必須取得 [部署套件](https://developer.apple.com/contact/fps/)。 說明您已使用媒體服務實作金鑰安全性模組 (KSM)，現在想要求最終的 FPS 套件。 最終 FPS 套件包含相關指示，用以產生憑證和取得應用程式密碼金鑰 (ASK)。 您可使用 ASK 來設定 FairPlay。
  * Azure 媒體服務 .NET SDK **3.6.0** 版或更新版本。

必須在媒體服務金鑰傳遞端設定下列各項︰

  * **應用程式憑證 (AC)**︰這是包含私密金鑰的 .pfx 檔案。 您可建立這個檔案並以密碼加密。

       當您設定金鑰傳遞原則時，您必須提供該密碼和 base64 格式的 .pfx 檔案。

      下列步驟說明如何產生 FairPlay 的 .pfx 憑證檔案：

    1. 從 https://slproweb.com/products/Win32OpenSSL.html 安裝 OpenSSL。

        移至 FairPlay 憑證和其他 Apple 提供檔案所在的資料夾。
    2. 從命令列執行下列命令。 這會將 .cer 檔案轉換成 .pem 檔案。

        "C:\OpenSSL-Win32\bin\openssl.exe" x509 -inform der -in FairPlay.cer -out FairPlay-out.pem
    3. 從命令列執行下列命令。 這會將 .pem 檔案轉換為包含私密金鑰的 .pfx 檔案。 OpenSSL 程式會接著要求 .Pfx 檔案的密碼。

        "C:\OpenSSL-Win32\bin\openssl.exe" pkcs12 -export -out FairPlay-out.pfx -inkey privatekey.pem -in FairPlay-out.pem -passin file:privatekey-pem-pass.txt
  * **應用程式憑證密碼** - 用來建立 .pfx 檔案的密碼。
  * **應用程式憑證密碼識別碼**︰您必須上傳密碼，做法類似於其上傳其他媒體服務金鑰的方式。 使用 **ContentKeyType.FairPlayPfxPassword** 列舉值來取得媒體服務識別碼。 這是它們要在金鑰傳遞原則選項內使用所需之物。
  * **iv**︰這是 16 位元組的隨機值。 其必須符合資產傳遞原則中的 iv。 您會產生 iv，並將它放在兩個位置︰資產傳遞原則和金鑰傳遞原則選項。
  * **ASK**：當您使用 Apple 開發人員入口網站產生憑證時，會收到此金鑰。 每個開發小組都會收到一個唯一的 ASK。 儲存一份 ASK，並將它存放在安全的地方。 您之後必須將 ASK 設定為媒體服務的 FairPlayAsk。
  * **ASK 識別碼**︰當您將 ASK 上傳至媒體服務時，會取得這個識別碼。 您必須使用 **ContentKeyType.FairPlayAsk** 列舉值來上傳 ASK。 結果會傳回媒體服務識別碼，設定金鑰傳遞原則選項時應使用此識別碼。

FPS 用戶端必須設定下列各項︰

  * **應用程式憑證 (AC)**：這是 .cer/.der 檔案，其中包含作業系統用來加密某些承載的公開金鑰。 媒體服務必須了解它，因為播放程式需要它。 金鑰傳遞服務會使用對應的私密金鑰來解密。

若要播放 FairPlay 加密的資料流，請先取得真正的 ASK，然後產生真正的憑證。 該處理程序會建立所有 3 個部分︰

  * .der 檔案
  * .pfx 檔案
  * .pfx 的密碼

下列用戶端支援採用 **AES-128 CBC** 加密的 HLS︰OS X 上的 Safari、Apple TV、iOS.

## <a name="configure-fairplay-dynamic-encryption-and-license-delivery-services"></a>設定 FairPlay 動態加密和授權傳遞服務
以下是使用媒體服務授權傳遞服務，以及使用動態加密時，透過 FairPlay 來保護資產的一般步驟。

1. 建立資產，並將檔案上傳到資產。
2. 將包含檔案的資產編碼為自適性位元速率 MP4 集。
3. 建立內容金鑰並將它與編碼的資產產生關聯。  
4. 設定內容金鑰的授權原則。 指定下列各項：

   * 傳遞方法 (在此例中為 FairPlay)。
   * FairPlay 原則選項組態。 如需有關如何設定 FairPlay 的詳細資訊，請參閱下列範例中的 **ConfigureFairPlayPolicyOptions()** 方法。

     > [!NOTE]
     > 您通常只想要設定一次 FairPlay 原則選項，因為您只會有一組憑證和 ASK。
     >
     >
   * 限制 (開放或權杖)。
   * 金鑰傳遞類型的特定資訊，可定義如何將金鑰傳遞至用戶端。
5. 設定資產傳遞原則。 傳遞原則組態包括︰

   * 傳遞通訊協定 (HLS)。
   * 動態加密類型 (一般 CBC 加密)。
   * 授權取得 URL。

     > [!NOTE]
     > 如果您想要傳遞使用 FairPlay 和另一個 Digital Rights Management (DRM) 系統加密的資料流，則必須設定個別的傳遞原則：
     >
     > * 一個 IAssetDeliveryPolicy 以設定使用一般加密 (CENC) (PlayReady + WideVine) 的 Dynamic Adaptive Streaming over HTTP (DASH)，以及使用 PlayReady 的 Smooth
     > * 另一個 IAssetDeliveryPolicy 以設定 HLS 的 FairPlay
     >
     >
6. 建立隨選定位器以取得串流 URL。

## <a name="use-fairplay-key-delivery-by-player-apps"></a>透過播放程式應用程式使用 FairPlay 金鑰傳遞
您可以使用 iOS SDK 開發播放應用程式。 為了能夠播放 FairPlay 內容，您必須實作授權交換通訊協定。 此通訊協定不是由 Apple 指定。 而是由每個應用程式如何傳送金鑰傳遞要求而決定。 媒體服務 FairPlay 金鑰傳遞服務會對即將到來的 SPC 視為如下列格式的 www-form-url 已編碼張貼訊息：

    spc=<Base64 encoded SPC>

> [!NOTE]
> Azure 媒體播放器支援 FairPlay 播放。 如需詳細資訊，請參閱 [Azure 媒體播放器文件](https://amp.azure.net/libs/amp/latest/docs/index.html) \(英文\)。
>
>

## <a name="streaming-urls"></a>串流 URL
如果使用一個以上 DRM 來加密您的資產，您應該使用串流 URL 中的加密標籤：(format='m3u8-aapl', encryption='xxx')。

您必須考量下列事項：

* 僅可指定零個或一個加密類型。
* 如果只有一個加密套用到資產，則無須在 URL 中指定加密類型。
* 加密類型不區分大小寫。
* 可以指定下列加密類型︰  
  * **cenc**︰一般加密 (PlayReady 或 Widevine)
  * **cbcs-aapl**：FairPlay
  * **cbc**：AES 信封加密

## <a name="create-and-configure-a-visual-studio-project"></a>建立和設定 Visual Studio 專案

1. 設定您的開發環境並在 app.config 檔案中填入連線資訊，如[使用 .NET 進行 Media Services 開發](media-services-dotnet-how-to-use.md)中所述。 
2. 將下列項目新增至 app.config 檔案中定義的 **appSettings**：

    ```xml
    <add key="Issuer" value="http://testissuer.com"/>
    <add key="Audience" value="urn:test"/>
    ```

## <a name="example"></a>範例

下列範例會示範如何使用媒體服務來傳遞使用 FairPlay 加密的內容。 Azure Media Services SDK for .NET 3.6.0 版已引進這項功能。 

以本章節中所顯示的程式碼覆寫 Program.cs 檔案中的程式碼。

>[!NOTE]
>對於不同的 AMS 原則 (例如 Locator 原則或 ContentKeyAuthorizationPolicy) 有 1,000,000 個原則的限制。 如果您一律使用相同的日期 / 存取權限，例如，要長時間維持就地 (非上載原則) 的定位器原則，您應該使用相同的原則識別碼。 如需詳細資訊，請參閱[本篇文章](media-services-dotnet-manage-entities.md#limit-access-policies)。

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
using Microsoft.WindowsAzure.MediaServices.Client.FairPlay;
using Newtonsoft.Json;
using System.Security.Cryptography.X509Certificates;

namespace DynamicEncryptionWithFairPlay
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

            IContentKey key = CreateCommonCBCTypeContentKey(encodedAsset);
            Console.WriteLine("Created key {0} for the asset {1} ", key.Id, encodedAsset.Id);
            Console.WriteLine("FairPlay License Key delivery URL: {0}", key.GetKeyDeliveryUrl(ContentKeyDeliveryType.FairPlay));
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
                // Deserializes a string containing an Xml representation of a TokenRestrictionTemplate
                // back into a TokenRestrictionTemplate class instance.
                TokenRestrictionTemplate tokenTemplate =
                    TokenRestrictionTemplateSerializer.Deserialize(tokenTemplateString);

                // Generate a test token based on the data in the given TokenRestrictionTemplate.
                // Note, you need to pass the key id Guid because we specified
                // TokenClaim.ContentKeyIdentifierClaim in during the creation of TokenRestrictionTemplate.
                Guid rawkey = EncryptionUtils.GetKeyIdAsGuid(key.Id);
                string testToken = TokenRestrictionTemplateSerializer.GenerateTestToken(tokenTemplate, null, rawkey,
                                            DateTime.UtcNow.AddDays(365));
                Console.WriteLine("The authorization token is:\nBearer {0}", testToken);
                Console.WriteLine();
            }

            string url = GetStreamingOriginLocator(encodedAsset);
            Console.WriteLine("Encrypted HLS URL: {0}/manifest(format=m3u8-aapl)", url);

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

            IJob job = _context.Jobs.Create(String.Format("Encoding {0}", inputAsset.Name));

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

        static public IContentKey CreateCommonCBCTypeContentKey(IAsset asset)
        {
            // Create HLS SAMPLE AES encryption content key
            Guid keyId = Guid.NewGuid();
            byte[] contentKey = GetRandomBuffer(16);

            IContentKey key = _context.ContentKeys.Create(
                        keyId,
                        contentKey,
                        "ContentKey",
                        ContentKeyType.CommonEncryptionCbcs);

            // Associate the key with the asset.
            asset.ContentKeys.Add(key);

            return key;
        }


        static public void AddOpenAuthorizationPolicy(IContentKey contentKey)
        {
            // Create ContentKeyAuthorizationPolicy with Open restrictions
            // and create authorization policy          

            List<ContentKeyAuthorizationPolicyRestriction> restrictions = new List<ContentKeyAuthorizationPolicyRestriction>
                    {
                    new ContentKeyAuthorizationPolicyRestriction
                    {
                        Name = "Open",
                        KeyRestrictionType = (int)ContentKeyRestrictionType.Open,
                        Requirements = null
                    }
                    };


            // Configure FairPlay policy option.
            string FairPlayConfiguration = ConfigureFairPlayPolicyOptions();

            IContentKeyAuthorizationPolicyOption FairPlayPolicy =
            _context.ContentKeyAuthorizationPolicyOptions.Create("",
            ContentKeyDeliveryType.FairPlay,
            restrictions,
            FairPlayConfiguration);


            IContentKeyAuthorizationPolicy contentKeyAuthorizationPolicy = _context.
                ContentKeyAuthorizationPolicies.
                CreateAsync("Deliver Common CBC Content Key with no restrictions").
                Result;

            contentKeyAuthorizationPolicy.Options.Add(FairPlayPolicy);

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

            // Configure FairPlay policy option.
            string FairPlayConfiguration = ConfigureFairPlayPolicyOptions();


            IContentKeyAuthorizationPolicyOption FairPlayPolicy =
            _context.ContentKeyAuthorizationPolicyOptions.Create("Token option",
                   ContentKeyDeliveryType.FairPlay,
                   restrictions,
                   FairPlayConfiguration);

            IContentKeyAuthorizationPolicy contentKeyAuthorizationPolicy = _context.
                ContentKeyAuthorizationPolicies.
                CreateAsync("Deliver Common CBC Content Key with token restrictions").
                Result;

            contentKeyAuthorizationPolicy.Options.Add(FairPlayPolicy);

            // Associate the content key authorization policy with the content key
            contentKey.AuthorizationPolicyId = contentKeyAuthorizationPolicy.Id;
            contentKey = contentKey.UpdateAsync().Result;

            return tokenTemplateString;
        }

        private static string ConfigureFairPlayPolicyOptions()
        {
            // For testing you can provide all zeroes for ASK bytes together with the cert from Apple FPS SDK.
            // However, for production you must use a real ASK from Apple bound to a real prod certificate.
            byte[] askBytes = Guid.NewGuid().ToByteArray();
            var askId = Guid.NewGuid();
            // Key delivery retrieves askKey by askId and uses this key to generate the response.
            IContentKey askKey = _context.ContentKeys.Create(
                        askId,
                        askBytes,
                        "askKey",
                        ContentKeyType.FairPlayASk);

            //Customer password for creating the .pfx file.
            string pfxPassword = "<customer password for creating the .pfx file>";
            // Key delivery retrieves pfxPasswordKey by pfxPasswordId and uses this key to generate the response.
            var pfxPasswordId = Guid.NewGuid();
            byte[] pfxPasswordBytes = System.Text.Encoding.UTF8.GetBytes(pfxPassword);
            IContentKey pfxPasswordKey = _context.ContentKeys.Create(
                        pfxPasswordId,
                        pfxPasswordBytes,
                        "pfxPasswordKey",
                        ContentKeyType.FairPlayPfxPassword);

            // iv - 16 bytes random value, must match the iv in the asset delivery policy.
            byte[] iv = Guid.NewGuid().ToByteArray();

            //Specify the .pfx file created by the customer.
            var appCert = new X509Certificate2("path to the .pfx file created by the customer", pfxPassword, X509KeyStorageFlags.Exportable);

            string FairPlayConfiguration =
            Microsoft.WindowsAzure.MediaServices.Client.FairPlay.FairPlayConfiguration.CreateSerializedFairPlayOptionConfiguration(
                appCert,
                pfxPassword,
                pfxPasswordId,
                askId,
                iv);

            return FairPlayConfiguration;
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

        static public void CreateAssetDeliveryPolicy(IAsset asset, IContentKey key)
        {
            var kdPolicy = _context.ContentKeyAuthorizationPolicies.Where(p => p.Id == key.AuthorizationPolicyId).Single();

            var kdOption = kdPolicy.Options.Single(o => o.KeyDeliveryType == ContentKeyDeliveryType.FairPlay);

            FairPlayConfiguration configFP = JsonConvert.DeserializeObject<FairPlayConfiguration>(kdOption.KeyDeliveryConfiguration);

            // Get the FairPlay license service URL.
            Uri acquisitionUrl = key.GetKeyDeliveryUrl(ContentKeyDeliveryType.FairPlay);

            // The reason the below code replaces "https://" with "skd://" is because
            // in the IOS player sample code which you obtained in Apple developer account,
            // the player only recognizes a Key URL that starts with skd://.
            // However, if you are using a customized player,
            // you can choose whatever protocol you want.
            // For example, "https".

            Dictionary<AssetDeliveryPolicyConfigurationKey, string> assetDeliveryPolicyConfiguration =
            new Dictionary<AssetDeliveryPolicyConfigurationKey, string>
            {
                    {AssetDeliveryPolicyConfigurationKey.FairPlayLicenseAcquisitionUrl, acquisitionUrl.ToString().Replace("https://", "skd://")},
                    {AssetDeliveryPolicyConfigurationKey.CommonEncryptionIVForCbcs, configFP.ContentEncryptionIV}
            };

            var assetDeliveryPolicy = _context.AssetDeliveryPolicies.Create(
                "AssetDeliveryPolicy",
            AssetDeliveryPolicyType.DynamicCommonEncryptionCbcs,
            AssetDeliveryProtocol.HLS,
            assetDeliveryPolicyConfiguration);

            // Add AssetDelivery Policy to the asset
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

            // Create a 30-day readonly access policy.
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

## <a name="next-steps-media-services-learning-paths"></a>後續步驟：媒體服務學習路徑
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
