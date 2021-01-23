---
title: 了解系統管理角色委派 - Azure Active Directory | Microsoft Docs
description: Azure Active Directory 中的委派模型、範例與角色安全性
services: active-directory
documentationcenter: ''
author: rolyon
manager: mtillman
editor: ''
ms.service: active-directory
ms.workload: identity
ms.subservice: roles
ms.topic: conceptual
ms.date: 11/05/2020
ms.author: rolyon
ms.reviewer: vincesm
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 77ba93b9017fa75f7247bda639880415eda1d280
ms.sourcegitcommit: 6272bc01d8bdb833d43c56375bab1841a9c380a5
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/23/2021
ms.locfileid: "98740307"
---
# <a name="delegate-administration-in-azure-active-directory"></a>在 Azure Active Directory 中委派系統管理

組織的成長日趨複雜。 有一個常見的回應是降低使用 Azure Active Directory (AD) 系統管理員角色進行存取管理的部分工作負載。 您可以為使用者指派最低的可能權限，使其能夠存取應用程式並執行工作。 即使您不將全域系統管理員角色指派給每個應用程式擁有者，還是要取代現有全域系統管理員上的應用程式管理責任。 讓組織邁向更分散式系統管理的原因有很多。 本文可協助您規劃組織中的委派。

<!--What about reporting? Who has which role and how do I audit?-->

## <a name="centralized-versus-delegated-permissions"></a>集中式與委派的權限

當組織成長時，可能難以持續追蹤哪些使用者具備特定的系統管理員角色。 如果員工具備他們不應擁有的系統管理員權限，組織就很容易受到安全性缺口的影響。 一般而言，您支援的系統管理員數目和其權限的細微程度，取決於您部署的大小和複雜度。

* 在小型或概念證明部署中，一個或少數幾位系統管理員必須完成所有工作；沒有任何委派。 在此情況下，請建立一或多個系統管理員，並使其具備全域系統管理員角色。
* 在具有更多電腦、應用程式與電腦的較大型部署中，需要更多委派。 多個系統管理員可能會具備更特定的功能性職責 (角色)。 例如，有些可能是具特殊權限的身分識別系統管理員，而其他的可能是應用程式系統管理員。 此外，系統管理員或許只能管理特定的物件群組，例如裝置。
* 甚至連更大的部署可能都需要更細微的權限，可能還要加上具備非傳統或混合式角色的系統管理員。

在 Azure AD 入口網站中，您可以[檢視任何角色的所有成員](manage-roles-portal.md)，這可協助您快速檢查部署和委派權限。

如果您想要委派 Azure 資源的存取權，而不是 Azure AD 中的系統管理存取權，請參閱 [指派 azure 角色](../../role-based-access-control/role-assignments-portal.md)。

## <a name="delegation-planning"></a>委派規劃

這適用於開發符合您需求的委派模型。 開發委派模型是一個反覆性設計程序，而我們建議您依照下列步驟執行：

* 定義您所需的角色
* 委派應用程式系統管理
* 授與註冊應用程式的能力
* 委派應用程式擁有權
* 開發安全性方案
* 建立緊急帳戶
* 保護您的系統管理員角色
* 暫時提高特殊權限

## <a name="define-roles"></a>定義角色

決定系統管理員要執行的 Active Directory 工作，以及這些工作如何對應至角色。 您可以在 Azure 入口網站中[檢視詳細的角色描述](manage-roles-portal.md)。

您應該評估每個工作的頻率、重要性與困難度。 這些準則是工作定義的重要層面，因為它們會左右是否應委派權限：

* 您定期執行、風險有限且容易完成的工作，就是適合進行委派的絕佳候選項目。
* 委派之前，應該先非常謹慎地考量您很少執行，但在整個組織中有很大影響且需要高技能層級的工作。 您可以改為[暫時將帳戶權限提高至必要的角色](../privileged-identity-management/pim-configure.md)或重新指派工作。

## <a name="delegate-app-administration"></a>委派應用程式系統管理

若組織內的應用程式數目激增，就會使得您的委派模型超過負荷。 如果它會將應用程式存取管理的負擔加諸於全域管理員身上，那麼該模型很可能會隨著時間過去而增加其額外負荷。 如果您已為人員授與全域系統管理員角色來執行像是設定企業應用程式的動作，您現在可將它們卸載至下列權限較低的角色。 這樣做有助於改善您的安全性狀態，並降低不幸發生錯誤的可能性。 具備最特殊權限的應用程式系統管理員角色如下：

* **應用程式系統管理員** 角色，可授與管理目錄中所有應用程式的能力，包括註冊、單一登入設定、使用者和群組指派與授權、應用程式 Proxy 設定，以及同意。 它不會授與管理條件式存取的能力。
* **雲端應用程式系統管理員** 角色，可授與應用程式系統管理員的所有能力，但它不會授與應用程式 Proxy 設定的存取權 (因為它沒有內部部署權限)。

## <a name="delegate-app-registration"></a>委派應用程式註冊

根據預設，所有使用者都可建立應用程式註冊。 若要選擇性地授與建立應用程式註冊的能力：

* 在 [使用者設定] 中，將 [使用者可以註冊應用程式] 設定為 [否]
* 將使用者指派給應用程式開發人員角色

若要選擇性地授與同意允許應用程式存取資料的能力：

* 在 [使用者設定] 中，將 [使用者可同意應用程式代表自己存取公司資料] 設定為 [否]
* 將使用者指派給應用程式開發人員角色

當應用程式開發人員建立新的應用程式註冊時，系統會自動將他們新增為第一個擁有者。

## <a name="delegate-app-ownership"></a>委派應用程式擁有權

進行更細微的應用程式存取委派時，您可以將擁有權指派給個別的企業應用程式。 這會補充目前對於指派應用程式註冊擁有者的支援。 擁有權是在 [企業應用程式] 刀鋒視窗中針對每個企業應用程式所指派的。 優點則是擁有者只能管理他們自己的企業應用程式。 例如，您可以指派 Salesforce 應用程式的擁有者，而該擁有者可以管理 Salesforce 的存取和設定，但無法管理任何其他應用程式。 一個企業應用程式可以有許多擁有者，而一個使用者可以是許多企業應用程式的擁有者。 以下為兩個應用程式擁有者角色：

* **企業應用程式擁有者** 角色會授與能力來管理使用者所擁有的企業應用程式，包括單一登入設定、使用者和群組指派，以及新增其他擁有者。 它不會授與管理應用程式 Proxy 設定或條件式存取的能力。
* **應用程式註冊擁有者** 角色會授與能力來管理使用者所擁有之應用程式的應用程式註冊，包括應用程式資訊清單，以及新增其他擁有者。

## <a name="develop-a-security-plan"></a>開發安全性方案

Azure AD 提供一份廣泛的指南來規劃和執行關於您 Azure AD 系統管理員角色的安全性方案，請參閱[保護混合式部署和雲端部署的特殊權限存取](security-planning.md)。

## <a name="establish-emergency-accounts"></a>建立緊急帳戶

若要在發生問題時保有對您身分識別管理存放區的存取權，請根據[建立緊急存取系統管理帳戶](security-emergency-access.md)來準備緊急存取帳戶。

## <a name="secure-your-administrator-roles"></a>保護您的系統管理員角色

取得特殊許可權帳戶控制權的攻擊者可能會造成嚴重的損毀，因此，請先使用預設提供給所有 Azure AD 組織 (的 [基準存取原則](https://cloudblogs.microsoft.com/enterprisemobility/2018/06/22/baseline-security-policy-for-azure-ad-admin-accounts-in-public-preview/) ，來保護這些帳戶) 。 該原則會針對具特殊權限的 Azure AD 帳戶強制執行多重要素驗證。 Azure AD 基準原則涵蓋下列 Azure AD 角色：

* 全域管理員
* SharePoint 管理員
* Exchange 系統管理員
* 條件式存取系統管理員
* 安全性系統管理員

## <a name="elevate-privilege-temporarily"></a>暫時提高權限

對於大部分的日常活動，並非所有使用者都需要全域管理員權限，且並非所有使用者都應永久指派全域管理員角色。 當使用者需要全域系統管理員的使用權限時，他們應該在其自己的帳戶或其他系統管理帳戶上，於 Azure AD [Privileged Identity Management](../privileged-identity-management/pim-configure.md) 中啟動角色指派。

## <a name="next-steps"></a>後續步驟

如需 Azure AD 角色描述的參考，請參閱[在 Azure AD 中指派系統管理員角色](permissions-reference.md)