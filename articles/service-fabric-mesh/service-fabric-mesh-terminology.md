---
title: 適用於 Azure Service Fabric Mesh 的術語 | Microsoft Docs
description: 了解適用於 Azure Service Fabric Mesh 的常見詞彙。
services: service-fabric-mesh
keywords: ''
author: rwike77
ms.author: ryanwi
ms.date: 07/12/2018
ms.topic: conceptual
ms.service: service-fabric-mesh
manager: timlt
ms.openlocfilehash: 7c3ce5571c54d6c613114ea49999e450934c8ff4
ms.sourcegitcommit: dc646da9fbefcc06c0e11c6a358724b42abb1438
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39136193"
---
# <a name="service-fabric-mesh-terminology"></a>Service Fabric Mesh 術語

Azure Service Fabric Mesh 是一個受到完整管理的服務，讓開發人員能夠部署微服務應用程式，而不需管理虛擬機器、儲存體或網路功能。 本文詳細說明 Azure Service Fabric Mesh 所使用的術語，以利您了解文件中使用的詞彙。

## <a name="service-fabric"></a>Service Fabric

Service Fabric 是一個開放原始碼的分散式系統平台，可讓您輕鬆封裝、部署及管理可調整規模和可信賴的微服務。 Service Fabric 是支援 Service Fabric Mesh 的協調器。 Service Fabric 提供如何建置及執行您微服務應用程式的選項。 您可以使用任何架構來撰寫服務，並從多種環境中選擇要執行應用程式的位置。

## <a name="deployment-and-application-models"></a>部署和應用程式模型 

若要部署您的服務，需要說明服務的執行方式。 Service Fabric 支援三種不同的部署模型：

### <a name="resource-model"></a>資源模型
資源是可個別部署至 Service Fabric 的任何項目，包括應用程式、服務、網路和磁碟區。 資源會利用 Azure 資源模型結構描述，藉由 YAML 檔案或 JSON 檔案來定義。

資源模型是描述 Service Fabric 應用程式的最簡單方式。 其主要重點是簡單的部署和容器化服務的管理。

資源可以部署於 Service Fabric 執行所在的任何位置。

### <a name="native-model"></a>原生模型
原生應用程式模型會為您的應用程式提供對 Service Fabric 的完整低層級存取權。 應用程式和服務會定義為已在 XML 資訊清單檔案中註冊的類型。

原生模型支援 Reliable Services 架構，可在 C# 和 Java 中提供對 Service Fabric 執行階段 API 及叢集管理 API 的存取權。 原生模型也支援任意的容器和可執行檔。

Service Fabric Mesh 環境中不支援原生模型。

### <a name="docker-compose"></a>Docker Compose 
[Docker Compose](https://docs.docker.com/compose/) \(英文\) 是 Docker 專案的一部分。 Service Fabric 針對使用 Docker Compose 模型來部署應用程式，提供有限的支援。

## <a name="environments"></a>環境

Service Fabric 是一項開放原始碼平台技術，有數個不同的服務和產品會以此技術為基礎。 Microsoft 提供下列選項：

 - **Service Fabric Mesh**：一個受到完整管理的服務，可用來在 Microsoft Azure 中執行 Service Fabric 應用程式。
 - **Azure Service Fabric**：Azure 裝載的 Service Fabric 叢集供應項目。 它提供 Service Fabric 和 Azure 基礎結構之間的整合，以及 Service Fabric 叢集的升級和設定管理。
 - **獨立 Service Fabric**：一組安裝和設定工具，[可將 Service Fabric 叢集部署到任何地方](/azure/service-fabric/service-fabric-deploy-anywhere) (內部部署或在任何雲端提供者上)。 不會由 Azure 管理。
 - **Service Fabric 開發叢集**：提供在 Windows、Linux 或 Mac 上的本機開發體驗，適用於開發 Service Fabric 應用程式。

## <a name="environment-framework-and-deployment-model-support-matrix"></a>環境、架構和部署模型支援對照表
不同的環境對於架構和部署模型會有不同層級的支援。 下表說明支援的架構和部署模型組合。

| 應用程式類型 | 描述依據 | Azure Service Fabric Mesh | Azure 服務叢集 (任何 OS)| 本機叢集 - Windows | 本機叢集 - Linux | 本機叢集 - Mac | 獨立叢集 (Windows)
|---|---|---|---|---|---|---|---|---|---|
| Service Fabric Mesh 應用程式 | 資源模型 (YAML & JSON) | 支援 |不支援 | 支援 |不支援 | 不支援 | 不支援 |
|Service Fabric 原生應用程式 | 原生應用程式模型 (XML) | 不支援| 支援|支援|支援|支援|支援|

下表針對 Service Fabric 描述不同的應用程式模型以及對應存在的工具。

| 應用程式類型 | 描述依據 | Visual Studio 2017 | Visual Studio 2015 | Eclipse | VS 程式碼 | SFCTL | AZ CLI | Powershell
|---|---|---|---|---|---|---|---|---|---|
| Service Fabric Mesh 應用程式 | 資源模型 (YAML & JSON) | 支援 |不支援 |不支援 |不支援 |不支援 | 支援 - 僅限 Mesh 環境 | 不支援
|Service Fabric 原生應用程式 | 原生應用程式模型 (XML) | 支援| 支援|支援|支援|支援|支援|支援|

## <a name="next-steps"></a>後續步驟

若要深入了解 Service Fabric Mesh，請閱讀下列概觀：
- [Service Fabric Mesh 概觀](service-fabric-mesh-overview.md)
