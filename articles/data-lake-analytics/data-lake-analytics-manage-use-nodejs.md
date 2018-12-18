---
title: 使用 Node.js 的 Azure SDK 管理 Azure 資料湖分析
description: 本文說明如何使用 Azure SDK for Node.js 來管理 Data Lake Analytics 帳戶、資料來源、作業和使用者。
services: data-lake-analytics
ms.service: data-lake-analytics
author: saveenr
ms.author: saveenr
ms.reviewer: jasonwhowell
ms.assetid: 9de1bcf4-b15b-4d0b-9284-8889ecf0c438
ms.topic: conceptual
ms.date: 12/05/2016
ms.openlocfilehash: 0603a60ea73d47dd6107ee80afc5c776ff8c83bc
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43047106"
---
# <a name="manage-azure-data-lake-analytics-using-azure-sdk-for-nodejs"></a>使用 Node.js 的 Azure SDK 管理 Azure 資料湖分析
[!INCLUDE [manage-selector](../../includes/data-lake-analytics-selector-manage.md)]

本文說明如何使用以 Azure SDK for Node.js 所撰寫的應用程式，來管理 Azure Data Lake Analytics 帳戶、資料來源、使用者和作業。 

支援的版本如下：
* **Node.js 版本：0.10.0 或更高版本**
* **帳戶的 REST API 版本：2015-10-01-preview**
* **目錄的 REST API 版本：2015-10-01-preview**
* **作業的 REST API 版本：2016-03-20-preview**

## <a name="features"></a>特性
* 帳戶管理：建立、取得、列出、更新及刪除。
* 作業管理︰提交、取得、列出及取消。
* 目錄管理：取得及列出。

## <a name="how-to-install"></a>如何安裝
```bash
npm install azure-arm-datalake-analytics
```

## <a name="authenticate-using-azure-active-directory"></a>使用 Azure Active Directory 進行驗證
 ```javascript
 var msrestAzure = require('ms-rest-azure');
 //user authentication
 var credentials = new msRestAzure.UserTokenCredentials('your-client-id', 'your-domain', 'your-username', 'your-password', 'your-redirect-uri');
 //service principal authentication
 var credentials = new msRestAzure.ApplicationTokenCredentials('your-client-id', 'your-domain', 'your-secret');
 ```

## <a name="create-the-data-lake-analytics-client"></a>建立 Data Lake Analytics 用戶端
```javascript
var adlaManagement = require("azure-arm-datalake-analytics");
var acccountClient = new adlaManagement.DataLakeAnalyticsAccountClient(credentials, 'your-subscription-id');
var jobClient = new adlaManagement.DataLakeAnalyticsJobClient(credentials, 'azuredatalakeanalytics.net');
var catalogClient = new adlaManagement.DataLakeAnalyticsCatalogClient(credentials, 'azuredatalakeanalytics.net');
```

## <a name="create-a-data-lake-analytics-account"></a>建立 Data Lake Analytics 帳戶
```javascript
var util = require('util');
var resourceGroupName = 'testrg';
var accountName = 'testadlaacct';
var location = 'eastus2';

// A Data Lake Store account must already have been created to create
// a Data Lake Analytics account. See the Data Lake Store readme for
// information on doing so. For now, we assume one exists already.
var datalakeStoreAccountName = 'existingadlsaccount';

// account object to create
var accountToCreate = {
  tags: {
    testtag1: 'testvalue1',
    testtag2: 'testvalue2'
  },
  name: accountName,
  location: location,
  properties: {
    defaultDataLakeStoreAccount: datalakeStoreAccountName,
    dataLakeStoreAccounts: [
      {
        name: datalakeStoreAccountName
      }
    ]
  }
};

client.account.create(resourceGroupName, accountName, accountToCreate, function (err, result, request, response) {
  if (err) {
    console.log(err);
    /*err has reference to the actual request and response, so you can see what was sent and received on the wire.
      The structure of err looks like this:
      err: {
        code: 'Error Code',
        message: 'Error Message',
        body: 'The response body if any',
        request: reference to a stripped version of http request
        response: reference to a stripped version of the response
      }
    */
  } else {
    console.log('result is: ' + util.inspect(result, {depth: null}));
  }
});
```

## <a name="get-a-list-of-jobs"></a>取得作業清單
```javascript
var util = require('util');
var accountName = 'testadlaacct';
jobClient.job.list(accountName, function (err, result, request, response) {
  if (err) {
    console.log(err);
  } else {
    console.log('result is: ' + util.inspect(result, {depth: null}));
  }
});
```

## <a name="get-a-list-of-databases-in-the-data-lake-analytics-catalog"></a>取得 Data Lake Analytics 目錄中的資料庫清單
```javascript
var util = require('util');
var accountName = 'testadlaacct';
catalogClient.catalog.listDatabases(accountName, function (err, result, request, response) {
  if (err) {
    console.log(err);
  } else {
    console.log('result is: ' + util.inspect(result, {depth: null}));
  }
});
```

## <a name="see-also"></a>另請參閱
* [Microsoft Azure SDK for Node.js](https://github.com/azure/azure-sdk-for-node)
* [Microsoft Azure SDK for Node.js - Data Lake Store 管理](https://github.com/Azure/azure-sdk-for-node/tree/autorest/lib/services/dataLake.Store)

