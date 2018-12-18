---
title: 物聯網安全性最佳作法 | Microsoft Docs
description: 本文提供 Microsoft 物聯網安全性最佳作法的策劃清單及一般建議。
services: security
documentationcenter: na
author: barclayn
manager: mbaldwin
editor: TomSh
ms.assetid: 2d5598c5-4c30-481d-b8f4-51ee024ea9a7
ms.service: security
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 04/26/2018
ms.author: barclayn
ms.openlocfilehash: 804dd6828931ca2801360ab18be938c8bacf2bfa
ms.sourcegitcommit: e2adef58c03b0a780173df2d988907b5cb809c82
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/28/2018
ms.locfileid: "32185335"
---
# <a name="internet-of-things-security-best-practices"></a>物聯網安全性最佳作法

保護物聯網 (IoT) 基礎結構是任何涉入 IoT 解決方案的人的重要任務。 由於涉及的裝置數目和這些裝置的分散式質性，危及數百萬個 IoT 裝置的相關安全性事件影響廣泛，不可等閒視之。

因此，IoT 安全性需要一套深度安全性作法。 資料在雲端中及流動於私人和公用網路上時都需要安全保護。 需要備妥方法來安全地佈建 IoT 裝置本身。 從裝置到網路，以至於雲端後端，每一層都需要確保絕對的安全性。

IoT 最佳作法可分類如下︰

* IoT 硬體製造商或整合者
* IoT 解決方案開發人員
* IoT 解決方案部署人員
* IoT 解決方案操作人員

本文摘要說明 [物聯網安全性最佳作法](../iot-suite/iot-security-best-practices.md)。 如需詳細資訊，請參閱該文件。

## <a name="iot-hardware-manufacturer-or-integrator"></a>IoT 硬體製造商或整合者

如果您是 IoT 硬體製造商或硬體整合者，請依循下面的最佳作法：

* **設定符合最小需求的硬體範圍**：硬體設計應包括硬體運作時所需的最少功能，僅此而已。 
* **讓硬體具備防竄改功能**：內建偵測硬體實體竄改 (例如開啟裝置外蓋、移除裝置零件等等) 的機制。 
* **建立周圍安全的硬體**：如果 [COGS](https://en.wikipedia.org/wiki/Cost_of_goods_sold) 允許，請建立安全性功能，例如以安全與加密儲存體及信任的平台模組 (TPM) 為基礎的開機功能。
* **保護在裝置升級安全**：在裝置存留期間升級韌體是不可避免的。

## <a name="iot-solution-developer"></a>IoT 解決方案開發人員

如果您是 IoT 解決方案開發人員，請依循下面的最佳作法：

* **依循安全軟體開發方法**：開發安全軟體需要從專案一開始時就思考安全性相關事項，一直到專案的實作、測試及部署。
* **小心選擇開放原始碼軟體**：開放原始碼軟體可提供快速開發解決方案的機會。
* **小心整合**：程式庫和 API 的界限中存在許多軟體安全性弱點。 

## <a name="iot-solution-deployer"></a>IoT 解決方案部署人員

如果您是 IoT 解決方案部署人員，請依循下面的最佳作法：

* **安全地部署硬體**：IoT 部署可能需要將硬體部署在不安全的位置，例如公共空間或不受監督的區域。
* **維護驗證金鑰安全**：在部署期間，每個裝置都需要由雲端服務所產生的裝置識別碼和關聯的驗證金鑰。 在部署之後也務必保護這些金鑰的實體安全。 任何洩漏的金鑰都可能被惡意裝置用來偽裝成現有的裝置。

## <a name="iot-solution-operator"></a>IoT 解決方案操作人員

如果您是 IoT 解決方案操作人員，請依循下面的最佳作法：

* **讓系統維持在最新狀態**：確保裝置的作業系統和所有裝置驅動程式都已更新至最新版本。 
* **針對惡意活動提供保護**：如果作業系統允許，請在每部裝置的作業系統中加裝防毒或反惡意程式碼功能。 
* **經常稽核**：回應安全性事件時，針對安全性相關問題稽核 IoT 基礎結構是關鍵所在。
* **實體保護 IoT 基礎結構**：對 IoT 基礎結構最嚴重的安全性攻擊是透過實際接觸裝置的方式進行攻擊。
* **保護雲端認證**：用來設定及操作 IoT 部署的雲端驗證認證可能是存取及危及 IoT 系統的最簡單方式。 

