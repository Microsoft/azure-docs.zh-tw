---
title: 混合式身分識別設計 - 管理工作 - Azure | Microsoft Docs
description: 透過條件式存取控制，Azure Active Directory 會在驗證使用者時以及允許存取應用程式之前，檢查您挑選的特定條件。 一旦符合這些條件，就會驗證使用者並允許存取應用程式。
documentationcenter: ''
services: active-directory
author: billmath
manager: mtillman
editor: ''
ms.assetid: 65f80aea-0426-4072-83e1-faf5b76df034
ms.service: active-directory
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 05/30/2018
ms.component: hybrid
ms.author: billmath
ms.custom: seohack1
ms.openlocfilehash: 16b09b7d4bddc24d9efa2e37c9e31f61a4333f86
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46305557"
---
# <a name="plan-for-hybrid-identity-lifecycle"></a>規劃混合式身分識別生命週期
身分識別是企業行動力和應用程式存取策略的基礎之一。 無論您登入行動裝置還是 SaaS 應用程式，您的身分識別都是您能否進行存取的關鍵。 在其最高層級上，身分識別管理解決方案牽涉到存放庫的統合和同步，而其中又包含佈建資源程序的自動化和集中化。 身分識別解決方案應為跨內部部署與雲端的集中式身分識別功能，且應使用某種形式的身分識別同盟，以維護集中式驗證，並且安全地與外部使用者和企業進行共用和共同作業。 資源的範圍涵蓋作業系統和應用程式，乃至於組織中或隸屬於組織的人員。 組織結構可以改變，以因應佈建原則和程序。

為您的使用者提供身分識別解決方案，使其體驗自助式服務以提升能力進而保持生產力，是很重要的。 您的身分識別解決方案如果針對使用者在其需要存取的所有資源中啟用單一登入，則會更強固。 所有層級的系統管理員都可以使用標準化的程序來管理使用者認證。 視佈建管理解決方案的範圍之不同，某些層級的管理可以減少或排除。 此外，您可以安全地在不同組織之間散發管理功能，手動或自動皆可。 例如，網域管理員只能為該網域中的人員和資源提供服務。 這名使用者可以執行管理和佈建工作，但無權執行組態工作，例如建立工作流程。

## <a name="determine-hybrid-identity-management-tasks"></a>判斷混合式身分識別管理工作
在您的組織中散發管理工作，可以改善管理的精確度和效率，並改善組織的工作負載平衡。 以下是定義健全的身分識別管理系統的要點。

 ![](./media/plan-hybrid-identity-design-considerations/Identity_management_considerations.png)

若要定義混合式身分識別管理工作，您必須了解將採用混合式身分識別的組織有哪些基本特性。 請務必了解目前要用於身分識別來源的儲存機制。 了解這些核心元素之後，將可得知您的基本需求，據此，您必須提出更細微的問題，進而為您的身分識別解決方案引導出更理想的設計決策。  

定義這些需求時，至少要回答下列問題

* 佈建選項： 
  
  * 混合式身分識別解決方案是否支援健全的帳戶存取管理和佈建系統？
  * 使用者、群組和密碼將以何種方式受到管理？
  * 身分識別生命週期管理是否有因應能力？ 
    * 密碼更新的帳戶暫止需要多久的時間？
* 授權管理： 
  
  * 混合式身分識別解決方案是否會處理授權管理？
    * 如果是，可用的功能為何？
* 解決方案是否會處理以群組為基礎的授權管理？ 
  
      - 如果是，是否可以將安全性群組指派給它？ 
       - 如果是，雲端目錄是否會自動將授權指派給群組的所有成員？ 
        - 如果後續在群組中新增或移除使用者，將會有何情況？會適當地自動指派或移除授權嗎？ 
* 與其他協力廠商身分識別提供者整合：
* 此混合式解決方案是否可與協力廠商身分識別提供者整合，以實作單一登入？
* 是否可將所有不同的身分識別提供者統合到一致的身分識別系統中？
* 如果是，有哪些系統？如何執行？可用的功能為何？

## <a name="synchronization-management"></a>同步處理管理
身分識別管理員的目標之一，是要能夠啟用所有身分識別提供者，並保持其同步處理狀態。 您可以根據授權的主要身分識別提供者，保持資料的同步處理狀態。 在混合式身分識別案例中，您可以透過同步處理的管理模型，在內部部署伺服器中管理所有的使用者和裝置身分識別，並將帳戶同步處理至雲端 (和選擇性地同步處理密碼)。 使用者在內部部署中輸入的密碼與雲端相同，且在登入時，身分識別解決方案會驗證密碼。 此模型會使用目錄同步處理工具。

![](./media/plan-hybrid-identity-design-considerations/Directory_synchronization.png) 若要適當設計混合式身分識別解決方案的同步處理，請確實回答下列問題：•    有哪些可供混合式身分識別解決方案使用的同步處理解決方案？
•    有哪些可用的單一登入功能？
•    B2B 與 B2C 之間有哪些身分識別同盟選項？

## <a name="next-steps"></a>後續步驟
[判斷混合式身分識別管理採用策略](plan-hybrid-identity-design-considerations-lifecycle-adoption-strategy.md)

## <a name="see-also"></a>另請參閱
[設計考量概觀](plan-hybrid-identity-design-considerations-overview.md)

