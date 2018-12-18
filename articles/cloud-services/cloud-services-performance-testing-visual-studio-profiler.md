---
title: 在本機的計算模擬器中分析雲端服務 | Microsoft Docs
services: cloud-services
description: 使用 Visual Studio 分析工具調查雲端服務中的效能問題
documentationcenter: ''
author: mikejo
manager: douge
editor: ''
tags: ''
ms.assetid: 25e40bf3-eea0-4b0b-9f4a-91ffe797f6c3
ms.service: cloud-services
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: multiple
ms.topic: article
ms.date: 11/18/2016
ms.author: mikejo
ms.openlocfilehash: ea46039583681bd89e254d153997e3a300041d4e
ms.sourcegitcommit: 4597964eba08b7e0584d2b275cc33a370c25e027
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/02/2018
ms.locfileid: "37341349"
---
# <a name="testing-the-performance-of-a-cloud-service-locally-in-the-azure-compute-emulator-using-the-visual-studio-profiler"></a>使用 Visual Studio 分析工具，在 Azure 計算模擬器中本機測試雲端服務的效能
各種工具和技術可用於測試雲端服務的效能。
當您將雲端服務發佈至 Azure 時，可以讓 Visual Studio 收集分析資料，然後在本機分析它 (如[分析雲端服務][11] 中所述)。
您也可以使用診斷來追蹤各種效能計數器 (如[在 Azure 中使用效能計數器][2]中所述)。
您也可能想要先在計算模擬器中本機分析應用程式，再將之部署至雲端。

本文涵蓋進行分析的「CPU 取樣」方法，這可以在模擬器上本機完成。 CPU 取樣不是非常侵入式的分析方法。 分析工具會按指定的取樣間隔取得呼叫堆疊的快照集。 會收集一段時間的資料，而且資料會顯示在報告中。 此分析方法傾向指出在計算密集應用程式中的哪個位置完成大部分的 CPU 工作。  這可讓您有機會聚焦在應用程式耗用最多時間的「最忙碌路徑」。

## <a name="1-configure-visual-studio-for-profiling"></a>1：設定 Visual Studio 進行分析
首先，有些 Visual Studio 組態選項可能在進行分析時很實用。 若要讓分析報告發揮作用，您需要應用程式的符號 (.pdb 檔案)，也需要系統庫的符號。 您會希望確定參考可用的符號伺服器。 若要這樣做，請在 Visual Studio 的 [工具] 功能表上，依序選擇 [選項]、[偵錯] 和 [符號]。 請確定 Microsoft Symbol Servers 列在 [符號檔 (.pdb) 位置] 下方。  您也可以參考 http://referencesource.microsoft.com/symbols，這裡可能有其他符號檔。

![符號選項][4]

如有需要，您可以設定 Just My Code 來簡化分析工具所產生的報告。 啟用 Just My Code 之後，會簡化函式呼叫堆疊，因此報告中會隱藏整個是程式庫和 .NET Framework 的內部呼叫。 在 [工具] 功能表上，選擇 [選項]。 然後展開 [效能工具] 節點，並選擇 [一般]。 選取 [啟用 Just My Code 以進行分析工具報告] 核取方塊。

![Just My Code 選項][17]

您可以將這些指示與現有專案或新的專案搭配使用。  如果您建立新的專案來嘗試上面所述的技術，請選擇 C# [Azure 雲端服務] 專案，然後選取 [Web 角色] 和 [背景工作角色]。

![Azure 雲端服務專案角色][5]

舉例來說，會在專案中新增某個程式碼，而這個程式碼耗用許多時間，並且示範某個明顯的效能問題。 例如，將下列程式碼新增至背景工作角色專案：

```csharp
public class Concatenator
{
    public static string Concatenate(int number)
    {
        int count;
        string s = "";
        for (count = 0; count < number; count++)
        {
            s += "\n" + count.ToString();
        }
        return s;
    }
}
```

請在背景工作角色的 RoleEntryPoint 衍生類別中，從 RunAsync 方法呼叫此程式碼。 (忽略有關同步執行方法的警告。)

```csharp
private async Task RunAsync(CancellationToken cancellationToken)
{
    // TODO: Replace the following with your own logic.
    while (!cancellationToken.IsCancellationRequested)
    {
        Trace.TraceInformation("Working");
        Concatenator.Concatenate(10000);
    }
}
```

在方案組態設定為 [發行] 的情況下，在本機建置和執行雲端服務，而不要進行偵錯 (Ctrl+F5)。 這確保建立在本機執行應用程式的所有檔案和資料夾，並確保已啟動所有模擬器。 從工作列啟動 Compute Emulator UI，以驗證您的背景工作角色執行中。

## <a name="2-attach-to-a-process"></a>2：連結至程序
您必須將分析工具連結至執行中程序，而不是從 Visual Studio 2010 IDE 啟動應用程式來分析應用程式。 

若要將分析工具連結至程序，請在 [分析] 功能表上選擇 [分析工具] 和 [連結/中斷連結]。

![附加設定檔選項][6]

若為背景工作角色，請尋找 WaWorkerHost.exe 程序。

![WaWorkerHost 程序][7]

如果您的專案資料夾位於網路磁碟機上，則分析工具會要求您提供另一個位置來儲存分析報告。

 您也可以藉由連結至 WaIISHost.exe 來連結至 Web 角色。
如果您的應用程式中有多個背景工作角色程序，則需使用 processID 進行區別。 您可以存取 Process 物件，以透過程式設計方式查詢 processID。 例如，如果您將此程式碼新增至角色中 RoleEntryPoint 衍生類別的 Run 方法，則可以查看計算模擬器 UI 中的記錄，得知要連線的程序。

```csharp
var process = System.Diagnostics.Process.GetCurrentProcess();
var message = String.Format("Process ID: {0}", process.Id);
Trace.WriteLine(message, "Information");
```

若要檢視記錄，請啟動計算模擬器 UI。

![啟動計算模擬器 UI][8]

按一下主控台視窗的標題列，以在計算模擬器 UI 中開啟背景工作角色記錄主控台視窗。 您可以在記錄中看到程序 ID。

![檢視處理序識別碼][9]

連結之後，請在應用程式 UI 中執行步驟 (需要時) 來重現案例。

當您想要停止分析時，請選擇 [停止分析] 連結。

![停止分析選項][10]

## <a name="3-view-performance-reports"></a>3：檢視效能報告
會顯示您應用程式的效能報告。

此時，分析工具會停止執行、以 .vsp 檔案儲存資料，並顯示可顯示此資料分析的報告。

![分析工具報告][11]

如果您在「最忙碌路徑」中看到 String.wstrcpy，請按一下 Just My Code 變更檢視，只顯示使用者程式碼。  如果您看到 String.Concat，請嘗試按 [顯示所有程式碼] 按鈕。

您應該會看到 Concatenate 方法和 String.Concat 耗用大部分的執行時間。

![報告的分析][12]

如果您已在本文中新增字串串連碼，則應該會在 [工作清單] 中看到警告。 您也可能會看到有大量記憶體回收的警告，原因是建立和處置的字串數目。

![效能警告][14]

## <a name="4-make-changes-and-compare-performance"></a>4：進行變更與比較效能
您也可以在變更程式碼之前和之後比較效能。  請停止執行中流程並編輯程式碼，以使用 StringBuilder 取代字串串連作業：

```csharp
public static string Concatenate(int number)
{
    int count;
    System.Text.StringBuilder builder = new System.Text.StringBuilder("");
    for (count = 0; count < number; count++)
    {
        builder.Append("\n" + count.ToString());
    }
    return builder.ToString();
}
```

請進行另一個效能執行，然後比較效能。 在 [效能總管] 中，如果這些執行都位於相同的工作階段中，則只能選取兩份報告，並開啟捷徑功能表，然後選擇 [比較效能報告]。 如果您想要與另一個效能工作階段中的執行進行比較，請開啟 [分析] 功能表，然後選擇 [比較效能報告]。 請在顯示的對話方塊中指定兩個檔案。

![比較效能報告選項][15]

報告會反白顯示兩個執行之間的差異。

![比較報告][16]

恭喜！ 您已開始使用分析工具。

## <a name="troubleshooting"></a>疑難排解
* 確定您正在分析發行組建並開始而不進行偵錯。
* 如果未在 [分析工具] 功能表上啟用 [連結/中斷連結] 選項，請執行 [效能精靈]。
* 使用計算模擬器 UI 檢視您應用程式的狀態。 
* 如果您在模擬器中啟動應用程式或是連結分析工具時發生問題，請關閉並重新啟動計算模擬器。 如果這樣未解決問題，請嘗試重新開機。 如果您使用計算模擬器來暫停和移除執行中部署，則可能會發生此問題。
* 如果您已從命令列使用任何分析命令 (尤其是全域設定)，請確定已呼叫 VSPerfClrEnv /globaloff，並已關閉 VsPerfMon.exe。
* 取樣時，如果您看到訊息 [PRF0025: No data was collected]，請確認您所連結的程序具有 CPU 活動。 未執行任何計算工作的應用程式可能不會產生任何取樣資料。  在執行任何取樣之前，也可能已結束程序。 請確認所分析角色的 Run 方法未終止。

## <a name="next-steps"></a>後續步驟
在 Visual Studio 分析工具中，不支援在模擬器中檢測 Azure 二進位，但是，如果您想要測試記憶體配置，則可以在分析時選擇該選項。 您也可以選擇並行分析來協助您判斷執行緒是否浪費時間來競爭鎖定，或選擇階層互動分析來協助您追蹤在應用程式階層之間互動時的效能問題 (最常發生在資料層與背景工作角色之間)。  您可以檢視應用程式所產生的資料庫查詢，以及使用分析資料來提高資料庫的使用。 如需階層互動分析的詳細資訊，請參閱部落格文章[逐步介紹：在 Visual Studio Team System 2010 中使用階層互動分析][3]。

[1]: https://docs.microsoft.com/azure/application-insights/app-insights-profiler
[2]: http://msdn.microsoft.com/library/azure/hh411542.aspx
[3]: http://blogs.msdn.com/b/habibh/archive/2009/06/30/walkthrough-using-the-tier-interaction-profiler-in-visual-studio-team-system-2010.aspx
[4]: ./media/cloud-services-performance-testing-visual-studio-profiler/ProfilingLocally09.png
[5]: ./media/cloud-services-performance-testing-visual-studio-profiler/ProfilingLocally10.png
[6]: ./media/cloud-services-performance-testing-visual-studio-profiler/ProfilingLocally02.png
[7]: ./media/cloud-services-performance-testing-visual-studio-profiler/ProfilingLocally05.png
[8]: ./media/cloud-services-performance-testing-visual-studio-profiler/ProfilingLocally010.png
[9]: ./media/cloud-services-performance-testing-visual-studio-profiler/ProfilingLocally07.png
[10]: ./media/cloud-services-performance-testing-visual-studio-profiler/ProfilingLocally06.png
[11]: ./media/cloud-services-performance-testing-visual-studio-profiler/ProfilingLocally03.png
[12]: ./media/cloud-services-performance-testing-visual-studio-profiler/ProfilingLocally011.png
[14]: ./media/cloud-services-performance-testing-visual-studio-profiler/ProfilingLocally04.png 
[15]: ./media/cloud-services-performance-testing-visual-studio-profiler/ProfilingLocally013.png
[16]: ./media/cloud-services-performance-testing-visual-studio-profiler/ProfilingLocally012.png
[17]: ./media/cloud-services-performance-testing-visual-studio-profiler/ProfilingLocally08.png
