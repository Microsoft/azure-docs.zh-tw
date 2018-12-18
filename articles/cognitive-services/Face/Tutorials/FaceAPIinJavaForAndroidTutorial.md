---
title: 教學課程：偵測並框出影像中的人臉 - 臉部 API、適用於 Android 的 Java
titleSuffix: Azure Cognitive Services
description: 在此教學課程中，您會建立簡單的 Android 應用程式，以使用臉部 API 來偵測並框出影像中的人臉。
services: cognitive-services
author: PatrickFarley
manager: cgronlun
ms.service: cognitive-services
ms.component: face-api
ms.topic: tutorial
ms.date: 07/12/2018
ms.author: pafarley
ms.openlocfilehash: 08344e21d4f425a021bdefe840390ede8b3fb01e
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49342376"
---
# <a name="tutorial-create-an-android-app-to-detect-and-frame-faces-in-an-image"></a>教學課程：建立 Android 應用程式來偵測並框出影像中的臉部

在本教學課程中，您會建立簡單的 Android 應用程式，以使用臉部服務 Java 類別庫來偵測影像中的人臉。 此應用程式會顯示選取的影像，且所偵測到的每個臉部都會以矩形框住。 GitHub 上的[偵測 Android 中的影像並框出人臉](https://github.com/Azure-Samples/cognitive-services-face-android-sample)中會提供完整的程式碼範例。

![相片中的臉部以紅色矩形框住的 Android 螢幕擷取畫面](../Images/android_getstarted2.1.PNG)

本教學課程說明如何：

> [!div class="checklist"]
> - 建立 Android 應用程式
> - 安裝臉部服務用戶端程式庫
> - 使用用戶端程式庫來偵測影像中的臉部
> - 在所偵測到的每個臉部周圍繪出邊框

## <a name="prerequisites"></a>必要條件

- 您需要有訂用帳戶金鑰才能執行範例。 您可以從[試用認知服務](https://azure.microsoft.com/try/cognitive-services/?api=face-api)取得免費的試用訂用帳戶金鑰。
- [Android Studio](https://developer.android.com/studio/)，且至少具有 SDK 22 (臉部用戶端程式庫所需)。
- 來自 Maven 的 [Com.microsoft.projectoxford:face:1.4.3](http://search.maven.org/#search%7Cga%7C1%7Cg%3A%22com.microsoft.projectoxford%22) 臉部用戶端程式庫。 您不必下載此套件。 下面會提供安裝指示。

## <a name="create-the-project"></a>建立專案

請遵循下列步驟來建立 Android 應用程式專案：

1. 開啟 Android Studio。 本教學課程使用 Android Studio 3.1。
1. 選取 [啟動新的 Android Studio 專案]。
1. 在 [建立 Android 專案] 畫面上，如有必要可修改預設欄位，然後按 [下一步]。
1. 在 [目標 Android 裝置] 畫面上，使用下拉式選取器選擇 [API 22] 或更高版本，然後按 [下一步]。
1. 選取 [空白活動]，然後按 [下一步]。
1. 取消核取 [回溯相容性]，然後按一下 [完成]。

## <a name="create-the-ui-for-selecting-and-displaying-the-image"></a>建立用於選取和顯示影像的 UI

開啟 activity_main.xml；您應該會看到配置編輯器。 選取 [文字] 索引標籤，然後將其中的內容取代為下列程式碼。

```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <ImageView
        android:layout_width="match_parent"
        android:layout_height="fill_parent"
        android:id="@+id/imageView1"
        android:layout_above="@+id/button1"
        android:contentDescription="Image with faces to analyze"/>

    <Button
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Browse for face image"
        android:id="@+id/button1"
        android:layout_alignParentBottom="true"/>
</RelativeLayout>
```

開啟 MainActivity.java，然後將所有內容 (第一個 `package` 陳述式除外) 取代為下列程式碼。

此程式碼會對 `Button` 設定事件處理常式，以啟動新的活動讓使用者可以選取圖片。 圖片一經選取就會顯示在 `ImageView` 中。

```java
import java.io.*;
import android.app.*;
import android.content.*;
import android.net.*;
import android.os.*;
import android.view.*;
import android.graphics.*;
import android.widget.*;
import android.provider.*;

public class MainActivity extends Activity {
    private final int PICK_IMAGE = 1;
    private ProgressDialog detectionProgressDialog;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_main);
            Button button1 = (Button)findViewById(R.id.button1);
            button1.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                Intent intent = new Intent(Intent.ACTION_GET_CONTENT);
                intent.setType("image/*");
                startActivityForResult(Intent.createChooser(
                        intent, "Select Picture"), PICK_IMAGE);
            }
        });

        detectionProgressDialog = new ProgressDialog(this);
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        if (requestCode == PICK_IMAGE && resultCode == RESULT_OK &&
                data != null && data.getData() != null) {
            Uri uri = data.getData();
            try {
                Bitmap bitmap = MediaStore.Images.Media.getBitmap(
                        getContentResolver(), uri);
                ImageView imageView = (ImageView) findViewById(R.id.imageView1);
                imageView.setImageBitmap(bitmap);

                // Uncomment
                //detectAndFrame(bitmap);
                } catch (IOException e) {
                    e.printStackTrace();
                }
        }
    }
}
```

應用程式現在可以瀏覽相片，並在與下圖類似的視窗中顯示該相片。

![相片中具有臉部的 Android 螢幕擷取畫面](../Images/android_getstarted1.1.PNG)

## <a name="configure-the-face-client-library"></a>設定臉部用戶端程式庫

臉部 API 是您可使用 HTTPS 要求所呼叫的雲端 API。 本教學課程會使用臉部用戶端程式庫，由其封裝這些 Web 要求以簡化您的工作。

在 [專案] 窗格中，使用下拉式選取器來選取 [Android]。 展開 [Gradle 指令碼]，然後開啟 build.gradle (模組：應用程式)。

為臉部用戶端程式庫 `com.microsoft.projectoxford:face:1.4.3` 新增相依性 (如以下螢幕擷取畫面所示)，然後按一下 [立即同步]。

![build.gradle 應用程式檔案的 Android Studio 螢幕擷取畫面](../Images/face-tut-java-gradle.png)

開啟 **MainActivity.java**，然後附加下列 import 指示詞：

```java
import com.microsoft.projectoxford.face.*;
import com.microsoft.projectoxford.face.contract.*;
```

## <a name="add-the-face-client-library-code"></a>新增臉部用戶端程式庫程式碼

在 `MainActivity` 類別的 `onCreate` 方法上方插入下列程式碼：

```java
private final String apiEndpoint = "<API endpoint>";
private final String subscriptionKey = "<Subscription Key>";

private final FaceServiceClient faceServiceClient =
        new FaceServiceRestClient(apiEndpoint, subscriptionKey);
```

將 `<API endpoint>` 取代為指派給金鑰的 API 端點。 **westcentralus** 區域會產生免費試用的訂用帳戶金鑰。 因此，如果您使用免費試用的訂用帳戶金鑰，陳述式會是：

```java
apiEndpoint = "https://westcentralus.api.cognitive.microsoft.com/face/v1.0";
```

將 `<Subscription Key>` 取代為訂用帳戶金鑰。 例如︰

```java
subscriptionKey = "0123456789abcdef0123456789ABCDEF"
```

在 [專案] 窗格中，依序展開 [應用程式] 和 [資訊清單]，然後開啟 AndroidManifest.xml。

將下列元素插入為 `manifest` 元素的直接子系：

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

建置專案以檢查是否有錯誤。 現在，您已準備好呼叫臉部服務。

## <a name="upload-an-image-to-detect-faces"></a>上傳影像來偵測臉部

若要偵測臉部，最簡單的方式是呼叫 `FaceServiceClient.detect` 方法。 這個方法會包裝[偵測](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236) API 方法，並傳回 `Face` 的陣列。

每個傳回的 `Face` 都會包含矩形來指出其位置，以及一系列的選擇性臉部屬性。 此範例只需要臉部位置。

如果發生錯誤，`AlertDialog` 會顯示根本原因。

將下列方法插入 `MainActivity` 類別中。

```java
// Detect faces by uploading a face image.
// Frame faces after detection.
private void detectAndFrame(final Bitmap imageBitmap) {
    ByteArrayOutputStream outputStream = new ByteArrayOutputStream();
    imageBitmap.compress(Bitmap.CompressFormat.JPEG, 100, outputStream);
    ByteArrayInputStream inputStream =
            new ByteArrayInputStream(outputStream.toByteArray());

    AsyncTask<InputStream, String, Face[]> detectTask =
            new AsyncTask<InputStream, String, Face[]>() {
                String exceptionMessage = "";

                @Override
                protected Face[] doInBackground(InputStream... params) {
                    try {
                        publishProgress("Detecting...");
                        Face[] result = faceServiceClient.detect(
                                params[0],
                                true,         // returnFaceId
                                false,        // returnFaceLandmarks
                                null          // returnFaceAttributes:
                                /* new FaceServiceClient.FaceAttributeType[] {
                                    FaceServiceClient.FaceAttributeType.Age,
                                    FaceServiceClient.FaceAttributeType.Gender }
                                */
                        );
                        if (result == null){
                            publishProgress(
                                    "Detection Finished. Nothing detected");
                            return null;
                        }
                        publishProgress(String.format(
                                "Detection Finished. %d face(s) detected",
                                result.length));
                        return result;
                    } catch (Exception e) {
                        exceptionMessage = String.format(
                                "Detection failed: %s", e.getMessage());
                        return null;
                    }
                }

                @Override
                protected void onPreExecute() {
                    //TODO: show progress dialog
                }
                @Override
                protected void onProgressUpdate(String... progress) {
                    //TODO: update progress
                }
                @Override
                protected void onPostExecute(Face[] result) {
                    //TODO: update face frames
                }
            };

    detectTask.execute(inputStream);
}

private void showError(String message) {
    new AlertDialog.Builder(this)
    .setTitle("Error")
    .setMessage(message)
    .setPositiveButton("OK", new DialogInterface.OnClickListener() {
            public void onClick(DialogInterface dialog, int id) {
        }})
    .create().show();
}
```

## <a name="frame-faces-in-the-image"></a>框出影像中的臉部

將下列協助程式方法插入 `MainActivity` 類別中。 此方法會在所偵測到的每個臉部周圍繪製矩形。

```java
private static Bitmap drawFaceRectanglesOnBitmap(
        Bitmap originalBitmap, Face[] faces) {
    Bitmap bitmap = originalBitmap.copy(Bitmap.Config.ARGB_8888, true);
    Canvas canvas = new Canvas(bitmap);
    Paint paint = new Paint();
    paint.setAntiAlias(true);
    paint.setStyle(Paint.Style.STROKE);
    paint.setColor(Color.RED);
    paint.setStrokeWidth(10);
    if (faces != null) {
        for (Face face : faces) {
            FaceRectangle faceRectangle = face.faceRectangle;
            canvas.drawRect(
                    faceRectangle.left,
                    faceRectangle.top,
                    faceRectangle.left + faceRectangle.width,
                    faceRectangle.top + faceRectangle.height,
                    paint);
        }
    }
    return bitmap;
}
```

在 `detectAndFrame` 方法中完成 `AsyncTask` 方法 (以 `TODO` 註解表示)。 成功時，便會在 `ImageView` 中顯示選取的影像，且其中的臉部已框起來。

```java
@Override
protected void onPreExecute() {
    detectionProgressDialog.show();
}
@Override
protected void onProgressUpdate(String... progress) {
    detectionProgressDialog.setMessage(progress[0]);
}
@Override
protected void onPostExecute(Face[] result) {
    detectionProgressDialog.dismiss();
    if(!exceptionMessage.equals("")){
        showError(exceptionMessage);
    }
    if (result == null) return;
    ImageView imageView = findViewById(R.id.imageView1);
    imageView.setImageBitmap(
            drawFaceRectanglesOnBitmap(imageBitmap, result));
    imageBitmap.recycle();
}
```

最後，在 `onActivityResult` 方法中，將 `detectAndFrame` 方法的呼叫取消註解，如下所示。

```java
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    super.onActivityResult(requestCode, resultCode, data);

    if (requestCode == PICK_IMAGE && resultCode == RESULT_OK &&
                data != null && data.getData() != null) {
        Uri uri = data.getData();
        try {
            Bitmap bitmap = MediaStore.Images.Media.getBitmap(
                    getContentResolver(), uri);
            ImageView imageView = findViewById(R.id.imageView1);
            imageView.setImageBitmap(bitmap);

            // Uncomment
            detectAndFrame(bitmap);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## <a name="run-the-app"></a>執行應用程式

執行此應用程式，然後瀏覽具有臉部的影像。 請等候幾秒，讓臉部服務回應。 之後，您就會取得與下圖類似的結果：

![GettingStartAndroid](../Images/android_getstarted2.1.PNG)

## <a name="summary"></a>總結

在本教學課程中，您已了解臉部服務的基本使用程序，並建立了應用程式來顯示影像中已框起的臉部。

## <a name="next-steps"></a>後續步驟

了解如何偵測和使用臉部特徵點。

> [!div class="nextstepaction"]
> [如何偵測影像中的人臉](../Face-API-How-to-Topics/HowtoDetectFacesinImage.md)

探索用來偵測臉部及其屬性 (例如，姿勢、性別、年齡、頭部姿勢、臉部汗毛和眼鏡) 的臉部 API。

> [!div class="nextstepaction"]
> [臉部 API 參考](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236)。