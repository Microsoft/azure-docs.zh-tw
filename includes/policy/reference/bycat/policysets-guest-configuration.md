---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 01/25/2021
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: d3d06bd9a790fe8dd83c180bf128ef78ec6401a4
ms.sourcegitcommit: fc8ce6ff76e64486d5acd7be24faf819f0a7be1d
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/26/2021
ms.locfileid: "98807117"
---
|名稱 |描述 |原則 |版本 |
|---|---|---|---|
|[稽核密碼安全性設定不安全的電腦](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policySetDefinitions/Guest%20Configuration/GuestConfiguration_WindowsPasswordSettingsAINE.json) |此方案會部署原則需求，並針對使用不安全密碼安全性設定的電腦進行稽核。 如需有關客體設定原則的詳細資訊，請造訪 [https://aka.ms/gcpol](https://aka.ms/gcpol) |9 |1.0.0 |
|[部署必要條件，以在虛擬機器上啟用來賓設定原則](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policySetDefinitions/Guest%20Configuration/GuestConfiguration_Prerequisites.json) |此計畫會新增系統指派的受控識別，並將適合平台的客體設定延伸模組部署到可由客體設定原則監視的虛擬機器。 這是所有客體設定原則的必要條件，必須先指派給原則指派範圍，才能使用任何客體設定原則。 如需有關客體設定的詳細資訊，請造訪 [https://aka.ms/gcpol](https://aka.ms/gcpol)。 |4 |1.0.0-preview |
|[Windows 電腦應符合 Azure 安全性基準的需求](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policySetDefinitions/Guest%20Configuration/GuestConfiguration_AzureBaseline.json) |此方案使用不符合 Azure 安全性基準的設定來稽核 Windows 電腦。 如需詳細資訊，請參閱 [https://aka.ms/gcpol](https://aka.ms/gcpol) |29 |2.0.0-preview |
