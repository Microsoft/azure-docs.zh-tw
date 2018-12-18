---
title: 混合式身分識別設計 - 事件回應需求 Azure | Microsoft Docs
description: 判斷混合式身分識別解決方案的監視和報告功能，讓 IT 可用來採取動作以識別和減緩潛在威脅。
documentationcenter: ''
services: active-directory
author: billmath
manager: mtillman
editor: ''
ms.assetid: a3d2a459-599b-4b67-8e51-7369ee25082d
ms.service: active-directory
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 07/18/2017
ms.component: hybrid
ms.author: billmath
ms.custom: seohack1
ms.openlocfilehash: baa1d1fec6438b937ba783adcc86c4f74fa1a6db
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46306037"
---
# <a name="determine-incident-response-requirements-for-your-hybrid-identity-solution"></a>判斷混合式身分識別解決方案的事件回應需求
中大型組織最可能具備適當的 [安全性事件回應](https://technet.microsoft.com/library/cc700825.aspx) ，可協助 IT 根據事件層級來採取動作。 身分識別管理系統是事件回應程序中的重要元件，因為它可以用來協助識別對目標執行特定動作的人員。 混合式身分識別解決方案必須能夠提供監視和報告功能，讓 IT 可用來採取動作以識別和減緩潛在威脅。 在一般事件回應計畫中，您可以將計畫分為下列階段：

1. 初步評估。
2. 事件傳達的訊息。
3. 損毀控制與風險降低。
4. 識別是否遭到入侵及嚴重性。
5. 保留證據。
6. 通知適當的合作對象。
7. 系統修復。
8. 文件。
9. 損毀和成本評估。
10. 修訂程序和計畫。

在識別遭到入侵及嚴重性階段期間，必須識別已遭入侵的系統、已遭存取的檔案，並判斷這些檔案的敏感度。 您的混合式身分識別系統應該能夠滿足這些需求，來協助您識別進行這些變更的使用者。 

## <a name="monitoring-and-reporting"></a>監視和報告
主要是如果系統已內建稽核和報告功能，身分識別系統也可以在初步評估階段多次提供協助。 在初步評估期間，IT 系統管理員必須能夠識別可疑的活動，或者系統應該能夠根據預先設定的工作自動觸發它。 有許多活動可代表潛在的攻擊，但在其他情況下，設定不良的系統可能會在入侵偵測系統中導致誤報數目。 

身分識別管理系統應該能協助 IT 系統管理員來識別及報告這些可疑的活動。 通常可藉由監視所有系統並具備可強調顯示潛在威脅的報告功能，來滿足這些技術需求。 在考量事件回應需求時，請使用下列問題來協助您設計混合式身分識別解決方案：

* 貴公司擁有適當的安全性事件回應嗎？
  * 如果是，目前的身分識別管理系統可用來做為程序的一部分嗎？
* 貴公司需要識別使用者在不同裝置上嘗試進行的單一登入嗎？
* 貴公司需要偵測可能入侵的使用者認證嗎？
* 貴公司需要稽核使用者的存取和動作嗎？
* 貴公司需要知道使用者何時重設密碼嗎？

## <a name="policy-enforcement"></a>強制執行原則
在損毀控制和風險降低階段期間，最重要的是快速降低攻擊的實際和潛在影響。 此時您所採取的動作可以在次要和主要之間產生差異。 確切的回應將取決於您的組織和您面對攻擊的性質。 如果最初評估歸納出帳戶已遭到入侵，則您必須強制執行原則來封鎖這個帳戶。 這只是運用身分識別管理系統的其中一個範例。 在考量如何強制執行原則以回應即將發生的事件時，請使用下列問題來協助您設計混合式身分識別解決方案：

* 貴公司具備適當的原則，可視需要封鎖使用者存取網路嗎？
  * 如果是，目前的解決方案能夠與您即將採用的混合式身分識別管理系統整合嗎？
* 貴公司是否需要針對隔離的使用者強制執行條件式存取？ 

> [!NOTE]
> 請確定會記下每個答案，並了解答案背後的原理。 [定義資料保護策略](plan-hybrid-identity-design-considerations-data-protection-strategy.md) 將介紹可用選項，以及每個選項的優點/缺點。  回答這些問題之後，您就能選取最適合業務需求的選項。
> 
> 

## <a name="next-steps"></a>後續步驟
[定義資料保護策略](plan-hybrid-identity-design-considerations-data-protection-strategy.md)

## <a name="see-also"></a>另請參閱
[設計考量概觀](plan-hybrid-identity-design-considerations-overview.md)

