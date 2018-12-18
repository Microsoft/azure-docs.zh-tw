---
title: 在 Azure 資訊安全中心管理連線的合作夥伴解決方案 | Microsoft Docs
description: 本文件逐步引導您使用 Azure 資訊安全中心，讓您監視與您的 Azure 訂用帳戶整合之合作夥伴解決方案的健康狀態，一目了然。
services: security-center
documentationcenter: na
author: TerryLanfear
manager: MBaldwin
editor: ''
ms.assetid: 70c076ef-3ad4-4000-a0c1-0ac0c9796ff1
ms.service: security-center
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/20/2018
ms.author: terrylan
ms.openlocfilehash: 3d4c9c79ff90ee816719f0179fbb5096b0035854
ms.sourcegitcommit: 3f8f973f095f6f878aa3e2383db0d296365a4b18
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/20/2018
ms.locfileid: "42142331"
---
# <a name="managing-connected-partner-solutions-with-azure-security-center"></a>在 Azure 資訊安全中心管理連線的合作夥伴解決方案
這篇文章會逐步解說如何管理和監視 Azure 資訊安全中心連線的安全性解決方案。

## <a name="monitoring-partner-solutions"></a>監視合作夥伴解決方案
若要監視連線的安全性解決方案之健康情況狀態，並執行基本管理：

1. 在 [資訊安全中心 - 概觀] 底下，選取 [安全性解決方案]。

  ![選取安全性解決方案][1]

  [連線的解決方案] 區段包含連線到資訊安全中心的安全性解決方案，和每個解決方案健康情況狀態的的資訊。

  ![合作夥伴解決方案][2]

   合作夥伴解決方案的狀態可以是︰

   * 狀況良好 (綠色) - 沒有任何健康問題。
   * 狀況不良 (紅色) - 有需要立即注意的健康狀態問題。
   * 健康情況問題 (橘色) - 解決方案已停止報告其健康情況。
   * 未報告 (灰色) - 解決方案尚未報告任何狀態，如果解決方案最近連線且仍在部署中，或者沒有健康情況資料可用，則可能未報告解決方案的狀態。

   > [!NOTE]
   > 如果健康情況狀態資料無法使用，資訊安全中心就會顯示最後收到之事件的日期和時間，以指出解決方案是否進行報告。 如果沒有健康情況資料可用，且過去 14 天內未收到任何警示，則資訊安全中心會表示解決方案狀況不良或未報告。
   >
   >

2. 選取 [檢視] 以取得其他資訊和選項，包括：

  - **解決方案主控台**。 開啟這個解決方案的管理體驗。
  - **連結 VM**。 開啟 [連結應用程式] 刀鋒視窗。 您可以在這裡將資源連接到合作夥伴解決方案。
  - **刪除解決方案**。
  - **設定**。

   ![合作夥伴解決方案詳細資料][3]

## <a name="next-steps"></a>後續步驟
在這篇文章中，您會了解如何管理和監視 資訊安全中心連線的安全性解決方案。 如要深入了解資訊安全中心，請參閱下列主題：

* [安全性解決方案概觀](security-center-partner-integration.md) — 了解如何連線和管理安全性解決方案。
* [連線 Microsoft Advanced Threat Analytics (ATA)](security-center-ata-integration.md) — 了解如何從 ATA 連線警示。
* [連線 Azure Active Directory (AD) Identity Protection](security-center-aadip-integration.md) — 了解如何從 Azure AD Identity Protection 連線警示。
* [合作夥伴和解決方案整合](security-center-partner-integration.md) — 取得整合其他安全性解決方案的概觀。
* [管理與回應安全性警示](security-center-managing-and-responding-alerts.md) — 了解如何管理與回應安全性警示。
* [Azure 資訊安全中心常見問題集](security-center-faq.md) — 尋找有關使用服務的常見問題。
* [Azure 安全性部落格](http://blogs.msdn.com/b/azuresecurity/) — 尋找有關 Azure 安全性與相容性的部落格文章。

<!--Image references-->
[1]: ./media/security-center-partner-solutions/partner-solutions-tile.png
[2]: ./media/security-center-partner-solutions/partner-solutions.png
[3]: ./media/security-center-partner-solutions/partner-solutions-detail.png
