---
title: 安裝應用程式 Proxy 代理程式連接器時遇到問題 | Microsoft Docs
description: 如何針對在安裝應用程式 Proxy 代理程式連接器時可能遇到的問題進行疑難排解
services: active-directory
documentationcenter: ''
author: barbkess
manager: mtillman
ms.assetid: ''
ms.service: active-directory
ms.component: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 05/21/2018
ms.author: barbkess
ms.reviewer: asteen
ms.openlocfilehash: 98dc3981e0f34ebfe7d8881d77b84cfe252e3f4f
ms.sourcegitcommit: af9cb4c4d9aaa1fbe4901af4fc3e49ef2c4e8d5e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44354981"
---
# <a name="problem-installing-the-application-proxy-agent-connector"></a>安裝應用程式 Proxy 代理程式連接器時遇到問題

Microsoft AAD 應用程式 Proxy 連接器是內部網域元件，它會使用輸出連線來建立雲端可用端點至內部網域的連線。

## <a name="general-problem-areas-with-connector-installation"></a>連接器安裝的一般問題區域

連接器安裝失敗時，根本原因通常是下列其中一個區域：

1.  **連線**：若要成功完成安裝，新的連接器必須註冊並建立未來信任內容。 這是透過連線至 AAD 應用程式 Proxy 雲端服務來完成。

2.  **信任建立**：新的連接器會建立自我簽署憑證，並向雲端服務註冊。

3.  **系統管理員的驗證**：在安裝期間，使用者必須提供系統管理員認證才能完成連接器安裝。

## <a name="verify-connectivity-to-the-cloud-application-proxy-service-and-microsoft-login-page"></a>確認與雲端應用程式 Proxy 服務和 Microsoft 登入頁面之間連線

**目標：** 確認連接器電腦可以連線到 AAD 應用程式 Proxy 註冊端點以及 Microsoft 登入頁面。

1.  開啟瀏覽器並前往下列網頁：<https://aadap-portcheck.connectorporttest.msappproxy.net>，確認可透過連接埠 80 和 443 與美國中部與美國東部資料中心進行連線。

2.  如果有任何連接埠未成功 (沒有綠色勾選記號)，請確認防火牆或後端 Proxy 是否已正確定義 \*.msappproxy.net 與連接埠 80 和 443。

3.  開啟瀏覽器 (個別索引標籤) 並前往下列網頁：<https://login.microsoftonline.com>，確定您可以登入該頁面。

## <a name="verify-machine-and-backend-components-support-for-application-proxy-trust-cert"></a>確認電腦和後端元件支援應用程式 Prxoy 信任憑證

**目標︰** 確認連接器電腦、後端 Proxy 和防火牆可支援由連接器針對未來信任所建立的憑證。

>[!NOTE]
>連接器會嘗試建立由 TLS1.2 所支援的 SHA512 憑證。 如果電腦或後端防火牆及 Proxy 不支援 TLS1.2，則安裝會失敗。
>
>

**解決此問題：**

1.  確認電腦支援 TLS1.2：2012 R2 之後的所有 Windows 版本都應該支援 TLS 1.2。 如果您的連接器電腦是 2012 R2 或更舊版本，請確定電腦已安裝下列 KB：<https://support.microsoft.com/help/2973337/sha512-is-disabled-in-windows-when-you-use-tls-1.2>

2.  連絡您的網路系統管理員，並要求確認後端 Proxy 和防火牆不會針對連出流量封鎖 SHA512。

## <a name="verify-admin-is-used-to-install-the-connector"></a>確認是以系統管理員身分安裝連接器

**目標︰** 確認嘗試安裝連接器的使用者是具有正確認證的系統管理員。 目前使用者必須是全域管理員，安裝才會成功。

**確認認證是否正確：**

連線至 <https://login.microsoftonline.com> 並使用相同的認證。 確定登入成功。 您可以檢查使用者角色，方法是移至 [Azure Active Directory] -&gt; [使用者和群組] -&gt; [所有使用者]。 

選取您的使用者帳戶，然後在產生的功能表中選取 [目錄角色]。 確認所選取的角色為 [全域管理員]。 如果您無法存取這些步驟上的任何頁面，您就不是全域管理員。

## <a name="next-steps"></a>後續步驟
[了解 Azure AD 應用程式 Proxy 連接器](application-proxy-connectors.md)
