# SkySim 與 USTP 硬體及部署授權規格

> 芯元數位科技股份有限公司（統編 62096000）｜更新日期：2026-09-07
>
> **SkySim**：無人機模擬訓練軟體。**USTP**：UAV Simulation Training Platform（無人機訓練管理平台）。GIS 為選購擴充。

## 先看本案需要哪些設備

**20 U、未選購 GIS：1 台 USTP Server（建議規格）＋20 台 SkySim PC（基本規格）。Meta Quest 頭盔按 VR 操作席次選配，不必每台 PC 都配置。**

| 設備 | 本案基本配置 | 何時需要 |
|---|---|---|
| USTP Server | 12–16 個實體核心、64 GB ECC、企業級 SSD、10 GbE；不需顯示卡 | 帳號驗證、課程指派、訓練成果、教官審核與報表 |
| SkySim PC | Core i5／Ryzen 5 六核心等級、32 GB RAM、RTX 4060 8 GB、1 TB NVMe SSD、1 GbE、1080p 螢幕 | 每個實際操作席次 1 台；不以高階顯示卡、64 GB RAM 或 QHD 螢幕為基本要求 |
| Meta Quest（選配） | Meta Quest 3S 或 Quest 3＋相容的 Link 資料線；配備方向與驗證條件見第 4 節 | 有 VR 訓練需求的席次；不是一般螢幕訓練的必需品 |
| USTP＋GIS Server（選配方案） | 採第 2 節 GIS 合併部署規格，取代僅 USTP 的配置 | 需要集中產生、發布及下載指定真實區域的地圖 |

**U 是可分配的學員授權席位，不是同時連線數。** 下列 Server 級距以「同時操作的 SkySim PC 台數」區分；教官及管理帳號不計學員席位，但其連線與報表工作仍須納入容量評估。

| 同時操作規模 | Server 選型 |
|---|---|
| 1–10 台 SkySim PC | 可採對應 Server 最低規格 |
| 11–30 台 SkySim PC | 採對應 Server 建議規格；20 U 全席同時使用採此級距 |
| 其他數量、跨據點或特殊負載 | 依實際數量與使用方式提供方案 |

以上為採購規劃級距，並非已完成全場景壓力測試的容量保證；不以所有人同時執行最大範圍 GIS 產圖為前提。

## 1. 僅 USTP Server：不選購 GIS

提供帳號與席位、班級、課程指派、成果與證據保存、教官審核、補強追蹤、報表及稽核。SkySim 使用已建置或已部署的考場，不需要購買 GIS 才能訓練。

| 項目 | 最低規格：1–10 台同時操作 | 建議規格：11–30 台同時操作 |
|---|---|---|
| 機型 | 可長時間運作的機架式 Server | 1U／2U 機架式 Server |
| CPU | 8 個實體核心／16 執行緒以上 | 12–16 個實體核心 |
| 記憶體 | 32 GB ECC | 64 GB ECC，可擴充至 128 GB |
| 系統碟 | 與資料共用 2 × 1.92 TB 企業級 SSD，RAID 1 | 2 × 960 GB 企業級 SSD，RAID 1 |
| 資料碟 | 上述 RAID 1，約 1.92 TB 可用 | 2 × 3.84 TB 企業級 SSD，RAID 1，約 3.84 TB 可用 |
| 網路 | 雙埠 1 GbE | 10 GbE |
| GPU | 不需要 | 不需要 |
| 作業系統 | Ubuntu Server 24.04 LTS | Ubuntu Server 24.04 LTS |
| 電源保護 | 冗餘電源及 UPS | 冗餘電源及 UPS |
| 備份 | 獨立 NAS 或備份設備，容量至少等於資料碟 | 獨立備份設備；異機／異地副本依保存政策配置 |

資料容量以標準成果及必要證據為基準；長期保存影片、高頻原始紀錄或大量回放時，依每日訓練人次及保存年限增加儲存容量。新版作業系統需經交付版本驗證後採用。

## 2. USTP＋GIS Server：選購真實地圖擴充

在 USTP 功能外，增加真實地圖框選、產生佇列、國家底圖、地形與影像管理，以及地圖包發布與集中下載。

| 項目 | 最低規格：1–10 台同時操作 | 建議規格：11–30 台同時操作 |
|---|---|---|
| 機型 | 1U／2U 機架式 Server | 2U 機架式 Server |
| CPU | 12 個實體核心／24 執行緒以上 | 24–32 個實體核心 |
| 記憶體 | 64 GB ECC | 256 GB DDR5 ECC |
| 系統碟 | 2 × 960 GB 企業級 SSD，RAID 1 | 2 × 960 GB 企業級 SSD，RAID 1 |
| GIS 與訓練資料碟 | 2 × 3.84 TB 企業級 SSD，RAID 1，約 3.84 TB 可用 | 4 × 3.84 TB 企業級 SSD，RAID 10，約 7.68 TB 可用 |
| 網路 | 雙埠 1 GbE；依既有機房環境選配 10 GbE | 雙埠 10 GbE |
| GPU | 不需要 | 不需要 |
| 作業系統 | Ubuntu Server 24.04 LTS | Ubuntu Server 24.04 LTS |
| 電源保護 | 冗餘電源及 UPS | 冗餘電源及 UPS |
| 備份 | 獨立備份設備，容量至少等於資料碟 | 獨立備份設備；異機／異地副本依保存政策配置 |

Server 不負責學員端的即時 3D 畫面。新 GIS 地圖透過工作佇列產生；多人同時產圖、大範圍資料匯入及大量冷圖磚產生，會增加等待時間，不能與既有地圖下載視為相同負載。

## 3. SkySim PC：一般螢幕訓練基本規格

**以能執行一般訓練為目的，採 1080p、標準畫質及已建置考場，不要求高階工作站。**

| 項目 | 基本採購規格 |
|---|---|
| CPU | Intel Core i5-12400／AMD Ryzen 5 5600 同級以上；至少 6 核心、12 執行緒，支援並啟用硬體虛擬化 |
| 記憶體 | 32 GB；供作業系統、模擬與物理計算共同使用，不以 64 GB 為基本要求 |
| 顯示卡 | NVIDIA GeForce RTX 4060 8 GB 或經相同考場測試的同級以上桌上型顯示卡 |
| 儲存 | 1 TB NVMe SSD；系統、軟體及考場安裝後至少保留 200 GB |
| 網路 | 1 GbE 有線網路 |
| 作業系統 | 正版 Windows 11 64-bit；版本依單位資訊管理需求選定 |
| 螢幕 | 1920 × 1080，24 吋左右即可 |
| 周邊 | 鍵盤、滑鼠、USB 連接埠及相容的模擬控制器 |
| 既有設備沿用 | 已有 RTX 3060 12 GB 等設備，可先以交付考場試機評估，通過後沿用 |

此配置為基本採購起點，尚非已在該組硬體完成的最低效能認證。大量樹木、複雜光影及大面積地圖可能需要降低畫質；採購前以正式交付版本及指定考場試機，不以顯示卡名稱承諾固定幀率。筆電同名 GPU 的功耗與效能不同，需另行驗證。

## 4. VR 頭盔：Meta Quest（選配）

**選配 Meta Quest 3S 或 Meta Quest 3，以 PC VR 方式連接 SkySim PC；不是將 Windows 版 SkySim 直接安裝在頭盔內獨立執行。**

**目前 SkySim 尚未完成 Meta Quest PC VR 的正式相容驗證。** 頭盔列為選配設備方向；VR 操作模式、頭部追蹤、介面與控制器整合及指定考場效能，須先完成導入驗證並確認交付範圍，再採購部署。一般螢幕版規格不等同 VR 畫質保證。

| 項目 | 配置方向 |
|---|---|
| 頭盔 | Meta Quest 3S（基本選項）或 Meta Quest 3（需要較高解析度及 Pancake 鏡片時選用） |
| PC | 以第 3 節為試機起點；依頭盔渲染解析度、更新率與指定考場實測確認，不直接套用頭盔官方最低門檻 |
| 連接方式 | 優先採有線 Meta Horizon Link（原 Meta Quest Link），使用相容的 USB 3 資料線與埠；線長至少 3 公尺，不能使用僅供充電的線材 |
| 無線方式 | Air Link 僅於完成場地無線網路驗證後採用；多席環境另行規劃，避免共用無線頻寬影響體驗 |
| 操作設備 | 原有模擬控制器供飛行操控；頭盔手持控制器是否用於選單操作，依 VR 模式整合結果確認 |
| 帳號與設備管理 | 依 Meta 當期啟用、組織管理及銷售地區條件配置；如需額外企業裝置管理服務，另行確認方案及費用 |

Meta 官方將 Quest 3／3S 列為可透過 Link 使用 PC VR 的裝置；此為頭盔能力，不代表 SkySim 已完成適配。[Meta 裝置比較](https://developers.meta.com/horizon/resources/compare-devices/)、[Meta Link PC 與線材需求](https://www.meta.com/help/quest/140991407990979/)、[Quest 3／3S 光學比較](https://www.meta.com/quest/compare/)。

## 5. Server 作業系統、資料庫及加購授權

**採本文件的 Ubuntu＋PostgreSQL 基本部署，不需要另外採購 Windows Server 或 Microsoft SQL Server。** 作業系統與資料庫的軟體使用授權費可為 0；不代表安裝導入、備份、維護、商業支援或其他選配服務均免費。

目前系統具備 SQLite 單機模式及 PostgreSQL 部署設定；正式多人部署採 PostgreSQL 規劃。以下按「基本使用不需加購」與「有指定需求才規劃」區分。

### 基本部署：不需另購商業使用授權

| 元件 | 用途 | 授權與費用 |
|---|---|---|
| Ubuntu Server | USTP／GIS 主機作業系統 | 基本系統可免費下載使用；Ubuntu Pro 與原廠支援為另選服務。[官方下載](https://ubuntu.com/download/server)、[支援方案](https://ubuntu.com/pricing/pro) |
| PostgreSQL | 帳號、課程、成績、授權及作業資料庫 | PostgreSQL License，商業使用不收軟體授權費，無需按學員購買資料庫 CAL。[官方授權](https://www.postgresql.org/about/licence/) |
| PostGIS（選購 GIS 時使用） | 地理資料庫擴充 | GPL-2.0-or-later，無需另購商業使用授權；散布時仍需履行授權與對應原始碼等義務。[官方授權](https://github.com/postgis/postgis/blob/master/LICENSE.TXT) |
| SQLite（單機模式／圖磚檔案） | 小型部署與本機資料檔 | 公眾領域，基本使用不需另購授權；不是正式多人資料庫容量的替代保證。[官方說明](https://www.sqlite.org/copyright.html) |
| Linux Docker Engine（採容器部署時） | 在 Linux Server 執行容器 | 開源 Engine 可用，不要求購買 Docker Desktop；容器內各元件仍須分別確認授權。[Engine 說明](https://docs.docker.com/engine/)、[授權區分](https://docs.docker.com/subscription-billing/desktop-license/) |
| 本機檔案儲存 | 訓練證據與地圖包保存 | 基本單機方案可使用 Server 檔案系統，不必為此購買獨立物件儲存產品 |

### 有指定需求時：依需求另行規劃

| 加購或指定項目 | 何時需要 | 費用／授權判斷 |
|---|---|---|
| Ubuntu Pro／原廠技術支援 | 需要擴充安全更新、特定合規功能或原廠支援 | 依主機與支援方案訂閱；不是基本系統運作必購項目。[官方方案](https://ubuntu.com/pricing/pro) |
| Windows Server 與存取授權 | 單位指定 Windows Server 主機、Windows 服務或虛擬化環境 | 不屬預設 Linux 部署；先確認相容架構，再按核心與適用的 CAL／External Connector 規則規劃。CAL 不能直接等同 20 U 學員席位，須納入實際存取人員或裝置；RDS 另依是否使用判斷。[Microsoft 授權指引](https://www.microsoft.com/licensing/guidance/Windows-Server-2025) |
| 商用資料庫／資料庫支援 | 指定 SQL Server、Oracle 或原廠代管、技術支援 | 本案預設不使用 SQL Server／Oracle，無需預購其授權；指定更換時需先確認系統相容性及另案費用 |
| 企業備份、虛擬化、EDR／防毒、WAF 或監控服務 | 機房政策、備援或資安規範指定 | 按既有授權可否沿用、設備／主機數及服務範圍詢價；不視為本案資安費用自動包含 |
| Docker Desktop | 單位另行指定在工作站使用 Desktop 管理工具 | 非 Linux Server 必要元件；政府機關及不符合免費使用條件的組織須付費訂閱，不可與開源 Engine 混為一談。[官方條款](https://docs.docker.com/subscription-billing/desktop-license/) |
| 多工作程序佇列／物件儲存 | GIS 多節點、共享儲存或較大併發需求 | 依交付架構確認。若採用 Redis 7.4（RSALv2／SSPLv1）或 MinIO（AGPLv3），由芯元依實際版本核對使用及交付義務；必要時採替代元件或另詢商業方案，不一概視為需付費或無條件免費。[Redis 授權](https://redis.io/legal/licenses/)、[MinIO 授權](https://github.com/minio/minio/blob/master/LICENSE) |
| GIS 圖資、影像與外部服務 | 需要指定國家、高程、衛星影像或其他外部資料 | 依區域、解析度、離線快取、衍生成果與交付範圍取得適用授權；與 GIS 軟體及本案軟體費用分開規劃 |

**SkySim PC 仍需各自具有合法 Windows 11 使用授權；USTP 的學員席位不包含 Windows 授權。** 芯元可依實際需求協助選型、詢價、採購安排及導入，不要求客戶自行判讀各項授權。

## 6. 地圖工具與部署注意事項

- 地圖擷取／產生工具為 **Arnis Runtime Fork**，上游 Arnis 採 Apache-2.0；不需另購商業使用授權，但交付須保留授權、版權及適用的 NOTICE，並標示修改。這是地圖內容工具，不取代 USTP 帳號與成果管理。[Arnis 授權](https://github.com/louis-e/arnis/blob/main/LICENSE)
- OpenStreetMap 資料仍須保留署名並遵守 ODbL；自行產圖不會取消來源資料義務，公共圖磚服務也不等於允許批次下載。[OSM 版權與授權](https://www.openstreetmap.org/copyright)、[公共圖磚使用政策](https://operations.osmfoundation.org/policies/tiles/)
- RAID 不是備份；備份需與正式 Server 分離，並驗證可還原性。單台 Server 不代表不中斷服務。
- 10 GbE Server 上行須搭配對應交換器及線材；SkySim PC 基本採 1 GbE。跨據點需另確認頻寬與延遲。
- 本文件為設備與授權選型依據；硬體容量、VR 適配與第三方授權依實際交付版本、指定考場、資料保存量及部署地區確認。
