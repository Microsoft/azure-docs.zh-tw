---
title: Azure IoT 裝置 SDK 平台支援 | Microsoft Docs
description: 概念 - Azure IoT 裝置 SDK 所支援的平台清單
author: yzhong94
manager: timlt
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 04/01/2018
ms.author: yizhon
ms.openlocfilehash: 02d9a326f0209bacca17a9617374105fcbf5ed6a
ms.sourcegitcommit: 63613e4c7edf1b1875a2974a29ab2a8ce5d90e3b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/29/2018
ms.locfileid: "43189760"
---
# <a name="azure-iot-sdks-platform-support"></a>Azure IoT SDK 平台支援

[Azure IoT SDK](iot-hub-devguide-sdks.md) 是一組可透過廣泛的語言與平台支援與 IoT 中樞和裝置佈建服務進行互動的程式庫。  這些 SDK 可在最常見的平台上執行，且開發人員可依照[移轉指引](https://github.com/Azure/azure-c-shared-utility/blob/master/devdoc/porting_guide.md)將 C SDK 移轉至特定平台。 

Microsoft 支援多種作業系統/平台/架構，並且可使用 Azure IoT C SDK 加以擴充。  其中有部分受到小組的正式支援，並分組為各層，代表使用者可預期的支援程度。  完全支援的平台意味著 Microsoft 會：
    - 持續建置對主機的端對端測試和 LTS 支援的版本
    - 提供適用的安裝指南或套件
    - GitHub 上的完整支援

此外，合作夥伴清單已將我們的 C SDK 移轉至更多平台，且這些平台保有平台抽象層 (PAL)。  [Azure IoT 認證裝置目錄](https://catalog.azureiotsolutions.com/)也提供已測試過各種 SDK 的作業系統平台清單。  SDK 也會在這些平台上定期建置，並進行有限的測試和支援：
- MBED2
- Arduino
- Windows CE 2013 (在 2018 年 10 月淘汰)
- 使用 .NET Core 2.1 和 .NET Framework 4.7 的 .NET Standard 1.3
- Xamarin iOS、Android、UWP
- 使用 Java 的 Android

## <a name="supported-platforms"></a>支援的平台
### <a name="c-sdk"></a>C SDK
| 作業系統                  | Arch | 編譯器             | TLS 程式庫       |
|---------------------|------|----------------------|-------------------|
| Ubuntu 16.04 LTS    | X64  | gcc-5.4.0            | openssl - 1.0.2g |
| Ubuntu 18.04 LTS    | X64  | gcc-7.3              | WolfSSL – 1.13    |
| Ubuntu 18.04 LTS    | X64  | Clang 6.0.X          | Openssl – 1.1.0g  |
| OSX 10.13.4         | x64  | XCode 9.4.1          | 原生 OSX        |
| Windows Server 2016 | x64  | Visual Studio 14.0.X | SChannel          |
| Windows Server 2016 | x86  | Visual Studio 14.0.X | SChannel          |
| Debian 9 Stretch    | x64  | gcc-7.3              | Openssl – 1.1.0f  |

### <a name="python-sdk"></a>Python SDK
| 作業系統                  | Arch | 編譯器   | TLS 程式庫 |
|---------------------|------|------------|-------------|
| Windows Server 2016 | x86  | Python 2.7 | openssl     |
| Windows Server 2016 | x64  | Python 2.7 | openssl     |
| Windows Server 2016 | x86  | Python 3.5 | openssl     |
| Windows Server 2016 | x64  | Python 3.5 | openssl     |
| Ubuntu 18.04 LTS    | x86  | Python 2.7 | openssl     |
| Ubuntu 18.04 LTS    | x86  | Python 3.4 | openssl     |
| MacOS High Sierra   | x64  | Python 2.7 | openssl     |

### <a name="net-sdk"></a>.NET SDK
| 作業系統                  | Arch | Framework            | 標準          |
|---------------------|------|----------------------|-------------------|
| Ubuntu 16.04 LTS    | X64  | .NET Core 2.1        | .NET Standard 2.0 |
| Windows Server 2016 | X64  | .NET Core 2.1        | .NET Standard 2.0 |
| Windows Server 2016 | X64  | .NET Framework 4.7   | .NET Standard 2.0 |
| Windows Server 2016 | X64  | .NET Framework 4.5.1 | N/A               |

### <a name="nodejs-sdk"></a>Node.js SDK
| 作業系統                                           | Arch | 節點版本 |
|----------------------------------------------|------|--------------|
| Ubuntu 16.04 LTS (使用節點 6 Docker 映像) | X64  | 節點 6       |
| Windows Server 2016                          | X64  | 節點 6       |

### <a name="java-sdk"></a>Java SDK
| 作業系統                  | Arch | Java 版本 |
|---------------------|------|--------------|
| Ubuntu 16.04 LTS    | X64  | Java 8       |
| Windows Server 2016 | X64  | Java 8       |

## <a name="partner-supported-platforms"></a>合作夥伴支援的平台
客戶可以藉由移轉 Azure IoT C SDK 來擴充我們的平台支援，具體來說，他們可以建立 SDK 的平台抽象層 (PAL)。  Microsoft 會與合作夥伴一起提供擴充的支援。  有一組合作夥伴已將 C SDK 移轉至更多平台上，並同時維護 PAL。
| 合作夥伴             | 裝置                            | 連結                     | 支援 |
|---------------------|------------------------------------|--------------------------|---------|
| Espressif           | ESP32 <br/> ESP8266                              | [Esp-azure](https://github.com/espressif/esp-azure)                | [GitHub](https://github.com/espressif/esp-azure)  
| Qualcomm            | Qualcomm MDM9206 LTE IoT Modem     | [Qualcomm LTE for IoT SDK](https://developer.qualcomm.com/software/lte-iot-sdk) | [論壇](https://developer.qualcomm.com/forums/software/lte-iot-sdk)   |
| ST Microelectronics | STM32L4 系列 <br/> STM32F4 系列 <br/>  STM32F7 系列 <br/>  適用於 IoT 節點的 STM32L4 Discovery Kit    | [X-CUBE-CLOUD](https://www.st.com/content/st_com/en/products/embedded-software/mcus-embedded-software/stm32-embedded-software/stm32cube-expansion-packages/x-cube-cloud.html) <br/> [X-CUBE-AZURE](https://www.st.com/content/st_com/en/products/embedded-software/mcus-embedded-software/stm32-embedded-software/stm32cube-expansion-packages/x-cube-azure.html) <br/> [P-NUCLEO-AZURE](https://www.st.com/content/st_com/en/products/evaluation-tools/solution-evaluation-tools/communication-and-connectivity-solution-eval-boards/p-nucleo-azure1.html) <br/> [FP-CLD-AZURE](https://www.st.com/content/st_com/en/products/embedded-software/mcus-embedded-software/stm32-embedded-software/stm32-ode-function-pack-sw/fp-cld-azure1.html)            | [支援](https://www.st.com/content/st_com/en/support/support-home.html)
| Texas Instruments   | CC3220SF 啟動控制板 <br/> CC3220S 啟動控制板 <br/> MSP432E4 啟動控制板      | [適用於 SimpleLink 的 Azure IoT 外掛程式](https://github.com/TexasInstruments/azure-iot-pal-simplelink) | [TI E2E 論壇](https://e2e.ti.com) <br/> [CC3220 的 TI E2E 論壇](https://e2e.ti.com/support/wireless_connectivity/simplelink_wifi_cc31xx_cc32xx/) <br/> [MSP432E4 的 TI E2E 論壇](https://e2e.ti.com/support/microcontrollers/msp430/) |



## <a name="next-steps"></a>後續步驟
- [裝置與服務 SDK](iot-hub-devguide-sdks.md)
- [移轉指引](https://github.com/Azure/azure-c-shared-utility/blob/master/devdoc/porting_guide.md)