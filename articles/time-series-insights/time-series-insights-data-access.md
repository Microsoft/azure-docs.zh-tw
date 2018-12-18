---
title: 設定安全性以存取和管理 Azure 時間序列深入解析 | Microsoft Docs
description: 此文章說明如何設定安全性和權限作為管理存取原則和資料存取原則，以保護 Azure 時間序列深入解析。
ms.service: time-series-insights
services: time-series-insights
author: ashannon7
ms.author: anshan
manager: cshankar
ms.reviewer: v-mamcge, jasonh, kfile, anshan
ms.workload: big-data
ms.topic: conceptual
ms.date: 11/15/2017
ms.openlocfilehash: c9bddf8e20524433b31793e277efd954a5d1320e
ms.sourcegitcommit: 42405ab963df3101ee2a9b26e54240ffa689f140
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47423369"
---
# <a name="grant-data-access-to-a-time-series-insights-environment-using-azure-portal"></a>使用 Azure 入口網站授與時間序列深入解析環境的資料存取權

此文章討論兩種類型的時間序列深入解析存取原則。

## <a name="video"></a>影片： 

### <a name="in-this-video-we-cover-creating-and-managing-access-policies-within-time-series-insights-br"></a>在此影片中，我們說明如何建立及管理時間序列深入解析內的存取原則。 </br>

> [!VIDEO https://www.youtube.com/embed/5zTIdyHMJW8]

時間序列深入解析環境有兩種獨立的存取原則︰

* 管理存取原則
* 資料存取原則

這兩種原則可將特定環境的各種權限授與給 Azure Active Directory 主體 (使用者和應用程式)。 主體 (使用者和應用程式) 必須屬於與含有此環境之訂用帳戶相關聯的 Active Directory (也稱為 Azure 租用戶)。

管理存取原則可授與環境設定相關權限，例如
*   建立和刪除環境、事件來源、參考資料集，以及
*   管理資料存取原則。

資料存取原則可授與下列權限：發出資料查詢、在環境中操作參考資料，以及共用與環境相關聯的已儲存查詢和檢視方塊。

這兩種原則能夠清楚區隔環境管理存取權與環境內資料存取權。 例如，可以設定環境，以便移除環境擁有者/建立者的資料存取權。 此外，獲准讀取環境中資料的使用者與服務，可能無法存取環境的組態。

## <a name="grant-data-access"></a>授與資料存取
依照下列步驟來授與使用者主體的資料存取權︰

1. 登入 [Azure 入口網站](https://portal.azure.com)。

2. 找到您的時間序列深入解析環境。 在**搜尋**方塊中輸入**時間序列深入解析**。 選取搜尋結果中的 [時間序列環境]。 

3. 從清單中選取時間序列深入解析環境。
   
4. 選取 [資料存取原則]，然後選取 [+新增]。
  ![管時間序列深入解析來源 - 環境](media/data-access/getstarted-grant-data-access1.png)

5. 選取 [選取使用者]。  搜尋使用者名稱或電子郵件地址，以找出您想要新增的使用者。 按一下 [選取] 以確認選取項目。 

   ![管理時間序列深入解析來源 - 新增](media/data-access/getstarted-grant-data-access2.png)

6. 選取 [選取角色]。 為使用者選擇適當的存取角色：
   - 如果您要允許使用者變更參考資料，並與其他環境使用者共用已儲存的查詢和觀點，請選取 [參與者]。 
   - 否則選取 [讀取者]，允許使用者查詢環境中的資料並且在環境中儲存個人 (非共用) 查詢。

   選取 [確定] 以確認角色選擇。

   ![管理時間序列深入解析來源 - 選取使用者](media/data-access/getstarted-grant-data-access3.png)

8. 在 [選取使用者角色] 頁面中選取 [確定]。

   ![管理時間序列深入解析來源 - 選取角色](media/data-access/getstarted-grant-data-access4.png)

9. [資料存取原則] 頁面會列出使用者和每個使用者的角色。

   ![管理時間序列深入解析來源 - 結果](media/data-access/getstarted-grant-data-access5.png)

## <a name="provide-guest-access-to-a-user-from-another-aad-tenant"></a>提供來賓存取權給來自另一個 AAD 租用戶的使用者

「來賓」不是管理角色；這個詞彙指的是從一個租用戶邀請到另一個租用戶的帳戶。 在來賓帳戶被邀請到租用戶的目錄之後，它能擁有套用到其上的存取控制權限 (就像其他帳戶一樣)，不論是使用 [存取控制 (IAM)] 刀鋒視窗授與對 TSI 環境的管理存取權，或透過 [資料存取原則] 刀鋒視窗授與對環境中資料的存取權。 如需有關 AAD 租用戶來賓存取權的詳細資訊，請參閱此[文件](https://docs.microsoft.com/azure/active-directory/b2b/add-users-administrator)。

依照這些步驟將對「時間序列深入解析」環境的存取權授與來自另一個租用戶的 AAD 使用者：

1. 登入 [Azure 入口網站](https://portal.azure.com)。

2. 找到您的時間序列深入解析環境。 在**搜尋**方塊中輸入**時間序列深入解析**。 選取搜尋結果中的 [時間序列環境]。

3. 從清單中選取時間序列深入解析環境。

4. 選取 [資料存取原則]，然後選取 [+邀請]。

    ![管理時間序列深入解析來源 - 邀請使用者](media/data-access/getstarted-grant-data-access6.png)

5. 提供您要邀請之使用者的電子郵件。 請注意，這必須是與 AAD 關聯的電子郵件。 (選擇性) 您可以在邀請中包括個人訊息。

    ![管理時間序列深入解析來源 - 選取使用者](media/data-access/getstarted-grant-data-access7.png)

6. 您應該會看到確認泡泡訊息出現在畫面上。

    ![管理時間序列深入解析來源 - 確認使用者](media/data-access/getstarted-grant-data-access8.png)

7. 選取 [選取使用者]。 搜尋您剛邀請之來賓使用者的電子郵件地址，以尋找您要新增的使用者。 按一下 [選取] 以確認選取項目。
  
    ![管理時間序列深入解析來源 - 確認使用者](media/data-access/getstarted-grant-data-access9.png)

8. 選取 [選取角色]。 為來賓使用者選擇適當的存取角色：

    * 如果您要允許使用者變更參考資料，並與其他環境使用者共用已儲存的查詢和觀點，請選取 [參與者]。

    * 否則選取 [讀取者]，允許使用者查詢環境中的資料並且在環境中儲存個人 (非共用) 查詢。

    選取 [確定] 以確認角色選擇。

    ![管理時間序列深入解析來源 - 選取角色](media/data-access/getstarted-grant-data-access10.png)

9. 在 [選取使用者角色] 頁面中選取 [確定]。

10. [資料存取原則] 頁面現在會列出來賓使用者與每個來賓使用者的角色。

    ![管理時間序列深入解析來源 - 確認角色](media/data-access/getstarted-grant-data-access11.png)

11. 現在來賓使用者將必須執行特定步驟，以存取位於您剛將其邀請到其中之 Azure 租用戶的環境。 首先，他們必須接受您剛傳送給他們的邀請。 此邀請是透過電子郵件傳送到您在步驟 5 邀請的電子郵件地址。 他們必須按一下 [開始使用] 以接受。

    ![管理時間序列深入解析來源 - 邀請使用者](media/data-access/getstarted-grant-data-access12.png)

12. 接著，來賓使用者將必須接受與系統管理員組織關聯的權限。

    ![管理時間序列深入解析來源 - 接受權限](media/data-access/getstarted-grant-data-access13.png)

13. 當來賓使用者登入您邀請的電子郵件地址並接受邀請之後，他們將被導向 insights.azure.com。 到達該處之後，他們將必須按一下畫面右上角其電子郵件旁的頭像。 

    ![管理時間序列深入解析來源 - 接受權限](media/data-access/getstarted-grant-data-access14.png)

14. 接著，來賓使用者將必須從目錄下拉式清單功能表選取您的 Azure 租用戶。 這是您將他們邀請到其中的租用戶。 

    ![管理時間序列深入解析來源 - 接受權限](media/data-access/getstarted-grant-data-access15.png)

15. 最後，當來賓使用者選取您的租用戶之後，他們將會看到您剛為其提供存取權的時間序列深入解析環境。 他們現在應該擁有與您在步驟 8 中提供之角色關聯的所有能力。

## <a name="next-steps"></a>後續步驟
* 了解[如何將事件中樞的事件來源新增至 Azure 時間序列深入解析環境](time-series-insights-how-to-add-an-event-source-eventhub.md)。
* [將事件傳送](time-series-insights-send-events.md)到事件來源。
* 在[時間序列深入解析總管](https://insights.timeseries.azure.com)中檢視您的環境。
