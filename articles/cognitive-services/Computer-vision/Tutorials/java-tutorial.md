---
title: 教學課程：電腦視覺 API Java
titlesuffix: Azure Cognitive Services
description: 探索使用「Azure 認知服務」中「電腦視覺 API」的基本 Java Swing 應用程式。 在影像中執行 OCR、建立縮圖，以及處理視覺特徵。
services: cognitive-services
author: KellyDF
manager: cgronlun
ms.service: cognitive-services
ms.component: computer-vision
ms.topic: tutorial
ms.author: kefre
ms.date: 09/21/2017
ms.openlocfilehash: 36a8a49ee49636d186ca217ae223b1eebf9bb54b
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49340251"
---
# <a name="tutorial-computer-vision-api-java"></a>教學課程：電腦視覺 API Java

此教學課程將說明「Azure 認知服務電腦視覺 REST API」的功能。

探索使用「電腦視覺 REST API」在影像中執行光學字元辨識 (OCR)、建立智慧型裁剪縮圖以及偵測、分類、標記及描述視覺特徵 (包括臉部) 的 Java Swing 應用程式。 此範例可讓您提交影像 URL 以供分析或處理。 您可以使用此開放原始碼範例作為範本來以 Java 建置自己的應用程式，以使用「電腦視覺 REST API」。

此教學課程會說明如何使用電腦視覺：

> [!div class="checklist"]
> * 分析影像
> * 識別影像中的自然或人工地標
> * 識別影像中的名人
> * 從影像建立優質縮圖
> * 讀取影像中的印刷文字
> * 讀取影像中的手寫文字

已經撰寫 Java Swing 表單應用程式，但還沒有任何功能。 在本教學課程中，您會新增「電腦視覺 REST API」特定的程式碼以完成應用程式的功能。

## <a name="prerequisites"></a>必要條件

### <a name="platform-requirements"></a>平台需求

本教學課程是使用 IntelliJ IDE 來開發的。 具體而言，您可以在[這裡下載](https://netbeans.org/downloads/index.html) NetBeans **Java SE** 版。

### <a name="subscribe-to-computer-vision-api-and-get-a-subscription-key"></a>訂閱電腦視覺 API 並取得訂用帳戶金鑰 

建立範例之前，您必須先訂閱「電腦視覺 API」，這是「Azure 認知服務」的組件。 如需訂用帳戶和金鑰管理詳細資料，請參閱[訂用帳戶](https://azure.microsoft.com/try/cognitive-services/)。 在本教學課程中，主要金鑰和次要金鑰都是有可用的有效金鑰。 

## <a name="acquire-the-incomplete-tutorial-project"></a>取得不完整的教學課程專案

### <a name="download-the-tutorial-project"></a>下載教學課程專案

1. 移至[認知服務 Java 電腦視覺教學課程](https://github.com/Azure-Samples/cognitive-services-java-computer-vision-tutorial) \(英文\) 存放庫。
1. 按一下 [複製或下載] 按鈕。
1. 按一下 [下載 ZIP] 以下載教學課程專案的 .zip 檔案。

因為 NetBeans 會從 .zip 檔案匯入專案，所以不需要將 .zip 檔案的內容解壓縮。

### <a name="import-the-tutorial-project"></a>匯入教學課程專案

將 **cognitive-services-java-computer-vision-tutorial-master.zip** 檔案匯入 NetBeans。

1. 在 NetBeans 中，按一下 [檔案] > [匯入專案] > [從 ZIP]。[從 ZIP 匯入專案] 對話方塊隨即出現。
1. 在 [ZIP 檔案:] 欄位中，按一下 [瀏覽] 按鈕以尋找 **cognitive-services-java-computer-vision-tutorial-master.zip** 檔案後，按一下 [開啟]。
1. 按一下 [從 ZIP 匯入專案] 對話方塊中的 [匯入]。
1. 在 [專案] 面板中，展開 [ComputerVision] > [原始碼套件] > [&lt;預設套件&gt;]。 
   有些 NetBeans 版本使用 [src]，而不是 [原始碼套件] > [&lt;預設套件&gt;]。 在此情況下，請展開 [src]。
1. 按兩下 **MainFrame.java** 以將該檔案載入 NetBeans 編輯器。 **MainFrame.java** 檔案的 [設計] 索引標籤隨即出現。
1. 按一下 [來源] 索引標籤來檢視 Java 原始程式碼。

### <a name="build-and-run-the-tutorial-project"></a>建置並執行教學課程專案

1. 按 **F6** 以建置並執行教學課程應用程式。

    在教學課程應用程式中，按一下索引標籤以顯示該功能的窗格。 按鈕的方法是空的，因此不會執行任何動作。

    [訂用帳戶金鑰] 與 [訂用帳戶區域] 欄位位於視窗底部。 必須在這些欄位中填入有效的訂用帳戶金鑰與該訂用帳戶金鑰的正確區域。 若要取得訂用帳戶金鑰，請參閱[訂用帳戶](https://azure.microsoft.com/try/cognitive-services/)。 如果您從該連結的試用版取得訂用帳戶金鑰，預設區域 **westcentralus** 就是您訂用帳戶金鑰的正確區域。

1. 結束教學課程應用程式。

## <a name="add-the-tutorial-code-to-the-project"></a>將教學課程的程式碼新增至專案中

Java Swing 應用程式已設有六個索引標籤。 每個索引標籤皆示範一種不同的「電腦視覺」功能 (分析、OCR 等)。 六個教學課程區段沒有相互依存性，因此您可以新增一個區段、全部六個區段，或任何子集。 您可依任意順序新增區段。

### <a name="analyze-an-image"></a>分析影像

「電腦視覺」的「分析」功能可掃描影像中超過 2,000 個可辨識的物體、生物、景象及動作。 在分析完成之後，「分析」就會傳回一個 JSON 物件，其中會以描述性標籤、色彩分析、標題等來描述影像。

若要完成教學課程應用程式的「分析」功能，請執行下列步驟：

#### <a name="add-the-event-handler-code-for-the-form-button"></a>新增表單按鈕的事件處理常式程式碼

**analyzeImageButtonActionPerformed** 事件處理常式方法會清除表單、顯示 URL 中指定的影像，然後呼叫 **AnalyzeImage** 方法來分析影像。 當 **AnalyzeImage** 傳回時，該方法會在 [回應] 文字區域中顯示格式化 JSON 回應，從 **JSONObject** 擷取第一個標題，然後顯示該標題和標題是否正確的信賴等級。

將下列程式碼複製並貼到 **analyzeImageButtonActionPerformed** 方法中。

> [!NOTE]
> NetBeans 不會讓您貼到方法定義行 (```private void```) 或該方法的右大括號。 若要複製程式碼，請複製方法定義與右大括號之間的各行，再貼到方法的內容。

```java
    private void analyzeImageButtonActionPerformed(java.awt.event.ActionEvent evt) {
        URL analyzeImageUrl;
        
        // Clear out the previous image, response, and caption, if any.
        analyzeImage.setIcon(new ImageIcon());
        analyzeCaptionLabel.setText("");
        analyzeResponseTextArea.setText("");
        
        // Display the image specified in the text box.
        try {
            analyzeImageUrl = new URL(analyzeImageUriTextBox.getText());
            BufferedImage bImage = ImageIO.read(analyzeImageUrl);
            scaleAndShowImage(bImage, analyzeImage);
        } catch(IOException e) {
            analyzeResponseTextArea.setText("Error loading Analyze image: " + e.getMessage());
            return;
        }
        
        // Analyze the image.
        JSONObject jsonObj = AnalyzeImage(analyzeImageUrl.toString());
        
        // A return of null indicates failure.
        if (jsonObj == null) {
            return;
        }
        
        // Format and display the JSON response.
        analyzeResponseTextArea.setText(jsonObj.toString(2));
                
        // Extract the text and confidence from the first caption in the description object.
        if (jsonObj.has("description") && jsonObj.getJSONObject("description").has("captions")) {

            JSONObject jsonCaption = jsonObj.getJSONObject("description").getJSONArray("captions").getJSONObject(0);
            
            if (jsonCaption.has("text") && jsonCaption.has("confidence")) {
                
                analyzeCaptionLabel.setText("Caption: " + jsonCaption.getString("text") + 
                        " (confidence: " + jsonCaption.getDouble("confidence") + ").");
            }
        }
    }
```

#### <a name="add-the-wrapper-for-the-rest-api-call"></a>新增 REST API 呼叫的包裝函式

**AnalyzeImage** 方法會包裝 REST API 呼叫來分析影像。 此方法會傳回描述影像的 **JSONObject**，如果發生錯誤，則傳回 **Null**。

複製 **AnalyzeImage** 方法並貼在 **analyzeImageButtonActionPerformed** 方法下方。

```java
    /**
     * Encapsulates the Microsoft Cognitive Services REST API call to analyze an image.
     * @param imageUrl: The string URL of the image to analyze.
     * @return: A JSONObject describing the image, or null if a runtime error occurs.
     */
    private JSONObject AnalyzeImage(String imageUrl) {
        try (CloseableHttpClient httpclient = HttpClientBuilder.create().build())
        {
            // Create the URI to access the REST API call for Analyze Image.
            String uriString = uriBasePreRegion + 
                    String.valueOf(subscriptionRegionComboBox.getSelectedItem()) + 
                    uriBasePostRegion + uriBaseAnalyze;
            URIBuilder builder = new URIBuilder(uriString);

            // Request parameters. All of them are optional.
            builder.setParameter("visualFeatures", "Categories,Description,Color,Adult");
            builder.setParameter("language", "en");

            // Prepare the URI for the REST API call.
            URI uri = builder.build();
            HttpPost request = new HttpPost(uri);

            // Request headers.
            request.setHeader("Content-Type", "application/json");
            request.setHeader("Ocp-Apim-Subscription-Key", subscriptionKeyTextField.getText());

            // Request body.
            StringEntity reqEntity = new StringEntity("{\"url\":\"" + imageUrl + "\"}");
            request.setEntity(reqEntity);

            // Execute the REST API call and get the response entity.
            HttpResponse response = httpclient.execute(request);
            HttpEntity entity = response.getEntity();

            // If we got a response, parse it and display it.
            if (entity != null)
            {
                // Return the JSONObject.
                String jsonString = EntityUtils.toString(entity);
                return new JSONObject(jsonString);
            } else {
                // No response. Return null.
                return null;
            }
        }
        catch (Exception e)
        {
            // Display error message.
            System.out.println(e.getMessage());
            return null;
        }
    }
 ```

#### <a name="run-the-application"></a>執行應用程式

按 **F6** 執行應用程式。 將您的訂用帳戶金鑰放入 [訂用帳戶金鑰] 欄位中，然後確認在 [訂用帳戶區域] 中使用的區域正確。 輸入要分析之影像的 URL，然後按一下 [分析影像] 按鈕來分析影像並查看結果。

### <a name="recognize-a-landmark"></a>辨識地標

「電腦視覺」的「地標」功能可分析影像中的自然和人工地標，例如山地或知名的建築物。 在分析完成之後，「地標」就會傳回一個 JSON 物件，其中會識別在影像中找到的地標。

若要完成教學課程應用程式的「地標」功能，請執行下列步驟：

#### <a name="add-the-event-handler-code-for-the-form-button"></a>新增表單按鈕的事件處理常式程式碼

**landmarkImageButtonActionPerformed** 事件處理常式方法會清除表單、顯示 URL 中指定的影像，然後呼叫 **LandmarkImage** 方法來分析影像。 當 **LandmarkImage** 傳回時，該方法會在 [回應] 文字區域中顯示格式化 JSON 回應，從 **JSONObject** 擷取第一個地標名稱，然後在視窗上顯示該地標和是否正確識別地標的信賴等級。

將下列程式碼複製並貼到 **landmarkImageButtonActionPerformed** 方法中。

> [!NOTE]
> NetBeans 不會讓您貼到方法定義行 (```private void```) 或該方法的右大括號。 若要複製程式碼，請複製方法定義與右大括號之間的各行，再貼到方法的內容。

```java
    private void landmarkImageButtonActionPerformed(java.awt.event.ActionEvent evt) {
        URL landmarkImageUrl;
        
        // Clear out the previous image, response, and caption, if any.
        landmarkImage.setIcon(new ImageIcon());
        landmarkCaptionLabel.setText("");
        landmarkResponseTextArea.setText("");
        
        // Display the image specified in the text box.
        try {
            landmarkImageUrl = new URL(landmarkImageUriTextBox.getText());
            BufferedImage bImage = ImageIO.read(landmarkImageUrl);
            scaleAndShowImage(bImage, landmarkImage);
        } catch(IOException e) {
            landmarkResponseTextArea.setText("Error loading Landmark image: " + e.getMessage());
            return;
        }
        
        // Identify the landmark in the image.
        JSONObject jsonObj = LandmarkImage(landmarkImageUrl.toString());
        
        // A return of null indicates failure.
        if (jsonObj == null) {
            return;
        }
        
        // Format and display the JSON response.
        landmarkResponseTextArea.setText(jsonObj.toString(2));
                
        // Extract the text and confidence from the first caption in the description object.
        if (jsonObj.has("result") && jsonObj.getJSONObject("result").has("landmarks")) {

            JSONObject jsonCaption = jsonObj.getJSONObject("result").getJSONArray("landmarks").getJSONObject(0);
            
            if (jsonCaption.has("name") && jsonCaption.has("confidence")) {

                landmarkCaptionLabel.setText("Caption: " + jsonCaption.getString("name") + 
                        " (confidence: " + jsonCaption.getDouble("confidence") + ").");
            }
        }
    }
```

#### <a name="add-the-wrapper-for-the-rest-api-call"></a>新增 REST API 呼叫的包裝函式

**LandmarkImage** 方法會包裝 REST API 呼叫來分析影像。 此方法會傳回描述影像中所找到地標的 **JSONObject**，如果發生錯誤，則傳回 **Null**。

複製 **LandmarkImage** 方法並貼在 **landmarkImageButtonActionPerformed** 方法下方。

```java
     /**
     * Encapsulates the Microsoft Cognitive Services REST API call to identify a landmark in an image.
     * @param imageUrl: The string URL of the image to process.
     * @return: A JSONObject describing the image, or null if a runtime error occurs.
     */
    private JSONObject LandmarkImage(String imageUrl) {
        try (CloseableHttpClient httpclient = HttpClientBuilder.create().build())
        {
            // Create the URI to access the REST API call to identify a Landmark in an image.
            String uriString = uriBasePreRegion + 
                    String.valueOf(subscriptionRegionComboBox.getSelectedItem()) + 
                    uriBasePostRegion + uriBaseLandmark;
            URIBuilder builder = new URIBuilder(uriString);

            // Request parameters. All of them are optional.
            builder.setParameter("visualFeatures", "Categories,Description,Color");
            builder.setParameter("language", "en");

            // Prepare the URI for the REST API call.
            URI uri = builder.build();
            HttpPost request = new HttpPost(uri);

            // Request headers.
            request.setHeader("Content-Type", "application/json");
            request.setHeader("Ocp-Apim-Subscription-Key", subscriptionKeyTextField.getText());

            // Request body.
            StringEntity reqEntity = new StringEntity("{\"url\":\"" + imageUrl + "\"}");
            request.setEntity(reqEntity);

            // Execute the REST API call and get the response entity.
            HttpResponse response = httpclient.execute(request);
            HttpEntity entity = response.getEntity();

            // If we got a response, parse it and display it.
            if (entity != null)
            {
                // Return the JSONObject.
                String jsonString = EntityUtils.toString(entity);
                return new JSONObject(jsonString);
            } else {
                // No response. Return null.
                return null;
            }
        }
        catch (Exception e)
        {
            // Display error message.
            System.out.println(e.getMessage());
            return null;
        }
    }
```

#### <a name="run-the-application"></a>執行應用程式

按 **F6** 執行應用程式。 將您的訂用帳戶金鑰放入 [訂用帳戶金鑰] 欄位中，然後確認在 [訂用帳戶區域] 中使用的區域正確。 按一下 [地標] 索引標籤，輸入要地標影像的 URL，然後按一下 [分析影像] 按鈕來分析影像並查看結果。

### <a name="recognize-celebrities"></a>辨識名人

「電腦視覺」的「名人」功能可分析影像中的知名人物。 在分析完成之後，「名人」就會傳回一個 JSON 物件，其中會識別在影像中找到的名人。

若要完成教學課程應用程式的「名人」功能，請執行下列步驟：

#### <a name="add-the-event-handler-code-for-the-form-button"></a>新增表單按鈕的事件處理常式程式碼

**celebritiesImageButtonActionPerformed** 事件處理常式方法會清除表單、顯示 URL 中指定的影像，然後呼叫 **CelebritiesImage** 方法來分析影像。 當 **CelebritiesImage** 傳回時，該方法會在 [回應] 文字區域中顯示格式化 JSON 回應，從 **JSONObject** 擷取第一個名人名稱，然後在視窗上顯示該名稱和是否正確識別名人的信賴等級。

將下列程式碼複製並貼到 **celebritiesImageButtonActionPerformed** 方法中。

> [!NOTE]
> NetBeans 不會讓您貼到方法定義行 (```private void```) 或該方法的右大括號。 若要複製程式碼，請複製方法定義與右大括號之間的各行，再貼到方法的內容。

```java
    private void celebritiesImageButtonActionPerformed(java.awt.event.ActionEvent evt) {
        URL celebritiesImageUrl;
        
        // Clear out the previous image, response, and caption, if any.
        celebritiesImage.setIcon(new ImageIcon());
        celebritiesCaptionLabel.setText("");
        celebritiesResponseTextArea.setText("");
        
        // Display the image specified in the text box.
        try {
            celebritiesImageUrl = new URL(celebritiesImageUriTextBox.getText());
            BufferedImage bImage = ImageIO.read(celebritiesImageUrl);
            scaleAndShowImage(bImage, celebritiesImage);
        } catch(IOException e) {
            celebritiesResponseTextArea.setText("Error loading Celebrity image: " + e.getMessage());
            return;
        }
        
        // Identify the celebrities in the image.
        JSONObject jsonObj = CelebritiesImage(celebritiesImageUrl.toString());
        
        // A return of null indicates failure.
        if (jsonObj == null) {
            return;
        }
        
        // Format and display the JSON response.
        celebritiesResponseTextArea.setText(jsonObj.toString(2));
                
        // Extract the text and confidence from the first caption in the description object.
        if (jsonObj.has("result") && jsonObj.getJSONObject("result").has("celebrities")) {

            JSONObject jsonCaption = jsonObj.getJSONObject("result").getJSONArray("celebrities").getJSONObject(0);
            
            if (jsonCaption.has("name") && jsonCaption.has("confidence")) {

                celebritiesCaptionLabel.setText("Caption: " + jsonCaption.getString("name") + 
                        " (confidence: " + jsonCaption.getDouble("confidence") + ").");
            }
        }
    }
```

#### <a name="add-the-wrapper-for-the-rest-api-call"></a>新增 REST API 呼叫的包裝函式

**CelebritiesImage** 方法會包裝 REST API 呼叫來分析影像。 此方法會傳回描述影像中所找到名人的 **JSONObject**，如果發生錯誤，則傳回 **Null**。

複製 **CelebritiesImage** 方法並貼在 **celebritiesImageButtonActionPerformed** 方法下方。

```java
     /**
     * Encapsulates the Microsoft Cognitive Services REST API call to identify celebrities in an image.
     * @param imageUrl: The string URL of the image to process.
     * @return: A JSONObject describing the image, or null if a runtime error occurs.
     */
    private JSONObject CelebritiesImage(String imageUrl) {
        try (CloseableHttpClient httpclient = HttpClientBuilder.create().build())
        {
            // Create the URI to access the REST API call to identify celebrities in an image.
            String uriString = uriBasePreRegion + 
                    String.valueOf(subscriptionRegionComboBox.getSelectedItem()) + 
                    uriBasePostRegion + uriBaseCelebrities;
            URIBuilder builder = new URIBuilder(uriString);

            // Request parameters. All of them are optional.
            builder.setParameter("visualFeatures", "Categories,Description,Color");
            builder.setParameter("language", "en");

            // Prepare the URI for the REST API call.
            URI uri = builder.build();
            HttpPost request = new HttpPost(uri);

            // Request headers.
            request.setHeader("Content-Type", "application/json");
            request.setHeader("Ocp-Apim-Subscription-Key", subscriptionKeyTextField.getText());

            // Request body.
            StringEntity reqEntity = new StringEntity("{\"url\":\"" + imageUrl + "\"}");
            request.setEntity(reqEntity);

            // Execute the REST API call and get the response entity.
            HttpResponse response = httpclient.execute(request);
            HttpEntity entity = response.getEntity();

            // If we got a response, parse it and display it.
            if (entity != null)
            {
                // Return the JSONObject.
                String jsonString = EntityUtils.toString(entity);
                return new JSONObject(jsonString);
            } else {
                // No response. Return null.
                return null;
            }
        }
        catch (Exception e)
        {
            // Display error message.
            System.out.println(e.getMessage());
            return null;
        }
    }
```

#### <a name="run-the-application"></a>執行應用程式

按 **F6** 執行應用程式。 將您的訂用帳戶金鑰放入 [訂用帳戶金鑰] 欄位中，然後確認在 [訂用帳戶區域] 中使用的區域正確。 按一下 [名人] 索引標籤，輸入名人影像的 URL，然後按一下 [分析影像] 按鈕來分析影像並查看結果。

### <a name="intelligently-generate-a-thumbnail"></a>以智慧方式產生縮圖

「電腦視覺」的「縮圖」功能可從影像產生縮圖。 透過使用「智慧型裁剪」功能，「縮圖」功能將可識別影像中的關注區並將縮圖集中在此區域上，以產生更賞心悅目的縮圖影像。

若要完成教學課程應用程式的「縮圖」功能，請執行下列步驟：

#### <a name="add-the-event-handler-code-for-the-form-button"></a>新增表單按鈕的事件處理常式程式碼

**thumbnailImageButtonActionPerformed** 事件處理常式方法會清除表單、顯示 URL 中指定的影像，然後呼叫 **getThumbnailImage** 方法來建立縮圖。 當 **getThumbnailImage** 傳回時，該方法會顯示產生的縮圖。

將以下程式碼複製並貼到 **thumbnailImageButtonActionPerformed** 方法中。

> [!NOTE]
> NetBeans 不會讓您貼到方法定義行 (```private void```) 或該方法的右大括號。 若要複製程式碼，請複製方法定義與右大括號之間的各行，再貼到方法的內容。

```java
    private void thumbnailImageButtonActionPerformed(java.awt.event.ActionEvent evt) {
        URL thumbnailImageUrl;
        JSONObject jsonError[] = new JSONObject[1];
        
        // Clear out the previous image, response, and thumbnail, if any.
        thumbnailSourceImage.setIcon(new ImageIcon());
        thumbnailResponseTextArea.setText("");
        thumbnailImage.setIcon(new ImageIcon());

        // Display the image specified in the text box.
        try {
            thumbnailImageUrl = new URL(thumbnailImageUriTextBox.getText());
            BufferedImage bImage = ImageIO.read(thumbnailImageUrl);
            scaleAndShowImage(bImage, thumbnailSourceImage);
        } catch(IOException e) {
            thumbnailResponseTextArea.setText("Error loading image to thumbnail: " + e.getMessage());
            return;
        }
        
        // Get the thumbnail for the image.
        BufferedImage thumbnail = getThumbnailImage(thumbnailImageUrl.toString(), jsonError);
        
        // A non-null value indicates error.
        if (jsonError[0] != null) {
            // Format and display the JSON error.
            thumbnailResponseTextArea.setText(jsonError[0].toString(2));
            return;
        }
        
        // Display the thumbnail.
        if (thumbnail != null) {
            scaleAndShowImage(thumbnail, thumbnailImage);
        }
    }
```

#### <a name="add-the-wrapper-for-the-rest-api-call"></a>新增 REST API 呼叫的包裝函式

**getThumbnailImage** 方法會包裝 REST API 呼叫來分析影像。 此方法會傳回包含縮圖的 **BufferedImage**，如果發生錯誤，則傳回 **Null**。 錯誤訊息會以 **jsonError** 字串陣列的第一個元素傳回。

將下列 **getThumbnailImage** 方法複製並貼到緊接在 **thumbnailImageButtonActionPerformed** 方法的下方。

```java
     /**
     * Encapsulates the Microsoft Cognitive Services REST API call to create a thumbnail for an image.
     * @param imageUrl: The string URL of the image to process.
     * @return: A BufferedImage containing the thumbnail, or null if a runtime error occurs. In the case
     * of an error, the error message will be returned in the first element of the jsonError string array.
     */
    private BufferedImage getThumbnailImage(String imageUrl, JSONObject[] jsonError) {
        try (CloseableHttpClient httpclient = HttpClientBuilder.create().build())
        {
            // Create the URI to access the REST API call to identify celebrities in an image.
            String uriString = uriBasePreRegion + 
                    String.valueOf(subscriptionRegionComboBox.getSelectedItem()) + 
                    uriBasePostRegion + uriBaseThumbnail;
            URIBuilder uriBuilder = new URIBuilder(uriString);

            // Request parameters.
            uriBuilder.setParameter("width", "100");
            uriBuilder.setParameter("height", "150");
            uriBuilder.setParameter("smartCropping", "true");

            // Prepare the URI for the REST API call.
            URI uri = uriBuilder.build();
            HttpPost request = new HttpPost(uri);

            // Request headers.
            request.setHeader("Content-Type", "application/json");
            request.setHeader("Ocp-Apim-Subscription-Key", subscriptionKeyTextField.getText());

            // Request body.
            StringEntity requestEntity = new StringEntity("{\"url\":\"" + imageUrl + "\"}");
            request.setEntity(requestEntity);

            // Execute the REST API call and get the response entity.
            HttpResponse response = httpclient.execute(request);
            HttpEntity entity = response.getEntity();

            // Check for success.
            if (response.getStatusLine().getStatusCode() == 200)
            {
                // Return the thumbnail.
                return ImageIO.read(entity.getContent());
            }
            else
            {
                // Format and display the JSON error message.
                String jsonString = EntityUtils.toString(entity);
                jsonError[0] = new JSONObject(jsonString);
                return null;
            }
        }
        catch (Exception e)
        {
            String errorMessage = e.getMessage();
            System.out.println(errorMessage);
            jsonError[0] = new JSONObject(errorMessage);
            return null;
        }
    }
```

#### <a name="run-the-application"></a>執行應用程式

按 **F6** 執行應用程式。 將您的訂用帳戶金鑰放入 [訂用帳戶金鑰] 欄位中，然後確認在 [訂用帳戶區域] 中使用的區域正確。 按一下 [縮圖] 索引標籤，輸入影像的 URL，然後按一下 [產生縮圖] 按鈕來分析影像並查看結果。

### <a name="read-printed-text-ocr"></a>讀取列印文字 (OCR)

「電腦視覺」的「光學字元辨識」(OCR) 功能可分析列印文字的影像。 在分析完成之後，OCR 就會傳回一個 JSON 物件，其中包含影像中的文字和文字位置。

若要完成教學課程應用程式的 OCR 功能，請執行下列步驟：

#### <a name="add-the-event-handler-code-for-the-form-button"></a>新增表單按鈕的事件處理常式程式碼

**ocrImageButtonActionPerformed** 事件處理常式方法會清除表單、顯示 URL 中指定的影像，然後呼叫 **OcrImage** 方法來分析影像。 當 **OcrImage** 傳回時，該方法會在 [回應] 文字區域中，將偵測到的文字顯示為格式化 JSON。

將下列程式碼複製並貼到 **ocrImageButtonActionPerformed** 方法中。

> [!NOTE]
> NetBeans 不會讓您貼到方法定義行 (```private void```) 或該方法的右大括號。 若要複製程式碼，請複製方法定義與右大括號之間的各行，再貼到方法的內容。

```java
    private void ocrImageButtonActionPerformed(java.awt.event.ActionEvent evt) {
        URL ocrImageUrl;
        
        // Clear out the previous image, response, and caption, if any.
        ocrImage.setIcon(new ImageIcon());
        ocrResponseTextArea.setText("");
        
        // Display the image specified in the text box.
        try {
            ocrImageUrl = new URL(ocrImageUriTextBox.getText());
            BufferedImage bImage = ImageIO.read(ocrImageUrl);
            scaleAndShowImage(bImage, ocrImage);
        } catch(IOException e) {
            ocrResponseTextArea.setText("Error loading OCR image: " + e.getMessage());
            return;
        }
        
        // Read the text in the image.
        JSONObject jsonObj = OcrImage(ocrImageUrl.toString());
        
        // A return of null indicates failure.
        if (jsonObj == null) {
            return;
        }
        
        // Format and display the JSON response.
        ocrResponseTextArea.setText(jsonObj.toString(2));
    }
```

#### <a name="add-the-wrapper-for-the-rest-api-call"></a>新增 REST API 呼叫的包裝函式

**OcrImage** 方法會包裝 REST API 呼叫來分析影像。 此方法會傳回從呼叫所傳回 JSON 資料的 **JSONObject**，如果發生錯誤，則傳回 **Null**。

複製 **OcrImage** 方法並貼在 **ocrImageButtonActionPerformed** 方法下方。

```java
     /**
     * Encapsulates the Microsoft Cognitive Services REST API call to read text in an image.
     * @param imageUrl: The string URL of the image to process.
     * @return: A JSONObject describing the image, or null if a runtime error occurs.
     */
    private JSONObject OcrImage(String imageUrl) {
        try (CloseableHttpClient httpclient = HttpClientBuilder.create().build())
        {
            // Create the URI to access the REST API call to read text in an image.
            String uriString = uriBasePreRegion + 
                    String.valueOf(subscriptionRegionComboBox.getSelectedItem()) + 
                    uriBasePostRegion + uriBaseOcr;
            URIBuilder uriBuilder = new URIBuilder(uriString);

            // Request parameters.
            uriBuilder.setParameter("language", "unk");
            uriBuilder.setParameter("detectOrientation ", "true");

            // Prepare the URI for the REST API call.
            URI uri = uriBuilder.build();
            HttpPost request = new HttpPost(uri);

            // Request headers.
            request.setHeader("Content-Type", "application/json");
            request.setHeader("Ocp-Apim-Subscription-Key", subscriptionKeyTextField.getText());

            // Request body.
            StringEntity reqEntity = new StringEntity("{\"url\":\"" + imageUrl + "\"}");
            request.setEntity(reqEntity);

            // Execute the REST API call and get the response entity.
            HttpResponse response = httpclient.execute(request);
            HttpEntity entity = response.getEntity();

            // If we got a response, parse it and display it.
            if (entity != null)
            {
                // Return the JSONObject.
                String jsonString = EntityUtils.toString(entity);
                return new JSONObject(jsonString);
            } else {
                // No response. Return null.
                return null;
            }
        }
        catch (Exception e)
        {
            // Display error message.
            System.out.println(e.getMessage());
            return null;
        }
    }
```

#### <a name="run-the-application"></a>執行應用程式

按 **F6** 執行應用程式。 將您的訂用帳戶金鑰放入 [訂用帳戶金鑰] 欄位中，然後確認在 [訂用帳戶區域] 中使用的區域正確。 按一下 [OCR] 索引標籤，輸入列印文字影像的 URL，然後按一下 [讀取影像] 按鈕來分析影像並查看結果。

### <a name="read-handwritten-text-handwriting-recognition"></a>讀取手寫文字 (手寫辨識)

「電腦視覺」的「手寫辨識」功能可分析手寫文字的影像。 在分析完成之後，「手寫辨識」就會傳回一個 JSON 物件，其中包含影像中的文字和文字位置。

若要完成教學課程應用程式的「手寫辨識」功能，請執行下列步驟：

#### <a name="add-the-event-handler-code-for-the-form-button"></a>新增表單按鈕的事件處理常式程式碼

**handwritingImageButtonActionPerformed** 事件處理常式方法會清除表單、顯示 URL 中指定的影像，然後呼叫 **HandwritingImage** 方法來分析影像。 當 **HandwritingImage** 傳回時，該方法會在 [回應] 文字區域中，將偵測到的文字顯示為格式化 JSON。

將下列程式碼複製並貼到 **handwritingImageButtonActionPerformed** 方法中。

> [!NOTE]
> NetBeans 不會讓您貼到方法定義行 (```private void```) 或該方法的右大括號。 若要複製程式碼，請複製方法定義與右大括號之間的各行，再貼到方法的內容。

```java
    private void handwritingImageButtonActionPerformed(java.awt.event.ActionEvent evt) {
        URL handwritingImageUrl;
        
        // Clear out the previous image, response, and caption, if any.
        handwritingImage.setIcon(new ImageIcon());
        handwritingResponseTextArea.setText("");
        
        // Display the image specified in the text box.
        try {
            handwritingImageUrl = new URL(handwritingImageUriTextBox.getText());
            BufferedImage bImage = ImageIO.read(handwritingImageUrl);
            scaleAndShowImage(bImage, handwritingImage);
        } catch(IOException e) {
            handwritingResponseTextArea.setText("Error loading Handwriting image: " + e.getMessage());
            return;
        }
        
        // Read the text in the image.
        JSONObject jsonObj = HandwritingImage(handwritingImageUrl.toString());
        
        // A return of null indicates failure.
        if (jsonObj == null) {
            return;
        }
        
        // Format and display the JSON response.
        handwritingResponseTextArea.setText(jsonObj.toString(2));
    }
```

#### <a name="add-the-wrapper-for-the-rest-api-call"></a>新增 REST API 呼叫的包裝函式

**HandwritingImage** 方法會包裝 REST API 呼叫來分析影像。 由於手寫辨識是一個費時的程序，因此會使用一個雙步驟程序。 第一個呼叫會提交要處理的影像；第二個呼叫會在處理完成時，擷取所偵測到的文字。

擷取文字之後，**HandwritingImage** 方法會傳回描述文字與該文字位置的 **JSONObject**，如果發生錯誤，則傳回 **Null**。

複製 **HandwritingImage** 方法並貼在 **handwritingImageButtonActionPerformed** 方法下方。

```java
     /**
     * Encapsulates the Microsoft Cognitive Services REST API call to read handwritten text in an image.
     * @param imageUrl: The string URL of the image to process.
     * @return: A JSONObject describing the image, or null if a runtime error occurs.
     */
    private JSONObject HandwritingImage(String imageUrl) {
        try (CloseableHttpClient textClient = HttpClientBuilder.create().build();
             CloseableHttpClient resultClient = HttpClientBuilder.create().build())
        {
            // Create the URI to access the REST API call to read text in an image.
            String uriString = uriBasePreRegion + 
                    String.valueOf(subscriptionRegionComboBox.getSelectedItem()) + 
                    uriBasePostRegion + uriBaseHandwriting;
            URIBuilder uriBuilder = new URIBuilder(uriString);
            
            // Request parameters.
            uriBuilder.setParameter("handwriting", "true");

            // Prepare the URI for the REST API call.
            URI uri = uriBuilder.build();
            HttpPost request = new HttpPost(uri);

            // Request headers.
            request.setHeader("Content-Type", "application/json");
            request.setHeader("Ocp-Apim-Subscription-Key", subscriptionKeyTextField.getText());

            // Request body.
            StringEntity reqEntity = new StringEntity("{\"url\":\"" + imageUrl + "\"}");
            request.setEntity(reqEntity);

            // Execute the REST API call and get the response.
            HttpResponse textResponse = textClient.execute(request);
            
            // Check for success.
            if (textResponse.getStatusLine().getStatusCode() != 202) {
                // An error occured. Return the JSON error message.
                HttpEntity entity = textResponse.getEntity();
                String jsonString = EntityUtils.toString(entity);
                return new JSONObject(jsonString);
            }
            
            String operationLocation = null;

            // The 'Operation-Location' in the response contains the URI to retrieve the recognized text.
            Header[] responseHeaders = textResponse.getAllHeaders();
            for(Header header : responseHeaders) {
                if(header.getName().equals("Operation-Location"))
                {
                    // This string is the URI where you can get the text recognition operation result.
                    operationLocation = header.getValue();
                    break;
                }
            }
            
            // NOTE: The response may not be immediately available. Handwriting recognition is an
            // async operation that can take a variable amount of time depending on the length
            // of the text you want to recognize. You may need to wait or retry this operation.
            //
            // This example checks once per second for ten seconds.
            
            JSONObject responseObj = null;
            int i = 0;
            do {
                // Wait one second.
                Thread.sleep(1000);
                
                // Check to see if the operation completed.
                HttpGet resultRequest = new HttpGet(operationLocation);
                resultRequest.setHeader("Ocp-Apim-Subscription-Key", subscriptionKeyTextField.getText());
                HttpResponse resultResponse = resultClient.execute(resultRequest);
                HttpEntity responseEntity = resultResponse.getEntity();
                if (responseEntity != null)
                {
                    // Get the JSON response.
                    String jsonString = EntityUtils.toString(responseEntity);
                    responseObj = new JSONObject(jsonString);
                }
            }
            while (i < 10 && responseObj != null &&
                    !responseObj.getString("status").equalsIgnoreCase("Succeeded"));
            
            // If the operation completed, return the JSON object.
            if (responseObj != null) {
                return responseObj;
            } else {
                // Return null for timeout error.
                System.out.println("Timeout error.");
                return null;
            }
        }
        catch (Exception e)
        {
            // Display error message.
            System.out.println(e.getMessage());
            return null;
        }
    }
```

#### <a name="run-the-application"></a>執行應用程式

若要執行應用程式，請按 **F6**。 將您的訂用帳戶金鑰放入 [訂用帳戶金鑰] 欄位中，然後確認在 [訂用帳戶區域] 中使用的區域正確。 按一下 [讀取手寫文字] 索引標籤，輸入手寫文字影像的 URL，然後按一下 [讀取影像] 按鈕來分析影像並查看結果。

## <a name="next-steps"></a>後續步驟

- [電腦視覺 API C&#35; 教學課程](CSharpTutorial.md)
- [電腦視覺 API Python 教學課程](PythonTutorial.md)
