---
title: 何時及如何使用 Azure Multi-Factor Auth Provider？
description: 何時應使用驗證提供者搭配 Azure MFA？
services: multi-factor-authentication
ms.service: active-directory
ms.component: authentication
ms.topic: conceptual
ms.date: 09/06/2018
ms.author: joflore
author: MicrosoftGuyJFlo
manager: mtillman
ms.reviewer: michmcla
ms.openlocfilehash: 8e77a33667bd6794f667348958e0edb9c6a8fb0d
ms.sourcegitcommit: af60bd400e18fd4cf4965f90094e2411a22e1e77
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44094972"
---
# <a name="when-to-use-an-azure-multi-factor-authentication-provider"></a>何時使用 Azure Multi-Factor Authentication Provider

依預設，擁有 Azure Active Directory 和 Office 365 使用者的全域管理員可以使用雙步驟驗證。 不過，如果您想要充分利用[進階功能](howto-mfa-mfasettings.md)，則應該購買完整版的 Azure Multi-Factor Authentication (MFA)。

Azure Multi-Factor Auth Provider 可讓**沒有授權**的使用者能夠充分利用 Azure Multi-Factor Authentication 提供的功能。

如果您有涵蓋組織中所有使用者的授權，則不需要 Azure Multi-Factor Auth Provider。 只有在也需要為一些沒有授權的使用者提供雙步驟驗證時，才需建立 Azure Multi-Factor Authentication 提供者。

> [!NOTE]
> 自 2018 年 9 月 1 日起，不再建立新的驗證提供者。 現有的驗證提供者可繼續使用與更新。 多重要素驗證將繼續為 Azure AD Premium 授權中的可用功能。

## <a name="caveats-related-to-the-azure-mfa-sdk"></a>Azure MFA SDK 的相關注意事項

下載 SDK 需要 Azure Multi-Factor Auth Provider。 請注意 SDK 已被取代，不再對新客戶提供支援，而且只會繼續運作到 2018 年 11 月 14 日為止。 在這個時間之後，對 SDK 的呼叫將會失敗。

若要下載 SDK，即使您有 Azure MFA、AAD Premium 或其他配套授權，還是必須建立 Azure Multi-Factor Auth Provider。 如果您針對此用途建立 Azure Multi-Factor Auth Provider，且已有授權，請務必使用**每個啟用的使用者**模型建立提供者。 然後，將提供者連結至包含 Azure MFA、Azure AD Premium 或其他配套授權的目錄。 這個組態可確保您只會在執行雙步驟驗證的唯一使用者超過您所擁有的授權數目時收到帳單。

## <a name="what-is-an-mfa-provider"></a>什麼是 MFA Provider？

驗證提供者的類型有兩種，差異在於您 Azure 訂用帳戶的收費方式。 每次驗證選項會計算一個月中對您的租用戶執行之驗證數目。 如果您有許多使用者偶爾才會進行驗證，最好使用此選項。 每位使用者選項會計算您的租用戶中一個月執行雙步驟驗證之個人數目。 如果您的某些使用者已擁有授權，但是需要將 MFA 擴充至您授權限制之外的更多使用者，最好使用此選項。

## <a name="manage-your-mfa-provider"></a>管理 MFA Provider

建立 MFA 提供者之後，您就無法變更使用量模型 (依據已啟用的使用者或依據驗證)。

如果您購買的授權已足夠讓啟用 MFA 的所有使用者使用，您可以一併刪除 MFA 提供者。

如果您的 MFA 提供者未連結至 Azure AD 租用戶，或是您將新的 MFA 提供者連結至不同 Azure AD 租用戶，則使用者設定和組態選項並不會進行轉移。 此外，現有 Azure MFA 伺服器也需要使用 MFA 提供者產生的啟用認證重新啟動。 重新啟用 MFA 伺服器以將其連結至 MFA 提供者，並不會影響電話和簡訊驗證，但所有使用者的行動裝置應用程式通知會停止運作，直到他們重新啟動行動裝置應用程式。

## <a name="next-steps"></a>後續步驟

[進行 Multi-Factor Authentication 設定](howto-mfa-mfasettings.md)
