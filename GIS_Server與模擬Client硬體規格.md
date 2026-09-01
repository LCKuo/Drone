# GIS Server 與模擬 Client 硬體規格

> 本文件依同時使用數量與 GIS 集中服務需求，提供 Server 與 Client 的硬體選型建議。
>
> U（Concurrent Client）代表**同時上線並使用系統的 Client 數量**，不等同於帳號總數。
>
> 適用版本：GIS Server 1.8.1、Unity Map Client 1.8.1。
>
> 本文件的 U 級距是採購選型基準。若多台 Client 需要同時產生全新地圖，仍須依框選範圍、影像品質、地圖複雜度及期望完成時間評估；不得把「同時上線」直接解讀成「全部同時產圖」。

## 方案選擇總覽

| 功能需求 | A. Client 單機方案 | B. GIS 集中服務方案 |
|---|---:|---:|
| 執行已部署或事先下載的地圖 | 支援 | 支援 |
| 使用本機 Arnis 開源地理資料擷取／地圖產生功能 | 支援 | 支援 |
| 建立 Satellite Real Terrain（真實地形、OSM 白灰建築及授權影像） | 不包含 | 支援；須配置合法且允許下載的影像來源 |
| 使用 Server 自架國家底圖框選範圍 | 不包含 | 支援；World Selector 提供 Zoom 5–16 |
| 產生及下載 Unity 地圖包 | 不包含 | 支援 |
| 使用已存入 Client 的衛星地圖包 | 支援 | 支援 |
| Web GIS 後台、工作佇列及地圖集中管理 | 不包含 | 支援 |
| 多台 Client 的地圖集中更新與共用 | 需另行配置檔案發佈方式 | 由 GIS Server 提供 |

### 容量與選型原則

- **Client 單機方案：**每台 Client 獨立執行，Client 硬體依目標畫質、解析度與場景複雜度選擇。
- **GIS 集中服務方案：**1–10U 以最低 Server 規格作為採購基準；11–30U 以建議 Server 規格作為採購基準。
- **超過 30U：**依實際 Client 數量、同時產圖比例、下載流量與資料保存量進行容量評估後提供方案。
- 一般操作、World Selector 瀏覽、下載既有地圖與產生新地圖的負載不同；大量同時產圖需求須另行評估。

---

## A. Client 單機方案（不含 GIS Server）

### A-1. 包含的功能

- 執行已部署在 Client 的模擬程式與既有地圖。
- 開啟已儲存的 Unity 地圖。
- 使用本機 **Arnis Runtime Fork** 擷取開放地理資料並建立體素化地圖；需完成本機工具及資料來源設定。
- 使用已下載並存入 Client 的衛星地圖包。
- 每台 Client 獨立執行即時 3D 渲染。

### A-2. 未包含的功能

- GIS Server 線上框選、Satellite Real Terrain 產生及新地圖包下載。
- 地圖產生工作的集中排程。
- Web GIS 後台、地圖、Client 與工作狀態的集中管理。
- 未配置可下載影像來源時的新衛星真實地形產生。
- 地圖與軟體的集中發佈；如有需求，可另行配置 NAS 或檔案發佈設備。

### A-3. Arnis 授權說明

| 項目 | 說明 |
|---|---|
| 整合工具 | Arnis Runtime Fork；上游專案為 Arnis |
| 軟體授權 | Apache License 2.0 |
| 授權費用 | 無需另購商業授權 |
| 商業使用 | 授權允許使用、修改及散布 |
| 交付要求 | 安裝包須保留 Apache-2.0 License、原作者版權聲明、NOTICE（如有），並標示本系統的修改內容 |
| 地理資料 | OpenStreetMap 資料須保留署名並遵守 ODbL；高程、影像與模型資料依各來源授權辦理 |

授權依據：[Arnis GitHub／License](https://github.com/louis-e/arnis/blob/main/LICENSE)。

### A-4. Client 最低規格

適用於 1080p、標準畫質、既有地圖及一般模擬場景。

| 項目 | 最低規格 |
|---|---|
| CPU | Intel Core i5／AMD Ryzen 5，至少 6 核心 12 執行緒 |
| 記憶體 | 32 GB |
| 顯示卡 | NVIDIA GeForce RTX 4060，8 GB VRAM |
| 系統碟 | 1 TB NVMe SSD |
| 可用空間 | 安裝完成後至少保留 200 GB |
| 網路 | 1 GbE |
| 作業系統 | Windows 11 64-bit |
| 建議輸出 | 1920 × 1080 |

### A-5. Client 建議規格

適用於 QHD、高畫質衛星地圖、較大城市、複雜光影、Unity 編輯或較長使用週期。

| 項目 | 建議規格 |
|---|---|
| CPU | Intel Core i7／Core Ultra 7／AMD Ryzen 7，至少 8 核心 |
| 記憶體 | 32 GB；同機執行 Unity Editor、Gazebo 或多項工具時採 64 GB |
| 顯示卡 | NVIDIA GeForce RTX 5070，12 GB VRAM |
| 系統碟 | 2 TB NVMe SSD |
| 可用空間 | 安裝完成後至少保留 500 GB |
| 網路 | 2.5 GbE |
| 作業系統 | Windows 11 Pro 64-bit |
| 建議輸出 | 2560 × 1440（QHD）IPS 螢幕，144 Hz |

### A-6. 依 U 數選擇 Client

| 同時使用數 | Server | Client 選擇原則 |
|---:|---|---|
| 1–10U | 不需配置 GIS Server | 1080p／既有地圖可採最低規格；高畫質場景採建議規格 |
| 11–30U | 不需配置 GIS Server | 單機需求不因 U 數增加；依共同的目標畫質統一最低或建議規格 |
| 超過 30U | 依實際數量提供方案 | 依軟體部署、地圖發佈、更新與管理需求進行規劃 |

此方案以各 Client 執行事先部署或本機產生的內容為主，不包含 GIS Server 提供的線上產圖及集中管理功能。

---

## B. GIS 集中服務方案（含 GIS Server）

### B-1. 依 U 數選擇 Server 規格

| 同時使用數 | GIS Server 規格 | 適用方式 |
|---:|---|---|
| 1–10U | 1 台最低規格 Server | 一般瀏覽、下載既有地圖及低頻率產圖的小型環境 |
| 11–30U | 1 台建議規格 Server | 多 Client 集中瀏覽、下載、管理，產圖工作由佇列安排 |
| 超過 30U | 依實際數量提供方案 | 依同時產圖比例、下載流量、地圖大小與保存需求進行容量規劃 |

U 數代表同時上線數量；若使用情境包含大量 Client 同時產生未快取地圖，應依實際工作負載調整 Server 數量或工作節點。超過 30U 的環境，於確認使用方式與資料規模後提供配置方案。

### B-2. 最低 GIS Server 規格

適用於 1–10U、小型正式環境及低頻率產圖需求。

| 項目 | 最低規格 |
|---|---|
| 機型 | 可長時間運作的機架式 Server |
| CPU | 12 核心／24 執行緒以上 |
| 記憶體 | 64 GB ECC |
| 系統碟 | 2 × 500 GB 企業級 SSD，RAID 1 |
| GIS 資料碟 | 2 × 2 TB 企業級 SSD，RAID 1 |
| 網路 | 1 GbE |
| GPU | 不需要 |
| 作業系統 | Ubuntu Server 24.04 LTS |
| 電源與保護 | 冗餘電源、UPS |
| 備份 | 獨立 NAS 或備份設備；RAID 用於提升可用性，不取代資料備份 |

最低規格的 GIS 資料碟用於國家底圖、OSM、DEM、授權影像、使用者產生的 GLB 地圖包及快取；不是只存放 World Selector 底圖。

### B-3. 建議 GIS Server 規格

適用於 11–30U、一般正式部署，以及地圖資料量或產圖頻率較高的環境。

| 項目 | 建議規格 |
|---|---|
| 機型 | 2U 機架式 Server |
| CPU | 24–32 實體核心 |
| 記憶體 | 256 GB DDR5 ECC |
| 系統碟 | 2 × 960 GB 企業級 SSD，RAID 1 |
| GIS 資料碟 | 4 × 3.84 TB 企業級 SSD，RAID 10；約 7.68 TB 可用 |
| 網路 | 最低 10 GbE；可依網路環境選配雙 25 GbE |
| GPU | 現有 GIS 服務不需要 |
| 作業系統 | Ubuntu Server 24.04 LTS |
| 電源與保護 | 冗餘電源、UPS |
| 備份 | 獨立 NAS 或備份主機，並建議保留異機或異地副本 |

截至 2026-09-01，GIS Server 1.8.1 的台灣 World Selector 現有核心資料約 443 MiB。建議規格的儲存容量主要預留給其他國家、授權影像、DEM、使用者地圖包、版本保留與備份，不代表單一台灣底圖需要 TB 級空間。

### B-4. 地圖產生與工作佇列

一般登入、World Selector 瀏覽、下載既有地圖與產生新地圖屬於不同負載。新地圖產生會使用 CPU、記憶體、磁碟及資料來源，並由工作佇列安排。

World Selector 的 Zoom 5–13 由 Server 預建國家底圖提供；Zoom 14–16 由 CPU 按需渲染並保存在 Server LRU 快取。Unity 會以批次方式取得目前可視範圍與拖曳緩衝區，並在 Client 保存本機快取。這條流程不需要 Server GPU。

| 使用情境 | 容量評估方式 |
|---|---|
| 一般登入、瀏覽已快取底圖 | 主要使用網路與磁碟快取，依同時使用 U 數選擇 Server 規格 |
| 首次瀏覽未快取的 Zoom 14–16 區域 | 增加 CPU 與磁碟負載；完成後由 Server 快取重用 |
| 下載既有 Unity 地圖包 | 主要使用網路與儲存讀取效能 |
| 少量新地圖產生 | 使用 CPU、記憶體、磁碟及資料來源，由工作佇列安排 |
| 大量或批次同時產圖 | 依框選面積、LOD、地圖複雜度、資料來源及期望完成時間另行評估 |

實際產圖時間將依地圖範圍、品質設定、資料來源速度與儲存效能而異。

### B-5. GIS 方案的 Client 最低規格

適用於 1080p 標準畫質、既有地圖及一般模擬場景。GIS Server 負責資料與產圖服務，Client 仍負責即時 3D 渲染。

| 項目 | 最低規格 |
|---|---|
| CPU | Intel Core i5／AMD Ryzen 5，至少 6 核心 12 執行緒 |
| 記憶體 | 32 GB |
| 顯示卡 | NVIDIA GeForce RTX 4060，8 GB VRAM |
| 系統碟 | 1 TB NVMe SSD |
| 可用空間 | 安裝完成後至少保留 200 GB |
| 網路 | 1 GbE |
| 作業系統 | Windows 11 64-bit |
| 建議輸出 | 1920 × 1080 |

### B-6. GIS 方案的 Client 建議規格

適用於 QHD、較大城市、複雜光影、較長使用週期，或需要統一採購較高效能等級的 Client。

| 項目 | 建議規格 |
|---|---|
| CPU | Intel Core i7／Core Ultra 7／AMD Ryzen 7，至少 8 核心 |
| 記憶體 | 32 GB；同機執行 Unity Editor、Gazebo 或多項工具時採 64 GB |
| 顯示卡 | NVIDIA GeForce RTX 5070，12 GB VRAM |
| 系統碟 | 2 TB NVMe SSD |
| 可用空間 | 安裝完成後至少保留 500 GB |
| 網路 | 2.5 GbE |
| 作業系統 | Windows 11 Pro 64-bit |
| 建議輸出 | 2560 × 1440（QHD）IPS 螢幕，144 Hz |

U 數用於評估 GIS Server 容量；每台 Client 的規格仍依目標畫質、解析度與場景複雜度選擇。

---

## C. 方案選擇摘要

| 使用需求 | 建議方案 | Server | Client |
|---|---|---|---|
| 執行既有地圖，不需線上新增 GIS 地圖 | A. Client 單機方案 | 不需配置 GIS Server | 依畫質選擇最低或建議規格 |
| 使用本機 Arnis Runtime Fork 擷取／產生地圖 | A. Client 單機方案 | 不需配置 GIS Server | 依畫質選擇最低或建議規格 |
| 從 Server 自架國家底圖框選區域並建立 Satellite Real Terrain | B. GIS 集中服務方案 | 需配置 GIS Server；Satellite 須有合法影像來源 | 依畫質選擇最低或建議規格 |
| GIS 功能，1–10U | B. GIS 集中服務方案 | 1 台最低規格 Server | 最低或建議 Client |
| GIS 功能，11–30U | B. GIS 集中服務方案 | 1 台建議規格 Server | 依畫質選擇最低或建議 Client |
| GIS 功能，超過 30U | B. GIS 集中服務方案 | 依實際數量、使用方式與資料量提供方案 | 依目標畫質與部署需求提供方案 |

## 選型摘要

- 不需線上建立 GIS 地圖時，可採 Client 單機方案。
- 需要框選、產圖、下載及集中管理時，採 GIS 集中服務方案。
- GIS 集中服務方案：1–10U 以最低 Server 規格為採購基準；11–30U 以建議 Server 規格為採購基準；超過 30U 依實際數量與需求提供方案。
- U 數是同時上線數量，不代表全部 Client 同時產圖；大量同時產圖須依工作負載規劃工作節點。
- Client 規格依畫質與場景需求選擇，不因 Client 總數自動提高；GIS Server 容量則依同時使用數量、未快取區域瀏覽、下載流量及產圖負載規劃。
