---
title: Azure Functions 執行階段概觀 | Microsoft Docs
description: Azure Functions 執行階段預覽的概觀
services: functions
documentationcenter: ''
author: apwestgarth
manager: stefsch
editor: ''
ms.assetid: ''
ms.service: functions
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: multiple
ms.topic: article
ms.date: 11/28/2017
ms.author: anwestg
ms.openlocfilehash: 2be04c7037dd26755300cf8b7794678a6a958278
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38488455"
---
# <a name="azure-functions-runtime-overview-preview"></a>Azure Functions 執行階段概觀 (預覽)

Azure Functions 執行階段 (預覽) 提供新的方法，讓您在內部部署利用 Azure Functions 程式設計模型的簡易性和彈性。 與 Azure Functions 一樣，Azure Functions 執行階段以相同的開放原始碼根源為基礎，並於內部部署，提供與雲端服務幾乎完全相同的開發經驗。

![Azure Functions 執行階段預覽入口網站][1]

Azure Functions 執行階段可讓您先體驗 Azure Functions，再移轉至雲端。 如此一來，您建置的程式碼資產可以隨著您一起移轉至雲端。  此執行階段也提供新的選項，例如，在夜間使用內部部署電腦閒置的計算能力，執行批次程序。 您也可以使用組織內的裝置，依情況將資料傳送至其他系統，包括在內部部署和雲端。

Azure Functions 執行階段包含兩個部分︰

* Azure Functions 執行階段管理角色
* Azure Functions 執行階段背景工作角色

## <a name="azure-functions-management-role"></a>Azure Functions 管理角色

Azure Functions 管理角色提供主機，讓您在內部部署管理您的 Functions。 此角色執行下列作業：

* 裝載 Azure Functions 管理入口網站，也就是您在 [Azure 入口網站](https://portal.azure.com)中看到的同一個入口網站。 就像在 Azure 入口網站中一樣，該入口網站提供一致的體驗，讓您以相同的方式開發函式。
* 將函式分配給多個 Functions 背景工作角色。
* 提供發佈端點，讓您透過下載和匯入發行設定檔，直接從 Microsoft Visual Studio 發佈函式。

## <a name="azure-functions-worker-role"></a>Azure Functions 背景工作角色

Azure Functions 背景工作角色部署在 Windows 容器中，您的函式程式碼就在此處執行。  您可以將多個背景工作角色部署到整個組織中，而此選項也是讓客戶利用閒置計算能力的主要方法。  閒置計算會存在多個組織中的一個範例是，機器電源始終保持開啟，但機器不會長時間使用。

## <a name="minimum-requirements"></a>最低需求

若要開始使用 Azure Functions 執行階段，您必須有一部機器已安裝 Windows Server 2016 或 Windows 10 Creators Update 且可存取 SQL Server 執行個體。

## <a name="next-steps"></a>後續步驟

安裝 [Azure Functions 執行階段預覽](https://aka.ms/azafrdoc)

<!--Image references-->
[1]: ./media/functions-runtime-overview/AzureFunctionsRuntime_Portal.png
