# GIS Server 與模擬 Client：商業交付風險與功能位置盤點

> 盤點日期：2026-09-01
> 對應版本：GIS Server 1.8.1、Unity Map Client 1.8.1
> 盤點範圍：GIS Server、World Selector、Unity 地圖下載、DroneBridge／模擬 EXE、標準 Docker 部署
> 性質：產品與技術決策清單，不取代個案法律意見。

## 一、先看結論

目前不是「整套都不能交付」，但也不是「直接把現在的公開測試環境搬給客戶即可」。

| 交付情境 | 現況判定 | 主要前提 |
|---|---|---|
| 單一客戶、內網、OpenStreetMap 底圖 | **可進入驗收** | 強制 API Key、匯入服務區 OSM／DEM、關閉公共 fallback、移除來源不明模型、補齊 notices |
| 單一客戶、內網、Satellite Real Terrain | **條件式可交付** | 客戶或公司必須提供允許下載、永久快取、衍生 GLB 與約定範圍再散布的影像來源 |
| 公開 Internet 服務 | **目前不可正式交付** | 先完成公開圖磚端點保護、正式反向代理／TLS、密鑰輪替、流量治理與壓力測試 |
| 多租戶 SaaS | **目前不可正式交付** | 先完成 file／tile／map 的 tenant ACL、共享生命週期、稽核與租戶隔離 |

優先級定義：**P0** 是適用交付情境的阻擋項；**P1** 應在正式上線前完成；**P2** 是不阻擋小規模驗收的產品化或維運改善。

### 2026-09-01 實機狀態

- 公開站台與本機回報 GIS Server **1.8.1**，health 正常，現在採 **SQLite／inline queue／local storage**。
- 台灣 World Selector 底圖已安裝完成：Zoom 5–13 預建、Zoom 14–16 按需 CPU 渲染與 Server LRU 快取。
- 實機資料量約 **443 MiB**：台灣 raster MBTiles 約 243 MiB、保留向量來源約 173 MiB、目前動態快取約 27 MiB；不含應用程式、DEM、影像與使用者產生的 3D 地圖。
- Unity 會先抓可視範圍加 2 格緩衝區，優先用單一 ZIP 批次端點，並在本機保存 7 天快取；舊 Server 才回退成 8 路單張下載。
- GIS 自動測試 **43 項通過**，Ruff 與 Python compileall 通過。
- Satellite 影像目前是安全預設：`IMAGERY_PROVIDER=disabled`，不會把未確認權利的來源包進 Unity 地圖。
- 公開站台目前仍回報 `apiKeyRequired=false`。這是測試狀態，不是客戶正式環境設定。

## 二、已完成，不應再列為現行缺失

| ID | 已完成項目 | 原本用在哪裡 | 目前結果 | 驗收證據 |
|---|---|---|---|---|
| C01 | 移除 GIS High／PBR Models 地圖產生模式 | GIS Generator、後台、Unity World Selector | Server 只保留 Satellite Real Terrain；不再用 GAME-READY ASSETS 補真實地圖 | discovery：`visualStyles=[satellite]`、`satelliteUsesGameReadyAssets=false` |
| C02 | 移除 3DMR 與 GIS 3D 模型庫 | 建築、植物、道具的模型解析與後台模型資產頁 | 正式 API 能力已關閉；建築改為 OSM footprint／height 的白灰程序化 GLB | discovery：`threeDMR=false`、`assetResolver=false`、`pbrMaterials=false` |
| C03 | 移除 Esri／來源不明的可下載預覽包 | 舊 Taipei 101 package、靜態 GLB demo | Esri 只剩 Web 即時底圖用途；不再當 Unity 離線包來源 | 下載影像與 Web 預覽走分離路徑，imagery gate 預設關閉 |
| C04 | 移除 Unity 直連 OSM 公用 raster endpoint | World Selector 背景與預抓 | Unity 拒絕 `tile.openstreetmap.org`，只接受 GIS discovery 回傳的同源底圖 | `DroneWorldMapSelector` 有明確拒絕規則 |
| C05 | 影像權利改為明確閘門 | terrain 貼圖、GLB、共享站台包 | 下載、永久快取、衍生與再散布四項權利分開設定；未知一律為 `false` | `ImageryProvider` 與 manifest 都保留權利旗標 |
| C06 | 新增站台共享地圖下載 | Unity 原本只能使用自己剛產生的地圖 | Unity 可列出並下載 ready、選擇分享且允許再散布的站台地圖 | `/api/v1/site-maps`、`/api/v1/site-maps/{mapId}` |
| C07 | 台灣國家底圖包已落地 | World Selector | 台灣已在 Server 安裝，區域外 tile 會被拒絕，Unity 不能拖到覆蓋範圍外 | 21,968 張預建 tile；discovery 回報 Taiwan、Zoom 5–16 |
| C08 | 國家底圖可由後台自行擴充 | 後台「國家底圖包」 | 內建台灣、菲律賓、越南、日本關東；管理者亦可貼入官方 Geofabrik Shortbread URL | URL 僅允許 HTTPS、`download.geofabrik.de` 與 Shortbread MBTiles |
| C09 | Unity World Selector 改為批次傳輸 | 首次進入、拖動後載入 | 可視範圍加緩衝區一次下載，已避免逐張串行抓取；Server 無需 GPU | `/api/v1/basemap/{regionId}/batch.zip`，每批最多 256 張 |

## 三、仍存在的風險與缺口

### A. GIS Server／圖資來源

| ID | 優先級 | 風險或缺口 | 實際用在何處 | 目前狀態 | 交付前處理 |
|---|---|---|---|---|---|
| G01 | **P0** | 可下載衛星／正射影像尚無統一商業授權 | Satellite terrain、Unity GLB、站台共享包 | provider 預設停用，因此目前安全但功能不可用 | 使用公司自有、客戶提供或有書面合約的 `local`／`template` 來源；逐一保存四項權利證據 |
| G02 | **P0** | NLSC 公用服務不得當成批次離線影像來源 | `taiwan_nlsc` adapter | adapter 存在但權利旗標預設全為 false；官方條款明載不得大量下載 | 保持停用；除非取得另行書面授權，否則不可改成可下載／衍生／再散布 |
| G03 | **P0** | Generator 仍可回退到公共 Overpass | OSM 建築、道路碰撞、metadata | `OSM_PROVIDER=hybrid`、`OSM_ALLOW_OVERPASS_FALLBACK=true` | 客戶服務區 PBF 匯入 PostGIS 後設為 `postgis` 並關閉 fallback；公共服務只供開發驗證 |
| G04 | **P0** | DEM 仍可使用網路 Terrarium fallback | terrain 高程與碰撞 | `DEM_PROVIDER=hybrid`、`DEM_ALLOW_NETWORK_FALLBACK=true`；資料來源與 attribution 未形成完整 release snapshot | 交付區域匯入可商用 DEM、固定資料版本、補 attribution／license，離線交付關閉 network fallback |
| G05 | **P1** | Web 互動式 3D 地圖仍直接使用 Esri World Imagery | 後台「3D 地圖」瀏覽器畫面 | 只顯示、不輸出 Unity package；畫面有 attribution，但未使用正式 token／計費方案 | 正式版改用合規 ArcGIS Location Platform 設定，或改接公司自架 imagery；不得把瀏覽權推定為離線再散布權 |
| G06 | **P1** | OSM／Geofabrik ODbL 履約需做成 release gate | World Selector、建築、道路、Arnis 輸出 | 畫面已有 `© OpenStreetMap contributors`，但安裝包與資料快照仍需逐版驗收 | 保留 attribution、ODbL 連結、資料日期與衍生資料庫評估；不得移除來源標示 |
| G07 | **P1** | 國家底圖沒有正式更新、備份與回復演練 | Taiwan／未來國家 MBTiles 與動態 cache | 可下載／重建，但尚未定義更新週期、rollback、磁碟警戒與備份演練 | 建立每月或專案版更新政策；保留來源 hash、版本、建置紀錄與可還原備份 |
| G08 | **P2** | 舊 `docker-compose.osm.yml` 仍以 `latest` renderer image 存在 | 舊 PBF raster renderer 流程 | 目前台灣主線用 Geofabrik Shortbread，不依賴此容器；但檔案仍可能被誤當正式方案 | 若保留就鎖 digest、做 SBOM；若不再支援舊流程則從正式交付包移除 |

### B. 公開端點、效能與安全

| ID | 優先級 | 風險或缺口 | 實際用在何處 | 目前狀態 | 交付前處理 |
|---|---|---|---|---|---|
| N01 | **P0** | 公開環境未強制 Client API Key | maps、jobs、site maps、files | 公開站台回報 `apiKeyRequired=false` | 正式環境設 `REQUIRE_API_KEY=true`，每客戶獨立 key、到期日、撤銷與輪替 |
| N02 | **P0** | basemap 單張與 `batch.zip` 沒有驗證或全域 rate limit | Unity World Selector | 即使開啟 API Key，這兩條 route 目前仍未套 `require_client`；batch 的 8 路限制是每個 request 自己的 semaphore | 加 API Key／簽名 URL、全域並發與每 IP／tenant 配額；z5–13 可交給反向代理快取，z14–16 要有 CPU backpressure |
| N03 | **P1** | 每次 batch 都重新組 ZIP，且回應 `no-store` | World Selector 首屏及拖動 | tile 有 Server cache，但 ZIP 本身不快取；多用戶同區域仍重做 JPEG／ZIP | 以 region＋version＋z＋range 做短期 archive cache／ETag，或由 proxy 快取；壓測後設定上限 |
| N04 | **P1** | Zoom 14–16 是 CPU 冷渲染，沒有全域工作池 | 首次觀看未快取區域 | 不需要 GPU；單機正常，但 30 個 Client 同時打不同冷區域尚無容量證據 | 建立 process worker pool、全域 queue、超時與拒絕策略；做 1／10／30 Client 冷熱快取壓測 |
| N05 | **P0** | 預設密碼、JWT、CORS 與 Internet 暴露設定不適合正式環境 | 後台登入、API、Tunnel | `.env.example` 是開發預設；CORS 預設 `*`；只有 Compose nginx 對登入做限流 | 交付腳本必須拒絕預設 secret、限制 CORS、加 TLS／VPN／IP policy、登入限流與稽核 |
| N06 | **P1** | `/metrics` 目前公開 | Prometheus 指標 | 不需要登入即可讀取 | 僅綁內網監控介面，或由反向代理加 ACL／驗證 |
| N07 | **P1** | 目前 Tunnel 是方便驗證的入口，不是服務容量證明 | 外部測試站台 | Tunnel 可連，但不代表 Server HA、備援或固定 SLA 已完成 | 客戶正式部署要確認為可管理的 named tunnel 或客戶既有 reverse proxy；另做備援、監控與恢復演練 |

### C. Unity 地圖下載／共享

| ID | 優先級 | 風險或缺口 | 實際用在何處 | 目前狀態 | 交付前處理 |
|---|---|---|---|---|---|
| U01 | **P1** | 站台共享缺少明確 opt-in／取消分享入口 | `MapRequest.shareWithSite`、站台地圖庫 | Server 只會公開允許再散布的資料，但 Unity UX 尚未完整管理分享狀態 | 新增建立時勾選、擁有者／管理員取消分享、引用提示與稽核記錄 |
| U02 | **P2** | 共享 catalog 缺少搜尋、游標分頁與專案群組 | World Selector 的 Shared Site Maps | 目前最多 200 筆、依建立時間排列 | 大量資料前補 tenant／project visibility、搜尋、分頁與地區 filter |
| U03 | **P0** | tile manifest 與 file endpoint 沒有逐檔 tenant ACL | GLB、manifest、metadata 下載 | map／job 有 owner 檢查，但知道 file 或 tile ID 的其他有效 Client 仍可能讀取 | 建立 file → tile → map 授權關係；private 檔案查 tenant，共享檔案查 shared entitlement，並記錄下載 |
| U04 | **P1** | 共享 package 沒有生命週期政策 | 刪除、cache 清理、舊存檔重開 | 尚無 retained／deprecated／removed 與 replacement map id | 定義保留期、引用數、替代版本及刪除延遲 |
| U05 | **P1** | attribution 需要每個正式 build 驗收 | World Selector 與載入後遊戲 HUD | World Selector 與 GIS runtime 已有顯示程式，但不能只驗收 editor 畫面 | QA 固定檢查所有 RequiredOnScreen 項目、語系、解析度與全螢幕模式 |
| U06 | **P2** | Unity package 的 `documentationUrl` 仍是 `map.example.com` | Package Manager 資訊 | 版本已是 1.8.1，但文件連結是 placeholder | 發版前改成正式文件網址 |

### D. 標準 Server 部署元件

| ID | 優先級 | 元件與風險 | 實際用在何處 | 目前狀態 | 建議決策 |
|---|---|---|---|---|---|
| S01 | **P0** | Redis 7.4 採 RSALv2／SSPLv1 | queue、rate limit、distributed tile lock | 本機 1.8.1 不使用 Redis；但標準 Compose 預設 `redis:7.4-alpine` | 標準交付改 Valkey，或退回適合的 BSD 版本並完成安全評估；不要把法律判斷留給客戶 |
| S02 | **P0** | MinIO server 為 AGPLv3，且上游 repository 已封存 | 多 worker 共享 object storage | 本機 1.8.1 用 local storage；標準 Compose 仍預設 MinIO 2025 image | 單機版用 local storage；多節點接客戶既有 S3／合規物件儲存，或完成 AGPL／商業授權評估 |
| S03 | **P1** | Container image 未全部鎖 digest | PostGIS、Redis、MinIO、nginx、舊 OSM renderer | 多數只鎖 tag；供應鏈與重建結果可能漂移 | 驗收版鎖 digest、保存離線 image、SBOM、掃描結果與更新流程 |
| S04 | **P0** | SBOM／第三方 notices 未自動化且內容已過時 | Server、Unity、容器、字型、資料來源 | GIS `THIRD_PARTY_NOTICES.md` 仍描述已移除的 Kenney／Poly Haven／Taiwan HD 資產，卻未完整覆蓋目前 Python、MapLibre、OSM、DEM 與 container | 從實際交付物自動產生 SBOM／notices；刪除不存在項目並補上版本、來源、授權、修改與資料日期 |

### E. 模擬 EXE／DroneBridge

| ID | 優先級 | 風險或缺口 | 實際用在何處 | 目前狀態 | 交付前處理 |
|---|---|---|---|---|---|
| E01 | **P1** | Arnis Runtime Fork 與 OSM 輸出義務 | OpenStreetMap 體素地圖模式 | 是保留的獨立模式，不是 GIS 模型庫 | 保留 Apache-2.0 LICENSE／NOTICE，並履行 OSM attribution／ODbL |
| E02 | **P0** | DJI Inspire 2 FBX 的商用與再散布證據不完整 | DroneBridge imported model、airframe profile、Unity QA | 檔案仍存在，旁邊只有使用說明，未找到可證明再散布權的 license | 正式安裝包排除，直到取得來源、作者、商用、修改與再散布證據 |
| E03 | **P1** | Parrot／Sketchfab 外觀與品牌使用 | 內建飛行器外觀與 profile | 兩個模型標示 CC BY 4.0；兩個 download-disabled 項目使用程序化 placeholder | 發版逐檔核對 attribution 與實際打包內容；download-disabled 資產不得打包，商標命名另行審查 |
| E04 | **P1** | pymavlink／PX4／glTFast／ufbx 等 notices 要對齊實際 build | SITL、遙測、GLB 與 FBX 載入 | 核心功能存在，部分 license 檔已隨專案放置，但沒有單一 release manifest | 依實際 build 產生第三方清單，附版本、來源、license、修改狀態與動態依賴 |
| E05 | **P0** | 訓練精度與安全用途界線 | GIS、飛行／駕駛模擬 EXE、驗收文件 | 地圖、高度、飛行物理都有資料日期或工程估算，不可作唯一導航或安全決策來源 | UI、手冊、報價與合約一致標示訓練／模擬用途、資料日期、精度與不適用情境 |

## 四、功能去留決策表

| 項目 | 用在哪裡 | 建議 | 砍除影響 |
|---|---|---|---|
| GIS High／PBR Models | 舊 GIS 產生模式 | **維持移除** | 無；Satellite 主線不依賴它 |
| 3DMR／GIS 3D 模型庫 | 舊模型解析與資產頁 | **維持移除** | 無；OSM 白灰建築仍可產生 |
| Esri 可下載預覽包 | 舊 demo／Unity package | **維持移除** | 無；避免把 Web 瀏覽權誤作離線再散布權 |
| Web Esri 即時底圖 | 後台 3D 地圖 | **條件保留或換自架 imagery** | 移除後仍可顯示 OSM 向量與 3D 建築，但沒有該衛星底圖 |
| NLSC 公用 imagery adapter | Satellite terrain | **保留介面、預設停用** | 無書面授權時不能用；取得授權後才有快速接入點 |
| Geofabrik Shortbread／OSM | World Selector | **保留** | 砍除後失去國家底圖與框選能力；應履行 ODbL，而不是移除 |
| Overpass fallback | 開發時補 OSM 幾何 | **正式環境關閉** | 關閉前必須先匯入服務區 PBF |
| Terrarium network fallback | 開發時補 DEM | **離線／正式環境關閉** | 關閉前必須準備本機 DEM |
| 站台共享地圖 | 讓其他 Unity 使用者重用區域 | **保留並補 ACL／生命週期** | 砍除會增加重複生成、流量與等待時間 |
| Redis 7.4 | 多 worker queue／協調 | **替換** | 單機 inline 不受影響；多 worker 需 Valkey 或其他 queue |
| MinIO Community | 多 worker object storage | **單機移除，多節點替換** | 單機 local storage 不受影響；多節點需 S3 類儲存 |
| DJI 來源不完整 FBX | 飛行器外觀 | **交付版排除** | 不影響 SITL 核心，只少一個外觀選項 |
| attribution、manifest、SBOM、用途告知 | 全產品 | **不得移除** | 會失去授權證據與責任邊界 |

## 五、建議的低風險交付組合

### GIS Server

- `REQUIRE_API_KEY=true`，每客戶獨立金鑰。
- PostGIS 放入交付區域 OSM，`OSM_PROVIDER=postgis`、`OSM_ALLOW_OVERPASS_FALLBACK=false`。
- 本機 DEM，`DEM_ALLOW_NETWORK_FALLBACK=false`。
- 沒有合格影像合約時維持 `IMAGERY_PROVIDER=disabled`；不要為了有衛星畫面而繞過權利閘門。
- 單機採 local storage；需要多 worker 時，先決定合規 queue 與 object storage。
- 由正式 reverse proxy 提供 TLS、登入限流、basemap rate limit、監控端點 ACL 與存取日誌。
- 台灣底圖保留現行 Zoom 5–13 預建、14–16 按需快取；不把所有 Zoom 14–16 全台預建當成必要條件。

### Unity／模擬 EXE

- Unity Map Client 與 GIS Server 固定使用相同 1.8.1 API schema。
- 保留 Server batch 下載與 Unity 本機 cache；加入 API Key／簽名請求後再做公開部署。
- 正式 build 排除無完整 license 的 DJI FBX，保留已驗證 CC BY 與程序化 placeholder。
- 所有地圖與模型的 attribution 必須在實際 Player build 中驗收，不只在 Unity Editor 看得到。

## 六、正式交付 Gate

### 已完成

- [x] GIS High、3DMR、GIS 模型庫與可下載 Esri demo package 已退出正式主線。
- [x] Unity 不直連或預抓 OSM 官方公用 raster tiles。
- [x] 台灣底圖 Zoom 5–16 已由 Server 提供，Unity 有邊界限制、批次下載與本機 cache。
- [x] 影像下載／快取／衍生／再散布權利採預設拒絕。
- [x] 站台 catalog 只列出 ready、選擇分享且允許再散布的地圖。
- [x] GIS 43 項自動測試與靜態檢查通過。

### 交付前必須完成

- [ ] 正式環境強制 API Key，並保護 basemap／batch／metrics。
- [ ] 更換預設 admin／JWT／資料庫／儲存密碼，限制 CORS，完成 TLS 與登入限流。
- [ ] 服務區 OSM 已匯入，正式環境關閉 Overpass fallback。
- [ ] 本機 DEM 已匯入並固定版本，正式環境關閉 Terrarium fallback。
- [ ] 若交付 Satellite，已保存影像四項權利及 attribution 的書面證據。
- [ ] Private tile／file 已完成 tenant ACL；共享 map 有 opt-in、取消分享與生命週期。
- [ ] Redis 7.4／MinIO 已替換，或交付方式已完成正式授權審查。
- [ ] DJI 來源不完整模型已從正式安裝包排除。
- [ ] 依實際交付物產生 SBOM、第三方 notices、資料授權與 container digest 快照。
- [ ] 完成 1／10／30 Client 的 World Selector 冷快取、熱快取、地圖產生與下載壓測。
- [ ] UI、手冊、報價與合約一致標示訓練用途、資料日期、精度與不適用情境。

## 七、官方依據

- [OpenStreetMap Tile Usage Policy](https://operations.osmfoundation.org/policies/tiles/)
- [OpenStreetMap Copyright and ODbL](https://www.openstreetmap.org/copyright)
- [Geofabrik OpenStreetMap Data Extracts](https://download.geofabrik.de/)
- [國土測繪圖資服務雲使用規定](https://maps.nlsc.gov.tw/pro/use_clause.jsp)
- [Esri basemap attribution](https://developers.arcgis.com/documentation/mapping-and-location-services/mapping/basemaps/introduction-basemap-styles-service/)
- [Esri terms of use](https://developers.arcgis.com/documentation/terms-of-use/)
- [Redis licenses](https://redis.io/legal/licenses/)
- [MinIO repository and license](https://github.com/minio/minio)
- [Cloudflare Quick Tunnels](https://developers.cloudflare.com/cloudflare-one/networks/connectors/cloudflare-tunnel/do-more-with-tunnels/trycloudflare/)
- [Mapzen Terrain Tiles on AWS](https://github.com/awslabs/open-data-registry/blob/main/datasets/terrain-tiles.yaml)

## 八、最終判斷

1.8.1 已把 World Selector 的核心技術缺口補起來：台灣底圖在 Server、Unity 有邊界、Zoom 16、批次下載與本機 cache，且 Server 不需要顯示卡。現在最該處理的不是再換地圖外觀，而是公開端點保護、資料來源離線化、多租戶 ACL、正式 deployment 元件選型，以及 release 授權證據。

在這些 Gate 完成前，可持續做單機與內網驗收；不能把目前公開測試站台直接視為已完成的商用 Internet／多租戶服務。
