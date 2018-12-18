---
title: Azure Analysis Services 教學課程第 1 課：建立新的表格式模型專案 | Microsoft Docs
description: 說明如何建立新的 Azure Analysis Services 教學課程專案。
author: minewiskan
manager: kfile
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 08/15/2018
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: b9d3e9df62ffd31a6a01ea427fec8d5a3d168bfa
ms.sourcegitcommit: f057c10ae4f26a768e97f2cb3f3faca9ed23ff1b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/17/2018
ms.locfileid: "42142308"
---
# <a name="create-a-tabular-model-project"></a>建立表格式模型專案

在此課程中，您會使用 Visual Studio 搭配 Analysis Services 專案或 SQL Server Data Tools (SSDT)，來建立 1400 相容性層級的新表格式模型專案。 建立新的專案之後，您就可以開始新增資料和製作模型。 本課程也簡介 Visual Studio 中的表格式模型製作環境。  
  
這堂課的預估完成時間：**10 分鐘**  
  
## <a name="prerequisites"></a>必要條件  
本主題是表格式模型製作教學課程的第一課。 若要完成本課程，您需要備齊許多必要條件。 若要深入了解，請參閱 [Azure Analysis Services - Adventure Works 教學課程](../tutorials/aas-adventure-works-tutorial.md)。  
  
## <a name="create-a-new-tabular-model-project"></a>建立新的表格式模型專案  
  
#### <a name="to-create-a-new-tabular-model-project"></a>建立新的表格式模型專案  
  
1.  在 Visual Studio 中，於 [檔案] 功能表上按一下 [新增] > [專案]。  
  
2.  在 [新增專案] 對話方塊中，展開 [已安裝] > [商業智慧] > [Analysis Services]，然後按一下 [Analysis Services 表格式專案]。  
  
3.  在 [名稱] 中，輸入 **AW 網際網路銷售**，然後指定專案檔案的位置。  
  
    根據預設，[解決方案名稱] 會與專案名稱相同。不過，您可以輸入不同的解決方案名稱。  
  
4.  按一下 [確定]。  
  
5.  在 [表格式模型設計師] 對話方塊中，選取 [整合式工作區]。  
  
    在製作模型期間，工作區會裝載與專案同名的表格式模型資料庫。 整合式工作區意味著 Visual Studio 會使用內建的執行個體，不必只為了撰寫模型而安裝個別的 Analysis Services 伺服器執行個體。
      
6.  在 [相容性等級] 中，選取 [SQL Server 2017 / Azure Analysis Services (1400)]。   
 
    ![aas 第 1 課 tmd](../tutorials/media/aas-lesson1-tmd.png)
      
    如果您在 [相容性等級] 清單方塊中沒看到 [SQL Server 2017 / Azure Analysis Services (1400)]，就表示您使用的 SQL Server Data Tools 不是最新版本。 若要取得最新版本，請參閱[安裝 SQL Server Data Tools](https://docs.microsoft.com/sql/ssdt/download-sql-server-data-tools-ssdt)。  
      
  
## <a name="understanding-the-visual-studio-tabular-model-authoring-environment"></a>了解 Visual Studio 表格式模型撰寫環境  
現在，您已建立新的表格式模型專案，讓我們花一點時間瀏覽 Visual Studio 中的表格式模型製作環境。  
  
專案建立之後，會在 Visual Studio 中開啟。 在右側的 [表格式模型總管] 中，您會看到樹狀檢視呈現模型中的物件。 您還沒有匯入資料，所以資料夾是空的。 您可以在物件資料夾上按一下滑鼠右鍵來執行動作，類似於功能表列。 當您逐步執行本教學課程時，您會使用 [表格式模型總管] 來瀏覽模型專案中的不同物件。

![aas 第 1 課 tme](../tutorials/media/aas-lesson1-tme.png)

按一下 [方案總管] 索引標籤。在這裡，您可看到 **Model.bim** 檔案。 如果您在左邊沒看到設計工具視窗 (含 Model.bim 索引標籤的空視窗)，請在 [方案總管] 的 [AW 網際網路銷售專案] 下，按兩下 [Model.bim] 檔案。 Model.bim 檔案包含模型專案的中繼資料。 

![aas 第 1 課 se](../tutorials/media/aas-lesson1-se.png)
  
按一下 [Model.bim]。 在 [屬性] 視窗中，您會看到模型屬性，最重要的是 [DirectQuery 模式] 屬性。 這個屬性可指定以 In-Memory 模式 (關閉) 還是 DirectQuery 模式 (開啟) 來部署模型。 本教學課程中，您會以 In-Memory 模式來製作和部署模型。

![aas 第 1 課屬性](../tutorials/media/aas-lesson1-properties.png)
  
當您建立模型專案時，根據 [工具] 功能表 > [選項] 對話方塊中可指定的 [資料模型化] 設定，自動會設定某些模型屬性。 [資料備份]、[ 工作空間保留] 和 [工作空間伺服器] 屬性可指定如何及在何處備份、在記憶體中保留和建置工作空間資料庫 (您的模型製作資料庫)。 稍後，如有必要，您可以變更這些設定，但現在，讓這些屬性保持不變即可。  

在 [方案總管] 中，以滑鼠右鍵按一下 [AW 網際網路銷售] \(專案)，然後按一下 [屬性]。 [AW 網際網路銷售屬性頁] 對話方塊隨即出現。 稍後部署模型時，您會設定其中一些屬性。  
  
當您安裝 Analysis Services 專案或 SSDT 時，就已將數個新的功能表項目新增至 Visual Studio 環境。 按一下 [模型] 功能表。 從這裡，您可以匯入資料、重新整理工作區資料、在 Excel 中瀏覽模型、建立檢視方塊和角色、選取模型檢視，以及設定計算選項。 按一下 [資料表] 功能表。 從這裡，您可以建立和管理關聯性、指定日期資料表設定、建立分割區，以及編輯資料表屬性。 如果您按一下 [資料行] 功能表，您可以新增和刪除資料表中的資料行、凍結資料行，以及指定排序順序。 Visual Studio 也會在工具列中新增一些按鈕。 「自動加總」功能最實用，可以為選取的資料行建立標準彙總量值。 其他工具列按鈕可讓您快速存取常用的功能和命令。  
  
請瀏覽一些對話方塊和位置，以了解與表格式模型製作有關的各種功能。 雖然某些項目尚無作用，但可讓您更熟悉表格式模型製作環境。  
  

## <a name="whats-next"></a>後續步驟
[第 2 課：取得資料](../tutorials/aas-lesson-2-get-data.md)。

  
  
  
