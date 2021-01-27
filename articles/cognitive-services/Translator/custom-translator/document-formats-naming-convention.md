---
title: 文件的格式和命名慣例 - 自訂翻譯工具
titleSuffix: Azure Cognitive Services
description: 本文說明自訂翻譯工具中的文件格式和命名慣例。 這個概念有助於更妥善地管理檔案名稱，並避免命名衝突。
author: laujan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.date: 08/17/2020
ms.author: lajanuar
ms.topic: conceptual
ms.openlocfilehash: 9598d3c1d339d1c37b02892cc83164acae447434
ms.sourcegitcommit: 100390fefd8f1c48173c51b71650c8ca1b26f711
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/27/2021
ms.locfileid: "98895844"
---
# <a name="document-formats-and-naming-convention-guidance"></a>文件格式和命名慣例指引

用於自訂翻譯的任何檔案都至少須包含 **四個** 字元。

下表包含所有可供您自行建置翻譯系統的支援檔案格式：

| 格式            | 延伸模組   | 描述                                                                                                                                                                                                                                                                    |
|-------------------|--------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| XLIFF             | .XLF、.XLIFF | 平行文件格式，翻譯記憶體系統的匯出。 使用的語言定義於檔案內。                                                                                                                                                              |
| TMX               | .TMX         | 平行文件格式，翻譯記憶體系統的匯出。 使用的語言定義於檔案內。                                                                                                                                                              |
| ZIP               | .ZIP         | ZIP 是封存檔案格式。                                                                                                                                                                                                        |
| Locstudio         | .LCL         | Microsoft 的平行文件格式                                                                                                                                                                                                                                      |
| Microsoft Word    | .DOCX        | Microsoft Word 文件                                                                                                                                                                                                                                                        |
| Adobe Acrobat     | .PDF         | Adobe Acrobat 可攜式文件                                                                                                                                                                                                                                                |
| HTML              | .HTML、.HTM  | HTML 文件                                                                                                                                                                                                                                                                  |
| 文字檔         | .TXT         | UTF-16 或 UTF-8 編碼的文字檔。 檔案名不得包含日文字元。                                                                                                                                                                                        |
| 對齊的文字檔 | .ALIGN       | 副檔名 `.ALIGN` 是一種特殊的副檔名，可供您在確知文件組中的句子已適當對齊時使用。 如果您提供 `.ALIGN` 檔案，自訂翻譯工具就不會為您對齊句子。 |
| Excel 檔案        | .XLSX        | Excel 檔案 (2013 或更新版本)。 試算表的第一行/列應為語言代碼。                                                                                                                                                                                                                                                      |

## <a name="dictionary-formats"></a>字典格式

針對字典，自訂翻譯支援定型集支援的所有檔案格式。 如果您使用的是 Excel 字典，試算表的第一行/列應為語言代碼。

## <a name="zip-file-formats"></a>ZIP 檔案格式

文件可分組到單一 ZIP 檔案中並上傳。 自訂翻譯工具支援 ZIP 檔案格式 (ZIP、GZ 和 TGZ)。

副檔名為 TXT、HTML、HTM、PDF、DOCX、ALIGN 的 ZIP 檔案中所含的每份文件都必須遵循下列命名慣例：

{文件名稱}\_{語言代碼}，其中，{文件名稱} 是您的文件名稱，{語言代碼} 是 ISO LanguageID (兩個字元)，表示文件包含該語言的句子。 語言代碼前面必須要有底線 (_)。

例如，若要為英文翻譯成西班牙文的系統上傳 ZIP 檔案內的兩個平行文件，檔案應命名為 “data_en” 和 “data_es”。

翻譯記憶體檔案 (TMX、XLF、XLIFF、LCL、XLSX) 不需要遵循特定語言的命名慣例。  

## <a name="next-steps"></a>後續步驟

- 了解[專案](workspace-and-project.md#what-is-a-custom-translator-project)以建立及管理專案。
