---
title: 比較 Microsoft 的機器學習產品選項 - Azure | Microsoft Docs
description: 比較 Microsoft 各種用來建置、部署及管理機器學習模型的不同產品。 決定您要為解決方案選擇哪些產品。
services: machine-learning
ms.service: machine-learning
ms.component: core
ms.topic: overview
ms.reviewer: jmartens
author: garyericson
ms.author: garye
ms.date: 09/24/2018
ms.openlocfilehash: 182504373795b3cb0f2794acbed5e253ac6bc95c
ms.sourcegitcommit: 3150596c9d4a53d3650cc9254c107871ae0aab88
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47419554"
---
# <a name="what-are-the-machine-learning-product-options-from-microsoft"></a>Microsoft 有哪些機器學習產品選項？

Microsoft 提供了多種用來建置、部署及管理機器學習模型的產品選項。 請比較這些產品，然後選擇可讓您最有效率地開發機器學習解決方案的產品。

| 機器學習產品 | 內容 | 產品用途 |
|-|-|-|
| 在雲端 | | |
| [Azure Machine Learning 服務](#azure-machine-learning-services) | 適用於 ML 的受控雲端服務  | 使用 Python 和 CLI 在 Azure 中訓練、部署及管理模型 |
| [Azure Machine Learning Studio](#azure-machine-learning-studio) | ML 的拖放功能視覺介面 | 使用預先設定的演算法來建置、實驗及部署模型 |
| [Azure Databricks](#azure-databricks) | 以 Spark 為基礎的分析平台 | 建置及部署模型和資料工作流程 |
| [Azure 認知服務](#azure-cognitive-services) | 預先建置了 AI 和 ML 模型的 Azure 服務 | 輕鬆地將智慧型功能新增至您的應用程式 |
| [Azure 資料科學虛擬機器](#azure-data-science-virtual-machine) | 預先安裝了資料科學工具的虛擬機器 | 在預先設定的環境中開發 ML 解決方案 |
| 內部部署 | | |
| [SQL Server Machine Learning 服務](#sql-server-machine-learning-services) | 內嵌在 SQL 中的分析引擎 | 在 SQL Server 內建置及部署模型 |
| [Microsoft Machine Learning Server](#microsoft-machine-learning-server) | 適用於預測分析的獨立企業伺服器 | 使用 R 和 Python 建置和部署模型 |
| 開發人員工具 | | |
| [ML.NET](#mlnet) | 開放原始碼的跨平台工具 ML SDK | 開發適用於 .NET 應用程式的 ML 解決方案 |
| [Windows ML](#windows-ml) | Windows 10 ML 平台 | 在 Windows 10 裝置上評估已定型的模型 |

## <a name="azure-machine-learning-service"></a>Azure Machine Learning 服務

[Azure Machine Learning 服務](overview-what-is-azure-ml.md) (預覽) 是完全受控的雲端服務，可用來大規模地訓練、部署及管理 ML 模型。 此服務可完整支援開放原始碼技術，因此您可以使用數以萬計的開放原始碼 Python 套件，例如 TensorFlow、PyTorch 與 scikit-learn。 此外也有齊備的工具 (例如 [Azure Notebooks](https://notebooks.azure.com/)、[Jupyter Notebooks](http://jupyter.org) 或 [Visual Studio Code Tools for AI](https://visualstudio.microsoft.com/downloads/ai-tools-vscode/)) 可供您輕鬆地瀏覽和轉換資料，然後訓練和部署模型。 Azure Machine Learning 服務包含自動產生模型的功能，並可讓您輕鬆、有效率且正確地進行調整。

使用 Azure Machine Learning 服務，可利用 Python 和 CLI 在雲端上大規模地訓練、部署及管理 ML 模型。

>[!Note]
> 您可以免費試用 Azure Machine Learning。 無需信用卡或 Azure 訂用帳戶。 立即開始使用。 https://azure.microsoft.com/free/

## <a name="azure-machine-learning-studio"></a>Azure Machine Learning Studio

[Azure Machine Learning Studio](../studio/what-is-ml-studio.md) 提供互動式的視覺化工作區，可讓您使用預先建置的機器學習演算法輕鬆快速地建置、測試和部署模型。 Machine Learning Studio 會以 Web 服務方式發佈模型，讓自訂應用程式或 BI 工具 (例如 Excel) 都能夠很容易地使用。
無需任何程式設計 - 您可以連接互動式畫布上的資料集和分析模組以建構機器學習服務模型，然後輕鬆按幾下滑鼠加以部署。

如果您想要直接開發及部署模型而不使用程式碼，請使用 Machine Learning Studio。

[!INCLUDE [machine-learning-free-trial](../../../includes/machine-learning-free-trial.md)]

## <a name="azure-databricks"></a>Azure Databricks

[Azure Databricks](/azure/azure-databricks/what-is-azure-databricks) 是一個針對 Microsoft Azure 雲端服務平台進行最佳化的 Apache Spark 分析平台。 Databricks 可與 Azure 整合，提供一鍵式設定、順暢的工作流程以及互動式的工作區，可讓資料科學家、資料工程師及企業分析師共同作業。
您可以在 Web 型 Notebook 中使用 Python、R、Scala 和 SQL 程式碼，來查詢、視覺化和模型化資料。

如果您想要在 Apache Spark 上共同建置機器學習解決方案，請使用 Databricks。

## <a name="azure-cognitive-services"></a>Azure 認知服務

[Azure 認知服務](/azure/cognitive-services/welcome)是一組 API，可讓您建置使用自然通訊方法的應用程式。 只要幾行程式碼，這些 API 即讓您的應用程式查看、聽取、說出、了解並解譯使用者的需求。 輕鬆地將智慧型功能新增至您的應用程式，例如： 

- 情緒和人氣偵測
- 願景與語音辨識
- Language Understanding (LUIS)
- 知識和搜尋

使用認知服務可開發跨裝置及平台的應用程式。 API 會持續改進，且易於設定。

## <a name="azure-data-science-virtual-machine"></a>Azure 資料科學虛擬機器

[Azure 資料科學虛擬機器](../data-science-virtual-machine/overview.md)是 Microsoft Azure 雲端上的自訂虛擬機器環境，專為進行資料科學建置。 它已預先安裝和預先設定許多常用的資料科學和其他工具，以開始建置智慧應用程式進行進階分析。
資料科學虛擬機器適用於 Windows 和 Linux Ubuntu 的版本 (Linux CentOS 不支援 Azure Machine Learning 服務)。
如需特定版本資訊及其內含項目的清單，請參閱 [Azure 資料科學虛擬機器簡介](../data-science-virtual-machine/overview.md)。
目前支援以資料科學虛擬機器作為 Azure Machine Learning 服務的目標。

當您需要在單一節點上執行或裝載您的作業時，請使用資料科學 VM。 或者，如果您需要從遠端相應增加在單一機器上的處理。

## <a name="sql-server-machine-learning-services"></a>SQL Server Machine Learning 服務

[SQL Server Microsoft Machine Learning 服務](https://docs.microsoft.com/sql/advanced-analytics/r/r-services)針對 SQL Server 資料庫中的關聯式資料新增了採用 R 和 Python 的統計分析、資料視覺效果和預測分析。 Microsoft 的 R 和 Python 程式庫包含進階模型化和 ML 演算法，可在 SQL Server 中以平行方式大規模地執行。

如果您在 SQL Server 中的關聯式資料需要內建的 AI 和預測分析，請使用 SQL Server Machine Learning 服務。

## <a name="microsoft-machine-learning-server"></a>Microsoft Machine Learning Server

[Microsoft Machine Learning 伺服器](https://docs.microsoft.com/machine-learning-server/what-is-machine-learning-server)是企業伺服器，用來裝載及管理 R 及 Python 處理程序的並行和分散式工作負載。 Microsoft Machine Learning 伺服器可在 Linux、Windows、Hadoop 和 Apache Spark 上執行，也可在 [HDInsight](https://azure.microsoft.com/services/hdinsight/r-server/) 中使用。 它可為使用 [RevoScaleR](https://docs.microsoft.com/machine-learning-server/r-reference/revoscaler/revoscaler)、[revoscalepy](https://docs.microsoft.com/machine-learning-server/python-reference/revoscalepy/revoscalepy-package) 和 [MicrosoftML 套件](https://docs.microsoft.com/r-server/r/concept-what-is-the-microsoftml-package)建置的解決方案提供執行引擎，並藉由對高效能分析、統計分析、機器學習和大型資料集的支援，來擴充開放原始碼 R 和 Python。 此功能會透過隨著伺服器安裝的專屬套件而提供。 對於開發，您可以使用 IDE，例如 [Visual Studio R 工具](https://www.visualstudio.com/vs/rtvs/)和[適用於 Visual Studio 的 Python 工具](https://www.visualstudio.com/vs/python/)。

如果您需要建置及運算化在伺服器上使用 R 和 Python 建置的模型，或是在 Hadoop 或 Spark 叢集上大規模散發 R 和 Python 訓練，請使用 Microsoft Machine Learning 伺服器。

## <a name="mlnet"></a>ML.NET

[ML.NET](https://docs.microsoft.com/dotnet/machine-learning/) 是免費、開放原始碼且跨平台的機器學習架構，可讓您建立自訂的機器學習解決方案，並將其整合到您的 .NET 應用程式中。

如果您想要將機器學習解決方案整合到您的 .NET 應用程式中，請使用 ML.NET。

## <a name="windows-ml"></a>Windows ML

[Windows ML](https://docs.microsoft.com/windows/uwp/machine-learning/) 可讓您在應用程式中使用定型的機器學習模型，在 Windows 10 裝置的本機上評估已定型的模型。

如果您想要在 Windows 應用程式中使用定型的機器學習模型，請使用 Windows ML。

## <a name="next-steps"></a>後續步驟

- 若要了解 Microsoft 所提供的所有人工智慧 (AI) 開發產品，請參閱 [Microsoft AI 平台](https://www.microsoft.com/ai)
- 如需有關於開發 AI 解決方案的教學，請參閱 [Microsoft AI School](https://aischool.microsoft.com/learning-paths)
