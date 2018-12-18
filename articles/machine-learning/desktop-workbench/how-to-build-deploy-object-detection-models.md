---
title: 使用適用於電腦視覺的 Azure Machine Learning 套件來建置及部署物件偵測模型。
description: 了解如何使用適用於電腦視覺的 Azure Machine Learning 套件，來建置、定型、測試及部署電腦視覺物件偵測模型。
services: machine-learning
ms.service: machine-learning
ms.component: core
ms.topic: conceptual
ms.reviewer: jmartens
ms.author: netahw
author: nhaiby
ms.date: 06/01/2018
ROBOTS: NOINDEX
ms.openlocfilehash: 166adc2b464ede5ba77d049075479e3b37a02a88
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46954852"
---
# <a name="build-and-deploy-object-detection-models-with-azure-machine-learning"></a>使用 Azure Machine Learning 來建置和部署物件偵測模型

[!INCLUDE [workbench-deprecated](../../../includes/aml-deprecating-preview-2017.md)]


在本文中，您將了解如何使用「適用於電腦視覺的 Azure Machine Learning 套件」來定型、測試及部署 [Faster R-CNN](https://arxiv.org/abs/1506.01497) 物件偵測模型。 

使用物件偵測可以解決電腦視覺領域中的大量問題。 這些問題包括建置模型來尋找影像上可變數量的物件。 

使用此套件來建置及部署此模型時，您會經歷下列步驟：
1.  資料集建立
2.  深度類神經網路 (DNN) 模型定義
3.  模型定型
4.  評估和視覺效果
5.  Web 服務部署
6.  Web 服務負載測試

此範例使用 TensorFlow 作為深度學習架構，並在以 GPU 加強的電腦上 (例如[深度學習資料科學 VM](https://azuremarketplace.microsoft.com/marketplace/apps/microsoft-ads.dsvm-deep-learning?tab=Overview)) 於本機執行定型，而部署則使用 Azure ML 作業化 CLI。

如需每個模組和類別的詳細參考資訊，請參閱[套件參考文件](https://aka.ms/aml-packages/vision) \(英文\)。

## <a name="prerequisites"></a>必要條件

1. 如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

1. 您必須設定並安裝下列帳戶和應用程式：
   - Azure Machine Learning 測試帳戶 
   - Azure Machine Learning 模型管理帳戶
   - 已安裝 Azure Machine Learning Workbench

   如果還未建立或安裝這三者，請依照 [Azure Machine Learning 快速入門和 Workbench 安裝](../desktop-workbench/quickstart-installation.md)一文來進行。 

1. 您必須安裝適用於電腦視覺的 Azure Machine Learning 套件。 在此了解如何[安裝此套件](https://aka.ms/aml-packages/vision) \(英文\)。

## <a name="sample-data-and-notebook"></a>範例資料和 Notebook

### <a name="get-the-jupyter-notebook"></a>取得 Jupyter Notebook

下載 Notebook 以自行執行此處所述的範例。

> [!div class="nextstepaction"]
> [取得 Jupyter Notebook](https://aka.ms/aml-packages/vision/notebooks/object_detection) \(英文\)

### <a name="load-the-sample-data"></a>載入範例資料

針對此示範，有提供一個冰箱內雜貨項目的資料集，由 30 個影像和 8 個類別所組成 (eggBox、joghurt、ketchup、mushroom、mustard、orange、squash 及 water)。 每個 jpg 影像都有一個名稱相似的註釋 xml 檔案。 

下圖顯示建議的資料夾結構。 

![資料夾結構](media/how-to-build-deploy-object-detection-models/data_directory.JPG)

## <a name="image-annotation"></a>影像註釋

必須提供已標註物件位置，才能進行物件偵測器定型和評估。 [LabelImg](https://tzutalin.github.io/labelImg) 是一個可用來標註影像的開放原始碼註解工具。 LabelImg 會以 Pascal-VOC 格式為每個影像撰寫一個此套件可讀取的 xml 檔案。 


```python
import warnings
warnings.filterwarnings("ignore")
import os, time
from cvtk.core import Context, ObjectDetectionDataset, TFFasterRCNN
from cvtk.evaluation import DetectionEvaluation
from cvtk.evaluation.evaluation_utils import graph_error_counts
from cvtk.utils import detection_utils

# Disable printing of logging messages
from azuremltkbase.logging import ToolkitLogger
ToolkitLogger.getInstance().setEnabled(False)

from matplotlib import pyplot as plt
# Display the images
%matplotlib inline
```

## <a name="create-a-dataset"></a>建立資料集

建立一個由一組影像及其個別週框方塊註釋組成的 CVTK 資料集。 在此範例中，會使用 "../sample_data/foods/training" 資料夾中所提供的冰箱影像。 僅支援 JPEG 影像。


```python
image_folder = "detection/sample_data/foods/train"
data_train = ObjectDetectionDataset.create_from_dir(dataset_name='training_dataset', data_dir=image_folder,
                                                    annotations_dir="Annotations", image_subdirectory='JPEGImages')

# Show some statistics of the training image, and also give one example of the ground truth rectangle annotations
data_train.print_info()
_ = data_train.images[2].visualize_bounding_boxes(image_size = (10,10))
```

    F1 2018-05-25 23:12:21,727 INFO azureml.vision:machine info {"is_dsvm": true, "os_type": "Windows"} 
    F1 2018-05-25 23:12:21,733 INFO azureml.vision:dataset creating dataset for scenario=detection 
    Dataset name: training_dataset
    Total classes: 8, total images: 25
    Label-wise object counts:
        Label eggBox: 20 objects
        Label joghurt: 20 objects
        Label ketchup: 20 objects
        Label mushroom: 20 objects
        Label mustard: 20 objects
        Label orange: 20 objects
        Label squash: 40 objects
        Label water: 20 objects
    Bounding box width and height distribution:
        Bounding box widths  0/5/25/50/75/95/100-percentile: 54/61/79/117/133/165/311 pixels
        Bounding box heights 0/5/25/50/75/95/100-percentile: 48/58/75/124/142/170/212 pixels
    


![png](media/how-to-build-deploy-object-detection-models/output_6_1.JPG)


## <a name="define-a-model"></a>定義模型

在此範例中，會使用 Faster R-CNN 模型。 定義此模型時，可以使用各種參數。 在 CVTK 的 API 文件中或 [Tensorflow 物件偵測網站](https://github.com/tensorflow/models/tree/master/research/object_detection) \(英文\) 上，會說明這些參數及用來進行定型 (請參閱下一節) 的參數。 如需有關 Faster R-CNN 模型的詳細資訊，請參閱[這個連結](https://docs.microsoft.com/cognitive-toolkit/Object-Detection-using-Faster-R-CNN#technical-details) \(英文\)。 此模型以 Fast R-CNN 為基礎，如需其相關詳細資訊，請參閱[這裡](https://docs.microsoft.com/cognitive-toolkit/Object-Detection-using-Fast-R-CNN#algorithm-details) \(英文\)。


```python
score_threshold = 0.0       # Threshold on the detection score, use to discard lower-confidence detections.
max_total_detections = 300  # Maximum number of detections. A high value slows down training but might increase accuracy.
my_detector = TFFasterRCNN(labels=data_train.labels, 
                           score_threshold=score_threshold, 
                           max_total_detections=max_total_detections)
```

## <a name="train-the-model"></a>訓練模型

搭配 ResNet50 的 COCO 定型 Faster R-CNN 模型會用來作為定型的起點。 

為了定型偵測器，程式碼中的定型步驟數目會設定為 350，以便讓定型以更快的速度執行 (使用 GPU 時可達 5 分鐘)。 實務上，請至少將它設定為定型集內影像數目的 10 倍。

在此範例中，偵測器定型步驟數目是設定為 350 來快速定型。 不過，實務上，依據良好的經驗法則，應將步驟數設定為定型集內影像數目的 10 倍以上。

兩個重要的定型參數為：
- 用以進行模型定型的步驟數目，以 num_seps 引數代表。 每個步驟會以一個批次大小為 1 的迷你批次進行模型定型。
- 學習速度，可由 initial_learning_rate 來設定

```python
print("tensorboard --logdir={}".format(my_detector.train_dir))

# to get good results, use a larger value for num_steps, e.g., 5000.
num_steps = 350
learning_rate = 0.001 # learning rate

start_train = time.time()
my_detector.train(dataset=data_train, num_steps=num_steps, 
                  initial_learning_rate=learning_rate)
end_train = time.time()
print(end_train-start_train)
```

    tensorboard --logdir=C:\Users\lixun\Desktop\AutoDL\CVTK\Src\API\cvtk_output\temp_faster_rcnn_resnet50\models\train
    F1 2018-05-25 23:12:22,764 INFO azureml.vision:Fit starting in experiment  1125722225 
    F1 2018-05-25 23:12:22,767 INFO azureml.vision:model starting trainging for scenario=detection 
    Using existing checkpoint file that's saved at 'C:\Users\lixun\Desktop\AutoDL\CVTK\Src\API\cvtk_output\models\detection\faster_rcnn_resnet50_coco_2018_01_28\model.ckpt.index'.
    TFRecords creation started.
    F1 2018-05-25 23:12:22,773 INFO On image 0 of 25
    TFRecords creation completed.
    Training started.
    Training progressing: step 0 ...
    Training progressing: step 100 ...
    Training progressing: step 200 ...
    Training progressing: step 300 ...
    F1 2018-05-25 23:18:02,730 INFO Graph Rewriter optimizations enabled
    Converted 275 variables to const ops.
    F1 2018-05-25 23:18:10,722 INFO 2953 ops in the final graph.
    F1 2018-05-25 23:18:24,244 INFO azureml.vision:Fit finished in experiment  1125722225 
    Training completed.
    361.604615688324
    

TensorBoard 可以用來將定型進度視覺化。 TensorBoard 事件位於模型物件的 train_dir 屬性所指定的資料夾中。 若要檢視 TensorBoard，請依照下列步驟進行操作：
1. 將開頭為 'tensorboard --logdir' 的列印成品複製到命令列並執行。 
2. 將傳回的 URL 從命令列複製到網頁瀏覽器以檢視 TensorBoard。 

TensorBoard 應該看起來如以下螢幕擷取畫面。 填入定型資料夾需要一些時間。 因此，如果 TensorBoard 一開始未正確顯示，請嘗試重複步驟 1-2。  

![tensorboard](media/how-to-build-deploy-object-detection-models/tensorboard.JPG)

## <a name="evaluate-the-model"></a>評估模型

'evaluate' 方法可用來評估模型。 此函式需要以 ObjectDetectionDataset 物件作為輸入。 您可以使用與用於定型資料集相同的函式來建立評估資料集。 支援的計量是如針對 [PASCAL VOC 挑戰](http://host.robots.ox.ac.uk/pascal/VOC/pubs/everingham10.pdf)定義的「平均精確度」。  


```python
image_folder = "detection/sample_data/foods/test"
data_val = ObjectDetectionDataset.create_from_dir(dataset_name='val_dataset', data_dir=image_folder)
eval_result = my_detector.evaluate(dataset=data_val)
```

    F1 2018-05-25 23:18:24,253 INFO azureml.vision:dataset creating dataset for scenario=detection 
    F1 2018-05-25 23:18:24,286 INFO On image 0 of 5
    F1 2018-05-25 23:18:29,300 INFO Starting evaluation at 2018-05-26-03:18:29
    F1 2018-05-25 23:18:32,403 INFO Creating detection visualizations.
    F1 2018-05-25 23:18:33,158 INFO Detection visualizations written to summary with tag image-0.
    F1 2018-05-25 23:18:33,518 INFO Creating detection visualizations.
    F1 2018-05-25 23:18:34,342 INFO Detection visualizations written to summary with tag image-1.
    F1 2018-05-25 23:18:34,714 INFO Creating detection visualizations.
    F1 2018-05-25 23:18:35,470 INFO Detection visualizations written to summary with tag image-2.
    F1 2018-05-25 23:18:35,835 INFO Creating detection visualizations.
    F1 2018-05-25 23:18:36,654 INFO Detection visualizations written to summary with tag image-3.
    F1 2018-05-25 23:18:37,010 INFO Creating detection visualizations.
    F1 2018-05-25 23:18:37,798 INFO Detection visualizations written to summary with tag image-4.
    F1 2018-05-25 23:18:37,804 INFO Running eval batches done.
    F1 2018-05-25 23:18:37,805 INFO # success: 5
    F1 2018-05-25 23:18:37,806 INFO # skipped: 0
    F1 2018-05-25 23:18:38,119 INFO Writing metrics to tf summary.
    F1 2018-05-25 23:18:38,121 INFO PASCAL/PerformanceByCategory/AP@0.5IOU/eggBox: 1.000000
    F1 2018-05-25 23:18:38,205 INFO PASCAL/PerformanceByCategory/AP@0.5IOU/joghurt: 0.942857
    F1 2018-05-25 23:18:38,206 INFO PASCAL/PerformanceByCategory/AP@0.5IOU/ketchup: 1.000000
    F1 2018-05-25 23:18:38,207 INFO PASCAL/PerformanceByCategory/AP@0.5IOU/mushroom: 1.000000
    F1 2018-05-25 23:18:38,208 INFO PASCAL/PerformanceByCategory/AP@0.5IOU/mustard: 1.000000
    F1 2018-05-25 23:18:38,209 INFO PASCAL/PerformanceByCategory/AP@0.5IOU/orange: 1.000000
    F1 2018-05-25 23:18:38,210 INFO PASCAL/PerformanceByCategory/AP@0.5IOU/squash: 1.000000
    F1 2018-05-25 23:18:38,211 INFO PASCAL/PerformanceByCategory/AP@0.5IOU/water: 1.000000
    F1 2018-05-25 23:18:38,211 INFO PASCAL/Precision/mAP@0.5IOU: 0.992857
    F1 2018-05-25 23:18:38,253 INFO Metrics written to tf summary.
    F1 2018-05-25 23:18:38,254 INFO Finished evaluation!
    

評估結果可以用全新格式列印出來。


```python
# print out the performance metric values
for label_obj in data_train.labels:
    label = label_obj.name
    key = 'PASCAL/PerformanceByCategory/AP@0.5IOU/' + label
    print('{0: <15}: {1: <3}'.format(label, round(eval_result[key], 2)))
print('{0: <15}: {1: <3}'.format("overall:", round(eval_result['PASCAL/Precision/mAP@0.5IOU'], 2))) 
```

    joghurt        : 0.94
    squash         : 1.0
    mushroom       : 1.0
    eggBox         : 1.0
    ketchup        : 1.0
    mustard        : 1.0
    water          : 1.0
    orange         : 1.0
    overall:       : 0.99
    

同樣地，您也可以在定型集上計算出模型的精確度。 這麼做有助於確保將定型聚合成一個優良的解決方案。 成功定型後的定型集精確度通常會接近 100%。

您也可以從 TensorBoard 檢視評估結果，包括 mAP 值及帶有預測週框方塊的影像。 請將來自下列程式碼的列印成品複製到命令列視窗中以啟動 TensorBoard 用戶端。 此處使用 8008 連接埠值來避免與預設值 6006 (用於檢視定型狀態) 發生衝突。


```python
print("tensorboard --logdir={} --port=8008".format(my_detector.eval_dir))
```

    tensorboard --logdir=C:\Users\lixun\Desktop\AutoDL\CVTK\Src\API\cvtk_output\temp_faster_rcnn_resnet50\models\eval --port=8008
    

## <a name="score-an-image"></a>為影像評分

在您對定型模型的表現感到滿意之後，便可使用模型物件的 'score' 函式來為新影像評分。 傳回的分數可以藉由 'visualize' 函式來視覺化。 


```python
image_path = data_val.images[1].storage_path
detections_dict = my_detector.score(image_path)
path_save = "./scored_images/scored_image_preloaded.jpg"
ax = detection_utils.visualize(image_path, detections_dict, image_size=(8, 12))
path_save_dir = os.path.dirname(os.path.abspath(path_save))
os.makedirs(path_save_dir, exist_ok=True)
ax.get_figure().savefig(path_save)
```

![png](media/how-to-build-deploy-object-detection-models/output_20_0.JPG)

##  <a name="save-the-model"></a>儲存模型

定型模型可以儲存至磁碟，然後載入回記憶體中，如以下程式碼範例所示。


```python
save_model_path = "./frozen_model/faster_rcnn.model"
my_detector.save(save_model_path)
```

    F1 2018-05-25 23:18:55,166 INFO Graph Rewriter optimizations enabled
    Converted 275 variables to const ops.
    F1 2018-05-25 23:19:03,867 INFO 2953 ops in the final graph.
    

## <a name="load-the-saved-model-for-scoring"></a>載入已儲存的模型來進行評分

若要使用已儲存的模型，請使用 'load' 函式將該模型載入到記憶體中。 您只需要載入一次。 

```python
my_detector_loaded = TFFasterRCNN.load(save_model_path)
```

載入模型之後，即可使用它來為影像或影像清單評分。 針對單一影像，會傳回一個帶有索引鍵 (例如 'detection_boxes'、'detection_scores' 及 'num_detections') 的字典。 如果輸入是影像清單，則會傳回字典清單，其中一個字典會定應一個影像。 


```python
detections_dict = my_detector_loaded.score(image_path)
```

您可以將所偵測到分數高於 0.5 的物件 (包括標籤、分數及座標) 列印出來。


```python
look_up = dict((v,k) for k,v in my_detector.class_map.items())
n_obj = 0
for i in range(detections_dict['num_detections']):
    if detections_dict['detection_scores'][i] > 0.5:
        n_obj += 1
        print("Object {}: label={:11}, score={:.2f}, location=(top: {:.2f}, left: {:.2f}, bottom: {:.2f}, right: {:.2f})".format(
            i, look_up[detections_dict['detection_classes'][i]], 
            detections_dict['detection_scores'][i], 
            detections_dict['detection_boxes'][i][0],
            detections_dict['detection_boxes'][i][1], 
            detections_dict['detection_boxes'][i][2],
            detections_dict['detection_boxes'][i][3]))    
        
print("\nFound {} objects in image {}.".format(n_obj, image_path))           
```

    Object 0: label=squash     , score=0.99, location=(top: 0.74, left: 0.30, bottom: 0.84, right: 0.42)
    Object 1: label=squash     , score=0.98, location=(top: 0.27, left: 0.21, bottom: 0.37, right: 0.33)
    Object 2: label=orange     , score=0.98, location=(top: 0.31, left: 0.39, bottom: 0.37, right: 0.48)
    Object 3: label=joghurt    , score=0.98, location=(top: 0.57, left: 0.29, bottom: 0.67, right: 0.39)
    Object 4: label=eggBox     , score=0.97, location=(top: 0.41, left: 0.53, bottom: 0.49, right: 0.69)
    Object 5: label=water      , score=0.95, location=(top: 0.23, left: 0.51, bottom: 0.37, right: 0.57)
    Object 6: label=mustard    , score=0.88, location=(top: 0.61, left: 0.47, bottom: 0.66, right: 0.57)
    Object 7: label=ketchup    , score=0.80, location=(top: 0.62, left: 0.62, bottom: 0.68, right: 0.72)
    
    Found 8 objects in image ../sample_data/foods/test\JPEGImages\10.jpg.
    

以和先前相同的方式將分數視覺化。


```python
path_save = "./scored_images/scored_image_frozen_graph.jpg"
ax = detection_utils.visualize(image_path, detections_dict, path_save=path_save, image_size=(8, 12))
# ax.get_figure() # use this code extract the returned image
```

![png](media/how-to-build-deploy-object-detection-models/output_30_0.JPG)

## <a name="operationalization-deploy-and-consume"></a>作業化：部署和取用

作業化是將模型和程式碼當作 Web 服務發行，並取用這些服務以產生商業結果的程序。 

一旦模型定型之後，您就可以將該模型部署為 Web 服務，並使用 [Azure Machine Learning CLI](https://docs.microsoft.com/azure/machine-learning/desktop-workbench/cli-for-azure-machine-learning) 來取用。 您的模型可以部署到您的本機電腦或 Azure 容器服務 (ACS) 叢集。 您可以使用 ACS，手動調整您的 Web 服務規模，或使用自動調整規模功能。

**使用 Azure CLI 登入**

以 [Azure](https://azure.microsoft.com/) 帳戶搭配有效的訂用帳戶，使用下列 CLI 命令登入：
<br>`az login`

+ 若要切換到另一個 Azure 訂用帳戶，請使用此命令：
<br>`az account set --subscription [your subscription name]`

+ 若要查看目前的模型管理帳戶，請使用此命令：
  <br>`az ml account modelmanagement show`

**建立並設定您的叢集部署環境**

您只需要設定部署環境一次。 如果您還沒有部署環境，請立即使用[這些指示](https://docs.microsoft.com/azure/machine-learning/desktop-workbench/deployment-setup-configuration#environment-setup)來設定部署環境。 

若要查看作用中的部署環境，請使用下列 CLI 命令：
<br>`az ml env show`
   
建立和設定部署環境的範例 Azure CLI 命令

```CLI
az provider register -n Microsoft.MachineLearningCompute
az provider register -n Microsoft.ContainerRegistry
az provider register -n Microsoft.ContainerService
az ml env setup --cluster -n [your environment name] -l [Azure region e.g. westcentralus] [-g [resource group]]
az ml env set -n [environment name] -g [resource group]
az ml env cluster
```
    
### <a name="manage-web-services-and-deployments"></a>管理 Web 服務和部署

下列 API 可用於將模型部署為 Web 服務、管理那些 Web 服務，以及管理部署。

|Task|API|
|----|----|
|建立部署物件|`deploy_obj = AMLDeployment(deployment_name=deployment_name, associated_DNNModel=dnn_model, aml_env="cluster")`
|部署 Web 服務|`deploy_obj.deploy()`|
|為影像計分|`deploy_obj.score_image(local_image_path_or_image_url)`|
|刪除 Web 服務|`deploy_obj.delete()`|
|建置不含 Web 服務的 Docker 影像|`deploy_obj.build_docker_image()`|
|列出現有的部署|`AMLDeployment.list_deployment()`|
|如果有具有部署名稱的服務存在則刪除|`AMLDeployment.delete_if_service_exist(deployment_name)`|

**API 文件**：如需每個模組和類別的詳細參考資訊，請參閱[套件參考文件](https://aka.ms/aml-packages/vision)。

**CLI 參考**：如需與部署相關的其他進階作業，請參閱[模型管理命令列介面參考](https://docs.microsoft.com/azure/machine-learning/desktop-workbench/model-management-cli-reference)。

**Azure 入口網站中的部署管理**：您可以在 [Azure 入口網站](https://ms.portal.azure.com/)中追蹤和管理您的部署。 從 Azure 入口網站中，使用其名稱尋找 [Machine Learning 模型管理帳戶] 頁面。 接著，移至 [模型管理帳戶] 頁面 > [模型管理] > [服務]。

```python
# ##### OPTIONAL - Interactive CLI setup helper ###### 
# # Interactive CLI setup helper, including model management account and deployment environment.
# # If you haven't setup you CLI before or if you want to change you CLI settings, you can use this block to help you interactively.
# # UNCOMMENT THE FOLLOWING LINES IF YOU HAVE NOT CREATED OR SET THE MODEL MANAGEMENT ACCOUNT AND DEPLOYMENT ENVIRONMENT

# from azuremltkbase.deployment import CliSetup
# CliSetup().run()
```


```python
from cvtk.operationalization import AMLDeployment

# set deployment name
deployment_name = "wsdeployment"

# Create deployment object
# It will use the current deployment environment (you can check it with CLI command "az ml env show").
deploy_obj = AMLDeployment(deployment_name=deployment_name, aml_env="cluster", associated_DNNModel=my_detector, replicas=1)

# Alternatively, you can provide azure machine learning deployment cluster name (environment name) and resource group name
# to deploy your model. It will use the provided cluster to deploy. To do that, please uncomment the following lines to create 
# the deployment object.

# azureml_rscgroup = "<resource group>"
# cluster_name = "<cluster name>"
# deploy_obj = AMLDeployment(deployment_name=deployment_name, associated_DNNModel=my_detector,
#                            aml_env="cluster", cluster_name=cluster_name, resource_group=azureml_rscgroup, replicas=1)

# Check if the deployment name exists, if yes remove it first.
if deploy_obj.is_existing_service():
    AMLDeployment.delete_if_service_exist(deployment_name)
    
# create the webservice
print("Deploying to Azure cluster...")
deploy_obj.deploy()
print("Deployment DONE")
```

### <a name="consume-the-web-service"></a>取用 Web 服務

在您建立 Web 服務之後，即可使用所部署的 Web 服務來為影像評分。 您有幾種選項：

   - 您可以藉由下列方式使用部署物件來直接為 Web 服務評分：deploy_obj.score_image(image_path_or_url) 
   - 或者，您也可以藉由下列方式使用服務端點 URL 和服務索引鍵 (如果是本機部署，則為 None)：AMLDeployment.score_existing_service_with_image(image_path_or_url, service_endpoint_url, service_key=None)
   - 直接形成 HTTP 要求來為 Web 服務端點評分 (適用於進階使用者)。

### <a name="score-with-existing-deployment-object"></a>使用現有的部署物件計分
```
deploy_obj.score_image(image_path_or_url)
```


```python
# Score with existing deployment object

# Score local image with file path
print("Score local image with file path")
image_path_or_url = data_train.images[0].storage_path
print("Image source:",image_path_or_url)
serialized_result_in_json = deploy_obj.score_image(image_path_or_url, image_resize_dims=[224,224])
print("serialized_result_in_json:", serialized_result_in_json[:50])

# Score image url and remove image resizing
print("Score image url")
image_path_or_url = "https://cvtkdata.blob.core.windows.net/publicimages/microsoft_logo.jpg"
print("Image source:",image_path_or_url)
serialized_result_in_json = deploy_obj.score_image(image_path_or_url)
print("serialized_result_in_json:", serialized_result_in_json[:50])

```


```python
# Time image scoring
import timeit

num_images = 3
for img_index, img_obj in enumerate(data_train.images[:num_images]):
    print("Calling API for image {} of {}: {}...".format(img_index, num_images, img_obj.name))
    tic = timeit.default_timer()
    return_json = deploy_obj.score_image(img_obj.storage_path, image_resize_dims=[224,224])
    print("   Time for API call: {:.2f} seconds".format(timeit.default_timer() - tic))
```

### <a name="score-with-service-endpoint-url-and-service-key"></a>使用服務端點 URL 和服務金鑰計分
```
    AMLDeployment.score_existing_service_with_image(image_path_or_url, service_endpoint_url, service_key=None)
```


```python
# Import related classes and functions
from cvtk.operationalization import AMLDeployment

service_endpoint_url = "http://xxx" # please replace with your own service url
service_key = "xxx" # please replace with your own service key

# score image url
image_path_or_url = "https://cvtkdata.blob.core.windows.net/publicimages/microsoft_logo.jpg"
print("Image source:",image_path_or_url)
serialized_result_in_json = AMLDeployment.score_existing_service_with_image(image_path_or_url,service_endpoint_url, service_key = service_key, image_resize_dims=[224,224])
print("serialized_result_in_json:", serialized_result_in_json[:50])
```

### <a name="score-endpoint-with-http-request-directly"></a>直接使用 http 要求為端點計分
以下是一些直接以 Python 形成 HTTP 要求的範例程式碼。 您也可以使用其他程式設計語言來進行此操作。


```python
def score_image_with_http(image, service_endpoint_url, service_key=None, parameters={}):
    """Score local image with http request

    Args:
        image (str): Image file path
        service_endpoint_url(str): web service endpoint url
        service_key(str): Service key. None for local deployment.
        parameters (dict): Additional request paramters in dictionary. Default is {}.


    Returns:
        str: serialized result 
    """
    import requests
    from io import BytesIO
    import base64
    import json

    if service_key is None:
        headers = {'Content-Type': 'application/json'}
    else:
        headers = {'Content-Type': 'application/json',
                   "Authorization": ('Bearer ' + service_key)}
    payload = []
    encoded = None
    
    # Read image
    with open(image,'rb') as f:
        image_buffer = BytesIO(f.read()) ## Getting an image file represented as a BytesIO object
        
    # Convert your image to base64 string
    # image_in_base64 : "b'{base64}'"
    encoded = base64.b64encode(image_buffer.getvalue())
    image_request = {"image_in_base64": "{0}".format(encoded), "parameters": parameters}
    payload.append(image_request)
    body = json.dumps(payload)
    r = requests.post(service_endpoint_url, data=body, headers=headers)
    try:
        result = json.loads(r.text)
        json.loads(result[0])
    except:
        raise ValueError("Incorrect output format. Result cant not be parsed: " + r.text)
    return result[0]

```

### <a name="parse-serialized-result-from-webservice"></a>從 Web 服務剖析序列化的結果
來自 Web 服務的結果會以可剖析的 JSON 字串表示。


```python
image_path_or_url = image_path
print("Image source:",image_path_or_url)
serialized_result_in_json = deploy_obj.score_image(image_path_or_url)
print("serialized_result_in_json:", serialized_result_in_json[:50])
```


```python
# Parse result from json string
import numpy as np
parsed_result = TFFasterRCNN.parse_serialized_result(serialized_result_in_json)
print("Parsed result:", parsed_result)
```


```python
ax = detection_utils.visualize(image_path, parsed_result)
path_save = "./scored_images/scored_image_web.jpg"
path_save_dir = os.path.dirname(os.path.abspath(path_save))
os.makedirs(path_save_dir, exist_ok=True)
ax.get_figure().savefig(path_save)
```

## <a name="next-steps"></a>後續步驟

在這些文章中，深入了解適用於電腦視覺的 Azure Machine Learning 套件：

+ 參閱[套件概觀](https://aka.ms/aml-packages/vision) \(英文\)。

+ 瀏覽此套件的[參考文件](https://docs.microsoft.com/python/api/overview/azure-machine-learning/computer-vision)。

+ 了解[適用於 Azure Machine Learning 的其他 Python 套件](reference-python-package-overview.md)。
