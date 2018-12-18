---
title: 包含檔案
description: 包含檔案
services: azure-policy
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 09/18/2018
ms.author: dacoulte
ms.custom: include file
ms.openlocfilehash: c3365450c90c4fda37884e8998fad70f5d164244
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47006467"
---
Azure 原則的每個物件類型都有最大計數。 _範圍_的項目表示訂用帳戶或[管理群組](../articles/governance/management-groups/overview.md)。

| Where | 何事 | 最大計數 |
|---|---|---|
| 影響範圍 | 原則定義 | 250 |
| 影響範圍 | 計畫定義 | 100 |
| 租用戶 | 計畫定義 | 1000 |
| 影響範圍 | 原則/方案指派 | 100 |
| 原則定義 | 參數 | 20 |
| 計畫定義 | 原則 | 100 |
| 計畫定義 | 參數 | 100 |
| 原則/方案指派 | 排除項目 (notScopes) | 100 |
| 原則規則 | 巢狀的條件 | 512 |
