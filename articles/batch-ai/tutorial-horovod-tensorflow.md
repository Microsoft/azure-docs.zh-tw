---
title: 教學課程 - 使用 Azure Batch AI 與 Horovod 進行分散式訓練 | Microsoft Docs
description: 教學課程 - 如何使用 Azure Batch AI 服務和 Azure CLI 並搭配 Horovod 來訓練分散式模型。
services: batch-ai
author: johnwu10
manager: jeconnoc
ms.service: batch-ai
ms.topic: tutorial
ms.date: 09/03/2018
ms.author: danlep
ms.custom: mvc
ms.openlocfilehash: de19b20865127fd37cd7bc1ac854288a95a68091
ms.sourcegitcommit: ebd06cee3e78674ba9e6764ddc889fc5948060c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44058080"
---
# <a name="tutorial-train-a-distributed-model-with-horovod"></a>教學課程：使用 Horovod 訓練分散式模型

在本教學課程中，您將會藉由跨 Batch AI 叢集的多個節點平行執行分散式的深度學習模型，來對該模型進行訓練。 Batch AI 是受控服務，可在 Azure GPU 的叢集上大規模地訓練機器學習和 AI 模型。 

本教學課程會介紹常見的 Batch AI 工作流程，以及如何透過 Azure CLI 與 Batch AI 資源進行互動。 涵蓋的主題包括：

> [!div class="checklist"]
> * 設定 Batch AI 工作區、實驗和叢集
> * 設定 Azure 檔案共用以供輸入和輸出使用
> * 使用 Horovod 對深度學習模型進行平行處理
> * 提交訓練作業
> * 監視作業
> * 擷取訓練結果

本教學課程會將物件偵測模型修改成使用 [Horovod](https://github.com/uber/horovod) 來平行執行。 該模型會透過 [CIFAR-10 影像資料集](https://www.cs.toronto.edu/~kriz/cifar.html)來進行訓練。 訓練作業會在包含 24 個 vCPU 和 4 個 GPU 的叢集上執行，大約 60 分鐘才能完成。

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

如果您選擇在本機安裝和使用 CLI，本教學課程會要求您執行 Azure CLI 2.0.38 版或更新版本。 執行 `az --version` 以尋找版本。 如果您需要安裝或升級，請參閱[安裝 Azure CLI](/cli/azure/install-azure-cli)。 

## <a name="why-use-horovod"></a>為何要使用 Horovod？

本教學課程所使用的 Horovod，是適用於 Tensorflow、Keras 和 PyTorch 的分散式訓練架構。 透過 Horovod，您只需要撰寫幾行程式碼，就能將原本設計為在單一 GPU 上執行的訓練指令碼，轉換成可有效率地在分散式系統上執行的訓練指令碼。

除了 Horovod，Batch AI 也支援搭配其他數個熱門的開放原始碼架構來進行分散式訓練。 針對用來在生產環境中訓練模型的架構，請務必檢閱其授權條款。

## <a name="prepare-the-batch-ai-environment"></a>準備 Batch AI 環境

### <a name="create-a-resource-group"></a>建立資源群組

使用 `az group create` 命令在 `eastus` 區域建立名為 `batchai.horovod` 的資源群組。 您可以使用資源群組來部署 Batch AI 資源。

```azurecli-interactive
az group create --name batchai.horovod --location eastus
```

### <a name="create-a-workspace"></a>建立工作區

使用 `az batchai workspace create` 命令建立 Batch AI 工作區。 工作區是其他 Batch AI 資源的最上層集合。 下列命令會在資源群組底下建立名為 `batchaidev` 的工作區。

```azurecli-interactive
az batchai workspace create --resource-group batchai.horovod --workspace batchaidev 
```

### <a name="create-an-experiment"></a>建立實驗

Batch AI 實驗會將一或多個您所查詢和管理的作業群組在一起。 下列 `az batchai experiment create` 命令會在工作區和資源群組底下建立名為 `cifar` 的實驗。

```azurecli-interactive
az batchai experiment create --resource-group batchai.horovod --workspace batchaidev --name cifar 
```

## <a name="set-up-a-gpu-cluster"></a>設定 GPU 叢集

接下來，設定 GPU 叢集以執行實驗。 Batch AI 提供一組彈性的選項供您自訂叢集以滿足特定需求。

下列 `az batchai cluster create` 命令會在工作區和資源群組底下建立名為 `nc6cluster` 的 4 節點叢集。 根據預設，叢集中的 VM 會執行 Ubuntu Server 映像，此映像的設計目的是要裝載容器型應用程式。 此範例中的叢集節點會使用 `Standard_NC6` 大小，此大小包含一個 NVIDIA Tesla K80 GPU。

```azurecli-interactive
az batchai cluster create --resource-group batchai.horovod --workspace batchaidev --name nc6cluster --vm-priority dedicated  --vm-size Standard_NC6 --target 4 --generate-ssh-keys
```

執行 `az batchai cluster show` 命令來檢視叢集狀態。 通常需要幾分鐘的時間才能完整佈建叢集。

```azurecli-interactive
az batchai cluster show --name nc6cluster --workspace batchaidev --resource-group batchai.horovod --output table
```

在叢集建立早期，叢集處於 `resizing` 狀態。 請在叢集狀態變更時繼續進行下列步驟。 當叢集狀態為 `steady` 且節點為 `idle` 時，叢集便準備好執行訓練作業。 例如︰

```
Name        Resource Group    Workspace    VM Size       State      Idle    Running    Preparing    Leaving    Unusable
----------  ----------------  -----------  ------------  -------  ------  ---------  -----------  ---------  ----------
nc6cluster  batchai.horovod  batchaidev   STANDARD_NC6  steady        4          0            0          0           0
```

## <a name="set-up-storage"></a>設定儲存體

使用 `az storage account create` 命令來建立儲存體帳戶，以便儲存訓練指令碼和訓練輸出。

```azurecli-interactive
az storage account create --resource-group batchai.horovod --name mystorageaccount --location eastus --sku Standard_LRS
```

使用 `az storage share create` 命令在帳戶中建立名為 `myshare` 的 Azure 檔案共用：

```azurecli-interactive
az storage share create --name myshare --account-name mystorageaccount
```

實際上，這個儲存體也可跨多個作業和實驗來使用。 為了讓一切井然有序，請在檔案共用內建立目錄，以儲存此特定實驗的相關檔案。 下列 `az storage directory create` 命令會建立名為 `cifar` 的目錄。

```azurecli-interactive
az storage directory create --name cifar --share-name myshare --account-name mystorageaccount
```

下一個步驟是準備實際的訓練指令碼，然後上傳至新建立的目錄。

## <a name="create-the-training-script"></a>建立訓練指令碼

在此實驗中，您會執行使用一些變更加以更新的 Python 指令碼，以便使用 Horovod 平行執行物件偵測模型。 [原始模型](https://raw.githubusercontent.com/keras-team/keras/master/examples/cifar10_cnn.py)會搭配使用 Keras 與 TensorFlow 後端。 

在殼層的工作目錄中，使用慣用的文字編輯器來建立具有下列內容、名為 `cifar_cnn_distributed.py` 的檔案。 對原始程式碼所做的變更會加上 `HOROVOD` 前置詞來加以註解。

```python
from __future__ import print_function
import keras
from keras.datasets import cifar10
from keras.preprocessing.image import ImageDataGenerator
from keras.models import Sequential
from keras.layers import Dense, Dropout, Activation, Flatten
from keras.layers import Conv2D, MaxPooling2D
import tensorflow as tf
import horovod.keras as hvd
import os
from keras import backend as K
import math
import argparse 

# HOROVOD: initialize Horovod.
hvd.init()

# HOROVOD: pin GPU to be used to process local rank (one GPU per process)
config = tf.ConfigProto()
config.gpu_options.allow_growth = True
config.gpu_options.visible_device_list = str(hvd.local_rank())
K.set_session(tf.Session(config=config))

batch_size = 32
num_classes = 10
# HOROVOD: adjust number of epochs based on number of GPUs.
epochs = int(math.ceil(100.0 / hvd.size()))

data_augmentation = True
num_predictions = 20
# BATCH AI: change save directory to mounted storage path
parser = argparse.ArgumentParser()
parser.add_argument("-d", "--dir", help="directory to save model to")
args = parser.parse_args()
save_dir = os.path.join(args.dir, 'saved_models')
model_name = 'keras_cifar10_trained_model.h5'

# The data, split between train and test sets:
(x_train, y_train), (x_test, y_test) = cifar10.load_data()
print('x_train shape:', x_train.shape)
print(x_train.shape[0], 'train samples')
print(x_test.shape[0], 'test samples')

# Convert class vectors to binary class matrices.
y_train = keras.utils.to_categorical(y_train, num_classes)
y_test = keras.utils.to_categorical(y_test, num_classes)

model = Sequential()
model.add(Conv2D(32, (3, 3), padding='same',
                 input_shape=x_train.shape[1:]))
model.add(Activation('relu'))
model.add(Conv2D(32, (3, 3)))
model.add(Activation('relu'))
model.add(MaxPooling2D(pool_size=(2, 2)))
model.add(Dropout(0.25))

model.add(Conv2D(64, (3, 3), padding='same'))
model.add(Activation('relu'))
model.add(Conv2D(64, (3, 3)))
model.add(Activation('relu'))
model.add(MaxPooling2D(pool_size=(2, 2)))
model.add(Dropout(0.25))

model.add(Flatten())
model.add(Dense(512))
model.add(Activation('relu'))
model.add(Dropout(0.5))
model.add(Dense(num_classes))
model.add(Activation('softmax'))

# HOROVOD: adjust learning rate based on number of GPUs.
opt = keras.optimizers.rmsprop(lr=0.0001 * hvd.size(), decay=1e-6)

# HOROVOD: add Horovod Distributed Optimizer.
opt = hvd.DistributedOptimizer(opt)

# Let's train the model using RMSprop
model.compile(loss='categorical_crossentropy',
              optimizer=opt,
              metrics=['accuracy'])

callbacks = [
    # HOROVOD: broadcast initial variable states from rank 0 to all other processes.
    # This is necessary to ensure consistent initialization of all workers when
    # training is started with random weights or restored from a checkpoint.
    hvd.callbacks.BroadcastGlobalVariablesCallback(0),
]

# HOROVOD: save checkpoints only on worker 0 to prevent other workers from corrupting them.
if hvd.rank() == 0:
    callbacks.append(keras.callbacks.ModelCheckpoint('./checkpoint-{epoch}.h5'))

x_train = x_train.astype('float32')
x_test = x_test.astype('float32')
x_train /= 255
x_test /= 255

if not data_augmentation:
    print('Not using data augmentation.')
    model.fit(x_train, y_train,
              batch_size=batch_size,
              epochs=epochs,
              validation_data=(x_test, y_test),
              shuffle=True)
else:
    print('Using real-time data augmentation.')
    # This will do preprocessing and realtime data augmentation:
    datagen = ImageDataGenerator(
        featurewise_center=False,  # set input mean to 0 over the dataset
        samplewise_center=False,  # set each sample mean to 0
        featurewise_std_normalization=False,  # divide inputs by std of the dataset
        samplewise_std_normalization=False,  # divide each input by its std
        zca_whitening=False,  # apply ZCA whitening
        zca_epsilon=1e-06,  # epsilon for ZCA whitening
        rotation_range=0,  # randomly rotate images in the range (degrees, 0 to 180)
        width_shift_range=0.1,  # randomly shift images horizontally (fraction of total width)
        height_shift_range=0.1,  # randomly shift images vertically (fraction of total height)
        shear_range=0.,  # set range for random shear
        zoom_range=0.,  # set range for random zoom
        channel_shift_range=0.,  # set range for random channel shifts
        fill_mode='nearest',  # set mode for filling points outside the input boundaries
        cval=0.,  # value used for fill_mode = "constant"
        horizontal_flip=True,  # randomly flip images
        vertical_flip=False,  # randomly flip images
        rescale=None,  # set rescaling factor (applied before any other transformation)
        preprocessing_function=None,  # set function that will be applied on each input
        data_format=None,  # image data format, either "channels_first" or "channels_last"
        validation_split=0.0)  # fraction of images reserved for validation (strictly between 0 and 1)

    # Compute quantities required for feature-wise normalization
    # (std, mean, and principal components if ZCA whitening is applied).
    datagen.fit(x_train)

    # Fit the model on the batches generated by datagen.flow().
    model.fit_generator(datagen.flow(x_train, y_train,
                                     batch_size=batch_size),
                        epochs=epochs,
                        validation_data=(x_test, y_test),
                        workers=4)

# Save model and weights
if not os.path.isdir(save_dir):
    os.makedirs(save_dir)
model_path = os.path.join(save_dir, model_name)
model.save(model_path)
print('Saved trained model at %s ' % model_path)

# Score trained model.
scores = model.evaluate(x_test, y_test, verbose=1)
print('Test loss:', scores[0])
print('Test accuracy:', scores[1])
```

如本範例所示，您只需要對模型做一些更新，即可啟用使用 Horovod 架構的分散式訓練。 

請記住，此指令碼會使用相對較小的模型和資料集來做示範，因此，這個分散式模型不一定能展顯出效能的巨大改善。 若要真正地看到分散式訓練的威力，請使用較大的模型和資料集。

## <a name="upload-the-training-script"></a>上傳訓練指令碼

指令碼準備就緒後，下一個步驟是將它上傳到您稍早建立的檔案共用目錄。 下列 `az storage file upload` 命令會將它從本機工作目錄上傳到正確位置。

```azurecli-interactive
az storage file upload --path cifar --share-name myshare --source cifar_cnn_distributed.py --account-name mystorageaccount
```

## <a name="submit-the-training-job"></a>提交訓練作業

完成前面的步驟之後，請建立訓練作業。 在 Batch AI 中，您可以使用 `job.json` 檔案來定義參數，以指出作業的執行方式。 請使用慣用的文字編輯器，建立具有下列內容、名為 `job.json` 的作業設定檔。

```json
{
    "$schema": "https://raw.githubusercontent.com/Azure/BatchAI/master/schemas/2018-05-01/job.json",
    "properties": {
        "nodeCount": 4,
        "horovodSettings": {
            "pythonScriptFilePath": "$AZ_BATCHAI_JOB_MOUNT_ROOT/myshare/cifar/cifar_cnn_distributed.py",
            "commandLineArgs": "--dir=$AZ_BATCHAI_JOB_MOUNT_ROOT/myshare/cifar"
        },
        "stdOutErrPathPrefix": "$AZ_BATCHAI_JOB_MOUNT_ROOT/myshare/cifar",
        "mountVolumes": {
            "azureFileShares": [
                {
                    "azureFileUrl": "https://<AZURE_BATCHAI_STORAGE_ACCOUNT>.file.core.windows.net/myshare",
                    "relativeMountPath": "myshare"
                }
            ]
        },
        "jobPreparation": {
            "commandLine": "apt update; apt install mpi-default-dev mpi-default-bin -y; pip install keras horovod"
        },
        "containerSettings": {
            "imageSourceRegistry": {
                "image": "tensorflow/tensorflow:1.8.0-gpu"
            }
        }
    }
}
```

屬性的說明：

| 屬性 | 說明 |
| --------- | --------- |
| `nodeCount` | 專用於作業的節點數目。 在這裡，作業會在 `4` 節點上平行執行。 |
| `horovodSettings` | `pythonScriptFilePath` 欄位會定義 Horovod 指令碼 (位於先前建立的 `cifar` 目錄) 的路徑。 `commandLineArgs` 欄位是用於執行指令碼的命令列引數。 在此實驗中，用來儲存模型的目錄會是唯一必要的引數。 `$AZ_BATCHAI_JOB_MOUNT_ROOT/myshare` 是檔案共用掛接所在的路徑。 | 
| `stdOutErrPathPrefix` | 用來儲存作業輸出和記錄的路徑，在此範例中為相同的 `cifar` 目錄。 |
| `jobPreparation` | 適用於準備環境和執行作業的任何特殊指示。 此指令碼需要安裝指定的 MPI 和 Horovod 套件。 |
| `containerSettings` | 作業執行所在容器的設定。 此作業會使用以 `tensorflow` 建置的 Docker 容器。

使用此設定，透過 `az batchai job create` 命令建立作業。 下列命令會將名為 `cifar_distributed` 的新作業排入佇列，其使用截至目前為止已設定的所有資源。 

```azurecli-interactive
az batchai job create --cluster nc6cluster --name cifar_distributed --resource-group batchai.horovod --workspace batchaidev --experiment cifar --config-file job.json --storage-account-name mystorageaccount
```

如果節點目前忙碌中，此工作可能需要等候一段時間才會開始執行。 使用 `az batchai job show` 命令來檢視作業的執行狀態。

```azurecli-interactive
az batchai job show --experiment cifar --name cifar_distributed --resource-group batchai.horovod --workspace batchaidev --query "executionState"
```

### <a name="visualize-the-distributed-training"></a>以視覺化方式呈現分散式訓練

作業開始執行後，請再次使用 `az batchai cluster show` 命令來查詢叢集節點的狀態。 

```azurecli-interactive
az batchai cluster show --name nc6cluster --workspace batchaidev --resource-group batchai.horovod --query "nodeStateCounts"
```

輸出應該會類似下列內容，其中顯示四個皆處於執行中狀態。 此結果顯示分散式訓練中的四個節點目前全都正在使用。

```
{
  "idleNodeCount": 0,
  "leavingNodeCount": 0,
  "preparingNodeCount": 0,
  "runningNodeCount": 4,
  "unusableNodeCount": 0
}
```

## <a name="monitor-the-job"></a>監視作業

### <a name="list-output-files"></a>列出輸出檔案

在作業執行時，使用 `az batchai job file list` 命令來列出作業所產生的輸出檔。

```azurecli-interactive
az batchai job file list --experiment cifar --job cifar_distributed --resource-group batchai.horovod --workspace batchaidev --output table
```

針對此特定實驗，輸出應該會類似下列內容。 作業的整體輸出會記錄到 `stdout.txt`，`stderr.txt` 則會輸出主要執行中所發生的任何錯誤。 其他檔案則是與每個個別節點對應的輸出、錯誤和作業準備記錄。

```
Name                                                    Type       Size  Modified
------------------------------------------------------  ------  -------  -------------------------
execution-tvm-676767296_1-20180718t174802z-p.log        file       8801  2018-07-18T22:41:28+00:00
execution-tvm-676767296_2-20180718t174802z-p.log        file      15094  2018-07-18T22:41:55+00:00
execution-tvm-676767296_3-20180718t174802z-p.log        file       8801  2018-07-18T22:41:28+00:00
execution-tvm-676767296_4-20180718t174802z-p.log        file       8801  2018-07-18T22:41:28+00:00
stderr-job_prep-tvm-676767296_1-20180718t174802z-p.txt  file        238  2018-07-18T22:41:50+00:00
stderr-job_prep-tvm-676767296_2-20180718t174802z-p.txt  file        238  2018-07-18T22:41:50+00:00
stderr-job_prep-tvm-676767296_3-20180718t174802z-p.txt  file        238  2018-07-18T22:41:50+00:00
stderr-job_prep-tvm-676767296_4-20180718t174802z-p.txt  file        238  2018-07-18T22:41:50+00:00
stderr.txt                                              file       7653  2018-07-18T22:46:32+00:00
stdout-job_prep-tvm-676767296_1-20180718t174802z-p.txt  file      13651  2018-07-18T22:41:55+00:00
stdout-job_prep-tvm-676767296_2-20180718t174802z-p.txt  file      13651  2018-07-18T22:41:54+00:00
stdout-job_prep-tvm-676767296_3-20180718t174802z-p.txt  file      13651  2018-07-18T22:41:54+00:00
stdout-job_prep-tvm-676767296_4-20180718t174802z-p.txt  file      13651  2018-07-18T22:41:55+00:00
stdout.txt                                              file    2316480  2018-07-18T22:46:32+00:00
```

### <a name="stream-an-output-file"></a>串流輸出檔

使用 `az batchai job file stream` 命令來串流檔案內容。 下列範例會串流主要的輸出記錄。

```azurecli-interactive
az batchai job file stream --experiment cifar --file-name stdout.txt --job cifar_distributed --resource-group batchai.horovod --workspace batchaidev
```

在作業執行時，此命令會串流訓練作業的標準輸出，以顯示類似下列內容的輸出。

```
...
50000 train samples
10000 test samples
Using real-time data augmentation.
Epoch 1/25


   1/1563 [..............................] - ETA: 2:42:25 - loss: 2.3334 - acc: 0.0312   1/1563 [..............................] - ETA: 2:30:42 - loss: 2.2973 - acc: 0.0938
   1/1563 [..............................] - ETA: 30:36 - loss: 2.3175 - acc: 0.1250
   1/1563 [..............................] - ETA: 2:32:58 - loss: 2.3489 - acc: 0.0625
   2/1563 [..............................] - ETA: 1:21:59 - loss: 2.3230 - acc: 0.0625

   2/1563 [..............................]   2/1563 [..............................] - ETA: 1:16:09 - loss: 2.2913 - acc: 0.0938 - ETA: 1:17:15 - loss: 2.3147 - acc: 0.0781
   2/1563 [..............................] - ETA: 16:07 - loss: 2.3678 - acc: 0.0938
   3/1563 [..............................] - ETA: 55:05 - loss: 2.3232 - acc: 0.0938  
   3/1563 [..............................] - ETA: 51:57 - loss: 2.3185 - acc: 0.1146  
   3/1563 [..............................] - ETA: 51:12 - loss: 2.3179 - acc: 0.1042  
   3/1563 [..............................] - ETA: 11:13 - loss: 2.3504 - acc: 0.0833
   4/1563 [..............................] - ETA: 39:43 - loss: 2.3224 - acc: 0.1094
   4/1563 [..............................] - ETA: 42:09 - loss: 2.3049 - acc: 0.1250
   4/1563 [..............................] - ETA: 39:15 - loss: 2.3089 - acc: 0.1094
   4/1563 [..............................] - ETA: 9:16 - loss: 2.3316 - acc: 0.1016 
   5/1563 [..............................] - ETA: 39:51 - loss: 2.3153 - acc: 0.1125
   5/1563 [..............................] - ETA: 37:58 - loss: 2.3197 - acc: 0.1125
   5/1563 [..............................] - ETA: 37:35 - loss: 2.3148 - acc: 0.1062
   5/1563 [..............................] - ETA: 13:38 - loss: 2.3263 - acc: 0.1062
   6/1563 [..............................] - ETA: 35:48 - loss: 2.3168 - acc: 0.1198

   6/1563 [..............................]   6/1563 [..............................] - ETA: 34:13 - loss: 2.3142 - acc: 0.1198 - ETA: 33:51 - loss: 2.3162 - acc: 0.1042
   6/1563 [..............................] - ETA: 13:54 - loss: 2.3225 - acc: 0.1094
   7/1563 [..............................] - ETA: 30:53 - loss: 2.3181 - acc: 0.1071

   7/1563 [..............................]   7/1563 [..............................] - ETA: 29:32 - loss: 2.3149 - acc: 0.1161 - ETA: 29:13 - loss: 2.3140 - acc: 0.0938
   7/1563 [..............................] - ETA: 12:09 - loss: 2.3174 - acc: 0.1205
   8/1563 [..............................] - ETA: 26:04 - loss: 2.3113 - acc: 0.1133
   8/1563 [..............................] - ETA: 27:15 - loss: 2.3169 - acc: 0.1133
   8/1563 [..............................] - ETA: 10:51 - loss: 2.3152 - acc: 0.1172
...
```

指令碼會訓練超過 25 個 Epoch 或通過訓練資料集。 此程序需要大約 60 分鐘才能完成。 

## <a name="retrieve-the-results"></a>擷取結果

指令碼完成時，如果一切順利，驗證精確度應該大約在 70% 至 75%，且經過訓練的模型會儲存至檔案共用的 `cifar/saved_models/keras_cifar10_trained_model.h5`。 

較大型的工作流程中通常會包含模型訓練。 例如，您可能會在另一個應用程式公開經過訓練的模型。 若要在本機下載經過訓練的模型，請使用 `az storage file download` 命令。 

```azurecli-interactive
az storage file download --path cifar/saved_models/keras_cifar10_trained_model.h5 --share-name myshare --account-name mystorageaccount
```
## <a name="clean-up-resources"></a>清除資源

作業執行完成後，您最好將所有叢集縮小為 `0 nodes`，這樣一來，您就不需要就閒置時間支付費用，而能節省計算成本。 請使用下列 `az batchai cluster resize` 命令。 

```azurecli-interactive
az batchai cluster resize --name nc6cluster --resource-group batchai.horovod --target 0 --workspace batchaidev
```

之後，請將其大小調整為 1 個以上的節點，以便執行作業。 

如果您未來不打算再使用工作區和儲存體帳戶，請使用 `az group delete` 命令刪除資源群組。 刪除資源群組也會刪除屬於該群組的所有資源。

```azurecli-interactive
az group delete --name batchai.horovod
```

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

> [!div class="checklist"]
> * 設定 Batch AI 工作區、實驗和叢集
> * 設定 Azure 檔案共用以供輸入和輸出使用
> * 使用 Horovod 對模型進行平行處理
> * 提交訓練作業
> * 監視作業
> * 擷取訓練結果

如需搭配不同架構來使用 Batch AI 的範例，請參閱 GitHub 上的配方。

> [!div class="nextstepaction"]
> [Batch AI 配方](https://github.com/Azure/BatchAI/tree/master/recipes)
