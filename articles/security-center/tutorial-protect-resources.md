---
title: Azure 資訊安全中心教學課程 - 使用 Azure 資訊安全中心保護您的資源 | Microsoft Docs
description: 本教學課程說明如何設定 Just-in-Time VM 存取原則和應用程式控制原則。
services: security-center
documentationcenter: na
author: rkarlin
manager: MBaldwin
editor: ''
ms.assetid: 61e95a87-39c5-48f5-aee6-6f90ddcd336e
ms.service: security-center
ms.devlang: na
ms.topic: tutorial
ms.custom: mvc
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/30/2018
ms.author: rkarlin
ms.openlocfilehash: 1d3d775a13f83ada6ca702eb984262668507f7e9
ms.sourcegitcommit: d211f1d24c669b459a3910761b5cacb4b4f46ac9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/06/2018
ms.locfileid: "44025441"
---
# <a name="tutorial-protect-your-resources-with-azure-security-center"></a>教學課程：使用 Azure 資訊安全中心保護您的資源
資訊安全中心使用存取和應用程式控制原則來阻擋惡意活動，以限制您暴露於威脅的風險。 Just-in-Time 虛擬機器 (VM) 存取透過讓您拒絕對 VM 的持續存取，進而減少您暴露於攻擊的風險。 不過，您可以只在需要的時候，提供對 VM 的受控制及稽核的存取。 自適性應用程式控制透過控制可在 VM 上執行的應用程式，進而協助強化 VM 以抵禦惡意軟體。 資訊安全中心會利用機器學習服務來分析在 VM 中執行的程序，並協助您利用此情報來套用列入允許清單規則。

在本教學課程中，您將了解如何：

> [!div class="checklist"]
> * 設定 Just-In-Time VM 存取原則
> * 設定應用程式控制原則

如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="prerequisites"></a>必要條件
若要逐步執行本教學課程中涵蓋的功能，您必須是在資訊安全中心的標準定價層。 前 60 天可以免費試用資訊安全中心標準定價層。 [將 Azure 訂用帳戶上架到資訊安全中心標準定價層](security-center-get-started.md)快速入門為您逐步解說如何升級至「標準」定價層。

## <a name="manage-vm-access"></a>管理 VM 存取
Just-In-Time VM 存取可用於鎖定 Azure VM 的連入流量，進而降低暴露於攻擊的風險，同時讓您視需要輕鬆地連線至 VM。

管理連接埠不需要隨時保持開啟。 只有在連線至 VM 時 (例如進行執行管理或維修工作)，才需要將管理連接埠開啟。 啟用 Just-In-Time 之後，資訊安全中心會使用「網路安全性群組」(NSG) 規則，以限制對管理連接埠的存取，讓攻擊者無法將這些連接埠作為攻擊目標。

1. 在 [資訊安全中心] 主功能表中，選取 [進階雲端防禦] 下的 [Just-in-Time VM 存取]。

  ![Just-In-Time 虛擬機器存取][1]

  [Just-In-Time VM 存取] 會提供 VM 狀態的相關資訊：

  - [已設定] - 已設定為支援 Just-In-Time VM 存取的 VM。
  - [建議] - 可支援但未設定 Just-In-Time VM 存取的 VM。
  - [不建議] - 可能會導致不建議 VM 進行設定的原因如下：

    - 缺少 NSG - Just-In-Time 解決方案需要 NSG。
    - 傳統 VM - 資訊安全中心 Just-In-Time VM 存取目前僅支援透過 Azure Resource Manager 部署的 VM。
    - 其他 - 如果訂用帳戶或資源群組的安全性原則已關閉 Just-In-Time 解決方案，或 VM 缺少公用 IP 且未設定 NSG，則該 VM 也屬於此類別。

2. 選取建議的 VM 並按一下 [在 1 VM 上啟用 JIT]，來為該 VM 設定 Just-In-Time 原則：

  您可以儲存資訊安全中心建議的預設連接埠，或是新增並設定您要在其上啟用 Just-In-Time 解決方案的連接埠。 在本教學課程中，讓我們選取 [新增] 來新增連接埠。

  ![新增連接埠設定][2]

3. 您可以在 [新增連接埠設定] 下看到：

  - 連接埠
  - 通訊協定類型
  - 允許的來源 IP - 收到核准的要求時允許取得存取權的 IP 範圍
  - 要求時間上限 - 開啟特定連接埠的時間範圍上限

4. 選取 [確定] 以儲存。

## <a name="harden-vms-against-malware"></a>強化 VM 以抵禦惡意軟體
自適性應用程式控制可協助您定義一組可以在設定之資源群組上執行的應用程式，再加上其他的好處可共同協助強化您的 VM 以抵禦惡意軟體。 資訊安全中心會利用機器學習服務來分析在 VM 中執行的程序，並協助您利用此情報來套用列入允許清單規則。

此功能只適用於 Windows 電腦。

1. 返回 [資訊安全中心] 主功能表。 在 [進階雲端防禦] 下，選取 [自適性應用程式控制]。

   ![自適性應用程式控制][3]

  [資源群組] 區段包含三個索引標籤：

  - **已設定**：內含已設定應用程式控制之 VM 的資源群組清單。
  - **建議**：建議採用應用程式控制的資源群組清單。
  - **無建議**：內含無任何應用程式控制建議之 VM 的資源群組清單。 例如，其上的應用程式一直改變且尚未達到穩定狀態的 VM。

2. 選取 [建議] 索引標籤以顯示具有應用程式控制建議的資源群組清單。

  ![應用程式控制建議][4]

3. 選取資源群組以開啟 [建立應用程式控制規則] 選項。 在 [選取 VM] 中，檢閱建議的 VM 清單，並取消選取任何不想套用應用程式控制的 VM。 在 [選取列入允許清單規則的程序] 中，檢閱建議的應用程式清單，並取消選取任何不想套用的規則。 此清單包括：

  - **名稱**：完整應用程式路徑
  - **處理序**：每個路徑內有多少個應用程式
  - **通用**：[是] 表示這些處理序已在此資源群組中的大部分 VM 上執行
  - **可利用進行攻擊**：警告圖示將會指出攻擊者是否可能使用應用程式來略過應用程式允許清單。 建議您在核准之前檢閱這些應用程式。

4. 一旦完成您的選擇，請選取 [建立]。

## <a name="clean-up-resources"></a>清除資源
此集合中的其他快速入門和教學課程會以本快速入門為基礎。 如果您打算繼續處理後續的快速入門和教學課程，請繼續執行標準層，並保持將自動佈建維持為啟用狀態。 如果您不打算繼續，或是要返回免費層：

1. 返回 [資訊安全中心] 主功能表，並選取 [安全性原則]。
2. 選取您需要返回免費層的訂用帳戶或原則。 [安全性原則] 隨即開啟。
3. 在 [原則元件] 下，選取 [定價層]。
4. 選取 [免費] 以將訂用帳戶從標準層變更為免費層。
5. 選取 [ **儲存**]。

如果您需要停用自動佈建：

1. 返回 [資訊安全中心] 主功能表，並選取 [安全性原則]。
2. 選取您想要停用自動佈建的訂用帳戶。
3. 在 [安全性原則 - 資料收集] 下，選取 [上架] 底下的 [關閉] 以停用自動佈建。
4. 選取 [ **儲存**]。

>[!NOTE]
> 停用自動佈建不會從已佈建代理程式的 Azure VM 移除 Microsoft Monitoring Agent。 停用自動佈建會限制對資源的安全性監視。
>

## <a name="next-steps"></a>後續步驟
在本教學課程中，您已了解如何透過下列方式限制暴露於威脅的風險：

> [!div class="checklist"]
> * 設定 Just-In-Time VM 存取原則，以只在需要時提供對 VM 的受控制及稽核的存取
> * 設定自適性應用程式控制原則，以控制哪些應用程式可在您 VM 上執行

請前進到下一個教學課程，以了解如何回應安全性事件。

> [!div class="nextstepaction"]
> [教學課程：回應安全性事件](tutorial-security-incident.md)

<!--Image references-->
[1]: ./media/tutorial-protect-resources/just-in-time-vm-access.png
[2]: ./media/tutorial-protect-resources/add-port.png
[3]: ./media/tutorial-protect-resources/adaptive-application-control-options.png
[4]: ./media/tutorial-protect-resources/recommended-resource-groups.png
