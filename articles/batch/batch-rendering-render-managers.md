---
title: Azure Batch 轉譯管理員支援
description: 使用 Azure 進行轉譯 (使用 Azure Batch 轉譯管理員整合)
services: batch
author: mscurrell
ms.author: markscu
ms.date: 08/02/2018
ms.topic: conceptual
ms.openlocfilehash: 798b2b457016856662f392af25d987788f73c242
ms.sourcegitcommit: 387d7edd387a478db181ca639db8a8e43d0d75f7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/10/2018
ms.locfileid: "40036705"
---
# <a name="using-azure-batch-with-render-farm-managers"></a>使用 Azure Batch 搭配轉譯伺服陣列管理員

如果您使用現有內部部署轉譯伺服陣列，則轉譯管理員很有可能會控制轉譯伺服陣列容量和轉譯作業。

Azure 會為受歡迎的轉譯管理員提供內建支援或附加元件。 您可以接著新增及移除 Azure VM，包括具有按使用付費授權的 VM 以及低優先順序的 VM。

支援下列轉譯器管理員：

* [PipelineFX Qube!](https://www.pipelinefx.com/)
* [Royal Render](http://www.royalrender.de/)
* [Thinkbox Deadline](https://deadline.thinkboxsoftware.com/)

## <a name="using-azure-with-pipelinefx-qube"></a>使用 Azure 搭配 PipelineFX Qube

在 [GitHub 存放庫](https://github.com/Azure/azure-qube)中可取得讓 Azure Batch 集區 VM 能作為 Qube 背景工作角色的相關指令碼和指示。

## <a name="using-azure-with-royal-render"></a>使用 Azure 搭配 Royal Render

Royal Render 已內建 Azure 和 Azure Batch 整合功能，可讓您使用以 Azure 為基礎的 VM 擴充轉譯伺服陣列。 如需摘要，請參閱[說明檔](http://www.royalrender.de/help8/index.html?Cloudrendering.html)。

如需使用 Azure 整合的 Royal Render 客戶範例，請參閱 [Jellyfish Pictures 客戶案例](https://customers.microsoft.com/en-gb/story/jellyfishpictures)。

## <a name="using-azure-with-thinkbox-deadline"></a>使用 Azure 搭配 Thinkbox Deadline

在 [GitHub 存放庫](https://github.com/Azure/azure-deadline)中可取得讓 Azure Batch 集區 VM 能作為 Deadline 從屬的相關指令碼和指示。

## <a name="next-steps"></a>後續步驟

使用 GitHub 上適當的外掛程式與指示，試用轉譯管理員的 Azure Batch 整合。