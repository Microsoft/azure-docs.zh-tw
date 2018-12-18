---
title: 開發適用於 Azure Stack 的應用程式 | Microsoft Docs
description: 在 Azure Stack 上建立原型應用程式的開發考量
services: azure-stack
documentationcenter: ''
author: sethmanheim
manager: femila
editor: ''
ms.assetid: d3ebc6b1-0ffe-4d3e-ba4a-388239d6cdc3
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/15/2018
ms.author: sethm
ms.reviewer: ''
ms.openlocfilehash: a6f89f9a7e5960e4749c14fc9a4adb648f6781f4
ms.sourcegitcommit: d2f2356d8fe7845860b6cf6b6545f2a5036a3dd6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/16/2018
ms.locfileid: "41946583"
---
# <a name="develop-for-azure-stack"></a>開發 Azure Stack

*適用於：Azure Stack 整合系統和 Azure Stack 開發套件*

即使您沒有 Azure Stack 環境的存取權，您也可以現在就開始開發應用程式。 因為 Azure Stack 會傳遞在您資料中心執行的 Microsoft Azure 服務，所以您可以使用類似工具和流程針對 Azure Stack 進行開發，就如同您為 Azure 進行開發一樣。 只需要一些準備並使用下列主題中的指導，您就可以使用 Azure 來模擬 Azure Stack 環境。

* 在 Azure 中，您可以建立可部署至 Azure Stack 的 Azure Resource Manager 範本。 請參閱[範本考量](azure-stack-develop-templates.md)以取得開發範本的指導並確保可攜性。
* 在 Azure 和 Azure Stack 之間的服務可用性和服務版本存在差異。 您可以使用 [Azure Stack 原則模組](azure-stack-policy-module.md)來限制 Azure Stack 可用的 Azure 服務可用性和資源型別。 限制服務可確保應用程式會依賴 Azure Stack 可用的服務。
* [Azure Stack 快速入門範本](https://github.com/Azure/AzureStack-QuickStart-Templates)是常見的情節範例，會說明如何開發可部署至 Azure 和 Azure Stack 的範本。
