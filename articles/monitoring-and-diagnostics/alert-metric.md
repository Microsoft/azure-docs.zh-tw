---
title: 使用 Azure 監視器建立、檢視及管理計量警示
description: 了解如何使用 Azure 入口網站或 CLI 來建立、檢視及管理計量警示規則。
author: snehithm
ms.service: azure-monitor
ms.topic: conceptual
ms.date: 09/18/2018
ms.author: snmuvva
ms.component: alerts
ms.openlocfilehash: 6ac31818a68aa2b4624410b423b7559b84275777
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47409366"
---
# <a name="create-view-and-manage-metric-alerts-using-azure-monitor"></a>使用 Azure 監視器建立、檢視及管理計量警示

想在其中一個計量超過閾值時收到通知，可以利用 Azure 監視器中的計量警示。 計量警示可在一系列的多維度平台計量、自訂計量、Application Insights 標準和自訂計量上運作。 在此文章中，我們將會說明如何透過 Azure 入口網站與 Azure CLI 建立、檢視及管理計量警示規則。 您也可以使用[另一篇文章](monitoring-enable-alerts-using-template.md)中所述的 Azure Resource Manager 範本，來建立計量警示規則。

您可以從[計量警示概觀](alert-metric-overview.md)深入了解計量警示工作的運作方式。

## <a name="create-with-azure-portal"></a>使用 Azure 入口網站建立

下列程序說明如何在 Azure 入口網站建立計量警示規則：

1. 在 [Azure 入口網站](https://portal.azure.com)中，按一下 [監視]。 [監視] 刀鋒視窗會將所有監視設定與資料合併在一個檢視中。

2. 按一下 [警示]，然後按一下 [+ 新增警示規則]。

    > [!TIP]
    > 大部分的資源刀鋒視窗也都有 [警示] (位於 [監視] 下的資源功能表)，您也可以從該處建立警示。

3. 在載入的內容窗格中，按一下 [選取目標]，選取您要修改的目標資源。 使用 [訂用帳戶] 與 [資源類型] 下拉式清單尋找您要監視的資源。 您也可以使用搜尋列來尋找您的資源。

4. 如果選取的資源有可對其建立警示的計量，位於右下方的 [可用訊號] 會包含計量。 您可以檢視此[文章](monitoring-near-real-time-metric-alerts.md#metrics-and-dimensions-supported)中的計量警示支援的完整資源類型清單

5. 選取目標資源之後，請按一下 [新增準則]

6. 您會看到該資源支援的訊號清單，請選取您要對其建立警示的計量。

7. 您會看到過去 6 小時的計量圖表。 定義 [期間]、[頻率]、[運算子] 與 [閾值]，這可決定計量警示規則的評估邏輯。

8. 使用計量圖表可以判斷哪些是合理的閾值。

9. (選擇性) 如果計量具有維度，您將會看到顯示維度資料表。 為每個維度選取一或多個值。 計量警示會對所選值的所有組合評估其條件。 [深入了解對多維度計量發出警示的運作方式](alert-metric-overview.md)。 **Select \*** 也可以讓您選取任何維度。 **Select \*** 會將選擇動態擴充至維度的所有目前與未來值。

10. 按一下 [完成]

11. (選擇性) 如果您想要監視複雜的警示規則，請新增另一個準則

12. 填寫 [警示詳細資料]，例如 [警示規則名稱]、[描述] 與 [嚴重性]

13. 透過選取現有動作群組或建立新的動作群組，將動作群組新增至警示。

14. 按一下 [完成] 以儲存計量警示規則。

> [!NOTE]
> 透過入口網站建立的計量警示規則，會在與目標資源相同的資源群組中建立。

## <a name="view-and-manage-with-azure-portal"></a>使用 Azure 入口網站檢視及管理

您可以使用 [警示] 下的 [管理規則] 刀鋒視窗來檢視及管理計量警示規則。 下面的程序示範如何檢視您的計量警示規則及進行編輯。

1. 在 Azure 入口網站中，瀏覽到 [監視]

2. 按一下 [警示] 與 [管理規則]

3. 在 [管理規則] 刀鋒視窗中，您可以跨訂用帳戶檢視您的所有警示規則。 您可以使用 [資源群組]、[資源類型] 與 [資源] 來進一步篩選規則。 如果您只想要查看計量警示，請選取 [訊號類型] 作為計量。

    > [!TIP]
    > 在 [管理規則] 刀鋒視窗中，您可以選取多個警示規則，並加以啟用/停用。 這在某些目標資源必須進行維護時可能有幫助

4. 按一下要編輯之計量警示規則的名稱

5. 在 [編輯規則] 中，按一下要編輯的 [警示準則]。 您可以視需要變更計量、閾值與其他欄位

    > [!NOTE]
    > 建立計量警示之後，就無法編輯 [目標資源] 與 [警示規則名稱]。

6. 按一下 [完成] 以儲存您的編輯。

## <a name="with-azure-cli"></a>透過 Azure CLI

先前幾節說明如何使用 Azure 入口網站建立、檢視及管理計量警示規則。 此節說明如何使用跨平台 [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/get-started-with-azure-cli?view=azure-cli-latest) 執行相同的作業。 開始使用 Azure CLI 最快的方式就是透過 [Azure Cloud Shell](https://docs.microsoft.com/en-us/azure/cloud-shell/overview?view=azure-cli-latest)。 針對此文章，我們將會使用 Cloud Shell。

1. 前往 Azure 入口網站，按一下 [Cloud Shell]。

2. 在出現提示時，您可以搭配 ``--help`` 選項來使用命令，以深入了解該命令與其使用方式。 例如，下列命令會向您顯示可用來建立、檢視及管理計量警示的命令清單

    ```azurecli
    az monitor metrics alert --help
    ```

3. 您可以建立簡單的計量警示規則，監視 VM 上的平均 CPU 百分比是否大於 70

    ```azurecli
    az monitor metrics alert create -n {nameofthealert} -g {ResourceGroup} --scopes {VirtualMachineResourceID} --condition "avg Percentage CPU > 90"
    ```

4. 您可以使用下列命令來檢視資源群組中的所有計量警示

    ```azurecli
    az monitor metrics alert list  -g {ResourceGroup}
    ```

5. 您可以使用規則的名稱或資源識別碼，來查看特定計量警示規則的詳細資料。

    ```azurecli
    az monitor metrics alert show -g {ResourceGroup} -n {AlertRuleName}
    ```

    ```azurecli
    az monitor metrics alert show --ids {RuleResourceId}
    ```

6. 您可以使用下列命令來停用計量警示規則。

    ```azurecli
    az monitor metrics alert update -g {ResourceGroup} -n {AlertRuleName} -enabled false
    ```

7. 您可以使用下列命令來刪除計量警示規則。

    ```azurecli
    az monitor metrics alert update -g {ResourceGroup} -n {AlertRuleName} -enabled false
    ```

## <a name="next-steps"></a>後續步驟

- [使用 Azure Resource Manager 範本建立計量警示](monitoring-enable-alerts-using-template.md)。
- [了解計量警示的運作方式](alert-metric-overview.md)。
- [了解計量警示的 Webhook 結構描述](monitoring-near-real-time-metric-alerts.md#payload-schema)
