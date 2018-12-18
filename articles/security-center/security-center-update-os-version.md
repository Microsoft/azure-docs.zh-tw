---
title: Azure 資訊安全中心的更新作業系統版本 | Microsoft Docs
description: 本文說明如何實作 Azure 資訊安全中心建議的**更新作業系統版本**。
services: security-center
documentationcenter: na
author: TerryLanfear
manager: MBaldwin
editor: ''
ms.assetid: aa372492-ecdb-4368-8fdd-d8ed31e216ee
ms.service: security-center
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 11/01/2016
ms.author: terrylan
ms.openlocfilehash: cab773412005093f8acabe9bd6e93e8740b14ece
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44298400"
---
# <a name="update-os-version-in-azure-security-center"></a>Azure 資訊安全中心的更新作業系統版本
對於雲端服務中的虛擬機器 (VM)，如果有更新的版本可用，則 Azure 資訊安全中心會建議更新作業系統 (OS)。  只監視生產位置中執行的雲端服務 Web 角色和背景工作角色。

> [!NOTE]
> 本文件將使用範例部署來介紹服務。  這不是逐步指南。
> 
> 

## <a name="implement-the-recommendation"></a>實作建議
1. 在 [建議] 刀鋒視窗中，選取 [更新作業系統版本]。
   ![更新作業系統版本][1]
2. [更新作業系統版本] 刀鋒視窗隨即開啟。 遵循此刀鋒視窗中的步驟來更新作業系統版本。

## <a name="see-also"></a>另請參閱
本文說明了如何實作資訊安全中心建議的「更新作業系統版本」。 若要深入了解雲端服務和更新雲端服務的作業系統版本，請參閱：

* [雲端服務概觀](../cloud-services/cloud-services-choose-me.md)
* [如何更新雲端服務](../cloud-services/cloud-services-update-azure-service.md)
* [如何設定雲端服務](../cloud-services/cloud-services-how-to-configure-portal.md)

如要深入了解資訊安全中心，請參閱下列主題：

* [在 Azure 資訊安全中心設定安全性原則](security-center-policies.md) -- 了解如何為您的 Azure 訂用帳戶及資源群組設定安全性原則。
* [管理 Azure 資訊安全中心的安全性建議](security-center-recommendations.md) -- 了解建議如何協助保護您的 Azure 資源。
* [Azure 資訊安全中心的安全性健全狀況監視](security-center-monitoring.md) -- 了解如何監視 Azure 資源的健全狀況。
* [管理與回應 Azure 資訊安全中心的安全性警示](security-center-managing-and-responding-alerts.md) -- 了解如何管理與回應安全性警示。
* [使用 Azure 資訊安全中心監視合作夥伴解決方案](security-center-partner-solutions.md) -- 了解如何監視合作夥伴解決方案的健全狀況。
* [Azure 資訊安全中心常見問題集](security-center-faq.md) -- 尋找有關使用服務的常見問題。
* [Azure 安全性部落格](http://blogs.msdn.com/b/azuresecurity/) -- 取得最新的 Azure 安全性新聞和資訊。

<!--Image references-->
[1]: ./media/security-center-update-os-version/update-os-version.png
