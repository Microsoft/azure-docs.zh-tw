---
title: Azure AD 密碼保護預覽
description: 使用 Azure AD 密碼保護 (預覽) 在內部部署 Active Directory 中禁用弱式密碼
services: active-directory
ms.service: active-directory
ms.component: authentication
ms.topic: conceptual
ms.date: 07/25/2018
ms.author: joflore
author: MicrosoftGuyJFlo
manager: mtillman
ms.reviewer: jsimmons
ms.openlocfilehash: 286f8e560ec653ed4f4f1cad5a2ae27b940f8d15
ms.sourcegitcommit: e2348a7a40dc352677ae0d7e4096540b47704374
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43781775"
---
# <a name="preview-enforce-azure-ad-password-protection-for-windows-server-active-directory"></a>預覽：強制執行 Windows Server Active Directory 的 Azure AD 密碼保護

|     |
| --- |
| Azure AD 密碼保護和自訂的禁用密碼清單是 Azure Active Directory 的公開預覽版功能。 如需預覽版的詳細資訊，請參閱 [Microsoft Azure 預覽版增補使用條款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)|
|     |

Azure AD 密碼保護是 Azure Active Directory (Azure AD) 的公開預覽版中提供的新功能，可強化組織中的密碼原則。 Azure AD 密碼保護的內部部署同時使用了儲存在 Azure AD 中的全域和自訂禁用密碼清單，並在內部部署執行與 Azure AD 雲端式變更相同的檢查。

Azure AD 密碼保護由三項軟體元件所組成：

* Azure AD 密碼保護 Proxy 服務會在目前的 Active Directory 樹系中任何已加入網域的機器上執行。 它會將來自網域控制站的要求轉送至 Azure AD，並將 Azure AD 的回應傳回至網域控制站。
* Azure AD 密碼保護 DC 代理程式服務會接收來自 DC 代理程式密碼篩選 dll 的密碼驗證要求，使用本機目前可用的密碼原則加以處理，然後傳回結果 (通過\失敗)。 這項服務負責定期 (每小時一次) 呼叫 Azure AD 密碼保護 Proxy 服務，以擷取密碼原則的新版本。 對 Azure AD 密碼保護 Proxy 服務的呼叫和來自該服務的呼叫，會經由「透過 TCP 的 RPC (遠端程序呼叫)」來處理其通訊。 經擷取後，新原則會儲存在後續可供其複寫至其他網域控制站的 sysvol 資料夾中。 DC 代理程式服務也會監視在有其他網域控制站於 sysvol 資料夾寫入了新的密碼原則時，該資料夾是否有所變更。如果已有適當的近期原則可供使用，則會略過 Azure AD 密碼保護 Proxy 服務的檢查。
* DC 代理程式密碼篩選 dll 會接收來自作業系統的密碼驗證要求，並將其轉送至在網域控制站上本機執行的 Azure AD 密碼保護 DC 代理程式服務。

![Azure AD 密碼保護元件搭配運作的方式](./media/concept-password-ban-bad-on-premises/azure-ad-password-protection.png)

## <a name="requirements"></a>需求

* 安裝 Azure AD 密碼保護元件的所有機器 (包括網域控制站)，都必須執行 Windows Server 2012 或更新版本。
* 安裝 Azure AD 密碼保護元件的所有機器 (包括網域控制站)，都必須安裝 Universal C 執行階段。 透過 Windows Update 來完整修補此機器，是達到此需求的較佳做法。 否則，可能會安裝適當的 OS 專屬更新套件 - 請參閱[更新 Windows 中的 Universal C 執行階段](https://support.microsoft.com/help/2999226/update-for-universal-c-runtime-in-windows)
* 每個網域中至少要有一個網域控制站有網路連線，且至少要有一部伺服器裝載 Azure AD 密碼保護 Proxy 服務。
* 任何使用密碼保護功能的 Active Directory 網域控制站都必須已安裝 DC 代理程式。
* 執行 DC 代理程式服務軟體的任何 Active Directory 網域，都必須使用 DFSR 進行 sysvol 複寫。
* 一個全域系統管理員帳戶，用以向 Azure AD 註冊 Azure AD 密碼保護 Proxy 服務。
* 一個在樹系根網域中具有 Active Directory 網域系統管理員權限的帳戶。

### <a name="license-requirements"></a>授權需求

Azure Active Directory (Azure AD) 的所有使用者均可受益於全域禁用密碼清單的功效。

自訂禁用密碼清單需要 Azure AD Basic 授權。

Windows Server Active Directory 的 Azure AD 密碼保護需要 Azure AD Premium 授權。

在 [Azure Active Directory 價格網站](https://azure.microsoft.com/pricing/details/active-directory/)上可以找到其他授權資訊 (包括成本)。

## <a name="download"></a>下載

Azure AD 密碼保護有兩個可從 [Microsoft 下載](https://www.microsoft.com/download/details.aspx?id=57071)的必要安裝程式

## <a name="answers-to-common-questions"></a>常見問題的解答

* 網域控制站不需要網際網路連線。 只有執行 Azure AD 密碼保護 Proxy 服務的機器才需要網際網路連線。
* 在網域控制站上不會開啟任何網路連接埠。
* 不需要 Active Directory 結構描述變更。
* 軟體會使用現有的 Active Directory 容器和 serviceConnectionPoint 結構描述物件。
* 沒有基本的 Active Directory 網域或樹系功能層級 (DFL\FFL) 需求。
* 軟體在其保護的 Active Directory 網域中不會建立或需要任何帳戶。
* 支援依權衡方式僅對安裝網域控制站代理程式的機器強制執行密碼原則的累加部署。
* 建議您在所有 DC 上安裝 DC 代理程式，以確保密碼保護的施行。 
* Azure AD 密碼保護不是即時原則應用程式引擎。 從密碼原則組態變更後，到變更同步至所有網域控制站並強制執行時，可能會有時間上的延遲。


## <a name="next-steps"></a>後續步驟

[部署 Azure AD 密碼保護](howto-password-ban-bad-on-premises.md)
