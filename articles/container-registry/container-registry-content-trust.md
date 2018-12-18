---
title: Azure Container Registry 中的內容信任
description: 了解如何為您的 Azure Container Registry 啟用內容信任，以及推送和提取已簽署的映像。
services: container-registry
author: dlepow
ms.service: container-registry
ms.topic: quickstart
ms.date: 08/20/2018
ms.author: danlep
ms.openlocfilehash: 1f1e7ce41c00078c0181fc5f32c43b7e5885eef8
ms.sourcegitcommit: 67abaa44871ab98770b22b29d899ff2f396bdae3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/08/2018
ms.locfileid: "48857664"
---
# <a name="content-trust-in-azure-container-registry"></a>Azure Container Registry 中的內容信任

對於任何在設計時將安全性納入考量的分散式系統，都必須對進入系統的資料進行*來源*和*完整性*驗證。 資料的取用者必須要能夠驗證資料的發行者 (來源)，並確定資料在發佈後並未經過修改 (完整性)。 Azure Container Registry 可藉由實作 Docker 的[內容信任][docker-content-trust]模型而支援這兩項作業，本文將為您提供入門解說。

> [!IMPORTANT]
> 此功能目前為預覽狀態。 若您同意[補充的使用規定][terms-of-use]，即可取得預覽。 在公開上市 (GA) 之前，此功能的某些領域可能會變更。

## <a name="how-content-trust-works"></a>內容信任的運作方式

身為映像發行者，內容信任可讓您**簽署**您推送至登錄的映像。 映像的取用者 (從您的登錄提取映像的人員或系統) 可將其用戶端設定成*僅*提取已簽署的映像。 當映像取用者提取已簽署的映像時，其 Docker 用戶端會驗證映像的完整性。 在此模型中，取用者可確知您的登錄中已簽署的映像確實是由您所發佈，且在發佈後未經修改。

### <a name="trusted-images"></a>信任的映像

內容信任會使用存放庫中的**標記**。 映像存放庫可包含具有已簽署和未簽署標記的映像。 例如，您可能會僅簽署 `myimage:stable` 和 `myimage:latest` 映像，而未簽署 `myimage:dev`。

### <a name="signing-keys"></a>簽署金鑰

內容信任可使用一組密碼編譯簽署金鑰來管理。 這些金鑰會與登錄中的特定存放庫相關聯。 Docker 用戶端和您的登錄會使用數種類型的簽署金鑰，根據存放庫中的標記來管理信任。 當您啟用內容信任，並將其整合至容器發佈和取用管線時，您必須謹慎管理這些金鑰。 如需詳細資訊，請參閱本文稍後的[金鑰管理](#key-management)，以及 Docker 文件中的[管理內容信任的金鑰][docker-manage-keys]。

> [!TIP]
> 本文僅就 Docker 的內容信任模型提供最基本的概觀。 如需內容信任的深入討論，請參閱 [Docker 中的內容信任][docker-content-trust]。

## <a name="enable-registry-content-trust"></a>啟用登錄內容信任

首要步驟是在登錄層級啟用內容信任。 啟用內容信任後，用戶端 (使用者或服務) 即可將已簽署的映像推送至您的登錄。 在您的登錄上啟用內容信任時，並不會限定只有已啟用內容信任的取用者才能使用登錄。 未啟用內容信任的取用者仍可繼續如常使用您的登錄。 不過，已在其用戶端中啟用內容信任的取用者，將*只能*在您的登錄中看到已簽署的映像。

若要為您的登錄啟用內容信任，請先瀏覽至 Azure 入口網站中的登錄。 在 [原則] 下方，選取 [內容信任 (預覽)] > [已啟用] > [儲存]。

![在 Azure 入口網站中為登錄啟用內容信任][content-trust-01-portal]

## <a name="enable-client-content-trust"></a>啟用用戶端內容信任

若要使用受信任的映像，映像發行者和取用者都必須為其 Docker 用戶端啟用內容信任。 身為發行者，您可以簽署您對已啟用內容信任的登錄推送的映像。 身為取用者，啟用內容信任後，您所能檢視的登錄將僅限於已簽署的映像。 在 Docker 用戶端中依預設會停用內容信任，但您可以依據殼層工作階段或命令加以啟用。

若要啟用殼層工作階段的內容信任，請將 `DOCKER_CONTENT_TRUST` 環境變數設定為 **1**。 例如，在 Bash 殼層中：

```bash
# Enable content trust for shell session
export DOCKER_CONTENT_TRUST=1
```

如果您想要改為單一命令啟用或停用內容信任，有數個 Docker 命令可支援 `--disable-content-trust` 引數。 若要啟用單一命令的內容信任：

```bash
# Enable content trust for single command
docker build --disable-content-trust=false -t myacr.azurecr.io/myimage:v1 .
```

如果您已啟用殼層工作階段的內容信任，而想要針對單一命令加以停用：

```bash
# Disable content trust for single command
docker build --disable-content-trust -t myacr.azurecr.io/myimage:v1 .
```

## <a name="grant-image-signing-permissions"></a>授與映像簽署權限

只有您已授與權限的使用者或系統，才可將信任的映像推送至您的登錄。 若要為使用者 (或使用服務主體的系統) 授與信任的映像推送權限，請為其 Azure Active Directory 身分識別授與 `AcrImageSigner` 角色。 除了 `Contributor` (或 `Owner`) 以外，還需要此角色才能將映像推送至登錄。

以下是在 Azure 入口網站和 Azure CLI 中授與 `AcrImageSigner` 角色的詳細資料。

### <a name="azure-portal"></a>Azure 入口網站

在 Azure 入口網站中瀏覽至您的登錄，然後選取 [存取控制 (IAM)] > [新增]。 在 [新增權限] 下方，選取 [角色] 下的 `AcrImageSigner`，然後**選取**一或多個使用者或服務主體，再按一下 [儲存]。

在此範例中，有兩個實體已被指派 `AcrImageSigner` 角色：名為 "service-principal" 的服務主體，以及名為 "Azure User" 的使用者。

![在 Azure 入口網站中為登錄啟用內容信任][content-trust-02-portal]

### <a name="azure-cli"></a>Azure CLI

若要使用 Azure CLI 為使用者授與簽署權限，請將 `AcrImageSigner` 角色指派給使用者，並將範圍設定為您的登錄。 此命令的格式為：

```azurecli
az role assignment create --scope <registry ID> --role AcrImageSigner --assignee <user name>
```

例如，若要為您自己授與此角色，您可以在已驗證的 Azure CLI 工作階段中執行下列命令。 請修改 `REGISTRY` 值，以反映您的 Azure Container Registry 名稱。

```bash
# Grant signing permissions to authenticated Azure CLI user
REGISTRY=myregistry
USER=$(az account show --query user.name --output tsv)
REGISTRY_ID=$(az acr show --name $REGISTRY --query id --output tsv)

az role assignment create --scope $REGISTRY_ID --role AcrImageSigner --assignee $USER
```

您也可以為[服務主體](container-registry-auth-service-principal.md)授與將信任的映像推送至登錄的權限。 在建置系統和其他必須將信任的映像推送至登錄的自動化系統時，使用服務主體將有其效用。 其格式和授與使用者權限相似，但會指定服務主體識別碼作為 `--assignee` 值。

```azurecli
az role assignment create --scope $REGISTRY_ID --role AcrImageSigner --assignee <service principal ID>
```

`<service principal ID>` 可以是服務主體的 **appId**、**objectId**，或其 **servicePrincipalNames** 之一。 如需使用服務主體與 Azure Container Registry 的詳細資訊，請參閱[使用服務主體進行 Azure Container Registry 驗證](container-registry-auth-service-principal.md)。

## <a name="push-a-trusted-image"></a>推送信任的映像

若要將信任的映像標記推送至您的容器登錄，請啟用內容信任，並使用 `docker push` 推送映像。 當您第一次推送已簽署的標記時，系統會要求您建立根簽署金鑰和存放庫簽署金鑰的複雜密碼。 根金鑰與存放庫金鑰都會在您的本機電腦產生並儲存。

```console
$ docker push myregistry.azurecr.io/myimage:v1
The push refers to repository [myregistry.azurecr.io/myimage]
ee83fc5847cb: Pushed
v1: digest: sha256:aca41a608e5eb015f1ec6755f490f3be26b48010b178e78c00eac21ffbe246f1 size: 524
Signing and pushing trust metadata
You are about to create a new root signing key passphrase. This passphrase
will be used to protect the most sensitive key in your signing system. Please
choose a long, complex passphrase and be careful to keep the password and the
key file itself secure and backed up. It is highly recommended that you use a
password manager to generate the passphrase and keep it safe. There will be no
way to recover this key. You can find the key in your config directory.
Enter passphrase for new root key with ID 4c6c56a:
Repeat passphrase for new root key with ID 4c6c56a:
Enter passphrase for new repository key with ID bcd6d98:
Repeat passphrase for new repository key with ID bcd6d98:
Finished initializing "myregistry.azurecr.io/myimage"
Successfully signed myregistry.azurecr.io/myimage:v1
```

在您第一次執行已啟用內容信任的 `docker push` 後，Docker 用戶端後續將會使用相同的根金鑰進行推送。 在後續每次推送至相同的存放庫時，系統都只會要求您提供存放庫金鑰。 每當您將信任的映像推送至新的存放庫時，系統都會要求您提供新存放庫金鑰的複雜密碼。

## <a name="pull-a-trusted-image"></a>提取信任的映像

若要提取的信任的映像，請啟用內容信任，並如常執行 `docker pull` 命令。 已啟用內容信任的取用者只能提取具有已簽署標記的映像。 以下範例說明如何提取已簽署的標記︰

```console
$ docker pull myregistry.azurecr.io/myimage:signed
Pull (1 of 1): myregistry.azurecr.io/myimage:signed@sha256:0800d17e37fb4f8194495b1a188f121e5b54efb52b5d93dc9e0ed97fce49564b
sha256:0800d17e37fb4f8194495b1a188f121e5b54efb52b5d93dc9e0ed97fce49564b: Pulling from myimage
8e3ba11ec2a2: Pull complete
Digest: sha256:0800d17e37fb4f8194495b1a188f121e5b54efb52b5d93dc9e0ed97fce49564b
Status: Downloaded newer image for myregistry.azurecr.io/myimage@sha256:0800d17e37fb4f8194495b1a188f121e5b54efb52b5d93dc9e0ed97fce49564b
Tagging myregistry.azurecr.io/myimage@sha256:0800d17e37fb4f8194495b1a188f121e5b54efb52b5d93dc9e0ed97fce49564b as myregistry.azurecr.io/myimage:signed
```

如果已啟用內容信任的用戶端嘗試提取未簽署的標記，作業將會失敗：

```console
$ docker pull myregistry.azurecr.io/myimage:unsigned
No valid trust data for unsigned
```

### <a name="behind-the-scenes"></a>在幕後

當您執行 `docker pull` 時，Docker 用戶端會使用與 [Notary CLI][docker-notary-cli] 中相同的程式庫，針對您要提取的標記要求標記與 SHA-256 的摘要對應。 在驗證信任資料的簽章後，用戶端會指示 Docker 引擎執行「依照摘要的提取」。 在提取期間，引擎會使用 SHA-256 總和檢查碼作為內容位址，以要求並驗證來自 Azure Container Registry 的映像資訊清單。

## <a name="key-management"></a>金鑰管理

如同您第一次推送信任的映像時的 `docker push` 輸出所顯示，根金鑰是敏感性最高的資料。 請務必備份您的根金鑰，並將其儲存在安全之處。 根據預設，Docker 用戶端會將簽署金鑰儲存在下列目錄中：

```sh
~/.docker/trust/private
```

請將您的根金鑰和存放庫金鑰壓縮在封存中，並儲存到安全的離線位置 (例如 USB 儲存裝置上)，而加以備份。 例如，在 Bash 中：

```bash
umask 077; tar -zcvf docker_private_keys_backup.tar.gz ~/.docker/trust/private; umask 022
```

除了在本機產生的根金鑰和存放庫金鑰以外，當您推送信任的映像時，Azure Container Registry 還會產生並儲存數個其他的金鑰。 如需 Docker 內容信任實作中各種金鑰的詳細討論 (包括額外的管理指南)，請參閱 Docker 文件中的[管理內容信任的金鑰][docker-manage-keys]。

### <a name="lost-root-key"></a>遺失的根金鑰

如果您無法存取根金鑰，您將無法對任何以該金鑰簽署標記的存放庫存取已簽署的標記。 Azure Container Registry 無法對以遺失的根金鑰簽署的映像標記進行存取權還原。 若要移除登錄的所有信任資料 (簽章)，請先停用登錄的內容信任，然後再重新啟用。

> [!WARNING]
> 在您的登錄中停用並重新啟用內容信任，會對**您的登錄每個存放庫中所有的已簽署標記刪除所有的信任資料**。 此動作無法復原--Azure Container Registry 無法復原已刪除的信任資料。 停用內容信任並不會刪除映像本身。

若要為您的登錄停用內容信任，請瀏覽至 Azure 入口網站中的登錄。 在 [原則] 下方，選取 [內容信任 (預覽)] > [已停用] > [儲存]。 系統會警告您登錄中的所有簽章都將遺失。 選取 [確定] 會永久刪除您登錄中的所有簽章。

![在 Azure 入口網站中為登錄停用內容信任][content-trust-03-portal]

## <a name="next-steps"></a>後續步驟

請參閱 Docker 文件，以取得關於內容信任的其他資訊。 本文已討論若干要點，但內容信任涉及的主題十分廣泛，這在 Docker 文件中會有更深入的說明。

[Docker 中的內容信任][docker-content-trust]

<!-- IMAGES> -->
[content-trust-01-portal]: ./media/container-registry-content-trust/content-trust-01-portal.png
[content-trust-02-portal]: ./media/container-registry-content-trust/content-trust-02-portal.png
[content-trust-03-portal]: ./media/container-registry-content-trust/content-trust-03-portal.png

<!-- LINKS - external -->
[docker-content-trust]: https://docs.docker.com/engine/security/trust/content_trust
[docker-manage-keys]: https://docs.docker.com/engine/security/trust/trust_key_mng/
[docker-notary-cli]: https://docs.docker.com/notary/getting_started/
[docker-push]: https://docs.docker.com/engine/reference/commandline/push/
[docker-tag]: https://docs.docker.com/engine/reference/commandline/tag/
[terms-of-use]: https://azure.microsoft.com/support/legal/preview-supplemental-terms/

<!-- LINKS - internal -->
[azure-cli]: /cli/azure/install-azure-cli
