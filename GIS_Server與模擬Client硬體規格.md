# GIS Server 與模擬 Client 系統執行規格

> 文件用途：說明客戶需要準備的硬體與作業環境，確保 GIS Server 與模擬 Client 可以正常執行。  
> 本文件僅為系統需求，不包含硬體品牌限制、採購條件、保固年限、到場服務、SLA 或供應商承諾。

## 一、適用範圍

本系統分成：

1. **GIS Server**：執行空間資料庫、地圖 API、管理後台、圖資下載、地形／建築處理及 3D 地圖生成。
2. **模擬 Client**：執行模擬程式、下載 GIS 地圖、顯示 PBR 建築、衛星地面、真實地形及飛行／駕駛模擬畫面。

以下為單台 GIS Server 與一般模擬 Client 的執行規格。實際容量會依地圖範圍、資料量、同時工作數與 Client 數量調整。

## 二、GIS Server 規格

### 最低執行規格

適用於測試、展示、小範圍地圖及低頻率生成工作。

| 項目 | 最低規格 |
|---|---|
| CPU | 64 位元 x86 CPU，12 核心／24 執行緒以上 |
| 記憶體 | 64 GB RAM |
| 系統碟 | 500 GB SSD |
| GIS 資料碟 | 2 TB SSD 可用空間 |
| 網路 | 1GbE |
| GPU | 不需要 |
| 作業系統 | Ubuntu Server 22.04 LTS 或 24.04 LTS，64 位元 |

### 建議正式規格

適用於正式部署、衛星影像、真實地形、較大範圍地圖及多台 Client 使用。

| 項目 | 建議規格 |
|---|---|
| CPU | Server 級 16 核心／32 執行緒以上 |
| 記憶體 | 128 GB ECC RAM |
| 系統碟 | 2 顆 500 GB 以上 SSD，RAID 1 |
| GIS 資料碟 | 2 顆 4 TB 以上企業 SSD，RAID 1 |
| 資料碟可用空間 | 4 TB 以上 |
| 網路 | 2.5GbE 以上；多 Client 或大量地圖下載建議 10GbE |
| GPU | 不需要 |
| 作業系統 | Ubuntu Server 24.04 LTS，64 位元 |

GIS Server 的地圖生成工作主要使用 CPU、記憶體及 SSD I/O。目前系統沒有 Server 端 AI 推論或 GPU 渲染需求，因此不需要 NVIDIA GPU。

### 未來擴充型規格

如果 Server 預計使用四至五年以上，而且未來可能加入全台圖資、更多並行地圖工作、LOD2／LOD3 建築、實景網格、歷史圖層、多租戶或大量 Client，建議直接選擇下列底座，避免日後整台更換：

| 項目 | 未來擴充型規格 |
|---|---|
| 機型 | 2U 標準企業 Server，雙 CPU 插槽、至少 16 個記憶體插槽、至少 8 個熱插拔 SSD 槽、完整 PCIe Gen5 擴充能力 |
| 參考等級 | Dell PowerEdge R760（非精簡型 R760xs）、HPE ProLiant DL380 Gen11 或同級 |
| CPU | 初期 1 顆 24～32 核心 Server CPU；保留增加 CPU 或擴充 Worker 節點的能力 |
| 記憶體 | 初期 256 GB DDR5 ECC RDIMM，可擴充至至少 512 GB |
| 系統碟 | 2 × 960 GB 企業 SSD，RAID 1 |
| GIS 資料碟 | 4 × 3.84 TB 企業 SSD，RAID 10 |
| GIS 可用空間 | 約 7.68 TB，並至少保留四個可用硬碟槽 |
| 網路 | 2 × 25GbE SFP28 或同級高速網路，另有獨立管理網路 |
| PCIe | 至少兩個可用 PCIe Gen5 x16 插槽 |
| GPU | 初期不安裝；AI／航測功能採獨立 GPU Worker，或在採購時明確選擇原廠 GPU-ready 機型 |

如果已確定未來會在同一台 Server 安裝資料中心 GPU，採購時必須一起選定 GPU 相容的 Riser、供電、風扇、散熱導流罩及機箱配置。這些零件不一定能在日後低成本補裝。標準 Dell PowerEdge R760 可依原廠指定配置支援最多兩張雙寬 GPU；GPU 應使用原廠驗證的資料中心型號，不使用一般消費級顯示卡。[Dell PowerEdge R760 GPU 規格](https://www.dell.com/support/manuals/en-us/poweredge-r760/per760_ism_pub/gpu-kit?guid=guid-e17fce54-1364-42c5-8ea6-0825237f1041)

沒有明確 AI／航測需求時，不需要現在購買 GPU。保留高速網路與獨立 GPU Worker 的擴充方式，通常比把資料庫、Web、地圖生成與 AI 全放在同一台 Server 更容易維護。

## 三、模擬 Client 規格

### 最低執行規格

適用於 1080p、一般畫質及中小型 GIS 地圖。

| 項目 | 最低規格 |
|---|---|
| CPU | Intel Core i5／AMD Ryzen 5 等級，6 核心／12 執行緒以上 |
| 記憶體 | 32 GB RAM |
| GPU | NVIDIA GeForce RTX 4060 8 GB 或同級以上 |
| 儲存 | 1 TB NVMe SSD |
| 可用空間 | 安裝後至少保留 200 GB |
| 網路 | 1GbE |
| 作業系統 | Windows 11 64 位元 |
| 圖形 API | DirectX 12 |

### 建議正式規格

適用於 QHD、GIS High、衛星地面、PBR 建築、真實地形及較大型地圖。

| 項目 | 建議規格 |
|---|---|
| CPU | Intel Core i7／Core Ultra 7／AMD Ryzen 7 等級，8 核心以上 |
| 記憶體 | 32 GB RAM；同機執行 Unity Editor、Gazebo 或開發工具時建議 64 GB |
| GPU | NVIDIA GeForce RTX 5070 12 GB 或同級以上 |
| 儲存 | 2 TB NVMe SSD |
| 可用空間 | 安裝後至少保留 500 GB |
| 網路 | 2.5GbE |
| 作業系統 | Windows 11 Pro 64 位元 |
| 圖形 API | DirectX 12 |

若使用 4K、多螢幕、VR 或更高建築密度，建議使用 16 GB 以上 VRAM 的顯示卡。

## 四、Server 軟體環境

GIS Server 需要：

- Ubuntu Server 22.04 LTS 或 24.04 LTS。
- Docker Engine 與 Docker Compose Plugin，或依部署文件使用原生服務。
- PostgreSQL／PostGIS。
- 足夠的本機檔案儲存空間。
- 可正常解析 DNS 及使用 HTTPS。
- 若需要即時取得 OSM、影像、高程或外部模型，Server 必須能連線到對應資料來源。
- 對 Client 開放 GIS 系統使用的 HTTPS 連接埠。

正式環境應使用固定內網 IP 或正式網域，避免 Client 因 Server 位址變動而無法下載地圖。

## 五、Client 軟體環境

模擬 Client 需要：

- Windows 11 64 位元。
- 最新穩定版 NVIDIA 顯示卡驅動。
- DirectX 12。
- 可連線至 GIS Server 的區域網路或 HTTPS 網路。
- 如需執行 PX4／Gazebo 模擬，需啟用 WSL2 及安裝專案指定的 Ubuntu 環境。
- 系統時間需正常同步，避免 HTTPS、登入權杖或 API 驗證失敗。

## 六、網路需求

| 使用情境 | 建議網路 |
|---|---|
| 單一 Client、一般地圖下載 | 1GbE 可使用 |
| 多台 Client、衛星地圖或大型 GLB | 2.5GbE 以上 |
| 多 Client 同時下載或集中式圖資環境 | Server 端建議 10GbE |
| 經 Internet 連線 GIS Server | 穩定的 HTTPS 連線，實際速度依地圖包大小及 Client 數量決定 |

Client 必須可以連線到 GIS API、登入端點及地圖下載端點。若客戶有防火牆、Proxy、VLAN 或資安閘道，需允許系統使用的網域與連接埠。

## 七、儲存需求

GIS Server 的儲存使用量會隨下列項目增加：

- 地圖區域大小。
- 衛星／正射影像解析度。
- 地形細節。
- 建築數量及模型品質。
- 同一區域保留的版本數量。
- 快取、工作暫存檔及備份保留時間。

建議正式環境至少保留 4 TB GIS 可用空間。若需要長期保存大量城市、全台圖資或多版本地圖，應擴充至 8 TB 以上可用空間。

Client 至少保留 200 GB 可用空間；需要保存多張大型地圖時，建議保留 500 GB 以上。

## 八、未來功能與硬體影響

未來新增的功能不一定只靠把原 Server 買更大解決。資料庫、一般地圖生成、GPU 運算、下載服務與備份可以分成不同節點，系統較容易逐步擴充。

| 未來可能新增的功能 | 主要硬體影響 | 建議擴充方式 |
|---|---|---|
| 全台灣或跨國圖資 | 儲存容量、資料匯入時間、PostGIS 索引及備份量大幅增加 | GIS 儲存擴充至 20 TB 以上；大型資料使用獨立儲存或物件儲存 |
| 更多同時地圖生成工作 | CPU 核心、RAM、SSD IOPS 增加 | 先增加 Worker 節點，不讓 API Server 無限增加 Worker |
| LOD2／LOD3 精細建築 | 模型容量、記憶體、生成時間及 Client VRAM 增加 | Server RAM 提升至 256～512 GB，使用 RAID 10／NVMe，Client 使用 16 GB 以上 VRAM |
| 實景網格／3D Tiles | 大量小檔案、切片、LOD、儲存與下載流量增加 | 獨立 Tile／Object Storage、10／25GbE、CDN 或邊緣快取 |
| 歷史影像與時間軸地圖 | 每一時期都會增加影像、索引與地圖版本 | 擴充儲存、建立生命週期與封存機制 |
| 航測照片轉 3D、攝影測量 | 高 CPU、RAM、NVMe 暫存及 GPU VRAM | 增加獨立 GPU Worker；建議 24～48 GB 以上 VRAM，依實際工具決定 |
| AI 建築辨識、道路分類、語意分割 | GPU、VRAM、模型儲存 | 獨立 AI Worker，不與主要資料庫共用全部資源 |
| NeRF／Gaussian Splatting | 高階 GPU、VRAM、NVMe 空間 | 專用 GPU Server 或工作站，不放入一般 GIS 主機 |
| 自動生成 PBR 材質或模型 | GPU 推論、資產儲存與審核流程 | 獨立 AI Worker＋資產庫 |
| 路徑規劃、地理編碼、搜尋 | RAM、CPU、索引容量 | 獨立 Routing／Geocoding Service；大型索引使用更多 RAM |
| 即時車輛／無人機遙測 | 持續寫入、時序資料庫、訊息佇列 | 獨立 Telemetry／Time-series 節點，避免影響地圖生成 |
| 多使用者協作編輯 | 資料庫連線、鎖定、稽核紀錄及 WebSocket | 分離 Web/API 與 Worker，增加資料庫資源 |
| 多客戶／多租戶 SaaS | 資料隔離、權限、稽核、計量及備份 | 獨立正式資料庫、API 節點、Worker Pool 與物件儲存 |
| 大量 Client 同時下載 | 網路頻寬與儲存讀取 | 10／25GbE、下載節點、快取或 CDN，不讓生成 Worker 同時負責所有下載 |
| 高可用與災難復原 | 需要避免 Server、資料庫及儲存單點故障 | 第二台 API／Worker、資料庫複寫、獨立備份；不是只升級單台 CPU |
| 批次模擬／大量情境生成 | CPU、GPU、佇列及暫存空間 | 建立可水平擴充的 Worker Pool |

### 建議擴充順序

1. **現有產品**：單台 GIS Server，PostGIS、API 與兩個 Worker 同機執行。
2. **工作量增加**：增加獨立 GIS Worker，主 Server 保留 API、帳號及資料庫。
3. **圖資大量化**：將地圖資產移至獨立儲存／物件儲存，Server 與儲存間使用 10／25GbE。
4. **AI／航測**：加入獨立 GPU Worker，不影響主要 GIS API 與資料庫。
5. **多租戶／高可用**：分離資料庫、API、Worker、下載與備份節點，依需要增加複寫及負載平衡。

### 需要擴充的觀察訊號

- CPU 長時間高負載，工作佇列持續累積。
- RAM 經常使用超過約七成，或開始大量使用 Swap。
- GIS 資料碟使用超過約七成。
- 地圖生成暫存檔造成 SSD I/O 長時間飽和。
- 多台 Client 同時下載時影響地圖生成或管理後台。
- 新功能需要 CUDA、AI 模型、航測重建或 24 GB 以上 VRAM。
- 客戶開始要求多租戶、不中斷服務或異地災難復原。

## 九、不需要的設備

一般部署不需要：

- GIS Server 專用顯示卡。
- AI GPU 或 NVIDIA Tesla／H 系列加速卡。
- Windows Server。
- Microsoft SQL Server。
- VMware 或其他商業虛擬化平台。

如果客戶既有環境要求使用特定虛擬化、儲存、備份或資安平台，應另外確認相容性與資源配置。

## 十、規格摘要

### GIS Server

```text
最低：12 核心 CPU／64 GB RAM／500 GB 系統 SSD／2 TB GIS SSD／1GbE／Ubuntu Server

建議：16 核心 Server CPU／128 GB ECC RAM／RAID 1 系統 SSD／
      4 TB 以上 RAID 1 GIS 可用空間／2.5GbE～10GbE／Ubuntu Server 24.04 LTS

未來擴充型：24～32 核心 Server CPU／256 GB ECC RAM／
            4 × 3.84 TB 企業 SSD RAID 10／25GbE／可擴充的 2U Server 底座

Server GPU：目前不需要；AI、航測或生成式功能以獨立 GPU Worker 擴充
```

### 模擬 Client

```text
最低：6 核心 CPU／32 GB RAM／RTX 4060 8 GB／1 TB NVMe／1GbE／Windows 11

建議：8 核心 CPU／32～64 GB RAM／RTX 5070 12 GB／
      2 TB NVMe／2.5GbE／Windows 11 Pro
```
