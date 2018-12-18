---
title: Azure Data Factory 中的整合執行階段 | Microsoft Docs
description: 了解 Azure Data Factory 中的整合執行階段。
services: data-factory
documentationcenter: ''
author: linda33wj
manager: craigg
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 06/14/2018
ms.author: jingwang
ms.openlocfilehash: dafbfb959e70563f8619f7aea877a3aa1c380453
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46997398"
---
# <a name="integration-runtime-in-azure-data-factory"></a>Azure Data Factory 中的整合執行階段
整合執行階段 (IR) 是 Azure Data Factory 所使用的計算基礎結構，可跨不同網路環境提供下列資料整合功能：

- **資料移動**：在公用網路中的資料存放區與私人網路 (內部部署或虛擬私人網路) 中的資料存放區之間移動資料。 它支援內建的連接器、格式轉換、資料行對應，以及高效能和可調式資料轉送。
- **活動分派**：分派和監視在 Azure HDInsight、Azure Machine Learning、Azure SQL Database、SQL Server 等各種計算服務上執行的轉換活動。
- **SSIS 套件執行**：在受控 Azure 計算環境中，以原生方式執行 SQL Server Integration Services (SSIS) 套件。

在 Data Factory 中，活動可定義要執行的動作。 連結服務可定義目標資料存放區或計算服務。 整合執行階段提供活動與連結服務之間的橋樑。  由連結服務所參考，並提供計算環境來執行活動或分派活動。  如此一來，就能在最接近目標資料存放區或計算服務的區域執行活動，效率最高，又滿足安全性和合規性需求。

## <a name="integration-runtime-types"></a>整合執行階段類型
Data Factory 提供三種整合執行階段，您應該選擇最符合所需之資料整合功能和網路環境需求的類型。  這三種類型為：

- Azure
- 自我裝載
- Azure-SSIS

下表描述每個整合執行階段類型的功能和網路支援：

IR 類型 | 公用網路 | 私人網路
------- | -------------- | ---------------
Azure | 資料移動<br/>活動分派 | &nbsp;
自我裝載 | 資料移動<br/>活動分派 | 資料移動<br/>活動分派
Azure-SSIS | SSIS 套件執行 | SSIS 套件執行

下圖顯示如何結合使用不同的整合執行階段，以提供豐富的資料整合功能和網路支援：

![不同類型的整合執行階段](media\concepts-integration-runtime\different-integration-runtimes.png)

## <a name="azure-integration-runtime"></a>Azure 整合執行階段
Azure 整合執行階段能夠：

- 在雲端資料存放區之間執行複製活動
- 在公用網路中分派下列轉換活動：HDInsight Hive 活動、HDInsight Pig 活動、HDInsight MapReduce 活動、HDInsight Spark 活動、HDInsight 串流活動、Machine Learning 批次執行活動、Machine Learning 更新資源活動、預存程序活動、Data Lake Analytics U-SQL 活動、.Net 自訂活動、網路活動、查閱活動，以及取得中繼資料活動。

### <a name="azure-ir-network-environment"></a>Azure IR 網路環境
在具有公開存取端點的公用網路中，Azure 整合執行階段支援連線至其中的資料存放區和計算服務。 在 Azure 虛擬網路環境中使用自我裝載整合執行階段。

### <a name="azure-ir-compute-resource-and-scaling"></a>Azure IR 計算資源和調整規模
Azure 整合執行階段在 Azure 中提供完全受控、無伺服器的計算。  您不必擔心基礎結構佈建、軟體安裝、修補或容量大小調整。  此外，您只需支付實際使用時間。

Azure 整合執行階段提供原生計算，能夠以安全、可靠且高效能的方式，在雲端資料存放區之間移動資料。  您可以設定要在複製活動上使用的資料整合單位數量，Azure IR 的計算大小會很有彈性地相應增加，您不必明確地調整 Azure Integration Runtime 的大小。

活動分派是輕量型的作業，可將活動路由傳送至目標計算服務，所以在此情節中，不需要相應增加計算大小。

如需有關建立及設定 Azure IR 的資訊，請參閱操作說明指南中的＜如何建立和設定 Azure IR＞。 

## <a name="self-hosted-integration-runtime"></a>自我裝載整合執行階段
自我裝載 IR 能夠：

- 在雲端資料存放區和私人網路中的資料存放區之間執行複製活動。
- 在內部部署或 Azure 虛擬網路中，針對計算資源分派下列轉換活動：HDInsight Hive 活動 (BYOC)、HDInsight Pig 活動 (BYOC)、HDInsight MapReduce 活動 (BYOC)、HDInsight Spark 活動 (BYOC)、HDInsight 串流活動 (BYOC)、Machine Learning 批次執行活動、Machine Learning 更新資源活動、預存程序活動、Data Lake Analytics U-SQL 活動、.Net 自訂活動、查閱活動，以及取得中繼資料活動。

> [!NOTE] 
> 使用自我裝載整合執行階段來支援需要自備驅動程式 (例如 SAP Hana、MySQL 等) 的資料存放區。如需詳細資訊，請參閱[支援的資料存放區](copy-activity-overview.md#supported-data-stores-and-formats)。

### <a name="self-hosted-ir-network-environment"></a>自我裝載 IR 網路環境
如果想要在私人網路環境 (無法從公用雲端環境直接存取) 中安全地執行資料整合，您可以在內部部署環境、公司防火牆後方或虛擬私人網路內安裝自我裝載 IR。  自我裝載整合執行階段對於開放網際網路，只會建立以 HTTP 為基礎的輸出連線。

### <a name="self-hosted-ir-compute-resource-and-scaling"></a>自我裝載 IR 計算資源和調整規模
自我裝載 IR 必須安裝在私人網路內的內部部署機器或虛擬機器上。 目前，我們只支援在 Windows 作業系統上執行自我裝載 IR。  

若要達到高可用性和延展性，您可以在主動-主動模式下，將邏輯執行個體和多個內部部署機器產生關聯，以相應放大自我裝載 IR。  如需詳細資訊，請參閱操作說明指南中的＜如何建立和設定自我裝載 IR＞一文。

## <a name="azure-ssis-integration-runtime"></a>Azure-SSIS 整合執行階段
若要隨即轉移現有的 SSIS 工作負載，您可以建立 Azure-SSIS IR，以原生方式執行 SSIS 套件。

### <a name="azure-ssis-ir-network-environment"></a>Azure-SSIS IR 網路環境
Azure-SSIS IR 可以佈建在公用網路或私人網路中。  將 Azure-SSIS IR 加入已連線至內部網路的虛擬網路，即可支援內部部署資料存取。  

### <a name="azure-ssis-ir-compute-resource-and-scaling"></a>Azure-SSIS IR 計算資源和調整規模
Azure-SSIS IR 是一個完全受控的 Azure VM 叢集，專門用來執行您的 SSIS 套件。 您可以自備 Azure SQL Database 或受控執行個體伺服器，以裝載要附加至伺服器的 SSIS 專案/套件 (SSISDB) 目錄。 指定節點大小可以相應增加計算能力，指定叢集的節點數目可以相應放大計算能力。 您可以依需求來停止和啟動 Azure-SSIS 整合執行階段，以掌控其執行成本。

如需詳細資訊，請參閱操作說明指南中的＜如何建立和設定 Azure-SSIS IR＞一文。  建立之後，您可以使用熟悉的工具，例如 SQL Server Data Tools (SSDT) 和 SQL Server Management Studio (SSMS)，就像在內部部署環境中使用 SSIS 一樣，不太需要變更就能部署和管理現有的 SSIS 套件。

如需 Azure-SSIS 執行階段的詳細資訊，請參閱下列文章： 

- [教學課程：將 SSIS 套件部署至 Azure](tutorial-create-azure-ssis-runtime-portal.md)。 本文逐步說明如何建立 Azure-SSIS IR，並使用 Azure SQL Database 裝載 SSIS 目錄。 
- [如何：建立 Azure-SSIS 整合執行階段](create-azure-ssis-integration-runtime.md)。 這篇文章會詳述教學課程，並提供使用 Azure SQL Database 受控執行個體，以及將 IR 加入虛擬網路的指示。 
- [監視 Azure-SSIS IR](monitor-integration-runtime.md#azure-ssis-integration-runtime). 本文示範如何在傳回的資訊中擷取 Azure-SSIS IR 的相關資訊和狀態描述。 
- [管理 Azure-SSIS IR](manage-azure-ssis-integration-runtime.md). 本文示範如何停止、啟動或移除 Azure-SSIS IR。 它也會示範如何將更多節點新增至 IR，藉此相應放大 Azure-SSIS IR。 
- [將 Azure-SSIS IR 加入虛擬網路](join-azure-ssis-integration-runtime-virtual-network.md)。 這篇文章提供將 Azure SSIS IR 加入至 Azure 虛擬網路的概念資訊。 它也提供使用 Azure 入口網站來設定虛擬網路，好讓 Azure SSIS IR 可加入虛擬網路的步驟。 

## <a name="integration-runtime-location"></a>整合執行階段位置
Data Factory 位置中儲存資料處理站的中繼資料，也是觸發管道的源頭。 同時，資料處理站可以存取其他 Azure 區域的資料存放區和計算資料，以在資料存放區之間移動資料或使用計算服務處理資料。 此行為會透過[全域可用的 IR](https://azure.microsoft.com/global-infrastructure/services/) 來達成，以確保資料合規性、效率，並降低網路輸出成本。

「IR 位置」定義其後端計算的位置，基本上還會定義執行資料移動、活動分派和 SSIS 套件執行的位置。 IR 位置及其所屬的資料處理站位置可能不同。 

### <a name="azure-ir-location"></a>Azure IR 位置
您可以設定特定的 Azure IR 位置，在此情況下，資料移動或活動分派將會在該區域中執行。 

如果您選擇使用自動解析 Azure IR (此為預設值)， 

- 針對複製活動，ADF 會盡可能地自動偵測您的接收和來源資料存放區，以選擇位於相同區域中的最佳位置 (如果有的話) 或相同地理位置中最接近的位置；如果未偵測到，則使用資料處理站區域作為替代區域。
- 針對查閱/GetMetadata 活動執行與轉換活動分派，ADF 將會在資料處理站區域中使用 IR。

您可以在 UI 上的管線活動監視檢視中或活動監視承載中，監視在活動執行期間生效的 IR 位置。

>[!TIP]
>如果您有嚴格的資料合規性需求，且必須確定資料不會離開特定地理位置，您可以明確地在特定區域中建立 Azure IR，並使用 ConnectVia 屬性將連結服務指向此 IR。 例如，如果您想要將位於英國南部的 Blob 資料複製到英國南部的 SQL DW，且想要確保資料不會離開英國，請在英國南部中建立 Azure IR，並將兩個連結服務都連結至此 IR。

### <a name="self-hosted-ir-location"></a>自我裝載 IR 位置
自我裝載 IR 在邏輯上會向 Data Factory 註冊，而用來支援其功能的計算由您提供。 因此，自我裝載 IR 沒有明確的位置屬性。 

用來執行資料移動時，自我裝載 IR 會從來源取出資料，並寫入目的地。

### <a name="azure-ssis-ir-location"></a>Azure-SSIS IR 位置
為了在擷取、轉換和下載 (ETL) 工作流程中達到高效能，務必選取正確的 Azure-SSIS IR 位置。

- Azure-SSIS IR 的位置不需要與資料處理站的位置相同，但應該與您自己的 Azure SQL Database/受控執行個體伺服器 (要裝載 SSISDB) 的位置相同。 如此一來，您的 Azure-SSIS 整合執行階段就可以輕易存取 SSISDB，而不會在不同的位置之間產生過多流量。
- 如果您沒有現有的 Azure SQL Database/受控執行個體伺服器來裝載 SSISDB，但有內部部署資料來源/目的地，您應該在已連線至內部部署網路之虛擬網路的相同位置中，建立新的 Azure SQL Database/受控執行個體伺服器。  如此一來，您就可以使用新的 Azure SQL Database/受控執行個體伺服器並加入該虛擬網路，以建立 Azure-SSIS IR，全部都在相同的位置中，能儘量避免在不同位置之間移動資料。
- 如果要裝載 SSISDB 的現有 Azure SQL Database/受控執行個體伺服器的位置，與連線至內部部署網路的虛擬網路的位置不相同，請先在相同位置使用現有的 Azure SQL Database/受控執行個體伺服器並加入另一個，以建立您的 Azure-SSIS IR，然後設定不同位置之間的虛擬網路對虛擬網路連線。

下圖顯示 Data Factory 及其整合執行階段的位置設定：

![整合執行階段位置](media/concepts-integration-runtime/integration-runtime-location.png)

## <a name="determining-which-ir-to-use"></a>決定使用哪一個 IR

### <a name="copy-activity"></a>複製活動

若是複製活動，需要來源和接收連結服務來定義資料流程的方向。 下列邏輯可決定使用哪個整合執行階段執行個體來執行複製： 

- **在兩個雲端資料來源之間複製**：當來源和接收連結服務皆使用 Azure IR 時，ADF 將會在您已指定區域 Azure IR 時使用該 IR，或在您選擇自動解析 IR (預設值) 時自動判斷 Azure IR 的位置，如[整合執行階段位置](#integration-runtime-location)一節所說明。
- **在雲端資料來源與私人網路中的資料來源之間複製**：如果任一來源或接收連結服務指向自我裝載 IR，則會在自我裝載整合執行階段上執行複製活動。
- **在私人網路中的兩個資料來源之間複製**：來源和接收連結服務必須指向相同的整合執行階段執行個體，而該整合執行階段會用來執行複製活動。

### <a name="lookup-and-getmetadata-activity"></a>查閱和 GetMetadata 活動

查閱和 GetMetadata 的活動會在與資料存放區連結服務相關聯的整合執行階段上執行。

### <a name="transformation-activity"></a>轉換活動

每個轉換活動都有一個指向整合執行階段的目標計算「連結服務」。 轉換活動就是從這個整合執行階段執行個體分派而來。

## <a name="next-steps"></a>後續步驟
請參閱下列文章：

- [建立自我裝載整合執行階段](create-self-hosted-integration-runtime.md)
- [建立 Azure-SSIS Integration Runtime](create-azure-ssis-integration-runtime.md)。 這篇文章會詳述教學課程，並提供使用 Azure SQL Database 受控執行個體，以及將 IR 加入虛擬網路的指示。 
