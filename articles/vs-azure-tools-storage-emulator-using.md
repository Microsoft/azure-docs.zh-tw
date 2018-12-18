---
title: 在 Visual Studio 中設定和使用儲存體模擬器 | Microsoft Docs
description: 在 Visual Studio 中設定和使用儲存體模擬器
services: visual-studio-online
author: ghogen
manager: douge
assetId: c8e7996f-6027-4762-806e-614b93131867
ms.prod: visual-studio-dev15
ms.technology: vs-azure
ms.custom: vs-azure
ms.workload: azure-vs
ms.topic: conceptual
ms.date: 8/17/2017
ms.author: ghogen
ms.openlocfilehash: 4b14fa44f484735f2a5efee4d631a6f48e6a8bf6
ms.sourcegitcommit: a3a0f42a166e2e71fa2ffe081f38a8bd8b1aeb7b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/01/2018
ms.locfileid: "43382631"
---
# <a name="configuring-and-using-the-storage-emulator-with-visual-studio"></a>在 Visual Studio 中設定和使用儲存體模擬器
[!INCLUDE [storage-try-azure-tools](../includes/storage-try-azure-tools.md)]

## <a name="overview"></a>概觀
Azure SDK 開發環境包含儲存體模擬器，這是一個公用程式，可在本機開發電腦上模擬 Azure 提供的 blob、佇列和資料表服務。 如果您要建置雲端服務來採用 Azure 儲存體服務，或撰寫任何外部應用程式來呼叫儲存體服務，您可以在本機利用儲存體模擬器來測試您的程式碼。 Azure Tools for Microsoft Visual Studio 已將儲存體模擬器的管理整合到 Visual Studio 中。 Azure Tools 會在第一次使用儲存體模擬器資料庫時初始化、當您從 Visual Studio 執行或偵錯程式碼時啟動儲存體模擬器服務，並以唯讀方式透過 [Azure 儲存體總管] 存取儲存體模擬器資料。

如需儲存體模擬器的詳細資訊，包括系統需求和自訂組態指示，請參閱 [使用 Azure 儲存體模擬器進行開發和測試](storage/common/storage-use-emulator.md)。

> [!NOTE]
> 儲存體模擬器模擬與 Azure 儲存體服務的功能有所差異。 如需特定差異的詳細資訊，請參閱 Azure SDK 文件中的 [儲存體模擬器和 Azure 儲存體服務之間的差異](storage/common/storage-use-emulator.md)。
> 
> 

## <a name="configuring-a-connection-string-for-the-storage-emulator"></a>設定儲存體模擬器的連接字串
若要在角色內從程式碼存取儲存體模擬器，您可以設定連接字串來指向儲存體模擬器，稍後再變更為指向 Azure 儲存體帳戶。 連接字串是您的角色在執行階段可讀取來連接到儲存體帳戶的組態設定。 如需如何建立連接字串的詳細資訊，請參閱[設定 Azure 儲存體連接字串](/azure/storage/common/storage-configure-connection-string)。

> [!NOTE]
> 您可以使用 **DevelopmentStorageAccount** 屬性，從程式碼傳回儲存體模擬器帳戶的參考。 如果您想要從程式碼存取儲存體模擬器，這種方法能正常運作，但如果您計劃將應用程式發佈至 Azure，則必須建立連接字串來存取 Azure 儲存體帳戶，並在發佈前修改程式碼以使用該連接字串。 如果您經常在儲存體模擬器帳戶和 Azure 儲存體帳戶之間切換，連接字串可以簡化這個程序。
> 
> 

## <a name="initializing-and-running-the-storage-emulator"></a>初始化及執行儲存體模擬器
您可以指定當您在 Visual Studio 中執行或偵錯服務時，讓 Visual Studio 自動啟動儲存體模擬器。 在 [方案總管] 中，開啟 **Azure** 專案的捷徑功能表並選擇 [屬性]。 在 [開發] 索引標籤的 [啟動 Azure 儲存體模擬器] 清單中，選擇 [True] \(如果未尚設定為此值)。

第一次從 Visual Studio 執行或偵錯服務時，儲存體模擬器會啟動初始化程序。 此程序會保留本機連接埠給儲存體模擬器，並建立儲存體模擬器資料庫。 完成後，除非刪除儲存體模擬器資料庫，否則此程序不必再次執行。

> [!NOTE]
> 從 Azure Tools 的 2012 年 6 月版本開始，依預設，儲存體模擬器會在 SQL Express LocalDB 中執行。 在舊版的 Azure Tools 中，儲存體模擬器會針對 SQL Express 2005 或 2008 的預設執行個體執行，必須先安裝此軟體才能安裝 Azure SDK。 您也可以針對 SQL Express 具名執行個體或是 Microsoft SQL Server 具名或預設執行個體來執行儲存體模擬器。 如果您需要設定儲存體模擬器來針對預設執行個體以外的執行個體執行，請參閱 [使用 Azure 儲存體模擬器進行開發和測試](storage/common/storage-use-emulator.md)。
> 
> 

儲存體模擬器提供使用者介面，可讓您檢視本機儲存體服務的狀態，並啟動、停止和重新設定它們。 一旦啟動儲存體模擬器服務，您可以用滑鼠右鍵按一下 Windows 工作列的 Microsoft Azure 模擬器通知區域圖示，以顯示使用者介面或是啟動或停止服務。

## <a name="viewing-storage-emulator-data-in-server-explorer"></a>在 [伺服器總管] 中檢視儲存體模擬器資料
[伺服器總管] 中的 [Azure 儲存體] 節點可讓您檢視儲存體帳戶中的資料，以及變更 blob 和資料表資料的設定，包括儲存體模擬器。 如需詳細資訊，請參閱[使用儲存體總管管理 Azure Blob 儲存體資源](https://docs.microsoft.com/azure/vs-azure-tools-storage-explorer-blobs)。

