---
title: 適用於 Azure Machine Learning 的開發環境 | Microsoft Docs
description: 您可以搭配 Azure Machine Learning 服務使用的開發環境概觀。 開發環境的唯一需求是 Python 3，但建議您也使用 Conda 環境。 針對開發工具，我們建議您使用 Jupyter Notebooks、Azure Notebooks 與 IDE/程式碼編輯器。
services: machine-learning
author: rastala
ms.author: roastala
manager: cgronlun
ms.service: machine-learning
ms.reviewer: larryfr
ms.topic: conceptual
ms.date: 9/24/2018
ms.openlocfilehash: 260e209bcf00396ee545851684038578c4fd148a
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46971037"
---
# <a name="development-environment-for-azure-machine-learning"></a>Azure Machine Learning 的開發環境 

了解您可以搭配 Azure Machine Learning 服務使用的開發環境。 

Azure Machine Learning 服務不限使用任何平台，而且不需要特定的開發環境。 它需要 __Python 3__，而且我們建議您也使用 __Conda__ 來建立隔離的環境。 __如果您已有符合這些需求的開發環境__，可以略過此文件，並前往[設定開發環境](how-to-configure-environment.md)文件。

此文件的其餘部分會討論建議使用的開發環境：

* __Jupyter Notebook__
* __Azure Notebooks__
* __整合式開發環境 (IDE) 與程式碼編輯器__
* __資料科學虛擬機器__

因為 Notebook 與 IDE 都可以擴充，因此要比較這些環境相當困難。 例如，有些 IDE 可用來作為 Jupyter Notebook 的用戶端。 一般而言，__Notebook__ 是專為__互動式實驗__與__視覺效果__而設計。 __IDE 與程式碼編輯器__提供的工具可__改善程式碼品質__及__與外部系統整合__，例如版本控制。

> [!TIP]
> 請採用兩種都能使用的環境。 Notebook 和 IDE 都只是工具，並可以視需要混用。 例如，您可能會在 Notebook 中開始實驗，然後匯出至 Python 指令碼，在 IDE 中編輯和偵錯。
>
> 這就是資料科學虛擬機器提供 Jupyter Notebook 和數種熱門的 Python IDE 的原因。

## <a name="jupyter-notebooks"></a>Jupyter Notebook

Jupyter Notebook 是 [Jupyter 專案](https://jupyter.org/)的一部分。 他們致力於提供互動式程式碼撰寫體驗，讓您用來建立混合即時程式碼與敘述文字和圖形的文件。 您可以在各種不同的平台上安裝 Jupyter Notebook。

安裝您自己的 Jupyter Notebook ，可讓您安裝並設定所需的環境。 不過，您要自行維護系統。

## <a name="azure-notebooks"></a>Azure Notebooks

[Azure Notebooks](https://notebooks.azure.com) (預覽) 是 Azure 雲端中的 Notebook 環境。 它以 Jupyter 為基礎，並提供已預先載入常用程式庫的環境。 Azure Machine Learning SDK 已安裝在 Azure Notebooks 中，因此您幾乎不需要設定就可以開始實驗。

Azure Notebooks 也可以簡化共用 Notebook 的程序。 您可以在分享 Notebook 的 URL、將程式庫設定為公用，或從 Azure Notebooks 介面分享到社交媒體。

Azure Notebooks 的缺點是，您對環境沒有完整的控制權，而且不能安裝所需的自訂軟體。 它也是共用環境，因此您的 Notebook 執行速度可能會比專用的 Jupyter Notebook 安裝更慢。

## <a name="ides-and-code-editors"></a>IDE 與程式碼編輯器

許多 IDE 與程式碼編輯器都能搭配 Azure Machine Learning 使用。 唯一的軟體需求是 Azure Machine Learning SDK，此 SDK 可以使用安裝 `pip` 公用程式來安裝。

建議您使用 [Visual Studio Code](https://code.visualstudio.com/)，因為它可提供可搭配 Python 語言與 Azure Machine Learning 使用的工具。 Linux、macOS 與 Windows 平台均適用。

## <a name="data-science-virtual-machine"></a>資料科學虛擬機器

資料科學虛擬機器 (DSVM) 是先前環境的組合。 它是 Azure 平台上的 VM，該平台已預先安裝 Jupyter Notebook、Visual Studio Code 與 Azure Machine Learning SDK。 建立 VM 會比 Azure Notebooks 更複雜，但會比從頭開始設定機器較不複雜。 因為 VM 映像中已預先安裝必要的軟體，所以建立 VM 之後，您就可以很快開始使用 Azure Machine Learning 進行實驗。

DSVM 可讓您選取所需的計算資源，例如 CPU、 GPU 與記憶體。 它也已預先安裝如 PyCharm 等其他編輯器，以及 TensorFlow、Keras 與 PyTorch 等常用機器學習軟體。 如果您所需的軟體未安裝，您也可以自行安裝。

如需預先安裝軟體的詳細資訊，請參閱[什麼是資料科學虛擬機器](../data-science-virtual-machine/overview.md)文件。

## <a name="next-steps"></a>後續步驟

既然您已了解開發環境的相關資訊，再來可了解[如何設定開發環境](how-to-configure-environment.md)。

