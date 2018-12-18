---
title: 在 Microsoft Flow 中使用 IoT Central 連接器建置工作流程 | Microsoft Docs
description: 在 Microsoft Flow 中使用 IoT Central 連接器觸發工作流程，以及在工作流程中建立、更新和刪除裝置。
services: iot-central
author: viv-liu
ms.author: viviali
ms.date: 09/18/2018
ms.topic: conceptual
ms.service: iot-central
manager: peterpr
ms.openlocfilehash: 7a52ee11e52367755d2a241a0bea2cfc053cff3d
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47039933"
---
# <a name="build-workflows-with-the-iot-central-connector-in-microsoft-flow"></a>在 Microsoft Flow 中使用 IoT Central 連接器建置工作流程

*此主題適用於建置人員和系統管理員。*

使用 Microsoft Flow，將商務使用者所依賴的多個應用程式和服務的工作流程自動化。 在 Microsoft Flow 中使用 IoT Central 連接器時，您可以在規則於 IoT Central 中被觸發時觸發工作流程。 在 IoT Central 或任何其他應用程式所觸發的工作流程中，您可以使用 IoT Central 連接器中的動作建立裝置、更新裝置的屬性和設定，或刪除裝置。 請了解可將 IoT Central 連線至其他服務 (例如行動裝置和 Microsoft Teams 等) 的[這些 Microsoft Flow 範本](https://aka.ms/iotcentralflowtemplates)。

> [!NOTE] 
> 您必須使用 Microsoft 的個人、公司或學校帳戶登入 Microsoft Flow。 在[此處](https://aka.ms/microsoftflowplans)深入了解 Microsoft Flow 方案。

## <a name="trigger-a-workflow-when-a-rule-is-triggered"></a>在觸發規則時觸發工作流程

本節說明如何在於 IoT Central 中觸發規則時，觸發 Flow 行動應用程式中的行動通知。

1. 首先，[在 IoT Central 中建立規則](howto-create-telemetry-rules.md)。 在儲存規則條件之後，請按一下 [Microsoft Flow 動作] 來作為新動作。 您的瀏覽器中應會開啟新的索引標籤或視窗，將您導向至 Microsoft Flow。

    ![建立新的 Microsoft Flow 動作](media/howto-add-microsoft-flow/createflowaction.PNG)

1. 登入 Microsoft Flow。 此帳戶不一定要與您在 IoT Central 中使用的帳戶相同。 您會進入 [概觀] 頁面，其中顯示連線至自訂動作的 IoT Central 連接器。

1. 登入 IoT Central 連接器，然後按一下 [繼續]。 您會前往 Microsoft Flow 設計工具，以建置工作流程。 工作流程具有已填入您的應用程式和規則的 IoT Central 觸發程序。

1. 選擇 [+ 新增步驟] 和 [新增動作]。 此時，您可以將任何您想要的動作新增至工作流程。 在範例中，我們將傳送行動通知。 請搜尋**通知**，然後選擇 [通知 - 傳送行動通知給我]。

1. 在動作中，將您的通知所要傳達的內容填入 [文字] 欄位中。 您可以納入 IoT Central 規則中的「動態內容」，將裝置名稱和時間戳記等重要資訊傳至您的通知。

    > [!NOTE]
    > 按一下 [動態內容] 視窗中的「更多資訊」文字，可取得觸發規則的測量和屬性值。

    ![動態窗格開啟時的流程編輯動作](./media/howto-add-microsoft-flow/flowdynamicpane.PNG)

1. 在您的動作編輯完成後，請按一下 [儲存]。 您將被導向至工作流程的概觀頁面。 在此處您可以看到執行歷程記錄，並將其與其他同事共用。

    > [!NOTE]
    > 如果您想讓 IoT Central 應用程式中的其他使用者編輯此規則，您必須在 Microsoft Flow 中與他們共用規則。 請在工作流程中新增他們的 Microsoft Flow 帳戶，作為擁有者。

1. 如果您回到 IoT Central 應用程式，您將會在 [動作] 區域下看到此規則具有 Microsoft Flow 動作。

您隨時都可在 Microsoft Flow 中使用 IoT Central 連接器開始建置工作流程。 然後，您可以選擇要連線至哪個 IoT Central 應用程式和規則。

## <a name="create-a-device-in-a-workflow"></a>在工作流程中建立裝置

本節說明如何使用 Microsoft Flow 行動應用程式，透過按下行動裝置的按鈕在 IoT Central 中建立新的裝置。 您可以在流程中使用此動作，而在某個裝置新增於他處時建立新的裝置，將 Dynamics 之類的 ERP 系統與 IoT Central 整合。

1. 首先，在 Microsoft Flow 中建立空的工作流程。

1. 搜尋**適用於行動裝置的 [流程] 按鈕**，作為觸發程序。

1. 在此觸發程序中，新增名為**裝置名稱**的文字輸入。 請將描述文字變更為**輸入新裝置的裝置名稱**。

1. 新增動作。 搜尋 [Azure IoT Central - 建立裝置] 動作。

1. 選擇您的應用程式，並從下拉式清單中選擇用來建立裝置的裝置範本。 您會看到展開以顯示所有裝置屬性和設定的動作。

1. 選取 [裝置名稱] 欄位。 在動態內容窗格中，選擇 [裝置名稱]。 此值會從使用者透過行動應用程式輸入的內容傳入，且將會是您的新裝置在 IoT Central 中的名稱。 在此範例中，唯一的必要欄位是裝置名稱，會以紅色星號指出。 另一個裝置範本可能會有多個必須填入才能建立新裝置的必要欄位。

    ![流程建立裝置動作動態窗格](./media/howto-add-microsoft-flow/flowcreatedevice.PNG)
1. (選擇性) 根據您建立新裝置時的需要，填寫其他欄位。

1. 最後，儲存您的工作流程。

1. 在 Microsoft Flow 行動應用程式中試用您的工作流程。 移至應用程式中的 [按鈕] 索引標籤。 您應該會看到 [按鈕] -> [建立新的裝置工作流程]。 請輸入新裝置的名稱，並確認它顯示於 IoT Central 中！

    ![流程建立裝置的行動裝置螢幕擷取畫面](./media/howto-add-microsoft-flow/flowmobilescreenshot.png)

## <a name="update-a-device-in-a-workflow"></a>更新工作流程中的裝置

本節說明如何使用 Microsoft Flow 行動應用程式，透過按下行動裝置的按鈕來更新 IoT Central 中的裝置設定和屬性。

1. 首先，在 Microsoft Flow 中建立空的工作流程。

1. 搜尋**適用於行動裝置的 [流程] 按鈕**，作為觸發程序。

1. 在此觸發程序中，新增如 [位置] 文字輸入之類、且與您要變更的設定或屬性相對應的輸入。 請變更描述文字。

1. 新增動作。 搜尋 [Azure IoT Central - 更新裝置] 動作。

1. 從下拉式清單中選擇您的應用程式。 現在您將需要所要更新之現有裝置的識別碼。 您可以在瀏覽器 URL 中取得 IoT Central 裝置的識別碼。

    ![IoT Central Device Explorer 裝置識別碼](./media/howto-add-microsoft-flow/iotcdeviceid.PNG)

1. 您可以更新裝置名稱。 若要更新裝置的任何屬性和設定，您必須在 [裝置範本] 下拉式清單中針對您要更新的裝置選取裝置範本。 動作圖格會展開以顯示您可以更新的所有屬性和設定。

1. 選取您要更新的每個屬性和設定。 在動態內容窗格中，從觸發程序中選擇對應的輸入。 在此範例中，[位置] 值會向下傳播，以更新裝置的 [位置] 屬性。

1. 最後，儲存您的工作流程。

1. 在 Microsoft Flow 行動應用程式中試用您的工作流程。 移至應用程式中的 [按鈕] 索引標籤。 您應該會看到 [按鈕] -> [更新裝置工作流程]。 輸入您的輸入內容，並查看裝置在 IoT Central 中更新的情形！

## <a name="delete-a-device-in-a-workflow"></a>刪除工作流程中的裝置

您可以使用 [Azure IoT Central - 刪除裝置] 動作，以依據裝置識別碼刪除裝置。 以下是在 Microsoft Flow 行動應用程式中按下按鈕以刪除裝置的範例工作流程。

   ![流程刪除裝置工作流程](./media/howto-add-microsoft-flow/flowdeletedevice.PNG)
    
## <a name="troubleshooting"></a>疑難排解

如果您無法建立對 Azure IoT Central 連接器的連線，以下提供一些秘訣供您參考。

1. 目前並不支援 Microsoft 個人帳戶 (例如 @hotmail.com、@live.com、@outlook.com 網域)。 您必須使用 AAD 公司或學校帳戶。

2. 若要在 Microsoft Flow 中使用 IoT Central 連接器，您必須至少已登入 IoT Central 應用程式一次。 否則，該應用程式不會出現在 [應用程式] 下拉式清單中。

3. 如果您在使用 AAD 帳戶時發生錯誤，請嘗試開啟 Windows PowerShell，並以系統管理員身分執行下列 commandlet。
    ``` PowerShell
    Install-Module AzureAD
    Connect-AzureAD
    New-AzureADServicePrincipal -AppId 9edfcdd9-0bc5-4bd4-b287-c3afc716aac7 -DisplayName "Azure IoT Central"
    ```
    
## <a name="next-steps"></a>後續步驟
您現在已了解如何使用 Microsoft Flow 建置工作流程，建議的後續步驟為[管理裝置](howto-manage-devices.md)。

