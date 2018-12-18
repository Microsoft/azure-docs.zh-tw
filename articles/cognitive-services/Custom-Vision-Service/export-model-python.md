---
title: 教學課程：以 Python 執行 TensorFlow 模型 - 自訂視覺服務
titlesuffix: Azure Cognitive Services
description: 以 Python 執行 TensorFlow 模型。
services: cognitive-services
author: areddish
manager: cgronlun
ms.service: cognitive-services
ms.component: custom-vision
ms.topic: tutorial
ms.date: 05/17/2018
ms.author: areddish
ms.openlocfilehash: 26427406b045b96f2f3f612e4444b7dc2afcefc6
ms.sourcegitcommit: 609c85e433150e7c27abd3b373d56ee9cf95179a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/03/2018
ms.locfileid: "48247307"
---
# <a name="tutorial-run-tensorflow-model-in-python"></a>教學課程：以 Python 執行 TensorFlow 模型

在您從自訂視覺服務[匯出 TensorFlow 模型](https://docs.microsoft.com/azure/cognitive-services/custom-vision-service/export-your-model)之後，本快速入門將示範如何在本機上使用此模型，來分類影像。

## <a name="install-required-components"></a>安裝必要的元件

### <a name="prerequisites"></a>必要條件

若要使用教學課程，您需要執行下列動作：

- 安裝 Python 2.7+ 或 Python 3.5+。
- 安裝 pip。

您也需要安裝下列套件：

```
pip install tensorflow
pip install pillow
pip install numpy
pip install opencv-python
```

## <a name="load-your-model-and-tags"></a>載入模型和標籤

已下載的 ZIP 檔案包含 model.pb 和 labels.txt。 這些檔案代表訓練過的模型和分類標籤。 第一個步驟是將模型載入到您的專案。

```Python
import tensorflow as tf
import os

graph_def = tf.GraphDef()
labels = []

# Import the TF graph
with tf.gfile.FastGFile(filename, 'rb') as f:
    graph_def.ParseFromString(f.read())
    tf.import_graph_def(graph_def, name='')

# Create a list of labels.
with open(labels_filename, 'rt') as lf:
    for l in lf:
        labels.append(l.strip())
```

## <a name="prepare-an-image-for-prediction"></a>準備要預測的影像

準備影像有幾個步驟，以預測出正確的形狀。 這些步驟模擬在訓練期間所執行的影像操作：

### <a name="open-the-file-and-create-an-image-in-the-bgr-color-space"></a>開啟檔案，並在 BGR 色彩空間中建立影像

```Python
from PIL import Image
import numpy as np
import cv2

# Load from a file
imageFile = "<path to your image file>"
image = Image.open(imageFile)

# Update orientation based on EXIF tags, if the file has orientation info.
image = update_orientation(image)

# Convert to OpenCV format
image = convert_to_opencv(image)
```

### <a name="deal-with-images-with-a-dimension-1600"></a>處理維度 >1600 的影像

```Python
# If the image has either w or h greater than 1600 we resize it down respecting
# aspect ratio such that the largest dimension is 1600
image = resize_down_to_1600_max_dim(image)
```

### <a name="crop-the-largest-center-square"></a>裁剪中央最大的正方形

```Python
# We next get the largest center square
h, w = image.shape[:2]
min_dim = min(w,h)
max_square_image = crop_center(image, min_dim, min_dim)
```

### <a name="resize-down-to-256x256"></a>將大小往下調整為 256x256

```Python
# Resize that square down to 256x256
augmented_image = resize_to_256_square(max_square_image)
```


### <a name="crop-the-center-for-the-specific-input-size-for-the-model"></a>裁剪模型特定輸入大小的中心

```Python
# The compact models have a network size of 227x227, the model requires this size.
network_input_size = 227

# Crop the center for the specified network_input_Size
augmented_image = crop_center(augmented_image, network_input_size, network_input_size)

```

上述步驟使用下列協助程式函式：

```Python
def convert_to_opencv(image):
    # RGB -> BGR conversion is performed as well.
    r,g,b = np.array(image).T
    opencv_image = np.array([b,g,r]).transpose()
    return opencv_image

def crop_center(img,cropx,cropy):
    h, w = img.shape[:2]
    startx = w//2-(cropx//2)
    starty = h//2-(cropy//2)
    return img[starty:starty+cropy, startx:startx+cropx]

def resize_down_to_1600_max_dim(image):
    h, w = image.shape[:2]
    if (h < 1600 and w < 1600):
        return image

    new_size = (1600 * w // h, 1600) if (h > w) else (1600, 1600 * h // w)
    return cv2.resize(image, new_size, interpolation = cv2.INTER_LINEAR)

def resize_to_256_square(image):
    h, w = image.shape[:2]
    return cv2.resize(image, (256, 256), interpolation = cv2.INTER_LINEAR)

def update_orientation(image):
    exif_orientation_tag = 0x0112
    if hasattr(image, '_getexif'):
        exif = image._getexif()
        if (exif != None and exif_orientation_tag in exif):
            orientation = exif.get(exif_orientation_tag, 1)
            # orientation is 1 based, shift to zero based and flip/transpose based on 0-based values
            orientation -= 1
            if orientation >= 4:
                image = image.transpose(Image.TRANSPOSE)
            if orientation == 2 or orientation == 3 or orientation == 6 or orientation == 7:
                image = image.transpose(Image.FLIP_TOP_BOTTOM)
            if orientation == 1 or orientation == 2 or orientation == 5 or orientation == 6:
                image = image.transpose(Image.FLIP_LEFT_RIGHT)
    return image
```

## <a name="predict-an-image"></a>預測影像

將影像準備為張量之後，可以透過預測模型傳送它：

```Python

# These names are part of the model and cannot be changed.
output_layer = 'loss:0'
input_node = 'Placeholder:0'

with tf.Session() as sess:
    prob_tensor = sess.graph.get_tensor_by_name(output_layer)
    predictions, = sess.run(prob_tensor, {input_node: [augmented_image] })
```

## <a name="view-the-results"></a>View the results

透過模型執行影像張量的結果接著需要對應回標籤。

```Python
    # Print the highest probability label
    highest_probability_index = np.argmax(predictions)
    print('Classified as: ' + labels[highest_probability_index])
    print()

    # Or you can print out all of the results mapping labels to probabilities.
    label_index = 0
    for p in predictions[0]:
        truncated_probablity = np.float64(np.round(p,8))
        print (labels[label_index], truncated_probablity)
        label_index += 1
```
## <a name="next-steps"></a>後續步驟

您也可以將模型包裝到行動應用程式：
* [在 Android 應用程式中使用匯出的 Tensorflow 模型](https://github.com/Azure-Samples/cognitive-services-android-customvision-sample)
* [在 Swift iOS 應用程式中使用匯出的 CoreML 模型](https://go.microsoft.com/fwlink/?linkid=857726)
* [搭配使用 iOS 應用程式中匯出的 CoreML 模型與 Xamarin](https://github.com/xamarin/ios-samples/tree/master/ios11/CoreMLAzureModel)

