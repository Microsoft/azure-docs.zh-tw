---
title: 資料探索和虛擬化工具 - Azure | Microsoft Docs
description: 資料科學虛擬機器的資料探索和視覺效果工具。
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
ms.openlocfilehash: df29d0a55317d06d656d8444c6bd7754c6c955eb
ms.sourcegitcommit: 9cdd83256b82e664bd36991d78f87ea1e56827cd
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/16/2018
ms.locfileid: "31407209"
---
# <a name="data-exploration-and-visualization-tools-on-the-data-science-virtual-machine"></a>資料科學虛擬機器上的資料探索和視覺效果工具

資料科學的關鍵步驟是了解資料。 視覺效果和資料探索工具有助於加速資料了解。 以下是 DSVM 提供且有利於進行此關鍵步驟的一些工具。 

## <a name="apache-drill"></a>Apache 深入探詢
|    |           |
| ------------- | ------------- |
| 這是什麼？   | 巨量資料的開放原始碼 SQL 查詢引擎    |
| 支援的 DSVM 版本      | Windows、Linux  |
| 它是如何在 DSVM 上設定/安裝的？      |  僅在 `/dsvm/tools/drill*` 內嵌模式中安裝   |
| 典型的使用案例      |  不需要 ETL 的原位資料探索。 查詢不同資料來源和格式，包括 CSV、JSON、關聯式資料表、Hadoop     |
| 如何使用/執行它？      | 桌面捷徑  <br/> [在 10 分鐘內開始使用深入探詢](https://drill.apache.org/docs/drill-in-10-minutes/)  |
| DSVM 上的相關工具      |   Rattle、Weka、SQL Server Management Studio      |

## <a name="weka"></a>Weka
|    |           |
| ------------- | ------------- |
| 這是什麼？   |  Weka 是資料採礦工作的機器學習演算法集合。 該演算法可以直接套用於資料集，也可從您自己的 Java 程式碼呼叫。 Weka 包含資料前置處理、分類、迴歸、群集、關聯規則和視覺效果的工具。 |
| 支援的 DSVM 版本     | Windows、Linux     |
| 典型的使用案例      | 一般 ML 工具     |
| 如何使用/執行它？      | 在 Windows 上，在 [開始] 功能表中搜尋 Weka。 在 Linux 上，登入 X2Go，然後巡覽至 [應用程式] -> [開發] -> [Weka]。 |
| 範例的連結      | [Weka 範例](http://www.cs.waikato.ac.nz/ml/weka/documentation.html) \(英文\) |
| DSVM 上的相關工具      |LightGBM、Rattle、Xgboost   |

## <a name="rattle"></a>Rattle
|    |           |
| ------------- | ------------- |
| 這是什麼？   |   使用 R 進行資料採礦的圖形化使用者介面   |
| 支援的 DSVM 版本     | Windows、Linux     |
| 典型的使用案例      | 適用於 R 的一般 UI 資料採礦工具    |
| 如何使用/執行它？      | UI 工具。 在 Windows 上，啟動命令提示字元，執行 R，然後在 R 內執行 `rattle()`。 在 Linux 上，以 X2Go 進行連線，啟動終端機，執行 R，然後在 R 內執行 `rattle()`。 |
| 範例的連結      | [Rattle](https://togaware.com/onepager/) \(英文\) |
| DSVM 上的相關工具      |LightGBM、Weka、Xgboost   |

## <a name="powerbi-desktop"></a>PowerBI Desktop 
|    |           |
| ------------- | ------------- |
| 這是什麼？   | 互動式資料視覺效果和 BI 工具    |
| 支援的 DSVM 版本      | Windows  |
| 典型的使用案例      |  資料視覺效果和建立儀表板   |
| 如何使用/執行它？      | 桌面捷徑 (`C:\Program Files\Microsoft Power BI Desktop\bin\PBIDesktop.exe`)      |
| DSVM 上的相關工具      |   Visual Studio 2017、Visual Studio Code、Juno      |

