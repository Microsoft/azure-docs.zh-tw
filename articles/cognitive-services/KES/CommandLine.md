---
title: 知識探索服務命令列介面 | Microsoft Docs
description: 使用 KES 命令列介面從結構化資料中建置索引和文法檔案，然後將其部署為 Microsoft 認知服務中的 Web 服務。
services: cognitive-services
author: bojunehsu
manager: stesp
ms.service: cognitive-services
ms.component: knowledge-exploration
ms.topic: article
ms.date: 03/24/2016
ms.author: paulhsu
ms.openlocfilehash: 71a6f5ac93e5605182a55de1bae9a99c5c3eddf4
ms.sourcegitcommit: dc646da9fbefcc06c0e11c6a358724b42abb1438
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39136350"
---
# <a name="command-line-interface"></a>命令列介面
KES 命令列介面提供的功能可從結構化資料中建置索引和文法檔案，並將其部署為 Web 服務。  其使用一般語法：`kes.exe <command> <required_args> [<optional_args>]`。  您可以在沒有引數的狀況下，執行 `kes.exe` 來顯示命令清單，或執行 `kes.exe <command>` 來顯示指定命令可用的引數清單。  以下是可用命令的清單：
* build_index
* build_grammar
* host_service
* deploy_service
* describe_index
* describe_grammar

<a name="build_index-command"></a>

## <a name="buildindex-command"></a>build_index 命令

針對要編製索引的物件，**Build_index** 命令可從該物件的結構描述定義檔和資料檔案中建置二進位索引檔案。  產生的索引檔案可用來評估結構化查詢運算式，或用來產生自然語言查詢，以與已編譯的文法檔案搭配使用。

`kes.exe build_index <schemaFile> <dataFile> <indexFile> [options]`

| 參數      | 說明               |
|----------------|---------------------------|
| `<schemaFile>` | 輸入結構描述路徑 |
| `<dataFile>`   | 輸入資料路徑   |
| `<indexFile>`  | 輸出索引路徑 |
| `--description <description>` | 描述描述 |
| `--remote <vmSize>`           | 用於遠端建置的 VM 大小 |

您可以使用本機檔案路徑或 Azure blob 的 URL 路徑來指定這些檔案。  結構描述檔案會描述要編製索引的物件結構，以及要支援的作業 (請參閱[結構描述格式](SchemaFormat.md))。  資料檔案會列舉要要編製索引的物件和屬性值 (請參閱[資料格式](DataFormat.md))。  建置成功時，輸出的索引檔案會包含輸入資料的壓縮表示，其可支援所需作業。  

您可選擇性地指定描述字串，以便之後可使用 **describe_index** 命令來識別二進位索引。  

根據預設，索引會建立在本機電腦上。  在 Azure 環境之外，本機組建有資料檔案最多只能包含 10,000 個物件的限制。  若指定--remote 旗標，則索引會建置在以指定大小臨時建立的 Azure VM 上。  如此一來，可有效率地使用 Azure VM 和更多記憶體來建置大型索引。  若要避免分頁造成建置程序變慢，建議您所用 VM 的 RAM 數量應是輸入資料檔案大小的 3 倍。  如需可用 VM 大小的清單，請參閱[虛擬機器的大小](../../../articles/virtual-machines/virtual-machines-windows-sizes.md)。

> [!TIP] 
> 若要加快建置速度，請藉由降低機率來預先分類資料檔案中的物件。

<a name="build_grammar-command"></a>

## <a name="buildgrammar-command"></a>build_grammar 命令

**Build_grammar** 命令會將 XML 中指定的文法編譯為二進位文法檔案。  產生的文法檔案可用來搭配索引檔案使用，以產生自然語言查詢的解譯。

`kes.exe build_grammar <xmlFile> <grammarFile>`

| 參數       | 說明               |
|-----------------|---------------------------|
| `<xmlFile>`     | 輸入 XML 文法規格的路經 |
| `<grammarFile>` | 輸出已編譯的文法路徑         |

您可以使用本機檔案路徑或 Azure blob 的 URL 路徑來指定這些檔案。  文法規格會描述一組加權的自然語言運算式及其語意解譯 (請參閱[文法格式](GrammarFormat.md))。  建置成功時，輸出的文法檔案會包含文法規格的二進位表示，以便進行快速解碼。

<a name="host_service-command"/>

## <a name="hostservice-command"></a>host_service 命令

**host_service** 命令會將 KES 服務的執行個體裝載在本機電腦上。

`kes.exe host_service <grammarFile> <indexFile> [options]`

| 參數       | 說明                |
|-----------------|----------------------------|
| `<grammarFile>` | 輸入二進位文法路徑         |
| `<indexFile>`   | 輸入二進位索引路徑           |
| `--port <port>` | 本機連接埠號碼。  預設值：8000 |

您可以使用本機檔案路徑或 Azure blob 的 URL 路徑來指定這些檔案。  Web 服務會裝載於 http://localhost:&lt;port&gt;/。  請參閱 [Web API](WebAPI.md) 以取得支援的作業清單。

在 Azure 環境之外，本機裝載的服務會有以下限制：索引檔案大小上限為 1 MB、每秒 10 個要求，以及總計 1000 個呼叫。  若要克服這些限制，請在 Azure VM 內執行 **host_service**，或使用 **deploy_service** 部署到 Azure 雲端服務。

<a name="deploy_service-command"/>

## <a name="deployservice-command"></a>deploy_service 命令

**deploy_service** 命令會將 KES 服務的執行個體部署至 Azure 雲端服務。

`kes.exe deploy_service <grammarFile> <indexFile> <serviceName> <vmSize>[options]`

| 參數       | 說明                  |
|-----------------|------------------------------|
| `<grammarFile>` | 輸入二進位文法路徑           |
| `<indexFile>`   | 輸入二進位索引路徑             |
| `<serviceName>` | 目標雲端服務的名稱 |
| `<vmSize>`      | 雲端服務 VM 的大小     |
| `--slot <slot>` | 雲端服務位置：「預備」(預設值)、「生產」 |

您可以使用本機檔案路徑或 Azure blob 的 URL 路徑來指定這些檔案。  服務名稱會指定預先設定的 Azure 雲端服務，(請參閱[如何建立及部署雲端服務](../../../articles/cloud-services/cloud-services-how-to-create-deploy-portal.md))。  此命令會自動將 KES 服務部署到指定的 Azure 雲端服務，並且使用指定大小的 VM。  若要避免分頁會大幅減少效能，我們建議所用 VM 的 RAM 應比輸入索引檔案大小多 1 GB。  如需可用 VM 大小的清單，請參閱[雲端服務的大小](../../../articles/cloud-services/cloud-services-sizes-specs.md)。

根據預設，服務會部署到預備環境中，您可以選擇性透過 --slot 參數加以覆寫。  請參閱 [Web API](WebAPI.md) 以取得支援的作業清單。

<a name="describe_index-command"/>

## <a name="describeindex-command"></a>describe_index 命令

**describe_index** 命令會輸出索引檔案的相關資訊，包括結構描述和描述。

`kes.exe describe_index <indexFile>`

| 參數     | 說明      |
|---------------|------------------|
| `<indexFile>` | 輸入索引路徑 |

此檔案可由本機檔案路徑或 Azure blob 的 URL 路徑來指定。  輸出描述字串可使用 **build_index** 命令的 --description 參數來指定。

<a name="describe_grammar-command"/>

## <a name="describegrammar-command"></a>describe_grammar 命令

**describe_grammar** 命令會輸出用來建置二進位文法的原始文法規格。

`kes.exe describe_grammar <grammarFile>`

| 參數       | 說明      |
|-----------------|------------------|
| `<grammarFile>` | 輸入文法路徑 |

此檔案可由本機檔案路徑或 Azure blob 的 URL 路徑來指定。

