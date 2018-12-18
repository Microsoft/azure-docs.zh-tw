---
title: 教學課程：建置影像分類專案 - 自訂視覺服務、Java
titlesuffix: Azure Cognitive Services
description: 建立專案、新增標籤、上傳影像、為您的專案定型，以及使用預設端點進行預測。
services: cognitive-services
author: areddish
manager: cgronlun
ms.service: cognitive-services
ms.component: custom-vision
ms.topic: tutorial
ms.date: 08/28/2018
ms.author: areddish
ms.openlocfilehash: 9a7f50e0eb33016d6a2d8f28be047b327135c51f
ms.sourcegitcommit: ce526d13cd826b6f3e2d80558ea2e289d034d48f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46367350"
---
# <a name="tutorial-build-an-image-classification-project-with-java"></a>教學課程：使用 Java 建置影像分類專案

了解如何使用自訂視覺服務利用 Java 來建立影像分類專案。 建立它之後，您就可以新增標記、上傳影像、為專案定型、取得專案的預設預測端點 URL，並使用它以程式設計方式測試影像。 使用自訂視覺 API，利用此開放原始碼範例作為範本，來建置自己的應用程式。

## <a name="prerequisites"></a>必要條件

- 已安裝 JDK 7 或 8。
- 已安裝 Maven。

## <a name="get-the-training-and-prediction-keys"></a>取得定型與預測金鑰

若要取得此範例中所使用的金鑰，請瀏覽[自訂視覺網頁](https://customvision.ai) \(英文\)，然後選取右上方的__齒輪圖示__。 在 [帳戶] 區段中，複製 [定型金鑰] 和 [預測金鑰] 欄位的值。

![金鑰 UI 的影像](./media/python-tutorial/training-prediction-keys.png)

## <a name="install-the-custom-vision-service-sdk"></a>安裝自訂視覺服務 SDK

您可以從 Maven 中央存放庫安裝自訂視覺 SDK：
* [定型 SDK](https://mvnrepository.com/artifact/com.microsoft.azure.cognitiveservices/azure-cognitiveservices-customvision-training)
* [預測 SDK](https://mvnrepository.com/artifact/com.microsoft.azure.cognitiveservices/azure-cognitiveservices-customvision-prediction)

## <a name="understand-the-code"></a>了解程式碼

您可以從[適用於 Java 的自訂視覺 Azure 範例存放庫](https://github.com/Azure-Samples/cognitive-services-java-sdk-samples/tree/master)取得包含影像的完整專案。 

使用喜愛的 Java IDE 開啟 `Vision/CustomVision` 專案。 

此應用程式會使用您稍早擷取的定型金鑰，來建立名為 __Sample Java Project__ 的新專案。 接著，它會上傳影像以便為分類器定型並進行測試。 分類器會識別樹是 __Hemlock__ 還是 __Japanese Cherry__。

下列程式碼片段會實作此範例的主要功能：

## <a name="create-a-custom-vision-service-project"></a>建立自訂視覺服務專案

> [!IMPORTANT]
> 將 `trainingApiKey` 設為您先前擷取的定型金鑰值。

```java
final String trainingApiKey = "insert your training key here";
TrainingApi trainClient = CustomVisionTrainingManager.authenticate(trainingApiKey);

Trainings trainer = trainClient.trainings();

System.out.println("Creating project...");
Project project = trainer.createProject()
            .withName("Sample Java Project")
            .execute();
```

## <a name="add-tags-to-your-project"></a>將標記新增到您的專案

```java
// create hemlock tag
Tag hemlockTag = trainer.createTag()
    .withProjectId(project.id())
    .withName("Hemlock")
    .execute();

// create cherry tag
Tag cherryTag = trainer.createTag()
    .withProjectId(project.id())
    .withName("Japanese Cherry")
    .execute();
```

## <a name="upload-images-to-the-project"></a>將影像上傳到專案

此範例設定為將影像包含在最終套件中。 從 jar 的資源區段讀取影像，然後上傳到服務。

```java
System.out.println("Adding images...");
for (int i = 1; i <= 10; i++) {
    String fileName = "hemlock_" + i + ".jpg";
    byte[] contents = GetImage("/Hemlock", fileName);
    AddImageToProject(trainer, project, fileName, contents, hemlockTag.id(), null);
}

for (int i = 1; i <= 10; i++) {
    String fileName = "japanese_cherry_" + i + ".jpg";
    byte[] contents = GetImage("/Japanese Cherry", fileName);
    AddImageToProject(trainer, project, fileName, contents, cherryTag.id(), null);
}
```

先前的程式碼片段利用兩個協助程式函式，將影像擷取為資源串流，然後上傳到服務。

```java
private static void AddImageToProject(Trainings trainer, Project project, String fileName, byte[] contents, UUID tag)
{
    System.out.println("Adding image: " + fileName);

    ImageFileCreateEntry file = new ImageFileCreateEntry()
        .withName(fileName)
        .withContents(contents);

    ImageFileCreateBatch batch = new ImageFileCreateBatch()
        .withImages(Collections.singletonList(file));

    batch = batch.withTagIds(Collections.singletonList(tag));

    trainer.createImagesFromFiles(project.id(), batch);
}

private static byte[] GetImage(String folder, String fileName)
{
    try {
        return ByteStreams.toByteArray(CustomVisionSamples.class.getResourceAsStream(folder + "/" + fileName));
    } catch (Exception e) {
        System.out.println(e.getMessage());
        e.printStackTrace();
    }

    return null;
}
```

## <a name="train-the-project"></a>為專案定型

這會建立專案中的第一個反覆項目，並將此設定為預設的反覆項目。 

```java
System.out.println("Training...");
Iteration iteration = trainer.trainProject(project.id());

while (iteration.status().equals("Training"))
{
    System.out.println("Training Status: "+ iteration.status());
    Thread.sleep(1000);
    iteration = trainer.getIteration(project.id(), iteration.id());
}

System.out.println("Training Status: "+ iteration.status());
trainer.updateIteration(project.id(), iteration.id(), iteration.withIsDefault(true));
```

## <a name="get-and-use-the-default-prediction-endpoint"></a>取得並使用預設預測端點

> [!IMPORTANT]
> 將 `predictionApiKey` 設為您先前擷取的預測金鑰值。

```java
final String predictionApiKey = "insert your prediction key here";
PredictionEndpoint predictClient = CustomVisionPredictionManager.authenticate(predictionApiKey);

// Use below for predictions from a url
// String url = "some url";
// ImagePrediction results = predictor.predictions().predictImage()
//                         .withProjectId(project.id())
//                         .withUrl(url)
//                         .execute();

// load test image
byte[] testImage = GetImage("/Test", "test_image.jpg");

// predict
ImagePrediction results = predictor.predictions().predictImage()
    .withProjectId(project.id())
    .withImageData(testImage)
    .execute();

for (Prediction prediction: results.predictions())
{
    System.out.println(String.format("\t%s: %.2f%%", prediction.tagName(), prediction.probability() * 100.0f));
}
```

## <a name="run-the-example"></a>執行範例

使用 Maven 編譯並執行解決方案：

```
mvn compile exec:java
```

應用程式的輸出類似下列文字：

```
Creating project...
Adding images...
Adding image: hemlock_1.jpg
Adding image: hemlock_2.jpg
Adding image: hemlock_3.jpg
Adding image: hemlock_4.jpg
Adding image: hemlock_5.jpg
Adding image: hemlock_6.jpg
Adding image: hemlock_7.jpg
Adding image: hemlock_8.jpg
Adding image: hemlock_9.jpg
Adding image: hemlock_10.jpg
Adding image: japanese_cherry_1.jpg
Adding image: japanese_cherry_2.jpg
Adding image: japanese_cherry_3.jpg
Adding image: japanese_cherry_4.jpg
Adding image: japanese_cherry_5.jpg
Adding image: japanese_cherry_6.jpg
Adding image: japanese_cherry_7.jpg
Adding image: japanese_cherry_8.jpg
Adding image: japanese_cherry_9.jpg
Adding image: japanese_cherry_10.jpg
Training...
Training status: Training
Training status: Training
Training status: Completed
Done!
        Hemlock: 93.53%
        Japanese Cherry: 0.01%
```