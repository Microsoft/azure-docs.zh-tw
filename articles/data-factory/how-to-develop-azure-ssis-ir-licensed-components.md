---
title: 安裝 Azure-SSIS 整合執行階段的授權元件 | Microsoft Docs
description: 了解 ISV 如何開發及安裝 Azure SSIS 整合執行階段的付費或授權自訂元件
services: data-factory
documentationcenter: ''
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 04/13/2018
author: swinarko
ms.author: sawinark
ms.reviewer: douglasl
manager: craigg
ms.openlocfilehash: 146dc8c4475a041f28d7fe7ca464dfbc104258c7
ms.sourcegitcommit: 301855e018cfa1984198e045872539f04ce0e707
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/19/2018
ms.locfileid: "36265949"
---
# <a name="install-paid-or-licensed-custom-components-for-the-azure-ssis-integration-runtime"></a>安裝 Azure-SSIS 整合執行階段的付費或授權自訂元件

本文說明 ISV 如何為 Azure SSIS 整合執行階段中在 Azure 執行的 SQL Server Integration Services (SSIS) 套件開發及安裝付費或授權自訂元件。

## <a name="the-problem"></a>問題

Azure SSIS 整合執行階段的本質會帶來幾項挑戰，使得一般授權方法不足以用來在內部部署環境安裝自訂元件。 因此，Azure SSIS IR 需要不同的方法。

-   Azure-SSIS IR 的節點是動態的，隨時可以配置或釋放。 例如，您可以啟動或停止節點以管理成本，或者透過各種不同的節點大小相應增加和相應減少。 因此，藉由使用電腦專屬資訊 (例如 MAC 位址或 CPU 識別碼) 將第三方元件授權繫結至特定節點就不可行。

-   您也可以將 Azure SSIS IR 相應縮小或相應放大，讓節點數目可以隨時縮小或擴展。

## <a name="the-solution"></a>解決方案

由於上節中所述關於傳統授權方法的限制，Azure SSIS IR 提供了新的解決方案。 此解決方案使用 Windows 環境變數和 SSIS 系統變數來處理授權繫結和第三方元件的驗證。 ISV 可以使用這些變數來取得 Azure-SSIS IR 的唯一且具持續性的資訊，例如叢集識別碼和叢集節點計數。 有了這項資訊，ISV 就可以將其元件的授權做為*叢集*繫結至 Azure SSIS IR。 無論客戶啟動或停止、向上擴充或向下擴充、相應放大或縮小，或者以任何方式重新設定 Azure-SSIS IR，這個繫結所使用的識別碼皆不會變更。

下圖顯示典型安裝、啟用和授權繫結，以及使用這些新變數的第三方元件驗證流程：

![授權元件的安裝](media/how-to-configure-azure-ssis-ir-licensed-components/licensed-component-installation.png)

## <a name="instructions"></a>範例的指示
1. ISV 可以在各種 SKU 或層級提供其授權元件 (例如，單一節點、最多 5 個節點、最多 10 個節點，依此類推)。 當客戶購買產品時，ISV 會提供對應的產品金鑰。 ISV 也會提供 Azure 儲存體 Blob 容器，其中包含 ISV 安裝指令碼和相關聯的檔案。 客戶可以將這些檔案複製到他們自己的儲存體容器，並且使用自己的產品金鑰進行修改 (例如，藉由執行 `IsvSetup.exe -pid xxxx-xxxx-xxxx`)。 然後客戶可以使用其容器的 SAS URI 作為參數，佈建或重新設定 Azure-SSIS IR。 如需詳細資訊，請參閱 [Azure-SSIS 整合執行階段自訂設定](how-to-configure-azure-ssis-ir-custom-setup.md)。

2. 當 Azure-SSIS IR 已佈建或重新設定時，ISV 安裝會在每個節點上執行以查詢 Windows 環境變數 (`SSIS_CLUSTERID` 和 `SSIS_CLUSTERNODECOUNT`)。 然後 Azure-SSIS IR 會將其授權產品的叢集識別碼和產品金鑰提交給 ISV 啟用伺服器，來產生啟用金鑰。

3. 接收啟用金鑰之後，ISV 安裝程序可以在每個節點上將金鑰儲存在本機 (例如，在「登錄」中)。

4. 當客戶執行套件 (該套件在 Azure-SSIS IR 的節點上使用 ISV 授權元件) 時，套件會讀取本機儲存的啟用金鑰，並且根據節點的叢集識別碼來加以驗證。 套件也可以選擇性地將叢集節點計數報告至 ISV 啟用伺服器。

    以下是驗證啟用金鑰以及報告叢集節點計數的程式碼範例：

    ```csharp
    public override DTSExecResult Validate(Connections, VariableDispenser, IDTSComponentEvents componentEvents, IDTSLogging log) 
                                                                                                                               
    {                                                                                                                             
                                                                                                                               
    Variables vars = null;                                                                                                        
                                                                                                                               
    variableDispenser.LockForRead("System::ClusterID");                                                                           
                                                                                                                               
    variableDispenser.LockForRead("System::ClusterNodeCount");                                                                    
                                                                                                                               
    variableDispenser.GetVariables(ref vars);                                                                                     
                                                                                                                               
    // Validate Activation Key with ClusterID                                                                                     
                                                                                                                               
    // Report on ClusterNodeCount                                                                                                 
                                                                                                                               
    vars.Unlock();                                                                                                                
                                                                                                                               
    return base.Validate(connections, variableDispenser, componentEvents, log);                                                   
                                                                                                                               
    }
    ```

## <a name="isv-partners"></a>ISV 合作夥伴

您可以在本篇部落格文章[「企業版、自訂設定和第三方 SSIS 擴充性 ADF」](https://blogs.msdn.microsoft.com/ssis/2018/04/27/enterprise-edition-custom-setup-and-3rd-party-extensibility-for-ssis-in-adf/)的結尾找到一份 ISV 合作夥伴清單，其中包含已將其元件和擴充模組調整至適合 Azure SSIS IR 的合作夥伴。

## <a name="next-steps"></a>後續步驟

-   [Azure-SSIS 整合執行階段自訂設定](how-to-configure-azure-ssis-ir-custom-setup.md)

-   [Azure-SSIS 整合執行階段的 Enterprise Edition](how-to-configure-azure-ssis-ir-enterprise-edition.md)
