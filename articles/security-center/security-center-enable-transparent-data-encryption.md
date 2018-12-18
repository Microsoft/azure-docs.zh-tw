---
title: 在 Azure 資訊安全中心啟用透明資料加密 | Microsoft Docs
description: 本文件說明如何實作 Azure 資訊安全中心建議的「啟用透明資料加密」。
services: security-center
documentationcenter: na
author: TerryLanfear
manager: MBaldwin
editor: ''
ms.assetid: e4be8a0e-2118-4ee9-a266-69e52d9f7f8e
ms.service: security-center
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 02/02/2017
ms.author: terrylan
ms.openlocfilehash: 2a2963affdbff3710ad08f86c6ed4e6304335559
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2017
ms.locfileid: "23040263"
---
# <a name="enable-transparent-data-encryption-in-azure-security-center"></a>在 Azure 資訊安全中心啟用透明資料加密
如果尚未啟用 TDE，Azure 資訊安全中心將建議您在 SQL Database 上啟用透明資料加密 (TDE)。 TDE 可保護您的資料，並在無須變更您應用程式的情況下，加密不在作用中的資料庫、相關聯的備份以及交易記錄檔，以協助您符合法規遵循需求。 若要深入了解，請參閱 [Azure SQL Database 的透明資料加密](https://msdn.microsoft.com/library/dn948096)。

此建議僅適用於 Azure SQL 服務；不包含在虛擬機器上執行的 SQL。

> [!NOTE]
> 本文件將使用範例部署來介紹服務。  這不是逐步指南。
>
>

## <a name="implement-the-recommendation"></a>實作建議
1. 在 [建議] 刀鋒視窗中，選取 [啟用透明資料加密]。
   ![啟用透明資料加密][1]
2. 這會開啟 [在 SQL 資料庫上啟用透明資料加密]  刀鋒視窗。 選取要在其上啟用 TDE 的 SQL Database。
   ![選取要在其上啟用 TDE 的 SQL DB][2]
3. 在 [透明資料加密] 刀鋒視窗中，選取 [資料加密] 下方的 [開啟]，然後選取刀鋒視窗頂端功能區中的 [儲存]。
   ![開啟 TDE][3]

   一旦在所選取的 SQL Database 上啟用 TDE 後，[加密狀態] 就會變成 [已加密]。    

   ![加密狀態][4]

## <a name="see-also"></a>另請參閱
本文說明了如何實作資訊安全中心建議的「啟用透明資料加密」。 如要深入了解 SQL TDE，請參閱下列主題：

* [Azure SQL Database 的透明資料加密](https://msdn.microsoft.com/library/dn948096)
* [開始使用透明資料加密 (TDE)](../sql-data-warehouse/sql-data-warehouse-encryption-tde.md)

如要深入了解資訊安全中心，請參閱下列主題：

* [在 Azure 資訊安全中心設定安全性原則](security-center-policies.md) -- 了解如何為您的 Azure 訂用帳戶及資源群組設定安全性原則。
* [管理 Azure 資訊安全中心的安全性建議](security-center-recommendations.md) -- 了解建議如何協助保護您的 Azure 資源。
* [Azure 資訊安全中心的安全性健全狀況監視](security-center-monitoring.md) -- 了解如何監視 Azure 資源的健全狀況。
* [管理與回應 Azure 資訊安全中心的安全性警示](security-center-managing-and-responding-alerts.md) -- 了解如何管理與回應安全性警示。
* [使用 Azure 資訊安全中心監視合作夥伴解決方案](security-center-partner-solutions.md) -- 了解如何監視合作夥伴解決方案的健全狀況。
* [Azure 資訊安全中心常見問題集](security-center-faq.md) -- 尋找有關使用服務的常見問題。
* [Azure 安全性部落格](http://blogs.msdn.com/b/azuresecurity/) -- 取得最新的 Azure 安全性新聞和資訊。

<!--Image references-->
[1]: ./media/security-center-enable-tde-on-sql-databases/enable-tde.png
[2]:./media/security-center-enable-tde-on-sql-databases/transparent-data-encryption-blade.png
[3]: ./media/security-center-enable-tde-on-sql-databases/turn-on-tde.png
[4]: ./media/security-center-enable-tde-on-sql-databases/encrypted.png
