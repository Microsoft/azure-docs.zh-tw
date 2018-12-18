---
title: 快速入門：Bing 實體搜尋 SDK，Java
titlesuffix: Azure Cognitive Services
description: 設定 Bing 實體搜尋 SDK 主控台應用程式。
services: cognitive-services
author: mikedodaro
manager: cgronlun
ms.service: cognitive-services
ms.component: bing-entity-search
ms.topic: quickstart
ms.date: 02/19/2018
ms.author: v-gedod
ms.openlocfilehash: eeb8e6e0ac132ef2d927aa2de03817389230191e
ms.sourcegitcommit: 6f59cdc679924e7bfa53c25f820d33be242cea28
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/05/2018
ms.locfileid: "48814486"
---
# <a name="quickstart-bing-entity-search-sdk-with-java"></a>快速入門：搭配使用 Bing 實體搜尋 SDK 與 Java

Bing 實體搜尋 SDK 提供用於實體查詢以及剖析結果的 REST API 功能。 

[Java Bing 實體搜尋 SDK 範例的原始程式碼](https://github.com/Azure-Samples/cognitive-services-java-sdk-samples/tree/master/Search/BingEntitySearch)可從 Git Hub 取得。 

## <a name="application-dependencies"></a>應用程式相依性
在 [搜尋] 下取得[認知服務存取金鑰](https://azure.microsoft.com/try/cognitive-services/)。 使用 Maven、Gradle 或另一個相依性管理系統來安裝 Bing 實體搜尋 SDK 相依性。 Maven POM 檔案需要宣告：
```
  <dependencies>
    <dependency>
        <groupId>com.microsoft.azure.cognitiveservices</groupId>
        <artifactId>azure-cognitiveservices-entitysearch</artifactId>
        <version>0.0.1-beta-SNAPSHOT</version>
    </dependency>
  </dependencies>
```
## <a name="entity-search-client"></a>實體搜尋用戶端
將匯入新增至類別實作。
```
import com.microsoft.azure.cognitiveservices.entitysearch.*;
import com.microsoft.azure.cognitiveservices.entitysearch.implementation.EntitySearchAPIImpl;
import com.microsoft.azure.cognitiveservices.entitysearch.implementation.SearchResponseInner;
import com.microsoft.rest.credentials.ServiceClientCredentials;
import okhttp3.Interceptor;
import okhttp3.OkHttpClient;
import okhttp3.Request;
import okhttp3.Response;

import java.io.IOException;
import java.util.ArrayList;
import java.util.List;
```
實作 **EntitySearchAPIImpl** 用戶端，而此用戶端需要 **ServiceClientCredentials** 類別執行個體。
```
public static EntitySearchAPIImpl getClient(final String subscriptionKey) {
    return new EntitySearchAPIImpl("https://api.cognitive.microsoft.com/bing/v7.0/",
            new ServiceClientCredentials() {
                @Override
                public void applyCredentialsFilter(OkHttpClient.Builder builder) {
                    builder.addNetworkInterceptor(
                            new Interceptor() {
                                @Override
                                public Response intercept(Interceptor.Chain chain) throws IOException {
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
搜尋單一實體 "Satya Nadella"，並列印簡短描述。
```
public static void dominantEntityLookup(final String subscriptionKey)
{
    try
    {
        EntitySearchAPIImpl client = getClient(subscriptionKey);
        SearchResponseInner entityData = client.entities().search(
                "satya nadella", null, null, null, null, null, null, "en-us", null, null, SafeSearch.STRICT, null);

        if (entityData.entities().value().size() > 0)
        {
            // Find the entity that represents the dominant entity
            List<Thing> entrys = entityData.entities().value();
            Thing dominateEntry = null;
            for(Thing thing : entrys) {
                if(thing.entityPresentationInfo().entityScenario() == EntityScenario.DOMINANT_ENTITY) {
                    System.out.println("\r\nSearched for \"Satya Nadella\" and found a dominant entity with this description:");
                    System.out.println(thing.description());
                    break;
                }
            }

            if(dominateEntry == null)
            {
                 System.out.println("Couldn't find main entity Satya Nadella!");
            }
        }
        else
        {
             System.out.println("Didn't see any data..");
        }
    }
    catch (ErrorResponseException ex)
    {
         System.out.println("Encountered exception. " + ex.getLocalizedMessage());
    }
}

```
搜尋 "William Gates"，並處理模稜兩可查詢的去除混淆結果。
```
public static void handlingDisambiguation(String subscriptionKey)
{
    try
    {
        EntitySearchAPIImpl client = getClient(subscriptionKey);
        SearchResponseInner entityData = client.entities().search(
                "william gates", null, null, null, null, null, null, "en-us", null, null, SafeSearch.STRICT, null);
        if (entityData.entities().value().size() > 0)
        {
            // Find the entity that represents the dominant entity
            List<Thing> entrys = entityData.entities().value();
            Thing dominateEntry = null;
            List<Thing> disambigEntities = new ArrayList<Thing>();
            for(Thing thing : entrys) {
                if(thing.entityPresentationInfo().entityScenario() == EntityScenario.DOMINANT_ENTITY) {
                    System.out.println("\r\nSearched for \"William Gates\" and found a dominant entity with this description:");
                    System.out.println(String.format("Searched for \"William Gates\" and found a dominant entity with type hint \"%s\" with this description:",
                            thing.entityPresentationInfo().entityTypeDisplayHint()));
                    System.out.println(thing.description());
                }

                if(thing.entityPresentationInfo().entityScenario() == EntityScenario.DISAMBIGUATION_ITEM) {
                    disambigEntities.add(thing);
                    System.out.println("Searched for \"William Gates\" and found a dominant entity with this description:");
                    System.out.println(thing.description());
                }

            }

            if (dominateEntry == null)
            {
                 System.out.println("Couldn't find a reliable dominant entity for William Gates!");
            }

            if (disambigEntities.size() > 0)
            {
                 System.out.println();
                 System.out.println("This query is pretty ambiguous and can be referring to multiple things. Did you mean one of these: ");

                StringBuilder sb = new StringBuilder();

                for (Thing disambig : disambigEntities)
                {
                    sb.append(String.format(", or %s the %s", disambig.name(), disambig.entityPresentationInfo().entityTypeDisplayHint()));
                }

                 System.out.println(sb.toString().substring(5) + "?");
            }
            else
            {
                 System.out.println("We didn't find any disambiguation items for William Gates, so we must be certain what you're talking about!");
            }
        }
        else
        {
             System.out.println("Didn't see any data..");
        }
    }
    catch (ErrorResponseException ex)
    {
         System.out.println("Encountered exception. " + ex.getLocalizedMessage());
    }
}

```
使用 "Microsoft Store" 查詢來搜尋單一商店，並列印結果中的電話號碼。
```
public static void storeLookup(String subscriptionKey)
{
    try
    {
        EntitySearchAPIImpl client = getClient(subscriptionKey);
        SearchResponseInner entityData = client.entities().search(
                "Microsoft Store", null, null, null, null, null, null, "en-us", null, null, SafeSearch.STRICT, null);

        if (entityData.places() != null && entityData.places().value().size() > 0)
        {
            // Some local entities are places, others are not. Depending on the data that you want, try to cast to the appropriate schema.
            // In this case, the returned item is technically a store, but the Place schema has the data that we want (telephone).
            Place store = (Place)entityData.places().value().get(0);

            if (store != null)
            {
                 System.out.println("\r\nSearched for \"Microsoft Store\" and found a store with this phone number:");
                 System.out.println(store.telephone());
            }
            else
            {
                 System.out.println("Couldn't find a place!");
            }
        }
        else
        {
             System.out.println("Didn't see any data..");
        }
    }
    catch (ErrorResponseException ex)
    {
         System.out.println("Encountered exception. " + ex.getLocalizedMessage());
    }
}

```
使用 "Seattle restaurants" 查詢來搜尋餐廳清單。 列印結果的名稱和電話號碼。
```
public static void multipleRestaurantLookup(String subscriptionKey)
{
    try
    {
        EntitySearchAPIImpl client = getClient(subscriptionKey);
        SearchResponseInner restaurants = client.entities().search(
                "Seattle restaurants", null, null, null, null, null, null, "en-us", null, null, SafeSearch.STRICT, null);
        
        System.out.println("\r\nSearched for \"multiple restaurants\""); 
        if (restaurants.places() != null && restaurants.places().value().size() > 0)
        {
            List<Thing> listItems = new ArrayList<Thing>();
            for(Thing place : restaurants.places().value()) {
                if (place.entityPresentationInfo().entityScenario() == EntityScenario.LIST_ITEM) {
                    listItems.add(place);
                }
            }

            if (listItems.size() > 0)
            {
                StringBuilder sb = new StringBuilder();

                for (Thing item : listItems)
                {
                    Place place = (Place)item;
                    if (place == null)
                    {
                         System.out.println(String.format("Unexpectedly found something that isn't a place named \"%s\"", place.name()));
                        continue;
                    }

                    sb.append(String.format(",%s (%s) ", place.name(), place.telephone()));
                }

                 System.out.println("Ok, we found these places: ");
                 System.out.println(sb.toString().substring(1));
            }
            else
            {
                 System.out.println("Couldn't find any relevant results for \"seattle restaurants\"");
            }
        }
        else
        {
             System.out.println("Didn't see any data..");
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
package entitySDK;
import com.microsoft.azure.cognitiveservices.entitysearch.*;

public class EntitySearchSDK {

    public static void main(String[] args) {
        
        dominantEntityLookup("YOUR-SUBSCRIPTION-KEY");
        handlingDisambiguation("YOUR-SUBSCRIPTION-KEY");
        restaurantLookup("YOUR-SUBSCRIPTION-KEY");
        multipleRestaurantLookup("YOUR-SUBSCRIPTION-KEY");

    }
    // Include the methods described in this article.
}
```
## <a name="next-steps"></a>後續步驟

[認知服務 Java SDK 範例](https://github.com/Azure-Samples/cognitive-services-java-sdk-samples)

