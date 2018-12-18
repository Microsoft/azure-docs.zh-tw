---
title: Azure 資訊安全中心快速入門 - 將 Azure 訂用帳戶上架到資訊安全中心的標準層 | Microsoft Docs
description: 本快速入門示範如何升級至資訊安全中心的標準定價層，以獲得額外的安全性。
services: security-center
documentationcenter: na
author: rkarlin
manager: MBaldwin
editor: ''
ms.assetid: 61e95a87-39c5-48f5-aee6-6f90ddcd336e
ms.service: security-center
ms.devlang: na
ms.topic: quickstart
ms.custom: mvc
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/2/2018
ms.author: rkarlin
ms.openlocfilehash: dc4c07b54085dfea6d97f2ca43a8cdddfcb9e57c
ms.sourcegitcommit: d211f1d24c669b459a3910761b5cacb4b4f46ac9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/06/2018
ms.locfileid: "44025459"
---
# <a name="quickstart-onboard-your-azure-subscription-to-security-center-standard"></a>快速入門：將 Azure 訂用帳戶上架到資訊安全中心的標準層
Azure 資訊安全中心為混合式雲端工作負載提供統一的安全性管理和威脅防護。 免費層只為 Azure 資源提供有限的安全性，而標準層會將這些功能延伸至內部部署與其他雲端。 資訊安全中心的標準層可協助您尋找和修正安全性弱點、套用存取和應用程式控制項以封鎖惡意活動、使用分析和情報來偵測威脅，以及在遭受攻擊時迅速回應。 前 60 天可以免費試用資訊安全中心標準定價層。

在本文中，您可升級到標準層以取得額外的安全性，並且在您的虛擬機器上安裝 Microsoft Monitoring Agent 以監視安全性弱點和威脅。

## <a name="prerequisites"></a>必要條件
若要開始使用資訊安全中心，您必須有 Microsoft Azure 訂用帳戶。 如果您沒有訂用帳戶，可以註冊[免費帳戶](https://azure.microsoft.com/pricing/free-trial/)。

若要將訂用帳戶升級到標準層，您必須取得「訂用帳戶擁有者」、「訂用帳戶參與者」或「安全性管理員」角色。

## <a name="enable-your-azure-subscription"></a>啟用您的 Azure 訂用帳戶

1. 登入 [Azure 入口網站](https://azure.microsoft.com/features/azure-portal/)。
2. 在 [Microsoft Azure] 功能表中，選取 [資訊安全中心]。 [資訊安全中心 - 概觀] 隨即開啟。

 ![資訊安全中心概觀][2]

[資訊安全中心 - 概觀] 可讓您統一檢視混合式雲端工作負載的安全性狀態、探索並評定工作負載的安全性，以及識別和降低風險。 資訊安全中心會自動啟用先前並未由您或另一個訂用帳戶使用者上架到免費層的任何 Azure 訂用帳戶。

按一下 [訂用帳戶] 功能表項目，即可檢視和篩選訂用帳戶清單。 資訊安全中心會立即開始評估這些訂用帳戶的安全性，以識別安全性弱點。 若要自訂評定的類型，您可以修改安全性原則。 安全性原則會定義工作負載的所需設定，並且協助確保符合公司或法規安全性需求。

在第一次啟動資訊安全中心的數分鐘內，您可能看到：

- 改善 Azure 訂用帳戶安全性的 [建議]。 按一下 [建議] 圖格會啟動已設定優先順序的清單。
- 資訊安全中心正在評估的 [計算和應用程式]、[網路]、[資料安全性] 和 [身分識別與存取權] 資源的詳細目錄，以及每項資源的安全性狀態。

若要充分利用資訊安全中心，您需要完成下列步驟，才能升級到標準層並安裝 Microsoft Monitoring Agent。

## <a name="upgrade-to-the-standard-tier"></a>升級至標準層
基於資訊安全中心快速入門和教學課程的目的，您必須升級至標準層。 前 60 天免費使用，而且您可隨時回到免費層。

1. 在 [資訊安全中心] 主功能表下，選取 [開始使用]。
 
  ![開始使用][4]

2. 在 [升級] 下，資訊安全中心會列出符合上架資格的訂用帳戶和工作區。 
   - 您可以按一下可擴展的 [適用試用版] 以查看所有訂用帳戶和工作區的清單與其試用版的資格狀態。
   -    您可以將沒有資格使用試用版的訂用帳戶和工作區升級。
   -    您可以選取符合資格的工作區和訂用帳戶來啟動試用版。
3.  按一下 [啟動試用版] 來啟動所選訂用帳戶上的試用版。


  ![安全性警示][9]

## <a name="automate-data-collection"></a>自動收集資料
資訊安全中心會從您的 Azure 虛擬機器和非 Azure 電腦收集資料，以監視安全性漏洞與威脅。 資料是使用 Microsoft Monitoring Agent 收集而得，收集的方式是讀取機器的各種安全性相關組態和事件記錄檔，並將資料複製到工作區進行分析。 根據預設，資訊安全中心會為您建立新的工作區。

啟用自動佈建時，資訊安全中心會在所有支援的 Azure 虛擬機器和任何新建立的虛擬機器上安裝 Microsoft Monitoring Agent。 強烈建議使用自動佈建。

若要啟用 Microsoft Monitoring Agent 的自動佈建：

1. 在 [資訊安全中心] 主功能表下，選取 [安全性原則]。
2. 在訂用帳戶的資料列，選取 [編輯設定] >。
3. 在 [資料收集] 索引標籤中，將 [自動佈建] 設定為 [開啟]。
4. 選取 [ **儲存**]。
****
  ![啟用自動佈建][6]

利用 Azure 虛擬機器的這項深入解析，資訊安全中心可提供與系統更新狀態、作業系統安全性組態、端點防護相關的額外建議，以及產生額外的安全性警示。

  ![建議][8]

## <a name="clean-up-resources"></a>清除資源
此集合中的其他快速入門和教學課程會以本快速入門為基礎。 如果您打算繼續處理後續的快速入門和教學課程，請繼續執行標準層，並保持將自動佈建維持為啟用狀態。 如果您不打算繼續，或是要返回免費層：

1. 返回 [資訊安全中心] 主功能表，並選取 [安全性原則]。
2. 在您想要免費返回的訂用帳戶資料列上，選取 [編輯設定]>。
3. 選取 [定價層] 和 [免費] 以將訂用帳戶從標準層變更為免費層。
5. 選取 [ **儲存**]。

如果您需要停用自動佈建：

1. 返回 [資訊安全中心] 主功能表，並選取 [安全性原則]。
2. 在您想要停用自動佈建的訂用帳戶資料列上，選取 [編輯設定]>。
3. 在 [資料收集] 索引標籤中，將 [自動佈建] 設定為 [關閉]。
4. 選取 [ **儲存**]。

>[!NOTE]
> 停用自動佈建不會從已佈建代理程式的 Azure VM 移除 Microsoft Monitoring Agent。 停用自動佈建會限制對資源的安全性監視。
>

## <a name="next-steps"></a>後續步驟
在本快速入門中，您已升級到標準層並且佈建 Microsoft Monitoring Agent，可取得各項混合式雲端工作負載的整合式安全性管理和威脅防護。 若要深入了解如何使用資訊安全中心，請繼續進行將內部部署和其他雲端中的 Windows 電腦上架的快速入門。

> [!div class="nextstepaction"]
> [快速入門：將 Windows 電腦上架到 Azure 資訊安全中心](quick-onboard-windows-computer.md)

<!--Image references-->
[2]: ./media/security-center-get-started/overview.png
[4]: ./media/security-center-get-started/get-started.png
[5]: ./media/security-center-get-started/pricing.png
[6]: ./media/security-center-get-started/enable-automatic-provisioning.png
[7]: ./media/security-center-get-started/security-alerts.png
[8]: ./media/security-center-get-started/recommendations.png
[9]: ./media/security-center-get-started/select-subscription.png
