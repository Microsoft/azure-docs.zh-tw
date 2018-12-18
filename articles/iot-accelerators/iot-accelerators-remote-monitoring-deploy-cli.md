---
title: 部署 Java 遠端監視解決方案架構 - Azure | Microsoft Docs
description: 此教學課程會示範如何使用 CLI 來佈建遠端監視解決方案加速器。
author: dominicbetts
manager: timlt
ms.author: dobett
ms.service: iot-accelerators
services: iot-accelerators
ms.date: 09/12/2018
ms.topic: conceptual
ms.openlocfilehash: 56f233afed8c403d19c9b668e98ecfec45470b64
ms.sourcegitcommit: c29d7ef9065f960c3079660b139dd6a8348576ce
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/12/2018
ms.locfileid: "44721614"
---
# <a name="deploy-the-remote-monitoring-solution-accelerator-using-the-cli"></a>使用 CLI 部署遠端監視解決方案加速器

此教學課程示範如何佈建遠端監視解決方案加速器。 您將使用 CLI 來部署解決方案。 您也可以使用 azureiotsuite.com 的 Web 型 UI 來部署解決方案；若要了解此選項，請參閱[部署遠端監視解決方案加速器](quickstart-remote-monitoring-deploy.md)。

## <a name="prerequisites"></a>先決條件

若要部署遠端監視解決方案加速器，您需要一個有效的 Azure 訂用帳戶。

如果您沒有帳戶，只需要幾分鐘的時間就可以建立免費試用帳戶。 如需詳細資料，請參閱 [Azure 免費試用](http://azure.microsoft.com/pricing/free-trial/)。

若要執行 CLI，您必須在本機電腦上安裝 [Node.js](https://nodejs.org/)。

## <a name="install-the-cli"></a>安裝 CLI

若要安裝 CLI，請在您的命令列環境中執行下列命令：

```cmd/sh
npm install iot-solutions -g
```

## <a name="sign-in-to-the-cli"></a>登入 CLI

您必須依照下列方式使用 CLI 來登入 Azure 訂用帳戶，才能部署解決方案加速器：

```cmd/sh
pcs login
```

請依照畫面上的指示來完成登入程序。

## <a name="deployment-options"></a>部署選項

部署解決方案加速器時，有數個可設定部署程序的選項：

| 選項 | 值 | 說明 |
| ------ | ------ | ----------- |
| SKU    | `basic`、`standard`, `local` | _basic_ 部署適用於測試和示範環境，它會將所有微服務部署至單一虛擬機器。 _standard_ 部署適用於生產環境，它會將微服務部署至多部虛擬機器。 _local_ 部署會將 Docker 容器設定為在本機電腦上執行微服務，並且使用雲端中的 Azure 服務 (例如儲存體和 Cosmos DB)。 |
| 執行階段 | `dotnet`、`java` | 選取微服務的語言實作。 |

若要了解如何使用本機部署，請參閱[在本機執行遠端監視解決方案](iot-accelerators-remote-monitoring-deploy-local.md)。

## <a name="basic-vs-standard-deployments"></a>基本與標準部署

### <a name="basic"></a>基本
基本部署是專門設計來展示解決方案。 為了降低此示範的成本，所有微服務都會部署於單一虛擬機器中。這不會被視為可實際執行的架構。

當您準備自訂可實際執行的架構 (基於調整和擴充性而建置) 時，則應使用我們的標準部署選項。

建立基本解決方案將會按定價在您的 Azure 訂用帳戶中佈建下列 Azure 服務： 

| Count | 資源                       | 類型         | 用於 |
|-------|--------------------------------|--------------|----------|
| 1     | [Linux 虛擬機器](https://azure.microsoft.com/services/virtual-machines/) | 標準 D1 V2  | 裝載微服務 |
| 1     | [Azure IoT 中心](https://azure.microsoft.com/services/iot-hub/)                  | S1 – 標準層 | 裝置管理與通訊 |
| 1     | [Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/)              | 標準        | 儲存設定資料、規則、警示和其他冷儲存體 |  
| 1     | [Azure 儲存體帳戶](https://docs.microsoft.com/azure/storage/common/storage-introduction#types-of-storage-accounts)  | 標準        | 適用於 VM 和串流檢查點的儲存體 |
| 1     | [Web 應用程式](https://azure.microsoft.com/services/app-service/web/)        |                 | 裝載前端 Web 應用程式 |
| 1     | [Azure Active Directory](https://azure.microsoft.com/services/active-directory/)        |                 | 管理使用者身分識別和安全性 |
| 1     | [Azure 地圖服務](https://azure.microsoft.com/services/azure-maps/)        | 標準                | 檢視資產位置 |
| 1     | [Azure 串流分析](https://azure.microsoft.com/services/stream-analytics/)        |   3 個單位              | 提供即時分析 |
| 1     | [Azure 裝置佈建服務](https://docs.microsoft.com/azure/iot-dps/)        |       S1          | 大規模佈建裝置 |
| 1     | [Azure 時間序列深入解析](https://azure.microsoft.com/services/time-series-insights/)        |   S1 - 1 個單位              | 適用於訊息資料的儲存體，並能提供深入遙測分析 |



### <a name="standard"></a>標準
標準部署是開發人員可自訂及擴充來符合其需求的可實際執行部署。 針對可靠性與規模調整，應用程式微服務會建置來作為 Docker 容器，並使用協調器 (依預設為 [Kubernetes](https://kubernetes.io/) \(英文\)) 來部署。 協調器會負責部署、調整及管理應用程式。

建立標準解決方案將會按定價在您的 Azure 訂用帳戶中佈建下列 Azure 服務:

| Count | 資源                                     | SKU / 大小      | 用於 |
|-------|----------------------------------------------|-----------------|----------|
| 4     | [Linux 虛擬機器](https://azure.microsoft.com/services/virtual-machines/)   | 標準 D2 V2  | 1 部主機和 3 個代理程式，以用於裝載含有備援的微服務 |
| 1     | [Azure Container Service](https://azure.microsoft.com/services/container-service/) |                 | [Kubernetes](https://kubernetes.io) \(英文\) 協調器 |
| 1     | [Azure IoT 中心](https://azure.microsoft.com/services/iot-hub/)                     | S2 – 標準層 | 裝置管理、命令和控制 |
| 1     | [Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/)                 | 標準        | 儲存設定資料，以及規則、警示和訊息等裝置遙測 |
| 5     | [Azure 儲存體帳戶](https://docs.microsoft.com/azure/storage/common/storage-introduction#types-of-storage-accounts)    | 標準        | 4 個用於 VM 儲存體，1 個用於串流檢查點 |
| 1     | [App Service](https://azure.microsoft.com/services/app-service/web/)             | S1 標準     | 透過 SSL 的應用程式閘道 |
| 1     | [Azure Active Directory](https://azure.microsoft.com/services/active-directory/)        |                 | 管理使用者身分識別和安全性 |
| 1     | [Azure 地圖服務](https://azure.microsoft.com/services/azure-maps/)        | 標準                | 檢視資產位置 |
| 1     | [Azure 串流分析](https://azure.microsoft.com/services/stream-analytics/)        |   3 個單位              | 提供即時分析 |
| 1     | [Azure 裝置佈建服務](https://docs.microsoft.com/azure/iot-dps/)        |       S1          | 大規模佈建裝置 |
| 1     | [Azure 時間序列深入解析](https://azure.microsoft.com/services/time-series-insights/)        |   S1 - 1 個單位              | 適用於訊息資料的儲存體，並能提供深入遙測分析 |

> 您可以在[這裡](https://azure.microsoft.com/pricing)找到這些服務的價格資訊。 訂用帳戶的使用量與計費詳細資料則可在 [Azure 入口網站](https://portal.azure.com/)中找到。

## <a name="deploy-the-solution-accelerator"></a>部署解決方案加速器

### <a name="example-deploy-net-version"></a>範例：部署 .NET 版本

下列範例會示範如何部署基本 .NET 版的遠端監視解決方案加速器：

```cmd/sh
pcs -t remotemonitoring -s basic -r dotnet
```

### <a name="example-deploy-java-version"></a>範例：部署 Java 版本

下列範例會示範如何部署標準 Java 版的遠端監視解決方案加速器：

```cmd/sh
pcs -t remotemonitoring -s standard -r java
```

### <a name="pcs-command-options"></a>pcs 命令選項

當您執行 `pcs` 命令來部署解決方案時，系統會要求您提供：

- 您解決方案的名稱。 此名稱必須是唯一的。
- 要使用的 Azure 訂用帳戶。
- 一個位置。
- 裝載微服務之虛擬機器帳戶的認證。 您可以使用這些認證來存取虛擬機器以進行疑難排解。

當 `pcs` 命令完成時，會顯示新解決方案加速器部署的 URL。 `pcs` 命令也會建立一個檔案 `{deployment-name}-output.json`，其中含有額外的資訊，例如為您佈建的「IoT 中樞」名稱。

如需有關命令列參數的詳細資訊，請執行：

```cmd/sh
pcs -h
```

如需有關 CLI 的詳細資訊，請參閱 [如何使用 CLI](https://github.com/Azure/pcs-cli/blob/master/README.md) \(英文\)。

## <a name="next-steps"></a>後續步驟

在此教學課程中，您已了解如何：

> [!div class="checklist"]
> * 設定解決方案加速器
> * 部署解決方案加速器
> * 登入解決方案加速器

既然您已部署遠端監視解決方案，下一步便是[探索解決方案儀表板的功能](./quickstart-remote-monitoring-deploy.md)。

<!-- Next tutorials in the sequence -->
