---
title: 停用 Azure Active Directory Domain Services | Microsoft Docs
description: 使用 Azure 入口網站停用 Azure Active Directory Domain Services
services: active-directory-ds
documentationcenter: ''
author: mahesh-unnikrishnan
manager: mtillman
editor: curtand
ms.assetid: 89e407e1-e1e0-49d1-8b89-de11484eee46
ms.service: active-directory
ms.component: domain-services
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 10/27/2017
ms.author: maheshu
ms.openlocfilehash: af24c7f9b721aab4f1ab810cf42f737fd14bdf88
ms.sourcegitcommit: 9222063a6a44d4414720560a1265ee935c73f49e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/03/2018
ms.locfileid: "39505570"
---
# <a name="disable-azure-active-directory-domain-services-using-the-azure-portal"></a>使用 Azure 入口網站停用 Azure Active Directory Domain Services
本文將示範如何使用 Azure 入口網站停用您 Azure AD 目錄的 Azure Active Directory (AD) 網域服務。

> [!WARNING]
> **這是無法回復的永久性刪除。**
> 請謹慎執行！ 當您刪除受控網域時：
  * 受控網域的網域控制站會解除佈建，並從虛擬網路中移除。
  * 受控網域上的資料會永久刪除。 這包括已在受控網域上建立的自訂 OU、GPO、自訂 DNS 記錄、服務主體和 GMSA 等。
  * 聯結至受控網域的機器會失去與網域間的信任關係，且必須從網域中解除聯結。
  * 您無法使用公司 AD 認證登入這些電腦。 須改用本機系統管理員認證登入機器。
刪除受控網域不會刪除您的 Azure AD 目錄，或對目錄造成負面影響。
>

執行下列步驟，以刪除您的 Azure AD Domain Services 的受控網域：
1. 在 Azure 入口網站中瀏覽至 [Azure AD Domain Services 擴充](https://portal.azure.com/#blade/HubsExtension/Resources/resourceType/Microsoft.AAD%2FdomainServices)。
2. 按一下受控網域名稱。

    ![選取要刪除的網域](./media/getting-started/domain-services-delete-select-domain.png)

3. 在 [概觀] 頁面上，按一下 [刪除] 按鈕。

    ![刪除網域](./media/getting-started/domain-services-delete-domain.png)

4. 若要確認刪除，請輸入受控網域的 DNS 網域名稱。 完成時，請按一下 [刪除] 按鈕。

    ![刪除網域確認](./media/getting-started/domain-services-delete-domain-confirm.png)

約 15 到 20 分鐘後，受控網域就會刪除。

請考慮 [分享意見反應](active-directory-ds-contact-us.md) ，幫助我們了解哪些功能可協助您在未來選擇 Azure AD 網域服務。 此意見反應協助我們提升服務，使其更符合您的部署需求和使用案例。
