---
title: 新增適用於 Azure Stack ADFS 的使用者 | Microsoft Docs
description: 了解如何新增適用於 Azure Stack 之 ADFS 部署的使用者
services: azure-stack
documentationcenter: ''
author: jeffgilb
manager: femila
editor: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/21/2018
ms.author: jeffgilb
ms.reviewer: unknown
ms.openlocfilehash: 5774750edc5b7380275d4f20aee3be47f2f62b4d
ms.sourcegitcommit: fbba5027fa76674b64294f47baef85b669de04b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 02/24/2018
ms.locfileid: "29554561"
---
# <a name="add-users-in-the-azure-stack-development-kit"></a>在 Azure Stack 開發套件中新增使用者

適用於：Azure Stack 開發套件

若要在開發套件部署中新增額外的使用者，您必須使用 Azure Stack 主機電腦上的 Microsoft Management Console，將他們新增至 Azure Stack 開發套件目錄。
1.  在 Azure Stack 主機電腦上，開啟 [Microsoft Management Console]。
2.  按一下 [檔案] > [新增或移除嵌入式管理單元]。
3.  選取 [Active Directory 使用者和電腦] > [AzureStack.local] > [使用者]。
4.  按一下 [動作] > [新增] > [使用者]。
5.  在 [新增物件 - 使用者] 視窗中，提供並確認密碼
6.  按一下 [下一步] 以完成值，然後按一下 [完成] 來建立使用者。


