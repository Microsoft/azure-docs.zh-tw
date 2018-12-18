---
title: 將 Azure 中的行動服務自動更新至 Azure 災害復原 | Microsoft Docs
description: 在使用 Azure Site Recovery 複寫 Azure 虛擬機器時，提供自動更新行動服務的概觀。
services: site-recovery
author: rajani-janaki-ram
manager: rochakm
ms.service: site-recovery
ms.topic: article
ms.date: 07/06/2018
ms.author: rajanaki
ms.openlocfilehash: 9f0a299fb8221554a3b0c9a19a616d5ba7613a67
ms.sourcegitcommit: e2ea404126bdd990570b4417794d63367a417856
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/14/2018
ms.locfileid: "45574397"
---
# <a name="automatic-update-of-the-mobility-service-in-azure-to-azure-replication"></a>將 Azure 中的行動服務自動更新至 Azure 複寫

Azure Site Recovery 每個月都會發行，可以增強現有功能或者新增新功能，並且會修正任何已知問題。 這表示為了要將服務保持在最新狀態，您必須規劃以每月的頻率部署這些修補程式。 為了避免與升級相關聯的額外負荷，使用者可以改為選擇允許 Site Recovery 來管理元件的更新。 如[架構參考](azure-to-azure-architecture.md)中針對 Azure 到 Azure 災害復原的詳細說明，行動服務會安裝在從一個 Azure 區域將虛擬機器複寫到其他區域時，在上面啟用複寫的所有 Azure 虛擬機器。 一旦您啟用自動更新，行動服務擴充會隨著每個新的版本更新。 這份文件會詳細說明下列事項：

- 自動更新的運作方式為何？
- 啟用自動更新
- 常見問題和疑難排解
 
## <a name="how-does-automatic-update-work"></a>自動更新的運作方式為何

一旦您允許 Site Recovery 管理更新，就會透過自動化帳戶 (建立在與保存庫相同的訂用帳戶中) 部署全域 Runbook (由 Azure 服務使用)。 會將一個自動化帳戶用於特定保存庫。 Runbook 會檢查保存庫中已開啟自動更新的每部虛擬機器，並且在有較新版本可用時，起始行動服務擴充功能升級。 Runbook 的預設排程是在各個複寫虛擬機器地區時區的上午 12:00 點。 Runbook 排程也可以由使用者視需要透過自動化帳戶進行修改。 

> [!NOTE]
> 啟用自動更新不需要重新啟動您的 Azure 虛擬機器，也不會影響進行中的複寫。

> [!NOTE]
> 自動化帳戶所使用作業的計費根據該月所使用的作業執行時間分鐘數，而且 500 分鐘預設包含為自動化帳戶的免費單位。 作業每日執行數量的範圍從**數秒到一分鐘**，並且**涵蓋在免費信用額度中**。

FREE UNITS INCLUDED (PER MONTH)**   PRICE Job run time    500 minutes ₹0.14/minute

## <a name="enable-automatic-updates"></a>啟用自動更新

您可以選擇允許 Site Recovery 透過下列方式管理更新：

- [在啟用複寫步驟的過程](#as-part-of-the-enable-replication-step)
- [切換保存庫內的擴充功能更新設定](#toggle-the-extension-update-settings-inside-the-vault)

### <a name="as-part-of-the-enable-replication-step"></a>在啟用複寫步驟的過程：

當您從[虛擬機器檢視](azure-to-azure-quickstart.md)或[復原服務保存庫](azure-to-azure-how-to-enable-replication.md)開始為虛擬機器啟用複寫時，您可以選擇允許 Site Recovery 管理 Site Recovery 擴充功能的更新，或者選擇手動管理更新。

![enable-replication-auto-update](./media/azure-to-azure-autoupdate/enable-rep.png)

### <a name="toggle-the-extension-update-settings-inside-the-vault"></a>切換保存庫內的擴充功能更新設定

1. 在保存庫內，瀏覽至 [管理] ->  [Site Recovery 基礎結構]
2. 在 [針對 Azure 虛擬機器] ->  [擴充功能更新設定] 底下，按一下切換以選擇您要允許「ASR 管理更新」或「手動管理」。 按一下 [檔案] 。

![vault-toggle-autuo-update](./media/azure-to-azure-autoupdate/vault-toggle.png)

> [!Important] 
> 當您選擇「允許 ASR 管理」時，會將設定套用到對應保存庫中的所有虛擬機器。


> [!Note] 
> 這兩個選項都會通知您管理更新所用的自動化帳戶。 如果您是第一次在保存庫中啟用這個功能，則將會建立新的自動化帳戶。 相同保存庫中所有後續啟用複寫將會使用先前建立的帳戶。

## <a name="common-issues--troubleshooting"></a>常見問題和疑難排解

如果自動更新有問題，您會在保存庫儀表板中的 [組態問題] 底下收到相同的通知。 

如果您嘗試啟用自動更新而失敗，請參閱下方項目以進行疑難排解。

**錯誤**：您沒有權限，無法建立 Azure 執行身分帳戶 (服務主體) 以及將參與者角色授與服務主體。 
- 建議的動作：確定登入的帳戶獲得指派「參與者」，然後再次嘗試作業。 如需指派正確權限的進一步資訊，請參閱[本](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-create-service-principal-portal#required-permissions)文件。
 
一旦開啟自動更新，只需要按一下 [修復] 按鈕，大部分的問題都可以由 Site Recovery 服務修復。

![repair-button](./media/azure-to-azure-autoupdate/repair.png)

萬一 [修復] 按鈕無法使用，請參閱擴充功能設定窗格底下顯示的錯誤訊息。

 - **錯誤**：執行身分帳戶沒有存取復原服務資源的權限。

    **建議的動作**：刪除然後[重新建立執行身分帳戶](https://docs.microsoft.com/azure/automation/automation-create-runas-account)，或確定自動化執行身分帳戶的 Azure Active Directory 應用程式具有復原服務資源的存取權。

- **錯誤**：找不到執行身分帳戶。 下列其中一個項目已刪除或未建立：Azure Active Directory 應用程式、服務主體、角色、自動化憑證資產、自動化連線資產，或憑證與連線之間的指紋不相同。 

    **建議的動作**：刪除然後[重新建立執行身分帳戶](https://docs.microsoft.com/azure/automation/automation-create-runas-account)。
