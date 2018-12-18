---
title: 使用 Azure Cosmos DB 進行 Java 應用程式開發教學課程 | Microsoft Docs
description: 本 Java Web 應用程式教學課程示範如何使用 Azure Cosmos DB 和 SQL API，來儲存和存取 Azure 網站上所託管的 Java 應用程式資料。
keywords: 應用程式開發, 資料庫教學課程, java 應用程式, java web 應用程式教學課程, azure, Microsoft azure
services: cosmos-db
author: tknandu
manager: kfile
ms.service: cosmos-db
ms.component: cosmosdb-sql
ms.devlang: java
ms.topic: tutorial
ms.date: 08/22/2017
ms.author: ramkris
ms.openlocfilehash: ef12f7e06462d89e34c2475ea34929b96bbf0432
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46954886"
---
# <a name="build-a-java-web-application-using-azure-cosmos-db-and-the-sql-api"></a>使用 Azure Cosmos DB 和 SQL API 來建置 Java Web 應用程式

> [!div class="op_single_selector"]
> * [.NET](sql-api-dotnet-application.md)
> * [Java](sql-api-java-application.md)
> * [Node.js](sql-api-nodejs-application.md)
> * [Python](sql-api-python-application.md)
> * [Xamarin](mobile-apps-with-xamarin.md)
> 

本 Java Web 應用程式教學課程示範如何使用 [Microsoft Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/) 服務，來儲存和存取 Azure App Service Web Apps 上所託管的 Java 應用程式資料。 在本文中，您將了解：

* 如何在 Eclipse 中建置基本的 JavaServer Pages (JSP) 應用程式。
* 如何透過 [Azure Cosmos DB Java SDK](https://github.com/Azure/azure-documentdb-java)，使用 Azure Cosmos DB 服務。

本 Java 應用程式教學課程會示範如何建立以 Web 為基礎的工作管理應用程式，方便您建立、抓取以及將工作標示為完成，如下圖所示。 在 Azure Cosmos DB 中，[待辦事項] 清單中的每項工作都會以 JSON 文件的形式儲存。

![我的待辦事項清單 Java 應用程式](./media/sql-api-java-application/image1.png)

> [!TIP]
> 本應用程式開發教學課程假設您先前已有使用 Java 的經驗。 如果您不熟悉 Java 或[必備工具](#Prerequisites)，我們建議您從 GitHub 下載完整的[待辦事項](https://github.com/Azure-Samples/documentdb-java-todo-app)專案，並使用[本文結尾的指示](#GetProject)開始建置。 建置完成後，您可以檢閱文件，以加深對專案內容中程式碼的了解。  
> 
> 

## <a id="Prerequisites"></a>針對此 Java Web 應用程式教學課程的必要條件
開始進行本應用程式開發教學課程之前，您必須具備下列條件：

*  如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。 

  [!INCLUDE [cosmos-db-emulator-docdb-api](../../includes/cosmos-db-emulator-docdb-api.md)]

* [Java Development Kit (JDK) 7+](http://www.oracle.com/technetwork/java/javase/downloads/index.html)。
* [Eclipse IDE for Java EE Developers。](http://www.eclipse.org/downloads/packages/release/luna/sr1/eclipse-ide-java-ee-developers)
* [已啟用某個 Java Runtime Environment (例如 Tomcat 或 Jetty) 的 Azure 網站。](../app-service/app-service-web-get-started-java.md)

如果您是第一次安裝這些工具，coreservlets.com 提供了安裝程序的的逐步解說，請參閱其 [教學課程：安裝 TomCat7 並與 Eclipse 搭配使用](http://www.coreservlets.com/Apache-Tomcat-Tutorial/tomcat-7-with-eclipse.html) 一文中的 [快速入門] 區段。

## <a id="CreateDB"></a>步驟 1：建立 Azure Cosmos DB 帳戶
我們將從建立 Azure Cosmos DB 帳戶開始著手。 如果您已經擁有帳戶，或如果您正在使用 Azure Cosmos DB 模擬器來進行本教學課程，可以跳到[步驟 2：建立 Java JSP 應用程式](#CreateJSP)。

[!INCLUDE [create-dbaccount](../../includes/cosmos-db-create-dbaccount.md)]

[!INCLUDE [keys](../../includes/cosmos-db-keys.md)]

## <a id="CreateJSP"></a>步驟 2：建立 Java JSP 應用程式
建立 JSP 應用程式：

1. 首先，我們將從建立 Java 專案開始。 啟動 Eclipse，依序按一下 [檔案]、[新增] 和 [動態 Web 專案]。 如果您在可用專案中沒有看到 [動態 Web 專案]，請執行下列動作：依序按一下 [檔案]、[新增]、[專案]，展開 [Web]，按一下 [動態 Web 專案]，然後按 [下一步]。
   
    ![JSP Java 應用程式開發](./media/sql-api-java-application/image10.png)
2. 在 [專案名稱] 方塊中輸入專案名稱，然後在 [目標執行階段] 下拉式選單中，選擇性地選取值 (例如 Apache Tomcat v7.0)，然後按一下 [完成]。 選取目標執行階段可讓您透過 Eclipse 在本機執行專案。
3. 在 Eclipse 的 [專案總管] 檢視中，展開您的專案。 在 [WebContent] 上按一下滑鼠右鍵、按一下 [新增]，然後按一下 [JSP 檔案]。
4. 在 [新增 JSP 檔案] 對話方塊中，將檔案命名為 **index.jsp**。 將上層資料夾保持為 **WebContent**，如下圖所示，然後按 [下一步]。
   
    ![建立新的 JSP 檔案 - Java Web 應用程式教學課程](./media/sql-api-java-application/image11.png)
5. 在 [選取 JSP 範本] 對話方塊中，基於本教學課程的目的，選取 [新增 JSP 檔案 (html)]，然後按一下 [完成]。
6. 在 Eclipse 中開啟 index.jsp 檔案時，請加入文字以顯示 **Hello World!**。 (在現有的 <body> 元素內加入)。 已更新的 <body> 內容看起來應該與下列程式碼類似：
   
        <body>
            <% out.println("Hello World!"); %>
        </body>
7. 儲存 index.jsp 檔案。
8. 如果您在步驟 2 中已設定目標執行階段，就可以依序按一下 [專案] 和 [執行]，即可在本機執行您的 JSP 應用程式：
   
    ![Hello World – Java 應用程式教學課程](./media/sql-api-java-application/image12.png)

## <a id="InstallSDK"></a>步驟 3：安裝 SQL Java SDK
導入 SQL Java SDK 及其相依性的最簡單方式就是透過 [Apache Maven](http://maven.apache.org/)。

若要這樣做，您必須完成下列步驟以將專案轉換成 maven 專案：

1. 在 [專案總管] 中，以滑鼠右鍵按一下您的專案，按一下 [設定]，然後按一下 [轉換成 Maven 專案]。
2. 在 [建立新的 POM] 視窗中，接受預設值，然後按一下 [完成]。
3. 在 [專案總管] 中，開啟 pom.xml 檔案。
4. 在 [相依性] 窗格的 [相依性] 索引標籤中，按一下 [新增]。
5. 在 [選取相依性]  視窗中，執行下列動作：
   
   * 在 [群組識別碼] 方塊中，輸入 com.microsoft.azure。
   * 在 [構件識別碼] 方塊中，輸入 azure-documentdb。
   * 在 [版本] 方塊中，輸入 1.5.1。
     
   ![安裝 SQL Java 應用程式 SDK](./media/sql-api-java-application/image13.png)
     
   * 或透過文字編輯器，將群組識別碼和構件識別碼的相依性 XML 直接新增至 pom.xml：
     
        <dependency> <groupId>com.microsoft.azure</groupId> <artifactId>azure-documentdb</artifactId> <version>1.9.1</version> </dependency>
6. 按一下 [確定]，Maven 便會開始安裝 SQL Java SDK。
7. 儲存 pom.xml 檔案。

## <a id="UseService"></a>步驟 4：在 Java 應用程式中使用 Azure Cosmos DB 服務
1. 首先，讓我們在 TodoItem.java 中定義 TodoItem 物件：
   
        @Data
        @Builder
        public class TodoItem {
            private String category;
            private boolean complete;
            private String id;
            private String name;
        }
   
    在此專案中，您會使用 [Project Lombok](http://projectlombok.org/) 來產生建構函式、getter、setter 及產生器。 或者，您也可以手動撰寫此程式碼，或讓 IDE 產生它。
2. 若要叫用 Azure Cosmos DB 服務，您必須將新的 **DocumentClient**具現化。 一般而言，最好是重複使用 **DocumentClient** ，而不要針對每個後續要求建構新的用戶端。 我們可以將用戶端包裝在 **DocumentClientFactory**中以重複使用用戶端。 您必須在 DocumentClientFactory.java 中，貼上您在[步驟 1](#CreateDB) 中儲存到剪貼簿的 URI 和主要金鑰值。 將 [YOUR\_ENDPOINT\_HERE] 以您的 URI 取代，並將 [YOUR\_KEY\_HERE] 以您的主要金鑰取代。
   
        private static final String HOST = "[YOUR_ENDPOINT_HERE]";
        private static final String MASTER_KEY = "[YOUR_KEY_HERE]";
   
        private static DocumentClient documentClient = new DocumentClient(HOST, MASTER_KEY,
                        ConnectionPolicy.GetDefault(), ConsistencyLevel.Session);
   
        public static DocumentClient getDocumentClient() {
            return documentClient;
        }
3. 現在，讓我們建立「資料存取物件」(DAO)，以將我們的待辦事項提取保存至 Azure Cosmos DB。
   
    為了將 ToDo 項目儲存至集合，用戶端必須知道要保存至哪個資料庫和集合 (會被自我連結參照)。 一般而言，最好是儘可能快取資料庫和集合，以避免對資料庫進行額外的來回存取。
   
    下列程式碼說明如何抓取我們的資料庫和集合 (如果存在)，或建立一個新的資料庫或集合 (如果不存在)：
   
        public class DocDbDao implements TodoDao {
            // The name of our database.
            private static final String DATABASE_ID = "TodoDB";
   
            // The name of our collection.
            private static final String COLLECTION_ID = "TodoCollection";
   
            // The Azure Cosmos DB Client
            private static DocumentClient documentClient = DocumentClientFactory
                    .getDocumentClient();
   
            // Cache for the database object, so we don't have to query for it to
            // retrieve self links.
            private static Database databaseCache;
   
            // Cache for the collection object, so we don't have to query for it to
            // retrieve self links.
            private static DocumentCollection collectionCache;
   
            private Database getTodoDatabase() {
                if (databaseCache == null) {
                    // Get the database if it exists
                    List<Database> databaseList = documentClient
                            .queryDatabases(
                                    "SELECT * FROM root r WHERE r.id='" + DATABASE_ID
                                            + "'", null).getQueryIterable().toList();
   
                    if (databaseList.size() > 0) {
                        // Cache the database object so we won't have to query for it
                        // later to retrieve the selfLink.
                        databaseCache = databaseList.get(0);
                    } else {
                        // Create the database if it doesn't exist.
                        try {
                            Database databaseDefinition = new Database();
                            databaseDefinition.setId(DATABASE_ID);
   
                            databaseCache = documentClient.createDatabase(
                                    databaseDefinition, null).getResource();
                        } catch (DocumentClientException e) {
                            // TODO: Something has gone terribly wrong - the app wasn't
                            // able to query or create the collection.
                            // Verify your connection, endpoint, and key.
                            e.printStackTrace();
                        }
                    }
                }
   
                return databaseCache;
            }
   
            private DocumentCollection getTodoCollection() {
                if (collectionCache == null) {
                    // Get the collection if it exists.
                    List<DocumentCollection> collectionList = documentClient
                            .queryCollections(
                                    getTodoDatabase().getSelfLink(),
                                    "SELECT * FROM root r WHERE r.id='" + COLLECTION_ID
                                            + "'", null).getQueryIterable().toList();
   
                    if (collectionList.size() > 0) {
                        // Cache the collection object so we won't have to query for it
                        // later to retrieve the selfLink.
                        collectionCache = collectionList.get(0);
                    } else {
                        // Create the collection if it doesn't exist.
                        try {
                            DocumentCollection collectionDefinition = new DocumentCollection();
                            collectionDefinition.setId(COLLECTION_ID);
   
                            collectionCache = documentClient.createCollection(
                                    getTodoDatabase().getSelfLink(),
                                    collectionDefinition, null).getResource();
                        } catch (DocumentClientException e) {
                            // TODO: Something has gone terribly wrong - the app wasn't
                            // able to query or create the collection.
                            // Verify your connection, endpoint, and key.
                            e.printStackTrace();
                        }
                    }
                }
   
                return collectionCache;
            }
        }
4. 下一步是撰寫一些程式碼以將 TodoItems 保存至集合。 在此範例中，我們將使用 [Gson](https://code.google.com/p/google-gson/) 將 TodoItem Plain Old Java Objects (POJO) 序列化及還原序列化成 JSON 文件。
   
        // We'll use Gson for POJO <=> JSON serialization for this example.
        private static Gson gson = new Gson();
   
        @Override
        public TodoItem createTodoItem(TodoItem todoItem) {
            // Serialize the TodoItem as a JSON Document.
            Document todoItemDocument = new Document(gson.toJson(todoItem));
   
            // Annotate the document as a TodoItem for retrieval (so that we can
            // store multiple entity types in the collection).
            todoItemDocument.set("entityType", "todoItem");
   
            try {
                // Persist the document using the DocumentClient.
                todoItemDocument = documentClient.createDocument(
                        getTodoCollection().getSelfLink(), todoItemDocument, null,
                        false).getResource();
            } catch (DocumentClientException e) {
                e.printStackTrace();
                return null;
            }
   
            return gson.fromJson(todoItemDocument.toString(), TodoItem.class);
        }
5. 與 Azure Cosmos DB 資料庫和集合相同，文件也會由自我連結參照。 下列 helper 函式可讓我們依另一個屬性 (例如 "id") 抓取文件，而不是依自我連結：
   
        private Document getDocumentById(String id) {
            // Retrieve the document using the DocumentClient.
            List<Document> documentList = documentClient
                    .queryDocuments(getTodoCollection().getSelfLink(),
                            "SELECT * FROM root r WHERE r.id='" + id + "'", null)
                    .getQueryIterable().toList();
   
            if (documentList.size() > 0) {
                return documentList.get(0);
            } else {
                return null;
            }
        }
6. 我們可以使用步驟 5 中的 helper 方法來依 id 抓取 TodoItem JSON 文件，然後再將其還原序列化成 POJO：
   
        @Override
        public TodoItem readTodoItem(String id) {
            // Retrieve the document by id using our helper method.
            Document todoItemDocument = getDocumentById(id);
   
            if (todoItemDocument != null) {
                // De-serialize the document in to a TodoItem.
                return gson.fromJson(todoItemDocument.toString(), TodoItem.class);
            } else {
                return null;
            }
        }
7. 我們也可以使用 SQL，以 DocumentClient 取得集合或 TodoItems 的清單：
   
        @Override
        public List<TodoItem> readTodoItems() {
            List<TodoItem> todoItems = new ArrayList<TodoItem>();
   
            // Retrieve the TodoItem documents
            List<Document> documentList = documentClient
                    .queryDocuments(getTodoCollection().getSelfLink(),
                            "SELECT * FROM root r WHERE r.entityType = 'todoItem'",
                            null).getQueryIterable().toList();
   
            // De-serialize the documents in to TodoItems.
            for (Document todoItemDocument : documentList) {
                todoItems.add(gson.fromJson(todoItemDocument.toString(),
                        TodoItem.class));
            }
   
            return todoItems;
        }
8. 以 DocumentClient 更新文件的方法有很多個。 在我們的待辦事項清單應用程式中，我們希望能夠切換顯示 TodoItem 是否已完成。 這個目的可以藉由更新文件中的 "complete" 屬性來達成：
   
        @Override
        public TodoItem updateTodoItem(String id, boolean isComplete) {
            // Retrieve the document from the database
            Document todoItemDocument = getDocumentById(id);
   
            // You can update the document as a JSON document directly.
            // For more complex operations - you could de-serialize the document in
            // to a POJO, update the POJO, and then re-serialize the POJO back in to
            // a document.
            todoItemDocument.set("complete", isComplete);
   
            try {
                // Persist/replace the updated document.
                todoItemDocument = documentClient.replaceDocument(todoItemDocument,
                        null).getResource();
            } catch (DocumentClientException e) {
                e.printStackTrace();
                return null;
            }
   
            return gson.fromJson(todoItemDocument.toString(), TodoItem.class);
        }
9. 最後，我們希望能夠從清單中刪除 TodoItem。 若要這樣做，我們可以使用我們稍早撰寫的 helper 方法來抓取自我連結，然後告訴用戶端將它刪除：
   
        @Override
        public boolean deleteTodoItem(String id) {
            // Azure Cosmos DB refers to documents by self link rather than id.
   
            // Query for the document to retrieve the self link.
            Document todoItemDocument = getDocumentById(id);
   
            try {
                // Delete the document by self link.
                documentClient.deleteDocument(todoItemDocument.getSelfLink(), null);
            } catch (DocumentClientException e) {
                e.printStackTrace();
                return false;
            }
   
            return true;
        }

## <a id="Wire"></a>步驟 5：將 Java 應用程式開發專案的其他部分串接在一起
既然我們已經完成主要的部分，剩下的就是建置一個快速的使用者介面，然後將其串接到我們的 DAO。

1. 首先，讓我們著手建置一個控制器來呼叫我們的 DAO：
   
        public class TodoItemController {
            public static TodoItemController getInstance() {
                if (todoItemController == null) {
                    todoItemController = new TodoItemController(TodoDaoFactory.getDao());
                }
                return todoItemController;
            }
   
            private static TodoItemController todoItemController;
   
            private final TodoDao todoDao;
   
            TodoItemController(TodoDao todoDao) {
                this.todoDao = todoDao;
            }
   
            public TodoItem createTodoItem(@NonNull String name,
                    @NonNull String category, boolean isComplete) {
                TodoItem todoItem = TodoItem.builder().name(name).category(category)
                        .complete(isComplete).build();
                return todoDao.createTodoItem(todoItem);
            }
   
            public boolean deleteTodoItem(@NonNull String id) {
                return todoDao.deleteTodoItem(id);
            }
   
            public TodoItem getTodoItemById(@NonNull String id) {
                return todoDao.readTodoItem(id);
            }
   
            public List<TodoItem> getTodoItems() {
                return todoDao.readTodoItems();
            }
   
            public TodoItem updateTodoItem(@NonNull String id, boolean isComplete) {
                return todoDao.updateTodoItem(id, isComplete);
            }
        }
   
    在較複雜的應用程式中，除了 DAO 之外，控制器可能還有複雜的商務邏輯。
2. 接著，我們將建立一個可將 HTTP 要求遞送至控制器的 Servlet：
   
        public class TodoServlet extends HttpServlet {
            // API Keys
            public static final String API_METHOD = "method";
   
            // API Methods
            public static final String CREATE_TODO_ITEM = "createTodoItem";
            public static final String GET_TODO_ITEMS = "getTodoItems";
            public static final String UPDATE_TODO_ITEM = "updateTodoItem";
   
            // API Parameters
            public static final String TODO_ITEM_ID = "todoItemId";
            public static final String TODO_ITEM_NAME = "todoItemName";
            public static final String TODO_ITEM_CATEGORY = "todoItemCategory";
            public static final String TODO_ITEM_COMPLETE = "todoItemComplete";
   
            public static final String MESSAGE_ERROR_INVALID_METHOD = "{'error': 'Invalid method'}";
   
            private static final long serialVersionUID = 1L;
            private static final Gson gson = new Gson();
   
            @Override
            protected void doGet(HttpServletRequest request,
                    HttpServletResponse response) throws ServletException, IOException {
   
                String apiResponse = MESSAGE_ERROR_INVALID_METHOD;
   
                TodoItemController todoItemController = TodoItemController
                        .getInstance();
   
                String id = request.getParameter(TODO_ITEM_ID);
                String name = request.getParameter(TODO_ITEM_NAME);
                String category = request.getParameter(TODO_ITEM_CATEGORY);
                boolean isComplete = StringUtils.equalsIgnoreCase("true",
                        request.getParameter(TODO_ITEM_COMPLETE)) ? true : false;
   
                switch (request.getParameter(API_METHOD)) {
                case CREATE_TODO_ITEM:
                    apiResponse = gson.toJson(todoItemController.createTodoItem(name,
                            category, isComplete));
                    break;
                case GET_TODO_ITEMS:
                    apiResponse = gson.toJson(todoItemController.getTodoItems());
                    break;
                case UPDATE_TODO_ITEM:
                    apiResponse = gson.toJson(todoItemController.updateTodoItem(id,
                            isComplete));
                    break;
                default:
                    break;
                }
   
                response.getWriter().println(apiResponse);
            }
   
            @Override
            protected void doPost(HttpServletRequest request,
                    HttpServletResponse response) throws ServletException, IOException {
                doGet(request, response);
            }
        }
3. 我們需要一個可對使用者顯示的 Web 使用者介面。 讓我們重新撰寫稍早建立的 index.jsp：
    ```html
        <html>
        <head>
          <meta http-equiv="Content-Type" content="text/html; charset=ISO-8859-1">
          <meta http-equiv="X-UA-Compatible" content="IE=edge;" />
          <title>Azure Cosmos DB Java Sample</title>
   
          <!-- Bootstrap -->
          <link href="//ajax.aspnetcdn.com/ajax/bootstrap/3.2.0/css/bootstrap.min.css" rel="stylesheet">
   
          <style>
            /* Add padding to body for fixed nav bar */
            body {
              padding-top: 50px;
            }
          </style>
        </head>
        <body>
          <!-- Nav Bar -->
          <div class="navbar navbar-inverse navbar-fixed-top" role="navigation">
            <div class="container">
              <div class="navbar-header">
                <a class="navbar-brand" href="#">My Tasks</a>
              </div>
            </div>
          </div>
   
          <!-- Body -->
          <div class="container">
            <h1>My ToDo List</h1>
   
            <hr/>
   
            <!-- The ToDo List -->
            <div class = "todoList">
              <table class="table table-bordered table-striped" id="todoItems">
                <thead>
                  <tr>
                    <th>Name</th>
                    <th>Category</th>
                    <th>Complete</th>
                  </tr>
                </thead>
                <tbody>
                </tbody>
              </table>
   
              <!-- Update Button -->
              <div class="todoUpdatePanel">
                <form class="form-horizontal" role="form">
                  <button type="button" class="btn btn-primary">Update Tasks</button>
                </form>
              </div>
   
            </div>
   
            <hr/>
   
            <!-- Item Input Form -->
            <div class="todoForm">
              <form class="form-horizontal" role="form">
                <div class="form-group">
                  <label for="inputItemName" class="col-sm-2">Task Name</label>
                  <div class="col-sm-10">
                    <input type="text" class="form-control" id="inputItemName" placeholder="Enter name">
                  </div>
                </div>
   
                <div class="form-group">
                  <label for="inputItemCategory" class="col-sm-2">Task Category</label>
                  <div class="col-sm-10">
                    <input type="text" class="form-control" id="inputItemCategory" placeholder="Enter category">
                  </div>
                </div>
   
                <button type="button" class="btn btn-primary">Add Task</button>
              </form>
            </div>
   
          </div>
   
          <!-- Placed at the end of the document so the pages load faster -->
          <script src="//ajax.aspnetcdn.com/ajax/jQuery/jquery-2.1.1.min.js"></script>
          <script src="//ajax.aspnetcdn.com/ajax/bootstrap/3.2.0/bootstrap.min.js"></script>
          <script src="assets/todo.js"></script>
        </body>
        </html>
    ```
4. 最後，撰寫一些用戶端 JavaScript，以將 Web 使用者介面與 Servlet 繫結在一起：
   
        var todoApp = {
          /*
           * API methods to call Java backend.
           */
          apiEndpoint: "api",
   
          createTodoItem: function(name, category, isComplete) {
            $.post(todoApp.apiEndpoint, {
                "method": "createTodoItem",
                "todoItemName": name,
                "todoItemCategory": category,
                "todoItemComplete": isComplete
              },
              function(data) {
                var todoItem = data;
                todoApp.addTodoItemToTable(todoItem.id, todoItem.name, todoItem.category, todoItem.complete);
              },
              "json");
          },
   
          getTodoItems: function() {
            $.post(todoApp.apiEndpoint, {
                "method": "getTodoItems"
              },
              function(data) {
                var todoItemArr = data;
                $.each(todoItemArr, function(index, value) {
                  todoApp.addTodoItemToTable(value.id, value.name, value.category, value.complete);
                });
              },
              "json");
          },
   
          updateTodoItem: function(id, isComplete) {
            $.post(todoApp.apiEndpoint, {
                "method": "updateTodoItem",
                "todoItemId": id,
                "todoItemComplete": isComplete
              },
              function(data) {},
              "json");
          },
   
          /*
           * UI Methods
           */
          addTodoItemToTable: function(id, name, category, isComplete) {
            var rowColor = isComplete ? "active" : "warning";
   
            todoApp.ui_table().append($("<tr>")
              .append($("<td>").text(name))
              .append($("<td>").text(category))
              .append($("<td>")
                .append($("<input>")
                  .attr("type", "checkbox")
                  .attr("id", id)
                  .attr("checked", isComplete)
                  .attr("class", "isComplete")
                ))
              .addClass(rowColor)
            );
          },
   
          /*
           * UI Bindings
           */
          bindCreateButton: function() {
            todoApp.ui_createButton().click(function() {
              todoApp.createTodoItem(todoApp.ui_createNameInput().val(), todoApp.ui_createCategoryInput().val(), false);
              todoApp.ui_createNameInput().val("");
              todoApp.ui_createCategoryInput().val("");
            });
          },
   
          bindUpdateButton: function() {
            todoApp.ui_updateButton().click(function() {
              // Disable button temporarily.
              var myButton = $(this);
              var originalText = myButton.text();
              $(this).text("Updating...");
              $(this).prop("disabled", true);
   
              // Call api to update todo items.
              $.each(todoApp.ui_updateId(), function(index, value) {
                todoApp.updateTodoItem(value.name, value.value);
                $(value).remove();
              });
   
              // Re-enable button.
              setTimeout(function() {
                myButton.prop("disabled", false);
                myButton.text(originalText);
              }, 500);
            });
          },
   
          bindUpdateCheckboxes: function() {
            todoApp.ui_table().on("click", ".isComplete", function(event) {
              var checkboxElement = $(event.currentTarget);
              var rowElement = $(event.currentTarget).parents('tr');
              var id = checkboxElement.attr('id');
              var isComplete = checkboxElement.is(':checked');
   
              // Toggle table row color
              if (isComplete) {
                rowElement.addClass("active");
                rowElement.removeClass("warning");
              } else {
                rowElement.removeClass("active");
                rowElement.addClass("warning");
              }
   
              // Update hidden inputs for update panel.
              todoApp.ui_updateForm().children("input[name='" + id + "']").remove();
   
              todoApp.ui_updateForm().append($("<input>")
                .attr("type", "hidden")
                .attr("class", "updateComplete")
                .attr("name", id)
                .attr("value", isComplete));
   
            });
          },
   
          /*
           * UI Elements
           */
          ui_createNameInput: function() {
            return $(".todoForm #inputItemName");
          },
   
          ui_createCategoryInput: function() {
            return $(".todoForm #inputItemCategory");
          },
   
          ui_createButton: function() {
            return $(".todoForm button");
          },
   
          ui_table: function() {
            return $(".todoList table tbody");
          },
   
          ui_updateButton: function() {
            return $(".todoUpdatePanel button");
          },
   
          ui_updateForm: function() {
            return $(".todoUpdatePanel form");
          },
   
          ui_updateId: function() {
            return $(".todoUpdatePanel .updateComplete");
          },
   
          /*
           * Install the TodoApp
           */
          install: function() {
            todoApp.bindCreateButton();
            todoApp.bindUpdateButton();
            todoApp.bindUpdateCheckboxes();
   
            todoApp.getTodoItems();
          }
        };
   
        $(document).ready(function() {
          todoApp.install();
        });
5. 好極了！ 現在只剩下測試應用程式。 在本機執行應用程式，並填入項目名稱和類別，然後按一下 [ **新增工作**] 來新增一些待辦事項。
6. 當項目出現時，您可以切換勾選核取方塊，然後按一下 [更新工作] ，來更新其完成狀態。

## <a id="Deploy"></a>步驟 6：將 Java 應用程式部署至 Azure 網站
Azure 網站讓部署 Java 應用程式變得相當簡單，您只需將應用程式匯出成 WAR 檔案，然後透過原始檔控制 (例如 Git) 或 FTP 上傳它即可。

1. 若要將應用程式匯出成 WAR 檔案，請以滑鼠右鍵按一下您在**專案總管**中的專案，按一下 [匯出]，然後按一下 [WAR 檔案]。
2. 在 [WAR 匯出]  視窗中，執行下列動作：
   
   * 在 [Web 專案] 方塊中，輸入 azure-documentdb-java-sample。
   * 在 [目的地] 方塊中，選擇用來儲存 WAR 檔案的目的地。
   * 按一下 [完成] 。
3. 現在您手上已經有了 WAR 檔案，您只需將它上傳至您 Azure 網站的 **webapps** 目錄即可。 如需上傳檔案的相關指示，請參閱[將 Java 應用程式新增至 Azure App Service Web Apps](../app-service/web-sites-java-add-app.md)。
   
    將 WAR 檔案上傳至 webapps 目錄之後，執行階段環境便會偵測到您已新增它，並自動將其載入。
4. 若要檢視您已完成的產品，請瀏覽至 http://YOUR\_SITE\_NAME.azurewebsites.net/azure-java-sample/ 並開始新增您的工作！

## <a id="GetProject"></a>從 GitHub 取得的專案
本教學課程中的所有範例都包含在 GitHub 上的 [待辦事項](https://github.com/Azure-Samples/documentdb-java-todo-app) 專案中。 若要將 todo 專案匯入 Eclipse，請確認您擁有 [必要條件](#Prerequisites) 區段中所列出的軟體和資源，然後執行下列動作：

1. 安裝 [專案 Lombok](http://projectlombok.org/)。 Lombok 可用來在專案中產生建構函式、getter、setter。 下載 lombok.jar 檔案之後，請連按兩下進行安裝，或從命令列進行安裝。
2. 如果 Eclipse 為開啟狀態，請將它關閉並重新啟動以載入 Lombok。
3. 在 Eclipse 的 [檔案] 功能表上，按一下 [匯入]。
4. 在 [匯入] 視窗中，依序按一下 [Git]、[使用 Git 的專案] 和 [下一步]。
5. 在 [選取儲存機制來源] 畫面上，按一下 [複製 URI]。
6. 在 [來源 Git 存放庫] 畫面的 [URI] 方塊中，輸入 https://github.com/Azure-Samples/documentdb-java-todo-app.git，然後按一下 [下一步]。
7. 在 [分支選取] 畫面上，確定已選取 [主要]，然後按 [下一步]。
8. 在 [本機目的地] 畫面上，按一下 [瀏覽] 以選取可以複製儲存機制的資料夾，然後按 [下一步]。
9. 在 [選取要用於匯入專案的精靈] 畫面上，確定已選取 [匯入現有的專案]，然後按 [下一步]。
10. 在 [匯入專案] 畫面上，取消選取 **DocumentDB** 專案，然後按一下 [完成]。 DocumentDB 專案包含 Azure Cosmos DB Java SDK，我們將會改成新增為相依性。
11. 在 [專案總管] 中，瀏覽至 azure-documentdb-java-sample\src\com.microsoft.azure.documentdb.sample.dao\DocumentClientFactory.java，並將 [主機] 和 [MASTER_KEY] 值取代為您 Azure Cosmos DB 帳戶的 [URI] 和 [主要金鑰]，然後儲存檔案。 如需詳細資訊，請參閱[步驟 1。建立 Azure Cosmos DB 資料庫帳戶](#CreateDB)。
12. 在 [專案總管] 中，以滑鼠右鍵按一下 **azure-documentdb-java-sample**，按一下 [組建路徑]，然後按一下 [設定組建路徑]。
13. 在 [Java 組建路徑] 畫面的右側窗格中，選取 [程式庫] 索引標籤，然後按一下 [新增外部 JAR]。 瀏覽至 lombok.jar 檔案的位置，按一下 [開啟]，然後按一下 [確定]。
14. 使用步驟 12 重新開啟 [屬性] 視窗，然後在左側窗格中按一下 [目標執行階段]。
15. 在 [目標執行階段] 畫面上，按一下 [新增]，選取 [Apache Tomcat v7.0]，然後按一下 [確定]。
16. 使用步驟 12 重新開啟 [屬性] 視窗，然後在左側窗格中按一下 [專案 Facet]。
17. 在 [專案 Facet] 畫面上，選取 [動態 Web 模組] 和 [Java]，然後按一下 [確定]。
18. 在螢幕底部的 [伺服器] 索引標籤上，以滑鼠右鍵按一下 [在 localhost 的 Tomcat v7.0 伺服器]，然後按一下 [新增和移除]。
19. 在 [新增和移除] 視窗中，將 [azure-documentdb-java-sample] 移至 [已設定] 方塊，然後按一下 [完成]。
20. 在 [伺服器] 索引標籤上，以滑鼠右鍵按一下 [Tomcat v7.0 Server at localhost] \(在 localhost 的 Tomcat v7.0 伺服器)，然後按一下 [重新啟動]。
21. 在瀏覽器中，瀏覽至 http://localhost:8080/azure-documentdb-java-sample/，並開始新增到工作清單。 請注意，如果您之前變更預設的連接埠值，請將 8080 變更為您所選取的值。
22. 若要將您的專案部署至 Azure 網站，請參閱[步驟 6：將應用程式部署至 Azure 網站](#Deploy)。

