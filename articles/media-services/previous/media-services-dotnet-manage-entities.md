---
title: 使用媒體服務 .NET SDK 管理資產和相關的實體
description: 深入了解使用 Media Services SDK for .NET 管理資產和相關的實體
author: juliako
manager: cfowler
editor: ''
services: media-services
documentationcenter: ''
ms.assetid: 1bd8fd42-7306-463d-bfe5-f642802f1906
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/17/2017
ms.author: juliako
ms.openlocfilehash: af5baf3444196e5a0e8412d9ab4f019fdccb033e
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33783677"
---
# <a name="managing-assets-and-related-entities-with-media-services-net-sdk"></a>使用媒體服務 .NET SDK 管理資產和相關的實體
> [!div class="op_single_selector"]
> * [.NET](media-services-dotnet-manage-entities.md)
> * [REST](media-services-rest-manage-entities.md)
> 
> 

本主題說明如何使用 .NET 管理 Azure 媒體服務實體。 

>[!NOTE]
> 從 2017 年 4 月 1 日起，您的帳戶中任何超過 90 天的作業記錄以及其相關工作記錄都會自動刪除，即使記錄總數低於配額上限亦然。 例如，在 2017 年 4 月 1 日，您帳戶中任何在 2016 年 12 月 31 日以前的作業記錄將會自動刪除。 如果您需要封存作業/工作資訊，您可以使用本主題中所述的程式碼。

## <a name="prerequisites"></a>先決條件

設定您的開發環境並在 app.config 檔案中填入連線資訊，如[使用 .NET 進行 Media Services 開發](media-services-dotnet-how-to-use.md)所述。 

## <a name="get-an-asset-reference"></a>取得資產參考
常見的工作是在媒體服務中取得現有資產的參考。 下列程式碼範例顯示如何根據資產識別碼，從伺服器內容物件的資產集合取得資產參考。下列程式碼範例會使用 Linq 查詢來取得現有 IAsset 物件的參考。

```csharp
    static IAsset GetAsset(string assetId)
    {
        // Use a LINQ Select query to get an asset.
        var assetInstance =
            from a in _context.Assets
            where a.Id == assetId
            select a;
        // Reference the asset as an IAsset.
        IAsset asset = assetInstance.FirstOrDefault();

        return asset;
    }
```

## <a name="list-all-assets"></a>列出所有資產
當您在儲存體中的資產數目增加時，最好能將資產列出。 下列程式碼範例顯示如何在伺服器內容物件上逐一查看「資產」集合。 使用各個資產時，程式碼範例也會將它的某些屬性值寫入主控台。 例如，每個資產可以包含多個媒體檔案。 程式碼範例會寫出與每個資產相關聯的所有檔案。

```csharp
    static void ListAssets()
    {
        string waitMessage = "Building the list. This may take a few "
            + "seconds to a few minutes depending on how many assets "
            + "you have."
            + Environment.NewLine + Environment.NewLine
            + "Please wait..."
            + Environment.NewLine;
        Console.Write(waitMessage);

        // Create a Stringbuilder to store the list that we build. 
        StringBuilder builder = new StringBuilder();

        foreach (IAsset asset in _context.Assets)
        {
            // Display the collection of assets.
            builder.AppendLine("");
            builder.AppendLine("******ASSET******");
            builder.AppendLine("Asset ID: " + asset.Id);
            builder.AppendLine("Name: " + asset.Name);
            builder.AppendLine("==============");
            builder.AppendLine("******ASSET FILES******");

            // Display the files associated with each asset. 
            foreach (IAssetFile fileItem in asset.AssetFiles)
            {
                builder.AppendLine("Name: " + fileItem.Name);
                builder.AppendLine("Size: " + fileItem.ContentFileSize);
                builder.AppendLine("==============");
            }
        }

        // Display output in console.
        Console.Write(builder.ToString());
    }
```

## <a name="get-a-job-reference"></a>取得工作參考

當您使用媒體服務程式碼中的處理工作時，您通常需要根據識別碼取得現有工作的參考。下列程式碼範例顯示如何從「工作」集合取得 IJob 物件的參考。

您可能需要在開始長時間執行編碼作業時，取得作業參考，並且需要檢查執行緒上的作業狀態。 在這種情況下，當此方法從執行緒傳回時，您需要擷取作業的重新整理的參考。

```csharp
    static IJob GetJob(string jobId)
    {
        // Use a Linq select query to get an updated 
        // reference by Id. 
        var jobInstance =
            from j in _context.Jobs
            where j.Id == jobId
            select j;
        // Return the job reference as an Ijob. 
        IJob job = jobInstance.FirstOrDefault();

        return job;
    }
```

## <a name="list-jobs-and-assets"></a>列出工作和資產
重要的相關工作是在媒體服務中列出資產及其相關聯的工作。 下列程式碼範例會示範如何列出每個 IJob 物件，然後它會針對每個工作，顯示作業的相關屬性、所有相關工作、所有輸入資產，以及所有輸出資產。 此範例中的程式碼可用於許多其他工作。 例如，如果您想要從您先前執行的一或多個編碼工作列出輸出資產，此程式碼會示範如何存取輸出資產。 當您有輸出資產的參考時，可以透過下載或提供 URL，將內容傳遞給其他使用者或應用程式。 

如需傳遞資產之選項的詳細資訊，請參閱 [使用 Media Services SDK for .NET 傳遞資產](media-services-deliver-streaming-content.md)。

```csharp
    // List all jobs on the server, and for each job, also list 
    // all tasks, all input assets, all output assets.

    static void ListJobsAndAssets()
    {
        string waitMessage = "Building the list. This may take a few "
            + "seconds to a few minutes depending on how many assets "
            + "you have."
            + Environment.NewLine + Environment.NewLine
            + "Please wait..."
            + Environment.NewLine;
        Console.Write(waitMessage);

        // Create a Stringbuilder to store the list that we build. 
        StringBuilder builder = new StringBuilder();

        foreach (IJob job in _context.Jobs)
        {
            // Display the collection of jobs on the server.
            builder.AppendLine("");
            builder.AppendLine("******JOB*******");
            builder.AppendLine("Job ID: " + job.Id);
            builder.AppendLine("Name: " + job.Name);
            builder.AppendLine("State: " + job.State);
            builder.AppendLine("Order: " + job.Priority);
            builder.AppendLine("==============");


            // For each job, display the associated tasks (a job  
            // has one or more tasks). 
            builder.AppendLine("******TASKS*******");
            foreach (ITask task in job.Tasks)
            {
                builder.AppendLine("Task Id: " + task.Id);
                builder.AppendLine("Name: " + task.Name);
                builder.AppendLine("Progress: " + task.Progress);
                builder.AppendLine("Configuration: " + task.Configuration);
                if (task.ErrorDetails != null)
                {
                    builder.AppendLine("Error: " + task.ErrorDetails);
                }
                builder.AppendLine("==============");
            }

            // For each job, display the list of input media assets.
            builder.AppendLine("******JOB INPUT MEDIA ASSETS*******");
            foreach (IAsset inputAsset in job.InputMediaAssets)
            {

                if (inputAsset != null)
                {
                    builder.AppendLine("Input Asset Id: " + inputAsset.Id);
                    builder.AppendLine("Name: " + inputAsset.Name);
                    builder.AppendLine("==============");
                }
            }

            // For each job, display the list of output media assets.
            builder.AppendLine("******JOB OUTPUT MEDIA ASSETS*******");
            foreach (IAsset theAsset in job.OutputMediaAssets)
            {
                if (theAsset != null)
                {
                    builder.AppendLine("Output Asset Id: " + theAsset.Id);
                    builder.AppendLine("Name: " + theAsset.Name);
                    builder.AppendLine("==============");
                }
            }

        }

        // Display output in console.
        Console.Write(builder.ToString());
    }
```

## <a name="list-all-access-policies"></a>列出所有存取原則
在媒體服務中，您可以定義資產或其檔案的存取原則。 存取原則會定義檔案或資產 (存取的類型和持續時間) 的權限。 在您的媒體服務程式碼中，通常會藉由建立 IAccessPolicy 物件，然後將其與現有資產產生關聯，來定義存取原則。 然後您會建立 ILocator 物件，這可讓您直接存取媒體服務中的資產。 本文件系列隨附的 Visual Studio 專案包含數個程式碼範例，示範如何建立及指派存取原則和定位器給資產。

下列程式碼範例顯示如何列出伺服器上的所有存取原則，並顯示每個相關聯的權限的類型。 檢視存取原則的另一個實用方式，是列出伺服器上的所有 ILocator 物件，然後針對每個定位器，您可以使用其 AccessPolicy 屬性列出其相關聯的存取原則。

```csharp
    static void ListAllPolicies()
    {
        foreach (IAccessPolicy policy in _context.AccessPolicies)
        {
            Console.WriteLine("");
            Console.WriteLine("Name:  " + policy.Name);
            Console.WriteLine("ID:  " + policy.Id);
            Console.WriteLine("Permissions: " + policy.Permissions);
            Console.WriteLine("==============");

        }
    }
```
    
## <a name="limit-access-policies"></a>限制存取原則 

>[!NOTE]
> 對於不同的 AMS 原則 (例如 Locator 原則或 ContentKeyAuthorizationPolicy) 有 1,000,000 個原則的限制。 如果您一律使用相同的日期 / 存取權限，例如，要長時間維持就地 (非上載原則) 的定位器原則，您應該使用相同的原則識別碼。 

例如，您可以使用只會在您應用程式中執行一次的下列程式碼來建立一組標準的原則。 您可以記錄識別碼至記錄檔以供稍後使用︰

```csharp
    double year = 365.25;
    double week = 7;
    IAccessPolicy policyYear = _context.AccessPolicies.Create("One Year", TimeSpan.FromDays(year), AccessPermissions.Read);
    IAccessPolicy policy100Year = _context.AccessPolicies.Create("Hundred Years", TimeSpan.FromDays(year * 100), AccessPermissions.Read);
    IAccessPolicy policyWeek = _context.AccessPolicies.Create("One Week", TimeSpan.FromDays(week), AccessPermissions.Read);

    Console.WriteLine("One year policy ID is: " + policyYear.Id);
    Console.WriteLine("100 year policy ID is: " + policy100Year.Id);
    Console.WriteLine("One week policy ID is: " + policyWeek.Id);
```

然後，您可以使用您程式碼中的現有識別碼，就像這樣︰

```csharp
    const string policy1YearId = "nb:pid:UUID:2a4f0104-51a9-4078-ae26-c730f88d35cf";


    // Get the standard policy for 1 year read only
    var tempPolicyId = from b in _context.AccessPolicies
                       where b.Id == policy1YearId
                       select b;
    IAccessPolicy policy1Year = tempPolicyId.FirstOrDefault();

    // Get the existing asset
    var tempAsset = from a in _context.Assets
                where a.Id == assetID
                select a;
    IAsset asset = tempAsset.SingleOrDefault();

    ILocator originLocator = _context.Locators.CreateLocator(LocatorType.OnDemandOrigin, asset,
        policy1Year,
        DateTime.UtcNow.AddMinutes(-5));
    Console.WriteLine("The locator base path is " + originLocator.BaseUri.ToString());
```

## <a name="list-all-locators"></a>列出所有定位器
定位器是 URL，提供存取資產的直接路徑，以及由定位器相關聯的存取原則所定義之對於資產的權限。 每個資產在其定位器屬性上可以有與其相關聯的 ILocator 物件的集合。 伺服器內容也有定位器集合，其中包含所有定位器。

下列程式碼範例列出伺服器上的所有定位器。 針對每個定位器，它會顯示相關資產與存取原則的識別碼。 也會顯示資產的權限類型、到期日和完整路徑。

請注意，資產的定位器路徑只是資產的基底 URL。 若要建立使用者或應用程式可以瀏覽之個別檔案的直接路徑，您的程式碼必須將特定檔案路徑加入至定位器路徑。 如需如何執行此動作的詳細資訊，請參閱 [使用 Media Services SDK for .NET 傳遞資產](media-services-deliver-streaming-content.md)主題。

```csharp
    static void ListAllLocators()
    {
        foreach (ILocator locator in _context.Locators)
        {
            Console.WriteLine("***********");
            Console.WriteLine("Locator Id: " + locator.Id);
            Console.WriteLine("Locator asset Id: " + locator.AssetId);
            Console.WriteLine("Locator access policy Id: " + locator.AccessPolicyId);
            Console.WriteLine("Access policy permissions: " + locator.AccessPolicy.Permissions);
            Console.WriteLine("Locator expiration: " + locator.ExpirationDateTime);
            // The locator path is the base or parent path (with included permissions) to access  
            // the media content of an asset. To create a full URL to a specific media file, take 
            // the locator path and then append a file name and info as needed.  
            Console.WriteLine("Locator base path: " + locator.Path);
            Console.WriteLine("");
        }
    }
```

## <a name="enumerating-through-large-collections-of-entities"></a>透過實體的大型集合列舉
查詢項目時，有一次最多傳回 1000 個實體的限制，因為公用 REST v2 有 1000 個查詢結果數目的限制。 透過實體的大型集合列舉時您需要使用 Skip 和 Take。 

下列函式會對「媒體服務帳戶」中提供的所有工作進行迴圈。 媒體服務會傳回工作集合中的 1000 個工作。 此函式利用 Skip 和 Take 來確保已列舉所有工作 (以免您帳戶中的工作超過 1000 個)。

```csharp
    static void ProcessJobs()
    {
        try
        {

            int skipSize = 0;
            int batchSize = 1000;
            int currentBatch = 0;

            while (true)
            {
                // Loop through all Jobs (1000 at a time) in the Media Services account
                IQueryable _jobsCollectionQuery = _context.Jobs.Skip(skipSize).Take(batchSize);
                foreach (IJob job in _jobsCollectionQuery)
                {
                    currentBatch++;
                    Console.WriteLine("Processing Job Id:" + job.Id);
                }

                if (currentBatch == batchSize)
                {
                    skipSize += batchSize;
                    currentBatch = 0;
                }
                else
                {
                    break;
                }
            }
        }
        catch (Exception ex)
        {
            Console.WriteLine(ex.Message);
        }
    }
```

## <a name="delete-an-asset"></a>刪除資產
下列範例會刪除資產。

```csharp
    static void DeleteAsset( IAsset asset)
    {
        // delete the asset
        asset.Delete();

        // Verify asset deletion
        if (GetAsset(asset.Id) == null)
            Console.WriteLine("Deleted the Asset");

    }
```

## <a name="delete-a-job"></a>刪除工作
若要刪除工作，您必須檢查工作的狀態，如 State 屬性所示。 可以刪除已完成或已取消的工作，而其他某些狀態，例如已佇列、已排程或處理中的工作，必須先取消，然後才能刪除。

下列程式碼範例示範刪除工作的方法，方法是檢查工作狀態，然後於狀態為已完成或已取消時刪除工作。 此程式碼相依於本主題中的前一節，以取得工作的參考：取得工作參考。

```csharp
    static void DeleteJob(string jobId)
    {
        bool jobDeleted = false;

        while (!jobDeleted)
        {
            // Get an updated job reference.  
            IJob job = GetJob(jobId);

            // Check and handle various possible job states. You can 
            // only delete a job whose state is Finished, Error, or Canceled.   
            // You can cancel jobs that are Queued, Scheduled, or Processing,  
            // and then delete after they are canceled.
            switch (job.State)
            {
                case JobState.Finished:
                case JobState.Canceled:
                case JobState.Error:
                    // Job errors should already be logged by polling or event 
                    // handling methods such as CheckJobProgress or StateChanged.
                    // You can also call job.DeleteAsync to do async deletes.
                    job.Delete();
                    Console.WriteLine("Job has been deleted.");
                    jobDeleted = true;
                    break;
                case JobState.Canceling:
                    Console.WriteLine("Job is cancelling and will be deleted "
                        + "when finished.");
                    Console.WriteLine("Wait while job finishes canceling...");
                    Thread.Sleep(5000);
                    break;
                case JobState.Queued:
                case JobState.Scheduled:
                case JobState.Processing:
                    job.Cancel();
                    Console.WriteLine("Job is scheduled or processing and will "
                        + "be deleted.");
                    break;
                default:
                    break;
            }

        }
    }
```


## <a name="delete-an-access-policy"></a>刪除存取原則
下列程式碼範例顯示如何根據原則識別碼，取得存取原則的參考，然後刪除原則。

```csharp
    static void DeleteAccessPolicy(string existingPolicyId)
    {
        // To delete a specific access policy, get a reference to the policy.  
        // based on the policy Id passed to the method.
        var policyInstance =
                from p in _context.AccessPolicies
                where p.Id == existingPolicyId
                select p;
        IAccessPolicy policy = policyInstance.FirstOrDefault();

        policy.Delete();

    }
```


## <a name="media-services-learning-paths"></a>媒體服務學習路徑
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

