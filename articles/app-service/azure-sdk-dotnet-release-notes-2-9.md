---
title: Azure SDK for .NET 2.9 版本資訊
description: Azure SDK for .NET 2.9 版本資訊
services: app-service\web
documentationcenter: .net
author: chrissfanos
editor: ''
ms.assetid: c83d815b-fc19-4260-821e-7d2a7206dffc
ms.service: app-service
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: integration
ms.date: 02/24/2017
ms.author: juliako
ms.openlocfilehash: 01b8ccc9fe6b5469408131bce68a903f15382998
ms.sourcegitcommit: 44fa77f66fb68e084d7175a3f07d269dcc04016f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/24/2018
ms.locfileid: "39222523"
---
# <a name="azure-sdk-for-net-29-release-notes"></a>Azure SDK for .NET 2.9 版本資訊

本主題包含 Azure SDK for .NET 2.9 和 2.9.6 版的版本資訊。

## <a name="azure-sdk-for-net-296-release-summary"></a>Azure SDK for .NET 2.9.6 發行摘要

發行日期︰2016/11/16
 
Azure SDK 2.9 在此版本中沒有重大變更。 在現有的雲端服務專案上使用這套 SDK 也不需要升級程序。

### <a name="visual-studio-2017-release-candidate"></a>Visual Studio 2017 候選版

- Visual Studio 2017 RC 中已內建這個版本的 Azure SDK for .NET，以便用於 Azure 工作負載。 未來將會在 Visual Studio 2017 RC 中提供您開發 Azure 所需的一切工具。 在 Visual Studio 2015 和 Visual Studio 2013 中，將仍然可以透過 WebPI 使用這套 SDK。 當 Visual Studio 2017 以最終產品發行時，我們就不再提供適用於 Visual Studio 2013 的 Azure SDK for .NET 版本。 請依照此連結下載 Visual Studio 2017 RC： https://www.visualstudio.com/vs/visual-studio-2017-rc/

### <a name="azure-diagnostics"></a>Azure 診斷

- 已變更為只儲存部分連接字串，並以雲端服務儲存體連接字串的權杖取代金鑰。 實際的儲存體金鑰現在儲存在使用者設定檔資料夾中，方便控制其存取權。 Visual Studio 會從使用者設定檔資料夾中讀取儲存體金鑰，以執行本機偵錯和發佈程序。 
- 為了因應上述變更，Visual Studio Online 小組已增強 Azure 雲端服務部署工作範本，當使用者在持續整合及部署中發佈至 Azure 時，可以指定儲存體金鑰來設定診斷擴充。
- 我們已可讓您儲存安全連接字串和 Azure 診斷 (WAD) 的 Token 化，協助您解決跨環境組態的問題。
 
### <a name="windows-server-2016-virtual-machines"></a>Windows Server 2016 虛擬機器

- Visual Studio 現在支援將雲端服務部署到作業系統系列 5 (Windows Server 2016) 虛擬機器。 對於現有的雲端服務，您可以變更設定，以新的作業系統系列為目標。 建立新的雲端服務時，如果您選擇使用 .net 4.6 或更高版本建立服務，服務會預設為使用作業系統系列 5。  如需詳細資訊，您可以檢閱[客體 OS 系列支援資料表](https://azure.microsoft.com/documentation/articles/cloud-services-guestos-update-matrix/)。

#### <a name="known-issues"></a>已知問題

- Azure.NET SDK 2.9.6 引進了一項限制，會使用不支援的 .NET Framework (例如 .NET 4.6) 至任何 < 5 的作業系統系列封鎖專案的部署。 因應措施提供於[這裡](https://github.com/MicrosoftDocs/azure-cloud-services-files/tree/master/Azure%20Targets%20SDK%202.9)。

 
### <a name="azure-in-role-cache"></a>Azure In-Role Cache 

- 2016 年 11 月 30 日終止支援 Azure In-Role Cache。 如需詳細資訊，請按一下[這裡](https://azure.microsoft.com/blog/azure-managed-cache-and-in-role-cache-services-to-be-retired-on-11-30-2016/)。

### <a name="azure-resource-manager-templates-for-azure-stack"></a>適用於 Azure Stack 的 Azure Resource Manager 範本

- 我們已引進 Azure Stack 做為 Azure Resource Manager 範本的部署目標。


## <a name="azure-sdk-for-net-29-summary"></a>Azure SDK for .NET 2.9 摘要

## <a name="overview"></a>概觀
本文包含 Azure SDK for .NET 2.9 版的版本資訊。 

如需此版本之更新內容的詳細資訊，請參閱 [Azure SDK 2.9 公告文章](https://azure.microsoft.com/blog/announcing-visual-studio-azure-tools-and-sdk-2-9/)。

## <a name="azure-sdk-29-for-visual-studio-2015-update-2-and-visual-studio-15-preview"></a>Azure SDK 2.9 for Visual Studio 2015 Update 2 和 Visual Studio "15" 預覽
此更新包含下列錯誤修正：

* 與 REST API 用戶端產生有關的問題，在此問題中，字串「未知類型」會出現成為 code-gen 資料夾的名稱和/或放入所產生程式碼之命名空間的名稱。
* 與排程的 WebJobs 有關的問題，在此問題中，驗證資訊無法傳遞給排程器佈建程序。

此更新包含下列新功能：

* 在 App Service 佈建對話方塊的 [服務] 索引標籤中支援第二個 App Service。 

## <a name="azure-data-lake-tools-for-visual-studio-2015-update-2"></a>Azure Data Lake Tools for Visual Studio 2015 Update 2
此更新包含下列工具：

* **Azure Data Lake Tools** for Visual Studio 現在已合併到 Azure SDK for .NET 版本。 當您安裝 Azure SDK 時，便會自動安裝此工具。 
  
    此工具會經常更新，請前往 [這裡](http://aka.ms/datalaketool) 取得更新。
* **伺服器總管** 現在可讓您全部檢視，並建立一些 U-SQL 中繼資料實體。 如需詳細資訊，請參閱 [此部落格](https://azure.microsoft.com/documentation/services/data-lake-analytics/) 。

## <a name="hdinsight-tools"></a>HDInsight 工具
**HDInsight Tools** for Visual Studio 現在支援 HDInsight 3.3 版，包括顯示 Tez 圖形和其他語言修正。

## <a name="azure-resource-manager"></a>Azure Resource Manager
此版本為 Resource Manager 範本新增[金鑰保存庫](../azure-resource-manager/resource-manager-keyvault-parameter.md)支援。

## <a name="see-also"></a>另請參閱
[Azure SDK 2.9 公告文章](https://azure.microsoft.com/blog/announcing-visual-studio-azure-tools-and-sdk-2-9/)

