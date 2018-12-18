---
title: Azure 資訊安全中心快速入門 - 將您的 Windows 電腦上架到資訊安全中心 | Microsoft Docs
description: 本快速入門說明如何在 Windows 電腦上佈建 Microsoft Monitoring Agent。
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
ms.openlocfilehash: 55bb7242427bf85d10fac0a44e2b2eac3a5181ad
ms.sourcegitcommit: d211f1d24c669b459a3910761b5cacb4b4f46ac9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/06/2018
ms.locfileid: "44022995"
---
# <a name="quickstart-onboard-windows-computers-to-azure-security-center"></a>快速入門：將 Windows 電腦上架到 Azure 資訊安全中心
將 Azure 訂用帳戶上架之後，您可以透過佈建 Microsoft Monitoring Agent，為在 Azure 外 (例如，內部部署或其他雲端服務) 執行的資源啟用資訊安全中心。

本快速入門說明如何在 Windows 電腦上安裝 Microsoft Monitoring Agent。

## <a name="prerequisites"></a>必要條件
若要開始使用資訊安全中心，您必須有 Microsoft Azure 訂用帳戶。 如果您沒有訂用帳戶，可以註冊[免費帳戶](https://azure.microsoft.com/pricing/free-trial/)。

確認您是使用資訊安全中心的標準定價層之後，再開始按照本快速入門操作。 如需升級指示，請參閱[將 Azure 訂用帳戶上架到資訊安全中心標準定價層](security-center-get-started.md)。 您可以在前 60 天免費試用資訊安全中心標準的定價層。

## <a name="add-new-windows-computer"></a>新增 Windows 電腦

1. 登入 [Azure 入口網站](https://azure.microsoft.com/features/azure-portal/)。
2. 在 [Microsoft Azure] 功能表中，選取 [資訊安全中心]。 [資訊安全中心 - 概觀] 隨即開啟。

 ![資訊安全中心概觀][2]

3. 在 [資訊安全中心] 主功能表下，選取 [開始使用]。
4. 選取 [開始使用] 索引標籤。

   ![開始使用][3]

5. 按一下 [新增非 Azure 電腦] 下方的 [設定]。 隨即會顯示 Log Analytics 工作區清單。 清單中顯示啟用自動佈建之後，資訊安全中心為您建立的預設工作區 (如果適用)。 選取此工作區或其他您要使用的工作區。

    ![新增非 Azure 電腦](./media/quick-onboard-windows-computer/non-azure.png)

  [直接代理程式] 刀鋒視窗提供下載 Windows 代理程式的連結，以及用於設定代理程式的工作區識別碼金鑰。

6.  選取適用於您電腦處理器類型的 [下載 Windows 代理程式] 連結以下載安裝檔。

7.  在 [工作區識別碼] 的右邊，選取複製圖示並將識別碼貼到記事本中。

8.  在 [主要金鑰] 的右邊，選取複製圖示並將識別碼貼到記事本中。

## <a name="install-the-agent"></a>安裝代理程式
接著在目的電腦上安裝下載的檔案。

1. 將檔案複製到目的電腦並執行安裝程式。
2. 在 [歡迎] 頁面上，選取 [下一步]。
3. 在 [授權條款] 頁面上，閱讀授權並選取 [我同意]。
4. 在 [目的資料夾] 頁面上，變更或保留預設的安裝資料夾，然後選取 [下一步]。
5. 在 [代理程式安裝選項] 頁面上，選擇將代理程式連線至 Azure Log Analytics，然後選取 [下一步]。
6. 在 [Azure Log Analytics] 頁面上，貼上您在先前程序中複製到「記事本」中的 [工作區識別碼] 和 [工作區金鑰 (主要金鑰)]。
7. 如果電腦應該向 Azure Government Cloud 中的 Log Analytics 工作區回報，請從 [Azure 雲端] 下拉式清單中選取 [Azure 美國政府]。  如果電腦需要透過 Proxy 伺服器與 Log Analytics 服務進行通訊，請選取 [進階]，然後提供 Proxy 伺服器的 URL 和連接埠號碼。
8. 完成提供必要的組態設定之後，選取 [下一步]。

  ![安裝代理程式][5]

9. 在 [安裝準備就緒] 頁面上，檢閱您的選擇，然後選取 [安裝]。
10. 在 [設定成功完成] 頁面上，選取 [完成]

完成時，[Microsoft 監視代理程式] 會出現在 [控制台] 中。 您可以在該處檢閱您的設定，並確認代理程式已連線。

如需安裝及設定代理程式的詳細資訊，請參閱[連線到 Windows 電腦](../log-analytics/log-analytics-agent-windows.md#install-the-agent-using-setup-wizard)。

您現在可於同一處監視您的 Azure VM 和非 Azure 電腦。 在 [計算] 下，您可以檢視所有 VM 和電腦的概觀及建議。 每一欄表示一組建議。 色彩代表 VM 或電腦目前對於該建議的安全性狀態。 資訊安全中心也會在安全性警示中顯示針對這些電腦所偵測到的任何項目。

  ![[計算] 刀鋒視窗][6]

[計算] 刀鋒視窗上顯示兩個類型的圖示：

![icon1](./media/quick-onboard-windows-computer/security-center-monitoring-icon1.png) 非 Azure 電腦

![icon2](./media/quick-onboard-windows-computer/security-center-monitoring-icon2.png) Azure VM

## <a name="clean-up-resources"></a>清除資源
不再需要時，您可以將代理程式從 Windows 電腦移除。

移除代理程式：

1. 開啟 [ **控制台**]。
2. 開啟 [程式和功能]。
3. 在 [程式和功能] 中，選取 [Microsoft Monitoring Agent]，然後按一下 [解除安裝]。

## <a name="next-steps"></a>後續步驟
在本快速入門中，您已在 Windows 電腦上佈建 Microsoft Monitoring Agent。 若要深入了解如何使用資訊安全中心，請繼續進行設定安全性原則及評估資源安全性的教學課程。

> [!div class="nextstepaction"]
> [教學課程：定義和評估安全性原則](tutorial-security-policy.md)

<!--Image references-->
[2]: ./media/quick-onboard-windows-computer/overview.png
[3]: ./media/quick-onboard-windows-computer/get-started.png
[4]: ./media/quick-onboard-windows-computer/add-computer.png
[5]: ./media/quick-onboard-windows-computer/log-analytics-mma-setup-laworkspace.png
[6]: ./media/quick-onboard-windows-computer/compute.png
