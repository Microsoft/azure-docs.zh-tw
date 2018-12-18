---
title: 使用 Azure Java SDK 來管理 Azure Data Lake Analytics
description: 本文說明如何使用 Azure Java SDK 來撰寫應用程式，以管理 Data Lake Analytics 作業、資料來源和使用者。
services: data-lake-analytics
ms.service: data-lake-analytics
author: saveenr
ms.author: saveenr
ms.reviewer: jasonwhowell
ms.assetid: 07830b36-2fe3-4809-a846-129cf67b6a9e
ms.topic: conceptual
ms.date: 06/18/2017
ms.openlocfilehash: 938b3776f320b7556394fff9aa070eee0c44ee88
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43047081"
---
# <a name="manage-azure-data-lake-analytics-using-a-java-app"></a>使用 Java 應用程式來管理 Azure Data Lake Analytics
[!INCLUDE [manage-selector](../../includes/data-lake-analytics-selector-manage.md)]

本文說明如何使用以 Azure Java SDK 所撰寫的應用程式，來管理 Azure Data Lake Analytics 帳戶、資料來源、使用者和作業。 

## <a name="prerequisites"></a>必要條件
* **Java Development Kit (JDK) 8** (使用 Java 1.8 版)。
* **IntelliJ** 或其他合適的 Java 開發環境。 本文件中的指示使用 IntelliJ。
* 建立 Azure Active Directory (AAD) 應用程式，並擷取其**用戶端識別碼**、**租用戶識別碼**和**金鑰**。 如需了解 AAD 應用程式，以及如何取得用戶端識別碼的指示，請參閱 [使用入口網站建立 Active Directory 應用程式和服務主體](../azure-resource-manager/resource-group-create-service-principal-portal.md)。 建立應用程式並產生金鑰後，可以從入口網站取得回覆 URI 和金鑰。

## <a name="authenticating-using-azure-active-directory"></a>使用 Azure Active Directory 進行驗證

下列程式碼片段提供**非互動式**驗證的程式碼，其中應用程式會提供它自己的認證。

## <a name="create-a-java-application"></a>建立 Java 應用程式
1. 開啟 IntelliJ，並使用 [命令列應用程式] 範本建立 Java 專案。
2. 在畫面左側的專案上按一下滑鼠右鍵，然後按一下 [新增架構支援] 。 選擇 [Maven] 並按一下 [確定]。
3. 開啟新建立的 **"pom.xml"** 檔案，並在 **\</version>** 標記和 **\<</project>** 標記之間新增下列一小段文字︰

```
<repositories>
    <repository>
        <id>adx-snapshots</id>
        <name>Azure ADX Snapshots</name>
        <url>http://adxsnapshots.azurewebsites.net/</url>
        <layout>default</layout>
        <snapshots>
            <enabled>true</enabled>
        </snapshots>
    </repository>
    <repository>
        <id>oss-snapshots</id>
        <name>Open Source Snapshots</name>
        <url>https://oss.sonatype.org/content/repositories/snapshots/</url>
        <layout>default</layout>
        <snapshots>
            <enabled>true</enabled>
            <updatePolicy>always</updatePolicy>
        </snapshots>
    </repository>
</repositories>
<dependencies>
    <dependency>
        <groupId>com.microsoft.azure</groupId>
        <artifactId>azure-client-authentication</artifactId>
        <version>1.0.0-20160513.000802-24</version>
    </dependency>
    <dependency>
        <groupId>com.microsoft.azure</groupId>
        <artifactId>azure-client-runtime</artifactId>
        <version>1.0.0-20160513.000812-28</version>
    </dependency>
    <dependency>
        <groupId>com.microsoft.rest</groupId>
        <artifactId>client-runtime</artifactId>
        <version>1.0.0-20160513.000825-29</version>
    </dependency>
    <dependency>
        <groupId>com.microsoft.azure</groupId>
        <artifactId>azure-mgmt-datalake-store</artifactId>
        <version>1.0.0-SNAPSHOT</version>
    </dependency>
    <dependency>
        <groupId>com.microsoft.azure</groupId>
        <artifactId>azure-mgmt-datalake-analytics</artifactId>
        <version>1.0.0-SNAPSHOT</version>
    </dependency>
</dependencies>
```

移至 [檔案] > [設定] > [建置] > [執行] > [部署]。 選取 [建置工具] > [Maven] > [匯入]。 然後勾選 [自動匯入 Maven 專案] 。

開啟 `Main.java`，並以下列程式碼片段取代現有的程式碼區塊：

```
package com.company;

import com.microsoft.azure.CloudException;
import com.microsoft.azure.credentials.ApplicationTokenCredentials;
import com.microsoft.azure.management.datalake.store.*;
import com.microsoft.azure.management.datalake.store.models.*;
import com.microsoft.azure.management.datalake.analytics.*;
import com.microsoft.azure.management.datalake.analytics.models.*;
import com.microsoft.rest.credentials.ServiceClientCredentials;
import java.io.*;
import java.nio.charset.Charset;
import java.nio.file.Files;
import java.nio.file.Paths;
import java.util.ArrayList;
import java.util.UUID;
import java.util.List;

public class Main {
    private static String _adlsAccountName;
    private static String _adlaAccountName;
    private static String _resourceGroupName;
    private static String _location;

    private static String _tenantId;
    private static String _subId;
    private static String _clientId;
    private static String _clientSecret;

    private static DataLakeStoreAccountManagementClient _adlsClient;
    private static DataLakeStoreFileSystemManagementClient _adlsFileSystemClient;
    private static DataLakeAnalyticsAccountManagementClient _adlaClient;
    private static DataLakeAnalyticsJobManagementClient _adlaJobClient;
    private static DataLakeAnalyticsCatalogManagementClient _adlaCatalogClient;

    public static void main(String[] args) throws Exception {

        _adlsAccountName = "<DATA-LAKE-STORE-NAME>";
        _adlaAccountName = "<DATA-LAKE-ANALYTICS-NAME>";
        _resourceGroupName = "<RESOURCE-GROUP-NAME>";
        _location = "East US 2";

        _tenantId = "<TENANT-ID>";
        _subId =  "<SUBSCRIPTION-ID>";
        _clientId = "<CLIENT-ID>";

        _clientSecret = "<CLIENT-SECRET>"; 
        
        String localFolderPath = "C:\\local_path\\"; 
        
        // ----------------------------------------
        // Authenticate
        // ----------------------------------------
        ApplicationTokenCredentials creds = new ApplicationTokenCredentials(_clientId, _tenantId, _clientSecret, null);
        SetupClients(creds);

        // ----------------------------------------
        // List Data Lake Store and Analytics accounts that this app can access
        // ----------------------------------------
        System.out.println(String.format("All ADL Store accounts that this app can access in subscription %s:", _subId));
        List<DataLakeStoreAccount> adlsListResult = _adlsClient.getAccountOperations().list().getBody();
        for (DataLakeStoreAccount acct : adlsListResult) {
            System.out.println(acct.getName());
        }
        
        System.out.println(String.format("All ADL Analytics accounts that this app can access in subscription %s:", _subId));
        List<DataLakeAnalyticsAccount> adlaListResult = _adlaClient.getAccountOperations().list().getBody();
        for (DataLakeAnalyticsAccount acct : adlaListResult) {
            System.out.println(acct.getName());
        }
        WaitForNewline("Accounts displayed.", "Creating files.");

        // ----------------------------------------
        // Create a file in Data Lake Store: input1.csv
        // ----------------------------------------

        byte[] bytesContents = "123,abc".getBytes();
        _adlsFileSystemClient.getFileSystemOperations().create(_adlsAccountName, "/input1.csv", bytesContents, true);

        WaitForNewline("File created.", "Submitting a job.");

        // ----------------------------------------
        // Submit a job to Data Lake Analytics
        // ----------------------------------------

string script = "@input =  EXTRACT Data string FROM \"/input1.csv\" USING Extractors.Csv(); OUTPUT @input TO @\"/output1.csv\" USING Outputters.Csv();", "testJob";
        UUID jobId = SubmitJobByScript(script);
        WaitForNewline("Job submitted.", "Getting job status.");

        // ----------------------------------------
        // Wait for job completion and output job status
        // ----------------------------------------
        System.out.println(String.format("Job status: %s", GetJobStatus(jobId)));
        System.out.println("Waiting for job completion.");
        WaitForJob(jobId);
        System.out.println(String.format("Job status: %s", GetJobStatus(jobId)));
        WaitForNewline("Job completed.", "Downloading job output.");

        // ----------------------------------------
        // Download job output from Data Lake Store
        // ----------------------------------------
        DownloadFile("/output1.csv", localFolderPath + "output1.csv");
        WaitForNewline("Job output downloaded.", "Deleting file.");

    }
}
```

提供在程式碼片段中呼叫之參數的值：
* `localFolderPath`
* `_adlaAccountName`
* `_adlsAccountName`
* `_resourceGroupName`

取代下列各項的預留位置：
* `CLIENT-ID`,
* `CLIENT-SECRET`,
* `TENANT-ID`
* `SUBSCRIPTION-ID`

## <a name="helper-functions"></a>協助程式函式

### <a name="setup-clients"></a>設定用戶端

```
public static void SetupClients(ServiceClientCredentials creds)
{
    _adlsClient = new DataLakeStoreAccountManagementClientImpl(creds);
    _adlsFileSystemClient = new DataLakeStoreFileSystemManagementClientImpl(creds);
    _adlaClient = new DataLakeAnalyticsAccountManagementClientImpl(creds);
    _adlaJobClient = new DataLakeAnalyticsJobManagementClientImpl(creds);
    _adlaCatalogClient = new DataLakeAnalyticsCatalogManagementClientImpl(creds);
    _adlsClient.setSubscriptionId(_subId);
    _adlaClient.setSubscriptionId(_subId);
}
```


### <a name="wait-for-input"></a>等待輸入

```
public static void WaitForNewline(String reason, String nextAction)
{
    if (nextAction == null)
        nextAction = "";

    System.out.println(reason + "\r\nPress ENTER to continue...");
    try{System.in.read();}
    catch(Exception e){}

    if (!nextAction.isEmpty())
    {
        System.out.println(nextAction);
    }
}
```

### <a name="create-accounts"></a>建立帳戶

```
public static void CreateAccounts() throws InterruptedException, CloudException, IOException 
{
    // Create ADLS account
    DataLakeStoreAccount adlsParameters = new DataLakeStoreAccount();
    adlsParameters.setLocation(_location);

    _adlsClient.getAccountOperations().create(_resourceGroupName, _adlsAccountName, adlsParameters);

    // Create ADLA account
    DataLakeStoreAccountInfo adlsInfo = new DataLakeStoreAccountInfo();
    adlsInfo.setName(_adlsAccountName);

    DataLakeStoreAccountInfoProperties adlsInfoProperties = new DataLakeStoreAccountInfoProperties();
    adlsInfo.setProperties(adlsInfoProperties);

    List<DataLakeStoreAccountInfo> adlsInfoList = new ArrayList<DataLakeStoreAccountInfo>();
    adlsInfoList.add(adlsInfo);

    DataLakeAnalyticsAccountProperties adlaProperties = new DataLakeAnalyticsAccountProperties();
    adlaProperties.setDataLakeStoreAccounts(adlsInfoList);
    adlaProperties.setDefaultDataLakeStoreAccount(_adlsAccountName);

    DataLakeAnalyticsAccount adlaParameters = new DataLakeAnalyticsAccount();
    adlaParameters.setLocation(_location);
    adlaParameters.setName(_adlaAccountName);
    adlaParameters.setProperties(adlaProperties);

    _adlaClient.getAccountOperations().create(_resourceGroupName, _adlaAccountName, adlaParameters);
}
```

### <a name="create-a-file"></a>建立檔案

```
public static void CreateFile(String path, String contents, boolean force) throws IOException, CloudException 
{
    byte[] bytesContents = contents.getBytes();

    _adlsFileSystemClient.getFileSystemOperations().create(_adlsAccountName, path, bytesContents, force);
}
```

### <a name="delete-a-file"></a>刪除檔案

```
public static void DeleteFile(String filePath) throws IOException, CloudException 
{
    _adlsFileSystemClient.getFileSystemOperations().delete(filePath, _adlsAccountName);
}
```

### <a name="download-a-file"></a>下載檔案

```
public static void DownloadFile(String srcPath, String destPath) throws IOException, CloudException 
{
    InputStream stream = _adlsFileSystemClient.getFileSystemOperations().open(srcPath, _adlsAccountName).getBody();

    PrintWriter pWriter = new PrintWriter(destPath, Charset.defaultCharset().name());

    String fileContents = "";
    if (stream != null) {
        Writer writer = new StringWriter();

        char[] buffer = new char[1024];
        try {
            Reader reader = new BufferedReader(
                    new InputStreamReader(stream, "UTF-8"));
            int n;
            while ((n = reader.read(buffer)) != -1) {
                writer.write(buffer, 0, n);
            }
        } finally {
            stream.close();
        }
        fileContents =  writer.toString();
    }

    pWriter.println(fileContents);
    pWriter.close();
}
```

### <a name="submit-a-u-sql-job"></a>提交 U-SQL 作業

```
public static UUID SubmitJobByScript(String script, String jobName) throws IOException, CloudException 
{
    UUID jobId = java.util.UUID.randomUUID();
    USqlJobProperties properties = new USqlJobProperties();
    properties.setScript(script);
    JobInformation parameters = new JobInformation();
    parameters.setName(jobName);
    parameters.setJobId(jobId);
    parameters.setType(JobType.USQL);
    parameters.setProperties(properties);

    JobInformation jobInfo = _adlaJobClient.getJobOperations().create(_adlaAccountName, jobId, parameters).getBody();

    return jobId;
}

// Wait for job completion
public static JobResult WaitForJob(UUID jobId) throws IOException, CloudException 
{
    JobInformation jobInfo = _adlaJobClient.getJobOperations().get(_adlaAccountName, jobId).getBody();
    while (jobInfo.getState() != JobState.ENDED)
    {
        jobInfo = _adlaJobClient.getJobOperations().get(_adlaAccountName,jobId).getBody();
    }
    return jobInfo.getResult();
}
```

### <a name="retrieve-job-status"></a>擷取作業狀態

```
public static String GetJobStatus(UUID jobId) throws IOException, CloudException 
{
    JobInformation jobInfo = _adlaJobClient.getJobOperations().get(_adlaAccountName, jobId).getBody();
    return jobInfo.getState().toValue();
}
```

## <a name="next-steps"></a>後續步驟

* 若要了解 U-SQL，請參閱[開始使用 Azure Data Lake Analytics U-SQL 語言](data-lake-analytics-u-sql-get-started.md)和 [U-SQL 語言參考](http://go.microsoft.com/fwlink/?LinkId=691348)。
* 針對管理工作，請參閱 [使用 Azure 入口網站管理 Azure Data Lake Analytics](data-lake-analytics-manage-use-portal.md)。
* 若要取得 Data Lake Analytics 概觀，請參閱 [Azure Data Lake Analytics 概觀](data-lake-analytics-overview.md)。
