---
title: 從 Machine Learning Studio 匯出和刪除資料 | Microsoft Docs
description: 透過 Azure 入口網站和經過驗證的 REST API，可以匯出和刪除 Azure Machine Learning Studio 所儲存的產品中資料。 透過 Azure 隱私權入口網站可以存取遙測資料。 本文將說明如何做到。
services: machine-learning
author: heatherbshapiro
ms.author: hshapiro
manager: cgronlun
ms.reviewer: jmartens, mldocs
ms.service: machine-learning
ms.component: studio
ms.topic: conceptual
ms.date: 05/25/2018
ms.openlocfilehash: 2ebd777a9723732de6ebbdf07020802190cb4b61
ms.sourcegitcommit: f606248b31182cc559b21e79778c9397127e54df
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/12/2018
ms.locfileid: "38969277"
---
# <a name="export-and-delete-in-product-user-data-from-machine-learning-studio"></a>從 Machine Learning Studio 匯出和刪除產品中使用者資料

您可以使用 Azure 入口網站、Studio 介面、PowerShell 和經過驗證的 REST API，刪除或匯出 Azure Machine Learning Studio 所儲存的產品中資料。 本文會說明如何做到。 

透過 Azure 隱私權入口網站可以存取遙測資料。 

[!INCLUDE [GDPR-related guidance](../../../includes/gdpr-dsr-and-stp-note.md)]

[!INCLUDE [GDPR-related guidance](../../../includes/gdpr-intro-sentence.md)]

## <a name="what-kinds-of-user-data-does-studio-collect"></a>Studio 會收集哪幾種使用者資料？

在這項服務中，使用者資料包含有權存取工作區的使用者相關資訊，以及使用者與服務互動的遙測記錄。

Machine Learning Studio 中有兩種使用者資料：
- **個人帳戶資料：** 與帳戶相關聯的帳戶識別碼和電子郵件地址。
- **客戶資料：** 您上傳以供分析的資料。

## <a name="studio-account-types-and-how-data-is-stored"></a>Studio 帳戶類型和資料的儲存方式

Machine Learning Studio 中有三種帳戶。 您擁有的帳戶種類可決定您資料的儲存方式，以及其刪除或匯出方式。

- **來賓工作區**是免費的匿名帳戶。 您不需提供認證 (例如電子郵件地址或密碼) 即可註冊。
    -  資料會在來賓工作區過期後清除。
    - 來賓使用者可以透過 UI、REST API 或 PowerShell 套件匯出客戶資料。
- **免費工作區**是您可使用 Microsoft 帳戶認證 (電子郵件地址和密碼) 登入的免費帳戶。
    - 您可以匯出及刪除個人和客戶資料，而這些資料受資料主體權限 (DSR) 要求所約束。
    - 您可以透過 UI、REST API 或 PowerShell 套件匯出客戶資料。
    - 若為未使用 Azure AD 帳戶的免費工作區，可以使用隱私權入口網站匯出遙測。
    - 當您刪除工作區時，您會刪除所有個人和客戶資料。
- **標準工作區**是您可使用登入認證存取的付費帳戶。
    - 您可以匯出及刪除個人和客戶資料，而這些資料受 DSR 要求所約束。
    - 您可以透過 Azure 隱私權入口網站存取資料
    - 您可以透過 UI、REST API 或 PowerShell 套件匯出個人和客戶資料。
    - 您可以在 Azure 入口網站中刪除您的資料。

## <a name="delete-workspace-data-in-studio"></a>在 Studio 中刪除工作區資料 

### <a name="delete-individual-assets"></a>刪除個別資產

使用者可藉由選取工作區中的資產，然後選取 [刪除] 按鈕加以刪除。

![刪除 Machine Learning Studio 中的資產](./media/export-delete-personal-data-dsr/delete-studio-asset.png)

### <a name="delete-an-entire-workspace"></a>刪除整個工作區

使用者也可以刪除整個工作區：
- 付費工作區：透過 Azure 入口網站刪除。
- 免費工作區：使用 [設定] 窗格中的 [刪除] 按鈕。

![在 Machine Learning Studio 中刪除免費工作區](./media/export-delete-personal-data-dsr/delete-studio-data-workspace.png)
 
## <a name="export-studio-data-with-powershell"></a>透過 PowerShell 匯出 Studio 資料
透過 PowerShell，使用命令從 Azure Machine Learning Studio 將您所有的資訊匯出為可攜式格式。 如需詳細資訊，請參閱[適用於 Azure Machine Learning 的 PowerShell 模組](powershell-module.md)。

## <a name="next-steps"></a>後續步驟

如需涵蓋 Web 服務和承諾計劃計費的文件，請參閱 [Azure Machine Learning REST API 參考](https://docs.microsoft.com/rest/api/machinelearning/)。 
