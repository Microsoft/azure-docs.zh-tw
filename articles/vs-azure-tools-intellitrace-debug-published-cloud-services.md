---
title: 使用 Visual Studio 和 IntelliTrace 進行已發佈 Azure 雲端服務的偵錯 | Microsoft Docs
description: 了解如何使用 Visual Studio 和 IntelliTrace，進行雲端服務的偵錯
services: visual-studio-online
documentationcenter: n/a
author: mikejo
manager: douge
editor: ''
ms.assetid: 5e6662fc-b917-43ea-bf2b-4f2fc3d213dc
ms.service: visual-studio-online
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: multiple
ms.custom: vs-azure
ms.workload: azure-vs
ms.date: 03/21/2017
ms.author: mikejo
ms.openlocfilehash: 1deec539d3dfe65a2ac02a9f4cd2b94d9a78274a
ms.sourcegitcommit: fab878ff9aaf4efb3eaff6b7656184b0bafba13b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/22/2018
ms.locfileid: "42444337"
---
# <a name="debugging-a-published-azure-cloud-service-with-visual-studio-and-intellitrace"></a>使用 Visual Studio 和 IntelliTrace 進行已發佈 Azure 雲端服務的偵錯
有了 IntelliTrace，您可以於角色執行個體在 Azure 中執行時，記錄其廣泛的偵錯資訊。 如果您需要找出問題的原因，您可以從 Visual Studio 使用 IntelliTrace 記錄檔來瀏覽程式碼，如同它是在 Azure 中執行。 實際上，Azure 應用程式在 Azure 中以雲端服務形式執行時，IntelliTrace 會記錄主要執行程式碼和環境資料，並且可讓您從 Visual Studio 重新播放記錄的資料。 

如果您有安裝 Visual Studio Enterprise，而您的 Azure 應用程式以 .NET Framework 4 或更新版本為目標，則可以使用 IntelliTrace。 IntelliTrace 會收集 Azure 角色的資訊。 這些角色的虛擬機器一律會執行 64 位元作業系統。

或者，您可以使用[遠端偵錯](http://go.microsoft.com/fwlink/p/?LinkId=623041)，直接附加到在 Azure 中執行的雲端服務。

> [!IMPORTANT]
> IntelliTrace 僅適用於偵錯，並且不應該用於生產環境部署。
> 

## <a name="configure-an-azure-application-for-intellitrace"></a>為 IntelliTrace 設定 Azure 應用程式
若要為 Azure 應用程式啟用 IntelliTrace，您必須從 Visual Studio Azure 專案建立並發佈應用程式。 發佈至 Azure 之前，您必須您的 Azure 應用程式設定 IntelliTrace。 如果您發佈應用程式，但未設定 IntelliTrace，則您必須重新發佈專案。 如需詳細資訊，請參閱[使用 Visual Studio 專案發佈 Azure 雲端服務](http://go.microsoft.com/fwlink/p/?LinkId=623012)。

1. 準備好要部署 Azure 應用程式時，請確認專案建置目標是設定為 [偵錯] 。

1. 在 [方案總管] 中，以滑鼠右鍵按一下專案，然後從操作功能表中選取 [發佈]。
   
1. 在 [發行 Azure 應用程式] 對話方塊中，選取 Azure 訂用帳戶，然後選取 [下一步]。

1. 在 [設定] 頁面上，選取 [進階設定] 索引標籤。

1. 開啟 [啟用 IntelliTrace]  選項，以便在將應用程式發佈至雲端時收集該應用程式的 IntelliTrace 記錄檔。
   
1. 若要自訂基本的 IntelliTrace 組態，請選取 [啟用 IntelliTrace] 旁的 [設定]。

    ![IntelliTrace 設定連結](./media/vs-azure-tools-intellitrace-debug-published-cloud-services/intellitrace-settings-link.png)
   
1. 在 [IntelliTrace 設定] 對話方塊中，您可以指定要記錄的事件、是否要收集呼叫資訊、要收集記錄的模組和處理序，以及配置多少空間來存放記錄。 如需有關 IntelliTrace 的詳細資訊，請參閱 [使用 IntelliTrace 進行偵錯](http://go.microsoft.com/fwlink/?LinkId=214468)。
   
    ![IntelliTrace 設定](./media/vs-azure-tools-intellitrace-debug-published-cloud-services/IC519063.png)

IntelliTrace 記錄檔是循環記錄檔，大小上限是在 IntelliTrace 設定中指定 (預設大小為 250 MB)。 IntelliTrace 記錄檔會收集到虛擬機器的檔案系統中的檔案。 要求記錄檔時，會擷取該時間點的快照，並下載到您的本機電腦。

將 Azure 雲端服務發佈至 Azure 之後，您可以從 [伺服器總管] 中的 Azure 節點判斷是否已啟用 IntelliTrace，如下圖所示：

![伺服器總管 - 已啟用 IntelliTrace](./media/vs-azure-tools-intellitrace-debug-published-cloud-services/IC744134.png)

## <a name="download-intellitrace-logs-for-a-role-instance"></a>下載角色執行個體的 IntelliTrace 記錄檔
使用 Visual Studio，您可以依照下列步驟來下載角色執行個體的 IntelliTrace 記錄檔：

1. 在 [伺服器總管] 中，展開 [雲端服務] 節點，然後找出您要下載其記錄的角色執行個體。 

1. 以滑鼠右鍵按一下角色執行個體，然後從操作功能表中，選取 [檢視 IntelliTrace 記錄檔]。 

    ![[檢視 IntelliTrace 記錄檔] 功能表選項](./media/vs-azure-tools-intellitrace-debug-published-cloud-services/view-intellitrace-logs.png)

1. IntelliTrace 記錄檔會下載到本機電腦的目錄中的檔案。 每當您要求 IntelliTrace 記錄檔，即會建立新的快照。 下載記錄檔時，Visual Studio 會在 [Azure 活動記錄] 視窗中顯示作業的進度。 如下圖所示，您可以展開作業的行項目，以查看詳細資料。

![VST_IntelliTraceDownloadProgress](./media/vs-azure-tools-intellitrace-debug-published-cloud-services/IC745551.png)

下載 IntelliTrace 記錄檔時，您可以繼續使用 Visual Studio。 當記錄檔完成下載時，它會在 Visual Studio 中開啟。

> [!NOTE]
> IntelliTrace 記錄檔可能包含架構隨後產生並處理的例外狀況。 內部架構程式碼會在正常啟動角色時產生這些例外狀況，因此您可以放心忽略。
> 
> 

## <a name="next-steps"></a>後續步驟
- [進行 Azure 雲端服務偵錯的選項](vs-azure-tools-debugging-cloud-services-overview.md)
- [使用 Visual Studio 發佈 Azure 雲端服務](vs-azure-tools-publishing-a-cloud-service.md)