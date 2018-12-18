---
title: Azure 匯入/匯出工具匯入作業命令的快速參考 | Microsoft Docs
description: 適用於常用匯入作業命令的 Azure 匯入/匯出工具命令參考。
author: muralikk
services: storage
ms.service: storage
ms.topic: article
ms.date: 01/15/2017
ms.author: muralikk
ms.component: common
ms.openlocfilehash: 8f3eb2bf2d9789b678849f9e2415816d15afc29e
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39526625"
---
# <a name="quick-reference-for-frequently-used-commands-for-import-jobs"></a>匯入作業的常用命令快速參考

本文章提供一些常用命令的快速參考。 如需詳細使用方式，請參閱[針對匯入作業準備硬碟](../storage-import-export-tool-preparing-hard-drives-import.md)。

## <a name="first-session"></a>第一個工作階段

```
WAImportExport.exe PrepImport /j:JournalTest.jrn /id:session#1 /sk:************* /InitialDriveSet:driveset-1.csv /DataSet:dataset-1.csv /logdir:F:\logs
```

## <a name="second-session"></a>第二個工作階段

```
WAImportExport.exe PrepImport /j:JournalTest.jrn /id:session#2 /DataSet:dataset-2.csv
```

## <a name="abort-latest-session"></a>中止最近的工作階段

```
WAImportExport.exe PrepImport /j:JournalTest.jrn /id:session#2 /AbortSession
```

## <a name="resume-latest-interrupted-session"></a>繼續最近中斷的工作階段

```
WAImportExport.exe PrepImport /j:JournalTest.jrn /id:session#3 /ResumeSession
```

## <a name="add-drives-to-latest-session"></a>將磁碟機加入最近的工作階段

```
WAImportExport.exe PrepImport /j:JournalTest.jrn /id:session#3 /AdditionalDriveSet:driveset-2.csv
```

## <a name="next-steps"></a>後續步驟

* [針對匯入作業準備硬碟的簡單工作流程](storage-import-export-tool-sample-preparing-hard-drives-import-job-workflow.md)
