---
title: 在 Azure Data Factory 中建立 Azure 整合執行階段 | Microsoft Docs
description: 了解如何在 Azure Data Factory 中建立 Azure 整合執行階段，它可用來複製資料和分派轉換活動。
services: data-factory
documentationcenter: ''
author: douglaslMS
manager: craigg
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 01/15/2018
ms.author: douglasl
ms.openlocfilehash: cb1a263c0a33a291a44e7c60b3c032d7f9dc16a3
ms.sourcegitcommit: 0c490934b5596204d175be89af6b45aafc7ff730
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37054070"
---
# <a name="how-to-create-and-configure-azure-integration-runtime"></a>如何建立和設定 Azure 整合執行階段
整合執行階段 (IR) 是 Azure Data Factory 所使用的計算基礎結構，可提供跨不同網路環境的資料整合功能。 如需 IR 的詳細資訊，請參閱[整合執行階段](concepts-integration-runtime.md)。

Azure IR 提供完全受控的計算，以原生方式執行資料移動並將資料轉換活動分派到計算服務 (例如 HDInsight)。 它裝載於 Azure 環境中，並且支援連線到具有可公開存取端點之公用網路環境中的資源。

本文件會介紹如何建立和設定 Azure 整合執行階段。 

## <a name="default-azure-ir"></a>預設 Azure IR
根據預設，每個資料處理站的後端都有 Azure IR，可支援在雲端資料存放區和在公用網路內計算服務的作業。 該 Azure IR 的位置是自動解析的。 如果連結的服務定義中並未指定 **connectVia** 屬性，則會使用預設的 Azure IR。 只有在您要明確定義 IR 位置，或是基於管理目的而以虛擬方式將不同 IR 上的活動執行作業分組時，才需要明確地建立 Azure IR。 

## <a name="create-azure-ir"></a>建立 Azure IR
整合執行階段可使用 **Set-AzureRmDataFactoryV2IntegrationRuntime** PowerShell Cmdlet 來建立。 若要建立 Azure IR，您要對該命令指定名稱、位置和類型。 以下是使用位置設定為 "West Europe" (西歐) 來建立 Azure IR 的範例命令：

```powershell
Set-AzureRmDataFactoryV2IntegrationRuntime -DataFactoryName "SampleV2DataFactory1" -Name "MySampleAzureIR" -ResourceGroupName "ADFV2SampleRG" -Type Managed -Location "West Europe"
```  
針對 Azure IR，類型必須設定為**受控**。 您不需要指定計算詳細資料，因為它在雲端中以彈性的方式受到完整管理。 當您要建立 Azure-SSIS IR 時，才需要指定例如節點大小和節點計數的計算詳細資料。 如需詳細資訊，請參閱[建立和設定 Azure-SSIS IR](create-azure-ssis-integration-runtime.md)。

您可以使用 Set-AzureRmDataFactoryV2IntegrationRuntime PowerShell Cmdlet 來設定現有的 Azure IR 以變更其位置。 如需 Azure IR 位置的詳細資訊，請參閱[整合執行階段簡介](concepts-integration-runtime.md)。

## <a name="use-azure-ir"></a>使用 Azure IR

Azure IR 建立之後，您可以在「已連結的服務」定義中參考它。 以下是從 Azure 儲存體連結的服務參考上述所建立之 Azure 整合執行階段的方法範例：  

```json
{
    "name": "MyStorageLinkedService",
    "properties": {
      "type": "AzureStorage",
      "typeProperties": {
        "connectionString": {
          "value": "DefaultEndpointsProtocol=https;AccountName=myaccountname;AccountKey=...",
          "type": "SecureString"
        }
      },
      "connectVia": {
        "referenceName": "MySampleAzureIR",
        "type": "IntegrationRuntimeReference"
      }   
    }
}

```

## <a name="next-steps"></a>後續步驟
若要了解如何建立其他類型的整合執行階段，請參閱下列文章：

- [建立自我裝載整合執行階段](create-self-hosted-integration-runtime.md)
- [建立 Azure-SSIS 整合執行階段](create-azure-ssis-integration-runtime.md)
 
