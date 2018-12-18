---
title: 將 Power BI Desktop 檔案匯入 Azure Analysis Services | Microsoft Docs
description: 描述如何使用 Azure 入口網站來匯入 Power BI Desktop 檔案 (pbix)。
author: minewiskan
manager: kfile
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 09/27/2018
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: e164488a1bf176d5b6c0e28a84cd1ec22cae4cce
ms.sourcegitcommit: 42405ab963df3101ee2a9b26e54240ffa689f140
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47423573"
---
# <a name="import-a-power-bi-desktop-file"></a>匯入 Power BI Desktop 檔案

您可以將 Power BI Desktop 檔案 (pbix) 中的資料模型匯入至 Azure Analysis Services。 將會匯入的項目包括模型中繼資料、快取資料和資料來源連線。 報告和視覺效果則不會匯入。 從 Power BI Desktop 匯入的資料模型是在 1400 相容性層級。

**限制**   

- 從 pbix 檔案匯入會使用入口網站的 Web 設計工具功能，此功能目前為**預覽**版。 功能會受到限制。 對於進階模型開發和測試，最好使用 Visual Studio (SSDT) 和 SQL Server Management Studio (SSMS)。
- 如果您的資料模型是以 Power BI Desktop 2018 年 7 月更新 (2.60.5169.3201) 或更新版本來建立的，請確定您並未啟用任何預覽功能。 Azure Analysis Services 中尚未支援預覽功能。  
如果您在匯入時接收到下列錯誤，則表示 pbix 檔案已啟用 Azure Analysis Services 中尚未支援的預覽功能。

    ![相容性層級的警告](./media/analysis-services-import-pbix/aas-import-pbix-cl-warning.png)   
- 您必須擁有伺服器管理員權限，才能從 pbix 檔案匯入。
- pbix 模型只能連線到 **Azure SQL Database** 和 **Azure SQL 資料倉儲**資料來源。
- Pbix 模型不能有即時或 DirectQuery 連線。 


## <a name="to-import-from-pbix"></a>從 pbix 匯入

1. 在您的伺服器 [概觀] > [Web 設計工具] 中，按一下 [開啟]。

    ![在 Azure 入口網站中建立模型](./media/analysis-services-create-model-portal/aas-create-portal-overview-wd.png)

2. 在 [Web 設計工具]  >  [模型] 中，按一下 [+ 新增]。

    ![在 Azure 入口網站中建立模型](./media/analysis-services-create-model-portal/aas-create-portal-models.png)

3. 在 [新增模型] 中輸入模型名稱，然後選取 Power BI Desktop 檔案。

    ![在 Azure 入口網站中的新增模型對話方塊](./media/analysis-services-import-pbix/aas-import-pbix-new-model.png)

4. 在 [匯入] 中找到並選取您的檔案。

     ![在 Azure 入口網站中的連線對話方塊](./media/analysis-services-import-pbix/aas-import-pbix-select-file.png)

## <a name="change-credentials"></a>管理認證

當您從 pbix 檔案匯入資料模型時，用來連線到資料來源的認證預設會設定為 ServiceAccount。 從 pbix 匯入模型後，您就可以使用下列方法來變更認證：

- 請使用 2018 年 7 月 (17.8.1 版) 或更新版本的 SSMS 來編輯認證。 這是最簡單的方法。
- 在[資料來源物件](https://docs.microsoft.com/sql/analysis-services/tabular-models-scripting-language-objects/datasources-object-tmsl)上使用 TMSL [Alter 命令](https://docs.microsoft.com/sql/analysis-services/tabular-models-scripting-language-commands/alter-command-tmsl)來修改連接字串屬性。 
- 在 Visual Studio 中開啟模型，編輯資料來源連線的認證，然後重新部署模型。

若要使用 SSMS 來變更認證。 

1. 在 SSMS 中，展開 [資料庫] > [連線]。 
2. 對資料庫連線按一下滑鼠右鍵，然後按一下 [重新整理認證]。 

    ![重新整理認證](./media/analysis-services-import-pbix/aas-import-pbix-creds.png)

3. 在 [認證] 對話方塊中，選取認證類型，然後輸入認證。 針對 SQL 驗證，請選取 [資料庫]。 針對組織帳戶 (OAuth)，則選取 [Microsoft 帳戶]。
    ![編輯認證](./media/analysis-services-import-pbix/aas-import-pbix-edit-creds.png)

2018 年 7 月版的 Power BI Desktop 包含可變更資料來源權限的新功能。 在 [首頁] 索引標籤上，按一下 [編輯查詢]  > [資料來源設定]。 選取資料來源連線，然後按一下 [編輯權限]。


## <a name="see-also"></a>另請參閱

[在 Azure 入口網站中建立模型](analysis-services-create-model-portal.md)   
[連線至 Azure Analysis Services](analysis-services-connect.md)  
