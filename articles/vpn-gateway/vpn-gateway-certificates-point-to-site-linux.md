---
title: 產生並匯出點對站的憑證：Linux：CLI：Azure | Microsoft Docs
description: 建立自我簽署的根憑證、匯出公開金鑰，以及使用 Linux (strongSwan) CLI 產生用戶端憑證。
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: article
ms.date: 09/06/2018
ms.author: cherylmc
ms.openlocfilehash: 8647b3b3eda980dbd5d5ec368b6b4b13949ecaf1
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44305721"
---
# <a name="generate-and-export-certificates-for-point-to-site-using-linux-strongswan-cli"></a>使用 Linux strongSwan CLI 產生並匯出點對站的憑證

點對站連線使用憑證進行驗證。 本文說明如何建立自我簽署的根憑證，以及使用 Linux CLI 和 strongSwan 產生用戶端憑證。 如果您要尋找不同的憑證指示，請參閱 [Powershell](vpn-gateway-certificates-point-to-site.md) 或 [MakeCert](vpn-gateway-certificates-point-to-site-makecert.md) 文章。

> [!NOTE]
> 本文中的步驟需要 strongSwan。
>

本文中的步驟所使用的電腦組態如下：

| | |
|---|---|
|**電腦**| Ubuntu Server 16.04<br>ID_LIKE=debian<br>PRETTY_NAME="Ubuntu 16.04.4 LTS"<br>VERSION_ID="16.04" |
|**相依項目**| apt-get install strongswan-ikev2 strongswan-plugin-eap-tls<br>apt-get install libstrongswan-standard-plugins |

## <a name="install-strongswan"></a>安裝 strongSwan

1. `apt-get install strongswan-ikev2 strongswan-plugin-eap-tls`
2. `apt-get install libstrongswan-standard-plugins`

如需如何使用 GUI 安裝 strongSwan 的相關資訊，請參閱[用戶端組態](point-to-site-vpn-client-configuration-azure-cert.md#install)一文中的步驟。

## <a name="generate-keys-and-certificate"></a>產生金鑰和憑證

1. 產生 CA 憑證。

  ```
  ipsec pki --gen --outform pem > caKey.pem
  ipsec pki --self --in caKey.pem --dn "CN=VPN CA" --ca --outform pem > caCert.pem
  ```
2. 列印 base64 格式的 CA 憑證。 這是 Azure 支援的格式。 您會稍後會將其上傳至 Azure 作為您 P2S 組態的一部分。

  ```
  openssl x509 -in caCert.pem -outform der | base64 -w0 ; echo
  ```
3. 產生使用者憑證。

  ```
  export PASSWORD="password"
  export USERNAME="client"

  ipsec pki --gen --outform pem > "${USERNAME}Key.pem"
  ipsec pki --pub --in "${USERNAME}Key.pem" | ipsec pki --issue --cacert caCert.pem --cakey caKey.pem --dn "CN=${USERNAME}" --san "${USERNAME}" --flag clientAuth --outform pem > "${USERNAME}Cert.pem"
  ```
4. 產生包含使用者憑證的 p12 組合。 在後續使用[用戶端組態檔](point-to-site-vpn-client-configuration-azure-cert.md#linuxinstallcli)的步驟中將會用到此組合。

  ```
  openssl pkcs12 -in "${USERNAME}Cert.pem" -inkey "${USERNAME}Key.pem" -certfile caCert.pem -export -out "${USERNAME}.p12" -password "pass:${PASSWORD}"
  ```

## <a name="next-steps"></a>後續步驟

請繼續進行點對站設定，以[建立和安裝 VPN 用戶端組態檔](point-to-site-vpn-client-configuration-azure-cert.md#linuxinstallcli)。