1. 登入 [Azure 入口網站][Azure portal]。
2. 在左功能表中選取 [+ 建立資源]。 然後，選取 [整合] > [轉送]。 如果清單中未顯示 [轉送]，請選取右上角的 [查看全部]。 
3. 在 [建立命名空間] 下，輸入命名空間名稱。 系統會立即檢查此名稱是否可用。
4. 在 [訂用帳戶] 方塊中，選擇要在其中建立命名空間的 Azure 訂用帳戶。
5. 在 [[資源群組](../articles/azure-resource-manager/resource-group-portal.md)] 方塊中，選取現有資源群組以便放置命名空間，或是建立新的資源群組。  
6. 在 [位置]中，選取要用來裝載命名空間的國家或地區。
   
    ![建立命名空間][create-namespace]
7. 選取 [建立] 。 系統便會建立並啟用命名空間。 幾分鐘後，系統便會為您的帳戶佈建資源。

### <a name="get-management-credentials"></a>取得管理認證

1. 選取 [所有資源]，然後選取新建立的命名空間名稱。
2. 在 [轉送命名空間] 下，選取 [共用存取原則]。  
3. 在 [共用存取原則] 下，選取 [RootManageSharedAccessKey]。
   
    ![connection-info][connection-info]
4. 在 [原則：RootManageSharedAccessKey] 下，選取 [連接字串 – 主要金鑰] 旁邊的 [複製] 按鈕。 這會將連接字串複製到剪貼簿以供稍後使用。 將此值貼到記事本或一些其他暫存位置。
   
    ![connection-string][connection-string]

5. 重複前一個步驟，複製 [主要金鑰] 的值並貼到暫存位置以供稍後使用。  

<!--Image references-->

[create-namespace]: ./media/relay-create-namespace-portal/create-namespace.png
[connection-info]: ./media/relay-create-namespace-portal/connection-info.png
[connection-string]: ./media/relay-create-namespace-portal/connection-string.png
[Azure portal]: https://portal.azure.com
