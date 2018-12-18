---
title: Azure Resource Manager 範本範例 - 事件方格 | Microsoft Docs
description: 事件方格的 Azure Resource Manager 範本範例
services: event-grid
author: tfitzmac
manager: timlt
ms.service: event-grid
ms.devlang: na
ms.topic: sample
ms.tgt_pltfrm: na
ms.date: 09/18/2018
ms.author: tomfitz
ms.openlocfilehash: c462334597b41b914b6a0a0e3c8a67ad97e2bd7a
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46295575"
---
# <a name="azure-resource-manager-templates-for-event-grid"></a>事件方格的 Azure Resource Manager 範本

下表包含事件方格的 Azure Resource Manager 範本連結。

| | |
|-|-|
|**事件方格訂用帳戶**||
| [使用 WebHook 端點的自訂主題和訂用帳戶](https://github.com/Azure/azure-quickstart-templates/tree/master/101-event-grid)| 部署事件方格自訂主題。 為使用 WebHook 端點的自訂主題建立訂用帳戶。 |
| [使用 EventHub 端點的自訂主題訂用帳戶](https://github.com/Azure/azure-quickstart-templates/tree/master/101-event-grid-event-hubs-handler)| 為自訂主題建立「事件方格」訂閱帳戶。 訂用帳戶會使用端點的事件中樞。 |
| [Azure 訂用帳戶或資源群組訂用帳戶](https://github.com/Azure/azure-quickstart-templates/tree/master/101-event-grid-resource-events-to-webhook)| 訂閱資源群組或 Azure 訂用帳戶的事件。 在部署期間指定為目標的資源群組為事件來源。 訂用帳戶會使用端點的 WebHook。 |
| [Blob 儲存體帳戶和訂用帳戶](https://github.com/Azure/azure-quickstart-templates/tree/master/101-event-grid-subscription-and-storage)| 部署 Azure Blob 儲存體帳戶及訂閱該儲存體帳戶的事件。 |
| | |
