---
title: 包含檔案
description: 包含檔案
services: storage
author: wmgries
ms.service: storage
ms.topic: include
ms.date: 07/18/2018
ms.author: wgries
ms.custom: include file
ms.openlocfilehash: 3f70a8cf2df25f487de7cd1a8c8cbdf9431839f0
ms.sourcegitcommit: f94f84b870035140722e70cab29562e7990d35a3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43283059"
---
| 資源 | 目標 | 固定限制 |
|----------|--------------|------------|
| 每個訂用帳戶的儲存體同步服務 | 15 個儲存體同步服務 | 否 |
| 每個儲存體同步服務的同步群組 | 100 個同步群組 | 是 |
| 每個儲存體同步服務的已註冊伺服器 | 99 部伺服器 | 是 |
| 每個同步群組的雲端端點 | 1 個雲端端點 | 是 |
| 每個同步群組的伺服器端點 | 50 個伺服器端點 | 否 |
| 每部伺服器的伺服器端點 | 33-99 個伺服器端點 | 是，但會隨著組態 (CPU、記憶體、磁碟區、檔案變換、檔案計數等) 而有所不同 |
| 端點大小 | 4 TiB | 否 |
| 每個同步群組的檔案系統物件 (目錄和檔案) | 2500 萬個物件 | 否 |
| 目錄中的檔案系統物件 (目錄和檔案) 數目上限 | 200,000 個物件 | 是 |
| 物件 (目錄和檔案) 名稱長度上限 | 255 個字元 | 是 |
| 物件 (目錄和檔案) 安全性描述元大小上限 | 4 KiB | 是 |
| 檔案大小 | 100 GiB | 否 |
| 要分層之檔案的檔案大小下限 | 64 KiB | 是 |
