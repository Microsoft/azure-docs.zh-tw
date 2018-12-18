---
title: Azure Stack 實體裝置稽核
description: 了解如何將實體裝置存取稽核整合至 Azure Stack
services: azure-stack
author: PatAltimore
manager: femila
ms.service: azure-stack
ms.topic: article
ms.date: 08/01/2018
ms.author: patricka
ms.reviewer: fiseraci
keywords: ''
ms.openlocfilehash: 459cdf4e1a70ee02d818dd6abe101e4fc3475b68
ms.sourcegitcommit: 387d7edd387a478db181ca639db8a8e43d0d75f7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/10/2018
ms.locfileid: "40036678"
---
# <a name="azure-stack-datacenter-integration---physical-device-auditing"></a>Azure Stack 資料中心整合 - 實體裝置稽核

Azure Stack 中的所有實體裝置 (例如基礎板管理控制器 (BMC) 和網路交換器) 都會發出稽核記錄，並且應該將這些記錄整合至整個稽核解決方案中。 由於裝置會因為 Azure Stack OEM 硬體廠商不同而有所差異，因此，如需稽核整合文件，請連絡您的廠商。 下列各節提供一些 Azure Stack 中有關實體裝置稽核的一般資訊。  

## <a name="physical-device-access-auditing"></a>實體裝置存取稽核

Azure Stack 中的所有實體裝置都支援使用 TACACS 或 RADIUS。 這包括存取基礎板管理控制器 (BMC) 和網路交換器。

Azure Stack 解決方案在推出時並未內建 RADIUS 或 TACACS。 不過，這些解決方案已通過驗證，可支援使用市面上現有的 RADIUS 或 TACACS 解決方案。

至於 RADIUS，則僅有 MSCHAPv2 通過驗證。 這表示使用 RADIUS 的最安全實作。
請洽詢您的 OEM 硬體廠商，以在 Azure Stack 解決方案所包含的裝置中啟用 TACAS 或 RADIUS。

## <a name="syslog-forwarding-for-network-devices"></a>網路裝置的 Syslog 轉送

Azure Stack 中的所有實體網路裝置都支援 Syslog 訊息。 Azure Stack 解決方案並未隨附 Syslog 伺服器。 不過，這些裝置已通過驗證，可支援將訊息傳送到市面上現有的 Syslog 解決方案。

Syslog 目的地位址是針對部署所收集的選擇性參數，但您也可以在部署後才新增此參數。 請洽詢您的 OEM 硬體廠商，以在網路裝置上設定 syslog 轉送功能。

## <a name="next-steps"></a>後續步驟

[服務原則](azure-stack-servicing-policy.md)
