---
title: 要移轉 Microsoft Translator Hub 工作區和專案嗎？ - 自訂翻譯工具
titleSuffix: Azure Cognitive Services
description: 本文說明如何將您的中樞工作區和專案遷移至 Azure 認知服務自訂翻譯。
author: laujan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.date: 05/26/2020
ms.author: lajanuar
ms.topic: conceptual
ms.openlocfilehash: 89658ce88b4f1ac9d5bacac7bd45511b4aa0a1be
ms.sourcegitcommit: 100390fefd8f1c48173c51b71650c8ca1b26f711
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/27/2021
ms.locfileid: "98895703"
---
# <a name="migrate-hub-workspace-and-projects-to-custom-translator"></a>將中樞工作區和專案移轉至自訂翻譯工具

您可以輕鬆地將 [Microsoft Translator Hub](https://hub.microsofttranslator.com/) 工作區和專案移轉至自訂翻譯工具。 若要從 Microsoft Hub 起始移轉，只要選取工作區或專案，然後選取自訂翻譯工具中的工作區，再選取您要移轉的訓練即可。 移轉開始後，選取的訓練設定將會連同所有相關的文件一起移轉。 已部署的模型已定型，並且可在完成後自動部署。

在移轉期間會執行下列動作：
* 所有檔和專案定義都會傳送其名稱，並在名稱前面加上 "hub_"。 自動產生的測試和調整資料將會命名為 hub_systemtune_\<modelid> 或 hub_systemtest_\<modelid>。
* 任何在移轉執行時處於已部署狀態的訓練，都將使用 Hub 訓練的文件自動定型。 此訓練不會對您的訂用帳戶產生費用。 如果在移轉時選取了自動部署，已定型的模型就會在完成後隨即部署。 這適用一般裝載費用。
* 任何已移轉、但未處於已部署狀態的訓練，都會進入已移轉的草稿狀態中。 在此狀態下，您可以選擇使用已移轉的定義為模型定型，但需支付一般訓練費用。
* 在任何時間點，從中樞定型遷移的 BLEU 分數都可在模型的 [TrainingDetails] 頁面中找到，其位於「MT 中樞的 Bleu 分數」標題中。

> [!Note] 
> 若要成功定型，自訂翻譯至少需要10000個唯一的解壓縮句子。 自訂翻譯無法進行比 [建議的最小值](./sentence-alignment.md#suggested-minimum-number-of-sentences)更少的訓練。

## <a name="find-custom-translator-workspace-id"></a>尋找自訂翻譯工具工作區識別碼

若要移轉 [Microsoft Translator Hub](https://hub.microsofttranslator.com/) 工作區，您必須要有自訂翻譯工具中的目的地工作區識別碼。 自訂翻譯工具中的目的地工作區，是您所有的中樞工作區和專案所應移轉到的位置。

您可以在自訂翻譯工具的 [設定] 頁面上找到目的地工作區識別碼：

1. 移至自訂翻譯工具入口網站中的 [設定] 頁面。

2. 您可以在 [基本資訊] 區段中找到工作區識別碼。

    ![如何尋找目的地工作區識別碼](media/how-to/how-to-find-destination-ws-id.png)

3. 妥善保存您的目的地工作區識別碼，以便在移轉程序期間參考。

## <a name="migrate-a-project"></a>移轉專案

如果您想要選擇性地移轉專案，可以使用 Microsoft Translator Hub 所提供的功能。

若要移轉專案：

1. 登入 Microsoft Translator Hub。

2. 移至 [專案] 頁面。

3. 按一下適當專案的 [移轉] 連結。

    ![反白顯示所選取專案的 [遷移] 按鈕的螢幕擷取畫面。](media/how-to/how-to-migrate-from-hub.png)

4. 按下移轉連結後，您將會看到可讓您執行下列動作的表單：
   * 指定要移轉至自訂翻譯工具的工作區
   * 指出您是要移轉所有成功的訓練，或只是已部署的訓練。 依預設會移轉所有成功的訓練。
   * 指出您是否要讓訓練在訓練完成後自動部署。 根據預設，您的訓練並不會在完成時自動部署。

5. 按一下 [提交要求]。

## <a name="migrate-a-workspace"></a>移轉工作區

除了移轉單一專案以外，您也可以移轉工作區中所有包含成功訓練的專案。 這會使工作區中的每個專案都受到評估，就像已按下移轉連結一樣。 有許多專案的使用者若想要將其所有專案以相同的設定移轉至自訂翻譯工具，就適用這項功能。 工作區移轉可以從 Translator Hub 的 [設定] 頁面起始。

若要移轉工作區：

1. 登入 Microsoft Translator Hub。

2. 移至 [設定] 頁面。

3. 在 [設定] 頁面上，按一下 [將工作區資料移轉至自訂翻譯工具]。

    ![醒目顯示 [將工作區資料移轉至自訂翻譯] 選項的螢幕擷取畫面。](media/how-to/how-to-migrate-workspace-from-hub.png)

4. 在下一個頁面上，選取下列兩個選項之一：

    a. 僅限已部署的定型：若選取此選項，將只會移轉您已部署的系統和相關文件。

    b. 所有成功的定型：若選取此選項，將會移轉您所有成功的定型和相關文件。

    c. 輸入您在自訂翻譯工具中的目的地工作區識別碼。

    ![如何從中樞移轉](media/how-to/how-to-migrate-from-hub-screen.png)

5. 按一下 [提交要求]。

## <a name="migration-history"></a>移轉歷程記錄

當您從中樞要求工作區/專案遷移時，您會在自訂翻譯設定] 頁面中找到您的遷移歷程記錄。

若要檢視移轉歷程記錄，請遵循下列步驟：

1. 移至自訂翻譯工具入口網站中的 [設定] 頁面。

2. 在 [設定] 頁面的 [移轉歷程記錄] 區段中，按一下 [移轉歷程記錄]。

    ![移轉歷程記錄](media/how-to/how-to-migration-history.png)

[移轉歷程記錄] 頁面會針對您所要求的各個移轉顯示下列摘要資訊。

1. 移轉者：提交此移轉要求的使用者名稱及其電子郵件

2. 移轉日期：移轉的日期和時間戳記

3. 專案：要求移轉的專案數目與成功移轉的專案數目。

4. 定型：要求移轉的定型數目與成功移轉的定型數目。

5. 文件：要求移轉的文件數目與成功移轉的文件數目。

    ![移轉歷程記錄詳細資料](media/how-to/how-to-migration-history-details.png)

如果您希望專案、定型和文件有更詳細的相關移轉報告，您可以選擇將詳細資料匯出為 CSV。

## <a name="implementation-notes"></a>實作附註
* 自訂翻譯中尚未提供語言組的系統將只能透過自訂翻譯來存取資料或解除部署。 這些專案在 [專案] 頁面上會標示為 [無法使用]。 當我們啟用自訂翻譯的新語言組時，專案就會變成使用中以進行定型和部署。 
* 將專案從 Hub 移轉至自訂翻譯工具，並不會對您的 Hub 訓練或專案造成任何影響。 在移轉期間，我們不會從 Hub 中刪除專案或文件，也不會將模型解除部署。
* 您的每個專案只能移轉一次。 如果您需要重複移轉某個專案，請與我們連絡。
* 自訂翻譯支援與英文的 NMT 語言配對。 [查看支援語言的完整清單](../language-support.md#customization)。 Hub 不需要基準模型，因此可支援數千種語言。 您可以移轉不受支援的語言組，但我們只會執行文件和專案定義的移轉。 我們無法為新模型定型。 此外，這些文件和專案將會顯示為非作用中，以指出它們目前無法使用。 如果新增了這些專案和 (或) 文件的支援，它們就會變成作用中狀態，且可以定型。
* 自訂翻譯工具目前不支援單一語言定型資料。 和不受支援的語言組一樣，您可以移轉單一語言文件，但在單一語言資料受到支援之前，這些文件都會顯示為非作用中。
* 自訂翻譯工具需要 10k 個平行句子才能執行定型。 Microsoft Hub 無法對較小的資料集定型。 不符合此需求的訓練若進行移轉，則不會定型。

## <a name="custom-translator-versus-hub"></a>自訂翻譯工具與中樞

下表比較 Microsoft Translator Hub 與自訂翻譯工具之間的功能。

| 功能 | 集線器 | 自訂翻譯工具 |
| ------- | :-: | :---------------: |
| 自訂功能狀態    | 正式運作    | 正式運作 |
| 文字 API 版本    | V2     | V3  |
| SMT 自訂    | 是    | 否 |
| NMT 自訂    | 否    | 是 |
| 新的統一語音服務自訂    | 否    | 是 |
| 不追蹤 | 是 | 是 |

## <a name="new-languages"></a>新語言

如果您是在建立 Translator 的新語言系統時所使用的社區或組織，請洽詢以 [custommt@microsoft.com](mailto:custommt@microsoft.com) 取得詳細資訊。

## <a name="next-steps"></a>後續步驟

- [定型模型](how-to-train-model.md)。
- 透過 [Translator V3](../reference/v3-0-translate.md?tabs=curl)開始使用您已部署的自訂翻譯模型。