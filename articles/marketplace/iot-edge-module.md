---
title: Azure IoT Edge 模組
description: Azure Marketplace 中適用於應用程式和服務發行者的 IoT Edge 模組供應項目。
services: Azure, Marketplace, Compute, Storage, Networking, Blockchain, IoT Edge module offer
documentationcenter: ''
author: qianw211
manager: pabutler
editor: ''
ms.assetid: ''
ms.service: marketplace
ms.workload: ''
ms.tgt_pltfrm: ''
ms.devlang: ''
ms.topic: article
ms.date: 09/22/2018
ms.author: qianw211
ms.openlocfilehash: ecc892a38d5e86a089085cd67a78ce7d00c86fd8
ms.sourcegitcommit: 5b8d9dc7c50a26d8f085a10c7281683ea2da9c10
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47181111"
---
# <a name="iot-edge-modules"></a>IoT Edge 模組

[Azure IoT Edge](https://azure.microsoft.com/services/iot-edge/) 平台是「Azure 雲端」所支持的平台。  此平台可讓使用者部署要直接在 IoT 裝置上執行的雲端工作負載。  IoT Edge 模組可以執行離線工作負載，並在本機進行資料分析。 此供應項目類型有助於節省頻寬、保護本機和敏感性資料，並可提供低延遲回應時間。  您現在可以選擇利用這些預先建置的工作負載。 到目前為止，只有來自 Microsoft 的少數第一方解決方案可供使用。  您必須投入時間和資源來建置自己的自訂 IoT 解決方案。

透過導入 [Azure Marketplace 中的 IoT Edge 模組](https://azuremarketplace.microsoft.com/marketplace/apps/category/internet-of-things?page=1)，我們現在提供一個可讓發行者向 IoT 對象展示及銷售其解決方案的單一目的地。 IoT 開發人員最終將能找到及購買可加快其解決方案開發速度的功能。  

## <a name="key-benefits-of-iot-edge-modules-in-azure-marketplace"></a>Azure Marketplace 中 IoT Edge 模組的主要優勢：

| **針對發行者**    | **針對客戶 (IoT 開發人員)**  |
| :------------------- | :-------------------|
| 觸達上百萬個想要建置及部署 IoT Edge 解決方案的開發人員。  | 信心十足地使用安全且經測試的元件來撰寫 IoT Edge 解決方案。 |
| 只需發佈一次，即可在支援容器的任何 IoT Edge 硬體上執行。 | 透過找出及部署符合特定需求的第一方和第三方 IoT Edge 模組，縮短上市時間。 |
| 運用彈性計費選項創造營收 <ul> <li> 免費和自備授權 (BYOL)。 </li> </ul> | 依據您選擇的計費模型進行購買。 <ul> <li> 免費和自備授權 (BYOL)。 </li> </ul> |

## <a name="what-is-an-iot-edge-module"></a>什麼是 IoT Edge 模組？

Azure IoT Edge 可讓您以模組形式，在邊緣上部署及管理商務邏輯。 Azure IoT Edge 模組是 IoT Edge 所管理的最小計算單位，並可包含 Microsoft 服務 (例如「Azure 串流分析」)、第三方服務，或您自己的解決方案特定程式碼。 若要深入了解 IoT Edge 模組，請參閱[了解 Azure IoT Edge 模組](https://docs.microsoft.com/azure/iot-edge/iot-edge-modules)。

**容器供應項目類型與 IoT Edge 模組供應項目類型之間有何差異？**

IoT Edge 模組供應項目類型是一種在 IoT Edge 裝置上執行的特定類型容器。 它隨附用以在 IoT Edge 內容中執行的預設組態設定，並可選擇使用要與 IoT Edge 執行階段整合的 IoT Edge 模組 SDK。

## <a name="publishing-your-iot-edge-module"></a>發佈您的 IoT Edge 模組

**選取正確的店面**

IoT Edge 模組只會發佈至 Azure Marketplace，AppSource 則不適用。  如需有關各店面之差異和目標對象的詳細資訊，請參閱[決定您解決方案的發佈選項](https://docs.microsoft.com/azure/marketplace/determine-your-listing-type)。
 
**帳單選項**

Marketplace 目前針對 IoT Edge 模組支援 [免費] 和 [自備授權 (BYOL)] 計費選項。
 
**發佈選項**

在所有情況下，IoT Edge 模組都應該選取 [交易] 發佈選項。  如需有關發佈選項的更多詳細資料，請參閱[選擇發佈選項](https://docs.microsoft.com/azure/marketplace/determine-your-listing-type)。  

## <a name="eligibility-criteria"></a>適用性準則

所有 Microsoft Azure Marketplace 合約和原則的條款都適用於 IoT Edge 模組供應項目。  此外，還有適用於 IoT Edge 模組的先決條件和技術需求。  

**先決條件**

若要將 IoT Edge 模組發佈至 Azure Marketplace，您必須滿足下列先決條件：

- 「Cloud Partner 入口網站」(CPP) 的存取權。 如需詳細資訊，請參閱 [Azure Marketplace 和 AppSource 發行指南](https://docs.microsoft.com/azure/marketplace/marketplace-publishers-guide)。
- 將您的 IoT Edge 模組裝載在 Azure Container Registry 中。 
- 備妥您的 IoT Edge 模組中繼資料，例如 (非完整清單)： 
    - 標題
    - 描述 (HTML 格式)
    - 標誌影像 (PNG 格式且影像大小固定，包括 40x40px、90x90px、115x115px、255x115px)
    - 使用規定及隱私權原則
    - 預設的模組設定 (路由、對應項所需屬性、createOptions、環境變數)
    - 文件
    - 支援連絡人

**技術需求**

[雲端發佈入口網站](https://cloudpartner.azure.com/)上的 [IoT Edge 模組認證程序](https://cloudpartner.azure.com/#documentation/iot-edge-module-certification-process)詳細說明了可讓 IoT Edge 模組通過認證並發佈在 Azure Marketplace 中的主要技術需求。  

## <a name="documentation-and-resources"></a>文件和資源

當您登入[雲端發佈入口網站](https://cloudpartner.azure.com/)時，會為您提供下列文章：

- [建立 IoT Edge 模組供應項目](https://cloudpartner.azure.com/#documentation/create-iot-edge-module-offer) – 使用「雲端發佈入口網站」來發佈新 IoT Edge 模組供應項目的步驟。
- [IoT Edge 模組認證程序](https://cloudpartner.azure.com/#documentation/iot-edge-module-certification-process) – 認證 IoT Edge 模組之步驟和需求的摘要。
- [IoT Edge 模組常見問題集](https://cloudpartner.azure.com/#documentation/iot-edge-module-faq) – 與 IoT Edge 模組相關的常見問題集清單。

## <a name="next-steps"></a>後續步驟

如果您還沒有這麼做，請

- 在 [Microsoft 合作夥伴網路](https://partner.microsoft.com/membership)中註冊。
- 建立 [Microsoft 帳戶](https://account.microsoft.com/account/)(對 Azure Marketplace 交易供應項目而言為必要步驟；對其他項目則為建議步驟)。
- 提交 [Marketplace 註冊表單](https://azuremarketplace.microsoft.com/sell/signup)。

如果您已註冊，且要建立新供應項目或使用現有供應項目，請

- 登入 [Cloud Partner 入口網站](https://cloudpartner.azure.com/)以建立或完成您的供應項目。
