---
title: Azure Machine Learning 模型管理命令列介面參考 | Microsoft Docs
description: Azure Machine Learning 模型管理命令列介面參考。
services: machine-learning
author: aashishb
ms.author: aashishb
manager: hjerez
ms.reviewer: jasonwhowell, mldocs
ms.service: machine-learning
ms.component: core
ms.workload: data-services
ms.custom: mvc
ms.topic: article
ms.date: 11/08/2017
ROBOTS: NOINDEX
ms.openlocfilehash: 6844537c512d10fccb244a18dafabe7521e697b1
ms.sourcegitcommit: 74941e0d60dbfd5ab44395e1867b2171c4944dbe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/15/2018
ms.locfileid: "49321830"
---
# <a name="model-management-command-line-interface-reference"></a>模型管理命令列介面參考

[!INCLUDE [workbench-deprecated](../../../includes/aml-deprecating-preview-2017.md)] 


## <a name="base-cli-concepts"></a>CLI 基礎概念：

    account : Manage model management accounts. 
    env     : Manage compute environments.
    image   : Manage operationalization images.
    manifest: Manage operationalization manifests.
    model   : Manage operationalization models.
    service : Manage operationalized services.

## <a name="account-commands"></a>帳戶命令
必須有模型管理帳戶才能使用服務，此帳戶可讓您部署及管理模型。 使用 `az ml account modelmanagement -h` 以查看下列清單：

    create: Create a Model Management Account.
    delete: Delete a specified Model Management Account.
    list  : Gets the Model Management Accounts in the current subscriptiong.
    set   : Set the active Model Management Account.
    show  : Show a Model Management Account.
    update: Update an existing Model Management Account.

**建立模型管理帳戶**

使用下列命令，建立模型管理帳戶以便計費：

`az ml account modelmanagement create --location [Azure region e.g. eastus2] --name [new account name] --resource-group [resource group name to store the account in]`

本機引數：

    --location -l       [Required]: Resource location.
    --name -n           [Required]: Name of the model management account.
    --resource-group -g [Required]: Resource group to create the model management account in.
    --description -d              : Description of the model management account.
    --sku-instances               : Number of instances of the selected SKU. Must be between 1 and
                                    16 inclusive.  Default: 1.
    --sku-name                    : SKU name. Valid names are S1|S2|S3|DevTest.  Default: S1.
    --tags -t                     : Tags for the model management account.  Default: {}.
    -v                            : Verbosity flag.

## <a name="environment-commands"></a>環境命令

    cluster        : Switch the current execution context to 'cluster'.
    delete         : Delete an MLCRP-provisioned resource.
    get-credentials: List the keys for an environment.
    list           : List all environments in the current subscription.
    local          : Switch the current execution context to 'local'.
    set            : Set the active MLC environment.
    setup          : Sets up an MLC environment.
    show           : Show an MLC resource; if resource_group or cluster_name are not provided, shows
                     the active MLC env.

**設定部署環境**

設定命令要求您具有訂用帳戶的參與者存取權。 如果您沒有該權限，您至少需有所要部署到資源群組的參與者存取權。 若要執行後者，您需要使用 `-g` 旗標將資源群組名稱指定為設定命令的一部分。 

有兩個部署選項：「本機」和「叢集」。 設定 `--cluster` (或 `-c`) 旗標可進行叢集部署，以佈建 ACS 叢集。 基本設定語法如下所示：

`az ml env setup [-c] --location [location of environment resources] --name[name of environment]`

這個命令會使用儲存體帳戶、ACR 登錄，以及訂用帳戶中所建立的 App Insights 服務來初始化 Azure Machine Learning 環境。 如果未指定任何旗標，則系統預設會將環境初始化為只進行本機部署 (沒有 ACS)。 如果您需要調整服務，請指定 `--cluster` (或 `-c`) 旗標以建立 ACS 叢集。

命令詳細資料：

    --location -l        [Required]: Location for environment resources; an Azure region, e.g. eastus2.
    --name -n            [Required]: Name of environment to provision.
    --acr -r                       : ARM ID of ACR to associate with this environment.
    --agent-count -z               : Number of agents to provision in the ACS cluster. Default: 3.
    --cert-cname                   : CNAME of certificate.
    --cert-pem                     : Path to .pem file with certificate bytes.
    --cluster -c                   : Flag to provision ACS cluster. Off by default; specify this to force an ACS cluster deployment.
    --key-pem                      : Path to .pem file with certificate key.
    --master-count -m              : Number of master nodes to provision in the ACS cluster. Acceptable values: 1, 3, 5. Default: 1.
    --resource-group -g            : Resource group in which to create compute resource. Is created if it does not exist.
                                     If not provided, resource group is created with 'rg' appended to 'name.'.
    --service-principal-app-id -a  : App ID of service principal to use for configuring ML compute.
    --service-principal-password -p: Password associated with service principal.
    --storage -s                   : ARM ID of storage account to associate with this environment.
    --yes -y                       : Flag to answer 'yes' to any prompts. Command fails if user is not logged in.

全域引數
```
    --debug                        : Increase logging verbosity to show all debug logs.
    --help -h                      : Show this help message and exit.
    --output -o                    : Output format.  Allowed values: json, jsonc, table, tsv. Default: json.
    --query                        : JMESPath query string. See http://jmespath.org/ for more information and examples.
    --verbose                      : Increase logging verbosity. Use --debug for full debug logs.
```
## <a name="model-commands"></a>模型命令

    list
    register
    show

**註冊模型**

用來註冊模型的命令。

`az ml model register --model [path to model file] --name [model name]`

命令詳細資料：

    --model -m [Required]: Model to register.
    --name -n  [Required]: Name of model to register.
    --description -d     : Description of the model.
    --tag -t             : Tags for the model. Multiple tags can be specified with additional -t arguments.
    -v                   : Verbosity flag.

全域引數

    --debug              : Increase logging verbosity to show all debug logs.
    --help -h            : Show this help message and exit.
    --output -o          : Output format.  Allowed values: json, jsonc, table, tsv.  Default: json.
    --query              : JMESPath query string. See http://jmespath.org/ for more information and
                           examples.
    --verbose            : Increase logging verbosity. Use --debug for full debug logs.

## <a name="manifest-commands"></a>資訊清單命令

    create: Create an Operationalization Manifest. This command has two different
            sets of required arguments, depending on if you want to use previously registered
            model/s.
    list
    show

**建立資訊清單**

下列命令會建立模型的資訊清單檔。 

`az ml manifest create --manifest-name [your new manifest name] -f [path to score file] -r [runtime for the image, e.g. spark-py]`

命令詳細資料：

    --manifest-name -n [Required]: Name of the manifest to create.
    -f                 [Required]: The score file to be deployed.
    -r                 [Required]: Runtime of the web service. Valid runtimes are spark-py|python.
    --conda-file -c              : Path to Conda Environment file.
    --dependency -d              : Files and directories required by the service. Multiple
                                   dependencies can be specified with additional -d arguments.
    --manifest-description       : Description of the manifest.
    --schema-file -s             : Schema file to add to the manifest.
    -p                           : A pip requirements.txt file needed by the score file.
    -v                           : Verbosity flag.

已註冊的模型引數

    --model-id -i                : [Required] Id of previously registered model to add to manifest.
                                   Multiple models can be specified with additional -i arguments.

已取消註冊的模型引數

    --model-file -m              : [Required] Model file to register. If used, must be combined with
                                   model name.

全域引數

    --debug                      : Increase logging verbosity to show all debug logs.
    --help -h                    : Show this help message and exit.
    --output -o                  : Output format.  Allowed values: json, jsonc, table, tsv.
                                   Default: json.
    --query                      : JMESPath query string. See http://jmespath.org/ for more
                                   information and examples.
    --verbose                    : Increase logging verbosity. Use --debug for full debug logs.


## <a name="image-commands"></a>映像命令

    create: Creates a docker image with the model and its dependencies. This command has two different sets of
            required arguments, depending on if you want to use a previously created manifest.
    list
    show
    usage

**建立映像**

您可以選擇先建立映像的資訊清單，然後再建立映像。 

`az ml image create -n [image name] --manifest-id [the manifest ID]`

或者，您也可以使用單一命令同時建立資訊清單和映像。 

`az ml image create -n [image name] --model-file [model file or folder path] -f [score file, e.g. the score.py file] -r [the runtime eg.g. spark-py which is the Docker container image base]`

命令詳細資料：

    --image-name -n [Required]: The name of the image being created.
    --image-description       : Description of the image.
    --image-type              : The image type to create. Defaults to "Docker".
    -v                        : Verbosity flag.

已註冊的資訊清單引數

    --manifest-id             : [Required] Id of previously registered manifest to use in image creation.

已取消註冊的資訊清單引數

    --conda-file -c           : Path to Conda Environment file.
    --dependency -d           : Files and directories required by the service. Multiple dependencies can
                                be specified with additional -d arguments.
    --model-file -m           : [Required] Model file to register.
    --schema-file -s          : Schema file to add to the manifest.
    -f                        : [Required] The score file to be deployed.
    -p                        : A pip requirements.txt file needed by the score file.
    -r                        : [Required] Runtime of the web service. Valid runtimes are python|spark-py.


## <a name="service-commands"></a>服務命令
服務支援下列命令。 若要查看每個命令的參數，請使用 -h 選項。 例如，使用 `az ml service create realtime -h` 來查看建立命令的詳細資料。

    create
    delete
    keys
    list
    logs
    run
    show
    update
    usage

**建立服務**

若要使用先前建立的映像來建立服務，請使用下列命令：

`az ml service create realtime --image-id [image to deploy] -n [service name]`

若要使用單一命令來建立服務、資訊清單和映像，請使用下列命令：

`az ml service create realtime --model-file [path to model file(s)] -f [path to model scoring file, e.g. score.py] -n [service name] -r [run time included in the image, e.g. spark-py]`

命令詳細資料：

    -n                                : [Required] Webservice name.
    --autoscale-enabled               : Enable automatic scaling of service replicas based on request demand.
                                        Allowed values: true, false. False if omitted.  Default: false.
    --autoscale-max-replicas          : If autoscale is enabled - sets the maximum number of replicas.
    --autoscale-min-replicas          : If autoscale is enabled - sets the minimum number of replicas.
    --autoscale-refresh-period-seconds: If autoscale is enabled - the interval of evaluating scaling demand.
    --autoscale-target-utilization    : If autoscale is enabled - target utilization of replicas time.
    --collect-model-data              : Enable model data collection. Allowed values: true, false. False if omitted.  Default: false.
    --cpu                             : Reserved number of CPU cores per service replica (can be fraction).
    --enable-app-insights -l          : Enable app insights. Allowed values: true, false. False if omitted.  Default: false.
    --memory                          : Reserved amount of memory per service replica, in M or G. (ex. 1G, 300M).
    --replica-max-concurrent-requests : Maximum number of concurrent requests that can be routed to a service replica.
    -v                                : Verbosity flag.
    -z                                : Number of replicas for a Kubernetes service.  Default: 1.

已註冊的映像引數

    --image-id                        : [Required] Image to deploy to the service.

已取消註冊的映像引數

    --conda-file -c                   : Path to Conda Environment file.
    --image-type                      : The image type to create. Defaults to "Docker".
    --model-file -m                   : [Required] The model to be deployed.
    -d                                : Files and directories required by the service. Multiple dependencies can be specified
                                        with additional -d arguments.
    -f                                : [Required] The score file to be deployed.
    -p                                : A pip requirements.txt file of package needed by the score file.
    -r                                : [Required] Runtime of the web service. Valid runtimes are python|spark-py.
    -s                                : Input and output schema of the web service.

全域引數

    --debug                           : Increase logging verbosity to show all debug logs.
    --help -h                         : Show this help message and exit.
    --output -o                       : Output format.  Allowed values: json, jsonc, table, tsv. Default: json.
    --query                           : JMESPath query string. See http://jmespath.org/ for more information and examples.
    --verbose                         : Increase logging verbosity. Use --debug for full debug logs.


請注意用來連結相依性的 `-d` 旗標：如果您傳遞尚未捆搭 (zip、tar 等) 之目錄的名稱，則該目錄會自動進行 tar 處理並傳遞，隨後再於另一端自動解除捆搭。 

如果您傳入已捆搭的目錄，系統會將該目錄視為檔案，並原封不動地進行傳遞。 該目錄將會自動解除捆搭；您應該在程式碼中處理這個程序。

**取得服務詳細資料**

取得服務詳細資料，包括 URL、使用方式 (如果已建立結構描述，則會包括資料範例)。

`az ml service show realtime --name [service name]`

命令詳細資料：

    --id -i    : The service id to show.
    --name -n  : Webservice name.
    -v         : Verbosity flag.

全域引數

    --debug    : Increase logging verbosity to show all debug logs.
    --help -h  : Show this help message and exit.
    --output -o: Output format.  Allowed values: json, jsonc, table, tsv.  Default: json.
    --query    : JMESPath query string. See http://jmespath.org/ for more information and examples.
    --verbose  : Increase logging verbosity. Use --debug for full debug logs.

**執行服務**

`az ml service run realtime -n [service name] -d [input_data]`

命令詳細資料：

    --id -i    : The service id to show.
    --name -n  : Webservice name.
    -v         : Verbosity flag.

全域引數

    --debug    : Increase logging verbosity to show all debug logs.
    --help -h  : Show this help message and exit.
    --output -o: Output format.  Allowed values: json, jsonc, table, tsv.  Default: json.
    --query    : JMESPath query string. See http://jmespath.org/ for more information and examples.
    --verbose  : Increase logging verbosity. Use --debug for full debug logs.

命令

    az ml service run realtime

引數 --id -i    ：[必要] 要作為評分依據的服務識別碼。
-d         ：用來呼叫 Web 服務的資料。
-v         ：詳細程度旗標。

全域引數

    --debug    : Increase logging verbosity to show all debug logs.
    --help -h  : Show this help message and exit.
    --output -o: Output format.  Allowed values: json, jsonc, table, tsv. Default: json.
    --query    : JMESPath query string. See http://jmespath.org/ for more information and examples.
    --verbose  : Increase logging verbosity. Use --debug for full debug logs.
