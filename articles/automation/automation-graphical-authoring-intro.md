---
title: Azure 自動化中的圖形化編寫
description: 圖形化編寫可讓您建立 Azure 自動化的 Runbook，而不使用程式碼。 本文章提供圖形化編寫的簡介和開始建立圖形化 Runbook 所需的所有詳細資料。
services: automation
ms.service: automation
ms.component: process-automation
author: georgewallace
ms.author: gwallace
ms.date: 03/16/2018
ms.topic: conceptual
manager: carmonm
ms.openlocfilehash: afc1ec8e171bc602f2698b4a36f249bc454cbed9
ms.sourcegitcommit: 744747d828e1ab937b0d6df358127fcf6965f8c8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/16/2018
ms.locfileid: "42144228"
---
# <a name="graphical-authoring-in-azure-automation"></a>Azure 自動化中的圖形化編寫

圖形化編寫可讓您為 Azure 自動化建立 Runbook，而沒有基礎 Windows PowerShell 或 PowerShell 工作流程程式碼的複雜度。 您可以從 Cmdlet 和 Runbook 的程式庫中將活動加入至畫布，並將其連結在一起並加以設定來形成工作流程。 如果您曾經用過 System Center Orchestrator 或 Service Management Automation (SMA)，則這應該看起來很熟悉

本文章提供圖形化編寫的簡介及開始建立圖形化 Runbook 所需的概念。

## <a name="graphical-runbooks"></a>圖形化 Runbook

Azure 自動化中的所有 Runbook 都是 Windows PowerShell 工作流程。 圖形化 Runbook 和圖形化 PowerShell 工作流程 Runbook 會產生由自動化背景工作執行的 PowerShell 程式碼，但是您無法檢視它或直接修改它。 圖形化 Runbook 可以轉換為圖形化 PowerShell 工作流程 Runbook，反之亦然，但它們無法轉換為文字式 Runbook。 現有的文字式 Runbook 無法匯入圖形化編輯器。

## <a name="overview-of-graphical-editor"></a>圖形化編輯器概觀

您可以透過建立或編輯圖形化 Runbook，在 Azure 入口網站中開啟圖形化編輯器。

![圖形化工作區](media/automation-graphical-authoring-intro/runbook-graphical-editor.png)

下列各節說明圖形化編輯器中的控制項。

### <a name="canvas"></a>畫布

畫布是您設計 Runbook 的位置。 您會從程式庫控制項中的節點將活動加入至 Runbook，並以連結將它們連接來定義 Runbook 的邏輯。

您可以使用畫布底部的控制項來放大或縮小。

### <a name="library-control"></a>程式庫控制項

程式庫控制項是您選取 [活動](#activities) 以加入至您的 Runbook 的位置。 您會將它們加入畫布，在畫布中將它們連接到其他活動。 它包含下表所述的四個區段：

| 區段 | 說明 |
|:--- |:--- |
| Cmdlet |包含可以在 Runbook 中使用的所有 Cmdlet。 Cmdlet 是依模組組織。 已在您自動化帳戶中安裝的所有模組皆可供使用。 |
| Runbook |包含自動化帳戶中的 Runbook。 這些 Runbook 可以加入至畫布以做為子 Runbook。 只會顯示與所編輯之 Runbook 相同核心類型的 Runbook。針對圖形化 Runbook，只會顯示以 PowerShell 為基礎的 Runbook，而針對圖形化 PowerShell 工作流程 Runbook，只會顯示以 PowerShell 工作流程為基礎的 Runbook。 |
| Assets |包含您的自動化帳戶中可以在 Runbook 中使用的 [自動化資產](http://msdn.microsoft.com/library/dn939988.aspx) 。 當您將資產加入 Runbook，它會新增工作流程活動，取得所選資產。 如果是變數資產，您可以選取是否要加入活動以取得變數或設定變數。 |
| Runbook 控制項 |包含可在目前的 Runbook 中使用的 Runbook 控制項活動。 「接合」  會接受多個輸入，並等待所有項目完成，然後再繼續工作流程。 「程式碼」  活動會根據圖形化 Runbook 類型而定，執行一或多行 PowerShell 或 PowerShell 工作流程程式碼。 您可以對很難利用其他活動來達成的自訂程式碼或功能使用此活動。 |

### <a name="configuration-control"></a>組態控制項

您可以在組態控制項中，針對畫布上所選取的物件提供詳細資料。 此控制項中的可用屬性取決於所選物件的類型。 在設定控制項中選取選項時，即會開啟其他刀鋒視窗以提供其他資訊。

### <a name="test-control"></a>測試控制項

第一次啟動圖形化編輯器時，不會顯示測試控制項。 當您以互動方式 [測試圖形化 Runbook](#graphical-runbook-procedures)時會開啟。

## <a name="graphical-runbook-procedures"></a>圖形化 Runbook 程序

### <a name="exporting-and-importing-a-graphical-runbook"></a>匯出和匯入圖形化 Runbook

您可以只匯出圖形化 Runbook 的已發行版本。 如果尚未發佈 Runbook，則 [匯出] 按鈕會停用。 當您按一下 [匯出] 按鈕，Runbook 就會下載到本機電腦。 檔案名稱須符合帶有 *graphrunbook* 副檔名的 Runbook 名稱。

您可以在加入 Runbook 時選取 [匯入] 選項，藉以匯入圖形化或圖形化 PowerShell 工作流程 Runbook 檔案。 當您選取要匯入的檔案時，您可以保留同一個**名稱**，或提供新名稱。 [Runbook 類型] 欄位將會在評估所選取的檔案類型之後顯示 Runbook 的類型，而且如果您嘗試選取其他不正確的類型，即會顯示訊息，表示可能發生衝突，而且在轉換期間，可能會發生語法錯誤。

![匯入 Runbook](media/automation-graphical-authoring-intro/runbook-import-revised20165.png)

### <a name="testing-a-graphical-runbook"></a>測試圖形化 Runbook

您可以在 Azure 入口網站中測試 Runbook 的草稿版本，同時讓 Runbook 的已發佈版本保持不變，或是在發佈全新的 Runbook 之前，先進行測試。 這可讓您驗證取代已發行的版本之前，Runbook 正常運作。 當您測試 Runbook 時，草稿 Runbook 會執行，而且它執行的任何動作都會完成。 不會建立工作歷程記錄，但是會在 [測試輸出] 窗格中顯示輸出。

開啟 Runbook 的測試控制項，方法是開啟 Runbook 進行編輯，然後按一下 [ **測試窗格** ] 按鈕。

測試控制項會提示您輸入任何輸入參數，而您可以按一下 [開始] 按鈕來啟動 Runbook。

### <a name="publishing-a-graphical-runbook"></a>發行圖形化 Runbook

Azure 自動化中的每個 Runbook 有草稿和已發行的版本。 只可執行已發行版本，而且只可編輯草稿版本。 已發行版本不會受到草稿版本的任何變更影響。 草稿版本就緒可供使用時，您將它發行，則草稿版本會覆寫已發行版本。

您可以開啟 Runbook 進行編輯，然後按一下 [ **發行** ] 按鈕來發行圖形化 Runbook。

尚未發行 Runbook 時，它的狀態為 **新增**。 發行時，它的狀態為 **已發行**。 如果在發行 Runbook 之後編輯 Runbook，且草稿和已發行版本不同，Runbook 的狀態會是 **編輯中**。

![Runbook 狀態](media/automation-graphical-authoring-intro/runbook-statuses-revised20165.png)

您也可以選擇還原成 Runbook 的已發行版本。 這樣會棄置上次發行 Runbook 後所進行的任何變更，並以已發行版本取代 Runbook 的草稿版本。

## <a name="activities"></a>活動

活動是 Runbook 的建置區塊。 活動可以是 PowerShell Cmdlet、子 Runbook 或工作流程活動。 以滑鼠右鍵按一下程式庫控制項的活動，然後選取 [加入至畫布]，即可將活動加入 Runbook。 然後可以按一下並拖曳活動，將它放置在畫布上的任何位置。 活動在畫布上的位置不會以任何方式影響 Runbook 的作業。 您可以配置您的 Runbook，不過您會發現以視覺化方式檢視其作業最適合。

![加入至畫布](media/automation-graphical-authoring-intro/add-to-canvas-revised20165.png)

在組態分頁中的畫布上選取活動以設定其屬性和參數。 您可以將活動的 [ **標籤** ] 變更成對您具意義的內容。 原始的 Cmdlet 仍在執行，您只是變更在圖形化編輯器中使用的顯示名稱。 標籤在 Runbook 內必須是唯一的。

### <a name="parameter-sets"></a>參數集

參數集會定義接受特定 Cmdlet 值的強制參數和選用參數。 所有的 Cmdlet 至少有一個參數集，而某些則有多個。 如果 Cmdlet 有多個參數集，則您必須選取要使用的參數集，才能設定參數。 您可以設定的參數取決於您選擇的參數集。 您可以變更活動使用的參數集，方法是選取 [ **參數集** ] 並選取其他設定。 在此情況下，您設定的任何參數值都會遺失。

在下列範例中，Get-AzureRmVM Cmdlet 有三個參數集。 您無法設定參數值，直到您選取其中一個參數集。 ListVirtualMachineInResourceGroupParamSet 參數集適用於傳回資源群組中的所有虛擬機器，並且具有單一選用參數。 **GetVirtualMachineInResourceGroupParamSet** 適用於指定您想要傳回的虛擬機器，而且具有兩個強制參數和一個選用參數。

![參數集](media/automation-graphical-authoring-intro/get-azurermvm-parameter-sets.png)

#### <a name="parameter-values"></a>參數值

在指定參數的值時，您可以選取資料來源以判斷如何指定值。 特定參數可使用的資料來源取決於該參數的有效值。 例如，對於不允許 Null 值的參數而言，Null 並非可用選項。

| 資料來源 | 說明 |
|:--- |:--- |
| 常數值 |輸入參數的值。 這只適用於下列資料類型：Int32、Int64、String、Boolean、DateTime、Switch。 |
| 活動輸出 |從優先於工作流程中的目前活動的活動輸出。 列出所有有效的活動。 對參數值只選取要使用它的輸出的活動。 如果活動會輸出具有多個屬性的物件，您可以在選取活動之後輸入名稱屬性。 |
| Runbook 輸入 |選取 Runbook 的輸入參數做為活動參數的輸入。 |
| 變數資產 |選取「自動化變數」做為輸入。 |
| 認證資產 |選取「自動化認證」做為輸入。 |
| 憑證資產 |選取「自動化憑證」做為輸入。 |
| 連線資產 |選取「自動化連線」做為輸入。 |
| PowerShell 運算式 |指定簡單 [PowerShell 運算式](#powershell-expressions)。 在活動和用於參數值的結果之前，會評估運算式。 您可以使用變數來參照活動或 Runbook 的輸入參數的輸出。 |
| 未設定 |清除先前設定的任何值。 |

#### <a name="optional-additional-parameters"></a>選擇性的其他參數

所有 Cmdlet 可選擇提供額外的參數。 這些是 PowerShell 一般參數或其他自訂參數。 您會看到一個文字方塊，您可以在其中使用 PowerShell 語法提供參數。 例如，若要使用 **Verbose** 一般參數，您會指定 **"-Verbose:$True"**。

### <a name="retry-activity"></a>重試活動

**重試行為** 可讓活動執行多次，直到符合特定條件為止 (與迴圈非常類似)。 您可以對應該執行多次的活動、容易出錯且可能需要嘗試一次以上才會成功的活動，或者針對有效資料測試活動輸出資訊的活動，來使用這項功能。

當您對活動啟用重試時，您可以設定延遲和條件。 延遲是 Runbook 再次執行活動之前所等待的時間 (以秒或分鐘計算)。 如果未指定延遲，則活動會在完成之後立即再次執行。

![活動重試延遲](media/automation-graphical-authoring-intro/retry-delay.png)

重試條件是 PowerShell 運算式，在每次活動執行之後評估。 如果運算式解析為 True，則活動會再次執行。 如果運算式解析為 False，則活動不會再次執行，且 Runbook 會移至下一個活動。

![活動重試延遲](media/automation-graphical-authoring-intro/retry-condition.png)

重試條件可以使用名為 $RetryData 的變數，提供活動重試相關資訊的存取權。 此變數具有下表中的屬性：

| 屬性 | 說明 |
|:--- |:--- |
| NumberOfAttempts |活動已執行的次數。 |
| 輸出 |活動上次執行的輸出。 |
| TotalDuration |活動第一次開始之後的經過時間。 |
| StartedAt |活動第一次開始的時間 (UTC 格式)。 |

以下是活動重試條件的範例。

```powershell-interactive
# Run the activity exactly 10 times.
$RetryData.NumberOfAttempts -ge 10
```

```powershell-interactive
# Run the activity repeatedly until it produces any output.
$RetryData.Output.Count -ge 1
```

```powershell-interactive
# Run the activity repeatedly until 2 minutes has elapsed.
$RetryData.TotalDuration.TotalMinutes -ge 2
```

設定活動的重試條件之後，該活動便會包含兩個視覺提示來提醒您。 一個顯示於活動中，另一個則會在您檢閱活動的組態時顯示。

![活動重試視覺指示器](media/automation-graphical-authoring-intro/runbook-activity-retry-visual-cue.png)

### <a name="workflow-script-control"></a>工作流程指令碼控制項

程式碼控制項是一種特殊活動，會根據所編寫的圖形化 Runbook 類型來接受 PowerShell 或 PowerShell 工作流程指令碼，以便提供可能無法使用的功能。 它不能接受參數，但它可以對活動輸出和 Runbook 輸入參數使用變數。 活動的任何輸出會加入至資料匯流排中，除非它在加入至 Runbook 的輸出中沒有連出的連結。

例如，下列程式碼會使用名為 $NumberOfDays 的 Runbook 輸入變數執行日期計算。 然後會將計算的日期時間傳送為輸出，供 Runbook 中後續的活動使用。

```powershell-interactive
$DateTimeNow = (Get-Date).ToUniversalTime()
$DateTimeStart = ($DateTimeNow).AddDays(-$NumberOfDays)}
$DateTimeStart
```

## <a name="links-and-workflow"></a>連結和工作流程

圖形化 Runbook 中的 **連結** 在連接兩個活動。 就會在畫布上顯示為箭號，從來源活動指向目的地活動。 活動會以箭頭的方向執行，在來源活動完成之後，目的地活動就會啟動。

### <a name="create-a-link"></a>建立連結

在兩個活動之間建立連結，方法是選取來源活動，然後按一下圖形底部的圓形。 將箭頭拖曳到目的地活動和版本。

![建立連結](media/automation-graphical-authoring-intro/create-link-revised20165.png)

在 [組態] 分頁中選取連結來設定其屬性。 這包括下表中所說明的連結類型：

| 連結類型 | 說明 |
|:--- |:--- |
| 管線 |目的地活動會對從來源活動輸出的每一個物件執行一次。 如果來源活動不會產生任何輸出，則不會執行目的地活動。 從來源活動輸出以物件形式提供。 |
| 順序 |目的地活動只會執行一次。 它會從來源活動接收物件的陣列。 從來源活動輸出以物件陣列形式提供。 |

### <a name="starting-activity"></a>啟動活動

圖形化 Runbook 會從沒有傳入連結的活動開始。 通常這只有一個活動，其作為 Runbook 的啟動活動。 如果多個活動皆沒有傳入連結，則 Runbook 會以平行方式執行它們來開始。 每個活動完成時，會遵循連結以執行其他活動。

### <a name="conditions"></a>條件

當您指定連結的條件時，只有條件解析為 true 時才會執行目的地活動。 您通常會在條件中使用 $ActivityOutput 變數，從來源活動擷取輸出

針對管線連結，您會指定單一物件的條件，並且來源活動會對每個物件輸出評估條件。 然後目的地活動會對符合條件的每個物件執行。 例如，透過 Get-AzureRmVm 的來源活動，可對條件式管線連結使用下列語法，僅擷取 *Group1*資源群組中的虛擬機器。

```powershell-interactive
$ActivityOutput['Get Azure VMs'].Name -match "Group1"
```

針對順序連結，因為單一陣列會傳回包含從來源活動輸出的所有物件，因此只會評估一次條件。 因此，順序連結不能用於篩選 (如管線連結)，但只能判斷下一個活動是否會執行。 例如，在我們的「啟動 VM」Runbook 中採用下列這組活動。

![具有順序的條件式連結](media/automation-graphical-authoring-intro/runbook-conditional-links-sequence.png)

有三個不同的序列連結，其會驗證提供給兩個 Runbook 輸入參數 (代表虛擬機器名稱和資源群組名稱) 的值，以判斷採取哪一個動作較適當 - 啟動單一虛擬機器、啟動資源群組中的所有虛擬機器，或訂用帳戶中的所有虛擬機器。 針對「連接到 Azure」和「取得單一 VM」之間的順序連結，以下是條件邏輯：

```powershell-interactive
<#
Both VMName and ResourceGroupName runbook input parameters have values
#>
(
(($VMName -ne $null) -and ($VMName.Length -gt 0))
) -and (
(($ResourceGroupName -ne $null) -and ($ResourceGroupName.Length -gt 0))
)
```

使用條件式連結時，可從來源活動提供該分支中的其他活動使用的資料將由條件篩選。 如果活動是多個連結的來源，則每個分支中活動可用的資料取決於連線到該分支的連結中的條件。

例如，下列 Runbook 中的 **Start-AzureRmVm** 活動會啟動所有虛擬機器。 它有兩個條件式連結。 第一個條件式連結會在 Start-AzureRmVm 活動順利完成時使用運算式 $ActivityOutput['Start-AzureRmVM'].IsSuccessStatusCode -eq $true  來篩選。 第二個會在 Start-AzureRmVm 活動無法啟動虛擬機器時使用運算式 $ActivityOutput['Start-AzureRmVM'].IsSuccessStatusCode -ne $true  來篩選。

![條件式連結範例](media/automation-graphical-authoring-intro/runbook-conditional-links.png)

遵循第一個連結並使用 Get-AzureVM 活動輸出的任何活動，將只會取得 Get-AzureVM 執行時啟動的虛擬機器。 遵循第二個連結的任何活動，只會取得 Get-AzureVM 執行時已停止的虛擬機器。 遵循第三個連結的任何活動，會取得所有虛擬機器，不論其執行狀態為何。

### <a name="junctions"></a>接合

接合是一個特殊的活動，會等候直到所有傳入的分支完成。 這可讓您以平行方式執行多個活動，並確保所有活動已完成再繼續。

雖然接合可以有無限的數量的連入連結，但這些連結只有一個可以是管線。 內送的順序連結數目不受限制。 您可以使用多個傳入的管線連結來建立接合，並儲存 Runbook，但是執行時將會失敗。

下列範例是 Runbook 的一個部分，其會啟動一組虛擬機器，同時下載修補程式以套用到這些機器。 接合用來確保兩個程序都已完成，才繼續 Runbook。

![接合](media/automation-graphical-authoring-intro/runbook-junction.png)

### <a name="cycles"></a>循環

循環是目的地活動連結回到其來源活動，或到最後連結回其來源的另一個活動。 圖形化編寫目前不允許循環。 如果您的 Runbook 有循環，它可以正確儲存，但是執行時會收到錯誤。

![循環](media/automation-graphical-authoring-intro/runbook-cycle.png)

### <a name="sharing-data-between-activities"></a>在活動之間共用資料

由連出連結活動輸出的任何資料會寫入Runbook 的 *資料匯流排* 。 Runbook 中的任何活動可以在資料匯流排上使用資料來填入參數值，或納入指令碼程式碼。 活動可以存取工作流程中任何先前的活動的輸出。

資料如何寫入至資料匯流排取決於活動上的連結類型。 針對 **管線**，資料會輸出為多個物件。 針對 **順序** 連結，資料會輸出為陣列。 如果只有一個值，則會輸出為單一元素陣列。

您可以使用兩種方法之一來存取資料匯流排上的資料。 第一個是使用 **活動輸出** 資料來源來填入另一個活動的參數。 如果輸出是物件，您可以指定單一屬性。

![活動輸出](media/automation-graphical-authoring-intro/activity-output-datasource-revised20165.png)

您也可以在 **PowerShell 運算式**資料來源中，或使用 ActivityOutput 變數從**工作流程指令碼**活動，擷取活動的輸出。 如果輸出是物件，您可以指定單一屬性。 ActivityOutput 變數使用下列語法。

```powershell-interactive
$ActivityOutput['Activity Label']
$ActivityOutput['Activity Label'].PropertyName
```

### <a name="checkpoints"></a>檢查點

您可以在圖形化 PowerShell 工作流程 Runbook 中設定[檢查點](automation-powershell-workflow.md#checkpoints)，方法是在任何活動上選取「檢查點 Runbook」。 這會導致在執行活動之後設定檢查點。

![檢查點](media/automation-graphical-authoring-intro/set-checkpoint.png)

檢查點只能在圖形化 PowerShell 工作流程 Runbook 中啟用，無法在圖形化 Runbook 中使用。 如果 Runbook 使用 Azure Cmdlet，當 Runbook 暫停並且在不同的背景工作從這個檢查點開始時，您應該使用 Connect-AzureRmAccount 遵循任何檢查點活動。

## <a name="authenticating-to-azure-resources"></a>向 Azure 資源驗證

管理 Azure 資源之 Azure 自動化中的 Runbook 需要向 Azure 驗證。 [執行身分帳戶](automation-create-runas-account.md) (也稱為服務主體) 是使用自動化 Runbook 存取您訂用帳戶中 Azure Resource Manager 資源的預設方法。 您可以將此功能加入至圖形化 Runbook，方法是將 **AzureRunAsConnection** 連線資產 (使用 PowerShell [Get-AutomationConnection](https://technet.microsoft.com/library/dn919922%28v=sc.16%29.aspx) Cmdlet) 和 [Connect-AzureRmAccount](/powershell/module/azurerm.profile/connect-azurermaccount) Cmdlet 加入至畫布。 下列範例會就此說明：

![執行身分驗證活動](media/automation-graphical-authoring-intro/authenticate-run-as-account.png)

「取得執行身分連線」活動 (亦即 Get-AutomationConnection) 是使用名為 AzureRunAsConnection 的常數值資料來源所設定。

![執行身分連線設定](media/automation-graphical-authoring-intro/authenticate-runas-parameterset.png)

下一個活動 (Connect-AzureRmAccount) 會加入已驗證的執行身分帳戶，以便在 Runbook 中使用。

![Connect-AzureRmAccount 參數集](media/automation-graphical-authoring-intro/authenticate-conn-to-azure-parameter-set.png)

> [!IMPORTANT]
> **Add-AzureRmAccount** 現在是 **Connect-AzureRMAccount** 的別名。 搜尋您的程式庫項目時，如果沒有看到 **Connect-AzureRMAccount**，便可以使用 **Add-AzureRmAccount**，或是在自動化帳戶中更新模組。

針對 **APPLICATIONID**、**CERTIFICATETHUMBPRINT** 和 **TENANTID** 等參數，您需要針對「欄位」路徑指定屬性名稱，因為活動會使用多個屬性來輸出物件。 否則當您執行 Runbook 時，會在嘗試進行驗證時失敗。 這就是您使用執行身分帳戶驗證 Runbook 時所需的最低限度。

若要針對使用 [Azure AD 使用者帳戶](automation-create-aduser-account.md)建立自動化帳戶來管理 Azure 傳統部署或 Azure Resource Manager 資源的訂戶維持回溯相容性，用來驗證的方法是使用 Add-AzureAccount Cmdlet 搭配[認證資產](automation-credentials.md)，其代表可存取 Azure 帳戶的 Active Directory 使用者。

您可以在 Add-AzureAccount 活動之後，將認證資產加入至畫布，以將這項功能加入至圖形化 Runbook。 Add-AzureAccount 會對其輸入使用認證活動。 下列範例會就此說明：

![驗證活動](media/automation-graphical-authoring-intro/authentication-activities.png)

您必須在開始 Runbook 時，和每個檢查點之後進行驗證。 這表示在任何 Checkpoint-Workflow 活動之後加入額外的 Add-AzureAccount 活動。 因為您可以使用相同認證，因此不需要額外的認證活動

![活動輸出](media/automation-graphical-authoring-intro/authentication-activity-output.png)

## <a name="runbook-input-and-output"></a>Runbook 輸入和輸出

### <a name="runbook-input"></a>Runbook 輸入

Runbook 可能需要來自使用者的輸入內容 (透過 Azure 入口網站啟動 Runbook 時進行的輸入)；如果目前的 Runbook 為子系，則需要來自另一個 Runbook 的輸入內容。
例如，如果您有一個會建立虛擬機器的 Runbook，您可能需要在每次啟動 Runbook 時提供資訊，例如虛擬機器的名稱和其他屬性。

您透過定義一或多個輸入參數來接受 Runbook 的輸入。 您會在每次啟動 Runbook 時提供值給這些參數。 當您透過 Azure 入口網站啟動 Runbook 時，它會提示您為 Runbook 的每個輸入參數提供值。

您可以按一下 Runbook 工具列上的 [ **輸入和輸出** ] 按鈕來存取 Runbook 的輸入參數。

這會開啟 [輸入和輸出] 控制項，您可以在其中編輯現有的輸入參數，或按一下 [加入輸入] 來建立一個新的。

![加入輸入](media/automation-graphical-authoring-intro/runbook-edit-add-input.png)

下表中的屬性定義每個輸入參數：

| 屬性 | 說明 |
|:--- |:--- |
| 名稱 |參數的唯一名稱。 這只能包含數字字元，而且不能包含空格。 |
| 說明 |輸入參數的選擇性描述。 |
| 類型 |對參數值預期的資料型別。 提示您輸入時，Azure 入口網站會對每個參數提供適當的資料類型控制項。 |
| 強制 |指定是否必須提供參數的值。 如果您未對未定義預設值的每個強制參數提供值，則無法啟動 Runbook。 |
| 預設值 |如果未提供其中一個的值，要對參數指定什麼值。 這可以是 Null 或特定值。 |

### <a name="runbook-output"></a>Runbook 輸出

沒有傳出連結的任何活動所建立的資料會加入 [Runbook 的輸出](https://docs.microsoft.com/azure/automation/automation-runbook-output-and-messages)。 輸出會隨著 Runbook 工作儲存，並且在 Runbook 用作子項時提供給父 Runbook 使用。

## <a name="powershell-expressions"></a>PowerShell 運算式

圖形化撰寫的優點之一是提供您以 PowerShell 的基本知識建立 Runbook 的能力。 目前，您還是需要稍微了解 PowerShell，以填入某些[參數值](#activities)和設定[連結條件](#links-and-workflow)。 本節提供 PowerShell 運算式的快速簡介，供不熟悉的使用者參考。 PowerShell 的完整詳細資料位於 [使用 Windows PowerShell 撰寫指令碼](http://technet.microsoft.com/library/bb978526.aspx)。

### <a name="powershell-expression-data-source"></a>PowerShell 運算式資料來源
您可以使用 PowerShell 運算式做為資料來源，使用一些 PowerShell 程式碼的結果來填入 [活動參數](#activities) 的值。 這可以是執行一些簡單函式的單行程式碼，或執行一些複雜邏輯的多行程式碼。 未指派給變數的任何命令輸出都會輸出到參數值。

例如，下列命令會輸出目前的日期。

例如，下列命令會輸出目前的日期。

```powershell-interactive
Get-Date
```

下列命令會從目前的日期建立字串並將它指派給變數。 然後將變數的內容傳送至輸出

```powershell-interactive
$string = "The current date is " + (Get-Date)
$string
```

下列命令會評估目前的日期並傳回表示當天是工作日或週末的字串。

```powershell-interactive
$date = Get-Date
if (($date.DayOfWeek = "Saturday") -or ($date.DayOfWeek = "Sunday")) { "Weekend" }
else { "Weekday" }
```

### <a name="activity-output"></a>活動輸出

若要在 Runbook 中使用上一個活動的輸出，請以下列語法使用 $ActivityOutput 變數。

```powershell-interactive
$ActivityOutput['Activity Label'].PropertyName
```

例如，您可能有一個活動，其具有需要虛擬機器名稱的屬性，在此情況下您可以使用下列運算式：

```powershell-interactive
$ActivityOutput['Get-AzureVm'].Name
```

如果是需要虛擬機器物件的屬性而非只是單純的屬性，則您要使用下列語法傳回整個物件。

```powershell-interactive
$ActivityOutput['Get-AzureVm']
```

您也可以在更複雜的運算式中使用活動的輸出，例如串連文字到虛擬機器名稱的下列運算式。

```powershell-interactive
"The computer name is " + $ActivityOutput['Get-AzureVm'].Name
```

### <a name="conditions"></a>條件

使用 [比較運算子](https://technet.microsoft.com/library/hh847759.aspx) 來比較值或判斷值是否符合指定的模式。 比較會傳回 $true 或 $false 的值。

例如，下列條件會判斷來自 *Get-AzureVM* 活動的虛擬機器目前是否「已停止」。

```powershell-interactive
$ActivityOutput["Get-AzureVM"].PowerState –eq "Stopped"
```

下列條件會檢查相同的虛擬機器是否處於「已停止」 以外的任何狀態。

```powershell-interactive
$ActivityOutput["Get-AzureVM"].PowerState –ne "Stopped"
```

您可以使用[邏輯運算子](https://technet.microsoft.com/library/hh847789.aspx) (例如 **-and** 或 **-or**) 加入多個條件。 例如，下列條件會檢查上述範例中相同虛擬機器的狀態是否為「已停止」或「正在停止」。

```powershell-interactive
($ActivityOutput["Get-AzureVM"].PowerState –eq "Stopped") -or ($ActivityOutput["Get-AzureVM"].PowerState –eq "Stopping")
```

### <a name="hashtables"></a>雜湊表

[雜湊表](http://technet.microsoft.com/library/hh847780.aspx) 是傳回一組值時很有用的名稱/值組。 某些活動的屬性可能是雜湊表而不是簡單值。 雜湊表也可能稱為字典。

使用下列語法建立雜湊表。 雜湊表可以包含任意數目的項目，但是每個項目都由一個名稱和值定義。

```powershell-interactive
@{ <name> = <value>; [<name> = <value> ] ...}
```

例如，下列運算式建立要在活動參數的資料來源中使用的雜湊表，這個雜湊表的值要做為網際網路搜尋。

```powershell-interactive
$query = "Azure Automation"
$count = 10
$h = @{'q'=$query; 'lr'='lang_ja';  'count'=$Count}
$h
```

下列範例使用稱為 *Get Twitter Connection* 的活動的輸出來填入雜湊表。

```powershell-interactive
@{'ApiKey'=$ActivityOutput['Get Twitter Connection'].ConsumerAPIKey;
    'ApiSecret'=$ActivityOutput['Get Twitter Connection'].ConsumerAPISecret;
    'AccessToken'=$ActivityOutput['Get Twitter Connection'].AccessToken;
    'AccessTokenSecret'=$ActivityOutput['Get Twitter Connection'].AccessTokenSecret}
```

## <a name="next-steps"></a>後續步驟

* 若要開始使用 PowerShell 工作流程 Runbook，請參閱 [我的第一個 PowerShell 工作流程 Runbook](automation-first-runbook-textual.md)
* 若要開始使用圖形化 Runbook，請參閱 [我的第一個圖形化 Runbook](automation-first-runbook-graphical.md)
* 若要深入了解 Runbook 類型、其優點和限制，請參閱 [Azure 自動化 Runbook 類型](automation-runbook-types.md)
* 若要了解如何使用自動化執行身分帳戶進行驗證，請參閱 [設定 Azure 執行身分帳戶](automation-sec-configure-azure-runas-account.md)