---
title: 安裝 Azure Data Lake Tools for Visual Studio
description: 本文說明如何安裝 Azure Data Lake Tools for Visual Studio。
services: data-lake-analytics
ms.service: data-lake-analytics
author: saveenr
ms.author: saveenr
ms.reviewer: jasonwhowell
ms.assetid: ad8a6992-02c7-47d4-a108-62fc5a0777a3
ms.topic: conceptual
ms.date: 05/03/2018
ms.openlocfilehash: e731f28ab416de7da30f41fe8a154108b8097b89
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43041793"
---
# <a name="install-data-lake-tools-for-visual-studio"></a>安裝 Data Lake Tools for Visual Studio

了解如何使用 Visual Studio 建立 Azure Data Lake Analytics 帳戶、在 [U-SQL](data-lake-analytics-u-sql-get-started.md) 中定義作業，以及將作業提交至 Data Lake Analytics 服務。 如需有關 Data Lake Analytics 的詳細資訊，請參閱 [Azure Data Lake Analytics 概觀](data-lake-analytics-overview.md)。

## <a name="prerequisites"></a>必要條件

* **Visual Studio**：支援 Express 以外的所有版本。
    * Visual Studio 2017
    * Visual Studio 2015
    * Visual Studio 2013
* **Microsoft Azure SDK for .NET** 2.7.1 版或更新版本。  使用 [Web Platform Installer](http://www.microsoft.com/web/downloads/platform.aspx) 來進行安裝。
* **Data Lake Analytics** 帳戶。 如需建立帳戶，請參閱[使用 Azure 入口網站開始使用 Azure Data Lake Analytics](data-lake-analytics-get-started-portal.md)。

## <a name="install-azure-data-lake-tools-for-visual-studio-2017"></a>安裝 Azure Data Lake Tools for Visual Studio 2017

Visual Studio 2017 15.3 或更新版本不支援 Azure Data Lake Tools for Visual Studio。 在 Visual Studio 安裝程式中，此工具屬於**資料儲存和處理**與 **Azure 開發**工作負載的一部分。 安裝 Visual Studio 時，請啟用其中一個工作負載。  

啟用**資料儲存和處理**工作負載，如下所示：![啟用資料儲存和處理工作負載](./media/data-lake-analytics-data-lake-tools-get-started/data-lake-tools-for-vs-2017-install-01.png)

啟用 **Azure 開發**工作負載，如下所示：![啟用 Azure 開發工作負載](./media/data-lake-analytics-data-lake-tools-get-started/data-lake-tools-for-vs-2017-install-02.png)

## <a name="install-azure-data-lake-tools-for-visual-studio-2013-and-2015"></a>安裝 Azure Data Lake Tools for Visual Studio 2013 與 2015

[從下載中心](http://aka.ms/adltoolsvs)下載並安裝 Azure Data Lake Tools for Visual Studio。 安裝後，請注意：
* **伺服器總管** > **Azure** 節點包含 **Data Lake Analytics** 節點。 
* [工具] 功能表包含 [Data Lake] 項目。


## <a name="next-steps"></a>後續步驟
* 若要記錄診斷資訊，請參閱 [為 Azure Data Lake Analytics 存取診斷記錄檔](data-lake-analytics-diagnostic-logs.md)
* 若要了解更複雜的查詢，請參閱 [使用 Azure Data Lake Analytics 來分析網站記錄檔](data-lake-analytics-analyze-weblogs.md)。
* 若要使用頂點執行檢視，請參閱[在 Data Lake Tools for Visual Studio 中使用頂點執行檢視](data-lake-analytics-data-lake-tools-use-vertex-execution-view.md)
