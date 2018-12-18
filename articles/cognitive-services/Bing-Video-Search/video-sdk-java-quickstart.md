---
title: 快速入門：Bing 影片搜尋 SDK (Java)
titleSuffix: Azure Cognitive Services
description: 了解如何設定 Bing 影片搜尋 SDK 主控台應用程式。
services: cognitive-services
author: mikedodaro
manager: cgronlun
ms.service: cognitive-services
ms.component: bing-video-search
ms.topic: quickstart
ms.date: 02/18/2018
ms.author: rosh
ms.openlocfilehash: b0e083a7397378956d9fe0d0ae2257aaf0bbdf1e
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47223408"
---
# <a name="quickstart-bing-video-search-sdk-java"></a>快速入門：Bing 影片搜尋 SDK Java

Bing 影片搜尋 SDK 提供用於影片查詢以及剖析結果的 REST API 功能。

[Java Bing 影片搜尋 SDK 範例的原始程式碼](https://github.com/Azure-Samples/cognitive-services-java-sdk-samples/tree/master/Search/BingVideoSearch)可從 Git Hub 取得。

## <a name="application-dependencies"></a>應用程式相依性
在「搜尋」下取得[認知服務存取金鑰](https://azure.microsoft.com/try/cognitive-services/)。 使用 Maven、Gradle 或另一個相依性管理系統，來安裝 Bing 影片搜尋 SDK 相依性。 Maven POM 檔案需要宣告：
```
  <dependencies>
    <dependency>
      <groupId>com.microsoft.azure.cognitiveservices</groupId>
      <artifactId>azure-cognitiveservices-videosearch</artifactId>
      <version>0.0.1-beta-SNAPSHOT</version>
    </dependency>
  </dependencies> 
```
## <a name="video-search-client"></a>影片搜尋用戶端
將匯入新增至類別實作。
```
import com.microsoft.azure.cognitiveservices.videosearch.*;
import com.microsoft.azure.cognitiveservices.videosearch.Freshness;
import com.microsoft.azure.cognitiveservices.videosearch.VideoObject;
import com.microsoft.azure.cognitiveservices.videosearch.implementation.TrendingVideosInner;
import com.microsoft.azure.cognitiveservices.videosearch.implementation.VideoDetailsInner;
import com.microsoft.azure.cognitiveservices.videosearch.implementation.VideoSearchAPIImpl;
import com.microsoft.azure.cognitiveservices.videosearch.implementation.VideosInner;
import com.microsoft.rest.credentials.ServiceClientCredentials;
import okhttp3.Interceptor;
import okhttp3.OkHttpClient;
import okhttp3.Request;
import okhttp3.Response;
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

```
實作 **VideoSearchAPIImpl** 用戶端，而此用戶端需要 **ServiceClientCredentials** 類別執行個體。
```
public static VideoSearchAPIImpl getClient(final String subscriptionKey) {
    return new VideoSearchAPIImpl("https://api.cognitive.microsoft.com/bing/v7.0/",
            new ServiceClientCredentials() {
                @Override
                public void applyCredentialsFilter(OkHttpClient.Builder builder) {
                    builder.addNetworkInterceptor(
                            new Interceptor() {
                                @Override
                                public Response intercept(Chain chain) throws IOException {
                                    Request request = null;
                                    Request original = chain.request();
                                    // Request customization: add request headers
                                    Request.Builder requestBuilder = original.newBuilder()
                                            .addHeader("Ocp-Apim-Subscription-Key", subscriptionKey);
                                    request = requestBuilder.build();
                                    return chain.proceed(request);
                                }
                            });
                }
            });
}
```
搜尋關於 "SwiftKey" 的影片。 驗證結果數目。 列印第一個影片結果的識別碼、名稱和 URL。
```
public static void VideoSearch(String subscriptionKey)
{
    VideoSearchAPIImpl client = VideoSDK.getClient(subscriptionKey);

    try
    {
        VideosInner videoResults = client.searchs().list("SwiftKey");
        System.out.println("\r\nSearch videos for query \"SwiftKey\"");

        if (videoResults == null)
        {
            System.out.println("Didn't see any video result data..");
        }
        else
        {
            if (videoResults.value().size() > 0)
            {
                VideoObject firstVideoResult = videoResults.value().get(0);

                System.out.println(String.format("Video result count: %d", videoResults.value().size()));
                System.out.println(String.format("First video id: %s", firstVideoResult.videoId()));
                System.out.println(String.format("First video name: %s", firstVideoResult.name()));
                System.out.println(String.format("First video url: %s", firstVideoResult.contentUrl()));
            }
            else
            {
                System.out.println("Couldn't find video results!");
            }
        }
    }

    catch (ErrorResponseException ex)
    {
        System.out.println("Encountered exception. " + ex.getLocalizedMessage());
    }

}


```
搜尋關於 "Bellevue Trailer" 的影片。 使用*免費*、*簡短*和 *1080p 解析度*等參數進行篩選和搜尋。 驗證結果數目。 列印第一個影片結果的識別碼、名稱和 URL。
```
public static void VideoSearchWithFilters(String subscriptionKey)
{
    VideoSearchAPIImpl client = VideoSDK.getClient(subscriptionKey);

    try
    {
        VideosInner videoResults = client.searchs().list("Bellevue Trailer", "en-us", null, null, null, null, null,
                Freshness.MONTH, null, VideoLength.SHORT, "en-us", null, VideoPricing.FREE, VideoResolution.HD1080P, null, null, null, null);
        System.out.println("\r\nSearch videos for query \"Bellevue Trailer\" free, short and 1080p resolution");

        if (videoResults == null)
        {
            System.out.println("Didn't see any video result data..");
        }
        else
        {
            if (videoResults.value().size() > 0)
            {
                VideoObject firstVideoResult = videoResults.value().get(0);
                System.out.println(String.format("Video result count: %d", videoResults.value().size()));
                System.out.println(String.format("First video id: %s", firstVideoResult.videoId()));
                System.out.println(String.format("First video name: %s", firstVideoResult.name()));
                System.out.println(String.format("First video url: %s", firstVideoResult.contentUrl()));
            }
            else
            {
                System.out.println("Couldn't find video results!");
            }
        }
    }

    catch (ErrorResponseException ex)
    {
        System.out.println("Encountered exception. " + ex.getLocalizedMessage());
    }
}

```
搜尋趨勢影片。 驗證 **bannerTiles** 和 **categories** 參數。
```
public static void VideoTrending(String subscriptionKey)
{
    VideoSearchAPIImpl client = VideoSDK.getClient(subscriptionKey);

    try
    {
        TrendingVideosInner trendingResults = client.trendings().list();
        System.out.println("\r\nSearch trending videos");

        if (trendingResults == null)
        {
            System.out.println("Didn't see any trending video data..");
        }
        else
        {
            // Verify the banner tiles
            if (trendingResults.bannerTiles().size() > 0)
            {
                TrendingVideosTile firstBannerTile = trendingResults.bannerTiles().get(0);
                System.out.println(
                        String.format("Banner tile count: {trendingResults.BannerTiles.Count}"));
                System.out.println(
                        String.format("First banner tile text: {firstBannerTile.Query.Text}"));
                System.out.println(
                        String.format("First banner tile url: {firstBannerTile.Query.WebSearchUrl}"));
            }
            else
            {
                System.out.println("Couldn't find banner tiles!");
            }

            // Verify the categories
            if (trendingResults.categories().size() > 0)
            {
                TrendingVideosCategory firstCategory = trendingResults.categories().get(0);
                System.out.println(
                        String.format("Category count: %d", trendingResults.categories().size()));
                System.out.println(
                        String.format("First category title: %s", firstCategory.title()));

                if (firstCategory.subcategories().size() > 0)
                {
                    TrendingVideosSubcategory firstSubCategory = firstCategory.subcategories().get(0);
                    System.out.println(
                            String.format("SubCategory count: %d", firstCategory.subcategories().size()));
                    System.out.println(
                            String.format("First sub category title: %s", firstSubCategory.title()));

                    if (firstSubCategory.tiles().size() > 0)
                    {
                        TrendingVideosTile firstTile = firstSubCategory.tiles().get(0);
                        System.out.println(
                                String.format("Tile count: %d", firstSubCategory.tiles().size()));
                        System.out.println(
                                String.format("First tile text: %s", firstTile.query().text()));
                        System.out.println(
                                String.format("First tile url: %s", firstTile.query().webSearchUrl()));
                    }
                    else
                    {
                        System.out.println("Couldn't find tiles!");
                    }
                }
                else
                {
                    System.out.println("Couldn't find subcategories!");
                }
            }
            else
            {
                System.out.println("Couldn't find categories!");
            }
        }
    }

    catch (ErrorResponseException ex)
    {
        System.out.println("Encountered exception. " + ex.getLocalizedMessage());
    }

}

```
搜尋關於 "Bellevue Trailer" 的影片，然後搜尋第一個影片結果的相關詳細資料。
```
public static void VideoDetail(String subscriptionKey)
{
    VideoSearchAPIImpl client = VideoSDK.getClient(subscriptionKey);

    try
    {
        VideosInner videoResults = client.searchs().list("Bellevue Trailer");
        if (videoResults.value().size() > 0)
        {
            VideoObject firstVideo = videoResults.value().get(0);
            List<VideoInsightModule> modules = new ArrayList<VideoInsightModule>();
            modules.add(VideoInsightModule.ALL);
            VideoDetailsInner videoDetail = client.details().list("Bellevue Trailer", null, null, null, null, null,
                    firstVideo.videoId(), modules, "en-us", null, null, null, null, null);
                    //nc(query: "Bellevue Trailer", id: firstVideo.VideoId, modules: modules).Result;
            System.out.println(
                    String.format("\r\nSearch detail for video id={firstVideo.VideoId}, name=%s", firstVideo.name()));

            if (videoDetail != null)
            {
                if (videoDetail.videoResult() != null)
                {
                    System.out.println(
                            String.format("Expected video id: %s", videoDetail.videoResult().videoId()));
                    System.out.println(
                            String.format("Expected video name: %s", videoDetail.videoResult().name()));
                    System.out.println(
                            String.format("Expected video url: %s", videoDetail.videoResult().contentUrl()));
                }
                else
                {
                    System.out.println("Couldn't find expected video!");
                }

                if (videoDetail.relatedVideos().value().size() > 0)
                {
                    VideoObject firstRelatedVideo = videoDetail.relatedVideos().value().get(0);
                    System.out.println(
                            String.format("Related video count: %d", videoDetail.relatedVideos().value().size() ));
                    System.out.println(
                            String.format("First related video id: %s", firstRelatedVideo.videoId()));
                    System.out.println(
                            String.format("First related video name: %s", firstRelatedVideo.name()));
                    System.out.println(
                            String.format("First related video url: %s", firstRelatedVideo.contentUrl()));
                }
                else
                {
                    System.out.println("Couldn't find any related video!");
                }
            }
            else
            {
                System.out.println("Couldn't find detail about the video!");
            }
        }
        else
        {
            System.out.println("Couldn't find video results!");
        }
    }

    catch (ErrorResponseException ex)
    {
        System.out.println("Encountered exception. " + ex.getLocalizedMessage());
    }
}
```
使用用於執行程式碼的 main 函式，將本文所述的方法新增至類別。
```
package videoSDK;
import com.microsoft.azure.cognitiveservices.videosearch.*;

public class VideoSDK {

    public static void main(String[] args) {
        
        
        VideoSDK.VideoSearch("YOUR-SUBSCRIPTION-KEY");
        VideoSDK.VideoSearchWithFilters("YOUR-SUBSCRIPTION-KEY");
        VideoSDK.VideoTrending("YOUR-SUBSCRIPTION-KEY");
        VideoSDK.VideoDetail("YOUR-SUBSCRIPTION-KEY");

    }

    // Include the methods described in this article.
}
```
## <a name="next-steps"></a>後續步驟

[認知服務 Java SDK 範例](https://github.com/Azure-Samples/cognitive-services-java-sdk-samples)
