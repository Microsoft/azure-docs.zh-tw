---
title: Azure 上適用於資料科學虛擬機器的語言 | Microsoft Docs
description: Azure 上適用於資料科學虛擬機器的語言
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
ms.openlocfilehash: 411729155f5135c7e45588b69995274c9cac1315
ms.sourcegitcommit: 9cdd83256b82e664bd36991d78f87ea1e56827cd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/16/2018
ms.locfileid: "31418310"
---
# <a name="languages-supported-on-the-data-science-virtual-machine"></a>資料科學虛擬機器上所支援的語言 

資料科學虛擬機器 (DSVM) 具有數種預先建置的語言及開發工具，可用來建置您的 AI 應用程式。 以下為幾個較主要的項目。 

## <a name="python-windows-server-2016-edition"></a>Python (Windows Server 2016 版)

|    |           |
| ------------- | ------------- |
| 支援的語言版本 | 2.7 和 3.6 |
| 支援的 DSVM 版本      | Windows Server 2016     |
| 它是如何在 DSVM 上設定/安裝的？  | 系統會建立兩個全域 `conda` 環境。 <br /> *  位於 `/anaconda/` 的 `root` 環境是 Python 3.6。 <br/> *  位於 `/anaconda/envs/python2` 的 `python2` 環境是 Python 2.7       |
| 範例的連結      | 已包含適用於 Python 的範例 Jupyter 筆記本     |
| DSVM 上的相關工具      | PySpark、R、Julia      |

> [!NOTE]
> 在 2018 年 3 月之前建立的 Windows Server 2016 包含 Python 3.5 和 Python 2.7。 此外，Python 2.7 是 conda **root** 環境，而 **py35** 則是 Python 3.5 環境。 

### <a name="how-to-use--run-it"></a>如何使用/執行它？    

* 於命令提示字元中執行

開啟命令提示字元，並根據您所要執行的 Python 版本執行下列命令。 

```
# To run Python 2.7
activate python2
python --version

# To run Python 3.6
activate 
python --version

```
* 在 IDE 中使用

使用安裝於 Visual Studio Community 版本的「適用於 Visual Studio 的 Python 工具」(PTVS)。 Python 3.6 是 PTVS 中預設唯一會自動設定的環境。 

> [!NOTE]
> 若要將 PTVS 指向 Python 2.7，您必須在 PTVS 中建立自訂環境。 若要在 Visual Studio Community 版本中設定此環境路徑，請瀏覽至 [工具] -> [Python 工具] -> [Python 環境]，然後按一下 [+ 自訂]。 然後將位置設定為 `c:\anaconda\envs\python2`，並按一下 [自動偵測]。 

* 在 Jupyter 中使用

開啟 Jupyter 並按一下 `New` 按鈕，以建立新的筆記本。 此時，您可以針對 Python 3.6 將核心類型選擇為 [Python [Conda Root]]，並針對 Python 2.7 環境將核心類型選擇為 [Python [Conda env:python2]]。 

* 安裝 Python 套件

DSVM 上的預設 Python 環境為全域環境，可由所有使用者讀取。 但只有系統管理員可以寫入/安裝全域套件。 若要將套件安裝至全域環境，請以系統管理員身分使用 `activate` 命令來啟動至 root 或 python2 環境。 接著，您便可以使用如 `conda` 或 `pip` 等套件管理員來安裝或更新套件。 

## <a name="python-linux-and-windows-server-2012-edition"></a>Python (Linux 和 Windows Server 2012 版)

|    |           |
| ------------- | ------------- |
| 支援的語言版本 | 2.7 和 3.5 |
| 支援的 DSVM 版本      | Linux、Windows Server 2012    |
| 它是如何在 DSVM 上設定/安裝的？  | 系統會建立兩個全域 `conda` 環境。 <br /> * 位於 `/anaconda/` 的 `root` 環境是 Python 2.7。 <br/> * 位於 `/anaconda/envs/py35` 的 `py35` 環境是 Python 3.5       |
| 範例的連結      | 已包含適用於 Python 的範例 Jupyter 筆記本     |
| DSVM 上的相關工具      | PySpark、R、Julia      |
### <a name="how-to-use--run-it"></a>如何使用/執行它？    

**Linux**
* 在終端機中執行

開啟終端機，並根據您所要執行的 Python 版本執行下列命令。 

```
# To run Python 2.7
source activate 
python --version

# To run Python 3.5
source activate py35
python --version

```
* 在 IDE 中使用

使用安裝於 Visual Studio Community 版本的 PyCharm。 

* 在 Jupyter 中使用

開啟 Jupyter 並按一下 `New` 按鈕，以建立新的筆記本。 此時，您可以針對 Python 2.7 將核心類型選擇為 [Python [Conda Root]]，並針對 Python 3.5 環境將核心類型選擇為 [Python [Conda env:py35]]。 

* 安裝 Python 套件

DSVM 上的預設 Python 環境為全域環境，可由所有使用者讀取。 但只有系統管理員可以寫入/安裝全域套件。 若要將套件安裝置全域環境，請以系統管理員或具有 sudo 權限的使用者身分使用 `source activate` 命令來啟動至 root 或 py35 環境。 接著，您便可以使用如 `conda` 或 `pip` 等套件管理員來安裝或更新套件。 

**Windows 2012**
* 於命令提示字元中執行

開啟命令提示字元，並根據您所要執行的 Python 版本執行下列命令。 

```
# To run Python 2.7
activate 
python --version

# To run Python 3.5
activate py35
python --version

```
* 在 IDE 中使用

使用安裝於 Visual Studio Community 版本的「適用於 Visual Studio 的 Python 工具」(PTVS)。 Python 2.7 是 PTVS 中唯一會自動設定的環境。 
> [!NOTE]
> 若要將 PTVS 指向 Python 3.5，您必須在 PTVS 中建立自訂環境。 若要在 Visual Studio Community 版本中設定此環境路徑，請瀏覽至 [工具] -> [Python 工具] -> [Python 環境]，然後按一下 [+ 自訂]。 然後將位置設定為 `c:\anaconda\envs\py35`，並按一下 [自動偵測]。 

* 在 Jupyter 中使用

開啟 Jupyter 並按一下 `New` 按鈕，以建立新的筆記本。 此時，您可以針對 Python 2.7 將核心類型選擇為 [Python [Conda Root]]，並針對 Python 3.5 環境將核心類型選擇為 [Python [Conda env:py35]]。 

* 安裝 Python 套件

DSVM 上的預設 Python 環境為全域環境，可由所有使用者讀取。 但只有系統管理員可以寫入/安裝全域套件。 若要將套件安裝置全域環境，請以系統管理員身分使用 `activate` 命令來啟動至 root 或 py35 環境。 接著，您便可以使用如 `conda` 或 `pip` 等套件管理員來安裝或更新套件。 

## <a name="r"></a>R

|    |           |
| ------------- | ------------- |
| 支援的語言版本 | Microsoft R Open 3.x (與 CRAN-R 100% 相容)<br /> Microsoft R Server 9.x Developer 版本 (可擴充且符合企業需求的 R 平台)|
| 支援的 DSVM 版本      | Linux、Windows     |
| 它是如何在 DSVM 上設定/安裝的？  | Windows：`C:\Program Files\Microsoft\ML Server\R_SERVER` <br />Linux：` /usr/lib64/microsoft-r/3.3/lib64/R`    |
| 範例的連結      | 已包含適用於 R 的範例 Jupyter 筆記本     |
| DSVM 上的相關工具      | SparkR、Python、Julia      |
### <a name="how-to-use--run-it"></a>如何使用/執行它？    

**Windows**：

* 於命令提示字元中執行

開啟命令提示字元並輸入 `R`。

* 在 IDE 中使用

使用安裝於 Visual Studio Community 版本或 RStudio 的「Visual Studio R 工具」(RTVS)。 這些可從開始功能表上或透過桌面圖示存取。 

* 在 Jupyter 中使用

開啟 Jupyter 並按一下 `New` 按鈕，以建立新的筆記本。 此時，您可以將核心類型選擇為 [R] 以使用 Jupyter R 核心 (IRKernel)。 

* 安裝 R 套件

R 是安裝在 DSVM 上可由所有使用者讀取的全域環境中。 但只有系統管理員可以寫入/安裝全域套件。 若要將套件安裝置全域環境，請使用上述其中一種方法執行 R。 接著，您便可以執行 R 套件管理員 `install.packages()` 來安裝或更新套件。 

**Linux**：

* 在終端機中執行

開啟終端機並執行 `R`。  

* 在 IDE 中使用

使用安裝在 Linux DSVM 上的 RStudio。  

* 在 Jupyter 中使用

開啟 Jupyter 並按一下 `New` 按鈕，以建立新的筆記本。 此時，您可以將核心類型選擇為 [R] 以使用 Jupyter R 核心 (IRKernel)。 

* 安裝 R 套件

R 是安裝在 DSVM 上可由所有使用者讀取的全域環境中。 但只有系統管理員可以寫入/安裝全域套件。 若要將套件安裝置全域環境，請使用上述其中一種方法執行 R。 接著，您便可以執行 R 套件管理員 `install.packages()` 來安裝或更新套件。 


## <a name="julia"></a>Julia

|    |           |
| ------------- | ------------- |
| 支援的語言版本 | 0.6 |
| 支援的 DSVM 版本      | Linux、Windows     |
| 它是如何在 DSVM 上設定/安裝的？  | Windows：安裝於 `C:\JuliaPro-VERSION`<br /> Linux：安裝於 `/opt/JuliaPro-VERSION`    |
| 範例的連結      | 已包含適用於 Julia 的範例 Jupyter 筆記本     |
| DSVM 上的相關工具      | Python、R      |
### <a name="how-to-use--run-it"></a>如何使用/執行它？    

**Windows**：

* 於命令提示字元中執行

開啟命令提示字元並執行 `julia`。 
* 在 IDE 中使用

使用 `Juno`，這是安裝在 DSVM 上並可透過桌面捷徑使用的 Julia IDE。

* 在 Jupyter 中使用

開啟 Jupyter 並按一下 `New` 按鈕，以建立新的筆記本。 此時，您可以將核心類型選擇為 `Julia VERSION` 

* 安裝 Julia 套件

預設的 Julia 位置為可由所有使用者讀取的全域環境。 但只有系統管理員可以寫入/安裝全域套件。 若要將套件安裝置全域環境，請使用上述其中一種方法執行 Julia。 接著，您便可以執行如 `Pkg.add()` 的 Julia 套件管理員命令來安裝或更新套件。 


**Linux**：
* 在終端機中執行。

開啟終端機並執行 `julia`。 
* 在 IDE 中使用

使用 `Juno`，這是安裝在 DSVM 上並可透過應用程式功能表捷徑使用的 Julia IDE。

* 在 Jupyter 中使用

開啟 Jupyter 並按一下 `New` 按鈕，以建立新的筆記本。 此時，您可以將核心類型選擇為 `Julia VERSION` 

* 安裝 Julia 套件

預設的 Julia 位置為可由所有使用者讀取的全域環境。 但只有系統管理員可以寫入/安裝全域套件。 若要將套件安裝置全域環境，請使用上述其中一種方法執行 Julia。 接著，您便可以執行如 `Pkg.add()` 的 Julia 套件管理員命令來安裝或更新套件。 

## <a name="other-languages"></a>其他語言

**C#**：可於 Windows 上使用，並可透過 Visual Studio Community 版本存取，或是在可直接執行 `csc` 命令的 `Developer Command Prompt for Visual Studio` 上存取。 

**Java**：OpenJDK 可於 DSVM 的 Linux 和 Windows 版本上使用，並已設定於路徑上。 您可以在 Windows 中的命令提示字元，或 Linux 中的 Bash 殼層上輸入 `javac` 或 `java` 命令來使用 Java。 

**node.js**：: node.js 可於 DSVM 的 Linux 和 Windows 版本上使用，並已設定於路徑上。 您可以在 Windows 中的命令提示字元，或 Linux 中的 Bash 殼層上輸入 `node` 或 `npm` 命令來使用 node.js。 在 Windows 上，系統會安裝「適用於 Visual Studio 的 Node.js 工具」擴充功能，以提供圖形化的 IDE 來開發 node.js 應用程式。 

**F#**：可於 Windows 上使用，並可透過 Visual Studio Community 版本存取，或是在可直接執行 `fsc` 命令的 `Developer Command Prompt for Visual Studio` 上存取。 


