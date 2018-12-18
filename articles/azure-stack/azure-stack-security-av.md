---
title: 更新 Azure Stack 上的 Windows Defender 防毒軟體
description: 如何在 Azure Stack 上保有最新防毒軟體的詳細資訊
services: azure-stack
author: PatAltimore
manager: femila
ms.service: azure-stack
ms.topic: article
ms.date: 09/26/2018
ms.author: patricka
ms.reviewer: fiseraci
ms.openlocfilehash: 59c59705e840d3cdd0d3c5310d84d711cd00c96a
ms.sourcegitcommit: d1aef670b97061507dc1343450211a2042b01641
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47394003"
---
# <a name="update-windows-defender-antivirus-on-azure-stack"></a>更新 Azure Stack 上的 Windows Defender 防毒軟體

[Windows Defender 防毒軟體](https://docs.microsoft.com/windows/security/threat-protection/windows-defender-antivirus/windows-defender-antivirus-in-windows-10)是一種反惡意程式碼解決方案，可提供安全性和防毒保護。 每個 Azure Stack 基礎結構元件 (Hyper-V 主機和虛擬機器) 都受到 Windows Defender 防毒軟體的保護。 為了提供最新的保護，Windows Defender 防毒軟體的定義、引擎和平台都需要定期更新。 套用更新的方式取決於您的組態。

## <a name="connected-scenario"></a>已連線的案例

為了進行反惡意程式碼定義和引擎的更新，Azure Stack [更新資源提供者](azure-stack-updates.md#the-update-resource-provider)每天會下載反惡意程式碼定義和引擎更新數次。 每個 Azure Stack 基礎結構元件都會取得來自於更新資源提供者的更新，並自動套用更新。

針對反惡意程式碼平台更新，請套用[每個月的 Azure Stack 更新](azure-stack-apply-updates.md)。 每個月的 Azure Stack 更新包括當月的 Windows Defender 防毒軟體平台更新。

## <a name="disconnected-scenario"></a>中斷連線的案例

 請套用[每個月的 Azure Stack 更新](azure-stack-apply-updates.md)。 每個月的 Azure Stack 更新包括當月的 Windows Defender 防毒軟體定義、引擎和平台更新。

## <a name="next-steps"></a>後續步驟

[深入了解 Azure Stack 安全性](azure-stack-security-foundations.md)
