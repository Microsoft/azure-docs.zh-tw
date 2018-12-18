---
title: 快速入門：Java 更新知識庫 - QnA Maker
titleSuffix: Azure Cognitive Services
description: 如何使用 Java 來更新 QnA Maker 的知識庫。
services: cognitive-services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: qna-maker
ms.topic: quickstart
ms.date: 09/12/2018
ms.author: diberry
ms.openlocfilehash: fc2792086d93e584535389c8c0f162fc1e592cbd
ms.sourcegitcommit: f20e43e436bfeafd333da75754cd32d405903b07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49388240"
---
# <a name="update-a-knowledge-base-in-java"></a>使用 Java 來更新知識庫

以下程式碼使用 [Update](https://westus.dev.cognitive.microsoft.com/docs/services/5a93fcf85b4ccd136866eb37/operations/5ac266295b4ccd1554da7600) 方法來更新現有的知識庫。

[!INCLUDE [Code is available in Azure-Samples Github repo](../../../../includes/cognitive-services-qnamaker-java-repo-note.md)]

如果您還沒有知識庫，可以建立要用於本快速入門的範例知識庫：[建立新的知識庫](create-new-kb-java.md)。

1. 在您偏好的 IDE 中建立新的 JAVA 專案。
1. 新增下方提供的程式碼。
1. 將 `key` 值換成您的有效訂用帳戶金鑰。
1. 將 `kb` 值換成您的有效知識庫識別碼。 藉由移至您的其中一個 [QnA Maker 知識庫](https://www.qnamaker.ai/Home/MyServices) (英文)，來尋找此值。 選取您想要更新的知識庫。 一旦移至該頁面，請在 URL 中尋找 'kdid='，如下所示。 針對您的程式碼範例使用其值。

    ![QnA Maker 知識庫識別碼](../media/qnamaker-quickstart-kb/qna-maker-id.png)

1. 您需要 [apache httpclient](https://hc.apache.org/downloads.cgi) (英文) 程式庫。 例如，下載二進位 HttpClient 4.x.x.tar.gz，以尋找需要匯入至專案的所有 jar 檔案。
1. 也需要 [GSON](https://github.com/google/gson) (英文) 程式庫。 無論是手動[建立](https://stackoverflow.com/questions/5258159/how-to-make-an-executable-jar-file) (英文) 並匯入 .jar 檔案，或者是將相依性新增至您慣用的專案管理工具 (例如 Maven) 皆可。
1. 執行程式。

```java
import java.io.*;
import java.lang.reflect.Type;
import java.net.*;
import java.util.*;
import javax.net.ssl.HttpsURLConnection;

/**
 * Gson: https://github.com/google/gson
 * Maven info:
 *    <dependency>
 *      <groupId>com.google.code.gson</groupId>
 *      <artifactId>gson</artifactId>
 *      <version>2.8.1</version>
 *    </dependency>
 */
import com.google.gson.Gson;
import com.google.gson.GsonBuilder;
import com.google.gson.JsonElement;
import com.google.gson.JsonObject;
import com.google.gson.JsonParser;
import com.google.gson.reflect.TypeToken;

/**
 * Java does not natively support HTTP PATCH requests; Apache HttpClient is required.
 * HttpClient: https://hc.apache.org/downloads.cgi
 * Maven info:
 *    <dependency>
 *      <groupId>org.apache.httpcomponents</groupId>
 *      <artifactId>httpclient</artifactId>
 *      <version>4.5.5</version>
 *    </dependency>
 */
import org.apache.commons.logging.LogFactory;
import org.apache.http.client.methods.HttpPatch;
import org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.entity.ByteArrayEntity;
import org.apache.http.Header;
import org.apache.http.HttpEntity;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;

/**
 * NOTE: To compile and run this code:
 * 1. Save this file as UpdateKB.java.
 * 2. Run:
 * javac UpdateKB.java -cp .;gson-2.8.1.jar;httpclient-4.5.5.jar;httpcore-4.4.9.jar;
 * commons-logging-1.2.jar -encoding UTF-8
 * 3. Run:
 * java -cp .;gson-2.8.1.jar;httpclient-4.5.5.jar;httpcore-4.4.9.jar;commons-logging
 * -1.2.jar UpdateKB
 */

public class UpdateKB {

    // **********************************************
    // *** Update or verify the following values. ***
    // **********************************************

    // Replace this with a valid subscription key.
    static String subscriptionKey = "ADD KEY HERE";

    // Replace this with a valid knowledge base ID.
    static String kb = "ADD ID HERE";

    // Represents the various elements used to create HTTP request URIs
    // for QnA Maker operations.
    static String host = "https://westus.api.cognitive.microsoft.com";
    static String service = "/qnamaker/v4.0";
    static String method = "/knowledgebases/";

    // We'll serialize these classes into JSON for our request to the server.
    // For the JSON request schema, see: 
    // https://westus.dev.cognitive.microsoft.com/docs/services/5a93fcf85b4ccd136866eb37/
    // operations/5ac266295b4ccd1554da7600
    public static class Request {
        Add add;
        Delete delete;
        Update update;
    }

    public static class Add {
        Question[] qnaList;
        String[] urls;
        File[] files;
    }

    public static class Delete {
        Integer[] ids;
    }

    public static class Update {
        String name;
        Question[] qnaList;
        String[] urls;
        File[] files;
    }

    public static class Question {
        Integer id;
        String answer;
        String source;
        String[] questions;
        Metadata[] metadata;
    }

    public static class Metadata {
        String name;
        String value;
    }

    public static class File {
        String fileName;
        String fileUri;
    }

    // This class contains the headers and body of the HTTP response.
    public static class Response {
        Map<String, List<String>> Headers;
        String Response;

        /**
         * Headers and response (body) of the HTTP request
         * @param headers A map containing the requested response
         * @param response The body of the requested response
         */
        public Response(Map<String, List<String>> headers, String response) {
            this.Headers = headers;
            this.Response = response;
        }
    }

    /**
     * Formats and indents JSON for display.
     * @param json_text The raw JSON input
     * @return String A string containing formatted and indented JSON.
     */
    public static String PrettyPrint (String json_text) {
        JsonParser parser = new JsonParser();
        JsonElement json = parser.parse(json_text);
        Gson gson = new GsonBuilder().setPrettyPrinting().create();
        return gson.toJson(json);
    }

    /**
     * Sends a GET HTTP request.
     * @param url The string URL request
     * @return A Response object with the header and body of the response
     */
    public static Response Get (URL url) throws Exception {
        HttpsURLConnection connection = (HttpsURLConnection) url.openConnection();
        connection.setRequestMethod("GET");
        connection.setRequestProperty("Ocp-Apim-Subscription-Key", subscriptionKey);
        connection.setDoOutput(true);

        StringBuilder response = new StringBuilder ();
        BufferedReader in = new BufferedReader(new InputStreamReader(connection.getInputStream(), "UTF-8"));

        String line;
        while ((line = in.readLine()) != null) {
            response.append(line);
        }
        in.close();

        return new Response (connection.getHeaderFields(), response.toString());
    }

   /**
     * Sends an HTTP PATCH request. We use Apache HttpClient, as HttpsURLConnection 
     * does not support PATCH.
     * @param url The string request
     * @param content The body of the request
     * @return Response Returns a Response object with the headers and body
     */
    public static Response Patch (URL url, String content) throws Exception {
        HttpPatch patch = new HttpPatch(url.toString());
        // HttpPatch implements HttpMessage, which includes addHeader.
        // See: <a href="https://hc.apache.org/httpcomponents-client-ga/httpclient/apidocs/org/
        // apache/http/client/methods/HttpPatch.html">HttpPatch</a>
        // See: <a href="https://hc.apache.org/httpcomponents-core-ga/httpcore/apidocs/org/apache/http/
        // HttpMessage.html">HttpMessage</a>
        patch.addHeader("Content-Type", "application/json");
        // Note: Adding the Content-Length header causes the exception:
        // "Content-Length header already present."
        patch.addHeader("Ocp-Apim-Subscription-Key", subscriptionKey);
        // HttpPatch implements HttpEntityEnclosingRequest, which includes setEntity.
        // See: <a href="https://hc.apache.org/httpcomponents-core-ga/httpcore/apidocs/org/apache/http/
        // HttpEntityEnclosingRequest.html">HttpEntityEnclosingRequest</a>
        HttpEntity entity = new ByteArrayEntity(content.getBytes("UTF-8"));
        patch.setEntity(entity);

        CloseableHttpClient httpClient = HttpClients.createDefault();
        CloseableHttpResponse response = httpClient.execute(patch);

        // CloseableHttpResponse implements HttpMessage, which includes getAllHeaders.
        // See: <a href="https://hc.apache.org/httpcomponents-client-ga/httpclient/apidocs/org/apache/http/
        // client/methods/CloseableHttpResponse.html">CloseableHttpResponse</a>
        // Header implements NameValuePair.
        // See: <a href="https://hc.apache.org/httpcomponents-core-ga/httpcore/apidocs/org/apache/http/Header.html">
        // Header</a>
        // See: <a href="https://hc.apache.org/httpcomponents-core-ga/httpcore/apidocs/org/apache/http/
        // NameValuePair.html">NameValuePair</a>
        Map<String, List<String>> headers = new HashMap<String, List<String>>();
        for (Header header : response.getAllHeaders()) {
            List<String> list = new ArrayList<String>() {
                {
                    add(header.getValue());
                }
            };
            headers.put(header.getName(), list);
        }

        // CloseableHttpResponse implements HttpResponse, which includes getEntity.
        // See: <a href="https://hc.apache.org/httpcomponents-core-ga/httpcore/apidocs/org/apache/http/
        // HttpResponse.html">HttpResponse</a>
        // HttpEntity implements getContent, which returns an InputStream.
        // See: <a href="https://hc.apache.org/httpcomponents-core-ga/httpcore/apidocs/org/apache/http/
        // HttpEntity.html">HttpEntity</a>
        StringBuilder output = new StringBuilder ();
        BufferedReader reader = new BufferedReader(new InputStreamReader(response.getEntity()
                                                                        .getContent(), "UTF-8"));
        String line;
        while ((line = reader.readLine()) != null) {
            output.append(line);
        }
        reader.close();

        return new Response (headers, output.toString());
    }

    /**
     * Sends the request to update the knowledge base.
     * @param kb The ID for the existing knowledge base
     * @param req The data source for the updated knowledge base
     * @return Reponse Returns the response from a PATCH request
     */
    public static Response UpdateKB (String kb, Request req) throws Exception {
        URL url = new URL(host + service + method + kb);
        System.out.println("Calling " + url.toString() + ".");
        String content = new Gson().toJson(req);
        return Patch(url, content);
    }

    /** 
     * Checks on the status of the request to update the knowledge base.
     * @param operation The QnA Maker operation to check
     * @return The Response object that represents the HTTP response
     */
    public static Response GetStatus (String operation) throws Exception {
        URL url = new URL(host + service + operation);
        System.out.println("Calling " + url.toString() + ".");
        return Get(url);
    }

    /**
     * Compiles the details of the request.
     * @return The Request object with request details.
     */
    public static Request GetRequest () {
        Request req = new Request ();

        Question q = new Question();
        q.id = 0;
        q.answer = "You can use our REST APIs to manage your Knowledge Base."
        + "See here for details: https://westus.dev.cognitive.microsoft.com/docs/services/"
        + "58994a073d9e04097c7ba6fe/operations/58994a073d9e041ad42d9baa";
        q.source = "Custom Editorial";
        q.questions = new String[]{"How do I programmatically update my Knowledge Base?"};

        Metadata md = new Metadata();
        md.name = "category";
        md.value = "api";
        q.metadata = new Metadata[]{md};

        req.add = new Add ();
        req.add.qnaList = new Question[]{q};
        req.add.urls = new String[]{"https://docs.microsoft.com/azure/cognitive-services"
        + "/qnamaker/faqs", "https://docs.microsoft.com/bot-framework/resources-bot-"
        + "framework-faq"};

        return req;
    }

    public static void main(String[] args) {
        try {
            // Send the request to update the knowledge base.
            Response response = UpdateKB(kb, GetRequest ());
            String operation = response.Headers.get("Location").get(0);
            System.out.println(PrettyPrint(response.Response));
            // Loop until the request is completed.
            Boolean done = false;
            while (true != done) {
                // Check on the status of the request.
                response = GetStatus (operation);
                System.out.println (PrettyPrint (response.Response));
                Type type = new TypeToken<Map<String, String>>(){}.getType();
                Map<String, String> fields = new Gson().fromJson(response.Response, type);
                String state = fields.get ("operationState");
                // If the request is still running, the server tells us how long to wait
                // before checking the status again.
                if (state.equals("Running") || state.equals("NotStarted")) {
                    String wait = response.Headers.get("Retry-After").get(0);
                    System.out.println("Waiting " + wait + " seconds...");
                    Thread.sleep(Integer.parseInt(wait) * 1000);
                }
                else {
                    done = true;
                }
            }
        }
        catch (Exception e) {
            System.out.println (e.getCause().getMessage());
        }
    }
}
```

## <a name="understand-what-qna-maker-returns"></a>了解 QnA Maker 會傳回的項目

成功的回應會以 JSON 格式傳回，如下列範例所示。 您的結果可能會稍有不同。 如果最後呼叫傳回「成功」狀態，代表您的知識庫已成功更新。 若要進行疑難排解，請參閱 QnA Maker API 的[更新知識庫](https://westus.dev.cognitive.microsoft.com/docs/services/5a93fcf85b4ccd136866eb37/operations/5ac266295b4ccd1554da7600) (英文) 回應碼。

```json
{
  "operationState": "NotStarted",
  "createdTimestamp": "2018-04-13T01:49:48Z",
  "lastActionTimestamp": "2018-04-13T01:49:48Z",
  "userId": "2280ef5917bb4ebfa1aae41fb1cebb4a",
  "operationId": "5156f64e-e31d-4638-ad7c-a2bdd7f41658"
}
...
{
  "operationState": "Succeeded",
  "createdTimestamp": "2018-04-13T01:49:48Z",
  "lastActionTimestamp": "2018-04-13T01:49:50Z",
  "resourceLocation": "/knowledgebases/140a46f3-b248-4f1b-9349-614bfd6e5563",
  "userId": "2280ef5917bb4ebfa1aae41fb1cebb4a",
  "operationId": "5156f64e-e31d-4638-ad7c-a2bdd7f41658"
}
Press any key to continue.
```

一旦更新您的知識庫，您可以在 QnA Maker 入口網站的[我的知識庫](https://www.qnamaker.ai/Home/MyServices) (英文) 頁面中檢視。 請注意，您的知識庫已根據您的自訂要求而變更。

若要修改知識庫的其他元素，請參閱 QnA Maker [JSON 結構描述](https://westus.dev.cognitive.microsoft.com/docs/services/5a93fcf85b4ccd136866eb37/operations/5ac266295b4ccd1554da7600) (英文)，並且修改 `getRequest()` 方法中的 `req` 物件。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [QnA Maker (V4) REST API 參考](https://westus.dev.cognitive.microsoft.com/docs/services/5a93fcf85b4ccd136866eb37/operations/5ac266295b4ccd1554da75ff)