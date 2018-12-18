### <a name="prerequisites"></a>必要條件
* [FTP](https://wikipedia.org/wiki/File_Transfer_Protocol) 帳戶  

您必須先授權邏輯應用程式連線到您的 FTP 帳戶，才能在該邏輯應用程式中使用您的 FTP 帳戶。幸運的是，您可以在「Azure 入口網站」上，從邏輯應用程式內輕鬆完成此操作。  

若要授權邏輯應用程式連線到您的 FTP 帳戶，其步驟如下：  

1. 若要建立與 FTP 的連線，請在邏輯應用程式設計工具中，從下拉式清單中選取 [顯示 Microsoft 受控 API]，然後在搜尋方塊中輸入「FTP」。 選取您要使用的觸發程序或動作：  
   ![FTP 連接的建立步驟](./media/connectors-create-api-ftp/ftp-1.png)  
2. 如果您之前尚未建立任何 FTP 連接，系統會提示您提供 FTP 認證。 這些認證將用來授權邏輯應用程式連線及存取您 FTP 帳戶的資料：  
   ![FTP 連接的建立步驟](./media/connectors-create-api-ftp/ftp-2.png)  
3. 請注意，此時已建立連接，您現可進行邏輯應用程式中的其他步驟：  
   ![FTP 連接的建立步驟](./media/connectors-create-api-ftp/ftp-3.png)  

