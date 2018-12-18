---
title: 在適用於 PostgreSQL 的 Azure 資料庫中設定 SSL 連線能力
description: 用以設定適用於 PostgreSQL 之 Azure 資料庫及相關聯應用程式以適當使用 SSL 連接的指示與資訊。
services: postgresql
author: JasonMAnderson
ms.author: janders
editor: jasonwhowell
manager: kfile
ms.service: postgresql
ms.custom: ''
ms.topic: article
ms.date: 02/28/2018
ms.openlocfilehash: f3efb99ddb47f167a0d9cbef064890e817a18841
ms.sourcegitcommit: e2adef58c03b0a780173df2d988907b5cb809c82
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/28/2018
ms.locfileid: "32186812"
---
# <a name="configure-ssl-connectivity-in-azure-database-for-postgresql"></a>在適用於 PostgreSQL 的 Azure 資料庫中設定 SSL 連線能力
適用於 PostgreSQL 的 Azure 資料庫偏好使用安全通訊端層 (SSL)，來將用戶端應用程式連接到 PostgreSQL 服務。 在您的資料庫伺服器和用戶端應用程式之間強制使用 SSL 連線，可將伺服器與應用程式之間的資料流加密，有助於抵禦「中間人」攻擊。

預設會將 PostgreSQL 資料庫服務設定為需要 SSL 連接。 (選擇性) 如果您的用戶端應用程式不支援 SSL 連線能力，您可以將需要 SSL 才能連線到資料庫服務的功能停用。 

## <a name="enforcing-ssl-connections"></a>強制使用 SSL 連線
針對透過 Azure 入口網站和 CLI 佈建之所有適用於 PostgreSQL 的 Azure 資料庫伺服器，預設會啟用強制執行 SSL 連接。 

同樣地，您在 Azure 入口網站的伺服器下方 [連接字串] 設定中預先定義的連接字串，會包含通用語言使用 SSL 連接到您資料庫伺服器所需的必要參數。 SSL 參數會根據連接器而有所不同，例如，"ssl=true" 或 "sslmode=require" 或 "sslmode=required" 及其他變化。

## <a name="configure-enforcement-of-ssl"></a>設定強制執行 SSL
您可以選擇性地停用強制執行 SSL 連線能力。 Microsoft Azure 建議您一律啟用 [強制執行 SSL 連接] 設定，以取得增強的安全性。

### <a name="using-the-azure-portal"></a>使用 Azure 入口網站
瀏覽您適用於 PostgreSQL 的 Azure 資料庫伺服器，然後按一下 [連接安全性]。 使用切換按鈕來啟用或停用 [強制使用 SSL 連線] 設定。 然後按一下 [ **儲存**]。 

![連接安全性：停用強制執行 SSL](./media/concepts-ssl-connection-security/1-disable-ssl.png)

您可以檢視 [概觀] 頁面來查看 [SSL 強制執行狀態] 指標，藉以確認設定。

### <a name="using-azure-cli"></a>使用 Azure CLI
您可以在 Azure CLI 中分別使用 `Enabled` 或 `Disabled` 值，來啟用或停用 **sssl-enforcement** 參數。

```azurecli
az postgres server update --resource-group myresourcegroup --name mydemoserver --ssl-enforcement Enabled
```

## <a name="ensure-your-application-or-framework-supports-ssl-connections"></a>確定您的應用程式或架構支援 SSL 連線
許多針對其資料庫服務使用 PostgreSQL 的常見應用程式架構 (例如 Drupal 和 Django)，都預設在安裝期間不啟用 SSL。 必須在安裝之後，或透過應用程式特定的 CLI 命令，啟用 SSL 連線能力。 如果您的 PostgreSQL 伺服器正強制執行 SSL 連接，且未正確設定相關聯的應用程式，則應用程式可能無法連接到您的資料庫伺服器。 請參閱您的應用程式文件，以了解如何啟用 SSL 連接。


## <a name="applications-that-require-certificate-verification-for-ssl-connectivity"></a>需要憑證驗證才能啟用 SSL 連線能力的應用程式
在某些情況下，應用程式需要從受信任憑證授權單位 (CA) 憑證檔案 (.cer) 所產生的本機憑證檔，才能安全地連接。 請參閱下列步驟來取得 .cer 檔案、解碼憑證，並將它繫結至您的應用程式。

### <a name="download-the-certificate-file-from-the-certificate-authority-ca"></a>從憑證授權單位 (CA) 下載憑證檔 
您可以在[這裡](https://www.digicert.com/CACerts/BaltimoreCyberTrustRoot.crt)找到要透過 SSL 與您適用於 PostgreSQL 之 Azure 資料庫伺服器通訊所需的憑證。 本機下載憑證檔。

### <a name="download-and-install-openssl-on-your-machine"></a>在您的電腦上下載並安裝 OpenSSL 
若要將應用程式所需的憑證檔解碼以便安全地連接到您的資料庫伺服器，您需要在本機電腦上安裝 OpenSSL。

#### <a name="for-linux-os-x-or-unix"></a>針對 Linux、OS X 或 Unix
[OpenSSL Software Foundation (英文)](http://www.openssl.org) 中會以原始程式碼形式直接提供 OpenSSL 程式庫。 下列指示會引導您逐步完成在 Linux 電腦上安裝 OpenSSL 的必要步驟。 這篇文章會使用已知可在 Ubuntu 12.04 和更新版本上運作的命令。

開啟終端機工作階段並下載 OpenSSL。
```bash
wget http://www.openssl.org/source/openssl-1.1.0e.tar.gz
``` 
從已下載套件中將檔案解壓縮。
```bash
tar -xvzf openssl-1.1.0e.tar.gz
```
進入已解壓縮檔案的目錄。 它預設應該如下。

```bash
cd openssl-1.1.0e
```
執行下列命令來設定 OpenSSL。 如果您想要使用與 /usr/local/openssl 不同資料夾中的檔案，請務必適當地變更下列內容。

```bash
./config --prefix=/usr/local/openssl --openssldir=/usr/local/openssl
```
既然已正確設定 OpenSSL，則您需要編譯它來轉換憑證。 若要進行編譯，請執行下列命令：

```bash
make
```
編譯完成之後，您就可以執行下列命令，以可執行檔形式安裝 OpenSSL：
```bash
make install
```
若要確認您已在系統上成功安裝 OpenSSL，請執行下列命令，然後檢查以確定您得到相同的輸出。

```bash
/usr/local/openssl/bin/openssl version
```
如果成功，您應該會看到下列訊息。
```bash
OpenSSL 1.1.0e 7 Apr 2014
```

#### <a name="for-windows"></a>若為 Windows
在 Windows 電腦上安裝 OpenSSL，可使用下列方式來完成：
1. **(建議)** 在 Window 10 和更新版本中使用內建的 Bash for Windows 功能，預設會安裝 OpenSSL。 您可以在[這裡](https://msdn.microsoft.com/commandline/wsl/install_guide)找到如何在 Window 10 啟用 Bash for Windows 功能的指示。
2. 透過下載社群所提供的 Win32/64 應用程式。 雖然 OpenSSL Software Foundation 並不提供或建議任何特定的 Windows 安裝程式，但他們在[這裡](https://wiki.openssl.org/index.php/Binaries)提供了可用的安裝程式清單。

### <a name="decode-your-certificate-file"></a>將憑證檔案解碼
下載的根 CA 檔案是加密格式。 使用 OpenSSL 來將憑證檔案解碼。 若要這樣做，請執行此 OpenSSL 命令：

```dos
openssl x509 -inform DER -in BaltimoreCyberTrustRoot.crt -text -out root.crt
```

### <a name="connecting-to-azure-database-for-postgresql-with-ssl-certificate-authentication"></a>使用 SSL 憑證驗證連接至適用於 PostgreSQL 的 Azure 資料庫
既然您已成功將憑證解碼，您現在可以透過 SSL 安全地連接到資料庫伺服器。 若要允許伺服器憑證驗證，憑證必須放置於使用者主目錄上的 ~/.postgresql/root.crt 檔案中 (在 Microsoft Windows 上，此檔案會命名為 %APPDATA%\postgresql\root.crt)。 以下提供連接到適用於 PostgreSQL 之 Azure 資料庫的指示。

#### <a name="using-psql-command-line-utility"></a>使用 psql 命令列公用程式
下列範例會示範如何使用 psql 命令列公用程式，來成功地連線到 PostgreSQL 伺服器。 使用所建立的 `root.crt` 檔案和 `sslmode=verify-ca` 或 `sslmode=verify-full` 選項。

使用 PostgreSQL 命令列介面，執行下列命令：
```bash
psql "sslmode=verify-ca sslrootcert=root.crt host=mydemoserver.postgres.database.azure.com dbname=postgres user=mylogin@mydemoserver"
```
如果成功，您應該會收到下列輸出：
```bash
Password for user mylogin@mydemoserver:
psql (9.6.2)
WARNING: Console code page (437) differs from Windows code page (1252)
     8-bit characters might not work correctly. See psql reference
     page "Notes for Windows users" for details.
SSL connection (protocol: TLSv1.2, cipher: ECDHE-RSA-AES256-SHA384, bits: 256, compression: off)
Type "help" for help.

postgres=>
```

#### <a name="using-pgadmin-gui-tool"></a>使用 pgAdmin GUI 工具
若要將 pgAdmin 4 設定為透過 SSL 進行安全連線，您必須如下所示地設定 `SSL mode = Verify-CA` 或 `SSL mode = Verify-Full`：

![pgAdmin - 連接 - 需要 SSL 模式的螢幕擷取畫面](./media/concepts-ssl-connection-security/2-pgadmin-ssl.png)

## <a name="next-steps"></a>後續步驟
在[適用於 PostgreSQL 的 Azure 資料庫的連線庫](concepts-connection-libraries.md)之後，檢閱各種應用程式連線能力選項。
