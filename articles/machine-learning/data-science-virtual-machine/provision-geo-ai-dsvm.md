---
title: 在 Azure 上佈建地區人工智慧虛擬機器 - Azure | Microsoft Docs
description: 如何在 Azure 上佈建地區人工智慧虛擬機器。
keywords: 深度學習, AI, 資料科學工具, 資料科學虛擬機器, 地理空間分析
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
ms.date: 03/05/2018
ms.author: gokuma
ms.openlocfilehash: 93dfe6594aeaf45a6905fe8cb55c98dd37cc9599
ms.sourcegitcommit: 9cdd83256b82e664bd36991d78f87ea1e56827cd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/16/2018
ms.locfileid: "31408603"
---
# <a name="provision-a-geo-artificial-intelligence-virtual-machine-on-azure"></a>在 Azure 上佈建地區人工智慧虛擬機器 

地區人工智慧資料科學虛擬機器 (Geo-DSVM) 是熱門 [Azure 資料科學虛擬機器](http://aka.ms/dsvm)的擴充功能，特別設定來合併人工智慧與地理空間分析。 VM 中的地理空間分析是由 [ArcGIS Pro](https://www.arcgis.com/features/index.html) 驅動。 資料科學 VM 可以對藉由地理資訊而更加豐富的資料，快速訓練機器學習模型，甚至是深度學習模型。 只在 Windows 2016 DSVM 上提供支援。 

Geo-DSVM 包含數個人工智慧工具，包括：

- GPU 版本的熱門深度學習架構，像是 Microsoft Cognitive Toolkit、TensorFlow、Keras、Caffe2、Chainer； 
- 用來取得和前置處理影像、文字資料的工具， 
- 適用於開發活動的工具，例如 Microsoft R Server Developer Edition、Anaconda Python、Python 和 R 適用的 Jupyter Notebook、Python 和 R 適用的 IDE、SQL 資料庫
- ESRI 的 ArcGIS Pro 桌面軟體，以及可與來自人工智慧應用程式之地理空間資料搭配運作的 Python 和 R 介面。 


## <a name="create-your-geo-ai-data-science-vm"></a>建立您的地區人工智慧資料科學 VM

以下是建立地區人工智慧資料科學 VM 執行個體的程序： 


1. 瀏覽至 [Azure 入口網站](https://ms.portal.azure.com/#create/microsoft-ads.geodsvmwindows)上的虛擬機器清單。
2. 選取底部的 [建立] 按鈕以進入精靈。
![create-geo-ai-dsvm](./media/provision-geo-ai-dsvm/Create-Geo-AI.png)
3. 針對此圖右邊列舉的**四個步驟**，用來建立 Geo-DSVM 的精靈會要求**輸入**每個步驟。 以下是設定每個步驟所需的輸入：



   - **基本概念**

      1. **名稱**：您建立的資料科學伺服器名稱。

      2. **使用者名稱**：系統管理員帳戶登入識別碼。

      3. **密碼**：系統管理員帳戶密碼。

      4. **訂用帳戶**：如果您有多個訂用帳戶，請選取要用來建立機器和開立帳單的訂用帳戶。

      5. **資源群組**：您可以建立新的群組，或使用訂用帳戶中的**空白**現有 Azure 資源群組。

      6. **位置**：選取最適合的資料中心。 它通常是擁有您大部分的資料或者是最接近您的實際位置以取得最快速度的網路存取的資料中心。 如果您需要對 GPU 進行深度學習，您必須在 Azure 中選擇具有 NC 系列 GPU VM 執行個體的其中一個位置。 目前具有 GPU VM 的位置有：**美國東部、美國中北部、美國中南部、美國西部 2、北歐、西歐**。 如需最新的清單，請參閱[依據區域的 Azure 產品分頁](https://azure.microsoft.com/regions/services/)，並且在 [計算] 底下尋找 **NC 系列**。 


   - **設定**：如果您打算在 Geo DSVM 的 GPU 上執行深度學習，請選取其中一個 NC 系列 GPU 虛擬機器大小。 否則，您可以選擇其中一個以 CPU 為基礎的執行個體。  為您的 VM 建立儲存體帳戶。 
   
   - **摘要**：請確認您輸入的所有資訊都正確無誤。

   - **購買**：按一下 [購買] 以開始佈建。 系統會提供服務條款的連結。 VM 除了計算您在 [大小]  步驟中所選擇的伺服器大小之外，不會收取任何其他費用。 

>[!NOTE]
> 佈建大約需要 20-30 分鐘。 在 Azure 入口網站中會顯示佈建的狀態。


## <a name="how-to-access-the-geo-ai-data-science-virtual-machine"></a>如何存取地區人工智慧資料科學虛擬機器

一旦建立 VM，您就可以開始使用其上安裝和預先設定的工具。 我們為許多工具提供了開始功能表磚和桌面圖示。 您可以使用您在前面的 [基本] 區段中設定的系統管理員帳戶認證從遠端桌面登入 VM。 


## <a name="using-arcgis-pro-installed-in-the-vm"></a>使用 VM 中安裝的 ArcGIS Pro

Geo-DSVM 已經預先安裝 ArcGIS Pro 桌面及預先設定環境，以搭配 DSVM 中的所有工具運作。 當您啟動 ArcGIS 時，它會提示您登入您的 ArcGIS 帳戶。 如果您已經有 ArcGIS 帳戶且具有軟體的授權，您可以使用現有的認證。  

![Arc-GIS-Logon](./media/provision-geo-ai-dsvm/ArcGISLogon.png)

否則，您可以註冊新的 ArcGIS 帳戶和授權，或者取得[免費試用](https://www.arcgis.com/features/free-trial.html)。 

![ArcGIS-Free-Trial](./media/provision-geo-ai-dsvm/ArcGIS-Free-Trial.png)

一旦註冊付費或免費試用 ArcGIS 帳戶，即可藉由遵循[開始使用 ArcGIS Pro 文件](http://www.esri.com/library/brochures/getting-started-with-arcgis-pro.pdf)中的指示，為您的帳戶授權 ArcGIS Pro。 

以您的 ArcGIS 帳戶登入 ArcGIS Pro 桌面之後，您就可以針對地理空間分析和機器學習專案，開始使用在 VM 上安裝及設定的資料科學工具。

## <a name="next-steps"></a>後續步驟

利用下列主題中的指引開始使用地區人工智慧資料科學 VM：

* [使用地區人工智慧資料科學 VM](use-geo-ai-dsvm.md)