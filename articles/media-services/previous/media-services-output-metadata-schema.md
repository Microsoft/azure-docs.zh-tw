---
title: Azure 媒體服務輸出中繼資料結構描述 | Microsoft Docs
description: 本主題提供 Azure 媒體服務輸出中繼資料結構描述的概觀。
author: Juliako
manager: cfowler
editor: ''
services: media-services
documentationcenter: ''
ms.assetid: 1ed84c88-eea5-4a24-9c4f-f2428157d08a
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/19/2017
ms.author: juliako
ms.openlocfilehash: 4babceb9454a229903c54aab7083c5e5ed138b8e
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33783827"
---
# <a name="output-metadata"></a>輸出中繼資料
## <a name="overview"></a>概觀
編碼作業會與您要在其上執行一些編碼工作的輸入資產相關聯。 例如，將 MP4 檔案編碼為 H.264 MP4 自動調整位元速率集、建立縮圖、建立疊加層。 完成工作時，就會產生輸出資產。  輸出資產包含視訊、音訊、縮圖等等。輸出資產也包含隨附關於輸出資產中繼資料的檔案。 中繼資料 XML 檔案的名稱具備下列格式︰&lt;source_file_name&gt;_manifest.xml (例如：BigBuckBunny_manifest.xml)。  

如果您想要檢查中繼資料檔案，可以建立 **SAS** 定位器並將檔案下載到您的本機電腦。  

本文討論輸出中繼資料 (&lt;來源檔案名稱&gt;_manifest.xml) 所根據之 XML 結構描述的元素和類型。 如需包含輸入資產相關中繼資料的檔案相關資訊，請參閱[輸入中繼資料](media-services-input-metadata-schema.md)。  

> [!NOTE]
> 您可以在本文結尾找到完整的結構描述程式碼和 XML 範例。  
>
>

## <a name="AssetFiles "></a> AssetFiles 根元素
編碼作業的 AssetFile 項目集合。  

### <a name="child-elements"></a>子元素
| Name | 說明 |
| --- | --- |
| **AssetFile**<br/><br/> minOccurs="0" maxOccurs="1" |屬於 AssetFiles 集合的 [AssetFile 元素](media-services-output-metadata-schema.md)。 |

## <a name="AssetFile "></a> AssetFile 元素
您可以找到 XML 範例 [XML 範例](media-services-output-metadata-schema.md#xml)。  

### <a name="attributes"></a>屬性
| Name | 類型 | 說明 |
| --- | --- | --- |
| **名稱**<br/><br/> 必要 |**xs:string** |媒體資產檔案名稱。 |
| **大小**<br/><br/> minInclusive ="0"<br/><br/> 必要 |**xs:long** |資產檔案大小 (以位元組為單位)。 |
| **Duration**<br/><br/> 必要 |**xs:duration** |內容播放持續時間。 |

### <a name="child-elements"></a>子元素
| Name | 說明 |
| --- | --- |
| **來源** |為了產生此 AssetFile 所處理之輸入/來源媒體檔案的集合。 如需詳細資訊，請參閱[來源元素](media-services-output-metadata-schema.md)。 |
| **VideoTracks**<br/><br/> minOccurs="0" maxOccurs="1" |每個實體 AssetFile 都可以內含零或多個交錯形成適當容器格式的視訊播放軌。 如需詳細資訊，請參閱 [VideoTracks 元素](media-services-output-metadata-schema.md)。 |
| **AudioTracks**<br/><br/> minOccurs="0" maxOccurs="1" |每個實體資產檔案可以內含零個或多個交錯形成適當容器格式的音訊播放軌。 這是所有音訊播放軌的集合。 如需詳細資訊，請參閱 [AudioTracks 元素](media-services-output-metadata-schema.md)。 |

## <a name="Sources "></a> 來源元素
為了產生此 AssetFile 所處理之輸入/來源媒體檔案的集合。  

您可以找到 XML 範例 [XML 範例](media-services-output-metadata-schema.md#xml)。  

### <a name="child-elements"></a>子元素
| Name | 說明 |
| --- | --- |
| **來源**<br/><br/> minOccurs="1" maxOccurs="unbounded" |產生此資產時所使用的輸入/來源檔案。 如需詳細資訊，請參閱[來源元素](media-services-output-metadata-schema.md)。 |

## <a name="Source "></a> 來源元素
產生此資產時所使用的輸入/來源檔案。  

您可以找到 XML 範例 [XML 範例](media-services-output-metadata-schema.md#xml)。  

### <a name="attributes"></a>屬性
| Name | 類型 | 說明 |
| --- | --- | --- |
| **名稱**<br/><br/> 必要 |**xs:string** |輸入的來源檔案名稱。 |

## <a name="VideoTracks "></a> VideoTracks 元素
每個實體 AssetFile 都可以內含零或多個交錯形成適當容器格式的視訊播放軌。 **VideoTracks** 元素代表所有視訊播放軌集合。  

您可以找到 XML 範例 [XML 範例](media-services-output-metadata-schema.md#xml)。  

### <a name="child-elements"></a>子元素
| Name | 說明 |
| --- | --- |
| **VideoTrack**<br/><br/> minOccurs="1" maxOccurs="unbounded" |父系 AssetFile 中的特定視訊播放軌。 如需詳細資訊，請參閱 [VideoTrack 元素](media-services-output-metadata-schema.md#VideoTrack)。 |

## <a name="VideoTrack"></a> VideoTrack 元素
父系 AssetFile 中的特定視訊播放軌。  

您可以找到 XML 範例 [XML 範例](media-services-output-metadata-schema.md#xml)。  

### <a name="attributes"></a>屬性
| Name | 類型 | 說明 |
| --- | --- | --- |
| **Id**<br/><br/> minInclusive ="0"<br/><br/> 必要 |**xs:int** |此視訊播放軌之以零為起始的索引。**注意︰** 此**識別碼**不一定是用於 MP4 檔案中的 TrackID。 |
| **FourCC**<br/><br/> 必要 |**xs:string** |視訊轉碼器 FourCC 代碼。 |
| **設定檔** |**xs:string** |H264 設定檔 (僅適用於 H264 轉碼器)。 |
| **Level** |**xs:string** |H264 層級 (僅適用於 H264 轉碼器)。 |
| **Width**<br/><br/> minInclusive ="0"<br/><br/> 必要 |**xs:int** |編碼的視訊寬度 (以像素為單位)。 |
| **Height**<br/><br/> minInclusive ="0"<br/><br/> 必要 |**xs:int** |編碼的視訊高度 (以像素為單位)。 |
| **DisplayAspectRatioNumerator**<br/><br/> minInclusive ="0"<br/><br/> 必要 |**xs:double** |視訊顯示長寬比的分子。 |
| **DisplayAspectRatioDenominator**<br/><br/> minInclusive ="0"<br/><br/> 必要 |**xs:double** |視訊顯示長寬比的分母。 |
| **Framerate**<br/><br/> minInclusive ="0"<br/><br/> 必要 |**xs:decimal** |測量的視訊畫面格速率 (採用 .3f 格式)。 |
| **TargetFramerate**<br/><br/> minInclusive ="0"<br/><br/> 必要 |**xs:decimal** |預設的目標視訊畫面格速率 (採用 .3f 格式)。 |
| **Bitrate**<br/><br/> minInclusive ="0"<br/><br/> 必要 |**xs:int** |從 AssetFile 計算出來的平均視訊位元速率 (千位元 / 秒)。 只會計算基本串流承載，而不會納入封裝負荷。 |
| **TargetBitrate**<br/><br/> minInclusive ="0"<br/><br/> 必要 |**xs:int** |此視訊播放軌依透過編碼預設值所要求的目標平均位元速率 (Kbps)。 |
| **MaxGOPBitrate**<br/><br/> minInclusive ="0" |**xs:int** |此視訊播放軌的最大 GOP 平均位元速率 (千位元 / 秒)。 |

## <a name="AudioTracks "></a> AudioTracks 元素
每個實體資產檔案可以內含零個或多個交錯形成適當容器格式的音訊播放軌。 **AudioTracks** 元素代表所有這些音訊播放軌集合。  

您可以找到 XML 範例 [XML 範例](media-services-output-metadata-schema.md#xml)。  

### <a name="child-elements"></a>子元素
| Name | 說明 |
| --- | --- |
| **AudioTrack**<br/><br/> minOccurs="1" maxOccurs="unbounded" |父系 AssetFile 中的特定音訊播放軌。 如需詳細資訊，請參閱 [AudioTrack 元素](media-services-output-metadata-schema.md)。 |

## <a name="AudioTrack "></a> AudioTrack 元素
父系 AssetFile 中的特定音訊播放軌。  

您可以找到 XML 範例 [XML 範例](media-services-output-metadata-schema.md#xml)。  

### <a name="attributes"></a>屬性
| Name | 類型 | 說明 |
| --- | --- | --- |
| **Id**<br/><br/> minInclusive ="0"<br/><br/> 必要 |**xs:int** |此音訊播放軌之以零為起始的索引。**注意︰** 這不一定是用於 MP4 檔中的 TrackID。 |
| **Codec** |**xs:string** |音訊播放軌轉碼器字串。 |
| **EncoderVersion** |**xs:string** |選用的編碼器版本字串，為 EAC3 所需。 |
| **Channels**<br/><br/> minInclusive ="0"<br/><br/> 必要 |**xs:int** |音訊聲道數目。 |
| **SamplingRate**<br/><br/> minInclusive ="0"<br/><br/> 必要 |**xs:int** |音訊取樣率 (每秒或每 Hz 的樣本數)。 |
| **Bitrate**<br/><br/> minInclusive ="0"<br/><br/> 必要 |**xs:int** |從 AssetFile 計算出來的平均音訊位元速率 (位元 / 秒)。 只會計算基本串流承載，而不會納入封裝負荷。 |
| **BitsPerSample**<br/><br/> minInclusive ="0"<br/><br/> 必要 |**xs:int** |wFormatTag 格式類型的每樣本位元數。 |

### <a name="child-elements"></a>子元素
| Name | 說明 |
| --- | --- |
| **LoudnessMeteringResultParameters**<br/><br/> minOccurs="0" maxOccurs="1" |音量計量結果參數。 如需詳細資訊，請參閱 [LoudnessMeteringResultParameters 元素](media-services-output-metadata-schema.md)。 |

## <a name="LoudnessMeteringResultParameters "></a> LoudnessMeteringResultParameters 元素
音量計量結果參數。  

您可以找到 XML 範例 [XML 範例](media-services-output-metadata-schema.md#xml)。  

### <a name="attributes"></a>屬性
| Name | 類型 | 說明 |
| --- | --- | --- |
| **DPLMVersionInformation** |**xs:string** |**Dolby** 專業音量計量開發套件版本。 |
| **DialogNormalization**<br/><br/> minInclusive="-31" maxInclusive="-1"<br/><br/> 必要 |**xs:int** |透過 DPLM 產生的 DialogNormalization，若設定了 LoudnessMetering 則為必要 |
| **IntegratedLoudness**<br/><br/> minInclusive="-70" maxInclusive="10"<br/><br/> 必要 |**xs:float** |整合的音量 |
| **IntegratedLoudnessUnit**<br/><br/> 必要 |**xs:string** |整合的音量單位。 |
| **IntegratedLoudnessGatingMethod**<br/><br/> 必要 |**xs:string** |閘控識別碼 |
| **IntegratedLoudnessSpeechPercentage**<br/><br/> minInclusive ="0" maxInclusive="100" |**xs:float** |透過程式的語音內容，以百分比表示。 |
| **SamplePeak**<br/><br/> 必要 |**xs:float** |自重設或上次清除後，每個聲道的尖峰絕對樣本值。  單位是 dBFS。 |
| **SamplePeakUnit**<br/><br/> fixed="dBFS"<br/><br/> 必要 |**xs:anySimpleType** |樣本尖峰單位。 |
| **TruePeak**<br/><br/> 必要 |**xs:float** |根據 ITU-R BS.1770-2，自重設或上次清除後，每個聲道的最大實際尖峰值。 單位是 dBTP。 |
| **TruePeakUnit**<br/><br/> fixed="dBTP"<br/><br/> 必要 |**xs:anySimpleType** |實際尖峰單位。 |

## <a name="schema-code"></a>結構描述程式碼
    <?xml version="1.0" encoding="utf-8"?>  
    <xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema" xmlns:msdata="urn:schemas-microsoft-com:xml-msdata" version="1.2"  
               xmlns="http://schemas.microsoft.com/windowsazure/mediaservices/2013/05/mediaencoder/metadata"  
               targetNamespace="http://schemas.microsoft.com/windowsazure/mediaservices/2013/05/mediaencoder/metadata"  
               elementFormDefault="qualified">  
      <xs:element name="AssetFiles">  
        <xs:annotation>  
          <xs:documentation>Collection of AssetFile entries for the encoding job</xs:documentation>  
        </xs:annotation>  
        <xs:complexType>  
          <xs:sequence>  
            <xs:element name="AssetFile" minOccurs="1" maxOccurs="unbounded">  
              <xs:annotation>  
                <xs:documentation>asset file</xs:documentation>  
              </xs:annotation>  
              <xs:complexType>  
                <xs:sequence>  
                  <xs:element name="Sources">  
                    <xs:annotation>  
                      <xs:documentation>Collection of input/source media files, that was processed in order to produce this AssetFile</xs:documentation>  
                    </xs:annotation>  
                    <xs:complexType>  
                      <xs:sequence>  
                        <xs:element name="Source" minOccurs="1" maxOccurs="unbounded">  
                          <xs:annotation>  
                            <xs:documentation>An input/source file used when generating this asset</xs:documentation>  
                          </xs:annotation>  
                          <xs:complexType>  
                            <xs:attribute name="Name" type="xs:string" use="required">  
                              <xs:annotation>  
                                <xs:documentation>input source file name</xs:documentation>  
                              </xs:annotation>  
                            </xs:attribute>  
                          </xs:complexType>  
                        </xs:element>  
                      </xs:sequence>  
                    </xs:complexType>  
                  </xs:element>  
                  <xs:element name="VideoTracks" minOccurs="0">  
                    <xs:annotation>  
                      <xs:documentation>Each physical AssetFile can contain in it zero or more video tracks interleaved into an appropriate container format. This is the collection of all those video tracks</xs:documentation>  
                    </xs:annotation>  
                    <xs:complexType>  
                      <xs:sequence>  
                        <xs:element name="VideoTrack" maxOccurs="unbounded">  
                          <xs:annotation>  
                            <xs:documentation>A specific video track in the parent AssetFile</xs:documentation>  
                          </xs:annotation>  
                          <xs:complexType>  
                            <xs:attribute name="Id" use="required">  
                              <xs:annotation>  
                                <xs:documentation>zero-based index of this video track. Note: this is not necessarily the TrackID as used in an MP4 file</xs:documentation>  
                              </xs:annotation>  
                              <xs:simpleType>  
                                <xs:restriction base="xs:int">  
                                  <xs:minInclusive value="0"/>  
                                </xs:restriction>  
                              </xs:simpleType>  
                            </xs:attribute>  
                            <xs:attribute name="FourCC" type="xs:string" use="required">  
                              <xs:annotation>  
                                <xs:documentation>video codec FourCC code</xs:documentation>  
                              </xs:annotation>  
                            </xs:attribute>  
                            <xs:attribute name="Profile" type="xs:string">  
                              <xs:annotation>  
                                <xs:documentation>H264 profile (only appliable for H264 codec)</xs:documentation>  
                              </xs:annotation>  
                            </xs:attribute>  
                            <xs:attribute name="Level" type="xs:string">  
                              <xs:annotation>  
                                <xs:documentation>H264 level (only appliable for H264 codec)</xs:documentation>  
                              </xs:annotation>  
                            </xs:attribute>  
                            <xs:attribute name="Width" use="required">  
                              <xs:annotation>  
                                <xs:documentation>encoded video width in pixels</xs:documentation>  
                              </xs:annotation>  
                              <xs:simpleType>  
                                <xs:restriction base="xs:int">  
                                  <xs:minInclusive value="0"/>  
                                </xs:restriction>  
                              </xs:simpleType>  
                            </xs:attribute>  
                            <xs:attribute name="Height" use="required">  
                              <xs:annotation>  
                                <xs:documentation>encoded video height in pixels</xs:documentation>  
                              </xs:annotation>  
                              <xs:simpleType>  
                                <xs:restriction base="xs:int">  
                                  <xs:minInclusive value="0"/>  
                                </xs:restriction>  
                              </xs:simpleType>  
                            </xs:attribute>  
                            <xs:attribute name="DisplayAspectRatioNumerator" use="required">  
                              <xs:annotation>  
                                <xs:documentation>video display aspect ratio numerator</xs:documentation>  
                              </xs:annotation>  
                              <xs:simpleType>  
                                <xs:restriction base="xs:double">  
                                  <xs:minInclusive value="0"/>  
                                </xs:restriction>  
                              </xs:simpleType>  
                            </xs:attribute>  
                            <xs:attribute name="DisplayAspectRatioDenominator" use="required">  
                              <xs:annotation>  
                                <xs:documentation>video display aspect ratio denominator</xs:documentation>  
                              </xs:annotation>  
                              <xs:simpleType>  
                                <xs:restriction base="xs:double">  
                                  <xs:minInclusive value="0"/>  
                                </xs:restriction>  
                              </xs:simpleType>  
                            </xs:attribute>  
                            <xs:attribute name="Framerate" use="required">  
                              <xs:annotation>  
                                <xs:documentation>measured video frame rate in .3f format</xs:documentation>  
                              </xs:annotation>  
                              <xs:simpleType>  
                                <xs:restriction base="xs:decimal">  
                                  <xs:minInclusive value="0"/>  
                                  <xs:fractionDigits value="3"/>  
                                </xs:restriction>  
                              </xs:simpleType>  
                            </xs:attribute>  
                            <xs:attribute name="TargetFramerate" use="required">  
                              <xs:annotation>  
                                <xs:documentation>preset target video frame rate in .3f format</xs:documentation>  
                              </xs:annotation>  
                              <xs:simpleType>  
                                <xs:restriction base="xs:decimal">  
                                  <xs:minInclusive value="0"/>  
                                  <xs:fractionDigits value="3"/>  
                                </xs:restriction>  
                              </xs:simpleType>  
                            </xs:attribute>  
                            <xs:attribute name="Bitrate" use="required">  
                              <xs:annotation>  
                                <xs:documentation>average video bit rate in kilobits per second, as calculated from the AssetFile. Counts only the elementary stream payload, and does not include the packaging overhead</xs:documentation>  
                              </xs:annotation>  
                              <xs:simpleType>  
                                <xs:restriction base="xs:int">  
                                  <xs:minInclusive value="0"/>  
                                </xs:restriction>  
                              </xs:simpleType>  
                            </xs:attribute>  
                            <xs:attribute name="TargetBitrate" use="required">  
                              <xs:annotation>  
                                <xs:documentation>target average bitrate for this video track, as requested via the encoding preset, in kilobits per second</xs:documentation>  
                              </xs:annotation>  
                              <xs:simpleType>  
                                <xs:restriction base="xs:int">  
                                  <xs:minInclusive value="0"/>  
                                </xs:restriction>  
                              </xs:simpleType>  
                            </xs:attribute>  
                            <xs:attribute name="MaxGOPBitrate">  
                              <xs:annotation>  
                                <xs:documentation>Max GOP average bitrate for this video track, in kilobits per second</xs:documentation>  
                              </xs:annotation>  
                              <xs:simpleType>  
                                <xs:restriction base="xs:int">  
                                  <xs:minInclusive value="0"/>  
                                </xs:restriction>  
                              </xs:simpleType>  
                            </xs:attribute>  
                          </xs:complexType>  
                        </xs:element>  
                      </xs:sequence>  
                    </xs:complexType>  
                  </xs:element>  
                  <xs:element name="AudioTracks" minOccurs="0">  
                    <xs:annotation>  
                      <xs:documentation>each physical AssetFile can contain in it zero or more audio tracks interleaved into an appropriate container format. This is the collection of all those audio tracks</xs:documentation>  
                    </xs:annotation>  
                    <xs:complexType>  
                      <xs:sequence>  
                        <xs:element name="AudioTrack" maxOccurs="unbounded">  
                          <xs:annotation>  
                            <xs:documentation>a specific audio track in the parent AssetFile</xs:documentation>  
                          </xs:annotation>  
                          <xs:complexType>  
                            <xs:sequence>  
                              <xs:element name="LoudnessMeteringResultParameters" minOccurs="0" maxOccurs="1">  
                                <xs:annotation>  
                                  <xs:documentation>Loudness Metering Result Parameters</xs:documentation>  
                                </xs:annotation>  
                                <xs:complexType>  
                                  <xs:attribute name="DPLMVersionInformation" type="xs:string">  
                                    <xs:annotation>  
                                      <xs:documentation>Dolby Professional Loudness Metering Development Kit Version</xs:documentation>  
                                    </xs:annotation>  
                                  </xs:attribute>  
                                  <xs:attribute name="DialogNormalization" use="required">  
                                    <xs:annotation>  
                                      <xs:documentation> DialogNormalization generated through DPLM, required when LoudnessMetering is set</xs:documentation>  
                                    </xs:annotation>  
                                    <xs:simpleType>  
                                      <xs:restriction base="xs:int">  
                                        <xs:minInclusive value="-31"/>  
                                        <xs:maxInclusive value="-1"/>  
                                      </xs:restriction>  
                                    </xs:simpleType>  
                                  </xs:attribute>  
                                  <xs:attribute name="IntegratedLoudness" use="required">  
                                    <xs:annotation>  
                                      <xs:documentation>Integrated loudness</xs:documentation>  
                                    </xs:annotation>  
                                    <xs:simpleType>  
                                      <xs:restriction base="xs:float">  
                                        <xs:minInclusive value="-70"/>  
                                        <xs:maxInclusive value="10"/>  
                                      </xs:restriction>  
                                    </xs:simpleType>  
                                  </xs:attribute>  
                                  <xs:attribute name="IntegratedLoudnessUnit" use="required" type="xs:string">  
                                  </xs:attribute>  
                                  <xs:attribute name="IntegratedLoudnessGatingMethod" use="required" type="xs:string">  
                                    <xs:annotation>  
                                      <xs:documentation>Gating identifier</xs:documentation>  
                                    </xs:annotation>  
                                  </xs:attribute>  
                                  <xs:attribute name="IntegratedLoudnessSpeechPercentage">  
                                    <xs:annotation>  
                                      <xs:documentation>Speech content over the program, as a percentage.</xs:documentation>  
                                    </xs:annotation>  
                                    <xs:simpleType>  
                                      <xs:restriction base="xs:float">  
                                        <xs:minInclusive value="0"/>  
                                        <xs:maxInclusive value="100"/>  
                                      </xs:restriction>  
                                    </xs:simpleType>  
                                  </xs:attribute>  
                                  <xs:attribute name="SamplePeak" use="required" type="xs:float">  
                                    <xs:annotation>  
                                      <xs:documentation>Peak absolute sample value, since reset or since it was last cleared, per channel.  Units are dBFS.</xs:documentation>  
                                    </xs:annotation>  
                                  </xs:attribute>  
                                  <xs:attribute name="SamplePeakUnit" use="required" fixed="dBFS">  
                                  </xs:attribute>  
                                  <xs:attribute name="TruePeak" use="required" type="xs:float">  
                                    <xs:annotation>  
                                      <xs:documentation>Maximum True Peak value, as per ITU-R BS.1770-2, since reset or since it was last cleared, per channel.  Units are dBTP.</xs:documentation>  
                                    </xs:annotation>  
                                  </xs:attribute>  
                                  <xs:attribute name="TruePeakUnit" use="required" fixed="dBTP">  
                                  </xs:attribute>  
                                </xs:complexType>  
                              </xs:element>  
                            </xs:sequence>  
                            <xs:attribute name="Id" use="required">  
                              <xs:annotation>  
                                <xs:documentation>zero-based index of this audio track. Note: this is not necessarily the TrackID as used in an MP4 file</xs:documentation>  
                              </xs:annotation>  
                              <xs:simpleType>  
                                <xs:restriction base="xs:int">  
                                  <xs:minInclusive value="0"/>  
                                </xs:restriction>  
                              </xs:simpleType>  
                            </xs:attribute>  
                            <xs:attribute name="Codec" type="xs:string">  
                              <xs:annotation>  
                                <xs:documentation>audio track codec string</xs:documentation>  
                              </xs:annotation>  
                            </xs:attribute>  
                            <xs:attribute name="EncoderVersion" type="xs:string">  
                              <xs:annotation>  
                                <xs:documentation>optional encoder version string, required for EAC3</xs:documentation>  
                              </xs:annotation>  
                            </xs:attribute>  
                            <xs:attribute name="Channels" use="required">  
                              <xs:annotation>  
                                <xs:documentation>number of audio channels</xs:documentation>  
                              </xs:annotation>  
                              <xs:simpleType>  
                                <xs:restriction base="xs:int">  
                                  <xs:minInclusive value="0"/>  
                                </xs:restriction>  
                              </xs:simpleType>  
                            </xs:attribute>  
                            <xs:attribute name="SamplingRate" use="required">  
                              <xs:annotation>  
                                <xs:documentation>audio sampling rate in samples/sec or Hz</xs:documentation>  
                              </xs:annotation>  
                              <xs:simpleType>  
                                <xs:restriction base="xs:int">  
                                  <xs:minInclusive value="0"/>  
                                </xs:restriction>  
                              </xs:simpleType>  
                            </xs:attribute>  
                            <xs:attribute name="Bitrate" use="required">  
                              <xs:annotation>  
                                <xs:documentation>average audio bit rate in bits per second, as calculated from the AssetFile. Counts only the elementary stream payload, and does not include the packaging overhead</xs:documentation>  
                              </xs:annotation>  
                              <xs:simpleType>  
                                <xs:restriction base="xs:int">  
                                  <xs:minInclusive value="0"/>  
                                </xs:restriction>  
                              </xs:simpleType>  
                            </xs:attribute>  
                            <xs:attribute name="BitsPerSample" use="required">  
                              <xs:annotation>  
                                <xs:documentation>Bits per sample for the wFormatTag format type</xs:documentation>  
                              </xs:annotation>  
                              <xs:simpleType>  
                                <xs:restriction base="xs:int">  
                                  <xs:minInclusive value="0"/>  
                                </xs:restriction>  
                              </xs:simpleType>  
                            </xs:attribute>  
                          </xs:complexType>  
                        </xs:element>  
                      </xs:sequence>  
                    </xs:complexType>  
                  </xs:element>  
                </xs:sequence>  
                <xs:attribute name="Name" type="xs:string" use="required">  
                  <xs:annotation>  
                    <xs:documentation>the media asset file name</xs:documentation>  
                  </xs:annotation>  
                </xs:attribute>  
                <xs:attribute name="Size" use="required">  
                  <xs:annotation>  
                    <xs:documentation>size of file in bytes</xs:documentation>  
                  </xs:annotation>  
                  <xs:simpleType>  
                    <xs:restriction base="xs:long">  
                      <xs:minInclusive value="0"/>  
                    </xs:restriction>  
                  </xs:simpleType>  
                </xs:attribute>  
                <xs:attribute name="Duration" use="required">  
                  <xs:annotation>  
                    <xs:documentation>content play back duration</xs:documentation>  
                  </xs:annotation>  
                  <xs:simpleType>  
                    <xs:restriction base="xs:duration"/>  
                  </xs:simpleType>  
                </xs:attribute>  
              </xs:complexType>  
            </xs:element>  
          </xs:sequence>  
        </xs:complexType>  
      </xs:element>  
    </xs:schema>  



## <a name="xml"></a> XML 範例

下列 XML 是輸出中繼資料檔案的範例。  

    <AssetFiles xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema"   
                xmlns="http://schemas.microsoft.com/windowsazure/mediaservices/2013/05/mediaencoder/metadata">  
      <AssetFile Name="BigBuckBunny_H264_3400kbps_AAC_und_ch2_96kbps.mp4" Size="4646283" Duration="PT8.4288444S">  
        <Sources>  
          <Source Name="BigBuckBunny.mp4"/>  
        </Sources>  
        <VideoTracks>  
          <VideoTrack Id="0" FourCC="AVC1" Profile="Main" Level="3.2" Width="1280" Height="720" DisplayAspectRatioNumerator="16" DisplayAspectRatioDenominator="9" Framerate="23.974" TargetFramerate="23.974" Bitrate="4250" TargetBitrate="3400" MaxGOPBitrate="5514"/>  
        </VideoTracks>  
        <AudioTracks>  
          <AudioTrack Id="0" Codec="AacLc" Channels="2" SamplingRate="44100" Bitrate="93" BitsPerSample="16"/>  
        </AudioTracks>  
      </AssetFile>  
      <AssetFile Name="BigBuckBunny_H264_2250kbps_AAC_und_ch2_96kbps.mp4" Size="3166728" Duration="PT8.4288444S">  
        <Sources>  
          <Source Name="BigBuckBunny.mp4"/>  
        </Sources>  
        <VideoTracks>  
          <VideoTrack Id="0" FourCC="AVC1" Profile="Main" Level="3.1" Width="960" Height="540" DisplayAspectRatioNumerator="16" DisplayAspectRatioDenominator="9" Framerate="23.974" TargetFramerate="23.974" Bitrate="2846" TargetBitrate="2250" MaxGOPBitrate="3630"/>  
        </VideoTracks>  
        <AudioTracks>  
          <AudioTrack Id="0" Codec="AacLc" Channels="2" SamplingRate="44100" Bitrate="93" BitsPerSample="16"/>  
        </AudioTracks>  
      </AssetFile>  
      <AssetFile Name="BigBuckBunny_H264_1500kbps_AAC_und_ch2_96kbps.mp4" Size="2205095" Duration="PT8.4288444S">  
        <Sources>  
          <Source Name="BigBuckBunny.mp4"/>  
        </Sources>  
        <VideoTracks>  
          <VideoTrack Id="0" FourCC="AVC1" Profile="Main" Level="3.1" Width="960" Height="540" DisplayAspectRatioNumerator="16" DisplayAspectRatioDenominator="9" Framerate="23.974" TargetFramerate="23.974" Bitrate="1932" TargetBitrate="1500" MaxGOPBitrate="2513"/>  
        </VideoTracks>  
        <AudioTracks>  
          <AudioTrack Id="0" Codec="AacLc" Channels="2" SamplingRate="44100" Bitrate="93" BitsPerSample="16"/>  
        </AudioTracks>  
      </AssetFile>  
      <AssetFile Name="BigBuckBunny_H264_1000kbps_AAC_und_ch2_96kbps.mp4" Size="1508567" Duration="PT8.4288444S">  
        <Sources>  
          <Source Name="BigBuckBunny.mp4"/>  
        </Sources>  
        <VideoTracks>  
          <VideoTrack Id="0" FourCC="AVC1" Profile="Main" Level="3.0" Width="640" Height="360" DisplayAspectRatioNumerator="16" DisplayAspectRatioDenominator="9" Framerate="23.974" TargetFramerate="23.974" Bitrate="1271" TargetBitrate="1000" MaxGOPBitrate="1527"/>  
        </VideoTracks>  
        <AudioTracks>  
          <AudioTrack Id="0" Codec="AacLc" Channels="2" SamplingRate="44100" Bitrate="93" BitsPerSample="16"/>  
        </AudioTracks>  
      </AssetFile>  
      <AssetFile Name="BigBuckBunny_H264_650kbps_AAC_und_ch2_96kbps.mp4" Size="1057155" Duration="PT8.4288444S">  
        <Sources>  
          <Source Name="BigBuckBunny.mp4"/>  
        </Sources>  
        <VideoTracks>  
          <VideoTrack Id="0" FourCC="AVC1" Profile="Main" Level="3.0" Width="640" Height="360" DisplayAspectRatioNumerator="16" DisplayAspectRatioDenominator="9" Framerate="23.974" TargetFramerate="23.974" Bitrate="843" TargetBitrate="650" MaxGOPBitrate="1086"/>  
        </VideoTracks>  
        <AudioTracks>  
          <AudioTrack Id="0" Codec="AacLc" Channels="2" SamplingRate="44100" Bitrate="93" BitsPerSample="16"/>  
        </AudioTracks>  
      </AssetFile>  
      <AssetFile Name="BigBuckBunny_H264_400kbps_AAC_und_ch2_96kbps.mp4" Size="699262" Duration="PT8.4288444S">  
        <Sources>  
          <Source Name="BigBuckBunny.mp4"/>  
        </Sources>  
        <VideoTracks>  
          <VideoTrack Id="0" FourCC="AVC1" Profile="Main" Level="1.3" Width="320" Height="180" DisplayAspectRatioNumerator="16" DisplayAspectRatioDenominator="9" Framerate="23.974" TargetFramerate="23.974" Bitrate="503" TargetBitrate="400" MaxGOPBitrate="661"/>  
        </VideoTracks>  
        <AudioTracks>  
          <AudioTrack Id="0" Codec="AacLc" Channels="2" SamplingRate="44100" Bitrate="93" BitsPerSample="16"/>  
        </AudioTracks>  
      </AssetFile>  
      <AssetFile Name="BigBuckBunny_AAC_und_ch2_96kbps.mp4" Size="166780" Duration="PT8.4288444S">  
        <Sources>  
          <Source Name="BigBuckBunny.mp4"/>  
        </Sources>  
        <AudioTracks>  
          <AudioTrack Id="0" Codec="AacLc" Channels="2" SamplingRate="44100" Bitrate="93" BitsPerSample="16"/>  
        </AudioTracks>  
      </AssetFile>  
      <AssetFile Name="BigBuckBunny_AAC_und_ch2_56kbps.mp4" Size="124576" Duration="PT8.4288444S">  
        <Sources>  
          <Source Name="BigBuckBunny.mp4"/>  
        </Sources>  
        <AudioTracks>  
          <AudioTrack Id="0" Codec="AacLc" Channels="2" SamplingRate="44100" Bitrate="53" BitsPerSample="16"/>  
        </AudioTracks>  
      </AssetFile>  
    </AssetFiles>  

## <a name="next-steps"></a>後續步驟
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
