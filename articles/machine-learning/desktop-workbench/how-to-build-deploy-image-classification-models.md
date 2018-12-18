---
title: 使用適用於電腦視覺的 Azure Machine Learning 套件，建置及部署影像分類模型。
description: 了解如何使用適用於電腦視覺的 Azure Machine Learning 套件，建置、定型、測試和部署電腦視覺影像分類模型。
services: machine-learning
ms.service: machine-learning
ms.component: core
ms.topic: conceptual
ms.reviewer: jmartens
ms.author: netahw
author: nhaiby
ms.date: 04/23/2018
ROBOTS: NOINDEX
ms.openlocfilehash: 1dfcda5004dcf30c6dc112fb8150180849383016
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46971564"
---
# <a name="build-and-deploy-image-classification-models-with-azure-machine-learning"></a>使用 Azure Machine Learning 建置及部署影像分類模型

[!INCLUDE [workbench-deprecated](../../../includes/aml-deprecating-preview-2017.md)]

在此文章中，您將了解如何使用「適用於電腦視覺的 Azure Machine Learning 套件」(AMLPCV) 來定型、測試及部署影像分類模型。 如需此套件的概觀與其詳細參考文件，請參閱[此文件](https://aka.ms/aml-packages/vision) \(英文\)。

使用影像分類可以解決電腦視覺領域中的大量問題。 這些問題包括建置可回答問題的模型，例如：
+ _物件存在於影像中嗎？例如，「狗」、「車子」、「船」等等_
+ _此病患的視網膜掃描顯示的眼睛疾病嚴重性等級為何？_

使用 AMLPCV 建置及部署此模型時，您會經歷下列步驟：
1. 資料集建立
2. 影像視覺效果和註釋
3. 影像增強指定
4. 深度類神經網路 (DNN) 模型定義
5. 分類器定型
6. 評估和視覺效果
7. Web 服務部署
8. Web 服務負載測試

[CNTK](https://www.microsoft.com/cognitive-toolkit/) 當作深入學習架構使用，定型則會在採用 GPU 機器的本機上執行，例如 ([Deep learning Data Science VM](https://azuremarketplace.microsoft.com/marketplace/apps/microsoft-ads.dsvm-deep-learning?tab=Overview))，而且部署會使用 Azure ML 作業化 CLI。

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
> [取得 Jupyter Notebook](https://aka.ms/aml-packages/vision/notebooks/image_classification) \(英文\)

### <a name="load-the-sample-data"></a>載入範例資料

下列範例使用包含 63 個餐具影像的資料集。 每個影像都有標示屬於哪一個類別 (bowl、cup、cutlery、plate)。 此範例中的影像數目不多，因此可以快速執行此範例。 在實務上，每個類別應該至少提供 100 個影像。 所有影像都位於 *"../sample_data/imgs_recycling/"*，稱為 "bowl"、"cup"、"cutlery" 和 "plate" 的子目錄中。

![Azure Machine Learning 資料集](media/how-to-build-deploy-image-classification-models/recycling_examples.jpg)


```python
import warnings
warnings.filterwarnings("ignore")
import json, numpy as np, os, timeit 
from azureml.logging import get_azureml_logger
from imgaug import augmenters
from IPython.display import display
from sklearn import svm
from cvtk import ClassificationDataset, CNTKTLModel, Context, Splitter, StorageContext
from cvtk.augmentation import augment_dataset
from cvtk.core.classifier import ScikitClassifier
from cvtk.evaluation import ClassificationEvaluation, graph_roc_curve, graph_pr_curve, graph_confusion_matrix
import matplotlib.pyplot as plt

from classification.notebook.ui_utils.ui_annotation import AnnotationUI
from classification.notebook.ui_utils.ui_results_viewer import ResultsUI
from classification.notebook.ui_utils.ui_precision_recall import PrecisionRecallUI

%matplotlib inline

# Disable printing of logging messages
from azuremltkbase.logging import ToolkitLogger
ToolkitLogger.getInstance().setEnabled(False)
```



## <a name="create-a-dataset"></a>建立資料集

一旦您匯入相依性並設定儲存體內容之後，就可以建立資料集物件。

若要使用適用於電腦視覺的 Azure Machine Learning 套件建立該物件，請提供影像在本機磁碟上的根目錄。 此目錄必須遵循與餐具資料夾相同的一般結構，亦即，包含具有實際影像的子目錄：
- root
    - 標籤 1
    - 標籤 2
    - ...
    - 標籤 n
  
針對不同的資料集為影像分類模型定型如同將下列程式碼中的根路徑 `dataset_location` 變更為指向不同影像般容易。


```python
# Root image directory
dataset_location = os.path.abspath("classification/sample_data/imgs_recycling")

dataset_name = 'recycling'
dataset = ClassificationDataset.create_from_dir(dataset_name, dataset_location)
print("Dataset consists of {} images with {} labels.".format(len(dataset.images), len(dataset.labels)))
print("Select information for image 2: name={}, label={}, unique id={}.".format(
    dataset.images[2].name, dataset.images[2]._labels[0].name, dataset.images[2]._storage_id))
```

    F1 2018-04-23 17:12:57,593 INFO azureml.vision:machine info {"is_dsvm": true, "os_type": "Windows"} 
    F1 2018-04-23 17:12:57,599 INFO azureml.vision:dataset creating dataset for scenario=classification 
    Dataset consists of 63 images with 4 labels.
    Select information for image 2: name=msft-plastic-bowl20170725152154282.jpg, label=bowl, unique id=3.

資料集物件提供使用 [Bing 影像搜尋 API](https://azure.microsoft.com/services/cognitive-services/bing-image-search-api/) 下載影像的功能。 

系統支援下列兩種類型的搜尋查詢： 
+ 一般文字查詢
+ 影像 URL 查詢

在 JSON 編碼的文字檔案內必須提供這些查詢以及類別標籤。 例如︰

```json
{
    "bowl": [
                    "plastic bowl",
                    "../imgs_recycling/bowl"
    ],
    "cup": [
                    "plastic cup",
                    "../imgs_recycling/cup",
                    "http://cdnimg2.webstaurantstore.com/images/products/main/123662/268841/dart-solo-ultra-clear-conex-tp12-12-oz-pet-plastic-cold-cup-1000-case.jpg"
    ],
    "cutlery": [
                    "plastic cutlery",
                    "../imgs_recycling/cutlery",
                    "http://img4.foodservicewarehouse.com/Prd/1900SQ/Fineline_2514-BO.jpg"
    ],
    "plate": [
                    "plastic plate",
                    "../imgs_recycling/plate"
    ]
}
```

此外，您必須明確地建立內容物件以包含 Bing 影像搜尋 API 金鑰。 這需要 Bing 影像搜尋 API 訂用帳戶。

## <a name="visualize-and-annotate-images"></a>將影像視覺化並加上註釋

您可以使用下列 Widget 將影像視覺化，並更正資料集物件中的標籤。 

如果您遇到「未偵測到 Widget Javascript」錯誤，請執行此命令來解決它：
<br>`jupyter nbextension enable --py --sys-prefix widgetsnbextension`


```python
annotation_ui = AnnotationUI(dataset, Context.get_global_context())
display(annotation_ui.ui)
```

![Azure Machine Learning 資料集](media/how-to-build-deploy-image-classification-models/image_annotation.png)

## <a name="augment-images"></a>增強影像

[`augmentation`模組](https://docs.microsoft.com/python/api/cvtk.augmentation)提供使用 [imgaug](https://github.com/aleju/imgaug) 程式庫中所述所有轉換增強資料集物件的功能。 影像轉換可以分組到單一管線中，在此情況下，每個影像都會同時套用管線中的所有轉換。 

如果您想要個別套用不同的增強指定步驟，或以不同的方式套用，則可以定義多個管線，並將其傳遞至 *augment_dataset* 函式。 如需影像增強指定的詳細資訊和範例，請參閱 [imgaug 文件](https://github.com/aleju/imgaug) \(英文\)。

將增強型影像新增至定型集對於小型資料集特別有幫助。 由於 DNN 定型程序會因為定型影像數目增加而更慢，因此建議您在不增強指定的情況下開始試驗。


```python
# Split the dataset into train and test  
train_set_orig, test_set = dataset.split(train_size = 0.66, stratify = "label")
print("Number of training images = {}, test images = {}.".format(train_set_orig.size(), test_set.size()))
```

    F1 2018-04-23 17:13:01,780 INFO azureml.vision:splitter splitting a dataset 
    F1 2018-04-23 17:13:01,805 INFO azureml.vision:dataset creating dataset for scenario=classification 
    F1 2018-04-23 17:13:01,809 INFO azureml.vision:dataset creating dataset for scenario=classification 
    Number of training images = 41, test images = 22.
    


```python
augment_train_set = False

if augment_train_set:
    aug_sequence = augmenters.Sequential([
            augmenters.Fliplr(0.5),             # horizontally flip 50% of all images
            augmenters.Crop(percent=(0, 0.1)),  # crop images by 0-10% of their height/width
        ])
    train_set = augment_dataset(train_set_orig, [aug_sequence])
    print("Number of original training images = {}, with augmented images included = {}.".format(train_set_orig.size(), train_set.size()))
else:
    train_set = train_set_orig  
```

## <a name="define-dnn-models"></a>定義 DNN 模型

此套件支援下列預先定型的深度類神經網路模型： 
+ Resnet-18
+ Resnet-34
+ Resnet-50
+ Resnet-101
+ Resnet-152

這些 DNN 可以當作分類器或 Featurizer 使用。 

您可以在[這裡](https://github.com/Microsoft/CNTK/blob/master/PretrainedModels/Image.md)找到關於網路的詳細資訊，並在[這裡](https://blog.slavv.com/a-gentle-intro-to-transfer-learning-2c0b674375a0)找到傳輸學習的基本簡介。

此套件的預設影像分類參數為 224 x 224 像素解析度和 Resnet-18 DNN。 這些參數獲選為適用於各種不同的工作。 精確度通常可以獲得改善，例如，將影像解析度增加到 500 x 500 像素，和/或選取更深入的模型 (Resnet-50)。 不過，變更參數可能會使定型時間大幅提升。 請參閱[如何改善精確度](https://docs.microsoft.com/azure/machine-learning/service/how-to-improve-accuracy-for-computer-vision-models)的相關文章。


```python
# Default parameters (224 x 224 pixels resolution, Resnet-18)
lr_per_mb = [0.05]*7 + [0.005]*7 +  [0.0005]
mb_size = 32
input_resoluton = 224
base_model_name = "ResNet18_ImageNet_CNTK"

# Suggested parameters for 500 x 500 pixels resolution, Resnet-50
# (see in the Appendix "How to improve accuracy", last row in table)
# lr_per_mb   = [0.01] * 7 + [0.001] * 7 + [0.0001]
# mb_size    = 8
# input_resoluton = 500
# base_model_name = "ResNet50_ImageNet_CNTK"

# Initialize model
dnn_model = CNTKTLModel(train_set.labels,
                       base_model_name=base_model_name,
                       image_dims = (3, input_resoluton, input_resoluton))
```

    Successfully downloaded ResNet18_ImageNet_CNTK
    

## <a name="train-the-classifier"></a>為分類器定型

您可以為預先定型的 DNN 選擇下列其中一種方法。

  - **DNN 調整**，這會為 DNN 定型，以便直接執行分類。 當 DNN 定型緩慢時，通常會導致最佳結果，因為在定型期間可以改善所有網路權重來提供最佳精確度。

  - **DNN 功能化**，這會以原狀執行 DNN，以取得影像更低維度的表示法 (512、2048 或 4096 個浮點)。 該表示法接著會當作輸入使用，以便為另一個分類器定型。 由於 DNN 維持不變，因此相較於 DNN 調整，這種方法會快許多，但是精確度就沒那麼好。 不過，為線性 SVM 之類的外部分類器定型 (如下列程式碼所示) 可提供強式基準，並有助於了解問題的可行性。
  
TensorBoard 可以用來將定型進度視覺化。 啟用 TensorBoard：
1. 加入參數 `tensorboard_logdir=PATH`，如以下程式碼所示
1. 在新的主控台中使用命令 `tensorboard --logdir=PATH` 來啟動 TensorBoard 用戶端。
1. 依 TensorBoard 的指示 (預設為 localhost:6006)，開啟網頁瀏覽器。 


```python
# Train either the DNN or a SVM as classifier 
classifier_name = "dnn"

if classifier_name.lower() == "dnn":  
    dnn_model.train(train_set, lr_per_mb = lr_per_mb, mb_size = mb_size, eval_dataset=test_set) #, tensorboard_logdir=r"tensorboard"
    classifier = dnn_model
elif classifier_name.lower() == "svm":
    learner = svm.LinearSVC(C=1.0, class_weight='balanced', verbose=0)
    classifier = ScikitClassifier(dnn_model, learner = learner)
    classifier.train(train_set)
else:
    raise Exception("Classifier unknown: " + classifier)   
```

    F1 2018-04-23 17:13:28,238 INFO azureml.vision:Fit starting in experiment  1541466320 
    F1 2018-04-23 17:13:28,239 INFO azureml.vision:model starting training for scenario=classification 
    <class 'int'>
    1 worker
    Training transfer learning model for 15 epochs (epoch_size = 41).
    non-distributed mode
    Training 15741700 parameters in 53 parameter tensors.
    Training 15741700 parameters in 53 parameter tensors.
    Learning rate per minibatch: 0.05
    Momentum per minibatch: 0.9
    PROGRESS: 0.00%
    Finished Epoch[1 of 15]: [Training] loss = 2.820586 * 41, metric = 68.29% * 41 5.738s (  7.1 samples/s);
    Evaluation Set Error :: 29.27%
    Finished Epoch[2 of 15]: [Training] loss = 0.286728 * 41, metric = 9.76% * 41 0.752s ( 54.5 samples/s);
    Evaluation Set Error :: 34.15%
    Finished Epoch[3 of 15]: [Training] loss = 0.206938 * 41, metric = 4.88% * 41 0.688s ( 59.6 samples/s);
    Evaluation Set Error :: 41.46%
    Finished Epoch[4 of 15]: [Training] loss = 0.098931 * 41, metric = 2.44% * 41 0.785s ( 52.2 samples/s);
    Evaluation Set Error :: 48.78%
    Finished Epoch[5 of 15]: [Training] loss = 0.046547 * 41, metric = 0.00% * 41 0.724s ( 56.6 samples/s);
    Evaluation Set Error :: 43.90%
    Finished Epoch[6 of 15]: [Training] loss = 0.059709 * 41, metric = 4.88% * 41 0.636s ( 64.5 samples/s);
    Evaluation Set Error :: 34.15%
    Finished Epoch[7 of 15]: [Training] loss = 0.005817 * 41, metric = 0.00% * 41 0.710s ( 57.7 samples/s);
    Evaluation Set Error :: 14.63%
    Learning rate per minibatch: 0.005
    Finished Epoch[8 of 15]: [Training] loss = 0.014917 * 41, metric = 0.00% * 41 0.649s ( 63.2 samples/s);
    Evaluation Set Error :: 9.76%
    Finished Epoch[9 of 15]: [Training] loss = 0.040539 * 41, metric = 2.44% * 41 0.777s ( 52.8 samples/s);
    Evaluation Set Error :: 9.76%
    Finished Epoch[10 of 15]: [Training] loss = 0.024606 * 41, metric = 0.00% * 41 0.626s ( 65.5 samples/s);
    Evaluation Set Error :: 7.32%
    PROGRESS: 0.00%
    Finished Epoch[11 of 15]: [Training] loss = 0.004225 * 41, metric = 0.00% * 41 0.656s ( 62.5 samples/s);
    Evaluation Set Error :: 4.88%
    Finished Epoch[12 of 15]: [Training] loss = 0.004364 * 41, metric = 0.00% * 41 0.702s ( 58.4 samples/s);
    Evaluation Set Error :: 4.88%
    Finished Epoch[13 of 15]: [Training] loss = 0.007974 * 41, metric = 0.00% * 41 0.721s ( 56.9 samples/s);
    Evaluation Set Error :: 4.88%
    Finished Epoch[14 of 15]: [Training] loss = 0.000655 * 41, metric = 0.00% * 41 0.711s ( 57.7 samples/s);
    Evaluation Set Error :: 4.88%
    Learning rate per minibatch: 0.0005
    Finished Epoch[15 of 15]: [Training] loss = 0.024865 * 41, metric = 0.00% * 41 0.688s ( 59.6 samples/s);
    Evaluation Set Error :: 4.88%
    Stored trained model at ../../../cvtk_output\model_trained\ImageClassification.model
    F1 2018-04-23 17:13:45,097 INFO azureml.vision:Fit finished in experiment  1541466320 
    


```python
# Plot how the training and test accuracy increases during gradient descent. 
if classifier_name == "dnn":
    [train_accs, test_accs, epoch_numbers] = classifier.train_eval_accs
    plt.xlabel("Number of training epochs") 
    plt.ylabel("Classification accuracy") 
    train_plot = plt.plot(epoch_numbers, train_accs, 'r-', label = "Training accuracy")
    test_plot = plt.plot(epoch_numbers, test_accs, 'b-.', label = "Test accuracy")
    plt.legend()
```

![png](media/how-to-build-deploy-image-classification-models/output_17_0.png)


## <a name="evaluate-and-visualize-model-performance"></a>評估模型效能並將其視覺化

您可以在獨立的測試資料集上，使用評估模組評估定型模型的效能。 它會計算的評估計量包括：
 
+ 精確度 (預設為 class-averaged)
+ PR 曲線
+ ROC 曲線
+ 曲線底下的區域
+ 混淆矩陣


```python
# Run the classifier on all test set images
ce = ClassificationEvaluation(classifier, test_set, minibatch_size = mb_size)

# Compute Accuracy and the confusion matrix
acc = ce.compute_accuracy()
print("Accuracy = {:2.2f}%".format(100*acc))
cm  = ce.compute_confusion_matrix()
print("Confusion matrix = \n{}".format(cm))

# Show PR curve, ROC curve, and confusion matrix
fig, ((ax1, ax2, ax3)) = plt.subplots(1,3)
fig.set_size_inches(18, 4)
graph_roc_curve(ce, ax=ax1)
graph_pr_curve(ce, ax=ax2)
graph_confusion_matrix(ce, ax=ax3)
plt.show()
```

    F1 2018-04-23 17:14:37,449 INFO azureml.vision:evaluation doing evaluation for scenario=classification 
    F1 2018-04-23 17:14:37,450 INFO azureml.vision:model scoring dataset for scenario=classification 
    Accuracy = 95.45%
    Confusion matrix = 
    [[ 0  1  0  1]
     [ 0  7  0  0]
     [ 0  0  2  0]
     [ 0  0  0 11]]
    


![png](media/how-to-build-deploy-image-classification-models/output_20_1.png)



```python
# Results viewer UI
labels = [l.name for l in dataset.labels] 
pred_scores = ce.scores #classification scores for all images and all classes
pred_labels = [labels[i] for i in np.argmax(pred_scores, axis=1)]

results_ui = ResultsUI(test_set, Context.get_global_context(), pred_scores, pred_labels)
display(results_ui.ui)
```

![Azure Machine Learning 資料集](media/how-to-build-deploy-image-classification-models/Image_Classification_Results.png)


```python
# Precision / recall curve UI
precisions, recalls, thresholds = ce.compute_precision_recall_curve() 
thresholds = list(thresholds)
thresholds.append(thresholds[-1])
pr_ui = PrecisionRecallUI(100*precisions[::-1], 100*recalls[::-1], thresholds[::-1])
display(pr_ui.ui) 
```

![Azure Machine Learning 資料集](media/how-to-build-deploy-image-classification-models/image_precision_curve.png)

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
# ##### OPTIONAL###### 
# Interactive CLI setup helper, including model management account and deployment environment.
# If you haven't setup you CLI before or if you want to change you CLI settings, you can use this block to help you interactively.
#
# UNCOMMENT THE FOLLOWING LINES IF YOU HAVE NOT CREATED OR SET THE MODEL MANAGEMENT ACCOUNT AND DEPLOYMENT ENVIRONMENT
#
# from azuremltkbase.deployment import CliSetup
# CliSetup().run()
```


```python
# # Optional. Persist your model on disk and reuse it later for deployment. 
# from cvtk import TFFasterRCNN, Context
# import os
# save_model_path = os.path.join(Context.get_global_context().storage.persistent_path, "saved_classifier.model")
# # Save model to disk
# dnn_model.serialize(save_model_path)
# # Load model from disk
# dnn_model = CNTKTLModel.deserialize(save_model_path)
```


```python
from cvtk.operationalization import AMLDeployment

# set deployment name
deployment_name = "wsdeployment"

# Optional Azure Machine Learning deployment cluster name (environment name) and resource group name
# If you don't provide here. It will use the current deployment environment (you can check with CLI command "az ml env show").
azureml_rscgroup = "<resource group>"
cluster_name = "<cluster name>"

# If you provide the cluster information, it will use the provided cluster to deploy. 
# Example: deploy_obj = AMLDeployment(deployment_name=deployment_name, associated_DNNModel=dnn_model,
#                            aml_env="cluster", cluster_name=cluster_name, resource_group=azureml_rscgroup, replicas=1)

# Create deployment object
deploy_obj = AMLDeployment(deployment_name=deployment_name, aml_env="cluster", associated_DNNModel=dnn_model, replicas=1)

# Check if the deployment name exists, if yes remove it first.
if deploy_obj.is_existing_service():
    AMLDeployment.delete_if_service_exist(deployment_name)
    
# Create the web service
print("Deploying to Azure cluster...")
deploy_obj.deploy()
print("Deployment DONE")
```

### <a name="consume-the-web-service"></a>取用 Web 服務 

一旦您將模型部署為 Web 服務之後，可以使用下列其中一種方法，為包含 Web 服務的影像計分：

- 使用 `deploy_obj.score_image(image_path_or_url)` 搭配部署物件，直接為 Web 服務計分

- 使用服務端點 URL 和服務金鑰 (不適用於本機部署) 搭配：`AMLDeployment.score_existing_service_with_image(image_path_or_url, service_endpoint_url, service_key=None)`

- 直接形成 HTTP 要求，以便為 Web 服務端點計分。 此選項適用於進階使用者。

### <a name="score-with-existing-deployment-object"></a>使用現有的部署物件計分

```
deploy_obj.score_image(image_path_or_url)
```


```python
# Score with existing deployment object

# Score local image with file path
print("Score local image with file path")
image_path_or_url = test_set.images[0].storage_path
print("Image source:",image_path_or_url)
serialized_result_in_json = deploy_obj.score_image(image_path_or_url, image_resize_dims=[224,224])
print("serialized_result_in_json:", serialized_result_in_json)

# Score image url and remove image resizing
print("Score image url")
image_path_or_url = "https://cvtkdata.blob.core.windows.net/publicimages/microsoft_logo.jpg"
print("Image source:",image_path_or_url)
serialized_result_in_json = deploy_obj.score_image(image_path_or_url)
print("serialized_result_in_json:", serialized_result_in_json)

# Score image url with added paramters. Add softmax to score.
print("Score image url with added paramters. Add softmax to score")
from cvtk.utils.constants import ClassificationRESTApiParamters
image_path_or_url = "https://cvtkdata.blob.core.windows.net/publicimages/microsoft_logo.jpg"
print("Image source:",image_path_or_url)
serialized_result_in_json = deploy_obj.score_image(image_path_or_url, image_resize_dims=[224,224], parameters={ClassificationRESTApiParamters.ADD_SOFTMAX:True})
print("serialized_result_in_json:", serialized_result_in_json)
```


```python
# Time image scoring
import timeit

for img_index, img_obj in enumerate(test_set.images[:10]):
    print("Calling API for image {} of {}: {}...".format(img_index, len(test_set.images), img_obj.name))
    tic = timeit.default_timer()
    return_json = deploy_obj.score_image(img_obj.storage_path, image_resize_dims=[224,224])
    print("   Time for API call: {:.2f} seconds".format(timeit.default_timer() - tic))
    print(return_json)
```

### <a name="score-with-service-endpoint-url-and-service-key"></a>使用服務端點 URL 和服務金鑰計分

`AMLDeployment.score_existing_service_with_image(image_path_or_url, service_endpoint_url, service_key=None)`

```python
# Import related classes and functions
from cvtk.operationalization import AMLDeployment

service_endpoint_url = "" # please replace with your own service url
service_key = "" # please replace with your own service key
# score local image with file path
image_path_or_url = test_set.images[0].storage_path
print("Image source:",image_path_or_url)
serialized_result_in_json = AMLDeployment.score_existing_service_with_image(image_path_or_url,service_endpoint_url, service_key = service_key)
print("serialized_result_in_json:", serialized_result_in_json)

# score image url
image_path_or_url = "https://cvtkdata.blob.core.windows.net/publicimages/microsoft_logo.jpg"
print("Image source:",image_path_or_url)
serialized_result_in_json = AMLDeployment.score_existing_service_with_image(image_path_or_url,service_endpoint_url, service_key = service_key, image_resize_dims=[224,224])
print("serialized_result_in_json:", serialized_result_in_json)
```

### <a name="score-endpoint-with-http-request-directly"></a>直接使用 http 要求為端點計分

下列範例程式碼會在 Python 中直接形成 HTTP 要求。 不過，您可以使用其他程式設計語言完成。


```python
def score_image_list_with_http(images, service_endpoint_url, service_key=None, parameters={}):
    """Score image list with http request

    Args:
        images(list): list of (input image file path, base64 image string, url or buffer)
        service_endpoint_url(str): endpoint url
        service_key(str): service key, None for local deployment.
        parameters(dict): service additional paramters in dictionary


    Returns:
        result (list): list of serialized result 
    """
    import requests
    from io import BytesIO
    import base64
    routing_id = ""

    if service_key is None:
        headers = {'Content-Type': 'application/json',
                   'X-Marathon-App-Id': routing_id}
    else:
        headers = {'Content-Type': 'application/json',
                   "Authorization": ('Bearer ' + service_key), 'X-Marathon-App-Id': routing_id}
    payload = []
    for image in images:
        encoded = None
        # read image
        with open(image,'rb') as f:
            image_buffer = BytesIO(f.read()) ## Getting an image file represented as a BytesIO object
        # convert your image to base64 string
        encoded = base64.b64encode(image_buffer.getvalue())
        image_request = {"image_in_base64": "{0}".format(encoded), "parameters": parameters}
        payload.append(image_request)
    body = json.dumps(payload)
    r = requests.post(service_endpoint_url, data=body, headers=headers)
    try:
        result = json.loads(r.text)
    except:
        raise ValueError("Incorrect output format. Result cant not be parsed: " + r.text)
    return result

# Test with images
images = [test_set.images[0].storage_path, test_set.images[1].storage_path] # A list of local image files
score_image_list_with_http(images, service_endpoint_url, service_key)
```

### <a name="parse-serialized-result-from-web-service"></a>從 Web 服務剖析序列化的結果

Web 服務的輸出是 JSON 字串。 您可以使用不同的 DNN 模型類別，剖析這個 JSON 字串。


```python
image_path_or_url = test_set.images[0].storage_path
print("Image source:",image_path_or_url)
serialized_result_in_json = deploy_obj.score_image(image_path_or_url, image_resize_dims=[224,224])
print("serialized_result_in_json:", serialized_result_in_json)
```


```python
# Parse result from json string
import numpy as np
parsed_result = CNTKTLModel.parse_serialized_result(serialized_result_in_json)
print("Parsed result:", parsed_result)
# Map result to image class
class_index = np.argmax(np.array(parsed_result))
print("Class index:", class_index)
dnn_model.class_map
print("Class label:", dnn_model.class_map[class_index])
```


## <a name="next-steps"></a>後續步驟

在這些文章中，深入了解適用於電腦視覺的 Azure Machine Learning 套件：

+ 了解如何[改善此模型的精確度](how-to-improve-accuracy-for-computer-vision-models.md)。

+ 參閱[套件概觀](https://aka.ms/aml-packages/vision) \(英文\)。

+ 瀏覽此套件的[參考文件](https://docs.microsoft.com/python/api/overview/azure-machine-learning/computer-vision)。

+ 了解[適用於 Azure Machine Learning 的其他 Python 套件](reference-python-package-overview.md)。
