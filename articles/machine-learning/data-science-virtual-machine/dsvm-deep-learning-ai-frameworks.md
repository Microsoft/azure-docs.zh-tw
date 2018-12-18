---
title: 深度學習和 AI 架構 - Azure | Microsoft Docs
description: 深度學習和 AI 架構
keywords: 資料科學工具、資料科學虛擬機器、資料科學工具、linux 資料科學
services: machine-learning
documentationcenter: ''
author: gopitk
manager: cgronlun
ms.assetid: ''
ms.service: machine-learning
ms.component: data-science-vm
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 09/11/2017
ms.author: gokuma
ms.openlocfilehash: 891059a440189112c834f3402725781a6b4a3960
ms.sourcegitcommit: a1e1b5c15cfd7a38192d63ab8ee3c2c55a42f59c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/10/2018
ms.locfileid: "37952819"
---
# <a name="deep-learning-and-ai-frameworks"></a>深度學習和 AI 架構
[資料科學虛擬機器](http://aka.ms/dsvm) \(英文\) (DSVM) 和[深度學習 VM](http://aka.ms/dsvm/deeplearning) \(英文\) 支援數種深度學習架構，可協助建置具備預測性分析及能了解影像及語言之辨識能力的人工智慧 (AI) 應用程式。 

以下是 DSVM 上所提供之所有深度學習架構的詳細資料。

## <a name="microsoft-cognitive-toolkit"></a>Microsoft 辨識工具組

|    |           |
| ------------- | ------------- |
| 這是什麼？   | 深度學習架構      |
| 支援的 DSVM 版本      | Windows、Linux     |
| 它是如何在 DSVM 上設定/安裝的？  | Microsoft Cognitive Toolkit (CNTK) 安裝在 [Linux 和 Windows 2012](dsvm-languages.md#python-linux-and-windows-server-2012-edition) 的 Python 3.5 中，以及安裝在 [Windows 2016](dsvm-languages.md#python-windows-server-2016-edition) 的 Python 3.6 中。   |
| 範例的連結      | 已包含範例 Jupyter 筆記本。     |
| DSVM 上的相關工具      | Keras      |
| 如何使用/執行它？    | * 在終端機：啟用正確的環境，然後執行 Python。 <br/>
 * 在 Jupyter：連線到 [Jupyter](provision-vm.md#tools-installed-on-the-microsoft-data-science-virtual-machine) 或 [JupyterHub](dsvm-ubuntu-intro.md#how-to-access-the-data-science-virtual-machine-for-linux)，然後開啟 CNTK 目錄找出範例。 |

## <a name="tensorflow"></a>TensorFlow

|    |           |
| ------------- | ------------- |
| 這是什麼？   | 深度學習架構      |
| 支援的 DSVM 版本      | Windows、Linux     |
| 它是如何在 DSVM 上設定/安裝的？  | TensorFlow 安裝在 [Linux 和 Windows 2012](dsvm-languages.md#python-linux-and-windows-server-2012-edition) 的 Python 3.5 中，以及安裝在 [Windows 2016](dsvm-languages.md#python-windows-server-2016-edition) 的 Python 3.6 中。  |
| 範例的連結      | 已包含範例 Jupyter 筆記本。     |
| DSVM 上的相關工具      | Keras      |
| 如何使用/執行它？    | * 在終端機：啟用正確的環境，然後執行 Python。 <br/>
 * 在 Jupyter：連線到 [Jupyter](provision-vm.md#tools-installed-on-the-microsoft-data-science-virtual-machine) 或 [JupyterHub](dsvm-ubuntu-intro.md#how-to-access-the-data-science-virtual-machine-for-linux)，然後開啟 TensorFlow 目錄找出範例。  |

## <a name="horovod"></a>Horovod

|    |           |
| ------------- | ------------- |
| 這是什麼？   | 適用於 TensorFlow 的分散式深度學習架構      |
| 支援的 DSVM 版本      | Ubuntu     |
| 它是如何在 DSVM 上設定/安裝的？  | Horovod 安裝在 [Ubuntu](dsvm-languages.md#python-linux-and-windows-server-2012-edition) 的 Python 3.5 中。  |
| 範例的連結      | [https://github.com/uber/horovod/tree/master/examples](https://github.com/uber/horovod/tree/master/examples)     |
| DSVM 上的相關工具      | TensorFlow      |
| 如何使用/執行它？    | 在終端機：啟用正確的環境，然後執行 Python。 |

## <a name="keras"></a>Keras

|    |           |
| ------------- | ------------- |
| 這是什麼？   | 高階深度學習 API      |
| 支援的 DSVM 版本      | Windows、Linux     |
| 它是如何在 DSVM 上設定/安裝的？  | TensorFlow 安裝在 [Linux 和 Windows 2012](dsvm-languages.md#python-linux-and-windows-server-2012-edition) 的 Python 3.5 中，以及安裝在 [Windows 2016](dsvm-languages.md#python-windows-server-2016-edition) 的 Python 3.6 中。 |
| 範例的連結      | https://github.com/fchollet/keras/tree/master/examples      |
| DSVM 上的相關工具      | Microsoft 辨識工具組、TensorFlow、Theano      |
| 如何使用/執行它？    | * 在終端機：啟用正確的環境，然後執行 Python。 <br/>
 * 在 Jupyter：從 Github 位置下載範例，連線到 [Jupyter](provision-vm.md#tools-installed-on-the-microsoft-data-science-virtual-machine) 或 [JupyterHub](dsvm-ubuntu-intro.md#how-to-access-the-data-science-virtual-machine-for-linux)，然後開啟範例目錄。 |

## <a name="caffe"></a>Caffe

|    |           |
| ------------- | ------------- |
| 這是什麼？   | 深度學習架構      |
| 支援的 DSVM 版本      | Ubuntu     |
| 它是如何在 DSVM 上設定/安裝的？  | Caffe 是安裝在 `/opt/caffe` 中。    |
| 如何切換至 Python 2.7 | 執行 `source activate root` |
| 範例的連結      | 範例會包含在 `/opt/caffe/examples`中。      |
| DSVM 上的相關工具      | Caffe2      |
### <a name="how-to-use--run-it"></a>如何使用/執行它？  

使用 X2Go 登入您的 VM，然後啟動新的終端機，並輸入

```
cd /opt/caffe/examples
source activate root
jupyter notebook
```

具有範例筆記本的新瀏覽器視窗將會開啟。

二進位檔安裝在 /opt/caffe/build/install/bin。

已安裝的 Caffe 版本需要 Python 2.7，而且在預設啟用 Python 3.5 的情況下無法運作。 執行 `source activate root` 以切換 Anaconda 環境。 

## <a name="caffe2"></a>Caffe2

|    |           |
| ------------- | ------------- |
| 這是什麼？   | 深度學習架構      |
| 支援的 DSVM 版本      | Ubuntu     |
| 它是如何在 DSVM 上設定/安裝的？  | Caffe2 安裝在 [Python 2.7 (root) conda 環境](dsvm-languages.md#python-linux-and-windows-server-2012-edition)。 原始檔在 `/opt/caffe2`。 |
| 範例的連結      | JupyterHub 中包含範例筆記本。 |
| DSVM 上的相關工具      | Caffe      |
| 如何使用/執行它？    | * 在終端機：啟用 [root Python 環境](dsvm-languages.md#python-linux-and-windows-server-2012-edition)，啟動 Python，然後匯入 caffe2。 <br/> * 在 JupyterHub：[連線到 JupyterHub](dsvm-ubuntu-intro.md#how-to-access-the-data-science-virtual-machine-for-linux)，然後瀏覽至 Caffe2 目錄以尋找範例筆記本。 某些筆記本會要求以 Python 程式碼設定 Caffe2 根，請輸入 /opt/caffe2。 |
| 組建注意事項 | Caffe2 是從 Linux 上的原始檔組建，包括 CUDA、cuDNN 和 Intel MKL。 目前的認可是 0d9c0d48c6f20143d6404b99cc568efd29d5a4be，選取的理由是在測試的所有 GPU 和範例的穩定性。 |

## <a name="chainer"></a>Chainer

|    |           |
| ------------- | ------------- |
| 這是什麼？   | 深度學習架構      |
| 支援的 DSVM 版本      | Windows、Linux     |
| 它是如何在 DSVM 上設定/安裝的？  | Chainer 安裝在 [Python 3.5](dsvm-languages.md#python-linux-and-windows-server-2012-edition) 中。 也會安裝 ChainerRL 和 ChainerCV。   |
| 範例的連結      | JupyterHub 中包含範例筆記本。 |
| DSVM 上的相關工具      | Caffe      |
| 如何使用/執行它？  | * 在終端機：啟用 [Python 3.5](dsvm-languages.md#python-linux-and-windows-server-2012-edition) 環境，執行 _python_，然後匯入 Chainer。 <br/>
* 在 JupyterHub：[連線到 JupyterHub](dsvm-ubuntu-intro.md#how-to-access-the-data-science-virtual-machine-for-linux)，然後瀏覽至 Chainer 目錄以尋找範例筆記本。


## <a name="deep-water"></a>Deep Water

|    |           |
| ------------- | ------------- |
| 這是什麼？   | 適用於 H2O 的深度學習架構      |
| 支援的 DSVM 版本      | Ubuntu     |
| 它是如何在 DSVM 上設定/安裝的？  | Deep Water 安裝在 [Python 3.5](dsvm-languages.md#python-linux-and-windows-server-2012-edition) 中，`/dsvm/tools/deep_water` 中也提供。   |
| 範例的連結      | JupyterHub 中包含範例筆記本。      |
| DSVM 上的相關工具      | H2O、Sparkling Water      |

### <a name="how-to-use--run-it"></a>如何使用/執行它？  

Deep Water 需要 CUDA 8 搭配 cuDNN 5.1。 這不是在預設的程式庫路徑中，因為其他深度學習架構使用 CUDA 9 和 cuDNN 7。 為 Deep Water 使用 CUDA 8 + cuDNN 5.1：

```
export LD_LIBRARY_PATH=/usr/local/cuda-8.0/lib64:${LD_LIBRARY_PATH}
export CUDA_ROOT=/usr/local/cuda-8.0
```

使用 Deep Water：
* * 在終端機：啟用 [Python 3.5](dsvm-languages.md#python-linux-and-windows-server-2012-edition) 環境，然後執行 _python_。 <br/>
* 在 JupyterHub：[連線到 JupyterHub](dsvm-ubuntu-intro.md#how-to-access-the-data-science-virtual-machine-for-linux)，然後瀏覽至 deep_water 目錄以尋找範例筆記本。

## <a name="mxnet"></a>MXNet

|    |           |
| ------------- | ------------- |
| 這是什麼？   | 深度學習架構      |
| 支援的 DSVM 版本      | Windows、Linux     |
| 它是如何在 DSVM 上設定/安裝的？  | MXNet 是安裝在 Windows 上的 `C:\dsvm\tools\mxnet` 中，以及 Linux 上的 `/dsvm/tools/mxnet` 中。 Python 繫結安裝在 [Linux 和 Windows 2012](dsvm-languages.md#python-linux-and-windows-server-2012-edition) 的 Python 3.5 中，以及安裝在 [Windows 2016](dsvm-languages.md#python-windows-server-2016-edition) 的 Python 3.6 中。 R 繫結也會安裝在 Ubuntu 上。   |
| 範例的連結      | 已包含範例 Jupyter 筆記本。    |
| DSVM 上的相關工具      | Keras      |
| 如何使用/執行它？    | * 在終端機：啟用正確的環境，然後執行 Python。 <br/>
 * 在 Jupyter：連線到 [Jupyter](provision-vm.md#tools-installed-on-the-microsoft-data-science-virtual-machine) 或 [JupyterHub](dsvm-ubuntu-intro.md#how-to-access-the-data-science-virtual-machine-for-linux)，然後開啟 mxnet 目錄找出範例。  |
 | 組建注意事項 | MXNet 是從 Linux 上的原始檔組建。 這個組建包含 CUDA、cuDNN、NCCL 和 MKL。 |

## <a name="nvidia-digits"></a>NVIDIA DIGITS

|    |           |
| ------------- | ------------- |
| 這是什麼？   | 來自 NVIDIA 的深度學習系統，可用來快速訓練深度學習模型      |
| 支援的 DSVM 版本      | Ubuntu     |
| 它是如何在 DSVM 上設定/安裝的？  | DIGITS 是安裝在 `/dsvm/tools/DIGITS` 中，並以稱為 _digits_ 之服務的形式提供。   |
### <a name="how-to-use--run-it"></a>如何使用/執行它？  

使用 X2Go 登入 VM。 在終端機中，啟動該服務：

    sudo systemctl start digits

服務需要約一分鐘才能啟動。 開啟網頁瀏覽器，並瀏覽至 `http://localhost:5000`。



## <a name="nvidia-smi"></a>nvidia-smi

|    |           |
| ------------- | ------------- |
| 這是什麼？   | 用來查詢 GPU 活動的 NVIDIA 工具      |
| 支援的 DSVM 版本      | Windows、Linux     |
| 它是如何在 DSVM 上設定/安裝的？  | _nvidia-smi_ 可於系統路徑上取得。   |
| 如何使用/執行它？ | 啟動命令提示字元 (在 Windows 上) 或終端機 (在 Linux 上)，然後執行 _nvidia-smi_。



## <a name="theano"></a>Theano

|    |           |
| ------------- | ------------- |
| 這是什麼？   | 深度學習架構      |
| 支援的 DSVM 版本      | Ubuntu     |
| 它是如何在 DSVM 上設定/安裝的？  | Theano 是安裝在 Python 2.7 (_root_) 及 Python 3.5 (_py35_) 環境中。   |
| DSVM 上的相關工具      | Keras      |
| 如何使用/執行它？    | * 在終端機中，啟動所需的 Python 版本 (root 或 py35)，執行 Python，然後匯入 Theano。 <br/> 
* 在 Jupyter 中，選取 [Python 2.7] 或 [Python 3.5] 核心，然後匯入 Theano。  
<br/>
若要解決最近的 MKL 錯誤 (bug)，您必須先設定 MKL 執行緒層：<br/><br/>
_export MKL_THREADING_LAYER=GNU_
|



## <a name="torch"></a>Torch

|    |           |
| ------------- | ------------- |
| 這是什麼？   | 深度學習架構      |
| 支援的 DSVM 版本      | Ubuntu     |
| 它是如何在 DSVM 上設定/安裝的？  | Torch 是安裝在 `/dsvm/tools/torch` 中。 PyTorch 是安裝在 Python 2.7 (_root_) 及 Python 3.5 (_py35_) 環境中。   |
| 範例的連結      | Torch 範例位於 `/dsvm/samples/torch`。 PyTorch 範例位於 `/dsvm/samples/pytorch`。      |


## <a name="pytorch"></a>PyTorch

|    |           |
| ------------- | ------------- |
| 這是什麼？   | 深度學習架構      |
| 支援的 DSVM 版本      | Linux     |
| 它是如何在 DSVM 上設定/安裝的？  | PyTorch 安裝在 [Python 3.5](dsvm-languages.md#python-linux-and-windows-server-2012-edition) 中。  |
| 範例的連結      | 隨附 Jupyter 筆記本範例，也可以在 /dsvm/samples/pytorch 中找到範例。      |
| DSVM 上的相關工具      | Torch      |
| 如何使用/執行它 | 
* 在終端機：啟用正確的環境，然後執行 Python。 <br/>
 * 在 Jupyter：連線到 [JupyterHub](dsvm-ubuntu-intro.md#how-to-access-the-data-science-virtual-machine-for-linux)，然後開啟 PyTorch 目錄找出範例。  |

## <a name="mxnet-model-server"></a>MXNet 模型伺服器

|    |           |
| ------------- | ------------- |
| 這是什麼？   | 建立 MXNet 和 ONNX 模型 HTTP 端點的伺服器      |
| 支援的 DSVM 版本      | Linux     |
| 它是如何在 DSVM 上設定/安裝的？  | 可在終端機上使用 _mxnet-model-server_。   |
| 範例的連結      | 在 [MXNet 模型伺服器頁面](https://github.com/awslabs/mxnet-model-server)上尋找最新的範例。    |
| DSVM 上的相關工具      | MXNet      |

## <a name="tensorflow-serving"></a>TensorFlow 服務

|    |           |
| ------------- | ------------- |
| 這是什麼？   | 要在 TensorFlow 模型上執行推斷的伺服器      |
| 支援的 DSVM 版本      | Linux     |
| 它是如何在 DSVM 上設定/安裝的？  | 可在終端機上使用 _tensorflow_model_server_。   |
| 範例的連結      | 可從[線上](https://www.tensorflow.org/serving/)取得範例。      |
| DSVM 上的相關工具      | TensorFlow      |

## <a name="tensorrt"></a>TensorRT

|    |           |
| ------------- | ------------- |
| 這是什麼？   | NVIDIA 的深度學習推斷伺服器。 |
| 支援的 DSVM 版本      | Ubuntu     |
| 它是如何在 DSVM 上設定/安裝的？  | TensorRT 是安裝為 _apt_ 套件。   |
| 範例的連結      | 可從[線上](https://docs.nvidia.com/deeplearning/sdk/tensorrt-developer-guide/index.html#samples)取得範例。      |
| DSVM 上的相關工具      | TensorFlow 服務、MXNet 模型伺服器  |



