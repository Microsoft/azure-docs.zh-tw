---
title: 包含檔案
description: 包含檔案
services: api-management
author: vladvino
ms.assetid: 1b813833-39c8-46be-8666-fd0960cfbf04
ms.service: api-management
ms.topic: include
ms.date: 03/22/2018
ms.author: vlvinogr
ms.custom: include file
ms.openlocfilehash: b40e2ba448236bcef72ffe501b4b14fa0dd64e1c
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38755530"
---
| 資源 | 限制 |
| --- | --- |
| 級別單位 | 每個區域 10 個<sup>1</sup> |
| 快取 | 每個單位 5 GB<sup>1</sup> |
| 每個 HTTP 授權單位的並行後端連線數目<sup>2</sup> | 每個單位 2048 個<sup>3</sup> |
| 最大快取回應大小 | 10MB |
| 原則文件大小上限 | 256KB |
| 最大自訂閘道網域數目 | 每個服務執行個體 20 個<sup>4</sup> |


<sup>1</sup>每個定價層的 API 管理限制不同。 若要查看定價層和調整限制，請移至 [API 管理價格](https://azure.microsoft.com/pricing/details/api-management/)。
<sup>2</sup> 除非後端明確關閉連線，否則連線會經過集區化和重複使用。
<sup>3</sup> 基本、標準和高階層的每個單位。 開發人員層的限制為 1024 個。
<sup>4</sup> 只有高階層提供。


