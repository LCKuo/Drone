# GIS Server 與模擬 Client 硬體規格書

> 文件日期：2026-08-31  
> 用途：客戶採購規格、供應商詢價及設備驗收  
> 基準數量：1 台 GIS Server＋1 台模擬 Client  
> 本文件不含任何價格，供應商應依完整規格逐項報價，不得以未註明型號的零件替代。

## 一、採購範圍

本案建議採購：

1. GIS Server 一台。
2. Server UPS 一台。
3. 獨立備份 NAS 一台。
4. 2.5／10GbE 網管交換器一台。
5. 機櫃、PDU 及相關網路配線一批。
6. 模擬 Client 一台。
7. Client 顯示器、鍵盤及滑鼠各一組。

本規格不包含 Unity 授權、方向盤、飛行搖桿、座艙、投影機、VR、動感平台、機房空調、固定 IP、網路線路與軟體維護服務。

## 二、系統負載與設計目標

目前 GIS Server 的軟體工作負載包含：

- PostgreSQL／PostGIS 空間資料庫。
- GIS API、管理後台及權限驗證。
- 地圖工作佇列與兩個並行 Worker。
- OSM 道路、建築與環境資料處理。
- DEM 高程與三維地形生成。
- 正射影像下載、拼接、快取與貼圖處理。
- 建築、道路、植被、地形及 GLB 地圖包輸出。
- 地圖資產儲存、版本管理、下載及備份。

目前軟體設定上限：

- 地圖產生 Worker：2 個。
- 每次工作最大圖磚數：1,600。
- GIS 最大選取半徑：10 km。
- NLSC 影像最大選取半徑：4 km。
- 單張貼圖最高 4,096 × 4,096。

硬體設計目標：

- 支援兩個地圖產生工作並行執行。
- 支援一至十台 Client 一般查詢、預覽與地圖下載。
- Server 記憶體可擴充至至少 256 GB。
- 儲存設備可增加企業 SSD。
- 具備 ECC、RAID、備援電源、UPS、遠端硬體管理及獨立備份。
- 單機正式部署，不含自動高可用叢集。

現行 GIS 程式沒有 Server 端 CUDA、AI 推論或 GPU 渲染工作，因此 GIS Server 不需要獨立 GPU。預算與機箱空間應優先用於 ECC 記憶體、企業 SSD、備援及備份。

## 三、GIS Server 規格

### 3.1 主機定位

採用 Dell PowerEdge R760xs、HPE ProLiant DL380 Gen11 或同級 2U 機架式企業伺服器。品牌可以替換，但所有替代品必須符合或高於本文件規格。

### 3.2 必要規格

| 項目 | 最低可接受規格 | 建議採購規格 |
|---|---|---|
| 機型 | 2U 機架式企業伺服器 | Dell PowerEdge R760xs、HPE DL380 Gen11 或同級 |
| CPU | 12 核心／24 執行緒 Server CPU | 1 × Intel Xeon Silver 4514Y 等級，16 核心／32 執行緒或以上 |
| CPU 數量 | 單顆 | 單顆；保留第二顆 CPU 或平台擴充能力不是必要條件 |
| 記憶體 | 64 GB ECC RDIMM | 128 GB DDR5 ECC RDIMM |
| 記憶體擴充 | 可擴充至 128 GB | 至少可擴充至 256 GB，並保留可用插槽 |
| 開機碟 | 兩顆企業 SSD | 2 × 480 GB 或 960 GB 企業 M.2 SSD，RAID 1 |
| 開機模組 | 支援 RAID 1 | BOSS 或同級獨立開機模組 |
| 資料碟 | 2 × 1.92 TB 企業 SSD | 2 × 3.84 TB 企業 SSD，RAID 1 |
| 資料可用容量 | 至少 1.92 TB | 約 3.84 TB |
| SSD 耐用度 | 企業級、有原廠保固 | 報價明列 DWPD、TBW、介面及料號 |
| RAID Controller | 支援 RAID 1 | 企業 RAID Controller，具快取及斷電保護 |
| 網路 | 2 × 1GbE | 2 × 10GbE＋獨立 BMC 管理網路 |
| 遠端管理 | 基本 IPMI | iDRAC Enterprise、iLO Advanced 或同級 |
| 電源 | 單電源 | 2 × 熱插拔白金級備援電源 |
| 風扇 | 一般風扇 | 熱插拔備援風扇 |
| 安全 | TPM 2.0 | TPM 2.0、Secure Boot、可設定 BIOS 密碼 |
| 滑軌 | 選配 | 原廠滑軌與理線臂 |
| 保固 | 一年 | 三年原廠次工作日到府，建議提供四小時服務選項 |
| 作業系統 | Linux | Ubuntu Server 24.04 LTS |
| Server GPU | 不需要 | 不配置 |

### 3.3 供應商報價要求

供應商報價必須明列：

- Server 品牌、完整型號與 Service Tag／序號取得方式。
- CPU 完整型號、核心數及數量。
- 記憶體單條容量、數量、頻率及空餘插槽數。
- 開機 SSD、資料 SSD 的品牌、型號、容量、介面、耐寫度及保固。
- RAID Controller 型號、快取容量及斷電保護方式。
- 網路卡型號、埠數、速度及介面。
- BMC 授權等級。
- 電源供應器瓦數、效率及數量。
- 滑軌、理線臂、電源線及必要配件。
- 原廠保固年限、服務時間與到場方式。

不得使用下列方式降規：

- 以桌上型 Core i9／Ryzen 主機取代企業 Server。
- 以非 ECC 記憶體取代 ECC RDIMM。
- 以單顆開機碟或單顆資料碟取代 RAID 1。
- 以未標明型號的一般消費 SSD 取代企業 SSD。
- 以軟體假 RAID 取代規格要求的企業 RAID Controller，除非事前獲得書面同意。
- 取消雙電源、BMC 遠端管理或原廠到府保固。

## 四、Server UPS

| 項目 | 規格 |
|---|---|
| 容量 | 1500 VA 或以上 |
| 輸出波形 | 純正弦波 |
| 類型 | 在線互動式或 Online UPS |
| 通訊 | USB 或網路管理，支援 Linux 自動安全關機 |
| 電池 | 可更換電池，具電池自我檢測 |
| 插座 | 足以供應 Server 雙 PSU、網路設備及必要管理設備 |
| 管理 | 可記錄停電、電壓、負載、電池狀態及告警 |
| 保固 | 主機至少三年；電池保固依原廠條件 |

UPS 必須安裝自動關機代理程式並完成實際斷電測試。UPS 的用途是讓系統安全關機，不是長時間停電供電。

## 五、獨立備份 NAS

RAID 不是備份。Server 內部 RAID 只能降低單顆磁碟故障造成的停機，不能處理誤刪、勒索軟體、資料庫邏輯損毀、主機失竊或機房事故。

### 5.1 NAS 規格

| 項目 | 規格 |
|---|---|
| 機型 | 四 Bay NAS 或以上 |
| CPU | 四核心或以上 |
| 記憶體 | 4 GB 以上，可擴充 |
| 網路 | 至少 2 × 2.5GbE；支援 Link Aggregation 或 SMB Multichannel |
| 硬碟 | 初期安裝 2 × 8 TB NAS／Enterprise HDD |
| RAID | RAID 1；可用容量約 8 TB |
| 擴充 | 保留至少兩個空碟槽 |
| 協定 | SMB、NFS、SFTP／rsync 或等效安全備份協定 |
| 快照 | 支援排程快照及版本保留 |
| 告警 | Email／Webhook／SNMP 或等效通知 |
| UPS | 具獨立 UPS，或接入可管理並有足夠容量的共用 UPS |
| 保固 | 至少三年 |

### 5.2 備份政策

- 每日增量備份。
- 每週完整備份。
- 保留至少三十天版本。
- PostgreSQL 邏輯備份與檔案層備份分開執行。
- 地圖包、模型、授權 manifest、設定及操作紀錄納入備份。
- 每月至少一份離線或異地備份。
- 每季執行一次完整還原測試並留下紀錄。

## 六、網路設備與配線

### 6.1 網管交換器

| 項目 | 規格 |
|---|---|
| 類型 | L2 Web 網管型交換器 |
| Client 埠 | 至少 6 × 2.5GbE RJ45 |
| Server／NAS 上行 | 至少 2 × 10GbE RJ45 或 SFP+ |
| 功能 | VLAN、LACP、RSTP、LLDP、流量統計、韌體更新 |
| 管理 | HTTPS 管理介面、管理帳號可更換、設定可備份 |
| 安裝 | 桌上型或機架式皆可，但需妥善固定及散熱 |

### 6.2 配線

- 固定銅纜使用 Cat6A 或以上。
- 機櫃內 10GbE 可使用相容的 SFP+ DAC。
- 每條線纜兩端需有一致標籤。
- Server、NAS、Switch 與 Client 的網路拓撲需隨案交付。
- GIS Server 管理網路與公開服務網路應以 VLAN 或實體網路隔離。

Server 建議使用 10GbE；一般模擬 Client 使用 2.5GbE。若客戶既有設備符合本節規格，可直接沿用。

## 七、機櫃與 PDU

| 項目 | 規格 |
|---|---|
| 標準 | 19 吋機櫃 |
| 高度 | 至少 12U；依現場既有設備增加空間 |
| 深度 | 必須容納約 720 mm 深的 2U Server、接頭、理線臂及散熱空間 |
| 承重 | 符合 Server、UPS、NAS 及網路設備總重量 |
| 通風 | 前進後出風流，前後門不得阻塞風流 |
| PDU | 機架式 PDU，插座數足夠，標示各設備電源 |
| 接地 | 完整接地 |
| 門鎖 | 可上鎖並管理鑰匙 |
| 線路 | 電源與網路線分開整理 |

壁掛式網路箱常因深度或承重不足而無法放入 2U Server。採購前必須取得機櫃內部有效深度與承重資料，不得只看 `12U` 標示。

## 八、模擬 Client 規格

Client 用於執行模擬 EXE、GIS 三維地圖、PBR 建築、衛星地面、PX4／Gazebo 與 Bridge。GPU 對畫面品質及幀率的影響大，但不需要 Xeon、ECC 或工作站顯示卡。

### 8.1 Client 主機

| 項目 | 最低可接受規格 | 建議採購規格 |
|---|---|---|
| CPU | 6 核心／12 執行緒 | AMD Ryzen 7 9700X／7800X3D，或 Intel Core Ultra 7／Core i7 同級以上 |
| 記憶體 | 32 GB | 32 GB DDR5；同機執行 Unity Editor、Gazebo 與開發工具時使用 64 GB |
| GPU | NVIDIA RTX 4060 Ti 8 GB | NVIDIA GeForce RTX 5070 12 GB 或以上 |
| SSD | 1 TB NVMe | 2 TB PCIe 4.0 NVMe SSD |
| 網路 | 1GbE | 2.5GbE RJ45 |
| 電源 | 650 W | 750～850 W、80 Plus Gold、原廠足瓦 |
| 散熱 | 可維持 CPU 額定效能 | 品質良好的塔式風冷或 240 mm 水冷 |
| 機殼 | 可容納 GPU 且有前後風流 | 前方進風、後方／上方排風，具可清潔濾網 |
| 作業系統 | Windows 11 64-bit | Windows 11 Pro 64-bit OEM |
| TPM | TPM 2.0 | TPM 2.0、Secure Boot |
| 保固 | 一年 | 原廠或整機三年保固優先 |

### 8.2 顯示器與基本周邊

| 項目 | 規格 |
|---|---|
| 顯示器 | 27 吋、QHD 2560 × 1440、IPS、至少 144 Hz、DisplayPort／HDMI |
| 顯示器人體工學 | 支援傾斜；建議支援高度調整 |
| 鍵盤 | 有線 USB、標準全尺寸或依操作需求 |
| 滑鼠 | 有線 USB、具滾輪及至少三鍵 |
| 視訊線 | DisplayPort 或符合 QHD 高更新率的 HDMI 線 |

方向盤、飛行搖桿、遙控器、踏板、VR、座艙與動感平台不在本規格內，應依實際訓練模式另案選型。

## 九、數量計算

### 基準建置

| 設備 | 數量 |
|---|---:|
| GIS Server | 1 |
| Server UPS | 1 |
| 備份 NAS | 1 |
| NAS UPS | 1，或使用已驗證容量足夠的共用 UPS |
| L2 網管交換器 | 1 |
| 深型機櫃＋PDU | 1 組 |
| 模擬 Client | 1 |
| QHD 顯示器 | 1 |
| 鍵盤滑鼠 | 1 組 |

### 增加 Client

每增加一個模擬操作席，增加：

- 模擬 Client 主機一台。
- QHD 顯示器一台。
- 鍵盤滑鼠一組。
- 2.5GbE 網路埠一個。
- Cat6A 網路線一條。
- 如需專用操控器，再依設備介面另購。

Client 超過交換器可用埠數時，必須同時擴充交換器，不得以家用無網管 Switch 隨意串接正式網路。

## 十、擴充選項

| 擴充項目 | 建議規格 | 適用情境 |
|---|---|---|
| Server RAM | 升級至 256 GB ECC RDIMM | 四個以上 Worker、大量 PostGIS 查詢或同機執行更多服務 |
| Server 資料碟 | 4 × 3.84 TB 企業 SSD RAID 10 | 大量區域、長期保留多版本地圖或高下載量 |
| 更高保固 | 四小時到場／Mission Critical | 客戶要求縮短硬體修復時間 |
| Client RAM | 64 GB DDR5 | Unity Editor、Gazebo、IDE、建置與 EXE 同時執行 |
| Client GPU | RTX 5070 Ti／5080 或以上 | 4K、多螢幕、VR、更高建築密度及更高陰影品質 |
| 第二台 GIS Server | 同主機規格 | 高可用、工作量增加或維修不中斷需求 |

## 十一、高可用需求

本規格的單台 GIS Server 具備 RAID、雙 PSU、UPS 及備份，但仍是單點故障。若合約承諾 24×7、99.9% 以上可用性或維修不停機，至少需要：

- 第二台同級 GIS Server。
- PostgreSQL 主從複寫及切換程序。
- 雙反向代理或客戶既有負載平衡器。
- 地圖資產共享或複寫。
- 雙交換器。
- 雙 UPS 與雙電力迴路。
- 異地備份及定期災難復原演練。
- 完整 HA 安裝、監控、切換及回復測試。

只增加第二台 Server，不代表系統已經具備自動高可用；資料庫、資產、佇列、網路及應用程式都必須完成整體測試後，才能在合約中承諾 HA。

## 十二、Server 驗收條件

1. CPU、ECC 記憶體、SSD、RAID、網卡及保固內容與報價一致。
2. 開機碟 RAID 1 與資料碟 RAID 1 狀態正常。
3. 模擬拔除任一資料碟後，系統保持運作並產生告警。
4. 完成更換測試碟與 RAID Rebuild 驗證。
5. 拔除任一 PSU 後，主機保持運作並產生告警。
6. BMC 遠端登入、遠端主控台、開關機與硬體告警正常。
7. BIOS、BMC、RAID、SSD 及網卡韌體更新至穩定版本。
8. Ubuntu Server、Docker／服務、PostgreSQL、GIS API 與兩個 Worker 正常。
9. 完成一張 4 km NLSC 模式與一張 GIS High 地圖生成測試。
10. 產生工作期間 CPU、記憶體、SSD 溫度、空間與 I/O 無異常。
11. UPS 斷電事件可觸發安全關機。
12. NAS 備份成功，並完成至少一次 PostgreSQL 及地圖資產還原。
13. 10GbE 連線、VLAN、管理網路與對外服務網路設定符合網路拓撲。

## 十三、Client 驗收條件

1. CPU、RAM、GPU、SSD、Windows 版本與報價一致。
2. Windows、GPU、晶片組及網卡驅動無裝置錯誤。
3. TPM 2.0、Secure Boot 與 Windows 啟用狀態正常。
4. 以 QHD 原生解析度及指定畫質執行模擬 EXE。
5. 可連線 GIS Server、選擇範圍、下載並載入地圖包。
6. GIS High、Satellite 與 Minecraft 模式依系統功能正常切換。
7. PX4、Gazebo、Bridge 與模擬 EXE 連線正常。
8. Web 與 EXE 的資料來源 attribution 正常顯示。
9. 連續運作兩小時無當機、顯存不足、明顯降頻或破圖。
10. 關機、重新開機及網路中斷恢復後，系統可正常重新連線。

## 十四、供應商交付文件

供應商應一併交付：

- 完整設備清冊及所有序號。
- Server、UPS、NAS、Switch、Client 的正式規格表。
- 原廠保固證明及報修方式。
- RAID 配置紀錄。
- BIOS／BMC／韌體版本紀錄。
- 網路拓撲、IP、VLAN 與埠位表。
- UPS 負載及自動關機設定。
- NAS RAID、帳號、排程、快照及保留政策。
- 作業系統與驅動程式版本。
- 驗收測試結果。

所有管理密碼、復原金鑰及設定備份，應透過安全方式移交指定管理人員，不得寫在公開手冊、設備外殼或一般 Email 中。

## 十五、供應商詢價摘要

可直接將以下文字交給硬體供應商：

> 請提供一台 2U 企業級 GIS Server，配置單顆 16 核心以上 Xeon Silver 等級 CPU、128 GB DDR5 ECC RDIMM、雙企業 M.2 RAID 1 開機碟、雙 3.84 TB 企業 SSD RAID 1 資料碟、企業 RAID Controller、雙 10GbE、獨立 BMC 遠端管理、雙熱插拔電源、原廠滑軌與三年原廠到府保固；另提供 1500 VA 可管理 Smart UPS、一台四 Bay 備份 NAS（初期 2 × 8 TB RAID 1）、2.5／10GbE L2 網管交換器、Cat6A／DAC 配線及可容納深型 2U Server 的 19 吋機櫃與 PDU。模擬 Client 採 Ryzen 7／Core Ultra 7 等級 CPU、32 GB DDR5、RTX 5070 12 GB、2 TB NVMe、2.5GbE、Windows 11 Pro、27 吋 QHD IPS 144 Hz 顯示器及有線鍵盤滑鼠。請逐項明列品牌、型號、料號、數量、保固與所有必要配件，不接受未註明型號或低於規格的替代品。
