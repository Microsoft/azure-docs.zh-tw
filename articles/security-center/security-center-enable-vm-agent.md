---
title: 在 Azure 資訊安全中心啟用 VM 代理程式 | Microsoft Docs
description: 本文件說明如何實作 Azure 資訊安全中心建議的「啟用 VM 代理程式」。
services: security-center
documentationcenter: na
author: TerryLanfear
manager: MBaldwin
editor: ''
ms.assetid: 5b431c25-4241-45b7-9556-cf2a1956f3da
ms.service: security-center
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 02/02/2017
ms.author: terrylan
ms.openlocfilehash: 337a7adfd93c76882a749685702bea6d1524c96a
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23040283"
---
# <a name="enable-vm-agent-in-azure-security-center"></a>在 Azure 資訊安全中心啟用 VM 代理程式
VM 代理程式必須安裝在虛擬機器 (VM) 上，才能 [啟用資料收集](security-center-enable-data-collection.md)。  Azure 資訊安全中心可讓您查看哪些 VM 需要 VM 代理程式，而且會建議您在那些 VM 上啟用 VM 代理程式。

預設會為從 Azure Marketplace 部署的 VM 安裝「VM 代理程式」。 如需如何安裝 VM 代理程式的相關資訊，請參閱 [VM 代理程式和擴充功能 – 第 2 部分](https://azure.microsoft.com/blog/vm-agent-and-extensions-part-2/) 。

> [!NOTE]
> 本文件將使用範例部署來介紹服務。 這不是逐步指南。
>
>

## <a name="implement-the-recommendation"></a>實作建議
1. 在 [建議] 刀鋒視窗中，選取 [啟用 VM 代理程式]。
   ![啟用 VM 代理程式][1]
2. 這會開啟 [VM 代理程式遺失或沒有回應] 刀鋒視窗。 此刀鋒視窗會列出需要 VM 代理程式的 VM。 依照刀鋒視窗中的指示安裝 VM 代理程式。
   ![VM 代理程式已遺失][2]

## <a name="see-also"></a>另請參閱
如要深入了解資訊安全中心，請參閱下列主題：

* [在 Azure 資訊安全中心設定安全性原則](security-center-policies.md)--了解如何為您的 Azure 訂用帳戶及資源群組設定安全性原則。
* [管理 Azure 資訊安全中心的安全性建議](security-center-recommendations.md)-- 了解建議如何協助保護您的 Azure 資源。
* [Azure 資訊安全中心的安全性健全狀況監視](security-center-monitoring.md)-- 了解如何監視 Azure 資源的健全狀況。
* [管理與回應 Azure 資訊安全中心的安全性警示](security-center-managing-and-responding-alerts.md)-- 了解如何管理與回應安全性警示。
* [使用 Azure 資訊安全中心監視合作夥伴解決方案](security-center-partner-solutions.md) -- 了解如何監視合作夥伴解決方案的健康狀態。
* [Azure 資訊安全中心常見問題集](security-center-faq.md)-- 尋找有關使用服務的常見問題。
* [Azure 安全性部落格](http://blogs.msdn.com/b/azuresecurity/)-- 取得最新的 Azure 安全性新聞和資訊。

<!--Image references-->
[1]: ./media/security-center-enable-vm-agent/enable-vm-agent.png
[2]: ./media/security-center-enable-vm-agent/vm-agent-is-missing.png
