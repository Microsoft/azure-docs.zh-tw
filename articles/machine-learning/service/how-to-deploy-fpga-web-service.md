---
title: 使用 Azure Machine Learning 在 FPGA 上將模型部署為 Web 服務
description: 了解如何使用 Azure Machine Learning 以 FPGA 上執行的模型來部署 Web 服務。
services: machine-learning
ms.service: machine-learning
ms.component: core
ms.topic: conceptual
ms.reviewer: jmartens
ms.author: tedway
author: tedway
ms.date: 09/24/2018
ms.openlocfilehash: ee67585a523ab96b1442d9eee3e9dfd55a758d32
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46971479"
---
# <a name="deploy-a-model-as-a-web-service-on-an-fpga-with-azure-machine-learning"></a>使用 Azure Machine Learning 在 FPGA 上將模型部署為 Web 服務

您可以將模型部署為[現場可程式化邏輯閘陣列 (FPGA)](concept-accelerate-with-fpgas.md) 上的 Web 服務。  使用 FPGA 時，即使是透過單一批次大小，也提供超低延遲推斷。   

## <a name="prerequisites"></a>必要條件

- Azure 訂用帳戶。 如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

- 已安裝 Azure Machine Learning 工作區與適用於 Python 的 Azure Machine Learning SDK。 了解如何使用[如何設定開發環境](how-to-configure-environment.md)文件來取得這些必要條件。
 
  - 您的工作區必須位於*美國東部 2* 區域。

  - 安裝 contrib extras：

    ```shell
    pip install --upgrade azureml-sdk[contrib]
    ```  

## <a name="create-and-deploy-your-model"></a>建立及部署您的模型
建立管道以預先處理輸入映像、在 FPGA 上使用 ResNet 50 將它功能化，然後透過在 ImageNet 資料集上定型的分類器執行功能。

依照指示執行以：

* 定義模型管道
* 部署模型
* 取用已部署的模型
* 刪除已部署的服務

> [!IMPORTANT]
> 若要將延遲和輸送量最佳化，您的用戶端應該位於與端點相同 Azure 區域中。  目前 API 是在美國東部的 Azure 區域中建立的。

### <a name="get-the-notebook"></a>取得 Notebook

為了方便起見，此教學課程以 Jupyter Notebook 形式提供。 使用這些方法的其中一種方法來執行 `project-brainwave/project-brainwave-quickstart.ipynb` Notebook :

[!INCLUDE [aml-clone-in-azure-notebook](../../../includes/aml-clone-in-azure-notebook.md)]

### <a name="preprocess-image"></a>預先處理映像
管道的第一個步驟是預先處理映像。

```python
import os
import tensorflow as tf

# Input images as a two-dimensional tensor containing an arbitrary number of images represented a strings
import azureml.contrib.brainwave.models.utils as utils
in_images = tf.placeholder(tf.string)
image_tensors = utils.preprocess_array(in_images)
print(image_tensors.shape)
```
### <a name="add-featurizer"></a>加入功能化器
初始化模型並下載要做為功能化器之 ResNet50 量化版本的 TensorFlow 檢查點。

```python
from azureml.contrib.brainwave.models import QuantizedResnet50, Resnet50
model_path = os.path.expanduser('~/models')
model = QuantizedResnet50(model_path, is_frozen = True)
feature_tensor = model.import_graph_def(image_tensors)
print(model.version)
print(feature_tensor.name)
print(feature_tensor.shape)
```

### <a name="add-classifier"></a>新增分類器
此分類器已在 ImageNet 資料集上定型。

```python
classifier_input, classifier_output = Resnet50.get_default_classifier(feature_tensor, model_path)
```

### <a name="create-service-definition"></a>建立服務定義
您現在已定義映像預先處理、功能化器與在服務上執行的分類器，您可以建立服務定義。 服務定義是從部署到 FPGA 服務之模型產生的一組檔案。 服務定義由管道組成。 管道是依序執行的一系列階段。  支援 TensorFlow 階段、Keras 階段與 BrainWave 階段。  階段會在服務上依序執行，而且每個階段的輸出會做為後續階段的輸入。

若要建立 TensorFlow 階段，請指定包含圖形 (在此案例中會使用預設圖形) 和此階段之輸入與輸出 tensors 的工作階段。  此資訊會用來將儲存圖形，以便它可以在服務上執行。

```python
from azureml.contrib.brainwave.pipeline import ModelDefinition, TensorflowStage, BrainWaveStage

save_path = os.path.expanduser('~/models/save')
model_def_path = os.path.join(save_path, 'service_def.zip')

model_def = ModelDefinition()
with tf.Session() as sess:
    model_def.pipeline.append(TensorflowStage(sess, in_images, image_tensors))
    model_def.pipeline.append(BrainWaveStage(sess, model))
    model_def.pipeline.append(TensorflowStage(sess, classifier_input, classifier_output))
    model_def.save(model_def_path)
    print(model_def_path)
```

### <a name="deploy-model"></a>部署模型
從服務定義建立服務。  您的工作區必須位於 美國東部 2 位置。

```python
from azureml.core import Workspace

ws = Workspace.from_config()
print(ws.name, ws.resource_group, ws.location, ws.subscription_id, sep = '\n')

from azureml.core.model import Model
model_name = "resnet-50-rtai"
registered_model = Model.register(ws, model_def_path, model_name)

from azureml.core.webservice import Webservice
from azureml.exceptions import WebserviceException
from azureml.contrib.brainwave import BrainwaveWebservice, BrainwaveImage
service_name = "imagenet-infer"
service = None
try:
    service = Webservice(ws, service_name)
except WebserviceException:
    image_config = BrainwaveImage.image_configuration()
    deployment_config = BrainwaveWebservice.deploy_configuration()
    service = Webservice.deploy_from_model(ws, service_name, [registered_model], image_config, deployment_config)
    service.wait_for_deployment(true)
```

### <a name="test-the-service"></a>測試服務
若要將影像傳送到 API 並測試回應，請新增從輸出類別識別碼到 ImageNet 類別名稱的對應。

```python
import requests
classes_entries = requests.get("https://raw.githubusercontent.com/Lasagne/Recipes/master/examples/resnet50/imagenet_classes.txt").text.splitlines()
```

呼叫您的服務並將下面的 "your-image.jpg" 檔案名稱取代為來自您電腦的影像。 

```python
with open('your-image.jpg') as f:
    results = service.run(f)
# map results [class_id] => [confidence]
results = enumerate(results)
# sort results by confidence
sorted_results = sorted(results, key=lambda x: x[1], reverse=True)
# print top 5 results
for top in sorted_results[:5]:
    print(classes_entries[top[0]], 'confidence:', top[1])
``` 

### <a name="clean-up-service"></a>清除服務
刪除服務。

```python
service.delete()
    
registered_model.delete()
```

## <a name="secure-fpga-web-services"></a>保護 FPGA Web 服務

在 FPGA 上執行的 Azure Machine Learning 模型提供 SSL 支援和金鑰型驗證。 這可讓您限制對服務的存取，並保護用戶端所提交的資料。

> [!IMPORTANT]
> 系統只會針對已經提供 SSL 憑證和金鑰的服務來啟用驗證。 
>
> 如果您未啟用 SSL，則網際網路上的任何使用者都將能夠呼叫服務。
>
> 如果您啟用了 SSL，則在存取服務時，便會要求驗證金鑰。

SSL 會加密在用戶端與服務之間傳送的資料。 用戶端也會使用 SSL 來驗證伺服器的身分識別。

您可以在 SSL 啟用時部署服務，或更新已部署的服務來加以啟用。 兩者步驟相同：

1. 取得網域名稱。

2. 取得 SSL 憑證。

3. 在 SSL 啟用時部署或更新服務。

4. 將您的 DNS 更新為指向此服務。

### <a name="acquire-a-domain-name"></a>取得網域名稱

如果您還未擁有網域名稱，可以向__網域名稱註冊機構__購買。 不同的註冊機構有不同的流程，而且費用也不同。 註冊機構也會為您提供管理網域名稱所使用的工具。 這類工具可用來將完整的網域名稱 (例如 www.contoso.com) 對應到裝載服務所在的 IP 位址。

### <a name="acquire-an-ssl-certificate"></a>取得 SSL 憑證

有很多種方法可以取得 SSL 憑證。 最常見的作法是向__憑證授權單位__ (CA) 購買。 不論您在哪裡取得憑證，都需要下列檔案：

* __憑證__。 憑證必須包含完整的憑證鏈結，而且必須以 PEM 編碼。
* __金鑰__。 此金鑰必須以 PEM 編碼。

> [!TIP]
> 如果憑證授權單位無法以 PEM 編碼的檔案來提供憑證和金鑰，您可以使用 [OpenSSL](https://www.openssl.org/) 之類的公用程式來變更格式。

> [!IMPORTANT]
> 自我簽署的憑證應只用於開發。 這些憑證不應用於實際生產環境。
>
> 如果您使用自我簽署的憑證，請參閱[使用自我簽署的憑證取用服務](#self-signed)一節的特定指示。

> [!WARNING]
> 要求憑證時，您必須提供打算用於服務的位址完整網域名稱 (FQDN)。 例如，www.contoso.com。 驗證服務的身分識別時，會比較憑證上的位址戳記和用戶端所使用的位址。
>
> 如果位址不相符，用戶端就會收到錯誤。 

### <a name="deploy-or-update-the-service-with-ssl-enabled"></a>在 SSL 啟用時部署或更新服務

若要在 SSL 啟用時部署服務，請將 `ssl_enabled` 參數設定為 `True`。 將 `ssl_certificate` 參數設定為__憑證__檔案的值，並將 `ssl_key` 設定為__金鑰__檔案的值。 下列範例會示範在 SSL 啟用時部署服務：

```python
from amlrealtimeai import DeploymentClient

subscription_id = "<Your Azure Subscription ID>"
resource_group = "<Your Azure Resource Group Name>"
model_management_account = "<Your AzureML Model Management Account Name>"
location = "eastus2"

model_name = "resnet50-model"
service_name = "quickstart-service"

deployment_client = DeploymentClient(subscription_id, resource_group, model_management_account, location)

with open('cert.pem','r') as cert_file:
    with open('key.pem','r') as key_file:
        cert = cert_file.read()
        key = key_file.read()
        service = deployment_client.create_service(service_name, model_id, ssl_enabled=True, ssl_certificate=cert, ssl_key=key)
```

`create_service` 作業的回應包含服務的 IP 位址。 將 DNS 名稱對應至服務的 IP 位址時，將會使用此 IP 位址。

回應也包含取用服務所使用的__主要金鑰__和__次要金鑰__。

### <a name="update-your-dns-to-point-to-the-service"></a>將您的 DNS 更新為指向此服務

使用網域名稱註冊機構所提供的工具來更新您網域名稱的 DNS 記錄。 這筆記錄必須指向此服務的 IP 位址。

> [!NOTE]
> 根據註冊機構以及針對網域名稱所設定的存留時間 (TTL)，可能需要等待數分鐘到數小時的時間，用戶端才能解析網域名稱。

### <a name="consume-authenticated-services"></a>取用已驗證的服務

下列範例會示範如何使用 Python 和 C# 來取用已驗證的服務：

> [!NOTE]
> 將 `authkey` 取代為建立服務時所傳回的主要或次要金鑰。

```python
from amlrealtimeai import PredictionClient
client = PredictionClient(service.ipAddress, service.port, use_ssl=True, access_token="authKey")
image_file = R'C:\path_to_file\image.jpg'
results = client.score_image(image_file)
```

```csharp
var client = new ScoringClient(host, 50051, useSSL, "authKey");
float[,] result;
using (var content = File.OpenRead(image))
    {
        IScoringRequest request = new ImageRequest(content);
        result = client.Score<float[,]>(request);
    }
```

其他 gRPC 用戶端可以透過設定授權標頭來驗證要求。 一般方法是建立結合 `SslCredentials` 與 `CallCredentials` 的 `ChannelCredentials` 物件。 這會新增至要求的授權標頭。 如需有關如何為特定標頭實作支援的詳細資訊，請參閱 [https://grpc.io/docs/guides/auth.html](https://grpc.io/docs/guides/auth.html)。

下列範例示範如何使用 C# 與 Go 來設定標頭：

```csharp
creds = ChannelCredentials.Create(baseCreds, CallCredentials.FromInterceptor(
                      async (context, metadata) =>
                      {
                          metadata.Add(new Metadata.Entry("authorization", "authKey"));
                          await Task.CompletedTask;
                      }));

```

```go
conn, err := grpc.Dial(serverAddr, 
    grpc.WithTransportCredentials(credentials.NewClientTLSFromCert(nil, "")),
    grpc.WithPerRPCCredentials(&authCreds{
    Key: "authKey"}))

type authCreds struct {
    Key string
}

func (c *authCreds) GetRequestMetadata(context.Context, uri ...string) (map[string]string, error) {
    return map[string]string{
        "authorization": c.Key,
    }, nil
}

func (c *authCreds) RequireTransportSecurity() bool {
    return true
}
```

### <a id="self-signed"></a>使用自我簽署的憑證來取用服務

針對受到自我簽署憑證保護的伺服器，有兩種方式可讓用戶端向伺服器進行驗證：

* 在用戶端系統上，將用戶端系統上的 `GRPC_DEFAULT_SSL_ROOTS_FILE_PATH` 環境變數設定為指向憑證檔案。

* 建構 `SslCredentials` 物件時，將憑證檔案的內容傳遞給建構函式。

使用任一種方法都會導致 gRPC 使用憑證作為根憑證。

> [!IMPORTANT]
> gRPC 不會接受未受信任的憑證。 使用未受信任的憑證將會失敗，並顯示 `Unavailable` 狀態碼。 失敗的詳細資料包含 `Connection Failed`。
