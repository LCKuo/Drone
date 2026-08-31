# GIS Server 與模擬 Client 硬體規格（無價格版）

> 用途：讓客戶快速確認「要不要 GIS Server」以及每台設備需要什麼規格。  
> U（User）代表**同時上線、同時使用系統的 Client 數量**，不是帳號總數。

## 先選方案

| 要做的事 | A. 不購買 GIS Server | B. 購買 GIS Server |
|---|---:|---:|
| 執行已經安裝或事先下載好的地圖 | 可以 | 可以 |
| 使用本機 Minecraft／Final Blocks 地圖模式 | 可以 | 可以 |
| 新增 GIS HIGH 高品質城市地圖 | 不可以 | 可以 |
| 從伺服器框選範圍、產生及下載地圖 | 不可以 | 可以 |
| 使用已經存進 Client 的衛星地圖包 | 可以 | 可以 |
| 現場產生新的衛星真實地形 | 不可以 | 可以，但仍須設定合法且可下載的影像來源 |
| Web GIS 後台、工作佇列、地圖集中管理 | 沒有 | 有 |
| 地圖集中更新及多台 Client 共用 | 需人工複製或另備檔案伺服器 | 由 GIS Server 提供 |

### 最重要的容量結論

- **不購買 GIS Server：**沒有 GIS Server 的 U 數瓶頸；每台 Client 各自在本機執行。Client 要買最低或建議規格，取決於畫質與場景複雜度，不取決於總共有幾台 Client。
- **購買 GIS Server：**1–10U 可採最低 Server；11–30U 採建議 Server；**單一 GIS Server 的商用規劃上限為 30U**。
- **31U 起：**不要用一台更大的 Server 硬撐，應拆成主 Server 與工作／下載 Server。
- 上述 U 數指一般操作及下載既有地圖；若要求所有 Client **同時產生新地圖且完全不排隊**，容量門檻會更低，詳見 B-4。

---

## A. 不購買 GIS Server

### A-1. 這個方案能做什麼

- 執行已安裝在 Client 的駕駛模擬程式與既有地圖。
- 開啟已存檔的 Unity 地圖。
- 使用本機 Minecraft／Final Blocks 模式建立或遊玩地圖；需先安裝並設定專案使用的本機地圖工具與資料來源。
- 重複使用已經下載並存入 Client 的衛星地圖包。
- 每台 Client 獨立運作，不會因其他 Client 同時上線而降低本機畫面幀率。

### A-2. 這個方案不能做什麼

- 不能從 GIS Server 即時框選區域、產生 GIS HIGH 地圖或下載新的地圖包。
- 不能集中排程地圖產生工作。
- 不能使用 Web GIS 後台集中管理地圖、Client 與工作狀態。
- 沒有新的可下載影像來源時，不能現場建立新的衛星真實地形。
- 地圖更新需逐台部署、透過共用資料夾，或另建 NAS／檔案發佈設備。

### A-3. Client 最低規格

適用：1080p、標準畫質、既有地圖、一般駕駛模擬。

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

### A-4. Client 建議規格

適用：QHD、高畫質衛星地圖、較大城市、複雜光影、Unity 編輯或未來擴充。

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

### A-5. 依 U 數選擇 Client

| 同時使用數 | Server | Client 選擇方式 |
|---:|---|---|
| 1–10U | 不需要 GIS Server | 1080p／既有地圖可用最低規格；高畫質場景用建議規格 |
| 11–30U | 不需要 GIS Server | U 數不會提高單機硬體需求；為方便大量部署，建議統一採建議規格 |
| 31U 以上 | 不需要 GIS Server | 本機運算仍不受 U 數影響；需另外規劃軟體、地圖與更新的集中發佈方式 |

> **不要誤解：**不買 GIS Server 並不代表 Client 能從網路建立 GIS HIGH 地圖。這個方案的核心是「各 Client 執行事先準備好的內容」。

---

## B. 購買 GIS Server

### B-1. 先按 U 數選 Server 架構

| 同時使用數 | GIS Server 架構 | 結論 |
|---:|---|---|
| 1–10U | 1 台最低規格 Server | 可用最低規格 |
| 11–30U | 1 台建議規格 Server | 單機正式部署的建議範圍 |
| 31–60U | 1 台主 Server + 1 台工作／下載 Server | 單一 Server 不建議負擔 |
| 61–100U | 1 台主 Server + 至少 2 台工作／下載 Server | 依產圖比例與下載流量增加節點 |
| 100U 以上 | API、資料庫、產圖 Worker、儲存與下載服務分離 | 專案化壓力測試後決定節點數 |

> **單一 GIS Server 的商用規劃上限：30U。**31U 起必須採多節點，不把重要限制藏在規格表後面。

### B-2. 最低 GIS Server 規格

適用：1–10U、小型正式環境、同時產圖需求低。

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
| 備份 | 外部 NAS 或獨立備份設備；不可只把 RAID 當備份 |

### B-3. 建議 GIS Server 規格

適用：11–30U、正式商用、未來增加地圖、資料與產圖工作。

| 項目 | 建議規格 |
|---|---|
| 機型 | 2U 機架式 Server |
| CPU | 24–32 實體核心 |
| 記憶體 | 256 GB DDR5 ECC |
| 系統碟 | 2 × 960 GB 企業級 SSD，RAID 1 |
| GIS 資料碟 | 4 × 3.84 TB 企業級 SSD，RAID 10；約 7.68 TB 可用 |
| 網路 | 最低 10 GbE；建議雙 25 GbE 以保留擴充空間 |
| GPU | 現有 GIS 服務不需要 |
| 作業系統 | Ubuntu Server 24.04 LTS |
| 電源與保護 | 冗餘電源、UPS |
| 備份 | 獨立 NAS／備份主機，並保留異機或異地副本 |

### B-4. 同時產生新地圖的限制

一般登入、瀏覽、下載既有地圖，和「產生新地圖」是不同負載。產圖會使用 CPU、記憶體、磁碟與外部資料來源，因此不能只看總 U 數。

| 需求 | 最低 Server | 建議 Server |
|---|---:|---:|
| 規劃中的同時產圖 Worker | 2 | 4（正式上線前需壓力測試） |
| 不排隊的同時產圖 Client | 約 2U | 約 4U |
| 超過 Worker 數 | 進入工作佇列 | 進入工作佇列 |

目前樣本中，單一成功工作中位數約 50 秒，較慢案例約 173 秒。若 30 台 Client 同時送出產圖工作，以 4 個 Worker 粗估：

- 一般樣本清空佇列：約 6.7 分鐘。
- 較慢樣本清空佇列：約 23.1 分鐘。

這是容量估算，不是保證時間；實際時間仍受框選面積、LOD、影像來源速度、地圖複雜度及儲存效能影響。

### B-5. GIS 方案的 Client 最低規格

適用：1–10U 中的 1080p 標準畫質 Client。GIS Server 負責資料與產圖，不代替 Client 的即時 3D 渲染。

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

適用：11U 以上的統一採購，或任何需要 QHD、高畫質城市、複雜光影與較長使用週期的 Client。

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

> **U 數影響的是 GIS Server 架構，不會自動提高每台 Client 的顯示卡需求。**Client 要選最低或建議規格，仍以目標畫質、解析度和場景複雜度為準。

---

## C. 一頁採購判斷

| 客戶需求 | 應採方案 | Server | Client |
|---|---|---|---|
| 只玩既有地圖，不需要線上新增 GIS 地圖 | A. 不購買 GIS Server | 不需要 | 依畫質選最低或建議規格 |
| 使用本機 Minecraft／Final Blocks 模式 | A. 不購買 GIS Server即可 | 不需要 | 依畫質選最低或建議規格 |
| 要從地圖框選區域並建立 GIS HIGH／衛星地形 | B. 購買 GIS Server | 必要 | 依畫質選最低或建議規格 |
| GIS 功能，1–10U | B. 購買 GIS Server | 1 台最低 Server | 最低或建議 Client |
| GIS 功能，11–30U | B. 購買 GIS Server | 1 台建議 Server | 建議統一採建議 Client |
| GIS 功能，31–60U | B. 購買 GIS Server | 主 Server + 1 台工作／下載 Server | 建議 Client |
| GIS 功能，61–100U | B. 購買 GIS Server | 主 Server + 至少 2 台工作／下載 Server | 建議 Client |
| GIS 功能，100U 以上 | B. 購買 GIS Server | 分散式架構，壓力測試後定案 | 建議 Client |

---

## D. 未來擴充時，哪些功能會增加 Server 需求

本節不影響前面的當前採購判斷；只有未來加入下列功能時才需要重新估算。

| 未來功能 | 主要增加的硬體需求 |
|---|---|
| 擴大到全台灣、大量歷史版本、更多衛星影像 | GIS 資料碟容量、備份容量、下載頻寬 |
| 更高 LOD、Reality Mesh、攝影測量 | 產圖 Worker、儲存容量；部分流程可能需要 GPU Worker |
| AI 辨識、AI 建築重建、影像推論 | 獨立 NVIDIA GPU Worker，不建議直接塞進現有主 Server |
| 多租戶、大量外部 Unity Client | API 節點、驗證服務、流量控制、負載平衡 |
| 高可用與不停機 | 至少雙主機、資料庫複寫、共享或物件儲存、備援網路 |
| 31U 以上或大量同時產圖 | 增加工作／下載 Server，不只升級單機 CPU |

## 最終結論

- **不需要線上建立 GIS 地圖：買 Client 即可，不必買 GIS Server。**
- **需要框選、產圖、下載及集中管理：必須買 GIS Server。**
- **GIS 方案 1–10U：最低 Server；11–30U：建議 Server；31U 起：多 Server。**
- **Client 規格看畫質，不看總 U 數；GIS Server 架構才按 U 數擴充。**
