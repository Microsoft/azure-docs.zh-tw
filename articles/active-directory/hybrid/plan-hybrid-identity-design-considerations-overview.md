---
title: Azure Active Directory 混合式身分識別設計考量 - 概觀 | Microsoft Docs
description: 混合式身分識別設計考量指南的概觀和內容對應
documentationcenter: ''
services: active-directory
author: billmath
manager: mtillman
editor: ''
ms.assetid: 100509c4-0b83-4207-90c8-549ba8372cf7
ms.service: active-directory
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 05/30/2018
ms.component: hybrid
ms.author: billmath
ms.openlocfilehash: 6d916e884e7764a1ba5ebaf224750f38e7d7673e
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46308657"
---
# <a name="azure-active-directory-hybrid-identity-design-considerations"></a>Azure Active Directory 混合式身分識別設計考量
以取用者為基礎的裝置正廣為公司組織採用，而且雲端架構的軟體即服務 (SaaS) 應用程式是很易於採用的。 因此，要持續掌控使用者在內部資料中心與雲端平台間的應用程式存取，是很不容易的。  

Microsoft 的身分識別解決方案可跨越內部部署和雲端架構功能，建立單一使用者身分識別以用於所有資源的驗證和授權，不論位於何處。 這個概念稱為混合式身分識別。 使用 Microsoft 解決方案時，混合式身分識別有不同的設計和組態可供選擇，且在某些情況下，可能會難以判斷哪些組合最符合組織的需求。 

此混合式身分識別設計考量指南將協助您了解如何設計與組織的商務和技術需求最相符的混合式身分識別解決方案。  本指南詳細說明一系列您可以依循的步驟和工作，以協助您設計符合組織獨特需求的混合式身分識別解決方案。 在這些步驟和工作中，本指南將提供相關的技術和功能選項，讓組織能夠因應其功能和服務品質 (例如可用性、延展性、效能、管理性和安全性) 層級需求。 

具體來說，混合式身分識別設計考量指南的目標，是為您解答下列問題： 

* 我需要提出及回答哪些問題，以針對某個技術或問題範疇完成最符合自身需求的混合式身分識別特定設計？
* 針對此技術或問題範疇設計混合式身分識別解決方案時，需要完成哪些活動？ 
* 哪些混合式身分識別技術和組態選項可協助我達到自身的需求？ 應如何取捨這些選項，以選擇出適用於我的企業的最佳選項？

## <a name="who-is-this-guide-intended-for"></a>本指南的適用對象為何？
 負責為中型或大型組織設計混合式身分識別解決方案的資訊長、資訊技術長、首席身分識別架構設計人員、企業架構設計人員和 IT 架構設計人員。

## <a name="how-can-this-guide-help-you"></a>本指南對您有何幫助？
您可以使用本指南來了解如何設計能夠將雲端架構身分識別管理系統與您目前的內部部署身分識別解決方案整合的混合式身分識別解決方案。 

下圖顯示的範例說明可讓 IT 系統管理員順利將其目前位於內部部署的 Windows Server Active Directory 解決方案與 Microsoft Azure Active Directory 整合，讓使用者能在位於雲端和內部部署的應用程式使用單一登入 (SSO) 的混合式身分識別解決方案。

![範例](media/plan-hybrid-identity-design-considerations/hybridID-example.png)

上方的圖例說明使用雲端服務與內部部署功能進行整合，為終端使用者提供以單一程序完成驗證的體驗，以及協助 IT 人員管理這些資源的混合式身分識別解決方案。 雖然此範例可能是常見的案例，但每個組織的混合式身分識別設計都可能會因不同的需求而與圖 1 所示的範例不同。 

本指南將提供一系列您可以依循的步驟和工作，用以設計符合組織獨特需求的混合式身分識別解決方案。 在下列步驟和工作中，本指南將提供相關的技術和功能選項，讓您能夠因應組織的功能和服務品質層級需求。

**假設**：您有 Windows Server、Active Directory 網域服務與 Azure Active Directory 的相關體驗。 在本文件中，假設您想要了解這些解決方案本身或在整合式解決方案中如何滿足您的商務需求。

## <a name="design-considerations-overview"></a>設計考量概觀
本文件將提供一組您可以依循的步驟和工作，用以設計最能符合您需求的混合式身分識別解決方案。 步驟會依序呈現。 不過，您在後續步驟中學習的設計考量，可能會因為設計選擇的衝突，而需要變更您在先前步驟中所做的決策。 在本文件中會持續嘗試警告您潛在的設計衝突。 

必須在充分地反覆執行相關步驟以整合文件中的所有考量之後，才能達到最符合您的需求的設計。 

| 混合式身分識別階段 | 主題清單 |
| --- | --- |
| 判斷身分識別需求 |[判斷商務需求](plan-hybrid-identity-design-considerations-business-needs.md)<br> [判斷目錄同步處理需求](plan-hybrid-identity-design-considerations-directory-sync-requirements.md)<br> [判斷多重要素驗證需求](plan-hybrid-identity-design-considerations-multifactor-auth-requirements.md)<br> [定義混合式身分識別採用策略](plan-hybrid-identity-design-considerations-identity-adoption-strategy.md) |
| 透過增強式身分識別解決方案規劃更高的資料安全性 |[判斷資料保護需求](plan-hybrid-identity-design-considerations-dataprotection-requirements.md) <br> [判斷內容管理需求](plan-hybrid-identity-design-considerations-contentmgt-requirements.md)<br> [判斷存取控制需求](plan-hybrid-identity-design-considerations-accesscontrol-requirements.md)<br> [判斷事件因應需求](plan-hybrid-identity-design-considerations-incident-response-requirements.md) <br> [定義資料保護策略](plan-hybrid-identity-design-considerations-data-protection-strategy.md) |
| 規劃混合式身分識別生命週期 |[判斷混合式身分識別管理工作](plan-hybrid-identity-design-considerations-hybrid-id-management-tasks.md) <br> [同步處理管理](plan-hybrid-identity-design-considerations-hybrid-id-management-tasks.md)<br> [判斷混合式身分識別管理採用策略](plan-hybrid-identity-design-considerations-lifecycle-adoption-strategy.md) |

## <a name="download-this-guide"></a>下載此指南
您可以從 [Technet 組件庫](https://gallery.technet.microsoft.com/Azure-Hybrid-Identity-b06c8288)下載 PDF 版本的《混合式身分識別設計考量》指南。 

