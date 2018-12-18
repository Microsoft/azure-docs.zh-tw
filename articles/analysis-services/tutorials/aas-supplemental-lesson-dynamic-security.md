---
title: Azure Analysis Services 教學課程補充課程︰動態安全性 | Microsoft Docs
description: 說明如何在 Azure Analysis Services 教學課程中使用資料列篩選條件，進而使用動態安全性。
author: minewiskan
manager: kfile
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 07/03/2018
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: f7e5999b2d64c4a9b409fac81841d15f7c6aa5e7
ms.sourcegitcommit: 86cb3855e1368e5a74f21fdd71684c78a1f907ac
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/03/2018
ms.locfileid: "37443659"
---
# <a name="supplemental-lesson---dynamic-security"></a>補充課程 - 動態安全性

在此補充課程中，您可以建立其他角色來實作動態安全性。 動態安全性會根據目前登入使用者的使用者名稱或登入識別碼來提供資料列層級安全性。 
  
若要實作動態安全性，您可以將資料表新增至模型 (其中包含可連線至該模型及瀏覽模型物件和資料之使用者的使用者名稱)。 您使用本教學課程建立的模型位於 Adventure Works 的環境中；不過，若要完成本課程，您必須新增包含您自有網域使用者的資料表。 您不需要所新增使用者名稱的密碼。 若要建立 EmployeeSecurity 資料表 (包含您自有網域中的少量使用者範例)，您可使用 [貼上] 功能中，貼上 Excel 試算表中的員工資料。 在真實的案例中，包含使用者名稱的資料表通常會是實際資料庫中做為資料來源的資料表；例如，真實的 DimEmployee 資料表。  
  
若要實作動態安全性，您可以使用兩個 DAX 函式︰[USERNAME 函式 (DAX)](http://msdn.microsoft.com/22dddc4b-1648-4c89-8c93-f1151162b93f) 和 [LOOKUPVALUE 函式 (DAX)](http://msdn.microsoft.com/73a51c4d-131c-4c33-a139-b1342d10caab)。 套用於資料列篩選公式的這些函式，會以新的角色定義。 使用 LOOKUPVALUE 函式，此公式會指定 EmployeeSecurity 資料表中的值。 此公式接著將該值傳遞至 USERNAME 函式，其指定已登入使用者的使用者名稱屬於此角色。 使用者接著可以只瀏覽角色的資料列篩選條件所指定的資料。 在此案例中，您指定銷售員工只能瀏覽其所屬銷售地區的網際網路銷售資料。  
  
這麼一來，便會識別此 Adventure Works 表格式模型案例獨有，但不一定適用於真實案例的工作。 每個工作都包含描述工作目的的其他資訊。  
  
這堂課的預估完成時間：**30 分鐘**  
  
## <a name="prerequisites"></a>先決條件  
此補充課程主題是表格式模型教學課程的一部分，請依序完成。 在此補充課程中執行工作之前，您必須已完成所有前面的課程。  
  
## <a name="add-the-dimsalesterritory-table-to-the-aw-internet-sales-tabular-model-project"></a>將 DimSalesTerritory 資料表新增至 AW 網際網路銷售表格式模型專案  
若要實作此 Adventure Works 案例的動態安全性，您必須將額外兩個資料表新增至您的模型。 您新增的第一個資料表是來自相同 AdventureWorksDW 資料庫的 DimSalesTerritory (做為銷售地區)。 您稍後會將資料列篩選條件套用到 SalesTerritory 資料表，以定義已登入使用者可以瀏覽的特定資料。  
  
#### <a name="to-add-the-dimsalesterritory-table"></a>新增 DimSalesTerritory 資料表  
  
1.  在 [表格式模型總管] > [資料來源] 中，以滑鼠右鍵按一下您的連線，然後按一下 [匯入新資料表]。  

    如果出現 [模擬認證] 對話方塊，請輸入您在「第 2 課︰新增資料」中使用的模擬認證。
  
2.  在 [導覽器] 中，選取 [DimSalesTerritory] 資料表，然後按一下 [確定]。    
  
3.  在 [查詢編輯器] 中，按一下 [DimSalesTerritory] 查詢，然後移除 [SalesTerritoryAlternateKey] 資料行。  
  
7.  按一下 [匯入] 。  
  
    新資料表會新增至模型工作區。 來源 DimSalesTerritory 資料表中的物件和資料則會匯入 AW 網際網路銷售表格式模型。  
  
9. 成功匯入資料表之後，請按一下 [關閉]。  

## <a name="add-a-table-with-user-name-data"></a>新增具有使用者名稱資料的資料表  
AdventureWorksDW 範例資料庫中的 DimEmployee 資料表包含來自 AdventureWorks 網域的使用者。 這些使用者名稱不存在於您自己的環境中。 您必須在您的模型中建立資料表，其中包含您組織中實際使用者 (至少三個) 的小型範例。 您接著會將這些使用者當作成員新增至新角色。 您不需要範例使用者名稱的密碼，但您需要您自有網域中的實際 Windows 使用者名稱。  
  
#### <a name="to-add-an-employeesecurity-table"></a>新增 EmployeeSecurity 資料表  
  
1.  開啟 Microsoft Excel，並建立工作表。  
  
2.  複製下列資料表 (包括標頭資料列)，然後將它貼到工作表中。  

    ```
      |EmployeeId|SalesTerritoryId|FirstName|LastName|LoginId|  
      |---------------|----------------------|--------------|-------------|------------|  
      |1|2|<user first name>|<user last name>|\<domain\username>|  
      |1|3|<user first name>|<user last name>|\<domain\username>|  
      |2|4|<user first name>|<user last name>|\<domain\username>|  
      |3|5|<user first name>|<user last name>|\<domain\username>|  
    ```

3.  以您組織中三個使用者的名稱和登入識別碼，取代名字、姓氏和網域\使用者名稱。 將相同的使用者 (例如 EmployeeId 1) 放在前兩個資料列上，顯示此使用者屬於一個以上的銷售領域。 讓 [EmployeeId] 和 [SalesTerritoryId] 欄位維持原樣。  
  
4.  將工作表儲存為 [SampleEmployee]。  
  
5.  在工作表中，選取具有員工資料的所有儲存格 (包括標頭)，然後以滑鼠右鍵按一下所選取的資料，然後按一下 [複製]。  
  
6.  在 SSDT 中，按一下 [編輯] 功能表，然後按一下 [貼上]。  
  
    如果 [貼上] 呈現灰色，按一下模型設計工具視窗中任何資料表中的任何資料行，然後再試一次。  
  
7.  在 [貼上預覽] 對話方塊的 [資料表名稱] 中，輸入 **EmployeeSecurity**。  
  
8.  在 [要貼上的資料] 中，確認資料包含 SampleEmployee 工作表中的所有使用者資料和標頭。  
  
9. 確認已核取 [使用第一個資料列做為資料行標頭]，然後按一下 [確定]。  
  
    隨即建立名為 EmployeeSecurity 的新資料表，其中包含從 SampleEmployee 工作表複製而來的員工資料。  
  
## <a name="create-relationships-between-factinternetsales-dimgeography-and-dimsalesterritory-table"></a>建立 FactInternetSales、DimGeography 和 DimSalesTerritory 資料表之間的關聯性  
FactInternetSales、DimGeography 和 DimSalesTerritory 資料表全都包含常用的資料行 (SalesTerritoryId)。 DimSalesTerritory 資料表中的 SalesTerritoryId 資料行包含每個銷售地區內不同識別碼值的值。  
  
#### <a name="to-create-relationships-between-the-factinternetsales-dimgeography-and-the-dimsalesterritory-table"></a>建立 FactInternetSales、DimGeography 和 DimSalesTerritory 資料表之間的關聯性  
  
1.  在 [圖表檢視] 中，於 [DimGeography] 資料表中按一下並按住 [SalesTerritoryId] 資料行，然後將游標拖曳至 [DimSalesTerritory] 資料表中的 [SalesTerritoryId] 資料行，然後放開。  
  
2.  在 [FactInternetSales] 資料表中，按一下並按住 [SalesTerritoryId] 資料行，然後將游標拖曳至 [DimSalesTerritory] 資料表中的 [SalesTerritoryId] 資料行，然後放開。  
  
    請注意，此關聯性的 [作用中] 屬性為 False，亦即非作用中。 FactInternetSales 資料表已經有另一個作用中關聯性。  
  
## <a name="hide-the-employeesecurity-table-from-client-applications"></a>在用戶端應用程式中隱藏 EmployeeSecurity 資料表  
在此工作中，您會隱藏 EmployeeSecurity 資料表，使其不會出現在用戶端應用程式的欄位清單中。 請注意，隱藏資料表並不會保護它。 使用者仍可查詢 EmployeeSecurity 資料表資料，但前提是他們知道方式。 若要保護 EmployeeSecurity 資料表資料，防止使用者查詢其中的任何資料，您可以在稍後的工作中套用篩選條件。  
  
#### <a name="to-hide-the-employeesecurity-table-from-client-applications"></a>在用戶端應用程式中隱藏 EmployeeSecurity 資料表  
  
-   在模型設計工具的 [圖表檢視] 中，以滑鼠右鍵按一下 [員工] 資料表標題，，然後按一下 [在用戶端工具中隱藏]。  
  
## <a name="create-a-sales-employees-by-territory-user-role"></a>建立「銷售地區員工」使用者角色  
在此工作中，您會建立使用者角色。 此角色包含一個資料列篩選條件，以定義使用者可見的 DimSalesTerritory 資料表資料列。 此篩選條件接著會以一對多關聯性方向套用至 DimSalesTerritory 相關的所有其他資料表。 您也可以套用一個篩選條件，以防止身為此角色成員的任何使用者查詢整個 EmployeeSecurity 資料表。  
  
> [!NOTE]  
> 您在這堂課中建立的「銷售地區員工」角色會限制成員只能瀏覽 (或查詢) 其所屬銷售地區的銷售資料。 如果您將使用者新增為「銷售地區員工」角色的成員，而該使用者同時也是在[第 11 課︰建立角色](../tutorials/aas-lesson-11-create-roles.md)中建立之角色的成員，您會取得權限組合。 當使用者是多個角色的成員時，則針對每個角色定義的權限和資料列篩選條件會累計。 也就是說，使用者具有角色組合所決定的較高權限。  
  
#### <a name="to-create-a-sales-employees-by-territory-user-role"></a>建立「銷售地區員工」使用者角色  
  
1.  在 SSDT 中，按一下 [模型] 功能表，然後按一下 [角色]。  
  
2.  在 [角色管理員] 中，按一下 [新增]。  
  
    清單中會新增 [無] 權限的新角色。  
  
3.  按一下新角色，然後在 [名稱] 資料行中，將角色重新命名為 [銷售地區員工]。  
  
4.  在 [權限] 資料行中，按一下下拉式清單，然後選取 [讀取] 權限。  
  
5.  按一下 [成員] 索引標籤，然後按一下 [新增]。  
  
6.  在 [選取使用者或群組] 對話方塊的 [輸入要選取的物件名稱] 中，輸入您建立 EmployeeSecurity 資料表時使用的第一個範例使用者名稱。 按一下 [檢查名稱] 來驗證使用者名稱是否有效，然後按一下 [確定]。  
  
    重複此步驟，並新增您建立 EmployeeSecurity 資料表時使用的其他範例使用者名稱。  
  
7.  按一下 [資料列篩選條件] 索引標籤。  
  
8.  對於 **EmployeeSecurity** 資料表，在 [DAX 篩選條件] 資料行中，輸入下列公式︰  
  
    ```
      =FALSE()  
    ```
  
    此公式指定所有資料行都會解析為 false 布林值條件。 「銷售地區員工」使用者角色的成員查詢無法查詢 EmployeeSecurity 資料表的任何資料行。  
  
9. 對於 **DimSalesTerritory** 資料表，輸入下列公式︰  

    ```  
    ='Sales Territory'[Sales Territory Id]=LOOKUPVALUE('Employee Security'[Sales Territory Id], 
      'Employee Security'[Login Id], USERNAME(), 
      'Employee Security'[Sales Territory Id], 
      'Sales Territory'[Sales Territory Id]) 
    ```
  
    在此公式中，LOOKUPVALUE 函式會傳回 DimEmployeeSecurity[SalesTerritoryId] 資料行的所有值，其中 EmployeeSecurity[LoginId] 與目前已登入的 Windows 使用者名稱相同，而 EmployeeSecurity[SalesTerritoryId] 與 DimSalesTerritory[SalesTerritoryId] 相同。  
  
    LOOKUPVALUE 所傳回的銷售地區 ID 集合會接著用來限制 DimSalesTerritory 資料表中顯示的資料列。 只會顯示資料列的 SalesTerritoryID 在 LOOKUPVALUE 函式所傳回之 ID 集合中的資料列。  
  
10. 在 [角色管理員] 中，按一下 [確定]。  
  
## <a name="test-the-sales-employees-by-territory-user-role"></a>測試「銷售地區員工」使用者角色  
在此工作中，您會使用 SSDT 中的 [使用 Excel 分析] 功能來測試「銷售地區員工」使用者角色的功效。 您可指定您新增到 EmployeeSecurity 資料表並成為角色成員的其中一個使用者名稱。 此使用者名稱接著會做為在 Excel 與模型之間建立之連線中的有效使用者名稱。  
  
#### <a name="to-test-the-sales-employees-by-territory-user-role"></a>測試「銷售地區員工」使用者角色  
  
1.  在 SSDT 中，按一下 [模型] 功能表，然後按一下 [使用 Excel 分析]。  
  
2.  在 [使用 Excel 分析] 對話方塊的 [指定要用來連線至模型的使用者名稱或角色] 中，選取 [其他 Windows 使用者]，然後按一下 [瀏覽]。  
  
3.  在 [選取使用者或群組] 對話方塊的 [輸入要選取的物件名稱] 中，輸入您包含在 EmployeeSecurity 資料表的使用者名稱，然後按一下 [檢查名稱]。  
  
4.  按一下 [確定] 以關閉 [選取使用者或群組] 對話方塊，然後按一下 [確定] 以關閉 [使用 Excel 分析] 對話方塊。  
  
    Excel 會開啟新的活頁簿。 將會自動建立樞紐分析表。 樞紐分析表欄位清單包含您的新模型可用的大部分資料欄位。  
  
    請注意，樞紐分析表欄位清單不會顯示 EmployeeSecurity 資料表。 您於先前工作中在用戶端工具中隱藏此資料表。  
  
5.  在 **[欄位]** 清單的 **∑ 網際網路銷售** \(量值) 中，選取 **[InternetTotalSales]** 量值。 此量值會輸入於 [值] 欄位中。  
  
6.  從 **DimSalesTerritory** 資料表選取 **SalesTerritoryId** 資料行。 此資料行會輸入於 [資料列標籤] 欄位中。  
  
    請注意，只會針對您使用的有效使用者名稱所屬的區域顯示網際網路銷售數字。 如果您選取另一個資料行 (例如，DimGeography 資料表中的 [城市]) 作為 [資料列標籤] 欄位，則只會顯示有效使用者所屬銷售地區中的城市。  
  
    此使用者無法瀏覽或查詢其所需地區以外的任何網際網路銷售資料。 這項限制是因為以「銷售地區員工」使用者角色針對 DimSalesTerritory 資料表定義的資料列篩選條件，會保護其他銷售地區的所有相關資料。  
  
## <a name="see-also"></a>另請參閱  
[USERNAME 函式 (DAX)](https://msdn.microsoft.com/library/hh230954.aspx)  
[LOOKUPVALUE 函式 (DAX)](https://msdn.microsoft.com/library/gg492170.aspx)  
[CUSTOMDATA 函式 (DAX)](https://msdn.microsoft.com/library/hh213140.aspx)  