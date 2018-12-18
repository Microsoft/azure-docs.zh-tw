---
title: 使用 Excel 來連接到 Azure Analysis Services | Microsoft Docs
description: 了解如何使用 Excel 來連接到 Azure Analysis Services 伺服器。
author: minewiskan
manager: kfile
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 07/03/2018
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: 807496584acb3f93fccd3495de005792b769b37f
ms.sourcegitcommit: 86cb3855e1368e5a74f21fdd71684c78a1f907ac
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/03/2018
ms.locfileid: "37442985"
---
# <a name="connect-with-excel"></a>使用 Excel 進行連接

在您建立伺服器並將表格式模型部署至該伺服器之後，用戶端即可連線及開始瀏覽資料。 

## <a name="before-you-begin"></a>開始之前
您登入的帳戶必須屬於模型資料庫角色，且至少具有讀取權限。 若要深入了解，請參閱[驗證和使用者權限](analysis-services-manage-users.md)。 

## <a name="connect-in-excel"></a>在 Excel 中連線

藉由使用 Excel 2016 中的 [取得資料]，即可支援在 Excel 中連線到伺服器。 不支援在 Power Pivot 中使用 [匯入資料表精靈] 連線。 

**在 Excel 2016 中進行連接**

1. 在 Excel 2016 中的 [資料] 功能區上，按一下 [取得外部資料] >  **[從其他來源]** > [從 Analysis Services]。

2. 在 [資料連線精靈] 的 [伺服器名稱] 中，輸入包含通訊協定和 URI 的伺服器名稱。 例如 asazure://westcentralus.asazure.windows.net/advworks。 然後在 [登入認證] 中，選取 [使用下列的使用者名稱和密碼]，接著輸入組織使用者名稱 (例如 nancy@adventureworks.com) 和密碼。

    > [!IMPORTANT]
    > 如果您使用 Microsoft 帳戶、Live ID、Yahoo、Gmail 等登入，或者必須使用多重要素驗證登入，密碼欄位請空白。 您按 [下一步] 之後系統會提示您輸入密碼。 

    ![從 Excel 登入來進行連接](./media/analysis-services-connect-excel/aas-connect-excel-logon.png)

3. 在 [選取資料庫和資料表] 中，選取資料庫和模型或檢視方塊，然後按一下 [完成]。
   
    ![從 Excel 選取模型來進行連接](./media/analysis-services-connect-excel/aas-connect-excel-select.png)


## <a name="see-also"></a>另請參閱
[用戶端程式庫](analysis-services-data-providers.md)   
[管理您的伺服器](analysis-services-manage.md)     


