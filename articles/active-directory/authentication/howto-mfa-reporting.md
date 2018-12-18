---
title: Azure MFA 的存取及使用報告 | Microsoft Docs
description: 說明如何使用 Azure Multi-Factor Authentication 功能 - 報告。
services: multi-factor-authentication
ms.service: active-directory
ms.component: authentication
ms.topic: conceptual
ms.date: 07/30/2018
ms.author: joflore
author: MicrosoftGuyJFlo
manager: mtillman
ms.reviewer: michmcla
ms.openlocfilehash: dc4cd28fe61c422f65f47c74c7cbc4686d73ab77
ms.sourcegitcommit: 4de6a8671c445fae31f760385710f17d504228f8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/08/2018
ms.locfileid: "39628829"
---
# <a name="reports-in-azure-multi-factor-authentication"></a>Azure Multi-Factor Authentication 中的報告

Azure Multi-Factor Authentication 提供數個報告，可供您和貴組織透過 Azure 入口網站使用。 下表列出可用的報告：

| 報告 | 位置 | 說明 |
|:--- |:--- |:--- |
| 已封鎖的使用者歷程記錄 | Azure AD > MFA Server > 封鎖/解除封鎖使用者 | 顯示使用者封鎖或解除封鎖要求的歷程記錄。 |
| 使用方式和詐騙警示 | Azure AD > 登入 | 提供整體使用量、使用者摘要和使用者詳細資料的相關資訊；以及在指定的日期範圍期間所提交的詐騙警示歷程記錄。 |
| 內部部署元件的使用方式 | Azure AD > MFA Server > 活動報表 | 透過 NPS 擴充功能、ADFS 和 MFA 伺服器提供 MFA 整體使用量的相關資訊。 |
| 已略過的使用者歷程記錄 | Azure AD > MFA Server > 單次許可 | 提供針對使用者許可 Multi-Factor Authentication 之要求的歷程記錄。 |
| 伺服器狀態 | Azure AD > MFA Server > 伺服器狀態 | 顯示與帳戶相關聯之 Multi-Factor Authentication Server 的狀態。 |

## <a name="view-mfa-reports"></a>檢視 MFA 報告

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 在左側，選取 [Azure Active Directory] > [MFA Server]。
3. 選取您要檢視的報告。

   <center>![雲端](./media/howto-mfa-reporting/report.png)</center>

## <a name="azure-ad-sign-ins-report"></a>Azure AD 登入報告

透過 [Azure 入口網站](https://portal.azure.com)中的**登入活動報告**，您可以取得判斷環境執行狀況所需的資訊。

登入報告可為您提供受控應用程式使用方式和使用者登入活動的相關資訊，包括 Multi-Factor Authentication (MFA) 使用方式的相關資訊。 MFA 資料可讓您深入了解 MFA 如何在您的組織中運作。 它可讓您回答問題，例如：

- 是否以 MFA 挑戰登入？
- 使用者如何完成 MFA？
- 為何使用者無法完成 MFA？
- 有多少使用者經過 MFA 挑戰？
- 有多少使用者無法完成 MFA 挑戰？
- 使用者會遇到的常見 MFA 問題是什麼？

此資料可透過 [Azure 入口網站](https://portal.azure.com)和[報告 API](../reports-monitoring/concept-reporting-api.md) 取得。

![雲端](./media/howto-mfa-reporting/sign-in-report.png)

### <a name="sign-ins-report-structure"></a>登入報告結構

MFA 的登入活動報告可讓您存取下列資訊：

**需要 MFA：** 登入是否需要 MFA。 依據每個使用者的 MFA、條件式存取或其他原因，可能需要 MFA。 可能的值為 [是] 或 [否]。

**MFA 結果：** MFA 是否已滿足或拒絕的詳細資訊：

- 如果已滿足 MFA，此資料行提供如何滿足 MFA 的詳細資訊。
   - Azure Multi-Factor Authentication
      - 在雲端中完成
      - 由於租用戶上設定的原則所以已過期
      - 註冊提示
      - 因為在權杖中宣告而滿足
      - 因為外部提供者提供的宣告而滿足
      - 因為強式驗證而滿足
      - 因為運用的流程是 Windows 訊息代理程式登入流程而略過
      - 因為應用程式密碼而略過
      - 因為位置而略過
      - 因為已註冊的裝置而略過
      - 因為已記住的裝置而略過
      - 已成功完成
   - 重新導向至外部提供者以進行多重要素驗證

- 如果已拒絕 MFA，此資料行會提供拒絕的原因。
   - Azure Multi-Factor Authentication 遭到拒絕；
      - 驗證進行中
      - 重複的驗證嘗試
      - 輸入太多次不正確的代碼
      - 無效的驗證
      - 無效的行動應用程式驗證碼
      - 設定錯誤
      - 撥打電話轉到語音信箱
      - 電話號碼的格式無效
      - 服務錯誤
      - 無法接通使用者的電話
      - 無法將行動應用程式通知傳送到裝置
      - 無法傳送行動應用程式通知
      - 使用者拒絕驗證
      - 使用者未回應行動應用程式通知
      - 使用者沒有任何已註冊的驗證方法
      - 使用者輸入不正確的代碼
      - 使用者輸入不正確的 PIN
      - 使用者未成功驗證即掛斷電話
      - 使用者遭到封鎖
      - 使用者從未輸入驗證碼
      - 找不到使用者
      - 驗證碼已使用過一次

**MFA 驗證方法：** 使用者用來完成 MFA 的驗證方法。 可能的值包括：

- 簡訊
- 行動應用程式通知
- 撥打電話 (驗證電話)
- 行動應用程式驗證碼
- 撥打電話 (辦公室電話)
- 撥打電話 (替代驗證電話)

**MFA 驗證詳細資料：** 電話號碼的清除版本，例如：+X XXXXXXXX64。

**條件式存取**：尋找會影響到登入嘗試的條件式存取原則相關資訊，包括：

- 原則名稱
- 授與控制
- 工作階段控制項
- 結果

## <a name="powershell-reporting"></a>Powershell 報告

識別已使用下列 Powershell 註冊 MFA 的使用者。

```Get-MsolUser -All | where {$_.StrongAuthenticationMethods -ne $null} | Select-Object -Property UserPrincipalName```

識別尚未使用下列 Powershell 註冊 MFA 的使用者。

```Get-MsolUser -All | where {$_.StrongAuthenticationMethods.Count -eq 0} | Select-Object -Property UserPrincipalName```

## <a name="next-steps"></a>後續步驟

* [適用於使用者](../user-help/multi-factor-authentication-end-user.md)
* [部署的位置](concept-mfa-whichversion.md)
