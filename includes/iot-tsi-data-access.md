---
title: 包含檔案
description: 包含檔案
services: time-series-insights
author: ashannon7
ms.service: time-series-insights
ms.topic: include
ms.date: 08/20/2018
ms.author: anshan
ms.custom: include file
ms.openlocfilehash: fb45ea02f365cf4e7b394e249f9b91a784e5469f
ms.sourcegitcommit: ce526d13cd826b6f3e2d80558ea2e289d034d48f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46369860"
---
## <a name="grant-data-access"></a>授與資料存取

依照下列步驟來授與使用者主體的資料存取權︰

1. 登入 [Azure 入口網站](https://portal.azure.com)。

2. 找到您的時間序列深入解析環境。 在**搜尋**方塊中輸入**時間序列深入解析**。 選取搜尋結果中的 [時間序列環境]。 

3. 從清單中選取 Time Series Insights 環境。

4. 選取 [資料存取原則]，然後選取 [+新增]。
    ![管時間序列深入解析來源 - 環境](media/iot-tsi-data-access/getstarted-grant-data-access1.png)

5. 選取 [選取使用者]。  搜尋使用者名稱或電子郵件地址，以找出您想要新增的使用者。 按一下 [選取] 以確認選取項目。 

    ![管理 Time Series Insights 來源 - 新增](media/iot-tsi-data-access/getstarted-grant-data-access2.png)

6. 選取 [選取角色]。 為使用者選擇適當的存取角色：
    - 如果您要允許使用者變更參考資料，並與其他環境使用者共用已儲存的查詢和觀點，請選取 [參與者]。 
    - 否則選取 [讀取者]，允許使用者查詢環境中的資料並且在環境中儲存個人 (非共用) 查詢。

    選取 [確定] 以確認角色選擇。

    ![管理 Time Series Insights 來源 - 選取使用者](media/iot-tsi-data-access/getstarted-grant-data-access3.png)

7. 在 [選取使用者角色] 頁面中選取 [確定]。

    ![管理 Time Series Insights 來源 - 選取角色](media/iot-tsi-data-access/getstarted-grant-data-access4.png)

8. [資料存取原則] 頁面會列出使用者和每個使用者的角色。

    ![管理 Time Series Insights 來源 - 結果](media/iot-tsi-data-access/getstarted-grant-data-access5.png)