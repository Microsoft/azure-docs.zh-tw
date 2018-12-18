---
title: 使用 Azure IoT 中樞裝置佈建服務 SDK 中提供的工具來簡化開發
description: 本文件將檢閱 Azure IoT 中樞裝置佈建服務 SDK 中提供來用於開發的工具
author: yzhong94
ms.author: yizhon
ms.date: 04/09/2018
ms.topic: conceptual
ms.service: iot-dps
services: iot-dps
manager: arjmands
ms.openlocfilehash: 647f54d8252c594a280f81d661a3de6270bf692b
ms.sourcegitcommit: e0a678acb0dc928e5c5edde3ca04e6854eb05ea6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/13/2018
ms.locfileid: "39001342"
---
# <a name="how-to-use-tools-provided-in-the-sdks-to-simplify-development-for-provisioning"></a>如何使用 SDK 中提供的工具來簡化開發以進行佈建
IoT 中樞裝置佈建服務會以安全且可擴充的方式，透過零接觸的 Just-In-Time [自動佈建](concepts-auto-provisioning.md)來簡化佈建程序。  需要有 X.509 憑證或信賴平台模組 (TPM) 形式的安全性證明。  Microsoft 也與[其他安全性硬體合作夥伴](https://azure.microsoft.com/blog/azure-iot-supports-new-security-hardware-to-strengthen-iot-security/) \(英文\) 合作，以提升保護 IoT 部署安全的信心。 對於開發人員而言，了解硬體安全性需求可能會是相當大的挑戰。 我們提供一組 Azure IoT 佈建服務 SDK，讓開發人員能夠輕鬆撰寫要與佈建服務交談之用戶端。 SDK 也提供適用於常見案例的範例以及一組工具，來簡化開發中的安全性證明。

## <a name="trusted-platform-module-tpm-simulator"></a>信賴平台模組 (TPM) 模擬器
[TPM](https://docs.microsoft.com/azure/iot-dps/concepts-security#trusted-platform-module-tpm) 可以參考安全地儲存金鑰以驗證平台的標準，或者參考用來與實作標準之模組互動的 I/O 介面。 TPM 可以是獨立硬體、整合式硬體、韌體或軟體。  在生產環境中，TPM 位於裝置上，作為個別的硬體、整合式硬體，或是以韌體為基礎。 在測試階段，會針對開發人員提供以軟體為基礎的 TPM 模擬器。  這個模擬器目前只適合用來在 Windows 平台上進行開發。

使用 TPM 模擬器的步驟如下：
1. [準備開發環境](https://docs.microsoft.com/azure/iot-dps/quick-enroll-device-x509-java#prepare-the-development-environment)並複製 GitHub 存放庫：
```
git clone https://github.com/Azure/azure-iot-sdk-java.git
```
2. 瀏覽至 ```azure-iot-sdk-java/provisioning/provisioning-tool/tpm-simulator/``` 底下的 TPM 模擬器資料夾。
3. 先執行 Simulator.exe，然後再執行任何用戶端應用程式來佈建裝置。
4. 讓模擬器於整個佈建程序期間在背景執行，以取得註冊識別碼和簽署金鑰。  這兩個值僅適用於執行的一個執行個體。

## <a name="x509-certificate-generator"></a>X.509 憑證產生器
[X.509 憑證](https://docs.microsoft.com/azure/iot-dps/concepts-security#x509-certificates)可用來作為證明機制，以調整生產環境並簡化裝置佈建。  有[數種方式](https://docs.microsoft.com/azure/iot-hub/iot-hub-x509ca-overview#how-to-get-an-x509-ca-certificate)可取得 X.509 憑證：
* 針對生產環境，建議向公用根憑證授權單位購買 X.509 CA 憑證。
* 針對測試環境，您可以使用下列方式來產生 X.509 根憑證或 X.509 憑證鏈結：
    * OpenSSL：您可以使用指令碼來產生憑證：
        * [Node.js](https://github.com/Azure/azure-iot-sdk-node/tree/master/provisioning/tools)
        * [PowerShell 或 Bash](https://github.com/Azure/azure-iot-sdk-c/blob/master/tools/CACertificates/CACertificateOverview.md)
        
    * 裝置識別組合引擎 (DICE) 模擬器：可根據 TLS 通訊協定和 X.509 用戶端憑證，將 DICE 用來進行密碼編譯裝置身分識別和證明。  [深入了解](https://www.microsoft.com/research/publication/device-identity-dice-riot-keys-certificates/) \(英文\) 搭配 DICE 的裝置身分識別。

### <a name="using-x509-certificate-generator-with-dice-emulator"></a>使用搭配 DICE 模擬器的 X.509 憑證產生器
SDK 提供搭配 DICE 模擬器的 X.509 憑證產生器，其位於 [Java SDK](https://github.com/Azure/azure-iot-sdk-java/tree/master/provisioning/provisioning-tools/provisioning-x509-cert-generator) \(英文\)。  這個產生器可跨平台使用。  產生的憑證可用來以其他語言進行開發。

目前，DICE 模擬器會輸出根憑證、中繼憑證、分葉憑證和相關聯的私密金鑰。  不過，根憑證或中繼憑證無法用於簽署個別的分葉憑證。  如果您想要測試使用單一簽署憑證來簽署多個裝置之分葉憑證的群組註冊案例，您可以使用 OpenSSL 來產生憑證鏈結。

使用這個產生器來產生 X.509 憑證：
1. [準備開發環境](https://docs.microsoft.com/azure/iot-dps/quick-enroll-device-x509-java#prepare-the-development-environment)並複製 GitHub 存放庫：
```
git clone https://github.com/Azure/azure-iot-sdk-java.git
```
2. 將根變更為 azure-iot-sdk-java。
3. 執行 ```mvn install -DskipTests=true```以下載所有的必要套件並編譯 SDK
4. 瀏覽至 X.509 憑證產生器位於 ```azure-iot-sdk-java/provisioning/provisioning-tools/provisioning-x509-cert-generator``` 中的根。
5. 使用 ```mvn clean install``` 進行建置
6. 使用下列命令來執行工具：
```
cd target
java -jar ./provisioning-x509-cert-generator-{version}-with-deps.jar
```
7. 出現提示時，您可以選擇輸入憑證的_一般名稱_。
8. 此工具會在本機產生**用戶端憑證**、**用戶端憑證私密金鑰**、**中繼憑證**及**根憑證**。

**用戶端憑證**是裝置上的分葉憑證。  **用戶端憑證**和相關聯的**用戶端憑證私密金鑰**都是裝置用戶端中所需的項目。 根據您選擇的語言而定，將此項目置於用戶端應用程式中的機制可能有所不同。  如需詳細資訊，請參閱關於使用 X.509 建立模擬裝置的[快速入門](https://docs.microsoft.com/azure/iot-dps/quick-create-simulated-device-x509)。

根憑證或中繼憑證均可[以程式設計方式](https://docs.microsoft.com/azure/iot-dps/how-to-manage-enrollments-sdks)或使用[入口網站](https://docs.microsoft.com/azure/iot-dps/how-to-manage-enrollments)來建立註冊群組或個別註冊。

## <a name="next-steps"></a>後續步驟
* 針對 Azure IoT 中樞和 Azure IoT 中樞裝置佈建服務，使用 [Azure IoT SDK]( https://github.com/Azure/azure-iot-sdks) \(英文\) 進行開發