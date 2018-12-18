---
title: Azure AD 自助式密碼重設 Windows 7 和 8.1
description: 如何在 Windows 7 或 8.1 的登入畫面啟用使用忘記密碼的自助式密碼重設
services: active-directory
ms.service: active-directory
ms.component: authentication
ms.topic: conceptual
ms.date: 09/11/2018
ms.author: joflore
author: MicrosoftGuyJFlo
manager: mtillman
ms.reviewer: sahenry
ms.openlocfilehash: ae2fab560ca9579860aeba91f9322c6521a83de1
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47167657"
---
# <a name="how-to-enable-password-reset-from-windows-7-8-and-81"></a>如何：從 Windows 7、8 和 8.1 啟用密碼重設

身為系統管理員，您已啟用自助式密碼重設 (SSPR)，但使用者卻一直連絡服務台來要求重設其密碼，原因是他們無法進入瀏覽器視窗來存取 [SSPR 入口網站](https://aka.ms/sspr)。 在 Windows 10 電腦上，您可以使用[從登入畫面重設 Azure AD 密碼](tutorial-sspr-windows.md)教學課程，在登入畫面啟用 [重設密碼] 連結，下列指導方針會協助您讓 Windows 7、8 和 8.1 使用者能夠在 Windows 登入畫面使用 SSPR 重設其密碼。

不同於 Windows 10 電腦，Windows 7、8 和 8.1 電腦不需要加入 Azure AD 網域或加入 Active Directory 網域就能重設密碼。

![顯示了「忘記密碼?」連結的 Windows 7 登入畫面範例 link shown](media/howto-sspr-windows-7-8/windows-7-logon-screen.png)

## <a name="prerequisites"></a>必要條件

* 必須啟用 Azure AD 自助式密碼重設。
* 已修補的 Windows 7 或 Windows 8.1 作業系統。
* 已使用[傳輸層安全性 (TLS) 登錄設定](https://docs.microsoft.com/windows-server/security/tls/tls-registry-settings#tls-12)中所找到的指南啟用 TLS 1.2。

> [!WARNING]
> 必須啟用 TLS 1.2，而不能只是設定為自動交涉。

## <a name="install"></a>Install

1. 下載適用於所要啟用 Windows 版本的安裝程式。

   1. 您可以在 Microsoft 下載中心 ([https://aka.ms/sspraddin](https://aka.ms/sspraddin)) 取得軟體

1. 登入要安裝的電腦，並執行安裝程式。
1. 安裝之後，強烈建議您重新開機。
1. 重新開機後，在登入畫面中選擇使用者，然後按一下 [忘記密碼?] 來起始密碼重設工作流程。
1. 完成工作流程，遵循螢幕上的步驟來重設密碼。

![按一下 [忘記密碼?] 自助式密碼重設流程的 Windows 7 範例 self-service password reset flow](media/howto-sspr-windows-7-8/windows-7-sspr.png)

### <a name="silent-installation"></a>無訊息安裝

* 若要進行無訊息安裝，請使用命令 “msiexec /i SsprWindowsLogon.PROD.msi /qn”
* 若要進行無訊息解除安裝，請使用命令 “msiexec /x SsprWindowsLogon.PROD.msi /qn”

## <a name="caveats"></a>需要注意的事項

您必須先註冊 SSPR，才能使用 [忘記密碼] 連結。

![重設密碼所需要的其他安全性資訊](media/howto-sspr-windows-7-8/windows-7-sspr-need-security-info.png)

在此初始版本中，無法使用 Microsoft Authenticator 應用程式獲取通知和程式碼來重設密碼。 使用者必須註冊可符合原則需求的替代方法。

## <a name="troubleshooting"></a>疑難排解

電腦和 Azure AD 中都會記錄事件。

Azure AD 事件會包含 IP 位址相關資訊以及發生密碼重設的 ClientType。

![Azure AD 稽核記錄中的 Windows 7 登入畫面密碼重設範例](media/howto-sspr-windows-7-8/windows-7-sspr-azure-ad-audit-log.png)

如果需要額外的記錄，則可變更電腦上的登錄機碼來啟用詳細資訊記錄。 只能為了進行疑難排解而啟用詳細資訊記錄。

```
HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Authentication\Credential Providers\{86D2F0AC-2171-46CF-9998-4E33B3D7FD4F}
```

* 若要啟用詳細資訊記錄，請建立 REG_DWORD: “EnableLogging”，並將它設定為 1。
* 若要停用詳細資訊記錄，請將 REG_DWORD “EnableLogging” 變更為 0。

## <a name="next-steps"></a>後續步驟

[讓 Windows 10 使用者在登入畫面重設其密碼](tutorial-sspr-windows.md)