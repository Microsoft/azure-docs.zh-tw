---
title: Azure 的命令列建置 | Microsoft Docs
description: Azure 的命令列建置
services: visual-studio-online
author: ghogen
manager: douge
assetId: 94b35d0d-0d35-48b6-b48b-3641377867fd
ms.prod: visual-studio-dev15
ms.technology: vs-azure
ms.workload: azure
ms.topic: conceptual
ms.date: 03/05/2017
ms.author: ghogen
ms.openlocfilehash: 7d0138abb07aea46ad8d0069c87964b393347dcf
ms.sourcegitcommit: fa493b66552af11260db48d89e3ddfcdcb5e3152
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/23/2018
ms.locfileid: "31791421"
---
# <a name="building-azure-projects-from-the-command-line"></a>從命令列建置 Azure 專案
使用 Microsoft Build Engine (MSBuild) 時，您可以在未安裝 Visual Studio 的測試版環境中建置產品。 MSBuild 的專案檔案會採用可擴充且 Microsoft 完全支援的 XML 格式。 使用 MSBuild 檔案格式時，您可以說明針對一或多個平台和組態所必須建置的項目。

您也可以在命令列執行 MSBuild，本主題將說明這個方法。 藉由在命令列設定屬性，您將可以建置專案的特定組態。 同樣地，您也可以定義 MSBuild 建置的目標。 如需有關命令列參數及 MSBuild 的詳細資訊，請參閱 [MSBuild 命令列參考](https://msdn.microsoft.com/library/ms164311.aspx)。

## <a name="msbuild-parameters"></a>MSBuild 參數
若要建立封裝，最簡單的方法是使用 `/t:Publish` 選項來執行 MSBuild。 此命令預設會建立專案根資料夾的相關目錄，例如 `<ProjectDirectory>\bin\Configuration\app.publish\`。 當您建置 Azure 專案時，會產生兩個檔案，即套件檔案本身及伴隨的組態檔：

* 套件檔案 (`project.cspkg`)
* 組態檔 (`ServiceConfiguration.TargetProfile.cscfg`)

每個 Azure 專案預設都會包含一個供本機 (偵錯) 組建使用的服務組態檔，以及另一個供雲端 (預備或生產環境) 組建使用的服務組態檔。 不過，您可以視需要新增或移除服務組態檔。 當您在 Visual Studio 內建置套件時，系統會詢問您要隨套件包含哪個服務組態檔。 使用 MSBuild 來建置封裝時，依預設會加入本機服務組態檔。 若要加入不同的服務組態檔，請設定 `TargetProfile`MSBuild 命令的屬性 (`MSBuild /t:Publish /p:TargetProfile=ProfileName`)。

如果您想要為儲存的封裝和組態檔使用替代目錄，請使用 `/p:PublishDir=Directory\` 選項來設定路徑 (包括結尾的反斜線分隔符號)。

## <a name="next-steps"></a>後續步驟
建立封裝之後，您可以將它部署至 Azure。