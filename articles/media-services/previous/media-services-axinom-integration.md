---
title: 使用 Axinom 將 Widevine 授權傳遞到 Azure 媒體服務 | Microsoft Docs
description: 本文說明如何使用 Azure 媒體服務 (AMS) 來傳遞 AMS 使用 PlayReady 與 Widevine DRM 動態加密的資料流。 PlayReady 授權來自媒體服務 PlayReady 授權伺服器，Widevine 授權由 Axinom 授權伺服器傳遞。
services: media-services
documentationcenter: ''
author: willzhan
manager: cfowler
editor: ''
ms.assetid: 9c93fa4e-b4da-4774-ab6d-8b12b371631d
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/19/2017
ms.author: willzhan;Mingfeiy;rajputam;Juliako
ms.openlocfilehash: 81247863eb86752113989f6e48e79f5c8bc75505
ms.sourcegitcommit: f06925d15cfe1b3872c22497577ea745ca9a4881
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37061149"
---
# <a name="using-axinom-to-deliver-widevine-licenses-to-azure-media-services"></a>使用 Axinom 將 Widevine 授權傳遞到 Azure 媒體服務
> [!div class="op_single_selector"]
> * [castLabs](media-services-castlabs-integration.md)
> * [Axinom](media-services-axinom-integration.md)
> 
> 

## <a name="overview"></a>概觀
Azure 媒體服務 (AMS) 已新增Google Widevine 動態保護 (如需詳細資訊，請參閱 [Mingfei 的部落格](https://azure.microsoft.com/blog/azure-media-services-adds-google-widevine-packaging-for-delivering-multi-drm-stream/) )。 此外，Azure 媒體播放器 (AMP) 也已新增 Widevine 支援 (如需詳細資訊，請參閱 [AMP 文件](http://amp.azure.net/libs/amp/latest/docs/) )。 談到在配備 MSE 和 EME 的現代瀏覽器上串流處理受到 CENC 與多重原生 DRM (PlayReady 和 Widevine) 保護的 DASH 內容時，這可說是一大成就。

從媒體服務 .NET SDK 版本 3.5.2 開始，媒體服務讓您可設定 Widevine 授權範本並取得 Widevine 授權。 您也可以使用下列 AMS 合作夥伴來協助您傳遞 Widevine 授權：[Axinom](http://www.axinom.com/press/ibc-axinom-drm-6/)、[EZDRM](http://ezdrm.com/)、[castLabs](http://castlabs.com/company/partners/azure/)。

本文說明如何整合和測試受 Axinom 管理的 Widevine 授權伺服器。 具體而言，其內容包括：  

* 使用多重 DRM (PlayReady 和 Widevine) 與對應的授權取得 URL，設定動態一般加密；
* 產生 JWT 權杖以符合授權伺服器需求；
* 開發可使用 JWT 權杖驗證處理授權取得作業的 Azure 媒體播放器應用程式；

完整的系統和內容金鑰、金鑰識別碼、金鑰種子、JTW 權杖及其宣告，皆可透過下圖詳盡說明：

![DASH 和 CENC](./media/media-services-axinom-integration/media-services-axinom1.png)

## <a name="content-protection"></a>內容保護
若要設定動態保護和金鑰傳遞原則，請參閱 Mingfei 的部落格： [如何使用 Azure 媒體服務設定 Widevine 封裝](http://mingfeiy.com/how-to-configure-widevine-packaging-with-azure-media-services)(英文)。

您可以使用多重 DRM 為同時具有下列二者的 DASH 串流設定動態 CENC 保護：

1. MS Edge 和 IE11 的 PlayReady 保護，可能有權杖授權限制。 權杖限制原則必須伴隨著安全權杖服務 (STS) 所發出的權杖，例如 Azure Active Directory；
2. Chrome 的 Widevine 保護，它可能需要對其他 STS 所發行的權杖進行權杖驗證。 

請參閱 [JWT 權杖產生](media-services-axinom-integration.md#jwt-token-generation)一節，了解 Azure Active Directory 為何無法作為 Axinom Widevine 授權伺服器的 STS。

### <a name="considerations"></a>考量
1. 您必須使用 Axinom 指定的金鑰種子 (8888000000000000000000000000000000000000) 和您產生或選取的金鑰識別碼，產生用以設定金鑰傳遞服務的內容金鑰。 Axinom 授權伺服器會根據相同的金鑰種子 (同時適用於測試和生產環境)，發行包含內容金鑰的所有授權。
2. 測試用的 Widevine 授權取得 URL：[https://drm-widevine-licensing.axtest.net/AcquireLicense](https://drm-widevine-licensing.axtest.net/AcquireLicense)。 HTTP 與 HTTS 皆可使用。

## <a name="azure-media-player-preparation"></a>準備 Azure 媒體播放器
AMP 1.4.0 版支援同時使用 PlayReady 和 Widevine DRM 動態封裝的 AMS 內容進行播放。
如果 Widevine 授權伺服器不需要權杖驗證，則不需要執行任何其他動作即可測試受到 Widevine 保護的 DASH 內容。 例如，AMP 團隊提供簡單的 [範例](https://amp.azure.net/libs/amp/latest/samples/dynamic_multiDRM_PlayReadyWidevineFairPlay_notoken.html)，在此您可以看到在 Edge 和 IE11 中搭配 PlayReady 以及在 Chrome 中搭配 Widevine 皆順利運作。
Axinom 提供的 Widevine 授權伺服器需要 JWT 權杖驗證。 JWT 權杖必須使用透過 HTTP 標頭 “X-AxDRM-Message” 發出的授權要求來提交。 為此，您必須在設定來源之前，在裝載 AMP 的網頁中新增下列 Javascript：

    <script>AzureHtml5JS.KeySystem.WidevineCustomAuthorizationHeader = "X-AxDRM-Message"</script>

AMP 程式碼的其餘部分是標準 AMP API，如 [這裡](http://amp.azure.net/libs/amp/latest/docs/)的 AMP 文件所說明。

上述用來設定自訂授權標頭的 Javascript 是在 AMP 中正式發行長期方法之前所使用的短期方法。

## <a name="jwt-token-generation"></a>JWT 權杖產生
測試用的 Axinom Widevine 授權伺服器需要 JWT 權杖驗證。 此外，JWT 權杖中的其中一個宣告屬於複雜物件類型，而非基本資料類型。

可惜的是，Azure AD 只能發行具有基本類型的 JWT 權杖。 同樣地，.NET Framework API (System.IdentityModel.Tokens.SecurityTokenHandler 和 JwtPayload) 也只能讓您輸入複雜物件類型做為宣告。 不過，這些宣告仍會序列化為字串。 因此，這兩者都無法用來產生 Widevine 授權要求的 JWT 權杖。

John Sheehan 的 [JWT NuGet 套件](https://www.nuget.org/packages/JWT) 符合這些需求，因此我們將使用此 NuGet 套件。

下列程式碼會產生 JWT 權杖，且具有 Axinom Widevine 授權伺服器進行測試時所需的必要宣告：

    using System;
    using System.Collections.Generic;
    using System.Linq;
    using System.Web;
    using System.IdentityModel.Tokens;
    using System.IdentityModel.Protocols.WSTrust;
    using System.Security.Claims;

    namespace OpenIdConnectWeb.Utils
    {
        public class JwtUtils
        {
            //using John Sheehan's NuGet JWT library: https://www.nuget.org/packages/JWT/
            public static string CreateJwtSheehan(string symmetricKeyHex, string key_id)
            {
                byte[] symmetricKey = ConvertHexStringToByteArray(symmetricKeyHex);  //hex string to byte[] Note: Note that the key is a hex string, however it must be treated as a series of bytes not a string when encoding.

                var payload = new Dictionary<string, object>()
                             {
                                 { "version", 1 },
                                 { "com_key_id", System.Configuration.ConfigurationManager.AppSettings["ax:com_key_id"] },
                                 { "message", new { type = "entitlement_message", key_ids = new string[] { key_id } }  }
                             };

                string token = JWT.JsonWebToken.Encode(payload, symmetricKey, JWT.JwtHashAlgorithm.HS256);

                return token;
            }

            //convert hex string to byte[]
            public static byte[] ConvertHexStringToByteArray(string hexString)
            {
                if (hexString.Length % 2 != 0)
                {
                    throw new ArgumentException(String.Format(System.Globalization.CultureInfo.InvariantCulture, "The binary key cannot have an odd number of digits: {0}", hexString));
                }

                byte[] HexAsBytes = new byte[hexString.Length / 2];
                for (int index = 0; index < HexAsBytes.Length; index++)
                {
                    string byteValue = hexString.Substring(index * 2, 2);
                    HexAsBytes[index] = byte.Parse(byteValue, System.Globalization.NumberStyles.HexNumber, System.Globalization.CultureInfo.InvariantCulture);
                }

                return HexAsBytes;
            }

        }  

    }  

Axinom Widevine 授權伺服器

    <add key="ax:laurl" value="http://drm-widevine-licensing.axtest.net/AcquireLicense" />
    <add key="ax:com_key_id" value="69e54088-e9e0-4530-8c1a-1eb6dcd0d14e" />
    <add key="ax:com_key" value="4861292d027e269791093327e62ceefdbea489a4c7e5a4974cc904b840fd7c0f" />
    <add key="ax:keyseed" value="8888000000000000000000000000000000000000" />

### <a name="considerations"></a>考量
1. 即使 AMS PlayReady 授權傳遞服務會要求驗證權杖之前必須有 “Bearer=”，Axinom Widevine 授權伺服器並不會加以使用。
2. Axinom 通訊金鑰會做為簽署金鑰。 此金鑰是十六進位字串，但在編碼時必須將其視為一系列的位元組，而不是字串。 這可藉由 ConvertHexStringToByteArray 方法來達成。

## <a name="retrieving-key-id"></a>擷取金鑰識別碼
您可能已注意到，在產生 JWT 權杖的程式碼中，金鑰識別碼是必要項目。 由於 JWT 權杖必須在載入 AMP 播放程式之前備妥，因此必須要擷取金鑰識別碼，才能產生 JWT 權杖。

當然，有多種方式可用來取得金鑰識別碼。 比方說，有人可能會將金鑰識別碼與內容中繼資料一起儲存在資料庫中。 或者，您可以從 DASH MPD (媒體顯示說明) 檔案擷取金鑰識別碼。 以下程式碼適用於後者。

    //get key_id from DASH MPD
    public static string GetKeyID(string dashUrl)
    {
        if (!dashUrl.EndsWith("(format=mpd-time-csf)"))
        {
            dashUrl += "(format=mpd-time-csf)";
        }

        XPathDocument objXPathDocument = new XPathDocument(dashUrl);
        XPathNavigator objXPathNavigator = objXPathDocument.CreateNavigator();
        XmlNamespaceManager objXmlNamespaceManager = new XmlNamespaceManager(objXPathNavigator.NameTable);
        objXmlNamespaceManager.AddNamespace("",     "urn:mpeg:dash:schema:mpd:2011");
        objXmlNamespaceManager.AddNamespace("ns1",  "urn:mpeg:dash:schema:mpd:2011");
        objXmlNamespaceManager.AddNamespace("cenc", "urn:mpeg:cenc:2013");
        objXmlNamespaceManager.AddNamespace("ms",   "urn:microsoft");
        objXmlNamespaceManager.AddNamespace("mspr", "urn:microsoft:playready");
        objXmlNamespaceManager.AddNamespace("xsi",  "http://www.w3.org/2001/XMLSchema-instance");
        objXmlNamespaceManager.PushScope();

        XPathNodeIterator objXPathNodeIterator;
        objXPathNodeIterator = objXPathNavigator.Select("//ns1:MPD/ns1:Period/ns1:AdaptationSet/ns1:ContentProtection[@value='cenc']", objXmlNamespaceManager);

        string key_id = string.Empty;
        if (objXPathNodeIterator.MoveNext())
        {
            key_id = objXPathNodeIterator.Current.GetAttribute("default_KID", "urn:mpeg:cenc:2013");
        }

        return key_id;
    }

## <a name="summary"></a>總結
透過 Azure 媒體服務內容保護和 Azure 媒體播放器中最新版的 Widevine 支援，我們得以同時使用 AMS 中的 PlayReady 授權服務和 Axinom 的 Widevine 授權伺服器，為下列現代瀏覽器實作 DASH + 多重原生 DRM (PlayReady + Widevine) 的串流：

* Chrome
* Windows 10 上的 Microsoft Edge
* Windows 8.1 和 Windows 10 上的 IE 11
* Firefox (桌面) 和 Mac (非 iOS) 上的 Safari 也都可透過 Silverlight 和與 Azure 媒體播放器相同的 URL 受到支援

以下是運用 Axinom Widevine 授權伺服器的迷你解決方案所需的參數。 除了金鑰識別碼以外，Axinom 會根據 Widevine 伺服器安裝來提供其餘參數。

| 參數 | 使用方式 |
| --- | --- |
| 通訊金鑰識別碼 |必須包含在 JWT 權杖中作為宣告 "com_key_id" 的值 (請參閱 [本節](media-services-axinom-integration.md#jwt-token-generation))。 |
| 通訊金鑰 |必須做為 JWT 權杖的簽署金鑰 (請參閱 [本節](media-services-axinom-integration.md#jwt-token-generation) )。 |
| 金鑰種子 |必須用來使用任何指定的內容金鑰識別碼來產生內容金鑰 (請參閱 [本節](media-services-axinom-integration.md#content-protection))。 |
| Widevine 授權取得 URL |必須用於設定 DASH 串流資產傳遞原則 (請參閱[本節](media-services-axinom-integration.md#content-protection))。 |
| 內容金鑰識別碼 |必須包含其中作為 JWT 權杖之權利訊息宣告值的一部分 (請參閱 [本節](media-services-axinom-integration.md#jwt-token-generation) )。 |

## <a name="media-services-learning-paths"></a>媒體服務學習路徑
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

### <a name="acknowledgments"></a>通知
我們想要向下列為建立此文件貢獻心力的人員致謝：Axinom 的 Kristjan Jõgi、Mingfei Yan 和 Amit Rajput。

