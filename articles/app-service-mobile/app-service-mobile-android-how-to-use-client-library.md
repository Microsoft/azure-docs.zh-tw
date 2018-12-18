---
title: 如何使用 Android 版 Azure Mobile Apps SDK | Microsoft Docs
description: 如何使用 Android 版 Azure Mobile Apps SDK
services: app-service\mobile
documentationcenter: android
author: conceptdev
manager: crdun
ms.assetid: 5352d1e4-7685-4a11-aaf4-10bd2fa9f9fc
ms.service: app-service-mobile
ms.workload: mobile
ms.tgt_pltfrm: mobile-android
ms.devlang: java
ms.topic: article
ms.date: 11/16/2017
ms.author: crdun
ms.openlocfilehash: 1ab7aa9ecdd51809f6e1d82958f21b78b16e7e63
ms.sourcegitcommit: 0b4da003fc0063c6232f795d6b67fa8101695b61
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/05/2018
ms.locfileid: "37859552"
---
# <a name="how-to-use-the-azure-mobile-apps-sdk-for-android"></a>如何使用 Android 版 Azure Mobile Apps SDK

本指南說明如何使用適用於 Mobile Apps 的 Android 用戶端 SDK 來實作常見案例，例如：

* 查詢資料 (插入、更新和刪除)。
* 驗證。
* 處理錯誤。
* 自訂用戶端。

本指南著重於用戶端 Android SDK。  若要深入了解 Mobile Apps 的伺服器端 SDK，請參閱[使用 .NET 後端 SDK][10]或[如何使用 Node.js 後端 SDK][11]。

## <a name="reference-documentation"></a>參考文件

您可以在 GitHub 找到 Android 用戶端程式庫的 [Javadocs API 參考][12]。

## <a name="supported-platforms"></a>支援的平台

Android 版 Azure Mobile Apps SDK 支援 API 層級 19 至 24 (KitKat 至 Nougat) 的手機和平板電腦外形規格。  特別是驗證，會使用常用的 web 架構方法來收集認證。  伺服器流程驗證不適用於小型外形規格的裝置，例如手錶。

## <a name="setup-and-prerequisites"></a>設定和必要條件

完成 [Mobile Apps 快速入門](app-service-mobile-android-get-started.md) 教學課程。  此工作可確保您已符合開發 Azure Mobile Apps 的所有必要條件。  快速入門也可協助您設定帳戶，並建立您的第一個行動應用程式後端。

如果您決定不要完成快速入門教學課程，請完成下列工作︰

* [建立行動應用程式後端][13]以搭配 Android 應用程式使用。
* 在 Android Studio 中， [更新 Gradle 組建檔案](#gradle-build)。
* [啟用網際網路權限](#enable-internet)。

### <a name="gradle-build"></a>更新 Gradle 組建檔案

變更以下兩個 build.gradle  檔案：

1. 將此程式碼新增至 *buildscript* 標籤內的*專案*層級 **build.gradle** 檔案：

    ```text
    buildscript {
        repositories {
            jcenter()
        }
    }
    ```

2. 將此程式碼新增至 *dependencies* 標籤內的*模組應用程式*層級 **build.gradle** 檔案：

    ```text
    compile 'com.microsoft.azure:azure-mobile-android:3.4.0@aar'
    ```

    目前最新版為 3.4.0。 支援的版本列在 [Bintray 上][14]。

### <a name="enable-internet"></a>啟用網際網路權限

若要存取 Azure，您的應用程式必須啟用網際網路權限。 如果尚未啟用，請將下列這一行程式碼加入至 AndroidManifest.xml  檔案：

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

## <a name="create-a-client-connection"></a>建立用戶端連線

Azure Mobile Apps 為您的行動應用程式提供四個函式︰

* Azure Mobile Apps Service 的「資料存取」和「離線同步處理」。
* 以 Azure Mobile Apps Server SDK 撰寫的「呼叫自訂 API」。
* Azure App Service 驗證和授權的「驗證」。
* 通知中樞的「推播通知註冊」。

這些函式首先會要求您建立 `MobileServiceClient` 物件。  您的行動用戶端內只應該建立一個 `MobileServiceClient` 物件 (亦即，它應該是單一子句模式)。  若要建立 `MobileServiceClient` 物件︰

```java
MobileServiceClient mClient = new MobileServiceClient(
    "<MobileAppUrl>",       // Replace with the Site URL
    this);                  // Your application Context
```

`<MobileAppUrl>` 是字串，或是指向您行動後端的 URL 物件。  如果您是使用 Azure App Service 來裝載您的行動後端，請確定您使用安全 `https://` 版本的 URL。

用戶端也需要存取活動或內容 - 本範例中為 `this` 參數。  MobileServiceClient 建構應該發生在 `AndroidManifest.xml` 檔案中所參考的 `onCreate()` 活動方法。

最佳做法是，您應該將伺服器通訊擷取到它自己的 (單一模式) 類別。  在此情況下，您應該傳遞建構函式內的活動，以適當地設定服務。  例如︰

```java
package com.example.appname.services;

import android.content.Context;
import com.microsoft.windowsazure.mobileservices.*;

public class AzureServiceAdapter {
    private String mMobileBackendUrl = "https://myappname.azurewebsites.net";
    private Context mContext;
    private MobileServiceClient mClient;
    private static AzureServiceAdapter mInstance = null;

    private AzureServiceAdapter(Context context) {
        mContext = context;
        mClient = new MobileServiceClient(mMobileBackendUrl, mContext);
    }

    public static void Initialize(Context context) {
        if (mInstance == null) {
            mInstance = new AzureServiceAdapter(context);
        } else {
            throw new IllegalStateException("AzureServiceAdapter is already initialized");
        }
    }

    public static AzureServiceAdapter getInstance() {
        if (mInstance == null) {
            throw new IllegalStateException("AzureServiceAdapter is not initialized");
        }
        return mInstance;
    }

    public MobileServiceClient getClient() {
        return mClient;
    }

    // Place any public methods that operate on mClient here.
}
```

您現在可以呼叫主要活動 `onCreate()` 方法中的 `AzureServiceAdapter.Initialize(this);`。  需要存取用戶端的其他任何方法會使用 `AzureServiceAdapter.getInstance();`，來取得服務配接器的參考。

## <a name="data-operations"></a>資料作業

Azure Mobile Apps SDK 的核心，是提供對行動裝置應用程式後端上的 SQL Azure 內所儲存資料的存取權。  您可以使用強型別類型 (建議選項) 或不具類型的查詢 (不建議) 來存取此資料。  本章節大部分都在討論使用強型別類型。

### <a name="define-client-data-classes"></a>定義用戶端資料類別

若要存取 SQL Azure 資料表的資料，請定義對應至行動應用程式後端中資料表的用戶端資料類別。 本主題中的範例採用名為 MyDataTable 的資料表，其中包含下列資料行：

* id
* text
* 完成

對應型別的用戶端物件位於名為 MyDataTable.java 的檔案中：

```java
public class ToDoItem {
    private String id;
    private String text;
    private Boolean complete;
}
```

針對您新增的每個欄位，新增 getter 和 setter 方法。  如果您的 SQL Azure 資料表包含多個資料行，您會將對應的欄位新增至此類別。  例如，如果 DTO (資料傳輸物件) 有整數 Priority 資料行，則您可能會新增此欄位，以及其 getter 和 setter 方法：

```java
private Integer priority;

/**
* Returns the item priority
*/
public Integer getPriority() {
    return mPriority;
}

/**
* Sets the item priority
*
* @param priority
*            priority to set
*/
public final void setPriority(Integer priority) {
    mPriority = priority;
}
```

若要了解如何在您的 Mobile Apps 後端建立其他資料表，請參閱[做法：定義資料表控制器][15] (.NET 後端) 或[使用動態結構描述定義資料表][16] (Node.js 後端)。

Azure Mobile Apps 後端資料表定義了五個特殊欄位，其中四個可供用戶端使用︰

* `String id`︰記錄的全域唯一識別碼。  最佳做法是，使識別碼成為 [UUID][17] 物件的字串表示法。
* `DateTimeOffset updatedAt`︰上次更新的日期/時間。  UpdatedAt 欄位由伺服器設定，且不得由用戶端程式碼設定。
* `DateTimeOffset createdAt`：建立物件的日期/時間。  CreatedAt 欄位由伺服器設定，且不得由用戶端程式碼設定。
* `byte[] version`︰通常會以字串表示，版本也是由伺服器設定。
* `boolean deleted`︰表示記錄已刪除，但尚未清除。  請勿使用 `deleted` 作為您類別中的屬性。

`id` 是必填欄位。  `updatedAt` 欄位和 `version` 欄位是用於離線同步處理 (分別適用於增量同步處理和衝突解決)。  `createdAt` 欄位是參考欄位，且用戶端不可使用。  名稱為屬性的 "across-the-wire" 名稱，且不可調整。  不過，您可以使用 [gson][3] 程式庫，建立物件與 "across-the-wire" 名稱之間的對應。  例如︰

```java
package com.example.zumoappname;

import com.microsoft.windowsazure.mobileservices.table.DateTimeOffset;

public class ToDoItem
{
    @com.google.gson.annotations.SerializedName("id")
    private String mId;
    public String getId() { return mId; }
    public final void setId(String id) { mId = id; }

    @com.google.gson.annotations.SerializedName("complete")
    private boolean mComplete;
    public boolean isComplete() { return mComplete; }
    public void setComplete(boolean complete) { mComplete = complete; }

    @com.google.gson.annotations.SerializedName("text")
    private String mText;
    public String getText() { return mText; }
    public final void setText(String text) { mText = text; }

    @com.google.gson.annotations.SerializedName("createdAt")
    private DateTimeOffset mCreatedAt;
    public DateTimeOffset getCreatedAt() { return mCreatedAt; }
    protected void setCreatedAt(DateTimeOffset createdAt) { mCreatedAt = createdAt; }

    @com.google.gson.annotations.SerializedName("updatedAt")
    private DateTimeOffset mUpdatedAt;
    public DateTimeOffset getUpdatedAt() { return mUpdatedAt; }
    protected void setUpdatedAt(DateTimeOffset updatedAt) { mUpdatedAt = updatedAt; }

    @com.google.gson.annotations.SerializedName("version")
    private String mVersion;
    public String getVersion() { return mVersion; }
    public final void setVersion(String version) { mVersion = version; }

    public ToDoItem() { }

    public ToDoItem(String id, String text) {
        this.setId(id);
        this.setText(text);
    }

    @Override
    public boolean equals(Object o) {
        return o instanceof ToDoItem && ((ToDoItem) o).mId == mId;
    }

    @Override
    public String toString() {
        return getText();
    }
}
```

### <a name="create-a-table-reference"></a>建立資料表參考

若要存取資料表，請先在 [MobileServiceClient][9] 上呼叫 **getTable** 方法，以建立 [MobileServiceTable][8] 物件。  此方法有兩個多載：

```java
public class MobileServiceClient {
    public <E> MobileServiceTable<E> getTable(Class<E> clazz);
    public <E> MobileServiceTable<E> getTable(String name, Class<E> clazz);
}
```

在下列程式碼中， **mClient** 是您的 MobileServiceClient 物件的參考。  第一個多載會在類別名稱與資料表名稱相同的情況下使用，而且會使用於 Quickstart：

```java
MobileServiceTable<ToDoItem> mToDoTable = mClient.getTable(ToDoItem.class);
```

第二個多載會在資料表名稱與類型名稱不同時使用：第一個參數為資料表名稱。

```java
MobileServiceTable<ToDoItem> mToDoTable = mClient.getTable("ToDoItemBackup", ToDoItem.class);
```

## <a name="query"></a>查詢後端資料表

首先，取得資料表參考。  然後針對資料表參考執行查詢。  查詢是以下的任何組合︰

* `.where()` [篩選子句](#filtering)。
* `.orderBy()` [排序子句](#sorting)。
* `.select()` [欄位選取子句](#selection)。
* [分頁結果](#paging)的 `.skip()` 和 `.top()`。

子句必須依上述順序呈現。

### <a name="filter"></a> 篩選結果

查詢的一般格式如下︰

```java
List<MyDataTable> results = mDataTable
    // More filters here
    .execute()          // Returns a ListenableFuture<E>
    .get()              // Converts the async into a sync result
```

上述範例會傳回所有結果 (最多為伺服器所設定的最大頁面大小)。  `.execute()` 方法會在後端執行查詢。  傳送至 Mobile Apps 後端之前，查詢會轉換成 [OData v3][19] 查詢。  在收到時，Mobile Apps 後端在 SQL Azure 執行個體上執行查詢之前，會將查詢轉換成 SQL 陳述式。  由於網路活動耗費時間，`.execute()` 方法會傳回 [`ListenableFuture<E>`][18]。

### <a name="filtering"></a>篩選傳回的資料

下列查詢執行會從 **complete** 等於 **false** 的 **ToDoItem** 資料表傳回所有項目。

```java
List<ToDoItem> result = mToDoTable
    .where()
    .field("complete").eq(false)
    .execute()
    .get();
```

**mToDoTable** 是我們先前建立的行動服務資料表的參考。

在資料表參考上使用 **where** 方法呼叫定義篩選器。 在 **where** 方法之後，依序是 **field** 方法以及指定邏輯述詞的方法。 可能的述詞方法包括 **eq** (等於)、**ne** (不等於)、**gt** (大於)、**ge** (大於或等於)、**lt** (小於)、**le** (小於或等於)。 這些方法可讓您比較數字和字串欄位與特定值。

您可以依日期進行篩選。 下列方法可讓您比較整個日期欄位或比較日期的某些部分：**year**、**month**、**day**、**hour**、**minute** 和 **second**。 下列範例會為 *到期日* 為 2013 年的項目新增篩選器。

```java
List<ToDoItem> results = MToDoTable
    .where()
    .year("due").eq(2013)
    .execute()
    .get();
```

下列方法在字串欄位上支援複雜的篩選器：**startsWith**、**endsWith**、**concat**、**subString**、**indexOf**、**replace**、**toLower**、**toUpper**、**trim** 和 **length**。 下列範例會對「text」  資料行的開頭為 "PRI0" 的資料表資料列進行篩選。

```java
List<ToDoItem> results = mToDoTable
    .where()
    .startsWith("text", "PRI0")
    .execute()
    .get();
```

數字欄位支援下列運算子方法：**add**、**sub**、**mul**、**div**、**mod**、**floor**、**ceiling** 和 **round**。 下列範例會對 **duration** 為偶數的資料表資料列進行篩選。

```java
List<ToDoItem> results = mToDoTable
    .where()
    .field("duration").mod(2).eq(0)
    .execute()
    .get();
```

您可以結合述詞與下列邏輯方法：**and**、**or** 和 **not**。 下列範例會結合上述的兩個範例。

```java
List<ToDoItem> results = mToDoTable
    .where()
    .year("due").eq(2013).and().startsWith("text", "PRI0")
    .execute()
    .get();
```

群組和巢狀邏輯運算子︰

```java
List<ToDoItem> results = mToDoTable
    .where()
    .year("due").eq(2013)
    .and(
        startsWith("text", "PRI0")
        .or()
        .field("duration").gt(10)
    )
    .execute().get();
```

如需篩選的詳細討論與範例，請參閱[探索功能豐富的 Android 用戶端查詢模型][20]。

### <a name="sorting"></a>排序傳回的資料

下列程式碼會從 **ToDoItems** 資料表傳回依 *text* 欄位遞增排序的所有項目。 *mToDoTable* 是您先前建立的後端資料表的參考。

```java
List<ToDoItem> results = mToDoTable
    .orderBy("text", QueryOrder.Ascending)
    .execute()
    .get();
```

**orderBy** 方法的第一個參數是一個字串，代表作為排序依據的欄位名稱。 第二個參數會使用 **QueryOrder** 列舉，指定要進行遞增還是遞減排序。  如果您使用 ***where*** 方法進行篩選，***where*** 方法必須要在 ***orderBy*** 方法之前叫用。

### <a name="selection"></a>選取特定資料行

下列程式碼將說明如何傳回 **ToDoItems** 資料表中的所有項目，但僅顯示 **complete** 和 **text** 欄位。 **mToDoTable** 是我們先前建立的後端資料表的參考。

```java
List<ToDoItemNarrow> result = mToDoTable
    .select("complete", "text")
    .execute()
    .get();
```

select 函數的參數是您要傳回之資料表資料行的字串名稱。  **select** 方法必須跟隨在 **where** 和 **orderBy** 等方法之後。 而其後可以跟隨 **skip** 和 **top** 之類的分頁方法。

### <a name="paging"></a>以分頁方式傳回資料

資料**一律**以分頁方式傳回。  傳回的記錄數目上限是由伺服器設定。  如果用戶端要求更多記錄，伺服器就會傳回最大記錄數目。  根據預設，伺服器上的最大頁面大小為 50 筆記錄。

第一個範例將說明如何從資料表中選取前五個項目。 查詢會傳回 **ToDoItems**資料表中的項目。 **mToDoTable** 是您先前建立的後端資料表的參考。

```java
List<ToDoItem> result = mToDoTable
    .top(5)
    .execute()
    .get();
```

以下是略過前五個項目，而傳回後續五個項目的查詢：

```java
List<ToDoItem> result = mToDoTable
    .skip(5).top(5)
    .execute()
    .get();
```

如果您想要取得資料表中的所有記錄，請實作程式碼來逐一查看所有頁面︰

```java
List<MyDataModel> results = new List<MyDataModel>();
int nResults;
do {
    int currentCount = results.size();
    List<MyDataModel> pagedResults = mDataTable
        .skip(currentCount).top(500)
        .execute().get();
    nResults = pagedResults.size();
    if (nResults > 0) {
        results.addAll(pagedResults);
    }
} while (nResults > 0);
```

使用此方法要求所有記錄，會向 Mobile Apps 後端建立最少兩個要求。

> [!TIP]
> 選擇正確的頁面大小是在要求過程中的記憶體使用量、頻寬使用量和完全接收資料的延遲時間之間取得平衡。  預設值 (50 筆記錄) 適用於所有裝置。  如果您在較大記憶體裝置上獨立操作，則最多增加至 500。  我們發現，將頁面大小增加至超過 500 筆記錄，會導致無法接受的延遲和大量記憶體問題。

### <a name="chaining"></a>作法：串連查詢方法

用來查詢後端資料表的方法是可以串連的。 鏈結查詢方法可讓您從排序和分頁的篩選資料列中選取特定資料行。 您可以建立複雜的邏輯篩選器。  每個查詢方法都會傳回 Query 物件。 若要結束這一系列的方法並實際執行查詢，請呼叫 **execute** 方法。 例如︰

```java
List<ToDoItem> results = mToDoTable
        .where()
        .year("due").eq(2013)
        .and(
            startsWith("text", "PRI0").or().field("duration").gt(10)
        )
        .orderBy(duration, QueryOrder.Ascending)
        .select("id", "complete", "text", "duration")
        .skip(200).top(100)
        .execute()
        .get();
```

鏈結的查詢方法必須依下列順序︰

1. 篩選 (**where**) 方法。
2. 排序 (**orderBy**) 方法。
3. 選取 (**select**) 方法。
4. 分頁 (**skip** 和 **top**) 方法。

## <a name="binding"></a>將資料繫結到使用者介面

資料繫結牽涉到三項要件：

* 資料來源
* 畫面配置
* 將這兩個項目連結在一起的配接器。

在範例程式碼中，我們會將 Mobile Apps SQL Azure 資料表 **ToDoItem** 中的資料傳回陣列中。 此活動是常見的資料應用模式。  資料庫查詢通常會傳回用戶端在清單或陣列中取得的資料列集合。 在此範例中，陣列是資料來源。  程式碼會指定一個畫面配置，以定義裝置上出現的資料檢視。  這兩個項目會透過配接器繫結在一起；在此程式碼中，配接器會是 **ArrayAdapter&lt;ToDoItem&gt;** 類別的擴充功能。

#### <a name="layout"></a>定義配置

配置可使用數個 XML 程式碼片段來定義。 在現有配置下，下列程式碼會顯示我們要以伺服器資料填入的 **ListView** 。

```xml
    <ListView
        android:id="@+id/listViewToDo"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        tools:listitem="@layout/row_list_to_do" >
    </ListView>
```

在上述程式碼中，「listitem」  屬性會指定清單中個別資料列的配置識別碼。 此程式碼會指定核取方塊及其相關文字，並對清單中的每個項目具現化一次。 此配置不會顯示 **id** 欄位，而更複雜的配置將會指定顯示畫面中的其他欄位。 此程式碼位於 **row_list_to_do.xml** 檔案中。

```java
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="horizontal">
    <CheckBox
        android:id="@+id/checkToDoItem"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/checkbox_text" />
</LinearLayout>
```

#### <a name="adapter"></a>定義配接器
由於我們的檢視資料來源是 **ToDoItem** 的陣列，因此我們將配接器設為 **ArrayAdapter&lt;ToDoItem&gt;** 類別的子類別。 這個子類別會為每個使用 **row_list_to_do** 配置的 **ToDoItem** 產生一個檢視。  我們在程式碼中定義了下列類別，這是 **ArrayAdapter&lt;E&gt;** 類別的擴充功能：

```java
public class ToDoItemAdapter extends ArrayAdapter<ToDoItem> {
}
```

覆寫配接器的 **getView** 方法。 例如︰

```
    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        View row = convertView;

        final ToDoItem currentItem = getItem(position);

        if (row == null) {
            LayoutInflater inflater = ((Activity) mContext).getLayoutInflater();
            row = inflater.inflate(R.layout.row_list_to_do, parent, false);
        }
        row.setTag(currentItem);

        final CheckBox checkBox = (CheckBox) row.findViewById(R.id.checkToDoItem);
        checkBox.setText(currentItem.getText());
        checkBox.setChecked(false);
        checkBox.setEnabled(true);

        checkBox.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View arg0) {
                if (checkBox.isChecked()) {
                    checkBox.setEnabled(false);
                    if (mContext instanceof ToDoActivity) {
                        ToDoActivity activity = (ToDoActivity) mContext;
                        activity.checkItem(currentItem);
                    }
                }
            }
        });
        return row;
    }
```

我們在「活動」中建立此類別的執行個體，如下所示：

```java
    ToDoItemAdapter mAdapter;
    mAdapter = new ToDoItemAdapter(this, R.layout.row_list_to_do);
```

ToDoItemAdapter 建構函式的第二個參數是配置的參考。 我們現在可以具現化 **ListView**，並對 **ListView** 指派配接器。

```java
    ListView listViewToDo = (ListView) findViewById(R.id.listViewToDo);
    listViewToDo.setAdapter(mAdapter);
```

#### <a name="use-adapter"></a>使用配接器來繫結至 UI

您現在已可使用資料繫結。 下列程式碼說明如何取得資料表中的項目，並以傳回的項目填入本機配接器。

```java
    public void showAll(View view) {
        AsyncTask<Void, Void, Void> task = new AsyncTask<Void, Void, Void>(){
            @Override
            protected Void doInBackground(Void... params) {
                try {
                    final List<ToDoItem> results = mToDoTable.execute().get();
                    runOnUiThread(new Runnable() {

                        @Override
                        public void run() {
                            mAdapter.clear();
                            for (ToDoItem item : results) {
                                mAdapter.add(item);
                            }
                        }
                    });
                } catch (Exception exception) {
                    createAndShowDialog(exception, "Error");
                }
                return null;
            }
        };
        runAsyncTask(task);
    }
```

請在每次修改 **ToDoItem** 資料表時呼叫配接器。 修改是對個別記錄逐一執行的，因此您會處理單一資料列，而不是集合。 在插入項目時，請對配接器呼叫 **add** 方法，刪除時則呼叫 **remove** 方法。

您可以在 [Android 快速入門專案][21]中找到完整的範例。

## <a name="inserting"></a>將資料插入後端

將「ToDoItem」  類別的執行個體具現化，並設定其屬性。

```java
ToDoItem item = new ToDoItem();
item.text = "Test Program";
item.complete = false;
```

然後使用 **insert()** 插入物件︰

```java
ToDoItem entity = mToDoTable
    .insert(item)       // Returns a ListenableFuture<ToDoItem>
    .get();
```

傳回的實體會符合插入後端資料表的資料，包括識別碼和後端上設定的任何其他值 (例如 `createdAt`、`updatedAt` 和 `version` 欄位)。

Mobile Apps 資料表需要名為 **識別碼**的主索引鍵資料行。此資料行必須是字串。 識別碼資料行的預設值是 GUID。  您可以提供其他的唯一值，例如電子郵件地址或使用者名稱。 未針對插入的記錄提供字串識別碼值時，後端會產生新的 GUID。

字串識別碼值提供下列優點：

* 不需要往返存取資料庫就能產生識別碼。
* 輕鬆合併不同資料表或資料庫的記錄。
* 識別碼值與應用程式邏輯的整合更理想。

若要支援離線同步處理，則 **需要** 字串識別碼值。  一旦識別碼儲存在後端資料庫中，您就無法將它變更。

## <a name="updating"></a>將行動裝置應用程式中的資料更新

若要更新資料表中的資料，請將新物件傳遞至 **update()** 方法。

```java
mToDoTable
    .update(item)   // Returns a ListenableFuture<ToDoItem>
    .get();
```

在此範例中，*item* 是 *ToDoItem* 資料表中某個資料列的參考，其中已有一些變更。  具有相同 **識別碼** 的資料列已更新。

## <a name="deleting"></a>將行動裝置應用程式中的資料刪除

下列程式碼說明如何藉由指定資料物件來刪除資料表中的資料。

```java
mToDoTable
    .delete(item);
```

您也可以藉由指定要刪除的資料列的 **id** 欄位來刪除項目。

```java
String myRowId = "2FA404AB-E458-44CD-BC1B-3BC847EF0902";
mToDoTable
    .delete(myRowId);
```

## <a name="lookup"></a>依識別碼查閱特定項目

以 **lookUp()** 方法查閱具有特定**識別碼**欄位的項目：

```java
ToDoItem result = mToDoTable
    .lookUp("0380BAFB-BCFF-443C-B7D5-30199F730335")
    .get();
```

## <a name="untyped"></a>作法：使用不具類型的資料

不具型別的程式設計模型可讓您精確掌控 JSON 序列化。  在某些常見情況下，您可能會想使用不具型別的程式設計模型。 例如，如果您的後端資料表包含許多資料行，但您只需要參考其中幾個資料行時。  使用具型別的模型時，您必須在資料類別中將 Mobile Apps 後端所定義的所有資料行進行定義。  用來存取資料的 API 呼叫大多會與型別程式設計呼叫相類似。 主要的差別在於，在不具型別的模型中，您會叫用 **MobileServiceJsonTable** 物件的方法，而不是 **MobileServiceTable** 物件。

### <a name="json_instance"></a>建立不具型別的資料表執行個體

和型別模型一樣，首先您必須取得資料表參考，但在此案例中這會是 **MobileServicesJsonTable** 物件。 在用戶端的執行個體上呼叫 **getTable** 方法以取得參考：

```java
private MobileServiceJsonTable mJsonToDoTable;
//...
mJsonToDoTable = mClient.getTable("ToDoItem");
```

在您建立 **MobileServiceJsonTable**的執行個體後，它就幾乎具有和型別程式設計模型所能使用的一樣的 API。 在某些情況下，這些方法會採用不具型別的參數，而非具型別的參數。

### <a name="json_insert"></a>插入不具型別的資料表中
下列程式碼將說明如何執行插入。 第一個步驟是建立屬於 [gson][3] 程式庫的 [JsonObject][1]。

```java
JsonObject jsonItem = new JsonObject();
jsonItem.addProperty("text", "Wake up");
jsonItem.addProperty("complete", false);
```

然後，使用 **insert()** 在資料表中插入不具型別的物件。

```java
JsonObject insertedItem = mJsonToDoTable
    .insert(jsonItem)
    .get();
```

如果您需要取得所插入之物件的識別碼，請使用 **getAsJsonPrimitive()** 方法。

```java
String id = insertedItem.getAsJsonPrimitive("id").getAsString();
```
### <a name="json_delete"></a>在不具型別的資料表中進行刪除
下列程式碼將說明如何刪除執行個體 (在此案例中，即為在前述 **插入** 範例中建立的同一個 *JsonObject* 執行個體)。 程式碼與典型案例相同，但方法具有不同的簽章，因為它會參考 **JsonObject**。

```java
mToDoTable
    .delete(insertedItem);
```

您也可以直接使用 ID 來刪除執行個體：

```java
mToDoTable.delete(ID);
```

### <a name="json_get"></a>從不具型別的資料表傳回所有資料列
下列程式碼將說明如何擷取整個資料表。 由於您使用 JSON 資料表，因此可以選擇性地只擷取資料表的某些資料行。

```java
public void showAllUntyped(View view) {
    new AsyncTask<Void, Void, Void>() {
        @Override
        protected Void doInBackground(Void... params) {
            try {
                final JsonElement result = mJsonToDoTable.execute().get();
                final JsonArray results = result.getAsJsonArray();
                runOnUiThread(new Runnable() {

                    @Override
                    public void run() {
                        mAdapter.clear();
                        for (JsonElement item : results) {
                            String ID = item.getAsJsonObject().getAsJsonPrimitive("id").getAsString();
                            String mText = item.getAsJsonObject().getAsJsonPrimitive("text").getAsString();
                            Boolean mComplete = item.getAsJsonObject().getAsJsonPrimitive("complete").getAsBoolean();
                            ToDoItem mToDoItem = new ToDoItem();
                            mToDoItem.setId(ID);
                            mToDoItem.setText(mText);
                            mToDoItem.setComplete(mComplete);
                            mAdapter.add(mToDoItem);
                        }
                    }
                });
            } catch (Exception exception) {
                createAndShowDialog(exception, "Error");
            }
            return null;
        }
    }.execute();
}
```

具型別的模型可使用的同一組篩選、篩選和分頁方法，可供不具型別的模型使用。

## <a name="offline-sync"></a>實作離線同步處理

Azure Mobile Apps 用戶端 SDK 也會使用 SQL Database 在本機儲存伺服器資料的副本，來實作離線資料同步處理。  離線資料表上執行的作業不需要行動連線能力。  離線同步處理可協助復原能力和效能，但代價是更複雜的衝突解決邏輯。  Mobile Apps 用戶端 SDK 會實作下列功能︰

* 增量同步處理︰只會下載更新和新的記錄，可節省頻寬和記憶體耗用量。
* 開放式並行存取︰作業會假設為成功。  會延後衝突解決，直到伺服器上執行更新為止。
* 衝突解決︰SDK 會偵測到伺服器上已進行的衝突變更，並提供攔截程序來警示使用者。
* 虛刪除︰已刪除的記錄會標示為已刪除，使其他裝置可更新其離線快取。

### <a name="initialize-offline-sync"></a>將離線同步處理初始化

必須在離線快取中定義每個離線資料表之後才能使用。  一般而言，用戶端建立後，會立即完成資料表定義︰

```java
AsyncTask<Void, Void, Void> initializeStore(MobileServiceClient mClient)
    throws MobileServiceLocalStoreException, ExecutionException, InterruptedException
{
    AsyncTask<Void, Void, Void> task = new AsyncTask<Void, Void, Void>() {
        @Override
        protected void doInBackground(Void... params) {
            try {
                MobileServiceSyncContext syncContext = mClient.getSyncContext();
                if (syncContext.isInitialized()) {
                    return null;
                }
                SQLiteLocalStore localStore = new SQLiteLocalStore(mClient.getContext(), "offlineStore", null, 1);

                // Create a table definition.  As a best practice, store this with the model definition and return it via
                // a static method
                Map<String, ColumnDataType> toDoItemDefinition = new HashMap<String, ColumnDataType>();
                toDoItemDefinition.put("id", ColumnDataType.String);
                toDoItemDefinition.put("complete", ColumnDataType.Boolean);
                toDoItemDefinition.put("text", ColumnDataType.String);
                toDoItemDefinition.put("version", ColumnDataType.String);
                toDoItemDefinition.put("updatedAt", ColumnDataType.DateTimeOffset);

                // Now define the table in the local store
                localStore.defineTable("ToDoItem", toDoItemDefinition);

                // Specify a sync handler for conflict resolution
                SimpleSyncHandler handler = new SimpleSyncHandler();

                // Initialize the local store
                syncContext.initialize(localStore, handler).get();
            } catch (final Exception e) {
                createAndShowDialogFromTask(e, "Error");
            }
            return null;
        }
    };
    return runAsyncTask(task);
}
```

### <a name="obtain-a-reference-to-the-offline-cache-table"></a>取得離線快取資料表的參考

如需線上資料表，請使用 `.getTable()`。  如需離線資料表，請使用 `.getSyncTable()`：

```java
MobileServiceSyncTable<ToDoItem> mToDoTable = mClient.getSyncTable("ToDoItem", ToDoItem.class);
```

可供線上資料表 (包括篩選、排序、分頁、插入資料、更新資料和刪除資料) 使用的所有方法，在線上或離線資料表上都可正常運作。

### <a name="synchronize-the-local-offline-cache"></a>同步處理本機離線快取

可在您的應用程式內控制同步處理。  以下是同步處理方法的範例︰

```java
private AsyncTask<Void, Void, Void> sync(MobileServiceClient mClient) {
    AsyncTask<Void, Void, Void> task = new AsyncTask<Void, Void, Void>(){
        @Override
        protected Void doInBackground(Void... params) {
            try {
                MobileServiceSyncContext syncContext = mClient.getSyncContext();
                syncContext.push().get();
                mToDoTable.pull(null, "todoitem").get();
            } catch (final Exception e) {
                createAndShowDialogFromTask(e, "Error");
            }
            return null;
        }
    };
    return runAsyncTask(task);
}
```

如果將查詢名稱提供給 `.pull(query, queryname)` 方法，則使用增量同步處理所傳回的記錄，只有自上次成功完成提取後所建立或變更的記錄。

### <a name="handle-conflicts-during-offline-synchronization"></a>處理離線同步處理期間的衝突

如果在 `.push()` 作業期間發生衝突，就會擲回 `MobileServiceConflictException`。   伺服器所發行的項目內嵌於例外狀況，而且可以由 `.getItem()` 例外狀況進行擷取。  在 MobileServiceSyncContext 物件上呼叫下列項目來調整推送︰

*  `.cancelAndDiscardItem()`
*  `.cancelAndUpdateItem()`
*  `.updateOperationAndItem()`

一旦所有衝突都如您想要的方式標記後，再次呼叫 `.push()` 來解決所有衝突。

## <a name="custom-api"></a>呼叫自訂 API

自訂 API 可讓您定義自訂端點，並用來公開無法對應插入、更新、刪除或讀取等操作的伺服器功能。 透過使用自訂 API，您可以進一步控制訊息，包括讀取與設定 HTTP 訊息標頭，並定義除了 JSON 以外的訊息內文格式。

從 Android 用戶端呼叫 **invokeApi** 方法，以呼叫自訂 API 端點。 下列範例示範如何呼叫名為 **completeAll** 的 API 端點，它會傳回名為 **MarkAllResult** 的集合類別。

```java
public void completeItem(View view) {
    ListenableFuture<MarkAllResult> result = mClient.invokeApi("completeAll", MarkAllResult.class);
    Futures.addCallback(result, new FutureCallback<MarkAllResult>() {
        @Override
        public void onFailure(Throwable exc) {
            createAndShowDialog((Exception) exc, "Error");
        }

        @Override
        public void onSuccess(MarkAllResult result) {
            createAndShowDialog(result.getCount() + " item(s) marked as complete.", "Completed Items");
            refreshItemsFromTable();
        }
    });
}
```

**invokeApi** 方法是在用戶端上呼叫，可將 POST 要求傳送給新的自訂 API。 如有任何錯誤，自訂 API 傳回的結果會顯示在訊息對話方塊中。 其他版本的 **invokeApi** 可讓您選擇性地在要求主體中傳送物件、指定 HTTP 方法，並隨著要求一起傳送查詢參數。 也會提供不具型別的版本 **invokeApi** 。

## <a name="authentication"></a>在您的應用程式中新增驗證

教學課程已詳細說明如何新增這些功能。

App Service 支援使用各種外部識別提供者 (Facebook、Google、Microsoft 帳戶、Twitter 以及 Azure Active Directory) 來 [驗證應用程式使用者](app-service-mobile-android-get-started-users.md)。 您可以在資料表上設定權限，以限制僅有通過驗證使用者可以存取特定操作。 您也可以使用經驗證使用者的身分識別，以在後端實作授權規則。

支援兩個驗證流程：**伺服器**流程和**用戶端**流程。 由於伺服器流程採用識別提供者的 Web 介面，因此所提供的驗證體驗也最為簡單。  不需要其他 SDK 就能實作伺服器流程驗證。 伺服器流程驗證不會與行動裝置密切整合，因此只建議用於概念證明案例。

用戶端流程依賴識別提供者提供的 SDK，因此可以與裝置特有的功能深入整合，例如單一登入。  例如，您可以將 Facebook SDK 整合到行動應用程式。  行動用戶端會切換到 Facebook 應用程式，並確認您已登入再切換回行動應用程式。

要在您的應用程式中啟用驗證，必須執行四個步驟：

* 對識別提供者註冊應用程式以進行驗證。
* 設定 App Service 後端。
* 限制只有 App Service 後端上經驗證的使用者會具有資料表權限。
* 將驗證碼新增至您的應用程式。

您可以在資料表上設定權限，以限制僅有通過驗證使用者可以存取特定操作。 您也可以使用已驗證的使用者 SID 來修改要求。  如需詳細資訊，請檢閱 [開始使用驗證] 和伺服器 SDK 做法文件。

### <a name="caching"></a>驗證︰伺服器流程

下列程式碼會使用 Google 提供者來啟動伺服器流程登入程序。  由於 Google 提供者的安全性需求，需要其他設定︰

```java
MobileServiceUser user = mClient.login(MobileServiceAuthenticationProvider.Google, "{url_scheme_of_your_app}", GOOGLE_LOGIN_REQUEST_CODE);
```

此外，將下列方法新增至主要活動類別︰

```java
// You can choose any unique number here to differentiate auth providers from each other. Note this is the same code at login() and onActivityResult().
public static final int GOOGLE_LOGIN_REQUEST_CODE = 1;

@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    // When request completes
    if (resultCode == RESULT_OK) {
        // Check the request code matches the one we send in the login request
        if (requestCode == GOOGLE_LOGIN_REQUEST_CODE) {
            MobileServiceActivityResult result = mClient.onActivityResult(data);
            if (result.isLoggedIn()) {
                // login succeeded
                createAndShowDialog(String.format("You are now logged in - %1$2s", mClient.getCurrentUser().getUserId()), "Success");
                createTable();
            } else {
                // login failed, check the error message
                String errorMessage = result.getErrorMessage();
                createAndShowDialog(errorMessage, "Error");
            }
        }
    }
}
```

在您主要活動中定義的 `GOOGLE_LOGIN_REQUEST_CODE` 會用於 `login()` 方法，以及 `onActivityResult()` 方法內。  只要 `login()` 方法和 `onActivityResult()` 方法內都使用相同的數字，您就可以選擇任何唯一的數字。  如果您將用戶端程式碼服務擷取至服務配接器 (如先前所示)，應該在服務配接器上呼叫適當的方法。

您也需要設定 customtabs 的專案。  先指定重新導向 URL。  將下列程式碼片段新增至 `AndroidManifest.xml`：

```xml
<activity android:name="com.microsoft.windowsazure.mobileservices.authentication.RedirectUrlActivity">
    <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data android:scheme="{url_scheme_of_your_app}" android:host="easyauth.callback"/>
    </intent-filter>
</activity>
```

將 **redirectUriScheme** 新增至應用程式的 `build.gradle` 檔案︰

```text
android {
    buildTypes {
        release {
            // … …
            manifestPlaceholders = ['redirectUriScheme': '{url_scheme_of_your_app}://easyauth.callback']
        }
        debug {
            // … …
            manifestPlaceholders = ['redirectUriScheme': '{url_scheme_of_your_app}://easyauth.callback']
        }
    }
}
```

最後，將 `com.android.support:customtabs:23.0.1` 新增至 `build.gradle` 檔案中的相依性清單︰

```text
dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    compile 'com.google.code.gson:gson:2.3'
    compile 'com.google.guava:guava:18.0'
    compile 'com.android.support:customtabs:23.0.1'
    compile 'com.squareup.okhttp:okhttp:2.5.0'
    compile 'com.microsoft.azure:azure-mobile-android:3.4.0@aar'
    compile 'com.microsoft.azure:azure-notifications-handler:1.0.1@jar'
}
```

使用 **getUserId** 方法從 **MobileServiceUser** 取得已登入使用者的識別碼。 如需如何使用 Futures 來呼叫非同步登入 API 的範例，請參閱 [開始使用驗證]。

> [!WARNING]
> 所述的 URL 配置會區分大小寫。  確認所有出現的 `{url_scheme_of_you_app}` 大小寫相符。

### <a name="caching"></a>快取驗證權杖

您必須將使用者 ID 和驗證語彙基元儲存在本機裝置上，才能快取驗證語彙基元。 當應用程式下次啟動時，您只需確認這些值仍存在於快取中，即可略過登入程序，並使用這項資料還原用戶端。 但這項資料具有敏感性，因此應加密儲存，以確保在手機失竊的狀況下仍保有安全性。  您可以在[快取驗證權杖一節][7]中看到如何快取驗證權杖的完整範例。

當您嘗試使用到期的權杖時，您會收到「401 未授權」  的回應。 您可以使用篩選器處理驗證錯誤。  篩選器會攔截對 App Service 後端提出的要求。 篩選器程式碼會測試 401 的回應、觸發登入程序，然後繼續執行產生 401 的要求。

### <a name="refresh"></a>使用重新整理權杖

Azure App Service 驗證與授權傳回的權杖都有一小時的定義存留時間。  在這段期間之後，您必須重新驗證使用者。  如果您使用的是透過用戶端流程驗證收到的長時間執行的權杖，您可以使用相同的權杖，透過 Azure App Service 驗證與授權進行重新驗證。  所產生的另一個 Azure App Service 權杖會含有新的存留期。

您也可以註冊提供者來使用重新整理權杖。  重新整理權杖並非一律可用。  需要進行其他設定：

* 對於 **Azure Active Directory**，請設定 Azure Active Directory 應用程式的用戶端祕密。  設定 Azure Active Directory 驗證時，請在 Azure App Service 中指定用戶端祕密。  呼叫 `.login()` 時，傳遞 `response_type=code id_token` 作為參數︰

    ```java
    HashMap<String, String> parameters = new HashMap<String, String>();
    parameters.put("response_type", "code id_token");
    MobileServiceUser user = mClient.login
        MobileServiceAuthenticationProvider.AzureActiveDirectory,
        "{url_scheme_of_your_app}",
        AAD_LOGIN_REQUEST_CODE,
        parameters);
    ```

* 若為 **Google**，請傳遞 `access_type=offline` 作為參數︰

    ```java
    HashMap<String, String> parameters = new HashMap<String, String>();
    parameters.put("access_type", "offline");
    MobileServiceUser user = mClient.login
        MobileServiceAuthenticationProvider.Google,
        "{url_scheme_of_your_app}",
        GOOGLE_LOGIN_REQUEST_CODE,
        parameters);
    ```

* 若為 **Microsoft 帳戶**，請選取 `wl.offline_access` 範圍。

若要重新整理權杖，請呼叫 `.refreshUser()`：

```java
MobileServiceUser user = mClient
    .refreshUser()  // async - returns a ListenableFuture<MobileServiceUser>
    .get();
```

最佳做法是，建立篩選條件偵測來自伺服器的 401 回應，並嘗試重新整理使用者權杖。

## <a name="log-in-with-client-flow-authentication"></a>使用用戶端流程驗證登入

使用用戶端流程驗證登入的一般程序如下所示︰

* 以您設定伺服器流程驗證的相同方式設定 Azure App Service 驗證與授權。
* 將驗證提供者 SDK 進行整合以供驗證，來產生存取權杖。
* 呼叫 `.login()` 方法，如下所示：

    ```java
    JSONObject payload = new JSONObject();
    payload.put("access_token", result.getAccessToken());
    ListenableFuture<MobileServiceUser> mLogin = mClient.login("{provider}", payload.toString());
    Futures.addCallback(mLogin, new FutureCallback<MobileServiceUser>() {
        @Override
        public void onFailure(Throwable exc) {
            exc.printStackTrace();
        }
        @Override
        public void onSuccess(MobileServiceUser user) {
            Log.d(TAG, "Login Complete");
        }
    });
    ```

將 `onSuccess()` 方法取代為您想要用於成功登入的任何程式碼。  `{provider}` 字串是有效的提供者︰**aad** (Azure Active Directory)、**facebook**、**google**、**microsoftaccount** 或 **twitter**。  如果您已實作自訂驗證，則也可以使用自訂驗證提供者的標記。

### <a name="adal"></a>使用 Active Directory Authentication Library (ADAL) 驗證使用者

您可以使用 Active Directory Authentication Library (ADAL)，利用 Azure Active Directory 將使用者登入應用程式。 使用用戶端流程登入通常會比使用 `loginAsync()` 方法還適合，因為它提供更原生的 UX 風格，並允許其他自訂。

1. 依照[如何設定 App Service 來進行 Active Directory 登入][22]教學課程的說明，設定您的行動應用程式後端來進行 AAD 登入。 請務必完成註冊原生用戶端應用程式的選擇性步驟。
2. 安裝 ADAL，方法是修改您的 build.gradle 檔案以納入下列定義：

```
repositories {
    mavenCentral()
    flatDir {
        dirs 'libs'
    }
    maven {
        url "YourLocalMavenRepoPath\\.m2\\repository"
    }
}
packagingOptions {
    exclude 'META-INF/MSFTSIG.RSA'
    exclude 'META-INF/MSFTSIG.SF'
}
dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    compile('com.microsoft.aad:adal:1.1.1') {
        exclude group: 'com.android.support'
    } // Recent version is 1.1.1
    compile 'com.android.support:support-v4:23.0.0'
}
```

1. 將下列程式碼新增至您的應用程式，進行下列取代：

* 以您佈建應用程式的租用戶名稱取代 **INSERT-AUTHORITY-HERE** 。 格式應為 https://login.microsoftonline.com/contoso.onmicrosoft.com。
* 以您行動應用程式後端的用戶端識別碼取代 INSERT-RESOURCE-ID-HERE  。 您可以從入口網站 [Azure Active Directory 設定] 底下的 [進階] 索引標籤取得用戶端識別碼。
* 以您從原生用戶端應用程式中複製的用戶端識別碼取代 INSERT-CLIENT-ID-HERE  。
* 使用 HTTPS 配置，以您網站的 **/.auth/login/done** 端點取代 *INSERT-REDIRECT-URI-HERE* 。 此值應該類似 https://contoso.azurewebsites.net/.auth/login/done。

```java
private AuthenticationContext mContext;

private void authenticate() {
    String authority = "INSERT-AUTHORITY-HERE";
    String resourceId = "INSERT-RESOURCE-ID-HERE";
    String clientId = "INSERT-CLIENT-ID-HERE";
    String redirectUri = "INSERT-REDIRECT-URI-HERE";
    try {
        mContext = new AuthenticationContext(this, authority, true);
        mContext.acquireToken(this, resourceId, clientId, redirectUri, PromptBehavior.Auto, "", callback);
    } catch (Exception exc) {
        exc.printStackTrace();
    }
}

private AuthenticationCallback<AuthenticationResult> callback = new AuthenticationCallback<AuthenticationResult>() {
    @Override
    public void onError(Exception exc) {
        if (exc instanceof AuthenticationException) {
            Log.d(TAG, "Cancelled");
        } else {
            Log.d(TAG, "Authentication error:" + exc.getMessage());
        }
    }

    @Override
    public void onSuccess(AuthenticationResult result) {
        if (result == null || result.getAccessToken() == null
                || result.getAccessToken().isEmpty()) {
            Log.d(TAG, "Token is empty");
        } else {
            try {
                JSONObject payload = new JSONObject();
                payload.put("access_token", result.getAccessToken());
                ListenableFuture<MobileServiceUser> mLogin = mClient.login("aad", payload.toString());
                Futures.addCallback(mLogin, new FutureCallback<MobileServiceUser>() {
                    @Override
                    public void onFailure(Throwable exc) {
                        exc.printStackTrace();
                    }
                    @Override
                    public void onSuccess(MobileServiceUser user) {
                        Log.d(TAG, "Login Complete");
                    }
                });
            }
            catch (Exception exc){
                Log.d(TAG, "Authentication error:" + exc.getMessage());
            }
        }
    }
};

@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    super.onActivityResult(requestCode, resultCode, data);
    if (mContext != null) {
        mContext.onActivityResult(requestCode, resultCode, data);
    }
}
```

## <a name="filters"></a>調整用戶端與伺服器通訊

用戶端連線通常是使用隨 Android SDK 提供之基礎 HTTP 程式庫的基本 HTTP 連線。  有幾個原因使您會想加以變更︰

* 您想要使用替代的 HTTP 程式庫來調整逾時。
* 您想要提供進度列。
* 您想要新增自訂標頭，以支援 API 管理功能。
* 您想要攔截失敗的回應，以便您可以實作重新驗證。
* 您想要記錄分析服務的後端要求。

### <a name="using-an-alternate-http-library"></a>使用替代的 HTTP 程式庫

建立用戶端參考之後，立即呼叫 `.setAndroidHttpClientFactory()` 方法。  例如，若要將連線逾時設定為 60 秒 (而不是預設值 10 秒)︰

```java
mClient = new MobileServiceClient("https://myappname.azurewebsites.net");
mClient.setAndroidHttpClientFactory(new OkHttpClientFactory() {
    @Override
    public OkHttpClient createOkHttpClient() {
        OkHttpClient client = new OkHttpClinet();
        client.setReadTimeout(60, TimeUnit.SECONDS);
        client.setWriteTimeout(60, TimeUnit.SECONDS);
        return client;
    }
});
```

### <a name="implement-a-progress-filter"></a>實作進度篩選條件

您可以藉由實作 `ServiceFilter` 來實作每個要求的攔截。  例如，下列作業會更新預先建立的進度列︰

```java
private class ProgressFilter implements ServiceFilter {
    @Override
    public ListenableFuture<ServiceFilterResponse> handleRequest(ServiceFilterRequest request, NextServiceFilterCallback next) {
        final SettableFuture<ServiceFilterResponse> resultFuture = SettableFuture.create();
        runOnUiThread(new Runnable() {
            @Override
            public void run() {
                if (mProgressBar != null) mProgressBar.setVisibility(ProgressBar.VISIBLE);
            }
        });

        ListenableFuture<ServiceFilterResponse> future = next.onNext(request);
        Futures.addCallback(future, new FutureCallback<ServiceFilterResponse>() {
            @Override
            public void onFailure(Throwable e) {
                resultFuture.setException(e);
            }
            @Override
            public void onSuccess(ServiceFilterResponse response) {
                runOnUiThread(new Runnable() {
                    @Override
                    pubic void run() {
                        if (mProgressBar != null)
                            mProgressBar.setVisibility(ProgressBar.GONE);
                    }
                });
                resultFuture.set(response);
            }
        });
        return resultFuture;
    }
}
```

您可以將此篩選條件附加至用戶端，如下所示︰

```java
mClient = new MobileServiceClient(applicationUrl).withFilter(new ProgressFilter());
```

### <a name="customize-request-headers"></a>自訂要求標頭

使用下列 `ServiceFilter`，並使用與 `ProgressFilter` 相同的方式附加篩選條件：

```java
private class CustomHeaderFilter implements ServiceFilter {
    @Override
    public ListenableFuture<ServiceFilterResponse> handleRequest(ServiceFilterRequest request, NextServiceFilterCallback next) {
        runOnUiThread(new Runnable() {
            @Override
            public void run() {
                request.addHeader("X-APIM-Router", "mobileBackend");
            }
        });
        SettableFuture<ServiceFilterResponse> result = SettableFuture.create();
        try {
            ServiceFilterResponse response = next.onNext(request).get();
            result.set(response);
        } catch (Exception exc) {
            result.setException(exc);
        }
    }
}
```

### <a name="conversions"></a>設定自動序列化

您可以使用 [gson][3] API 指定套用至每個資料行的轉換策略。 Android 用戶端程式庫會先在背景使用 [gson][3] 來將 Java 物件序列化為 JSON 資料，再將資料傳送至 Azure App Service。  下列程式碼使用 **setFieldNamingStrategy()** 方法來設定策略。 此範例會刪除每個欄位名稱的起始字元 ("m")，然後將下一個字元轉換為小寫。 比方說，它會將「mId」變成「id」。  實作轉換策略，以減少大部分欄位上的 `SerializedName()` 註釋需要。

```java
FieldNamingStrategy namingStrategy = new FieldNamingStrategy() {
    public String translateName(File field) {
        String name = field.getName();
        return Character.toLowerCase(name.charAt(1)) + name.substring(2);
    }
}

client.setGsonBuilder(
    MobileServiceClient
        .createMobileServiceGsonBuilder()
        .setFieldNamingStrategy(namingStategy)
);
```

必須先執行此程式碼，才能使用 **MobileServiceClient** 來建立行動用戶端參考。

<!-- URLs. -->
[Get started with Azure Mobile Apps]: app-service-mobile-android-get-started.md
[ASCII control codes C0 and C1]: http://en.wikipedia.org/wiki/Data_link_escape_character#C1_set
[Mobile Services SDK for Android]: http://go.microsoft.com/fwlink/p/?LinkID=717033
[Azure portal]: https://portal.azure.com
[開始使用驗證]: app-service-mobile-android-get-started-users.md
[1]: https://static.javadoc.io/com.google.code.gson/gson/2.8.5/com/google/gson/JsonObject.html
[2]: http://hashtagfail.com/post/44606137082/mobile-services-android-serialization-gson
[3]: https://www.javadoc.io/doc/com.google.code.gson/gson/2.8.5
[4]: http://go.microsoft.com/fwlink/p/?LinkId=296840
[5]: app-service-mobile-android-get-started-push.md
[6]: ../notification-hubs/notification-hubs-push-notification-overview.md#integration-with-app-service-mobile-apps
[7]: app-service-mobile-android-get-started-users.md#cache-tokens
[8]: http://azure.github.io/azure-mobile-apps-android-client/com/microsoft/windowsazure/mobileservices/table/MobileServiceTable.html
[9]: http://azure.github.io/azure-mobile-apps-android-client/com/microsoft/windowsazure/mobileservices/MobileServiceClient.html
[10]: app-service-mobile-dotnet-backend-how-to-use-server-sdk.md
[11]: app-service-mobile-node-backend-how-to-use-server-sdk.md
[12]: http://azure.github.io/azure-mobile-apps-android-client/
[13]: app-service-mobile-android-get-started.md#create-a-new-azure-mobile-app-backend
[14]: http://go.microsoft.com/fwlink/p/?LinkID=717034
[15]: app-service-mobile-dotnet-backend-how-to-use-server-sdk.md#define-table-controller
[16]: app-service-mobile-node-backend-how-to-use-server-sdk.md#TableOperations
[17]: https://developer.android.com/reference/java/util/UUID.html
[18]: https://github.com/google/guava/wiki/ListenableFutureExplained
[19]: http://www.odata.org/documentation/odata-version-3-0/
[20]: http://hashtagfail.com/post/46493261719/mobile-services-android-querying
[21]: https://github.com/Azure-Samples/azure-mobile-apps-android-quickstart
[22]: ../app-service/app-service-mobile-how-to-configure-active-directory-authentication.md
[Future]: http://developer.android.com/reference/java/util/concurrent/Future.html
[AsyncTask]: http://developer.android.com/reference/android/os/AsyncTask.html
