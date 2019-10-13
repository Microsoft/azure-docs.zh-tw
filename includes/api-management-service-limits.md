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
ms.openlocfilehash: fc945a7e9389c8aec48a6a1dba969fbf92002d3a
ms.sourcegitcommit: cd70273f0845cd39b435bd5978ca0df4ac4d7b2c
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2019
ms.locfileid: "67174382"
---
| Resource | 限制 |
| --- | --- |
| 縮放單位數上限 | 每個區域 10 個<sup>1</sup> |
| 快取大小 | 每個單位 5 GB<sup>2</sup> |
| 每個 HTTP 授權單位的並行後端連線數<sup>3</sup> | 每單位 2048<sup>4</sup> |
| 最大快取回應大小 | 2 MB |
| 原則文件大小上限 | 256 KB<sup>5</sup> | 
| 每個服務執行個體的自訂閘道網域數目上限<sup>6</sup> | 20 |
| 每個服務實例的 CA 憑證數目上限 | 10 | 
| 每個訂用帳戶的服務執行個體數目上限<sup>7</sup> | 20 | 
| 每個服務執行個體的訂用帳戶數目上限<sup>7</sup> | 500 |
| 每個服務執行個體的用戶端憑證數目上限<sup>7</sup> | 50 | 
| 每個服務執行個體的 API 數目上限<sup>7</sup> | 50 | 
| 每個服務執行個體的 API 作業數目上限<sup>7</sup> | 1,000 | 
| 總要求持續時間上限<sup>7</sup> | 30 秒 | 
| 緩衝承載大小上限<sup>7</sup> | 2 MB | 


<sup>1</sup>調整限制取決於定價層。 若要查看定價層和其調整限制，請參閱[API 管理價格](https://azure.microsoft.com/pricing/details/api-management/)。<br/>
<sup>2</sup>每個單位的快取大小取決於定價層。 若要查看定價層和其調整限制，請參閱[API 管理價格](https://azure.microsoft.com/pricing/details/api-management/)。<br/>
<sup>3</sup>除非後端明確關閉，否則連線會進行共用並重複使用。<br/>
<sup>4</sup>這項限制是基本、標準和高階層的每個單位。 開發人員層的限制為1024。 此限制不適用於消費層。<br/> 
<sup>5</sup>此限制適用于基本、標準和高階層。 在消費層中，原則檔案大小限制為 4 KB。<br/>
<sup>6</sup>此資源僅適用于高階層。<br/>
<sup>7</sup>此資源僅適用于消費層。<br/>



