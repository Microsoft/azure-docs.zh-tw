---
title: 準備從 Visual Studio 發佈或部署雲端服務 | Microsoft Docs
description: 了解設定雲端和儲存體帳戶服務以及 Azure 應用程式的程序。
services: visual-studio-online
author: ghogen
manager: douge
ms.assetid: 92ee2f9e-ec49-4c7a-900d-620abe5e9d8a
ms.prod: visual-studio-dev15
ms.technology: vs-azure
ms.custom: vs-azure
ms.workload: azure-vs
ms.topic: conceptual
ms.date: 11/10/2017
ms.author: ghogen
ms.openlocfilehash: 084ba64389e2f3f8d62b77697df368dac7e09eac
ms.sourcegitcommit: 30c7f9994cf6fcdfb580616ea8d6d251364c0cd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/18/2018
ms.locfileid: "42140074"
---
# <a name="prepare-to-publish-or-deploy-a-cloud-service-from-visual-studio"></a>準備從 Visual Studio 發佈或部署雲端服務

若要發佈雲端服務專案，必須依本文所述設定下列服務：

* **雲端服務**，可在 Azure 環境中執行角色，以及 
* 提供 Blob、佇列和表格服務等存取權的 **儲存體帳戶** 。

## <a name="create-a-cloud-service"></a>建立雲端服務

雲端服務可在 Azure 環境中執行角色。 您可以使用 Visual Studio 或是透過 [Azure 入口網站](https://portal.azure.com/)建立雲端服務，以下各節會作說明。

### <a name="create-a-cloud-service-from-visual-studio"></a>從 Visual Studio 建立雲端服務

1. 使用先前建立的雲端服務專案，以滑鼠右鍵按一下專案，選取 [發佈]。
1. 如有需要，請使用與您的 Azure 訂用帳戶相關聯的 Microsoft 帳戶或組織帳戶登入，然後選取 [下一步]，前進到 [設定] 頁面。
1. [建立雲端服務與儲存體帳戶] 對話方塊會隨即出現，如果沒有顯示，請在 [雲端服務] 清單選取 [新建]。
1. 為您的雲端服務輸名稱 (不區分大小寫)，該名稱會成為 URL 的一部分，必須為專屬的名稱。 另請選擇 [區域] 或 [同質群組]，然後選取 [複寫] 選項。

### <a name="create-a-cloud-service-through-the-azure-portal"></a>透過 Azure 入口網站建立雲端服務

1. 登入 [Azure 入口網站](https://portal.azure.com/)。
1. 選取頁面左邊的 [雲端服務 (傳統)]。
1. 選取 [+新增]，然後提供所需的資訊 (DNS 名稱、訂用帳戶、資源群組，以及位置)。 此時不需要上傳封裝，因為稍後會在 Visual Studio 中上傳。
1. 選取 [建立]，完成此流程：

## <a name="create-a-storage-account"></a>建立儲存體帳戶

儲存體帳戶會提供 Blob、佇列和表格服務等存取權。 您可以透過 Visual Studio 或 [Azure 入口網站](https://portal.azure.com/)建立儲存體帳戶。

### <a name="create-a-storage-account-from-visual-studio"></a>從 Visual Studio 建立儲存體帳戶

1. 在具有先前建立之雲端服務專案的 [方案總管] 中，找出角色專案中的**已連線的服務**節點，按一下滑鼠右鍵，然後選取 [新增已連線的服務]。 (在 Visual Studio 2015 中，需以滑鼠右鍵按一下**儲存體**節點，然後選取 [建立儲存體帳戶]。)
1. 在顯示的 [已連線的服務] 清單中，選取 [使用 Azure 儲存體的雲端儲存體]。
1. 在顯示的 [Azure 儲存體] 對話方塊中，選取 [+建立新的儲存體帳戶]，會隨即開啟一個對話方塊，可以從中指定您的訂用帳戶、該帳戶的名稱、定價層、資源群組以及位置。
1. 完成後，選取 [建立]。 新的儲存體帳戶會出現在訂用帳戶ˇ的可用儲存體帳戶清單中。
1. 選取該帳戶，然後選取 [新增]。

### <a name="create-a-storage-account-through-the-azure-portal"></a>透過 Azure 入口網站建立儲存體帳戶

1. 登入 [Azure 入口網站](https://portal.azure.com/)。
1. 選取左上方的 [+ 新增]。
1. 在「Azure Marketplace」下方選取 [儲存體]，然後從右側選取 [儲存體帳戶 - Blob、檔案、表格、佇列]。
1. 提供所需的資訊，如名稱、部署模型等。
1. 選取 [建立]，完成此流程：

## <a name="configure-your-app-to-use-the-storage-account"></a>設定您的應用程式以使用儲存體帳戶

建立儲存體帳戶之後，若從 Visual Studio 連線到該帳戶，即會自動更新該專案的服務設定，URL 與存取金鑰也包括在內。

如果您已使用 [新增已連線的服務] 從 Visual Studio 建立雲端服務，便可開啟 `ServiceConfiguration.Cloud.cscfg` 和 `ServiceConfiguration.Local.cscfg` 來檢查連線。

如果您已透過 Azure 入口網站建立雲端服務，請按照[從 Visual Studio 建立儲存體帳戶](#create-a-storage-account-from-visual-studio)的相同步驟，不過請選取現有帳戶，而非建立新帳戶。 接著 Visual Studio 會更新您的設定。

若要手動設定設定值，請在 Visual Studio 中，使用您的雲端服務專案適用角色的屬性頁面 (以滑鼠右鍵按一下角色，選取 [屬性])。 如需詳細資訊，請參閱[設定儲存體帳戶的連接字串](https://docs.microsoft.com/azure/vs-azure-tools-multiple-services-project-configurations#configuring-a-connection-string-to-a-storage-account)。

### <a name="about-access-keys"></a>關於存取金鑰

Azure 入口網站會顯示可用來存取每個 Azure 儲存體服務中之資源的 URL，以及帳戶的主要和次要存取金鑰。 您可以使用這些金鑰來驗證對儲存體服務提出的要求。

次要存取金鑰提供和主要存取金鑰相同的儲存體帳戶存取權，並會在主要存取金鑰遭到入侵時產生做為備份。 此外，建議您定期重新產生存取金鑰。 您可以修改連接字串設定以在您重新產生主要金鑰時使用次要金鑰，然後您可以修改它以在重新產生次要金鑰時使用重新產生的主要金鑰。

## <a name="next-steps"></a>後續步驟

若要深入了解如何將應用程式從 Visual Studio 發佈至 Azure，請參閱 [使用 Azure Tools 發佈雲端服務](vs-azure-tools-publishing-a-cloud-service.md)。
