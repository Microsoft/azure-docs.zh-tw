---
title: 如何使用 Xamarin 的物件 (Blob) 儲存體 | Microsoft Docs
description: 適用於 Xamarin 的 Azure 儲存體用戶端程式庫可讓開發人員使用其原生的使用者介面建立 iOS、Android 和 Windows 市集應用程式。 本教學課程示範如何使用 Xamarin 來建立使用 Azure Blob 儲存體的應用程式。
services: storage
documentationcenter: xamarin
author: michaelhauss
ms.service: storage
ms.topic: article
ms.date: 05/11/2017
ms.author: michaelhauss
ms.openlocfilehash: b35bec31035c0219bf34a31cb34e20f7dc3a72c5
ms.sourcegitcommit: d4c076beea3a8d9e09c9d2f4a63428dc72dd9806
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/01/2018
ms.locfileid: "39397024"
---
# <a name="how-to-use-blob-storage-from-xamarin"></a>如何使用 Xamarin 的 Blob 儲存體

Xamarin 可讓開發人員使用共用的 C# 程式碼基底，使用其原生的使用者介面以建立 iOS、Android 和 Windows 市集應用程式。 本教學課程會示範如何搭配 Xamarin 應用程式使用 Azure Blob 儲存體。 如果您想要深入了解 Azure 儲存體，在一頭栽進程式碼之前，請先參閱 [Microsoft Azure 儲存體簡介](../common/storage-introduction.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)。

[!INCLUDE [storage-create-account-include](../../../includes/storage-create-account-include.md)]

[!INCLUDE [storage-mobile-authentication-guidance](../../../includes/storage-mobile-authentication-guidance.md)]

## <a name="create-a-new-xamarin-application"></a>建立新的 Xamarin 應用程式
針對此教學課程，我們將建立以 Android、iOS 和 Windows 為目標的應用程式。 此應用程式只會建立容器，並將 blob 上傳到此容器。 我們將在 Windows 上使用 Visual Studio，但是在 macOS 上使用 Xamarin Studio 建立應用程式時，可以套用相同的做法。

請遵循下列步驟來建立應用程式：

1. 下載並安裝 [Xamarin for Visual Studio](https://www.xamarin.com/download)(若您尚未這麼做)。
2. 開啟 Visual Studio，建立一個空的應用程式 (原生可攜式)︰[檔案] > [新增] > [專案] > [跨平台] > [空的應用程式 (原生可攜式)]。
3. 以滑鼠右鍵按一下 [方案總管] 窗格中的方案，然後選取 [管理方案的 NuGet 套件]。 搜尋 **WindowsAzure.Storage**，將最新穩定版安裝到您方案中的所有專案。
4. 建置並執行專案。

您現在應該有一個應用程式，在您按一下按鈕時就會遞增計數器。

## <a name="create-container-and-upload-blob"></a>建立容器並上傳 blob
接下來，在您的 `(Portable)` 專案之下，將一些程式碼加入 `MyClass.cs`。 此程式碼會建立一個容器，並上傳 Blob 到此容器中。 `MyClass.cs` 看起來應該如下所示︰

```csharp
using Microsoft.WindowsAzure.Storage;
using Microsoft.WindowsAzure.Storage.Blob;
using System.Threading.Tasks;

namespace XamarinApp
{
    public class MyClass
    {
        public MyClass ()
        {
        }

        public static async Task performBlobOperation()
        {
            // Retrieve storage account from connection string.
            CloudStorageAccount storageAccount = CloudStorageAccount.Parse("DefaultEndpointsProtocol=https;AccountName=your_account_name_here;AccountKey=your_account_key_here");

            // Create the blob client.
            CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

            // Retrieve reference to a previously created container.
            CloudBlobContainer container = blobClient.GetContainerReference("mycontainer");

            // Create the container if it doesn't already exist.
            await container.CreateIfNotExistsAsync();

            // Retrieve reference to a blob named "myblob".
            CloudBlockBlob blockBlob = container.GetBlockBlobReference("myblob");

            // Create the "myblob" blob with the text "Hello, world!"
            await blockBlob.UploadTextAsync("Hello, world!");
        }
    }
}
```

務必將 "your_account_name_here" 和 "your_account_key_here" 取代為您的實際帳戶名稱和帳戶金鑰。 

您的 iOS、Android 和 Windows Phone 專案都會有可攜式專案的參考，這表示您可以在單一位置撰寫所有的共用程式碼，而且可以在所有專案中使用該程式碼。 您現在可以將下列程式碼行加入每個專案，以開始利用：`MyClass.performBlobOperation()`

### <a name="xamarinappdroid--mainactivitycs"></a>XamarinApp.Droid > MainActivity.cs

```csharp
using Android.App;
using Android.Widget;
using Android.OS;

namespace XamarinApp.Droid
{
    [Activity (Label = "XamarinApp.Droid", MainLauncher = true, Icon = "@drawable/icon")]
    public class MainActivity : Activity
    {
        int count = 1;

        protected override async void OnCreate (Bundle bundle)
        {
            base.OnCreate (bundle);

            // Set our view from the "main" layout resource
            SetContentView (Resource.Layout.Main);

            // Get our button from the layout resource,
            // and attach an event to it
            Button button = FindViewById<Button> (Resource.Id.myButton);

            button.Click += delegate {
                button.Text = string.Format ("{0} clicks!", count++);
            };

            await MyClass.performBlobOperation();
            }
        }
    }
}
```

### <a name="xamarinappios--viewcontrollercs"></a>XamarinApp.iOS > ViewController.cs

```csharp
using System;
using UIKit;

namespace XamarinApp.iOS
{
    public partial class ViewController : UIViewController
    {
        int count = 1;

        public ViewController (IntPtr handle) : base (handle)
        {
        }

        public override async void ViewDidLoad ()
        {
            int count = 1;

            public ViewController (IntPtr handle) : base (handle)
            {
            }

            public override async void ViewDidLoad ()
            {
                base.ViewDidLoad ();
                // Perform any additional setup after loading the view, typically from a nib.
                Button.AccessibilityIdentifier = "myButton";
                Button.TouchUpInside += delegate {
                    var title = string.Format ("{0} clicks!", count++);
                    Button.SetTitle (title, UIControlState.Normal);
                };

                await MyClass.performBlobOperation();
            }

            public override void DidReceiveMemoryWarning ()
            {
                base.DidReceiveMemoryWarning ();
                // Release any cached data, images, etc that aren't in use.
            }
        }
    }
}
```

### <a name="xamarinappwinphone--mainpagexaml--mainpagexamlcs"></a>XamarinApp.WinPhone > MainPage.xaml > MainPage.xaml.cs

```csharp
using Windows.UI.Xaml.Controls;
using Windows.UI.Xaml.Navigation;

// The Blank Page item template is documented at http://go.microsoft.com/fwlink/?LinkId=391641

namespace XamarinApp.WinPhone
{
    /// <summary>
    /// An empty page that can be used on its own or navigated to within a Frame.
    /// </summary>
    public sealed partial class MainPage : Page
    {
        int count = 1;

        public MainPage()
        {
            this.InitializeComponent();

            this.NavigationCacheMode = NavigationCacheMode.Required;
        }

        /// <summary>
        /// Invoked when this page is about to be displayed in a Frame.
        /// </summary>
        /// <param name="e">Event data that describes how this page was reached.
        /// This parameter is typically used to configure the page.</param>
        protected override async void OnNavigatedTo(NavigationEventArgs e)
        {
            int count = 1;

            public MainPage()
            {
                this.InitializeComponent();

                this.NavigationCacheMode = NavigationCacheMode.Required;
            }

            /// <summary>
            /// Invoked when this page is about to be displayed in a Frame.
            /// </summary>
            /// <param name="e">Event data that describes how this page was reached.
            /// This parameter is typically used to configure the page.</param>
            protected override async void OnNavigatedTo(NavigationEventArgs e)
            {
                // TODO: Prepare page for display here.

                // TODO: If your application contains multiple pages, ensure that you are
                // handling the hardware Back button by registering for the
                // Windows.Phone.UI.Input.HardwareButtons.BackPressed event.
                // If you are using the NavigationHelper provided by some templates,
                // this event is handled for you.
                Button.Click += delegate {
                    var title = string.Format("{0} clicks!", count++);
                    Button.Content = title;
                };

                await MyClass.performBlobOperation();
            }
        }
    }
}
```

## <a name="run-the-application"></a>執行應用程式
您現在可以在 Android 或 Windows Phone 模擬器中執行這個應用程式。 您也可以在 iOS 模擬器中執行此應用程式，但這需要 Mac 作業系統。 如需具體的做法指示，請閱讀 [將 Visual Studio 連接到 Mac](https://developer.xamarin.com/guides/ios/getting_started/installation/windows/connecting-to-mac/)

一旦您執行應用程式，它會在您的儲存體帳戶中建立 `mycontainer` 容器。 它應該包含 blob `myblob`，其具有文字 `Hello, world!`。 您可以使用 [Microsoft Azure 儲存體總管](http://storageexplorer.com/)來驗證。

## <a name="next-steps"></a>後續步驟
在本教學課程中，您已學會如何在使用 Azure 儲存體的 Xamarin 中建立跨平台的應用程式，尤其將焦點放在 Blob 儲存體中的一個案例中。 不過，除了 Blob 儲存體以外，您還可以處理更多，例如資料表、檔案和佇列儲存體。 若要深入了解，請參閱下列文章：

* [以 .NET 開始使用 Azure Blob 儲存體](storage-dotnet-how-to-use-blobs.md)
* [Azure 檔案服務簡介](../files/storage-files-introduction.md)
* [使用 .NET 開發 Azure 檔案服務](../files/storage-dotnet-how-to-use-files.md)
* [以 .NET 開始使用 Azure 表格儲存體](../../cosmos-db/table-storage-how-to-use-dotnet.md)
* [以 .NET 開始使用 Azure 佇列儲存體](../queues/storage-dotnet-how-to-use-queues.md)

[!INCLUDE [storage-try-azure-tools-blobs](../../../includes/storage-try-azure-tools-blobs.md)]