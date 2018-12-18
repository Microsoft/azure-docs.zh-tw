---
title: 對話學習模組預設設定 - Microsoft 認知服務 | Microsoft Docs
titleSuffix: Azure
description: 了解預設的對話學習模組設定。
services: cognitive-services
author: v-jaswel
manager: nolachar
ms.service: cognitive-services
ms.component: conversation-learner
ms.topic: article
ms.date: 04/30/2018
ms.author: v-jaswel
ms.openlocfilehash: b048509f38356b19d52507f4dfb5bfa61122f9f6
ms.sourcegitcommit: e2ea404126bdd990570b4417794d63367a417856
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/14/2018
ms.locfileid: "45576420"
---
# <a name="default-values-and-boundaries"></a>預設值和界限

本文件說明對話學習模組的預設設定，以及金鑰服務界限。

## <a name="default-configuration"></a>預設組態

參數 | 預設值
--- | --- 
預設工作階段逾時 | 30 分鐘

## <a name="boundaries"></a>界線

參數 | 限制
--- | --- 
撰寫 API，每月的 HTTP 呼叫次數上限 | 5M
撰寫 API，每秒的 HTTP 呼叫次數上限 | 25
工作階段 API，每月的 HTTP 呼叫次數上限 | 500K
工作階段 API，每秒的 HTTP 呼叫次數上限 | 10
每個模型的自訂 (非程式設計) 實體數目上限 | 請參閱 [LUIS 界限文件](https://docs.microsoft.com/azure/cognitive-services/luis/luis-boundaries) \(英文\)；實務上，實際數目可能會稍微低一點
每個模型的預先建置實體數目上限 | 請參閱 [LUIS 界限文件](https://docs.microsoft.com/azure/cognitive-services/luis/luis-boundaries) \(英文\)
每個模型的實體數目上限 (總計) | 100
每個模型的動作數目上限 | 32
每個模型的定型對話數目上限 | 1000
每則訓練對話輪到使用者的次數上限 | 100
每個模型的記錄對話數目上限 | 沒有預先設定的限制，但記錄對話在捨棄之前只會保留一段固定期間。  此外，對話學習模組 UI 一次將顯示 100 則記錄對話。 
每個使用者的模型數目上限 | 沒有預先設定的限制
循序非等候動作的數目上限 | 5 (*)

(*) 在 5 個循序非等候動作之後，所有非等候動作都會加上遮罩，而交談學習模組將在可用的等待動作中做出選擇。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [開始使用對話學習模組](./quickstart.md)
