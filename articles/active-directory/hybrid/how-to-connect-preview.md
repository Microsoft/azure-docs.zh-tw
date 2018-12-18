---
title: Azure AD Connect：預覽版功能 | Microsoft Docs
description: 本主題詳細描述 Azure AD Connect 中位於預覽的功能。
services: active-directory
documentationcenter: ''
author: billmath
manager: mtillman
editor: ''
ms.assetid: c75cd8cf-3eff-4619-bbca-66276757cc07
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/13/2017
ms.component: hybrid
ms.author: billmath
ms.openlocfilehash: ab64cc2fc206772fe0f842af9d2f4c3596d76c07
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46309087"
---
# <a name="more-details-about-features-in-preview"></a>有關預覽中之功能的其他詳細資料
本主題描述如何使用預覽中目前的功能。

## <a name="group-writeback"></a>群組回寫
選用功能中的群組回寫選項可讓您將「Office 365 群組」回寫至已安裝 Exchange 的樹系。 這是一律在雲端中控制的群組。 如果您有 Exchange 內部部署，則可以將這些群組回寫到內部部署，讓具有內部部署 Exchange 信箱的使用者可以從這些群組傳送和接收電子郵件。

如需有關 Office 365 群組及其使用方式的詳細資訊，可在 [這裡](https://aka.ms/O365g)找到。

Office 365 群組將會在內部部署 AD DS 中顯示為通訊群組。 您的內部部署 Exchange 伺服器必須是 Exchange 2013 累積更新 8 (2015 年 3 月發行) 或 Exchange 2016，才能辨識這個新的群組類型。

**預覽期間的注意事項**

* 目前在預覽中不會填入通訊錄屬性。 若沒有此屬性，群組就不會顯示在 GAL 中。 若要填入此屬性，最簡單的方法是使用 Exchange PowerShell Cmdlet `update-recipient`。
* 只有使用 Exchange 結構描述的樹系才是群組的有效目標。 如果沒有偵測到 Exchange，則會無法啟用群組回寫功能。
* 目前只支援單一樹系 Exchange 組織部署。 如果您的內部部署環境中有多個 Exchange 組織，則需要擁有內部部署 GALSync 解決方案才能讓這些群組出現在其他樹系中。
* 群組回寫功能無法處理安全性群組或通訊群組。

> [!NOTE]
> 需要 Azure AD Premium 的訂用帳戶才能使用群組回寫功能。
> 
>

## <a name="user-writeback"></a>使用者回寫
> [!IMPORTANT]
> 在 Azure AD Connect 的 2015 年 8 月更新中，已移除使用者的回寫預覽功能。 如果已啟用它，則您應該停用這個功能。
>
>

## <a name="next-steps"></a>後續步驟
繼續[自訂 Azure AD Connect 安裝](how-to-connect-install-custom.md)。

深入了解 [整合內部部署身分識別與 Azure Active Directory](whatis-hybrid-identity.md)。
