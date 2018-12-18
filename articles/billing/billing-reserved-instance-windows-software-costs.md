---
title: Azure 保留的 Windows 軟體成本 | Microsoft Docs
description: 了解哪些 Windows 軟體計量不包含在 Azure 保留的 VM 執行個體成本中。
services: billing
documentationcenter: ''
author: manish-shukla01
manager: manshuk
editor: ''
tags: billing
ms.service: billing
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 05/09/2018
ms.author: manshuk
ms.openlocfilehash: ec4869ed5c4d823a8144ba430ebb29c65c2abc13
ms.sourcegitcommit: 1fb353cfca800e741678b200f23af6f31bd03e87
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43302061"
---
# <a name="windows-software-costs-not-included-with-azure-reserved-vm-instances"></a>Windows 軟體的成本不包括在 Azure 保留的虛擬機器執行個體內

如果保留的虛擬機器執行個體上沒有 Azure Hybrid Use Benefit，則必須為下列區段所列的 Windows 軟體計量支付費用。

## <a name="windows-software-meters-not-included-in-reservation-cost"></a>Windows 軟體計量不包括在保留成本中

| 計量識別碼 | 使用量檔案中的計量名稱 | 適用的虛擬機器 |
| ------- | ------------------------| --- |
| e7e152ac-f29c-4cce-ad6e-026192c01ef2 | Reservation-Windows Svr Burst (1 核心) | B 系列 |
| cac255a2-9f0f-4c62-8bd6-f0fa449c5f76 | Reservation-Windows Svr Burst (2 核心) | B 系列 |
| 09756b58-3fb5-4390-976d-9ddd14f9ed18 | Reservation-Windows Svr Burst (4 核心) | B 系列 |
| e828cb37-5920-4dc7-b30f-664e4dbcb6c7 | Reservation-Windows Svr Burst (8 核心) | B 系列 |
| f65a06cf-c9c3-47a2-8104-f17a8542215a | Reservation-Windows Svr (1 核心) | B 系列以外皆適用 |
| b99d40ae-41fe-4d1d-842b-56d72f3d15ee | Reservation-Windows Svr (2 核心) | B 系列以外皆適用 |
| 1cb88381-0905-4843-9ba2-7914066aabe5 | Reservation-Windows Svr (4 核心) | B 系列以外皆適用 |
| 07d9e10d-3e3e-4672-ac30-87f58ec4b00a | Reservation-Windows Svr (6 核心) | B 系列以外皆適用 |
| 603f58d1-1e96-460b-a933-ce3775ac7e2e | Reservation-Windows Svr (8 核心) | B 系列以外皆適用 |
| 36aaadda-da86-484a-b465-c8b5ab292d71 | Reservation-Windows Svr (12 核心) | B 系列以外皆適用 |
| 02968a6b-1654-4495-ada6-13f378ba7172 | Reservation-Windows Svr (16 核心) | B 系列以外皆適用 |
| 175434d8-75f9-474b-9906-5d151b6bed84 | Reservation-Windows Svr (20 核心) | B 系列以外皆適用 |
| 77eb6dd0-88f5-4a16-ab39-05d1742efb25 | Reservation-Windows Svr (24 核心) | B 系列以外皆適用 |
| 0d5bdf46-b719-4b1f-a780-b9bdfffd0591 | Reservation-Windows Svr (32 核心) | B 系列以外皆適用 |
| f1214b5c-cc16-445f-be6c-a3bb75f8395a | Reservation-Windows Svr (40 核心) | B 系列以外皆適用 |
| 637b7c77-65ad-4486-9cc7-dc7b3e9a8731 | Reservation-Windows Svr (64 核心) | B 系列以外皆適用 |
| da612742-e7cc-4ca3-9334-0fb7234059cd | Reservation-Windows Svr (72 核心) | B 系列以外皆適用 |
| a485cb8c-069b-4cf3-9a8e-ddd84b323da2 | Reservation-Windows Svr (128 核心) | B 系列以外皆適用 |
| 904c5c71-1eb7-43a6-961c-d305a9681624 | Reservation-Windows Svr (256 核心) | B 系列以外皆適用 |
| 6fdab81b-4284-4df9-8939-c237cc7462fe | Reservation-Windows Svr (96 核心) | B 系列以外皆適用 |

您可以透過 Azure RateCard API 來取得這些計量中每一個的成本。 如需如何取得 Azure 計量表費率的資訊，請參閱[取得 Azure 訂用帳戶中所使用資源的價格和中繼資料資訊](https://msdn.microsoft.com/library/azure/mt219004)。

## <a name="next-steps"></a>後續步驟
若要深入了解 Azure 保留項目，請參閱下列文章：

- [什麼是 Azure 保留項目？](billing-save-compute-costs-reservations.md)
- [預付具有 Azure 保留 VM 執行個體的虛擬機器](../virtual-machines/windows/prepay-reserved-vm-instances.md)
- [管理 Azure 保留項目](billing-manage-reserved-vm-instance.md)
- [了解保留項目折扣的套用方式](billing-understand-vm-reservation-charges.md)
- [了解預付型方案訂用帳戶的保留項目使用量](billing-understand-reserved-instance-usage.md)
- [了解 Enterprise 註冊的保留項目使用量](billing-understand-reserved-instance-usage-ea.md)

## <a name="need-help-contact-support"></a>需要協助嗎？ 請連絡支援人員

如果您仍有其他問題，請[連絡支援人員](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)以快速解決您的問題。



