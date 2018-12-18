---
title: 保護 Azure 資訊安全中心內的  Azure SQL 服務和資料 | Microsoft Docs
description: 本文件說明「Azure 資訊安全中心」中的建議，這些建議可協助您保護您的資料和 Azure SQL 服務，以及遵守安全性原則。
services: security-center
documentationcenter: na
author: TerryLanfear
manager: MBaldwin
editor: ''
ms.assetid: bcae6987-05d0-4208-bca8-6a6ce7c9a1e3
ms.service: security-center
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 04/03/2017
ms.author: terrylan
ms.openlocfilehash: 45f5dc840f015793912e314ab3d47e54a409708e
ms.sourcegitcommit: f10653b10c2ad745f446b54a31664b7d9f9253fe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2018
ms.locfileid: "46126661"
---
# <a name="protecting-azure-sql-service-and-data-in-azure-security-center"></a>保護 Azure 資訊安全中心內的 Azure SQL 服務和資料
「Azure 資訊安全中心」會分析 Azure 資源的安全性狀態。 當資訊安全中心發現潛在的安全性弱點時，它會建立可引導您完成所需控制之設定程序的建議。  這些建議適用於下列 Azure 資源類型︰虛擬機器 (VM)、網路、SQL 和資料，以及應用程式。

本文說明適用於 Azure SQL 服務和資料的建議。 這些建議是以下列主題為中心：為 Azure SQL 伺服器和資料庫啟用稽核、為 SQL 資料庫啟用加密，以及啟用 Azure 儲存體帳戶加密。  請使用下表作為參考來協助您了解可用的 SQL 服務和資料建議，以及每一項建議在套用後將產生的作用。
### <a name="monitor-data-security"></a>監視資料安全性

當您按一下 [防護] 區段中的 [資料安全性] 時，隨即會開啟包含 SQL 和儲存體建議的 [資料資源]。 它也具有資料庫的一般健全狀況狀態 [建議](security-center-sql-service-recommendations.md) 。 如需儲存體加密的詳細資訊，請閱讀[在 Azure 資訊安全中心啟用 Azure 儲存體帳戶的加密](security-center-enable-encryption-for-storage-account.md)。

![資料資源](./media/security-center-monitoring/security-center-monitoring-fig13-newUI-2017.png)

在 [SQL 建議] 底下，您可以按一下任何建議，取得解決問題所需之進一步動作的更多詳細資料。 下列範例顯示已展開的 **SQL Database 上的資料庫稽核和威脅偵測**建議。

![有關 SQL 建議的詳細資料](./media/security-center-monitoring/security-center-monitoring-fig14-ga-new.png)

[在 SQL 資料庫上啟用稽核和威脅偵測] 具有下列資訊：

* SQL 資料庫的清單
* 其所在的伺服器
* 此設定是否繼承自伺服器，或此設定在此資料庫中是否為唯一的相關資訊
* 目前的狀態
* 問題的嚴重性

當您在資料庫上按一下以解決這項建議時，便會開啟 [稽核與威脅的偵測]，如下列畫面所示。

![稽核與威脅偵測](./media/security-center-monitoring/security-center-monitoring-fig15-ga.png)

若要啟用稽核，請選取 [稽核] 選項底下的 [開啟]。

## <a name="available-sql-service-and-data-recommendations"></a>可用的 SQL 服務和資料建議
| 建議 | 說明 |
| --- | --- |
| [在 SQL Server 上啟用稽核與威脅偵測](security-center-enable-auditing-on-sql-servers.md) |建議您針對 Azure SQL Server 開啟稽核與威脅偵測 (僅適用於 Azure SQL 服務，不包括在您虛擬機器上執行的 SQL)。 |
| [在 SQL 資料庫上啟用稽核與威脅偵測](security-center-enable-auditing-on-sql-databases.md) |建議您針對 Azure SQL Database 開啟稽核與威脅偵測 (僅適用於 Azure SQL 服務，不包括在您虛擬機器上執行的 SQL)。 |
| [在 SQL 資料庫上啟用透明資料加密](security-center-enable-transparent-data-encryption.md) |建議您針對 SQL 資料庫啟用加密 (僅適用於 Azure SQL 服務)。 |

## <a name="see-also"></a>另請參閱
若要深入了解適用於其他 Azure 資源類型的建議，請參閱下列文章︰

* [保護 Azure 資訊安全中心內的虛擬機器](security-center-virtual-machine-recommendations.md)
* [保護 Azure 資訊安全中心內的應用程式](security-center-application-recommendations.md)
* [保護 Azure 資訊安全中心內的網路](security-center-network-recommendations.md)

如要深入了解資訊安全中心，請參閱下列主題：

* [在 Azure 資訊安全中心設定安全性原則](security-center-policies.md) -- 了解如何為您的 Azure 訂用帳戶及資源群組設定安全性原則。
* [管理與回應 Azure 資訊安全中心的安全性警示](security-center-managing-and-responding-alerts.md) -- 了解如何管理與回應安全性警示。
* [Azure 資訊安全中心常見問題集](security-center-faq.md) -- 尋找有關使用服務的常見問題。
