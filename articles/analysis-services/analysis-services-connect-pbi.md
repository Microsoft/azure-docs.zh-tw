---
title: 使用 Power BI 來連接到 Azure Analysis Services | Microsoft Docs
description: 了解如何使用 Power BI 來連接到 Azure Analysis Services 伺服器。
author: minewiskan
manager: kfile
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 07/03/2018
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: 00a975ae3acb470912c702d8a37717e5c270c9af
ms.sourcegitcommit: 86cb3855e1368e5a74f21fdd71684c78a1f907ac
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/03/2018
ms.locfileid: "37446787"
---
# <a name="connect-with-power-bi"></a>使用 Power BI 進行連接

您在 Azure 中建立一個伺服器，並將表格式模型部署至該伺服器後，您組織中的使用者便可以連線與開始瀏覽資料。 

> [!TIP]
> 請務必使用最新版的 [Power BI Desktop](https://powerbi.microsoft.com/desktop/)。
> 
> 
  
## <a name="connect-in-power-bi-desktop"></a>在 Power BI Desktop 中連線

1. 在 Power BI Desktop 中，按一下 [取得資料] > [Azure] > [Azure Analysis Services 資料庫]。

2. 在 [伺服器] 中，輸入伺服器名稱。 請務必包含完整的 URL，例如 asazure://westcentralus.asazure.windows.net/advworks。

3. 在 [資料庫] 中，如果您知道所要連線表格式模型資料庫或檢視方塊的名稱，請在此貼上。 否則，您可以將此欄位保留空白，然後稍後選取資料庫或檢視方塊。

4. 選取連接選項，然後按 [連接]。 

    同時支援 [即時連接] 和 [匯入] 選項。 不過，建議您使用即時連接，因為匯入模式確實有一些限制；最值得注意的是，在匯入期間，伺服器效能可能會受到影響。 此外，如果要重新整理 Power BI 服務中的模型，只有選擇 [即時連接]，才適用 [允許從 Power BI 存取] 設定。

5. 如果出現提示，請輸入您的登入認證。 

6. 在 [導覽器] 中展開伺服器，然後選取您要連線的模型或檢視方塊，再按一下 [連線]。 按一下某個模型或檢視方塊，即可顯示該檢視的所有物件。

    模型會在 Power BI Desktop 中開啟，其中會在 [報表] 檢視下顯示一個空白報表。 [欄位] 清單會顯示所有非隱藏的模型物件。 連線狀態會顯示在右下角。

## <a name="connect-in-power-bi-service"></a>在 Power BI Desktop 中連線 (服務)

1. 建立與您伺服器上模型即時連線的 Power BI Desktop 檔案。
2. 在 [Power BI](https://powerbi.microsoft.com) 中，按一下 [取得資料] > [檔案]，然後尋找並選取您的 .pbix 檔案。



## <a name="see-also"></a>另請參閱
[連接到 Azure Analysis Services](analysis-services-connect.md)   
[用戶端程式庫](analysis-services-data-providers.md)

