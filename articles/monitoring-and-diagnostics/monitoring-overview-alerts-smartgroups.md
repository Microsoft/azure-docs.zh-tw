---
title: 智慧群組
description: 智慧群組是警示的彙總，可協助您減少警示干擾
author: anantr
services: azure-monitor
ms.service: azure-monitor
ms.topic: conceptual
ms.date: 05/15/2018
ms.author: anantr
ms.component: alerts
ms.openlocfilehash: 58a66cfdfaada34981ef166635b0d04e2e6e8f61
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46976871"
---
# <a name="smart-groups"></a>智慧群組
處理警示時所面臨的一個常見挑戰是篩選干擾以找出真正重要的事情 - 智慧群組的目的就是在解決該問題。  

智慧群組是使用機器學習演算法自動建立的，結合了代表單一問題的相關警示。  當警示建立時，演算法會根據歷史模式、相似的屬性和相似的結構等資訊，將警示加入至新的智慧群組或現有的智慧群組。 例如，如果訂用帳戶中的數個虛擬機上的 % CPU 同時出現突增，並導致許多單獨警示，而且如果此類警示在過去任何時間曾經一起發生，則這些警示可能會分組到一個智慧群組中，表示有可能的共同根本原因。 這表示，對疑難排解警示的人來說，智慧群組不僅可讓他們以單一彙總單元來管理相關警示以減少干擾，也能引導他們了解其警示的可能常見根本原因。

目前，演算法只會考慮訂用帳戶之內相同監視器服務的警示。 智慧群組可以透過此彙總減少高達 99% 的警示干擾。 您可以檢視智慧群組詳細資料頁面中的群組包含警示的原因。

您可以檢視智慧群組的詳細資料，並以類似您設定警示的方式來設定狀態。 每個警示都是單一專屬智慧群組的成員。 

## <a name="smart-group-state"></a>智慧群組狀態
智慧型群組狀態會是類似警示狀態的概念，可讓您在智慧群組層級上管理解析流程。 與警示狀態類似，當建立智慧群組時，其狀態有 [新] 狀態，此狀態可變更為 [已認可] 或 [已關閉]。

支援下列智慧群組狀態。

| State | 說明 |
|:---|:---|
| 新增 | 已經偵測到問題，但尚未檢閱。 |
| 已認可 | 系統管理員已檢閱智慧群組，並已開始處理。 |
| 關閉 | 已解決問題。 關閉智慧群組之後，您可以將警示變更為另一個狀態以重新開啟它。 |

[了解如何變更智慧群組的狀態。](https://aka.ms/managing-alert-smart-group-states)

> [!NOTE]
>  變更智慧群組的狀態不會變更個別成員警示的狀態。

## <a name="smart-group-details-page"></a>智慧群組詳細資料頁面

當您選取智慧群組時，會顯示 [智慧群組詳細資料] 頁面。 它提供智慧群組的相關詳細資料，包括建立該群組的原因，且可讓您變更其狀態。
 
![智慧群組詳細資料](media/monitoring-overview-unified-alerts/smart-group-detail.png)


[智慧群組詳細資料] 頁面包含下列區段。

| 區段 | 說明 |
|:---|:---|
| 警示 | 列出智慧群組中包含的個別警示。 選取警示以開啟其 [警示詳細資料] 頁面。 |
| 歷程記錄 | 列出智慧群組採取的每個動作，以及對智慧群組所做的任何變更。 這目前僅限於狀態變更和警示成員資格變更。 |

## <a name="smart-group-taxonomy"></a>智慧群組分類法

智慧群組的名稱是其第一個警示的名稱。 您無法建立或重新命名智慧群組。

## <a name="next-steps"></a>後續步驟

- [管理智慧群組](https://aka.ms/managing-smart-groups)
- [變更警示與智慧群組狀態](https://aka.ms/managing-alert-smart-group-states)

