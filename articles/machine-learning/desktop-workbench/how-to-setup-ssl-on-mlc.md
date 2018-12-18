---
title: 在 Azure Machine Learning 計算 (MLC) 叢集上啟用 SSL | Microsoft Docs
description: 取得在 Azure Machine Learning 計算 (MLC) 叢集上設定評分呼叫 SSL 的指示
services: machine-learning
author: SerinaKaye
ms.author: serinak
manager: hjerez
ms.reviewer: jmartens, jasonwhowell, mldocs
ms.service: machine-learning
ms.component: core
ms.workload: data-services
ms.custom: mvc
ms.topic: article
ms.date: 01/24/2018
ROBOTS: NOINDEX
ms.openlocfilehash: 2a7733468ec082c8954f623f3ebe2cea1fbad561
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46976213"
---
# <a name="enable-ssl-on-an-azure-machine-learning-compute-mlc-cluster"></a>在 Azure Machine Learning 計算 (MLC) 叢集上啟用 SSL 

[!INCLUDE [workbench-deprecated](../../../includes/aml-deprecating-preview-2017.md)]


這些指示可讓您在 Azure Machine Learning 計算 (MLC) 叢集上設定評分呼叫的安全通訊端層 (SSL)。 

## <a name="prerequisites"></a>必要條件 

1. 決定進行即時評分呼叫時要使用的 CNAME (DNS 名稱)。

2. 以此 CNAME 建立「免用密碼」憑證。

3. 如果憑證不是 PEM 格式，請使用工具 (例如 *openssl*) 將它轉換成 PEM。

完成先決條件之後，您會有兩個檔案：

* 憑證檔案，例如 `cert.pem`。 請確定該檔案具有完整的信任鏈結。
* 金鑰檔案，例如 `key.pem`



## <a name="set-up-an-ssl-certificate-on-a-new-acs-cluster"></a>在新的 ACS 叢集上設定 SSL 憑證

使用 Azure Machine Learning CLI，搭配 `-c` 參數執行下列命令，以建立附加了 SSL 憑證的 ACS 叢集：

```
az ml env create -c -g <resource group name> -n <cluster name> --cert-cname <CNAME> --cert-pem <path to cert.pem file> --key-pem <path to key.pem file>
```


## <a name="set-up-an-ssl-certificate-on-an-existing-acs-cluster"></a>在現有 ACS 叢集上設定 SSL 憑證

如果您的目標叢集在建立時未附加 SSL，您可以使用 Azure PowerShell Cmdlet 新增憑證。

您必須提供原始 PEM 格式的金鑰與憑證。 上述項目可讀入為 PowerShell 變數：

```
$keyValueInPemFormat = [IO.File]::ReadAllText('<path to key.pem file>')
$certValueInPemFormat = [IO.File]::ReadAllText('<path to cert.pem file>')
```
將憑證新增至叢集： 

```
Set-AzureRmMlOpCluster -ResourceGroupName my-rg -Name my-cluster -SslStatus Enabled -SslCertificate $certValueInPemFormat -SslKey $keyValueInPemFormat -SslCName foo.mycompany.com
```

## <a name="map-the-cname-and-the-ip-address"></a>對應 CNAME 與 IP 位址

在您於必要條件中選取的 CNAME 與即時前端 (FE) 的 IP 位址之間建立對應。 若要探索 FE 的 IP 位址，請執行下列命令。 輸出會顯示名為 "publicIpAddress" 的欄位，其中包含即時叢集前端的 IP 位址。 請參閱您 DNS 提供者的指示，設定從 FQDN (用於 CNAME) 到公用 IP 位址的記錄。



## <a name="auto-detection-of-a-certificate"></a>自動偵測憑證 

當您使用 "`az ml service create realtime`" 命令建立即時 Web 服務時，Azure Machine Learning 會自動偵測叢集上設定的 SSL，並以指定的憑證自動設定評分 URL。 

若要查看憑證狀態，請執行下列命令。 請注意評分 URI 的 "https" 前置詞和主機名稱中的 CNAME。 

``` 
az ml service show realtime -n <service name>
{
                "id" : "<your service id>",
                "name" : "<your service name >",
                "state" : "Succeeded",
                "createdAt" : "<datetime>”,
                "updatedAt" : "< datetime>",
                "scoringUri" : "https://<your CNAME>/api/v1/service/<service name>/score"
}
```
