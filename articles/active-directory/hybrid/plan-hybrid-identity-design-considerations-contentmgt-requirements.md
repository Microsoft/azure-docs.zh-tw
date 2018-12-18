---
title: 混合式身分識別設計 - 內容管理需求 Azure | Microsoft Docs
description: 深入了解如何判斷適合您企業的內容管理需求。 通常當使用者擁有自己的裝置時，可能也會有多個認證，將根據其使用的應用程式來替換。 請務必區分使用個人認證建立的內容以及使用公司認證建立的內容。 您的身分識別解決方案應該能夠與雲端服務互動，為終端使用者提供順暢的體驗，同時確保其隱私權並提高保護來防止資料外洩。
documentationcenter: ''
services: active-directory
author: billmath
manager: mtillman
editor: ''
ms.assetid: dd1ef776-db4d-4ab8-9761-2adaa5a4f004
ms.service: active-directory
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 05/30/2018
ms.component: hybrid
ms.author: billmath
ms.openlocfilehash: a3e5ae61fcefcb65d08288a2ddc2dd239e68b211
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46306317"
---
# <a name="determine-content-management-requirements-for-your-hybrid-identity-solution"></a>判斷混合式身分識別解決方案的內容管理需求
了解適合您企業的內容管理需求，可能會直接影響您決定要使用哪一個混合式身分識別解決方案。 由於裝置數目激增，而且可讓使用者攜帶自己的裝置 ([BYOD](https://aka.ms/byodcg))，所以公司必須保護它自己的資料，但它也必須完整保持使用者的隱私權。 通常當使用者擁有自己的裝置時，可能也會有多個認證，將根據其使用的應用程式來替換。 請務必區分使用個人認證建立的內容以及使用公司認證建立的內容。 您的身分識別解決方案應該能夠與雲端服務互動，為終端使用者提供順暢的體驗，同時確保其隱私權並提高保護來防止資料外洩。 

您的身分識別解決方案是透過不同技術控制項來運用，以便提供內容管理，如下圖所示：

![](./media/plan-hybrid-identity-design-considerations/securitycontrols.png)

**充分運用身分識別管理系統的安全性控制項**

一般而言，內容管理需求將在下列區域中充分運用您的身分識別管理系統：

* 隱私權：識別擁有資源的使用者，並套用適當的控制項來保有完整性。
* 資料分類：識別使用者或群組，以及根據物件分類來識別該物件的存取層級。 
* 資料外洩防護：負責保護資料以避免外洩的安全性控制項將需要與身分驗證系統互動，來驗證使用者的身分識別。 這對於稽核記錄用途也很重要。

> [!NOTE]
> 如需資料分類的最佳做法與指導方針，請參閱 [雲端整備的資料分類](http://download.microsoft.com/download/0/A/3/0A3BE969-85C5-4DD2-83B6-366AA71D1FE3/Data-Classification-for-Cloud-Readiness.pdf) 。
> 
> 

規劃您的混合式身分識別解決方案時，請確定會根據貴組織的需求來回答下列問題：

* 貴公司擁有適當的安全性控制項，可用來強制執行資料隱私權嗎？
  * 如果是，該安全性控制項將能夠與您即將採用的混合式身分識別解決方案整合嗎？
* 貴公司是否使用資料分類？
  * 如果是，目前的解決方案能夠與您即將採用的混合式身分識別解決方案整合嗎？
* 貴公司目前對於資料外洩有任何解決方案嗎？ 
  * 如果是，目前的解決方案能夠與您即將採用的混合式身分識別解決方案整合嗎？
* 貴公司需要稽核資源的存取嗎？
  * 如果是，是哪種類型的資源？
  * 如果是，需要哪一個資訊層級？
  * 如果是，稽核記錄檔必須位於何處？ 內部部署或在雲端中？
* 貴公司需要加密任何包含敏感性資料 (SSN、信用卡號碼等) 的電子郵件嗎？
* 貴公司需要加密所有與外部商務合作夥伴共用的文件/內容嗎？
* 貴公司需要強制執行有關特定電子郵件類型的公司原則 (不要全部回覆、不要轉寄) 嗎？

> [!NOTE]
> 請確定會記下每個答案，並了解答案背後的原理。 [定義資料保護策略](plan-hybrid-identity-design-considerations-data-protection-strategy.md) 將介紹可用選項，以及每個選項的優點/缺點。  回答這些問題之後，您就能選取最適合業務需求的選項。
> 
> 

## <a name="next-steps"></a>後續步驟
[判斷存取控制需求](plan-hybrid-identity-design-considerations-accesscontrol-requirements.md)

## <a name="see-also"></a>另請參閱
[設計考量概觀](plan-hybrid-identity-design-considerations-overview.md)

