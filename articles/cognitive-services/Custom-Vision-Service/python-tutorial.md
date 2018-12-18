---
title: 教學課程：建立影像分類專案 - 自訂視覺服務、Python
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
ms.openlocfilehash: 14b805a60637a889698132e169d5a41670a8bce0
ms.sourcegitcommit: ce526d13cd826b6f3e2d80558ea2e289d034d48f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46363372"
---
# <a name="tutorial-create-an-image-classification-project-using-the-custom-vision-service-with-python"></a>教學課程：使用自訂視覺服務搭配 Python 建立影像分類專案

了解如何使用自訂視覺服務和基本的 Python 指令碼來建立影像分類專案。 建立它之後，您就可以新增標記、上傳影像、為專案定型、取得專案的預設預測端點 URL，並使用它以程式設計方式測試影像。 使用自訂視覺 API，利用此開放原始碼範例作為範本，來建置自己的應用程式。



## <a name="prerequisites"></a>必要條件

- Python 2.7+ 或 Python 3.5+。
- pip 工具。

## <a name="get-the-training-and-prediction-keys"></a>取得定型和預測金鑰

若要取得此範例中所使用的金鑰，請瀏覽[自訂視覺網頁](https://customvision.ai) \(英文\)，然後選取右上方的__齒輪圖示__。 在 [帳戶] 區段中，複製 [定型金鑰] 和 [預測金鑰] 欄位的值。

![金鑰 UI 的影像](./media/python-tutorial/training-prediction-keys.png)

## <a name="install-the-custom-vision-service-sdk"></a>安裝自訂視覺服務 SDK

若要安裝自訂視覺服務 SDK，請使用下列命令：

```
pip install azure-cognitiveservices-vision-customvision
```

## <a name="get-example-images"></a>取得範例影像

這個範例所使用的影像來自 [https://github.com/Microsoft/Cognitive-CustomVision-Windows](https://github.com/Microsoft/Cognitive-CustomVision-Windows/tree/master/Samples/Images) 專案的 `Samples/Images` 目錄。 複製或下載專案，並解壓縮到您的開發環境。

## <a name="create-a-custom-vision-service-project"></a>建立自訂視覺服務專案

若要建立新的自訂視覺服務專案，請建立名為 `sample.py` 的新檔案。 使用下列程式碼作為檔案內容：

> [!IMPORTANT]
> 將 `training_key` 設為您先前擷取的定型金鑰值。
>
> 將 `prediction_key` 設為您先前擷取的預測金鑰值。

```python
from azure.cognitiveservices.vision.customvision.training import training_api
from azure.cognitiveservices.vision.customvision.training.models import ImageUrlCreateEntry

# Replace with a valid key
training_key = "<your training key>"
prediction_key = "<your prediction key>"

trainer = training_api.TrainingApi(training_key)

# Create a new project
print ("Creating project...")
project = trainer.create_project("My Project")
```

## <a name="add-tags-to-your-project"></a>將標記新增到您的專案

若要將標記新增到您的專案，在 `sample.py` 檔案結尾處新增以下程式碼：

```python
# Make two tags in the new project
hemlock_tag = trainer.create_tag(project.id, "Hemlock")
cherry_tag = trainer.create_tag(project.id, "Japanese Cherry")
```

## <a name="upload-images-to-the-project"></a>將影像上傳到專案

若要將範例影像新增到專案，在標記建立之後插入下列程式碼。 此程式碼會上傳影像以及對應的標記：

> [!IMPORTANT]
>
> 根據您稍早下載 Cognitive-CustomVision-Windows 專案的位置，變更影像的路徑。

```python
base_image_url = "https://raw.githubusercontent.com/Microsoft/Cognitive-CustomVision-Windows/master/Samples/"

print ("Adding images...")
for image_num in range(1,10):
    image_url = base_image_url + "Images/Hemlock/hemlock_{}.jpg".format(image_num)
    trainer.create_images_from_urls(project.id, [ ImageUrlCreateEntry(url=image_url, tag_ids=[ hemlock_tag.id ] ) ])

for image_num in range(1,10):
    image_url = base_image_url + "Images/Japanese Cherry/japanese_cherry_{}.jpg".format(image_num)
    trainer.create_images_from_urls(project.id, [ ImageUrlCreateEntry(url=image_url, tag_ids=[ cherry_tag.id ] ) ])


# Alternatively, if the images were on disk in a folder called Images alongside the sample.py, then
# they can be added by using the following:
#
#import os
#hemlock_dir = "Images\\Hemlock"
#for image in os.listdir(os.fsencode("Images\\Hemlock")):
#    with open(hemlock_dir + "\\" + os.fsdecode(image), mode="rb") as img_data: 
#        trainer.create_images_from_data(project.id, img_data, [ hemlock_tag.id ])
#
#cherry_dir = "Images\\Japanese Cherry"
#for image in os.listdir(os.fsencode("Images\\Japanese Cherry")):
#    with open(cherry_dir + "\\" + os.fsdecode(image), mode="rb") as img_data: 
#        trainer.create_images_from_data(project.id, img_data, [ cherry_tag.id ])
```

## <a name="train-the-project"></a>為專案定型

若要為分類器定型，在 `sample.py` 檔案結尾處新增以下程式碼：

```python
import time

print ("Training...")
iteration = trainer.train_project(project.id)
while (iteration.status != "Completed"):
    iteration = trainer.get_iteration(project.id, iteration.id)
    print ("Training status: " + iteration.status)
    time.sleep(1)

# The iteration is now trained. Make it the default project endpoint
trainer.update_iteration(project.id, iteration.id, is_default=True)
print ("Done!")
```

## <a name="get-and-use-the-default-prediction-endpoint"></a>取得並使用預設預測端點

若要將影像傳送到預測端點並擷取預測，在 `sample.py` 檔案結尾處新增以下程式碼：

```python
from azure.cognitiveservices.vision.customvision.prediction import prediction_endpoint
from azure.cognitiveservices.vision.customvision.prediction.prediction_endpoint import models

# Now there is a trained endpoint that can be used to make a prediction

predictor = prediction_endpoint.PredictionEndpoint(prediction_key)

test_img_url = base_image_url + "Images/Test/test_image.jpg"
results = predictor.predict_image_url(project.id, iteration.id, url=test_img_url)

# Alternatively, if the images were on disk in a folder called Images alongside the sample.py, then
# they can be added by using the following.
#
# Open the sample image and get back the prediction results.
# with open("Images\\test\\test_image.jpg", mode="rb") as test_data:
#     results = predictor.predict_image(project.id, test_data, iteration.id)

# Display the results.
for prediction in results.predictions:
    print ("\t" + prediction.tag_name + ": {0:.2f}%".format(prediction.probability * 100))
```

## <a name="run-the-example"></a>執行範例

執行方案。 預測結果會出現在主控台上。

```
python sample.py
```

應用程式的輸出類似下列文字：

```
Creating project...
Adding images...
Training...
Training status: Training
Training status: Completed
Done!
        Hemlock: 93.53%
        Japanese Cherry: 0.01%
```