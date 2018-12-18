---
title: 機器學習與資料科學工具 - Azure | Microsoft Docs
description: 機器學習與資料科學工具
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
ms.date: 03/16/2018
ms.author: gokuma
ms.openlocfilehash: 9a9dc868c4f22f95ca5027e3c95513d176c69eac
ms.sourcegitcommit: d1aef670b97061507dc1343450211a2042b01641
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47392392"
---
# <a name="machine-learning-and-data-science-tools"></a>機器學習與資料科學工具
資料科學虛擬機器 (DSVM) 具有一系列適用於機器學習服務的豐富工具和程式庫，並以各種熱門的語言提供使用 (例如 Python、R、Julia)。 

以下是 DSVM 上的一些機器學習工具和程式庫。 

## <a name="azure-machine-learning-servicehttpsdocsmicrosoftcomazuremachine-learningserviceoverview-what-is-azure-ml-sdk"></a>[Azure Machine Learning 服務](https://docs.microsoft.com/azure/machine-learning/service/overview-what-is-azure-ml) SDK
|    |           |
| ------------- | ------------- |
| 這是什麼？   |   Azure Machine Learning 服務是一項雲端服務，其可用於開發及部署機器學習模型。  您可以使用 Python SDK 來在建置、定型、調整及管理模型期間進行追蹤。 將模型部署為容器，並在雲端、內部部署或 IoT Edge 上執行。   |
| 支援的 DSVM 版本     | Windows (Conda 環境：AzureML)、Linux (Conda 環境：py36)    |
| 典型的使用案例      | 一般 ML 平台      |
| 它是如何在 DSVM 上設定/安裝的？      |  搭配 GPU 支援安裝   |
| 如何使用/執行它？      | 作為 Python SDK，以及 Azure 命令列工具 (AZ CLI)。 啟動至 Conda 環境 `AzureML` (Windows 版) 或 `py36` (Linux 版)。      |
| 範例的連結      | Jupyter Notebook 範例包含在 `AzureML` 目錄中的 Notebook 下  |
| DSVM 上的相關工具      | Visual Studio Code、Jupyter   |

## <a name="xgboost"></a>XGBoost 
|    |           |
| ------------- | ------------- |
| 這是什麼？   |    快速、可攜且分散式的漸層提升 (GBDT、GBRT 或 GBM) 程式庫，適用於 Python、R、Java、Scala、C++ 等。 可於單一電腦、Hadoop、Spark 上執行    |
| 支援的 DSVM 版本     | Windows、Linux     |
| 典型的使用案例      | 一般 ML 程式庫      |
| 它是如何在 DSVM 上設定/安裝的？      |  搭配 GPU 支援安裝   |
| 如何使用/執行它？      | 以 Python 程式庫 (2.7 和 3.5)、R 套件及路徑上命令列工具 (`C:\dsvm\tools\xgboost\bin\xgboost.exe` (針對 Windows)、`/dsvm/tools/xgboost/xgboost` (針對 Linux))    |
| 範例的連結      | 範例包括在 VM 上，位於 `/dsvm/tools/xgboost/demo` (針對 Linux) 及 `C:\dsvm\tools\xgboost\demo` (針對 Windows)   |
| DSVM 上的相關工具      | LightGBM、MXNet   |



## <a name="vowpal-wabbit"></a>Vowpal Wabbit
|    |           |
| ------------- | ------------- |
| 這是什麼？   |   Vowpal Wabbit (也稱為 "VW") 為開放原始碼的快速核心外學習系統程式庫    |
| 支援的 DSVM 版本     | Windows、Linux     |
| 典型的使用案例      | 一般 ML 程式庫      |
| 它是如何在 DSVM 上設定/安裝的？      |  msi 安裝程式 (針對 Windows)、apt-get (針對 Linux) |
| 如何使用/執行它？      | 以路徑上命令列工具 (`C:\Program Files\VowpalWabbit\vw.exe` (針對 Windows)、`/usr/bin/vw` (針對 Linux))    |
| 範例的連結      | [VowPal Wabbit 範例](https://github.com/JohnLangford/vowpal_wabbit/wiki/Examples) \(英文\) |
| DSVM 上的相關工具      |LightGBM、MXNet、XGBoost   |


## <a name="weka"></a>Weka
|    |           |
| ------------- | ------------- |
| 這是什麼？   |  Weka 是資料採礦工作的機器學習演算法集合。 該演算法可以直接套用於資料集，也可從您自己的 Java 程式碼呼叫。 Weka 包含資料前置處理、分類、迴歸、群集、關聯規則和視覺效果的工具。 |
| 支援的 DSVM 版本     | Windows、Linux     |
| 典型的使用案例      | 一般 ML 工具     |
| 如何使用/執行它？      | 在 Windows 上，在 [開始] 功能表中搜尋 Weka。 在 Linux 上，登入 X2Go，然後巡覽至 [應用程式] -> [開發] -> [Weka]。 |
| 範例的連結      | [Weka 範例](http://www.cs.waikato.ac.nz/ml/weka/documentation.html) \(英文\) |
| DSVM 上的相關工具      |LightGBM、Rattle、XGBooost   |

## <a name="rattle"></a>Rattle
|    |           |
| ------------- | ------------- |
| 這是什麼？   |   使用 R 進行資料採礦的圖形化使用者介面   |
| 支援的 DSVM 版本     | Windows、Linux     |
| 典型的使用案例      | 適用於 R 的一般 UI 資料採礦工具    |
| 如何使用/執行它？      | UI 工具。 在 Windows 上，啟動命令提示字元，執行 R，然後在 R 內執行 `rattle()`。 在 Linux 上，以 X2Go 進行連線，啟動終端機，執行 R，然後在 R 內執行 `rattle()`。 |
| 範例的連結      | [Rattle](https://togaware.com/onepager/) \(英文\) |
| DSVM 上的相關工具      |LightGBM、Weka、XGBoost   |

## <a name="lightgbm"></a>LightGBM
|    |           |
| ------------- | ------------- |
| 這是什麼？   | 以決策樹演算法為基礎的一種快速、分散式的高效能漸層提升 (GBDT、GBRT、GBM 或 MART) 架構，用於進行排序、分類及許多其他的機器學習工作。    |
| 支援的 DSVM 版本      | Windows、Linux    |
| 典型的使用案例      | 一般用途的漸層提升架構      |
| 它是如何在 DSVM 上設定/安裝的？      | 在 Windows 上，LightGBM 是以 Python 套件的形式安裝。 在 Linux 上，命令列可執行檔位於 `/opt/LightGBM/lightgbm`，R 套件和 Python 套件皆已安裝。     |
| 範例的連結      | [LightGBM 指南](https://github.com/Microsoft/LightGBM/tree/master/examples/python-guide) \(英文\)   |
| DSVM 上的相關工具      | MXNet、XgBoost  |

## <a name="h2o"></a>H2O
|    |           |
| ------------- | ------------- |
| 這是什麼？   | 支援記憶體內部、分散式、快速且可擴充機器學習的開放原始碼 AI 平台  |
| 支援的 DSVM 版本      | Linux   |
| 典型的使用案例      | 一般用途的分散式、可擴充 ML   |
| 它是如何在 DSVM 上設定/安裝的？      | H2O 是安裝在 `/dsvm/tools/h2o` 中。      |
| 如何使用/執行它？      | 使用 X2Go 連線至 VM。 啟動新的終端機並執行 `java -jar /dsvm/tools/h2o/current/h2o.jar`。 然後啟動網頁瀏覽器，並連線至 `http://localhost:54321`。      |
| 範例的連結      | 範例可於 VM 上，於 Jupyter 的 `h2o` 目錄底下取得。      |
| DSVM 上的相關工具      | Apache Spark、MXNet、XGBoost、Sparkling Water、Deep Water    |

DSVM 上有數個其他 ML 程式庫 (例如熱門的 `scikit-learn` 套件) 是屬於安裝在 DSVM 上之 Anaconda Python 散發的一部分。 請務必執行相對應的套件管理員，來查看以 Python、R 及 Julia 提供的套件清單。 
