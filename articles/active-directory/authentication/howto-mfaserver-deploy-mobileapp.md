---
title: Azure MFA Server Mobile App Web 服務 | Microsoft Docs
description: 設定 MFA 伺服器以將推播通知傳送給具有 Microsoft Authenticator App 的使用者。
services: multi-factor-authentication
ms.service: active-directory
ms.component: authentication
ms.topic: conceptual
ms.date: 07/11/2018
ms.author: joflore
author: MicrosoftGuyJFlo
manager: mtillman
ms.reviewer: michmcla
ms.openlocfilehash: d31cf6569f642d2c7b61abe25148f5cfb199f0a2
ms.sourcegitcommit: 1478591671a0d5f73e75aa3fb1143e59f4b04e6a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/19/2018
ms.locfileid: "39159435"
---
# <a name="enable-mobile-app-authentication-with-azure-multi-factor-authentication-server"></a>使用 Azure Multi-Factor Authentication Server 來啟用行動應用程式驗證

Microsoft Authenticator 應用程式提供額外的頻外驗證選項。 Azure Multi-Factor Authentication 不會在使用者登入時，撥打自動電話或傳送 SMS 給使用者，而是會將通知推送到使用者智慧型手機或平板電腦上的 Microsoft Authenticator 應用程式。 使用者只需要在應用程式中點選 驗證 \(或輸入 PIN 再點選 [驗證]) 即可完成登入。

當手機收訊不可靠時，建議使用行動應用程式進行兩步驟驗證。 如果您使用此應用程式作為 OATH 權杖產生器，它並不需要任何網路或網際網路連線。

> [!IMPORTANT]
> 若您已安裝 Azure Multi-Factor Authentication Server v8.x 或更新版本，則不需要執行以下大部分的步驟都。 您可以依照[設定行動應用程式](#configure-the-mobile-app-settings-in-the-azure-multi-factor-authentication-server)下的步驟，設定行動應用程式驗證。

## <a name="requirements"></a>需求

若要使用 Microsoft Authenticator 應用程式，您必須執行 Multi-Factor Authentication Server v8.x 或更高版本

## <a name="configure-the-mobile-app-settings-in-the-azure-multi-factor-authentication-server"></a>在 Azure Multi-Factor Authentication Server 中配置行動應用程式設定

1. 在 Multi-Factor Authentication Server 主控台中，按一下 [使用者入口網站] 圖示。 如果您允許使用者控制其驗證方法，請在 [設定] 索引標籤之 [允許使用者選取方法] 下方選取 [行動應用程式]。 若未啟用這個功能，使用者就必須連絡技術支援人員來完成行動裝置應用程式啟用。
2. 選取 [允許使用者啟用行動裝置應用程式] 方塊。
3. 選取 [允許使用者註冊] 方塊。
4. 按一下 [行動裝置應用程式] 圖示。
5. 以要顯示在此帳戶的行動裝置應用程式中的公司或組織名稱填入 [帳戶名稱] 欄位。
   ![MFA Server 組態行動裝置應用程式設定](./media/howto-mfaserver-deploy-mobileapp/mobile.png)

## <a name="next-steps"></a>後續步驟

- [使用 Azure Multi-Factor Authentication 與協力廠商 VPN 的進階案例](howto-mfaserver-nps-vpn.md)。
