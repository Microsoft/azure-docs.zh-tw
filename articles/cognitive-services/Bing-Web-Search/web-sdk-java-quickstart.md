---
title: 快速入門：使用適用於 Java 的 Bing Web 搜尋 SDK
titleSuffix: Azure Cognitive Services
description: Bing Web 搜尋 SDK 可讓您輕鬆地將 Bing Web 搜尋整合到 Java 應用程式。 在本快速入門中，您將了解如何傳送要求、接收 JSON 回應，以及篩選和剖析結果。
services: cognitive-services
author: erhopf
manager: cgronlun
ms.service: cognitive-services
ms.component: bing-web-search
ms.topic: quickstart
ms.date: 08/22/2018
ms.author: erhopf
ms.openlocfilehash: 8db2f6a67f64216276ccdb0b16bbeb48565f6bc6
ms.sourcegitcommit: f10653b10c2ad745f446b54a31664b7d9f9253fe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2018
ms.locfileid: "46127596"
---
# <a name="quickstart-use-the-bing-web-search-sdk-for-java"></a>快速入門：使用適用於 Java 的 Bing Web 搜尋 SDK

Bing Web 搜尋 SDK 可讓您輕鬆地將 Bing Web 搜尋整合到 Java 應用程式。 在本快速入門中，您將了解如何傳送要求、接收 JSON 回應，以及篩選和剖析結果。

要立即查看程式碼嗎？ GitHub 上提供[適用於 Java 的 Bing Web 搜尋 SDK 範例](https://github.com/Azure-Samples/cognitive-services-java-sdk-samples/) (英文)。

[!INCLUDE [bing-web-search-quickstart-signup](../../../includes/bing-web-search-quickstart-signup.md)]

## <a name="prerequisites"></a>必要條件

以下是執行本快速入門之前的幾個必備項目：

* [JDK 7 或 8](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html) (英文)
* [Apache Maven](https://maven.apache.org/download.cgi) 或您最愛的組建自動化工具
* 訂用帳戶金鑰

## <a name="create-a-project-and-configure-your-pom-file"></a>建立專案並設定您的 POM 檔案

使用 Maven 或您最愛的組建自動化工具，建立新的 Java 專案。 假設您使用 Maven，請將下列幾行新增至您的 POM 檔案。 以您的應用程式取代 `mainClass` 的所有執行個體。

```xml
<build>
    <plugins>
      <plugin>
        <groupId>org.codehaus.mojo</groupId>
        <artifactId>exec-maven-plugin</artifactId>
        <version>1.4.0</version>
        <configuration>
          <!--Your comment
            Replace the mainClass with the path to your java application.
            It should begin with com and doesn't require the .java extension.
            For example: com.bingwebsearch.app.BingWebSearchSample. This maps to
            The following directory structure:
            src/main/java/com/bingwebsearch/app/BingWebSearchSample.java.
          -->
          <mainClass>com.path.to.your.app.APP_NAME</mainClass>
        </configuration>
      </plugin>
      <plugin>
        <artifactId>maven-compiler-plugin</artifactId>
        <version>3.0</version>
        <configuration>
          <source>1.7</source>
          <target>1.7</target>
        </configuration>
      </plugin>
      <plugin>
        <artifactId>maven-assembly-plugin</artifactId>
        <executions>
          <execution>
            <phase>package</phase>
            <goals>
              <goal>attached</goal>
            </goals>
            <configuration>
              <descriptorRefs>
                <descriptorRef>jar-with-dependencies</descriptorRef>
              </descriptorRefs>
              <archive>
                <manifest>
                  <!--Your comment
                    Replace the mainClass with the path to your java application.
                    For example: com.bingwebsearch.app.BingWebSearchSample.java.
                    This maps to the following directory structure:
                    src/main/java/com/bingwebsearch/app/BingWebSearchSample.java.
                  -->
                  <mainClass>com.path.to.your.app.APP_NAME.java</mainClass>
                </manifest>
              </archive>
            </configuration>
          </execution>
        </executions>
      </plugin>
    </plugins>
  </build>
  <dependencies>
    <dependency>
      <groupId>com.microsoft.azure</groupId>
      <artifactId>azure</artifactId>
      <version>1.9.0</version>
    </dependency>
    <dependency>
      <groupId>commons-net</groupId>
      <artifactId>commons-net</artifactId>
      <version>3.3</version>
    </dependency>
    <dependency>
      <groupId>com.microsoft.azure.cognitiveservices</groupId>
      <artifactId>azure-cognitiveservices-websearch</artifactId>
      <version>1.0.1</version>
    </dependency>
  </dependencies>
```

## <a name="declare-dependencies"></a>宣告相依性

在您慣用的 IDE 或編輯器中開啟專案，並匯入這些相依性：

```java
import com.microsoft.azure.cognitiveservices.search.websearch.BingWebSearchAPI;
import com.microsoft.azure.cognitiveservices.search.websearch.BingWebSearchManager;
import com.microsoft.azure.cognitiveservices.search.websearch.models.ImageObject;
import com.microsoft.azure.cognitiveservices.search.websearch.models.NewsArticle;
import com.microsoft.azure.cognitiveservices.search.websearch.models.SearchResponse;
import com.microsoft.azure.cognitiveservices.search.websearch.models.VideoObject;
import com.microsoft.azure.cognitiveservices.search.websearch.models.WebPage;
```

如果您使用 Maven 建立專案，應該就已宣告套件。 否則，請現在宣告套件。 例如︰

```java
package com.bingwebsearch.app
```

## <a name="declare-the-bingwebsearchsample-class"></a>宣告 BingWebSearchSample 類別

宣告 `BingWebSearchSample` 類別。 此類別包含我們的大部分程式碼，其中包括 `main` 方法。  

```java
public class BingWebSearchSample {

// The code in the following sections goes here.

}
```

## <a name="construct-a-request"></a>建構要求

`runSample` 方法 (位於 `BingWebSearchSample` 類別中) 會建構要求。 將此程式碼複製到您的應用程式：

```java
public static boolean runSample(BingWebSearchAPI client) {
    /*
     * This function performs the search.
     *
     * @param client instance of the Bing Web Search API client
     * @return true if sample runs successfully
     */
    try {
        /*
         * Performs a search based on the .withQuery and prints the name and
         * url for the first web pages, image, news, and video result
         * included in the response.
         */
        System.out.println("Searched Web for \"Xbox\"");
        // Construct the request.
        SearchResponse webData = client.bingWebs().search()
            .withQuery("Xbox")
            .withMarket("en-us")
            .withCount(10)
            .execute();

// Code continues in the next section...
```

## <a name="handle-the-response"></a>處理回應

接下來，讓我們新增一些程式碼來剖析回應並列印結果。 當第一個網頁、影像、新聞文章和影片出現在回應物件中時，將會列印其 `name` 和 `url`。

```java
/*
* WebPages
* If the search response contains web pages, the first result's name
* and url are printed.
*/
if (webData != null && webData.webPages() != null && webData.webPages().value() != null &&
        webData.webPages().value().size() > 0) {
    // find the first web page
    WebPage firstWebPagesResult = webData.webPages().value().get(0);

    if (firstWebPagesResult != null) {
        System.out.println(String.format("Webpage Results#%d", webData.webPages().value().size()));
        System.out.println(String.format("First web page name: %s ", firstWebPagesResult.name()));
        System.out.println(String.format("First web page URL: %s ", firstWebPagesResult.url()));
    } else {
        System.out.println("Couldn't find the first web result!");
    }
} else {
    System.out.println("Didn't find any web pages...");
}
/*
 * Images
 * If the search response contains images, the first result's name
 * and url are printed.
 */
if (webData != null && webData.images() != null && webData.images().value() != null &&
        webData.images().value().size() > 0) {
    // find the first image result
    ImageObject firstImageResult = webData.images().value().get(0);

    if (firstImageResult != null) {
        System.out.println(String.format("Image Results#%d", webData.images().value().size()));
        System.out.println(String.format("First Image result name: %s ", firstImageResult.name()));
        System.out.println(String.format("First Image result URL: %s ", firstImageResult.contentUrl()));
    } else {
        System.out.println("Couldn't find the first image result!");
    }
} else {
    System.out.println("Didn't find any images...");
}
/*
 * News
 * If the search response contains news articles, the first result's name
 * and url are printed.
 */
if (webData != null && webData.news() != null && webData.news().value() != null &&
        webData.news().value().size() > 0) {
    // find the first news result
    NewsArticle firstNewsResult = webData.news().value().get(0);
    if (firstNewsResult != null) {
        System.out.println(String.format("News Results#%d", webData.news().value().size()));
        System.out.println(String.format("First news result name: %s ", firstNewsResult.name()));
        System.out.println(String.format("First news result URL: %s ", firstNewsResult.url()));
    } else {
        System.out.println("Couldn't find the first news result!");
    }
} else {
    System.out.println("Didn't find any news articles...");
}

/*
 * Videos
 * If the search response contains videos, the first result's name
 * and url are printed.
 */
if (webData != null && webData.videos() != null && webData.videos().value() != null &&
        webData.videos().value().size() > 0) {
    // find the first video result
    VideoObject firstVideoResult = webData.videos().value().get(0);

    if (firstVideoResult != null) {
        System.out.println(String.format("Video Results#%s", webData.videos().value().size()));
        System.out.println(String.format("First Video result name: %s ", firstVideoResult.name()));
        System.out.println(String.format("First Video result URL: %s ", firstVideoResult.contentUrl()));
    } else {
        System.out.println("Couldn't find the first video result!");
    }
} else {
    System.out.println("Didn't find any videos...");
}
```

## <a name="declare-the-main-method"></a>宣告 main 方法

在此應用程式中，main 方法包含可具現化用戶端、驗證 `subscriptionKey` 及呼叫 `runSample` 的程式碼。 請務必輸入 Azure 帳戶的有效訂用帳戶金鑰再繼續。

```java
public static void main(String[] args) {
    try {
        // Enter a valid subscription key for your account.
        final String subscriptionKey = "YOUR_SUBSCRIPTION_KEY";
        // Instantiate the client.
        BingWebSearchAPI client = BingWebSearchManager.authenticate(subscriptionKey);
        // Make a call to the Bing Web Search API.
        runSample(client);
    } catch (Exception e) {
        System.out.println(e.getMessage());
        e.printStackTrace();
    }
}
```

## <a name="run-the-program"></a>執行程式

最後一步就是執行您的程式！

```console
mvn compile exec:java
```

## <a name="clean-up-resources"></a>清除資源

此專案使用完畢時，請務必從程式的程式碼中移除訂用帳戶金鑰。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [認知服務 Java SDK 範例](https://github.com/Azure-Samples/cognitive-services-java-sdk-samples/tree/master/Search/BingWebSearch)

## <a name="see-also"></a>另請參閱

* [Azure Java SDK 參考](https://docs.microsoft.com/java/api/overview/azure/cognitiveservices/client/websearch) (英文)
