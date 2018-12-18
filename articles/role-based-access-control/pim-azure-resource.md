---
title: 使用 Privileged Identity Management (PIM) 管理 Azure 資源的存取權
description: 了解如何使用 Privileged Identity Management (PIM) 和角色型存取控制 (RBAC) 管理 Azure 資源的存取權。
services: active-directory
documentationcenter: ''
author: rolyon
manager: mtillman
editor: skwan
ms.assetid: ba06b8dd-4a74-4bda-87c7-8a8583e6fd14
ms.service: role-based-access-control
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 03/30/2018
ms.author: rolyon
ms.reviewer: skwan
ms.openlocfilehash: 141cba29f5027ce092775d97c1abe9ecf11badf5
ms.sourcegitcommit: e0834ad0bad38f4fb007053a472bde918d69f6cb
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/03/2018
ms.locfileid: "37436040"
---
# <a name="manage-access-to-azure-resources-with-privileged-identity-management"></a>使用 Privileged Identity Management 來管理對 Azure 資源的存取

若要保護具特殊權限的帳戶免受惡意網路攻擊的威脅，您可以使用 Azure Active Directory Privileged Identity Management (PIM) 來降低權限的曝光時間，並透過報告和警示提升您對其使用情況的可見度。 PIM 會限制使用者只能在 Just-In-Time (JIT) 狀態下行使其權限，或指派持續時間較短的權限，並在時間到達之後自動撤銷該權限，以達成上述目的。 

您現在可以使用 PIM 與 Azure 角色型存取控制 (RBAC) 來管理、控制和監視 Azure 資源的存取權。 PIM 可以管理內建和自訂角色的成員資格，以助您： 

- 視需要啟用 Just-In-Time 存取 Azure 資源。
- 讓已指派使用者和群組的資源存取權自動過期。
- 針對快速工作和值勤排程指派暫時的 Azure 資源存取權。
- 在新使用者或群組獲指派資源存取，以及啟用符合資格的指派時取得警示

如需詳細資訊，請參閱 [Azure PIM 中的角色型存取控制概觀](../active-directory/privileged-identity-management/azure-pim-resource-rbac.md)。