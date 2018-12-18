---
title: Batch 轉譯應用程式
description: 預先安裝的 Batch 轉譯應用程式
services: batch
author: mscurrell
ms.author: markscu
ms.date: 08/02/2018
ms.topic: conceptual
ms.openlocfilehash: 28acd1b7275694d38a52f14d2b2c32b79cc8183e
ms.sourcegitcommit: 387d7edd387a478db181ca639db8a8e43d0d75f7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/10/2018
ms.locfileid: "40036749"
---
# <a name="pre-installed-applications-on-rendering-vm-images"></a>在轉譯的 VM 映像上預先安裝的應用程式

您可以透過 Azure Batch 使用任何轉譯應用程式。 不過，此外也有預先安裝了通用應用程式的 Azure Marketplace VM 映像可供使用。

情況允許時，預先安裝的轉譯應用程式即適用按使用次數付費的授權。 在 Batch 集區建立時，可以指定必要的應用程式，且 VM 和應用程式的成本都會按分鐘計費。 應用程式的價格會列在 [Azure Batch 定價頁面](https://azure.microsoft.com/pricing/details/batch/#graphic-rendering)上。

有些應用程式僅支援 Windows，但大部分都是 Windows 和 Linux 均支援。

## <a name="applications-on-centos-7-rendering-nodes"></a>CentOS 7 轉譯節點上的應用程式

* Autodesk Maya I/O 2017 Update 5 (201708032230 版)
* Autodesk Maya I/O 2018 Update 2 (201711281015 版)
* Autodesk Arnold for Maya 2017 (Arnold 5.0.1.1 版) MtoA-2.0.1.1-2017
* Autodesk Arnold for Maya 2018 (Arnold 5.0.1.4 版) MtoA-2.1.0.3-2018
* Chaos Group V-Ray for Maya 2017 (3.60.04 版)
* Chaos Group V-Ray for Maya 2018 (3.60.04 版)
* Blender (2.68)

## <a name="applications-on-windows-server-2016-rendering-nodes"></a>Windows Server 2016 轉譯節點上的應用程式

* Autodesk Maya I/O 2017 Update 5 (17.4.5459 版)
* Autodesk Maya I/O 2018 Update 3 (18.3.0.7040 版)  
* Autodesk 3ds Max I/O 2019 Update 1 (21.10.1314 版)
* Autodesk 3ds Max I/O 2018 Update 4 (20.4.0.4254 版)
* Autodesk Arnold for Maya (Arnold 5.0.1.1 版) MtoA-2.0.1.1-2017
* Autodesk Arnold for Maya (Arnold 5.0.1.4 版) MtoA-2.0.2.3-2018
* Autodesk Arnold for 3ds Max (Arnold 5.0.2.4 版) (1.2.926 版)
* Chaos Group V-Ray for Maya (3.52.03 版)
* Chaos Group V-Ray for 3ds Max (3.60.02 版)
* Blender (2.79)

## <a name="next-steps"></a>後續步驟

若要使用轉譯 VM 映像，則必須在建立集區時在集區組態中指定這些映像；請參閱[用於轉譯的 Batch 集區功能](https://docs.microsoft.com/azure/batch/batch-rendering-functionality#batch-pools)。