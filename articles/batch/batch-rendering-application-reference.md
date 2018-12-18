---
title: 透過 Azure Batch 使用轉譯應用程式
description: 如何透過 Azure Batch 使用轉譯應用程式
services: batch
author: mscurrell
ms.author: markscu
ms.date: 08/02/2018
ms.topic: conceptual
ms.openlocfilehash: 500246dc98618aead11ba539ce4485d25ac62941
ms.sourcegitcommit: 387d7edd387a478db181ca639db8a8e43d0d75f7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/10/2018
ms.locfileid: "40036742"
---
# <a name="rendering-applications"></a>轉譯應用程式

轉譯應用程式可藉由建立 Batch 作業和工作來使用。 工作命令列屬性會指定適當的命令列和參數。  要建立作業工作，最簡單的方式是使用[這篇文章](https://docs.microsoft.com/azure/batch/batch-rendering-using#using-batch-explorer)中指定的 Batch Explorer 範本。  您可以檢視範本，並視需要修改已建立的版本。

本文將簡單說明如何執行每個轉譯應用程式。

## <a name="rendering-with-autodesk-3ds-max"></a>使用 Autodesk 3ds Max 進行轉譯

### <a name="renderer-support"></a>轉譯器支援

除了 3ds Max 的內建轉譯器以外，下列轉譯器也可用於轉譯的 VM 映像，並且可供 3ds Max 場景檔案參考：

* Autodesk Arnold
* Chaos Group V-Ray

### <a name="task-command-line"></a>工作命令列

叫用 `3dsmaxcmdio.exe` 應用程式以執行集區節點的命令列轉譯。  當工作執行時，此應用程式會位於路徑上。 `3dsmaxcmdio.exe` 應用程式具有與 `3dsmaxcmd.exe` 應用程式相同的可用參數，如 [3ds Max 說明文件](https://help.autodesk.com/view/3DSMAX/2018/ENU/) (「轉譯 | 命令列轉譯」一節) 所說明。

例如︰

```
3dsmaxcmdio.exe -v:5 -rfw:0 -start:{0} -end:{0} -bitmapPath:"%AZ_BATCH_JOB_PREP_WORKING_DIR%\sceneassets\images" -outputName:dragon.jpg -w:1280 -h:720 "%AZ_BATCH_JOB_PREP_WORKING_DIR%\scenes\dragon.max"
```

注意：

* 請謹慎處理，以確保可以找到資產檔案。  請使用 [資產追蹤] 視窗或在命令列上使用 `-bitmapPath` 參數，以確定路徑的正確性和相對性。
* 在工作執行時，請檢查 3ds Max 所寫入的 `stdout.txt` 檔案，以確認轉譯是否有任何問題，例如找不到資產。

### <a name="batch-explorer-templates"></a>Batch Explorer 範本

您可以從 Batch Explorer 中的**資源庫**存取集區和作業範本。  在 [GitHub 上的 Batch Explorer 資料存放庫](https://github.com/Azure/BatchExplorer-data/tree/master/ncj/3dsmax)中可以找到範本來源檔案。

## <a name="rendering-with-autodesk-maya"></a>使用 Autodesk Maya 進行轉譯

### <a name="renderer-support"></a>轉譯器支援

除了 Maya 的內建轉譯器以外，下列轉譯器也可用於轉譯的 VM 映像，並且可供 3ds Max 場景檔案參考：

* Autodesk Arnold
* Chaos Group V-Ray

### <a name="task-command-line"></a>工作命令列

在工作命令列中會使用 `renderer.exe` 命令列轉譯器。 命令列轉譯器的相關資訊載於 [Maya 說明中](http://help.autodesk.com/view/MAYAUL/2018/ENU/?guid=GUID-EB558BC0-5C2B-439C-9B00-F97BCB9688E4)。

在下列範例中，會使用作業準備工作將場景檔案和資產複製到作業準備工作目錄、使用輸出資料夾來儲存轉譯的映像，並轉譯畫面格 10。

```
render -renderer sw -proj "%AZ_BATCH_JOB_PREP_WORKING_DIR%" -verb -rd "%AZ_BATCH_TASK_WORKING_DIR%\output" -s 10 -e 10 -x 1920 -y 1080 "%AZ_BATCH_JOB_PREP_WORKING_DIR%\scene-file.ma"
```

進行 V-Ray 轉譯時，Maya 場景檔案通常會將 V-Ray 指定為轉譯器。  它也可指定於命令列上：

```
render -renderer vray -proj "%AZ_BATCH_JOB_PREP_WORKING_DIR%" -verb -rd "%AZ_BATCH_TASK_WORKING_DIR%\output" -s 10 -e 10 -x 1920 -y 1080 "%AZ_BATCH_JOB_PREP_WORKING_DIR%\scene-file.ma"
```

進行 Arnold 轉譯時，Maya 場景檔案通常會將 Arnold 指定為轉譯器。  它也可指定於命令列上：

```
render -renderer arnold -proj "%AZ_BATCH_JOB_PREP_WORKING_DIR%" -verb -rd "%AZ_BATCH_TASK_WORKING_DIR%\output" -s 10 -e 10 -x 1920 -y 1080 "%AZ_BATCH_JOB_PREP_WORKING_DIR%\scene-file.ma"
```

### <a name="batch-explorer-templates"></a>Batch Explorer 範本

您可以從 Batch Explorer 中的**資源庫**存取集區和作業範本。  在 [GitHub 上的 Batch Explorer 資料存放庫](https://github.com/Azure/BatchExplorer-data/tree/master/ncj/maya)中可以找到範本來源檔案。

## <a name="next-steps"></a>後續步驟

透過 Batch Explorer 使用 [GitHub 中的資料存放庫](https://github.com/Azure/BatchExplorer-data/tree/master/ncj)所包含的集區和作業範本。  必要時請建立新的範本，或修改其中一個提供的範本。