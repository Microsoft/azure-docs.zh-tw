---
title: 將 Microsoft Advanced Threat Analytics 連線至 Azure 資訊安全中心 | Microsoft Docs
description: 了解 Azure 資訊安全中心如何與 Microsoft Advanced Threat Analytics 整合。
services: security-center
documentationcenter: na
author: terrylan
manager: MBaldwin
editor: ''
ms.assetid: 5d80bf91-16c3-40b3-82fc-e0805e6708db
ms.service: security-center
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 03/05/2018
ms.author: yurid
ms.openlocfilehash: 1922bc25eb7072341848dd7081c31ba6b1f07a3c
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44298823"
---
# <a name="connecting-microsoft-advanced-threat-analytics-to-azure-security-center"></a>將 Microsoft Advanced Threat Analytics 連線至 Azure 資訊安全中心
本文件可協助您設定 Microsoft Advanced Threat Analytics 與 Azure 資訊安全中心之間的整合。

## <a name="why-add-advanced-threat-analytics-data"></a>為何新增 Advanced Threat Analytics 資料？
[Advanced Threat Analytics (ATA)](https://docs.microsoft.com/advanced-threat-analytics/what-is-ata) 是內部部署平台，可協助偵測可疑的使用者行為。 連線時，您就能夠在資訊安全中心檢視 ATA 所偵測到的可疑動作。 此整合可讓您在資訊安全中心檢視、相互關聯和調查與混合式雲端工作負載相關的所有安全性警示。 

## <a name="how-do-i-configure-this-integration"></a>如何設定此整合？
假設已安裝 ATA，並可在內部部署上正常運作，請遵循下列步驟來設定此整合：

1. 登入 ATA 中心，然後存取 ATA 入口網站。
2. 按一下左窗格中的 [Syslog 伺服器]。

    ![Syslog 伺服器](./media/security-center-ata-integration/security-center-ata-integration-fig1.png)

3. 在 [Syslog 伺服器端點] 欄位中，輸入 127.0.0.7 (必須是這個地址)，然後在連接埠上輸入 5114 (建議值)。 當連接埠號碼是建議的號碼時，任何唯一的連接埠應可運作。 將所有其他選項保留原狀，然後按一下 [儲存]。
4. 按一下左窗格中的 [通知]，並啟用所有 Syslog 通知 (建議動作)，如下圖所示：

    ![通知](./media/security-center-ata-integration/security-center-ata-integration-fig2.png)

5. 按一下 [檔案] 。
6. 開啟 [資訊安全中心] 儀表板。
7. 在左窗格上，按一下 [安全性解決方案]。
8. 在 **Advanced Threat Analytics** 下，按一下 [新增]**ADD**。

    ![ATA](./media/security-center-ata-integration/security-center-ata-integration-fig3.png)
    
9. 移至最後一個步驟，然後按一下 [下載代理程式]。

    ![ATA](./media/security-center-ata-integration/security-center-ata-integration-fig4.png)

10. 在 [新增非 Azure 電腦] 頁面上，選取工作區。

    ![非 Azure](./media/security-center-ata-integration/security-center-ata-integration-fig5.png)

11. 在 [直接代理程式] 頁面上，下載適當的 Windows 代理程式，並記下**工作區識別碼**和**主索引鍵**。

    ![直接代理程式](./media/security-center-ata-integration/security-center-ata-integration-fig6.png)

12. 在 ATA 中心安裝此代理程式。 在安裝期間，確定選取 [將代理程式連線至 Azure Log Analytics] 選項，並在要求時，提供*工作區識別碼*和*主索引鍵*。


一旦完成安裝之後，整合即算完成，而且您將能夠在 [搜尋] 結果中看到從 ATA 傳送到資訊安全中心的新警示。 解決方案會出現在 [安全性解決方案]頁面的 [連線的解決方案] 之下。 

## <a name="next-steps"></a>後續步驟
在本文件中，您已了解如何將 Microsoft ATA 連線至資訊安全中心。 如要深入了解資訊安全中心，請參閱下列文章：

* [將 Azure Active Directory Identity Protection 連線至 Azure 資訊安全中心](security-center-aadip-integration.md)
* [在 Azure 資訊安全中心設定安全性原則](security-center-policies.md) — 了解如何為您的 Azure 訂用帳戶及資源群組設定安全性原則。
* [管理 Azure 資訊安全中心的安全性建議](security-center-recommendations.md) — 了解建議如何協助保護您的 Azure 資源。
* [Azure 資訊安全中心的安全性健全狀況監視](security-center-monitoring.md) — 了解如何監視 Azure 資源的健全狀況。
* [管理與回應 Azure 資訊安全中心的安全性警示](security-center-managing-and-responding-alerts.md) — 了解如何管理與回應安全性警示。
* [使用 Azure 資訊安全中心監視合作夥伴解決方案](security-center-partner-solutions.md) — 了解如何監視合作夥伴解決方案的健全狀況。
- [Azure 資訊安全中心資料安全性](security-center-data-security.md) - 了解資訊安全中心如何管理及保護其中的資料。
* [Azure 資訊安全中心常見問題集](security-center-faq.md) — 尋找有關使用服務的常見問題。
* [Azure 安全性部落格](http://blogs.msdn.com/b/azuresecurity/) — 取得最新的 Azure 安全性新聞和資訊。


