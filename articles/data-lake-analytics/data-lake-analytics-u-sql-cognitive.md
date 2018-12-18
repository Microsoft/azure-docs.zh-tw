---
title: 使用 Azure Data Lake Analytics 中的 U-SQL 辨識功能
description: 了解如何使用 U-SQL 中辨識功能的智慧
services: data-lake-analytics
author: saveenr
ms.author: saveenr
ms.reviewer: jasonwhowell
ms.assetid: 019c1d53-4e61-4cad-9b2c-7a60307cbe19
ms.service: data-lake-analytics
ms.topic: conceptual
ms.date: 06/05/2018
ms.openlocfilehash: 550a57b1dc90256fb2d94e270aba2f79f7b0d370
ms.sourcegitcommit: 616e63d6258f036a2863acd96b73770e35ff54f8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/14/2018
ms.locfileid: "45603068"
---
# <a name="get-started-with-the-cognitive-capabilities-of-u-sql"></a>開始使用 U-SQL 的辨識功能

## <a name="overview"></a>概觀
U-SQL 的辨識功能讓開發人員可以在其公司的巨量資料程式中使用 put 智慧。 

下列使用認知功能的範例可供使用：
* 影像處理：[偵測臉部](https://github.com/Azure-Samples/usql-cognitive-imaging-ocr-hello-world)
* 影像處理：[偵測情緒](https://github.com/Azure-Samples/usql-cognitive-imaging-emotion-detection-hello-world)
* 影像處理：[偵測物件 (標記)](https://github.com/Azure-Samples/usql-cognitive-imaging-object-tagging-hello-world)
* 影像處理：[OCR (光學字元辨識)](https://github.com/Azure-Samples/usql-cognitive-imaging-ocr-hello-world)
* 文字：[關鍵片語擷取與情感分析](https://github.com/Azure-Samples/usql-cognitive-text-hello-world)

## <a name="registering-cognitive-extensions-in-u-sql"></a>在 U-SQL 中註冊認知擴充功能
開始之前，請依照此文章中的步驟，在 U-SQL 中註冊認知擴充功能：[在 U-SQL 中註冊認知擴充功能](https://msdn.microsoft.com/azure/data-lake-analytics/u-sql/cognitive-capabilities-in-u-sql#registeringExtensions)。

## <a name="next-steps"></a>後續步驟
* [U-SQL/認知範例](https://github.com/Azure-Samples?utf8=✓&q=usql%20cognitive)
* [使用 Data Lake Tools for Visual Studio 開發 U-SQL 指令碼](data-lake-analytics-data-lake-tools-get-started.md)
* [針對 Azure 資料湖分析工作使用 U-SQL 視窗函式](data-lake-analytics-use-window-functions.md)
