---
title: Azure Active Directory B2B 共同作業的限制 | Microsoft Docs
description: Azure Active Directory B2B 共同作業目前的限制
services: active-directory
ms.service: active-directory
ms.component: B2B
ms.topic: conceptual
ms.date: 05/23/2017
ms.author: mimart
author: msmimart
manager: mtillman
ms.reviewer: sasubram
ms.openlocfilehash: 4e997e33eaccc6938209937d3157498db8bf4781
ms.sourcegitcommit: 776b450b73db66469cb63130c6cf9696f9152b6a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2018
ms.locfileid: "45982335"
---
# <a name="limitations-of-azure-ad-b2b-collaboration"></a>Azure AD B2B 共同作業的限制
Azure Active Directory (Azure AD) B2B 共同作業目前受限於本文所述的限制。

## <a name="possible-double-multi-factor-authentication"></a>可能的雙重多重要素驗證
使用 Azure AD B2B，您便可以在資源組織 (邀請組織) 強制執行多重要素驗證。 若要了解採用此方法的詳細原因，請參閱 [B2B 共同作業使用者的條件式存取](conditional-access.md)。 如果合作夥伴已設定並強制使用 Multi-Factor Authentication，其使用者可能必須在其所屬組織執行一次驗證，然後在您的組織中再次執行驗證。

## <a name="instant-on"></a>即時
在 B2B 共同作業流程中，我們會新增使用者到目錄中，並在邀請兌換與應用程式指派等期間動態更新它們。 更新與寫入通常會在一個目錄執行個體中進行，而且必須複寫到所有執行個體。 在所有執行個體都更新後，複寫作業便已完成。 有時，在某個執行個體中寫入或更新物件，但另一個執行個體呼叫擷取此物件時，就會發生複寫延遲問題。 如果發生該情況，請重新整理或重試來提供幫助。 如果您正在使用我們的 API 來撰寫應用程式，則採用某種讓步來重試是可減輕此問題的良好防禦性做法。

## <a name="azure-ad-directories"></a>Azure AD 目錄
Azure AD B2B 受限於 Azure AD 服務目錄限制。 如需使用者可建立的目錄數目以及使用者或來賓使用者可屬於的目錄數目詳細資訊，請參閱 [Azure AD 服務限制](https://docs.microsoft.com/azure/active-directory/users-groups-roles/directory-service-limits-restrictions)。

## <a name="next-steps"></a>後續步驟

請參閱下列有關 Azure AD B2B 共同作業的文章：

- [何謂 Azure AD B2B 共同作業？](what-is-b2b.md)
- [委派 B2B 共同作業邀請](delegate-invitations.md)

