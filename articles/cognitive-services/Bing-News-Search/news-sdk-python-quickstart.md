---
title: 快速入門：Bing 新聞搜尋 SDK，Python
titleSuffix: Azure Cognitive Services
description: 設定 Bing 新聞搜尋 SDK 主控台應用程式。
services: cognitive-services
author: mikedodaro
manager: cgronlun
ms.service: cognitive-services
ms.component: bing-news-search
ms.topic: quickstart
ms.date: 02/14/2018
ms.author: v-gedod
ms.openlocfilehash: 8e4343b053835c0fc2219373ad60f96c7b80636a
ms.sourcegitcommit: 9eaf634d59f7369bec5a2e311806d4a149e9f425
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/05/2018
ms.locfileid: "48803336"
---
# <a name="quickstart-bing-news-search-sdk-with-python"></a>快速入門：搭配使用 Bing 新聞搜尋 SDK 與 Python

新聞搜尋 SDK 包含用於 Web 查詢以及剖析結果的 REST API 功能。 

[Python Bing 新聞搜尋 SDK 範例的原始程式碼](https://github.com/Azure-Samples/cognitive-services-python-sdk-samples/blob/master/samples/search/news_search_samples.py)可從 Git Hub 取得。

## <a name="application-dependencies"></a>應用程式相依性
若還沒有 Python，請安裝。 SDK 與 Python 2.7、3.3、3.4、3.5 和 3.6 相容。

Python 開發的一般建議是使用[虛擬環境](https://docs.python.org/3/tutorial/venv.html)。 使用 [venv 模組](https://pypi.python.org/pypi/virtualenv)來安裝和初始化虛擬環境。 您必須安裝適用於 Python 2.7 的 virtualenv。
```
python -m venv mytestenv
```
安裝 Bing 新聞搜尋 SDK 相依性：
```
cd mytestenv
python -m pip install azure-cognitiveservices-search-newssearch
```
## <a name="news-search-client"></a>新聞搜尋用戶端
在 [搜尋] 下取得[認知服務存取金鑰](https://azure.microsoft.com/try/cognitive-services/)。 新增匯入：
```
from azure.cognitiveservices.search.newssearch import NewsSearchAPI
from msrest.authentication import CognitiveServicesCredentials

subscription_key = "YOUR-SUBSCRIPTION-KEY"
```
建立 `CognitiveServicesCredentials` 執行個體。 具現化用戶端：
```
client = NewsSearchAPI(CognitiveServicesCredentials(subscription_key))
```
搜尋結果，並列印第一個網頁結果：
```
news_result = client.news.search(query="Quantum Computing", market="en-us", count=10)
print("Search news for query \"Quantum Computing\" with market and count")

if news_result.value:
    first_news_result = news_result.value[0]
    print("Total estimated matches value: {}".format(news_result.total_estimated_matches))
    print("News result count: {}".format(len(news_result.value)))
    print("First news name: {}".format(first_news_result.name))
    print("First news url: {}".format(first_news_result.url))
    print("First news description: {}".format(first_news_result.description))
    print("First published time: {}".format(first_news_result.date_published))
    print("First news provider: {}".format(first_news_result.provider[0].name))
else:
    print("Didn't see any news result data..")

```
使用 `freshness` 和 `sortBy` 參數，利用篩選搜尋有關 "Artificial Intelligence" 的最新新聞。 驗證結果數目，並印出第一個新聞項目結果的 `totalEstimatedMatches`、`name`、`url`、`description`、`published time` 和 `name of provider`。
```
def news_search_with_filtering(subscription_key):

    client = NewsSearchAPI(CognitiveServicesCredentials(subscription_key))

    try:
        news_result = client.news.search(
            query="Artificial Intelligence",
            market="en-us",
            freshness="Week",
            sort_by="Date"
        )
        print("\r\nSearch most recent news for query \"Artificial Intelligence\" with freshness and sortBy")

        if news_result.value:
            first_news_result = news_result.value[0]
            print("News result count: {}".format(len(news_result.value)))
            print("First news name: {}".format(first_news_result.name))
            print("First news url: {}".format(first_news_result.url))
            print("First news description: {}".format(first_news_result.description))
            print("First published time: {}".format(first_news_result.date_published))
            print("First news provider: {}".format(first_news_result.provider[0].name))
        else:
            print("Didn't see any news result data..")

    except Exception as err:
        print("Encountered exception. {}".format(err))

```
使用安全搜尋，搜尋電影和電視娛樂的分類新聞。 驗證結果數目，並印出第一個新聞項目結果的 `category`、`name`、`url`、`description`、`published time` 和 `name of provider`。
```
def news_category(subscription_key):

    client = NewsSearchAPI(CognitiveServicesCredentials(subscription_key))

    try:
        news_result = client.news.category(
            category="Entertainment_MovieAndTV",
            market="en-us",
            safe_search="strict"
        )
        print("\r\nSearch category news for movie and TV entertainment with safe search")

        if news_result.value:
            first_news_result = news_result.value[0]
            print("News result count: {}".format(len(news_result.value)))
            print("First news category: {}".format(first_news_result.category))
            print("First news name: {}".format(first_news_result.name))
            print("First news url: {}".format(first_news_result.url))
            print("First news description: {}".format(first_news_result.description))
            print("First published time: {}".format(first_news_result.date_published))
            print("First news provider: {}".format(first_news_result.provider[0].name))
        else:
            print("Didn't see any news result data..")

    except Exception as err:
        print("Encountered exception. {}".format(err))


```
在 Bing 中搜尋新聞趨勢主題。  驗證結果數目，並印出第一個新聞結果的 `name`、`text of query`、`webSearchUrl`、`newsSearchUrl` 和 `image Url`。
```
def news_trending(subscription_key):

    client = NewsSearchAPI(CognitiveServicesCredentials(subscription_key))

    try:
        trending_topics = client.news.trending(market="en-us")
        print("\r\nSearch news trending topics in Bing")

        if trending_topics.value:
            first_topic = trending_topics.value[0]
            print("News result count: {}".format(len(trending_topics.value)))
            print("First topic name: {}".format(first_topic.name))
            print("First topic query: {}".format(first_topic.query.text))
            print("First topic image url: {}".format(first_topic.image.url))
            print("First topic webSearchUrl: {}".format(first_topic.web_search_url))
            print("First topic newsSearchUrl: {}".format(first_topic.news_search_url))
        else:
            print("Didn't see any topics result data..")

    except Exception as err:
        print("Encountered exception. {}".format(err))

```

## <a name="next-steps"></a>後續步驟

[認知服務 Python SDK 範例](https://github.com/Azure-Samples/cognitive-services-python-sdk-samples)


