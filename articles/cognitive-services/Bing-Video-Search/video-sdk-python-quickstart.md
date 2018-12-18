---
title: 快速入門：影片搜尋 SDK (Python)
titleSuffix: Azure Cognitive Services
description: 設定影片搜尋 SDK 主控台應用程式。
services: cognitive-services
author: mikedodaro
manager: cgronlun
ms.service: cognitive-services
ms.component: bing-video-search
ms.topic: quickstart
ms.date: 02/15/2018
ms.author: rosh
ms.openlocfilehash: 9784894ea9a9deb350171e0d19042eb65644093b
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47225645"
---
# <a name="quickstart-bing-video-search-sdk-with-python"></a>快速入門：使用 Python 的 Bing 影片搜尋 SDK

Bing 影像搜尋 SDK 包含用於 Web 查詢以及剖析結果的 REST API 功能。

[Python Bing 影片搜尋 SDK 範例的原始程式碼](https://github.com/Azure-Samples/cognitive-services-python-sdk-samples/blob/master/samples/search/video_search_samples.py)可從 Git Hub 取得。


## <a name="application-dependencies"></a>應用程式相依性
若還沒有 Python，請安裝。 SDK 可與 Python 2.7、3.3、3.4、3.5 和 3.6 相容。

Python 開發的一般建議是使用[虛擬環境](https://docs.python.org/3/tutorial/venv.html)。 使用 [venv 模組](https://pypi.python.org/pypi/virtualenv) 安裝和初始化虛擬環境。 安裝適用於 Python 2.7 的 virtualenv。
```
python -m venv mytestenv
```
安裝 Bing 影片搜尋 SDK 相依性：
```
cd mytestenv
python -m pip install azure-cognitiveservices-search-videosearch
```
## <a name="video-search-client"></a>影片搜尋用戶端
在「搜尋」下取得[認知服務存取金鑰](https://azure.microsoft.com/try/cognitive-services/)。 新增匯入：
```
subscription_key = "YOUR-SUBSCRIPTION-KEY"
```
建立 `CognitiveServicesCredentials` 的執行個體，並具現化用戶端：
```
client = VideoSearchAPI(CognitiveServicesCredentials(subscription_key))
```
搜尋 (SwiftKey) 的影片，然後驗證結果數目。 列印出第一個影片結果的 `ID`、`name` 和 `URL`。
```
client = VideoSearchAPI(CognitiveServicesCredentials(subscription_key))

try:
    video_result = client.videos.search(query="SwiftKey")
    print("Search videos for query \"SwiftKey\"")

    if video_result.value:
        first_video_result = video_result.value[0]
        print("Video result count: {}".format(len(video_result.value)))
        print("First video id: {}".format(first_video_result.video_id))
        print("First video name: {}".format(first_video_result.name))
        print("First video url: {}".format(first_video_result.content_url))
    else:
        print("Didn't see any video result data..")

except Exception as err:
    print("Encountered exception. {}".format(err))

```
針對 (Bellevue Trailer) 搜尋免費、簡短且解析度為 1080p 的影片。 驗證結果數目，並列印出第一個影片結果的 `ID`、`name` 和 `URL`。
```
def video_search_with_filtering(subscription_key):

    client = VideoSearchAPI(CognitiveServicesCredentials(subscription_key))

    try:
        video_result = client.videos.search(
            query="Bellevue Trailer",
            pricing=VideoPricing.free,  # Can use the str "free" too
            length=VideoLength.short,   # Can use the str "short" too
            resolution=VideoResolution.hd1080p  # Can use the str "hd1080p" too
        )
        print("Search videos for query \"Bellevue Trailer\" that is free, short and 1080p resolution")

        if video_result.value:
            first_video_result = video_result.value[0]
            print("Video result count: {}".format(len(video_result.value)))
            print("First video id: {}".format(first_video_result.video_id))
            print("First video name: {}".format(first_video_result.name))
            print("First video url: {}".format(first_video_result.content_url))
        else:
            print("Didn't see any video result data..")

    except Exception as err:
        print("Encountered exception. {}".format(err))

```

取得趨勢結果。 驗證橫幅圖格和類別：
```
def video_trending(subscription_key):

    client = VideoSearchAPI(CognitiveServicesCredentials(subscription_key))

    try:
        trending_result = client.videos.trending()
        print("Search trending video")

        # Banner tiles
        if trending_result.banner_tiles:
            first_banner_tile = trending_result.banner_tiles[0]
            print("Banner tile count: {}".format(len(trending_result.banner_tiles)))
            print("First banner tile text: {}".format(first_banner_tile.query.text))
            print("First banner tile url: {}".format(first_banner_tile.query.web_search_url))
        else:
            print("Couldn't find banner tiles!")

        # Categorires
        if trending_result.categories:
            first_category = trending_result.categories[0]
            print("Category count: {}".format(len(trending_result.categories)))
            print("First category title: {}".format(first_category.title))
            if first_category.subcategories:
                first_subcategory = first_category.subcategories[0]
                print("Subcategory count: {}".format(len(first_category.subcategories)))
                print("First subcategory title: {}".format(first_subcategory.title))
                if first_subcategory.tiles:
                    first_tile = first_subcategory.tiles[0]
                    print("Subcategory tile count: {}".format(len(first_subcategory.tiles)))
                    print("First tile text: {}".format(first_tile.query.text))
                    print("First tile url: {}".format(first_tile.query.web_search_url))
                else:
                    print("Couldn't find subcategory tiles!")
            else:
                print("Couldn't find subcategories!")
        else:
            print("Couldn't find categories!")

    except Exception as err:
        print("Encountered exception. {}".format(err))

```
搜尋 (Bellevue Trailer) 的影片，然後搜尋第一個影片的詳細資訊。
```
def video_detail(subscription_key):

    client = VideoSearchAPI(CognitiveServicesCredentials(subscription_key))

    try:
        video_result = client.videos.search(query="Bellevue Trailer")
        first_video_result = video_result.value[0]

        video_details = client.videos.details(
            query="Bellevue Trailer",
            id=first_video_result.video_id,
            modules=[VideoInsightModule.all]  # Can use ["all"] too
        )
        print("Search detail for video id={}, name={}".format(
            first_video_result.video_id,
            first_video_result.name
        ))

        if video_details.video_result:
            print("Expected Video id: {}".format(video_details.video_result.video_id))
            print("Expected Video name: {}".format(video_details.video_result.name))
            print("Expected Video url: {}".format(video_details.video_result.content_url))
        else:
            print("Couldn't find expected video")

        if video_details.related_videos.value:
            first_related_video = video_details.related_videos.value[0]
            print("Related video count: {}".format(len(video_details.related_videos.value)))
            print("First related video id: {}".format(first_related_video.video_id))
            print("First related video name: {}".format(first_related_video.name))
            print("First related video content url: {}".format(first_related_video.content_url))
        else:
            print("Couldn't find any related video!")

    except Exception as err:
        print("Encountered exception. {}".format(err))

```
## <a name="next-steps"></a>後續步驟

[認知服務 Python SDK 範例](https://github.com/Azure-Samples/cognitive-services-python-sdk-samples)

