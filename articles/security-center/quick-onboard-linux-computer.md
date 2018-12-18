---
title: Azure 資訊安全中心快速入門 - 將您的 Linux 電腦上架到資訊安全中心 | Microsoft Docs
description: 本快速入門說明如何將您的 Linux 電腦上架到資訊安全中心。
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
ms.date: 09/02/2018
ms.author: rkarlin
ms.openlocfilehash: 1eab54d16dcc0facace9a51a4cf5b5d1243baf4f
ms.sourcegitcommit: d211f1d24c669b459a3910761b5cacb4b4f46ac9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/06/2018
ms.locfileid: "44025388"
---
# <a name="quickstart-onboard-linux-computers-to-azure-security-center"></a>快速入門：將 Linux 電腦上架到 Azure 資訊安全中心
將 Azure 訂用帳戶上架之後，您可以透過佈建 Linux 代理程式，為在 Azure 外 (例如，內部部署或其他雲端服務) 執行的 Linux 資源啟用資訊安全中心。

本快速入門說明如何在 Linux 電腦上安裝 Linux 代理程式。

## <a name="prerequisites"></a>必要條件
若要開始使用資訊安全中心，您必須有 Microsoft Azure 訂用帳戶。 如果您沒有訂用帳戶，可以註冊[免費帳戶](https://azure.microsoft.com/pricing/free-trial/)。

確認您是使用資訊安全中心的標準定價層之後，再開始按照本快速入門操作。 如需升級指示，請參閱[將 Azure 訂用帳戶上架到資訊安全中心標準定價層](security-center-get-started.md)。 您可以在前 60 天免費試用資訊安全中心標準的定價層。

## <a name="add-new-linux-computer"></a>新增 Linux 電腦

1. 登入 [Azure 入口網站](https://azure.microsoft.com/features/azure-portal/)。
2. 在 [Microsoft Azure] 功能表中，選取 [資訊安全中心]。 [資訊安全中心 - 概觀] 隨即開啟。

 ![資訊安全中心概觀][2]

3. 在 [資訊安全中心] 主功能表下，選取 [開始使用]。
4. 選取 [開始使用] 索引標籤。![開始使用][3]

5. 按一下 [新增非 Azure 電腦] 底下的 [設定]，隨即會顯示 Log Analytics 工作區清單。 清單中顯示啟用自動佈建之後，資訊安全中心為您建立的預設工作區 (如果適用)。 選取此工作區或其他您要使用的工作區。

    ![新增非 Azure 電腦](./media/quick-onboard-linux-computer/non-azure.png)

6.  在 [直接代理程式] 頁面的 [下載並啟動 Linux 代理程式] 下，選取 [複製] 按鈕以複製 *wget* 命令。

7.  開啟 [記事本] 並貼上此命令。 將此檔案儲存到可從您的 Linux 電腦存取的位置。

## <a name="install-the-agent"></a>安裝代理程式

1.  在您的 Linux 電腦上，開啟先前儲存的檔案。 選取整個內容，複製內容，然後開啟終端機主控台並貼上命令。
2.  安裝完成之後，您可以執行 *pgrep* 來驗證是否已安裝 *omsagent*。 該命令會傳回如下的 *omsagent* PID (處理序識別碼)：

  ![安裝代理程式][5]

Linux 資訊安全中心代理程式的記錄檔可以在下列位置找到：*/var/opt/microsoft/omsagent/<workspace id>/log/*

  ![代理程式記錄檔][6]

有時候可能需要最多 30 分鐘，Linux 電腦才會在資訊安全中心中出現。

您現在可於同一處監視您的 Azure VM 和非 Azure 電腦。 在 [計算] 下，您可以檢視所有 VM 和電腦的概觀及建議。 每一欄表示一組建議。 色彩代表 VM 或電腦目前對於該建議的安全性狀態。 資訊安全中心也會在安全性警示中顯示針對這些電腦所偵測到的任何項目。

  ![[計算] 刀鋒視窗][7]：[計算] 刀鋒視窗上顯示兩個類型的圖示：

  ![icon1](./media/quick-onboard-linux-computer/security-center-monitoring-icon1.png) 非 Azure 電腦

  ![icon2](./media/quick-onboard-linux-computer/security-center-monitoring-icon2.png) Azure VM

## <a name="clean-up-resources"></a>清除資源
不再需要時，您可以將代理程式從 Linux 電腦移除。

移除代理程式：

1. 將 Linux 代理程式[通用指令碼](https://github.com/Microsoft/OMS-Agent-for-Linux/releases)下載到電腦。
2. 在電腦上搭配 *--purge* 引數執行 .sh 檔案，如此可將代理程式及其設定完全移除。

    `sudo sh ./omsagent-<version>.universal.x64.sh --purge`

## <a name="next-steps"></a>後續步驟
在本快速入門中，您已在 Linux 電腦上佈建代理程式。 若要深入了解如何使用資訊安全中心，請繼續進行設定安全性原則及評估資源安全性的教學課程。

> [!div class="nextstepaction"]
> [教學課程：定義和評估安全性原則](tutorial-security-policy.md)

<!--Image references-->
[1]: ./media/quick-onboard-linux-computer/portal.png
[2]: ./media/quick-onboard-linux-computer/overview.png
[3]: ./media/quick-onboard-linux-computer/get-started.png
[4]: ./media/quick-onboard-linux-computer/add-computer.png
[5]: ./media/quick-onboard-linux-computer/pgrep-command.png
[6]: ./media/quick-onboard-linux-computer/logs-for-agent.png
[7]: ./media/quick-onboard-linux-computer/compute.png
