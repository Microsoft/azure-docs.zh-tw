---
title: 快速入門：使用適用於 Python 的 Bing Web 搜尋 SDK
titleSuffix: Azure Cognitive Services
description: Bing Web 搜尋 SDK 可讓您輕鬆地將 Bing Web 搜尋整合到 Python 應用程式。 在本快速入門中，您將了解如何傳送要求、接收 JSON 回應，以及篩選和剖析結果。
services: cognitive-services
author: erhopf
manager: cgronlun
ms.service: cognitive-services
ms.component: bing-web-search
ms.topic: quickstart
ms.date: 08/16/2018
ms.author: erhopf
ms.openlocfilehash: c83848bc3c8f14aa49c1f87f1cd44224bb3e8081
ms.sourcegitcommit: f10653b10c2ad745f446b54a31664b7d9f9253fe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2018
ms.locfileid: "46127630"
---
# <a name="quickstart-use-the-bing-web-search-sdk-for-python"></a>快速入門：使用適用於 Python 的 Bing Web 搜尋 SDK

Bing Web 搜尋 SDK 可讓您輕鬆地將 Bing Web 搜尋整合到 Python 應用程式。 在本快速入門中，您將了解如何傳送要求、接收 JSON 回應，以及篩選和剖析結果。

要立即查看程式碼嗎？ GitHub 上提供[適用於 Python 的 Bing Web 搜尋 SDK 範例](https://github.com/Azure-Samples/cognitive-services-python-sdk-samples) (英文)。

[!INCLUDE [bing-web-search-quickstart-signup](../../../includes/bing-web-search-quickstart-signup.md)]

## <a name="prerequisites"></a>必要條件

Bing Web 搜尋 SDK 與 Python 2.7、3.3、3.4、3.5 和 3.6 相容。 建議在虛擬環境下使用此快速入門。

* Python 2.7、3.3、3.4、3.5 或 3.6
* 適用於 Python 2.7 的 [virtualenv](https://docs.python.org/3/tutorial/venv.html) (英文)
* 適用於 Python 3.x 的 [venv](https://pypi.python.org/pypi/virtualenv) (英文)

## <a name="create-and-configure-your-virtual-environment"></a>建立及設定您的虛擬環境

Python 2.x 和 Python 3.x 在安裝和設定虛擬環境方面各有不同的指示。 請遵循下列步驟來建立和初始化您的虛擬環境。

### <a name="python-2x"></a>Python 2.x

使用適用於 Python 2.7 的 `virtualenv` 來建立虛擬環境：

```console
virtualenv mytestenv
```

啟動您的環境：

```console
cd mytestenv
source bin/activate
```

安裝 Bing Web 搜尋 SDK 相依性：

```console
python -m pip install azure-cognitiveservices-search-websearch
```

### <a name="python-3x"></a>Python 3.x

使用適用於 Python 3.x 的 `venv` 來建立虛擬環境：

```console
python -m venv mytestenv
```

安裝 Bing Web 搜尋 SDK 相依性：

```console
cd mytestenv
python -m pip install azure-cognitiveservices-search-websearch
```

## <a name="create-a-client-and-print-your-first-results"></a>建立用戶端並列印您的第一個結果

既然您已設定虛擬環境並安裝相依性，讓我們來建立用戶端。 用戶端會處理對 Bing Web 搜尋 API 提出的要求及其回應。

如果回應包含網頁、影像、新聞或影片，則會列印各項的第一個結果。

1. 在您慣用的 IDE 或編輯器建立新的 Python 專案。
2. 將此範例程式碼複製到您的專案：  
    ```python
    # Import required modules.
    from azure.cognitiveservices.search.websearch import WebSearchAPI
    from azure.cognitiveservices.search.websearch.models import SafeSearch
    from msrest.authentication import CognitiveServicesCredentials

    # Replace with your subscription key.
    subscription_key = "YOUR_SUBSCRIPTION_KEY"

    # Instantiate the client.
    client = WebSearchAPI(CognitiveServicesCredentials(subscription_key))

    # Make a request. Replace Yosemite if you'd like.
    web_data = client.web.search(query="Yosemite")
    print("\r\nSearched for Query# \" Yosemite \"")

    '''
    Web pages
    If the search response contains web pages, the first result's name and url
    are printed.
    '''
    if hasattr(web_data.web_pages, 'value'):

        print("\r\nWebpage Results#{}".format(len(web_data.web_pages.value)))

        first_web_page = web_data.web_pages.value[0]
        print("First web page name: {} ".format(first_web_page.name))
        print("First web page URL: {} ".format(first_web_page.url))

    else:
        print("Didn't find any web pages...")

    '''
    Images
    If the search response contains images, the first result's name and url
    are printed.
    '''
    if hasattr(web_data.images, 'value'):

        print("\r\nImage Results#{}".format(len(web_data.images.value)))

        first_image = web_data.images.value[0]
        print("First Image name: {} ".format(first_image.name))
        print("First Image URL: {} ".format(first_image.url))

    else:
        print("Didn't find any images...")

    '''
    News
    If the search response contains news, the first result's name and url
    are printed.
    '''
    if hasattr(web_data.news, 'value'):

        print("\r\nNews Results#{}".format(len(web_data.news.value)))

        first_news = web_data.news.value[0]
        print("First News name: {} ".format(first_news.name))
        print("First News URL: {} ".format(first_news.url))

    else:
        print("Didn't find any news...")

    '''
    If the search response contains videos, the first result's name and url
    are printed.
    '''
    if hasattr(web_data.videos, 'value'):

        print("\r\nVideos Results#{}".format(len(web_data.videos.value)))

        first_video = web_data.videos.value[0]
        print("First Videos name: {} ".format(first_video.name))
        print("First Videos URL: {} ".format(first_video.url))

    else:
        print("Didn't find any videos...")
    ```
3. 將 `subscription_key` 換成有效的訂用帳戶金鑰。
4. 執行程式。 例如： `python your_program.py` 。

## <a name="define-functions-and-filter-results"></a>定義函式和篩選結果

既然您已完成第一次呼叫 Bing Web 搜尋 API，讓我們看看一些函式，以突顯 SDK 在改善查詢和篩選結果方面的實用性。 每個函式都可以新增至上一節所建立的 Python 程式。

### <a name="limit-the-number-of-results-returned-by-bing"></a>限制 Bing 所傳回的結果數目

這個範例會使用 `count` 和 `offset` 參數，來限制以 SDK 的 [`search` 方法](https://docs.microsoft.com/python/api/azure-cognitiveservices-search-websearch/azure.cognitiveservices.search.websearch.operations.weboperations?view=azure-python#search) (英文) 所傳回的結果數目。 第一個結果的 `name` 和 `URL` 會列印出來。

1. 將此程式碼新增至 Python 專案：
    ```python
    # Declare the function.
    def web_results_with_count_and_offset(subscription_key):
        client = WebSearchAPI(CognitiveServicesCredentials(subscription_key))

        try:
            '''
            Set the query, offset, and count using the SDK's search method. See:
            https://docs.microsoft.com/python/api/azure-cognitiveservices-search-websearch/azure.cognitiveservices.search.websearch.operations.weboperations?view=azure-python#search.
            '''
            web_data = client.web.search(query="Best restaurants in Seattle", offset=10, count=20)
            print("\r\nSearching for \"Best restaurants in Seattle\"")

            if web_data.web_pages.value:
                '''
                If web pages are available, print the # of responses, and the first and second
                web pages returned.
                '''
                print("Webpage Results#{}".format(len(web_data.web_pages.value)))

                first_web_page = web_data.web_pages.value[0]
                print("First web page name: {} ".format(first_web_page.name))
                print("First web page URL: {} ".format(first_web_page.url))

            else:
                print("Didn't find any web pages...")

        except Exception as err:
            print("Encountered exception. {}".format(err))
    ```
2. 執行程式。

### <a name="filter-for-news-and-freshness"></a>篩選新聞及時效性

這個範例會使用 `response_filter` 和 `freshness` 參數，來篩選以 SDK 的 [`search` 方法](https://docs.microsoft.com//api/azure-cognitiveservices-search-websearch/azure.cognitiveservices.search.websearch.operations.weboperations?view=azure-python#search) (英文) 所傳回的搜尋結果。 傳回的搜尋結果僅限於 Bing 在過去 24 小時內探索過的新聞文章和網頁。 第一個結果的 `name` 和 `URL` 會列印出來。

1. 將此程式碼新增至 Python 專案：
    ```python
    # Declare the function.
    def web_search_with_response_filter(subscription_key):
        client = WebSearchAPI(CognitiveServicesCredentials(subscription_key))
        try:
            '''
            Set the query, response_filter, and freshness using the SDK's search method. See:
            https://docs.microsoft.com/python/api/azure-cognitiveservices-search-websearch/azure.cognitiveservices.search.websearch.operations.weboperations?view=azure-python#search.
            '''
            web_data = client.web.search(query="xbox",
                response_filter=["News"],
                freshness="Day")
            print("\r\nSearching for \"xbox\" with the response filter set to \"News\" and freshness filter set to \"Day\".")

            '''
            If news articles are available, print the # of responses, and the first and second
            articles returned.
            '''
            if web_data.news.value:

                print("# of news results: {}".format(len(web_data.news.value)))

                first_web_page = web_data.news.value[0]
                print("First article name: {} ".format(first_web_page.name))
                print("First article URL: {} ".format(first_web_page.url))

                print("")

                second_web_page = web_data.news.value[1]
                print("\nSecond article name: {} ".format(second_web_page.name))
                print("Second article URL: {} ".format(second_web_page.url))

            else:
                print("Didn't find any news articles...")

        except Exception as err:
            print("Encountered exception. {}".format(err))

    # Call the function.
    web_search_with_response_filter(subscription_key)
    ```
2. 執行程式。

### <a name="use-safe-search-answer-count-and-the-promote-filter"></a>使用安全搜尋、回應計數和宣傳篩選條件

這個範例會使用 `answer_count`、`promote` 和 `safe_search`參數，來篩選以 SDK 的 [`search` 方法](https://docs.microsoft.com/python/api/azure-cognitiveservices-search-websearch/azure.cognitiveservices.search.websearch.operations.weboperations?view=azure-python#search) (英文) 所傳回的搜尋結果。 第一個結果的 `name` 和 `URL` 會出現。

1. 將此程式碼新增至 Python 專案：
    ```python
    # Declare the function.
    def web_search_with_answer_count_promote_and_safe_search(subscription_key):

        client = WebSearchAPI(CognitiveServicesCredentials(subscription_key))

        try:
            '''
            Set the query, answer_count, promote, and safe_search parameters using the SDK's search method. See:
            https://docs.microsoft.com/python/api/azure-cognitiveservices-search-websearch/azure.cognitiveservices.search.websearch.operations.weboperations?view=azure-python#search.
            '''
            web_data = client.web.search(
                query="Niagara Falls",
                answer_count=2,
                promote=["videos"],
                safe_search=SafeSearch.strict  # or directly "Strict"
            )
            print("\r\nSearching for \"Niagara Falls\"")

            '''
            If results are available, print the # of responses, and the first result returned.
            '''
            if web_data.web_pages.value:

                print("Webpage Results#{}".format(len(web_data.web_pages.value)))

                first_web_page = web_data.web_pages.value[0]
                print("First web page name: {} ".format(first_web_page.name))
                print("First web page URL: {} ".format(first_web_page.url))

            else:
                print("Didn't see any Web data..")

        except Exception as err:
            print("Encountered exception. {}".format(err))
    ```
2. 執行程式。

## <a name="clean-up-resources"></a>清除資源

此專案使用完畢時，請務必從程式的程式碼中移除訂用帳戶金鑰，並停用虛擬環境。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [認知服務 Python SDK 範例](https://github.com/Azure-Samples/cognitive-services-python-sdk-samples)

## <a name="see-also"></a>另請參閱

* [Azure Python SDK 參考](https://docs.microsoft.com/python/api/overview/azure/cognitiveservices/websearch)
