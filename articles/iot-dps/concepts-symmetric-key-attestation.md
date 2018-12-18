---
title: Azure IoT 中樞裝置佈建服務 - 對稱金鑰證明
description: 本文將在概念上略述使用 IoT 中樞裝置佈建服務時的對稱金鑰證明。
author: wesmc7777
ms.author: wesmc
ms.date: 08/18/2018
ms.topic: conceptual
ms.service: iot-dps
services: iot-dps
manager: timlt
ms.openlocfilehash: 80828876ffe8b58697cfaacad4991354ac070730
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46971785"
---
# <a name="symmetric-key-attestation"></a>對稱金鑰證明

本文說明搭配裝置佈建服務使用對稱金鑰進行的身分識別證明程序。 

對稱金鑰證明是驗證裝置與裝置佈建服務執行個體的簡單方法。 對於不熟悉裝置佈建或沒有嚴格安全性需求的開發人員，這個證明方法代表 "Hello world" 經驗。 使用 [TPM](concepts-tpm-attestation.md) 或 [X.509 憑證](concepts-security.md#x509-certificates) 的裝置證明會更加安全，並且應該用於更嚴格的安全性需求。

對稱金鑰的註冊項目也相當適合安全性功能有限的舊版裝置，可透過 Azure IoT 啟動到雲端。 如需舊版裝置的對稱金鑰證明有關的詳細資訊，請參閱[如何在舊版的裝置上使用對稱金鑰](how-to-legacy-device-symm-key.md)。


## <a name="symmetric-key-creation"></a>對稱金鑰建立

在**自動產生金鑰**選項啟用的情況下儲存新的註冊項目時，裝置佈建服務預設會建立預設長度為 32 位元組的新對稱金鑰。

![自動產生對稱金鑰](./media/concepts-symmetric-key-attestation/auto-generate-keys.png)

您也可以停用此選項，以提供您自己的對稱金鑰進行註冊。 指定您自己的對稱金鑰時，金鑰長度必須介於 16 位元組到 64 位元組之間。 此外，必須以有效的 Base64 格式提供對稱金鑰。



## <a name="detailed-attestation-process"></a>詳細的證明程序

裝置佈建服務的對稱金鑰證明是使用與 IoT 中樞所支援相同的[安全性權杖](../iot-hub/iot-hub-devguide-security.md#security-token-structure)進行裝置識別。 這些安全性權杖是[共用存取簽章 (SAS) 權杖](../service-bus-messaging/service-bus-sas.md)。 

SAS 權杖包含了使用對稱金鑰所建立的雜湊*簽章*。 裝置佈建服務會重新建立簽章，以確認證明期間呈現的安全性權杖是否真實。

SAS 權杖具有下列格式：

`SharedAccessSignature sig={signature}&se={expiry}&skn={policyName}&sr={URL-encoded-resourceURI}`

下列為每個權杖的元件：

| 值 | 說明 |
| --- | --- |
| {signature} |HMAC-SHA256 簽章字串。 對於個別註冊，此簽章是使用對稱金鑰（主要或次要）來執行雜湊所產生的。 對於註冊群組，衍生自註冊群組索引鍵的索引鍵會用來執行雜湊。 雜湊是在表單的訊息上執行：`URL-encoded-resourceURI + "\n" + expiry`。 **重要事項**：金鑰必須是從 base64 解碼而來，才能用來執行 HMAC-SHA256 計算的金鑰。 此外，簽章結果必須以 URL 編碼。 |
| {resourceURI} |可以使用此權杖存取的註冊端點 URI，開頭為裝置佈建服務執行個體的範圍 ID。 例如， `{Scope ID}/registrations/{Registration ID}` |
| {expiry} |從新紀元時間 (Epoch) 1970 年 1 月 1日 00:00:00 UTC 時間至今秒數的 UTF8 字串。 |
| {URL-encoded-resourceURI} |小寫資源 URI 的小寫 URL 編碼 |
| {policyName} |此權杖所參考的共用存取原則名稱。 使用對稱金鑰證明佈建時使用的原則名稱是**註冊**。 |

以個別註冊證明裝置時，裝置會使用個別註冊項目中定義的對稱金鑰建立 SAS 權杖的雜湊簽章。

關於建立 SAS 權杖的程式碼範例，請參閱[安全性權杖](../iot-hub/iot-hub-devguide-security.md#security-token-structure)。

建立對稱金鑰證明的安全性權杖受 Azure IoT C SDK 支援。 如需使用 Azure IoT C SDK 以個別註冊證明的範例，請參閱[使用對稱金鑰佈建模擬的裝置](quick-create-simulated-device-symm-key.md)。


## <a name="group-enrollments"></a>群組註冊

佈建時，裝置不會直接使用群組註冊的對稱金鑰。 屬於註冊群組的裝置則是使用衍生的裝置金鑰進行佈建。 

首先，對於向註冊群組證明的每個裝置，都會定義唯一的註冊識別碼。 註冊識別碼的有效字元為小寫英數字元與破折號 ('-')。 這個註冊識別碼應該是識別裝置的唯一識別碼。 例如，繼承裝置可能不支援許多安全性功能。 繼承裝置可能只有唯一識別該裝置的 MAC 位址或序號。 在此情況下，註冊識別碼可由如下所示的 MAC 位址和序號組成：

```
sn-007-888-abc-mac-a1-b2-c3-d4-e5-f6
```

這個確切的範例會在[如何使用對稱金鑰佈建繼承裝置](how-to-legacy-device-symm-key.md)文章中使用。

對於裝置定義註冊識別碼後，註冊群組的對稱金鑰會用來計算註冊識別碼的 [HMAC-SHA256](https://wikipedia.org/wiki/HMAC)雜湊，以產生衍生的裝置金鑰。 下列 C# 程式碼可以執行註冊識別碼的雜湊：

```C#
using System; 
using System.Security.Cryptography; 
using System.Text;  

public static class Utils 
{ 
    public static string ComputeDerivedSymmetricKey(byte[] masterKey, string registrationId) 
    { 
        using (var hmac = new HMACSHA256(masterKey)) 
        { 
            return Convert.ToBase64String(hmac.ComputeHash(Encoding.UTF8.GetBytes(registrationId))); 
        } 
    } 
} 
```

```C#
String deviceKey = Utils.ComputeDerivedSymmetricKey(Convert.FromBase64String(masterKey), registrationId);
```

產生的裝置金鑰接著會用來產生用於證明的 SAS 權杖。 註冊群組中的每個裝置需要使用從唯一的衍生金鑰產生的安全性權杖進行證明。 註冊群組對稱金鑰不能直接用於證明。

#### <a name="installation-of-the-derived-device-key"></a>安裝衍生的裝置金鑰

在理想情況下，裝置金鑰是在工廠中衍生並安裝。 這個方法可確保群組索引鍵永遠不會包含部署至裝置的任何軟體。 裝置被指派 MAC 位址或序號時，可以衍生金鑰並插入到裝置中，不過製造商可選擇儲存金鑰。

請參看下圖中顯示的裝置金鑰表，其中的裝置金鑰是由工廠對於每個裝置註冊識別碼使用群組註冊金鑰 (**K**) 進行雜湊處理而產生的。 

![由工廠指派的裝置金鑰](./media/concepts-symmetric-key-attestation/key-diversification.png)

每個裝置的身分識別都是以註冊識別碼和在工廠安裝的衍生裝置金鑰表示。 裝置金鑰永遠不會複製到其他位置，群組金鑰永遠不會儲存在裝置上。

如果未在工廠安裝裝置金鑰，[硬體安全性模組 HSM](concepts-security.md#hardware-security-module) 應該用來安全地儲存裝置身分識別。

## <a name="next-steps"></a>後續步驟

您了解對稱金鑰證明後，請參閱下列文章以瞭解詳細資訊：

* [快速入門：使用對稱金鑰來佈建模擬的裝置](quick-create-simulated-device-symm-key.md)
* [了解自動佈建中的概念](./concepts-auto-provisioning.md)
* [開始使用自動佈建](./quick-setup-auto-provision.md) 
