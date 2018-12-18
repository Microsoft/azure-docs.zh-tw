---
title: 教學課程：電子商務目錄仲裁 - Content Moderator
titlesuffix: Azure Cognitive Services
description: 使用機器學習和 AI 自動產生合適的電子商務目錄。
services: cognitive-services
author: sanjeev3
manager: cgronlun
ms.service: cognitive-services
ms.component: content-moderator
ms.topic: tutorial
ms.date: 09/25/2017
ms.author: sajagtap
ms.openlocfilehash: 0bd61c3f1a4f660076be4e87bb5443302e5dc013
ms.sourcegitcommit: 6361a3d20ac1b902d22119b640909c3a002185b3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49363989"
---
# <a name="tutorial-ecommerce-catalog-moderation-with-machine-learning"></a>教學課程：使用機器學習產生合適的電子商務目錄

在本教學課程中，我們會了解如何結合機器輔助 AI 技術和人力合適性做法來實作機器學習型智慧電子商務目錄合適性機制，以提供智慧型目錄系統。

![經過分類的產品影像](images/tutorial-ecommerce-content-moderator.PNG)

## <a name="business-scenario"></a>商務案例

使用機器輔助技術來將產品影像分為下列合適類別：

1. 成人 (裸露)
2. 猥褻 (性暗示)
3. 名人
4. 美國國旗
5. 玩具
6. 筆

## <a name="tutorial-steps"></a>教學課程步驟

本教學課程會引導您完成下列步驟：

1. 註冊並建立「內容仲裁」小組。
2. 針對可能的名人和國旗內容設定合適性標記 (標籤)。
3. 使用「內容仲裁」的影像 API 來掃描可能的成人和猥褻內容。
4. 使用電腦視覺 API 來掃描可能的名人。
5. 使用自訂視覺服務來掃描可能出現的國旗。
6. 提出細微掃描結果以進行人工審核和做出最終決策。

## <a name="create-a-team"></a>建立小組

請參閱[快速入門](quick-start.md)頁面，以註冊「內容仲裁」並建立小組。 請記下 [認證] 頁面中的 [小組識別碼]。


## <a name="define-custom-tags"></a>定義自訂標記

請參閱[標記](https://docs.microsoft.com/azure/cognitive-services/content-moderator/review-tool-user-guide/tags)一文以新增自訂標記。 除了內建的**成人**和**猥褻**標記，新標記可讓檢閱工具顯示標記的描述性名稱。

在我們的案例中，我們會定義這些自訂標記 (**名人**、**國旗**、**美國**、**玩具**、**筆**)：

![設定自訂標記](images/tutorial-ecommerce-tags2.PNG)

## <a name="list-your-api-keys-and-endpoints"></a>列出 API 金鑰與端點

1. 本教學課程會使用三個 API 和對應的金鑰與 API 端點。
2. API 端點會隨訂用帳戶區域與內容仲裁檢閱小組識別碼而不同。

> [!NOTE]
> 本教學課程依設計會使用下列端點中可見區域的訂用帳戶金鑰。 請務必讓 API 金鑰與區域 Uri 相符，否則，金鑰可能不適用於下列端點：

         // Your API keys
        public const string ContentModeratorKey = "XXXXXXXXXXXXXXXXXXXX";
        public const string ComputerVisionKey = "XXXXXXXXXXXXXXXXXXXX";
        public const string CustomVisionKey = "XXXXXXXXXXXXXXXXXXXX";

        // Your end points URLs will look different based on your region and Content Moderator Team ID.
        public const string ImageUri = "https://westus.api.cognitive.microsoft.com/contentmoderator/moderate/v1.0/ProcessImage/Evaluate";
        public const string ReviewUri = "https://westus.api.cognitive.microsoft.com/contentmoderator/review/v1.0/teams/YOURTEAMID/reviews";
        public const string ComputerVisionUri = "https://westcentralus.api.cognitive.microsoft.com/vision/v1.0";
        public const string CustomVisionUri = "https://southcentralus.api.cognitive.microsoft.com/customvision/v1.0/Prediction/XXXXXXXXXXXXXXXXXXXX/url";

## <a name="scan-for-adult-and-racy-content"></a>掃描成人和猥褻內容

1. 此函式會以影像 URL 和金鑰值組陣列作為參數。
2. 它會呼叫「內容仲裁」的影像 API 來取得成人和猥褻分數。
3. 如果分數大於 0.4 (範圍從 0 到 1)，它便會將 **ReviewTags** 陣列中的值設定為 **True**。
4. **ReviewTags** 陣列可用來醒目提示檢閱工具中的對應標記。

        public static bool EvaluateAdultRacy(string ImageUrl, ref KeyValuePair[] ReviewTags)
        {
            float AdultScore = 0;
            float RacyScore = 0;

            var File = ImageUrl;
            string Body = $"{{\"DataRepresentation\":\"URL\",\"Value\":\"{File}\"}}";

            HttpResponseMessage response = CallAPI(ImageUri, ContentModeratorKey, CallType.POST,
                                                   "Ocp-Apim-Subscription-Key", "application/json", "", Body);

            if (response.IsSuccessStatusCode)
            {
                // {“answers”:[{“answer”:“Hello”,“questions”:[“Hi”],“score”:100.0}]}
                // Parse the response body. Blocking!
                GetAdultRacyScores(response.Content.ReadAsStringAsync().Result, out AdultScore, out RacyScore);
            }

            ReviewTags[0] = new KeyValuePair();
            ReviewTags[0].Key = "a";
            ReviewTags[0].Value = "false";
            if (AdultScore > 0.4)
            {
                ReviewTags[0].Value = "true";
            }

            ReviewTags[1] = new KeyValuePair();
            ReviewTags[1].Key = "r";
            ReviewTags[1].Value = "false";
            if (RacyScore > 0.3)
            {
                ReviewTags[1].Value = "true";
            }
            return response.IsSuccessStatusCode;
        }

## <a name="scan-for-celebrities"></a>掃描名人

1. 註冊[電腦視覺 API](https://azure.microsoft.com/services/cognitive-services/computer-vision/) 的[免費試用](https://azure.microsoft.com/try/cognitive-services/?api=computer-vision)。
2. 按一下 [取得 API 金鑰] 按鈕。
3. 接受條款。
4. 若要登入，請從可用的網際網路帳戶清單中選擇。
5. 請記下服務頁面上顯示的 API 金鑰。
    
   ![電腦視覺 API 金鑰](images/tutorial-computer-vision-keys.PNG)
    
6. 請參閱專案的原始程式碼，以取得會使用電腦視覺 API 掃描影像的函式。

         public static bool EvaluateComputerVisionTags(string ImageUrl, string ComputerVisionUri, string ComputerVisionKey, ref KeyValuePair[] ReviewTags)
        {
            var File = ImageUrl;
            string Body = $"{{\"URL\":\"{File}\"}}";

            HttpResponseMessage Response = CallAPI(ComputerVisionUri, ComputerVisionKey, CallType.POST,
                                                   "Ocp-Apim-Subscription-Key", "application/json", "", Body);

            if (Response.IsSuccessStatusCode)
            {
                ReviewTags[2] = new KeyValuePair();
                ReviewTags[2].Key = "cb";
                ReviewTags[2].Value = "false";

                ComputerVisionPrediction CVObject = JsonConvert.DeserializeObject<ComputerVisionPrediction>(Response.Content.ReadAsStringAsync().Result);

                if ((CVObject.categories[0].detail != null) && (CVObject.categories[0].detail.celebrities.Count() > 0))
                {                 
                    ReviewTags[2].Value = "true";
                }
            }

            return Response.IsSuccessStatusCode;
        }

## <a name="classify-into-flags-toys-and-pens"></a>分類為國旗、玩具和筆

1. [登入](https://azure.microsoft.com/services/cognitive-services/custom-vision-service/)至[自訂視覺 API 預覽版](https://www.customvision.ai/)。
2. 使用[快速入門](https://docs.microsoft.com/azure/cognitive-services/custom-vision-service/getting-started-build-a-classifier)來建置自訂分類器，以偵測可能出現的國旗、玩具和筆。
   ![自訂視覺訓練影像](images/tutorial-ecommerce-custom-vision.PNG)
3. 針對自訂分類器[取得預測端點 URL](https://docs.microsoft.com/azure/cognitive-services/custom-vision-service/use-prediction-api)。
4. 請參閱專案的原始程式碼，以查看會呼叫自訂分類器預測端點以掃描影像的函式。

        public static bool EvaluateCustomVisionTags(string ImageUrl, string CustomVisionUri, string CustomVisionKey, ref KeyValuePair[] ReviewTags)
        {
            var File = ImageUrl;
            string Body = $"{{\"URL\":\"{File}\"}}";

            HttpResponseMessage response = CallAPI(CustomVisionUri, CustomVisionKey, CallType.POST,
                                                   "Prediction-Key", "application/json", "", Body);

            if (response.IsSuccessStatusCode)
            {
                // Parse the response body. Blocking!
                SaveCustomVisionTags(response.Content.ReadAsStringAsync().Result, ref ReviewTags);
            }
            return response.IsSuccessStatusCode;
        }       
 
## <a name="reviews-for-human-in-the-loop"></a>人機互動檢閱

1. 在前面幾節中，您掃描了傳入的影像中是否有成人和猥褻 (內容仲裁)、名人 (電腦視覺) 和國旗 (自訂版本)。
2. 根據每個掃描的相符閾值，在檢閱工具中讓細微的案例可供進行人工檢閱。
        public static bool CreateReview(string ImageUrl, KeyValuePair[] Metadata) {

            ReviewCreationRequest Review = new ReviewCreationRequest();
            Review.Item[0] = new ReviewItem();
            Review.Item[0].Content = ImageUrl;
            Review.Item[0].Metadata = new KeyValuePair[MAXTAGSCOUNT];
            Metadata.CopyTo(Review.Item[0].Metadata, 0);

            //SortReviewItems(ref Review);

            string Body = JsonConvert.SerializeObject(Review.Item);

            HttpResponseMessage response = CallAPI(ReviewUri, ContentModeratorKey, CallType.POST,
                                                   "Ocp-Apim-Subscription-Key", "application/json", "", Body);

            return response.IsSuccessStatusCode;
        }

## <a name="submit-batch-of-images"></a>提交一批影像

1. 本教學課程會假設有一個 "C:Test" 目錄，以及一個有影像 Url 清單的文字檔。
2. 下列程式碼會檢查該檔案是否存在，並將所有 Url 讀入記憶體。
            // 檢查測試目錄中是否有內含所要掃描影像 URL 清單的文字檔，var topdir = @"C:\test\"; var Urlsfile = topdir + "Urls.txt";

            if (!Directory.Exists(topdir))
                return;

            if (!File.Exists(Urlsfile))
            {
                return;
            }

            // Read all image URLs in the file
            var Urls = File.ReadLines(Urlsfile);

## <a name="initiate-all-scans"></a>起始所有掃描

1. 這個最上層函式會對先前所述文字檔中的所有影像 URL 執行迴圈。
2. 它會使用每個 API 來掃描 URL，如果相符可信度落在條件內，則會建立人工仲裁審核。
             // for each image URL in the file... foreach (var Url in Urls) { // Initiatize a new review tags array ReviewTags = new KeyValuePair[MAXTAGSCOUNT];

                // Evaluate for potential adult and racy content with Content Moderator API
                EvaluateAdultRacy(Url, ref ReviewTags);

                // Evaluate for potential presence of celebrity (ies) in images with Computer Vision API
                EvaluateComputerVisionTags(Url, ComputerVisionUri, ComputerVisionKey, ref ReviewTags);

                // Evaluate for potential presence of custom categories other than Marijuana
                EvaluateCustomVisionTags(Url, CustomVisionUri, CustomVisionKey, ref ReviewTags);

                // Create review in the Content Moderator review tool
                CreateReview(Url, ReviewTags);
            }

## <a name="license"></a>授權

所有 Microsoft 認知服務 SDK 和範例都是透過 MIT 授權來獲得授權的。 如需詳細資料，請參閱[授權](https://microsoft.mit-license.org/) \(英文\)。

## <a name="developer-code-of-conduct"></a>開發人員行為準則

使用「認知服務」(包括此用戶端程式庫和範例) 的開發人員應當遵守 http://go.microsoft.com/fwlink/?LinkId=698895 中的「Microsoft 認知服務的開發人員行為準則」。

## <a name="next-steps"></a>後續步驟

使用 GitHub 上的[專案原始程式檔](https://github.com/MicrosoftContentModerator/samples-eCommerceCatalogModeration)來建置和延伸本教學課程。
