---
title: 監視 Azure 雲端服務 | Microsoft Docs
description: 描述監視 Azure 雲端服務所涉及的作業以及您的一些選項。
services: cloud-services
documentationcenter: ''
author: jpconnock
manager: timlt
editor: ''
ms.assetid: ''
ms.service: cloud-services
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/29/2018
ms.author: jeconnoc
ms.openlocfilehash: e9fbe59c40fe55218429a3b479ddbbec7220c66a
ms.sourcegitcommit: 8ebcecb837bbfb989728e4667d74e42f7a3a9352
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/21/2018
ms.locfileid: "42145564"
---
# <a name="introduction-to-cloud-service-monitoring"></a>雲端服務監視簡介

您可以監視任何雲端服務的關鍵效能計量。 每個雲端服務角色都會收集最少資料：CPU 使用量、網路使用量和磁碟使用狀況。 如果雲端服務已將 `Microsoft.Azure.Diagnostics` 延伸模組套用至角色，則該角色可以收集其他資料點。 本文介紹 Azure Diagnostics for Cloud Services。

使用基本監視，角色執行個體的效能計數器資料會依 3 分鐘的間隔進行取樣和收集。 此基本監視資料不會儲存在儲存體帳戶中，而且沒有任何與其建立關聯的額外成本。

使用進階監視，其他計量會依 5 分鐘、1 小時和 12 小時的間隔進行取樣和收集。 彙總的資料會儲存在儲存體帳戶和資料表中，並會在 10 天之後予以清除。 所使用的儲存體帳戶是依角色所設定；您可以對不同的角色使用不同的儲存體帳戶。 這是使用 [.csdef](cloud-services-model-and-package.md#servicedefinitioncsdef) 和 [.cscfg](cloud-services-model-and-package.md#serviceconfigurationcscfg) 檔案中的連接字串所設定。


## <a name="basic-monitoring"></a>基本監視

如簡介所述，雲端服務會自動從主機虛擬機器收集基本監視資料。 此資料包含 CPU 百分比、網路輸入/輸出和磁碟讀取/寫入。 在 Azure 入口網站之雲端服務的概觀和計量頁面上，會自動顯示所收集的監視資料。 

基本監視不需要儲存體帳戶。 

![基本雲端服務監視磚](media/cloud-services-how-to-monitor/basic-tiles.png)

## <a name="advanced-monitoring"></a>進階監視

進階監視包含對您想要監視的角色使用 **Azure 診斷**延伸模組 (和選擇性的 Application Insights SDK)。 診斷延伸模組會使用名為 **diagnostics.wadcfgx** 的設定檔 (每個角色)，來設定所監視的診斷計量。 Azure 診斷延伸模組會收集資料並且儲存在 Azure 儲存體帳戶。 這些設定是在 **.wadcfgx**、[.csdef](cloud-services-model-and-package.md#servicedefinitioncsdef) 和 [.cscfg](cloud-services-model-and-package.md#serviceconfigurationcscfg) 檔案中設定。 這表示會有與進階監視建立關聯的額外成本。

建立每個角色時，Visual Studio 會在其中新增 Azure 診斷延伸模組。 此診斷延伸模組可以收集下列類型的資訊：

* 自訂效能計數器
* 應用程式記錄檔
* Windows 事件記錄檔
* .NET 事件來源
* IIS 記錄檔
* 以資訊清單為基礎的 ETW
* 損毀傾印
* 客戶錯誤記錄

> [!IMPORTANT]
> 雖然此資料全都會彙總至儲存體帳戶，但是入口網站**不會**提供製作資料圖表的原生方法。 強烈建議您將其他像是 Application Insights 的服務整合到您的應用程式。

## <a name="setup-diagnostics-extension"></a>設定診斷延伸模組

首先，如果您沒有**傳統**儲存體帳戶，則請[建立傳統儲存體帳戶](../storage/common/storage-quickstart-create-account.md)。 請確定建立已指定**傳統部署模型**的儲存體帳戶。

接下來，巡覽至 [儲存體帳戶 (傳統)] 資源。 選取 [設定] > [存取金鑰]，然後複製 [主要連接字串] 值。 雲端服務需要有此值。 

您必須變更兩個設定檔才能啟用進階診斷：**ServiceDefinition.csdef** 和 **ServiceConfiguration.cscfg**。

### <a name="servicedefinitioncsdef"></a>ServiceDefinition.csdef

在 **ServiceDefinition.csdef** 檔案中，針對每個使用進階診斷的角色，新增名為 `Microsoft.WindowsAzure.Plugins.Diagnostics.ConnectionString` 的新設定。 當您建立新專案時，Visual Studio 會將這個值新增至檔案。 若遺失，您可以立即予以新增。 

```xml
<ServiceDefinition name="AnsurCloudService" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceDefinition" schemaVersion="2015-04.2.6">
  <WorkerRole name="WorkerRoleWithSBQueue1" vmsize="Small">
    <ConfigurationSettings>
      <Setting name="Microsoft.WindowsAzure.Plugins.Diagnostics.ConnectionString" />
```

這會定義必須新增至每個 **ServiceConfiguration.cscfg** 檔案的新設定。 

您很可能會有兩個 **.cscfg** 檔案，一個名為 **ServiceConfiguration.cloud.cscfg** 以用於部署至 Azure，一個名為 **ServiceConfiguration.local.cscfg** 以在模擬環境中用於本機部署。 開啟並變更每個 **.cscfg** 檔案。 新增名為 `Microsoft.WindowsAzure.Plugins.Diagnostics.ConnectionString` 的設定。 將值設定為傳統儲存體帳戶的**主要連接字串**。 如果您想要在開發電腦上使用本機儲存體，請使用 `UseDevelopmentStorage=true`。

```xml
<ServiceConfiguration serviceName="AnsurCloudService" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceConfiguration" osFamily="4" osVersion="*" schemaVersion="2015-04.2.6">
  <Role name="WorkerRoleWithSBQueue1">
    <Instances count="1" />
    <ConfigurationSettings>
      <Setting name="Microsoft.WindowsAzure.Plugins.Diagnostics.ConnectionString" value="DefaultEndpointsProtocol=https;AccountName=mystorage;AccountKey=KWwkdfmskOIS240jnBOeeXVGHT9QgKS4kIQ3wWVKzOYkfjdsjfkjdsaf+sddfwwfw+sdffsdafda/w==" />
      
      <!-- or use the local development machine for storage
      <Setting name="Microsoft.WindowsAzure.Plugins.Diagnostics.ConnectionString" value="UseDevelopmentStorage=true" />
      -->
```

## <a name="use-application-insights"></a>使用 Application Insights

當您從 Visual Studio 發佈雲端服務時，可以選擇將診斷資料傳送至 Application Insights。 您可以在當時建立 Application Insights Azure 資源，或將資料傳送至現有的 Azure 資源。 Application Insights 可以監視雲端服務的可用性、效能、失敗和使用情況。 自訂圖表可以新增至 Application Insights，因此您可以看到最重要的資料。 在雲端服務專案中使用 Application Insights SDK，即可收集角色執行個體資料。 如需如何整合 Application Insights 的詳細資訊，請參閱[含雲端服務的 Application Insights](../application-insights/app-insights-cloudservices.md)。

請注意，雖然您可以使用 Application Insights 來顯示透過 Windows Azure 診斷延伸模組所指定的效能計數器 (和其他設定)，但是只有將 Application Insights SDK 整合到背景工作和 Web 角色，才能獲得較豐富的體驗。


## <a name="next-steps"></a>後續步驟

- [深入了解含雲端服務的 Application Insights](../application-insights/app-insights-cloudservices.md)
- [設定效能計數器](diagnostics-performance-counters.md)

