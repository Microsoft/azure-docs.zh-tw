---
title: Azure AD Connect - 傳遞驗證 - 升級驗證代理程式 | Microsoft Docs
description: 本文說明如何將 Azure Active Directory (Azure AD) 傳遞驗證組態升級。
services: active-directory
keywords: Azure AD Connect 傳遞驗證, 安裝 Active Directory, Azure AD 的必要元件, SSO, 單一登入
documentationcenter: ''
author: billmath
manager: mtillman
ms.assetid: 9f994aca-6088-40f5-b2cc-c753a4f41da7
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/27/2018
ms.component: hybrid
ms.author: billmath
ms.custom: seohack1
ms.openlocfilehash: ba5c3704d89f75617683ae886da1857d51251631
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46306287"
---
# <a name="azure-active-directory-pass-through-authentication-upgrade-preview-authentication-agents"></a>Azure Active Directory 傳遞驗證：將預覽驗證代理程式升級

## <a name="overview"></a>概觀

本文適用於透過預覽版使用 Azure AD 傳遞驗證的客戶。 我們最近已將驗證代理程式軟體升級 (及改版)。 您需要_手動_升級在內部部署伺服器上安裝的預覽驗證代理程式。 此手動升級只是一次性動作。 驗證代理程式的所有未來更新都是自動的。 升級的原因如下所示：

- 預覽版的驗證代理程式不會收到任何進一步的安全性或錯誤修正。
-   預覽版的驗證代理程式無法安裝在其他伺服器上，以獲得高可用性。

## <a name="check-versions-of-your-authentication-agents"></a>檢查驗證代理程式的版本

### <a name="step-1-check-where-your-authentication-agents-are-installed"></a>步驟 1：檢查驗證代理程式的安裝位置

遵循下列步驟來檢查驗證代理程式的安裝位置：

1. 使用租用戶的全域管理員認證來登入 [Azure Active Directory 管理中心](https://aad.portal.azure.com)。
2. 按一下左側導覽上的 [Azure Active Directory]。
3. 選取 [Azure AD Connect]。 
4. 選取 [傳遞驗證]。 此刀鋒視窗會列出驗證代理程式的安裝位置。

![Azure Active Directory 管理中心 - 傳遞驗證刀鋒視窗](./media/how-to-connect-pta-upgrade-preview-authentication-agents/pta8.png)

### <a name="step-2-check-the-versions-of-your-authentication-agents"></a>步驟 2：檢查驗證代理程式的版本

若要檢查驗證代理程式的版本，請在前一個步驟中識別的每部伺服器上，依照下列指示操作：

1. 移至內部部署伺服器上的 [控制台]-> [程式]-> [程式和功能]。
2. 如果有「Microsoft Azure AD Connect 驗證代理程式」項目，您就不需要在此伺服器上採取任何動作。
3. 如果有適用於「**Microsoft Azure AD 應用程式 Proxy 連接器**」的項目，您需要在此伺服器上以手動方式進行升級。

![驗證代理程式的預覽版本](./media/how-to-connect-pta-upgrade-preview-authentication-agents/pta6.png)

## <a name="best-practices-to-follow-before-starting-the-upgrade"></a>開始升級之前所應遵循的最佳做法

升級之前，請確定您已備妥下列項目：

1. **建立僅限雲端的全域管理員帳戶**：在傳遞驗證代理程式無法正常運作的緊急情況下，若沒有僅限雲端的全域管理員帳戶可使用，則不要升級。 了解如何[新增僅限雲端管理員帳戶 (英文)](../active-directory-users-create-azure-portal.md)。 這是確保您不會被租用戶封鎖的關鍵步驟。
2.  **確保高可用性**：如果先前未完成，則使用相關[指示](how-to-connect-pta-quick-start.md#step-4-ensure-high-availability)來安裝第二個獨立驗證代理程式，以提供高可用性來滿足登入要求。

## <a name="upgrading-the-authentication-agent-on-your-azure-ad-connect-server"></a>將 Azure AD Connect 伺服器上的驗證代理程式升級

您必須先升級 Azure AD Connect，才能在同一部伺服器上升級驗證代理程式。 在主要伺服器和暫存 Azure AD Connect 伺服器上執行下列步驟：

1. **升級 Azure AD Connect**：依照[本文](how-to-upgrade-previous-version.md)操作並升級至最新版的 Azure AD Connect。
2. **解除預覽版的安裝驗證代理程式**：下載[此 PowerShell 指令碼](https://aka.ms/rmpreviewagent)並在伺服器上以系統管理員身分執行該指令碼。
3. **下載最新版的驗證代理程式 (1.5.389.0 版或更新版本)**：使用您租用戶的全域管理員認證登入 [Azure Active Directory 管理中心](https://aad.portal.azure.com)。 選取 [Azure Active Directory] -> [Azure AD Connect] -> [傳遞驗證] -> [下載代理程式]。 接受[服務條款](https://aka.ms/authagenteula)並下載最新版的驗證代理程式。 您也可以從[這裡](https://aka.ms/getauthagent)下載驗證代理程式。
4. **安裝最新版的驗證代理程式**：執行在步驟 3 中下載的可執行檔。 出現提示時，提供您租用戶的全域管理員認證。
5. **已安裝最新版本**：如之前所示，請移至 [控制台]-> [程式]-> [程式和功能]，並確認有 [Microsoft Azure AD Connect 驗證代理程式] 項目。

>[!NOTE]
>完成前述步驟後，若到 [Azure Active Directory 管理中心](https://aad.portal.azure.com)查看傳遞驗證刀鋒視窗，將會發現每個伺服器有兩個驗證代理程式項目：一個項目會顯示驗證代理程式為**使用中**，另一個則顯示為**非使用中**。 這是_預期行為_。 **非使用中**的項目會在幾天後自動卸除。

## <a name="upgrading-the-authentication-agent-on-other-servers"></a>在其他伺服器上升級驗證代理程式

在其他伺服器 (未安裝 Azure AD Connect) 上依照下列步驟來升級驗證代理程式：

1. **解除預覽版的安裝驗證代理程式**：下載[此 PowerShell 指令碼](https://aka.ms/rmpreviewagent)並在伺服器上以系統管理員身分執行該指令碼。
2. **下載最新版的驗證代理程式 (1.5.389.0 版或更新版本)**：使用您租用戶的全域管理員認證登入 [Azure Active Directory 管理中心](https://aad.portal.azure.com)。 選取 [Azure Active Directory] -> [Azure AD Connect] -> [傳遞驗證] -> [下載代理程式]。 接受服務條款並下載最新版本。
3. **安裝最新版的驗證代理程式**：執行在步驟 2 中下載的可執行檔。 出現提示時，提供您租用戶的全域管理員認證。
4. **已安裝最新版本**：如之前所示，請移至 [控制台]-> [程式]-> [程式和功能]，並確認有一個叫做 [Microsoft Azure AD Connect 驗證代理程式] 的項目。

>[!NOTE]
>完成前述步驟後，若到 [Azure Active Directory 管理中心](https://aad.portal.azure.com)查看傳遞驗證刀鋒視窗，將會發現每個伺服器有兩個驗證代理程式項目：一個項目會顯示驗證代理程式為**使用中**，另一個則顯示為**非使用中**。 這是_預期行為_。 **非使用中**的項目會在幾天後自動卸除。

## <a name="next-steps"></a>後續步驟
- [**疑難排解**](tshoot-connect-pass-through-authentication.md) - 了解如何解決此功能的常見問題。
