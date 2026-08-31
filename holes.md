# GIS Server 與模擬 Client：商業交付風險與功能位置盤點

> 盤點日期：2026-08-31  
> 對應版本：GIS Server 1.6.0、Unity Map Client 1.6.0  
> 範圍：GIS Server、World Selector、Unity 地圖下載、DroneBridge／模擬 EXE 與標準 Docker 部署  
> 本文件是產品與技術決策清單，不取代個案法律意見。

## 一、這次已完成的移除與修正

下列項目已不再是正式產品功能；隔離備份只供回溯，不得放入客戶安裝包。

| ID | 已完成項目 | 原本用在哪裡 | 現在狀態 | 對客戶功能的影響 |
|---|---|---|---|---|
| C01 | GIS High／PBR Models 地圖模式 | GIS Generator、後台區域產生、Unity World Selector | **已移除**；Server 只接受 Satellite Real Terrain | 不再有兩套外觀不一致的真實地圖；保留 OpenStreetMap 體素模式與 Satellite 模式 |
| C02 | 3DMR 及 GIS 內建 3D 模型庫 | 建築、植物、道具的模型解析與後台模型瀏覽器 | **程式、API、UI、Docker 掛載及資產匯入工具均已移除** | Satellite 建築固定使用 OSM footprint／height 的白灰程序化 GLB；不再補第三方模型 |
| C03 | Esri／來源不明的可下載預覽地圖包 | DroneBridge 兩組 Taipei 101 local package、Web 靜態 GLB preview | **已從正式路徑移除並隔離** | 不影響即時 Web 3D 地圖或 Server 新產生的合法 GLB；少了舊示範存檔 |
| C04 | Unity 直連 OSM 官方公用 raster tiles | World Selector 背景圖、可見範圍抓取與快取 | **已移除**；只接受 GIS discovery 回傳的同源自架底圖 | 自架 renderer 未部署時會清楚顯示未設定，不會偷偷向公用服務批次抓取 |
| C05 | 影像權限硬編碼 | `licenses.json`、地圖是否能分享 | **已修正**；下載、永久快取、衍生與再散布分開設定，未知一律為 `false` | 不會再把無證據的圖資標成可交付或可分享 |
| C06 | 站台地圖下載缺口 | Unity 只能下載自己剛產生的地圖 | **已新增** `/api/v1/site-maps`、站台 package 載入及 World Selector 選擇 UI | 其他使用者可下載已完成、已選擇分享且授權允許再散布的區域 |

## 二、仍需決策或完成的項目

### A. GIS Server／World Selector

| ID | 風險或缺口 | 實際用在何處 | 現況 | 若直接砍除 | 建議決策 | 交付狀態 |
|---|---|---|---|---|---|---|
| G01 | 可下載影像來源尚未取得統一商業授權 | Satellite terrain 貼圖、Unity 站台地圖包 | `IMAGERY_PROVIDER=disabled` 為安全預設；NLSC adapter 不預設宣告下載、衍生或再散布權 | Satellite 模式無法產生，但 OSM 體素模式與既有合法地圖仍可用 | 由公司或客戶提供自有 imagery，或簽訂可下載、衍生及交付的供應商合約；逐 provider 保存書面證據 | **Release blocker** |
| G02 | 自架 OSM renderer 尚未在正式主機完成 PBF 匯入 | World Selector 底圖 | 程式、Compose overlay、容量檢查與同源 proxy 已完成；目前工作站沒有 Docker，且約 432 GB 可用空間低於 Planet 1.2 TB 防線 | Selector 沒有背景圖；Satellite 既有 package 仍可下載 | 先用 Taiwan extract 驗證；全球服務在有足夠 NVMe 的 Server 匯入 Planet PBF。不得在目前工作站硬抓完整世界 | **Deployment blocker** |
| G03 | 地圖幾何仍可回退到公共 Overpass | OSM 建築輪廓、道路碰撞、metadata | `OSM_PROVIDER=hybrid` 且 Compose 預設允許 fallback；自架 raster renderer 的資料庫不是 Generator 的 OSM feature schema | 關閉前若未匯入本機資料，建築與道路可能缺失 | 用 `scripts/import-osm.ps1` 匯入服務範圍 PBF 至 GIS PostGIS，再設 `OSM_ALLOW_OVERPASS_FALLBACK=false` | **Release blocker（正式／離線部署）** |
| G04 | DEM 仍有網路 fallback | GIS terrain、Unity 高程與碰撞 | 本機 HGT 優先，缺檔時可用 Terrarium；DroneBridge 的 Arnis 流程另有 Mapterhorn 依賴 | 關閉且未放 DEM 時地形會變平 | 對交付範圍匯入具授權的 DEM，關閉 fallback；另行確認 Mapterhorn 條款或替換 | **Release blocker（離線部署）** |
| G05 | Web 互動式 3D 地圖仍使用 Esri browser-only imagery | 管理後台「3D 地圖」即時瀏覽 | 它只做瀏覽器預覽，不會再被包成 Unity GLB；可下載輸出走獨立授權閘門 | Web 仍可用向量底圖與 OSM 建築，但失去該衛星底圖 | 若要零 Esri 依賴，改接公司自架 imagery；在替換前保持 browser-only、正確 attribution，禁止輸出或離線包裝 | **條件保留** |
| G06 | OSM ODbL 義務 | OSM PBF、建築／道路、World Selector 自架 tiles、Arnis Runtime Fork 輸出 | attribution 已在 Server／manifest 設計中，但每個正式畫面與安裝包仍需驗收 | 砍掉 OSM 會失去核心真實世界幾何與底圖 | 保留；顯示 `© OpenStreetMap contributors`、ODbL 連結、資料日期，並就分發的衍生資料庫做個案審查 | **必須保留並履約** |
| G07 | Planet 與圖磚更新策略尚未正式化 | 全球 World Selector 底圖 | importer 可做一次性匯入；更新、expire、備份與 rollback 尚未在客戶 Server 演練 | 不更新會逐漸過時；全砍則沒有自架全球底圖 | pin renderer image、建立月／週更新政策、監控磁碟與 render queue、演練 volume 備份還原 | **上線前完成** |
| G08 | OSM tile renderer image 使用 `latest` | `docker-compose.osm.yml` | 功能可用，但不可重現且存在供應鏈漂移 | 移除後需自行維護完整 renderer | 在驗收版本鎖定 image digest，保存 SBOM 與離線 image；更新需重新驗證 | **上線前完成** |

### B. Unity 地圖下載／共享

| ID | 風險或缺口 | 實際用在何處 | 現況 | 若直接砍除 | 建議決策 | 交付狀態 |
|---|---|---|---|---|---|---|
| U01 | 新地圖預設提出站台分享 | `MapRequest.shareWithSite`、World Selector 動態生成 | 只有 imagery 允許再散布時才真的公開；但目前 UI 沒有明顯的分享勾選與取消分享入口 | 砍共享功能則其他使用者不能重用地圖，Server 會重複生成 | 加上「站台共享」明確勾選、地圖擁有者／管理員取消分享、刪除前引用提示與稽核記錄 | **產品化缺口** |
| U02 | 共享 catalog 沒有分頁游標、搜尋或權限群組 | World Selector 站台 package 清單 | 目前最多 200 筆、依建立時間排列，所有合法 shared maps 對有效 Client 可見 | 小型站台可用；大量客戶時清單難管理 | 加 tenant／project visibility、搜尋、分頁、地區與建立者 filter | **擴充前完成** |
| U03 | File endpoint 只驗證 Client，沒有逐檔 tenant ACL | GLB、manifest、metadata immutable download | 非共享 private map 的 file ID 很難猜，但若 ID／manifest 洩漏，其他有效 Client 可直接抓檔 | 全砍 file endpoint 會使 Unity 無法下載 | 建立 file → tile → map 授權關係；private 檔案需 tenant ACL，共享檔案需 shared entitlement；加下載稽核 | **Release blocker（多租戶）** |
| U04 | Shared package 生命週期未定義 | 站台地圖刪除、cache 清理、客戶存檔重開 | Catalog 會重用原 map 檔案，尚無保留期限或版本淘汰政策 | 永不刪會膨脹；直接清 cache 會破壞客戶存檔 | 設定 retained／deprecated／removed 狀態、保留期、引用數及 replacement map id | **產品化缺口** |
| U05 | Unity 必顯 attribution 仍需 UI 驗收 | `MapLoadResult.Attribution` | SDK 已提供資料，但各遊戲畫面是否持續顯示需逐 build 驗收 | 移除 attribution 會增加 OSM／imagery 違約風險 | 在遊戲 HUD／地圖畫面固定顯示 `RequiredOnScreen` 項目，加入 QA | **Release blocker** |

### C. 標準 Server 部署元件

| ID | 元件與風險 | 實際用在何處 | 現況 | 若移除 | 建議決策 | 交付狀態 |
|---|---|---|---|---|---|---|
| S01 | Redis 7.4 採 RSALv2／SSPLv1 | queue、rate limit、distributed tile lock | `redis:7.4-alpine` 是 Compose 預設 | 單程序可改 inline；多 worker 會失去協調 | 標準交付改用 Valkey，或由法務確認選定 Redis license 與交付方式 | **Release blocker** |
| S02 | MinIO server 為 AGPLv3，且上游 repository 已封存 | GLB／manifest 的多 worker 共享 object storage | Compose 使用固定的 2025 image；單機可用 local storage | 單機影響小；多節點需另一個 S3 相容服務 | 單機標準版改 local storage；多節點接客戶既有 S3 或另選維護中的合規產品 | **Release blocker（標準 Compose）** |
| S03 | 預設 API 防護不足 | 公開 tunnel／Internet GIS | `REQUIRE_API_KEY=false`、預設密碼與寬鬆設定只適合 LAN | 砍驗證不可接受 | 對外一律 API key、限制 CORS、TLS、強密碼、secret rotation、登入限流與管理端 IP／VPN 控制 | **Release blocker** |
| S04 | SBOM／第三方 notices 未自動化 | Server、Unity、容器、字型與資料來源 | 有部分 notices，但不是每次 release 的完整快照 | 刪除文件只會失去合規證據 | CI 產生 Python／Unity／container SBOM、license 清單、image digest 與資料來源快照 | **Release blocker** |

### D. 模擬 EXE／DroneBridge

| ID | 風險或缺口 | 實際用在何處 | 現況 | 若直接砍除 | 建議決策 | 交付狀態 |
|---|---|---|---|---|---|---|
| E01 | Arnis Runtime Fork 及其 OSM 輸出義務 | World Selector 的 OpenStreetMap 體素模式 | 這是另一種保留模式；不是 GIS 模型庫 | 體素地圖模式消失，Satellite 模式不受影響 | 保留 Apache-2.0 LICENSE／NOTICE，並履行 OSM attribution／ODbL | **條件保留** |
| E02 | DJI Inspire 2 外觀模型的再散布證據不完整 | `DroneBridge/config/imported_models`、airframe profile、Unity QA | 目前仍存在於模擬專案，並非本次 GIS 修改範圍 | 只少一個選配外觀；SITL 與其他機型仍可用 | 交付版先排除，除非取得來源、作者、商用及再散布證明 | **Release blocker** |
| E03 | Parrot／Sketchfab 外觀與品牌 | 內建飛行器外觀及 profile | 部分有 CC BY、部分只提供不可下載連結或程序化 placeholder | 物理可保留，外觀選項減少 | 每個實際隨 EXE 分發的檔案逐一驗證 license；不可下載資產不得打包，品牌名稱與商標需個案審查 | **Release blocker** |
| E04 | pymavlink／PX4 等模擬依賴 | SITL 控制、遙測與飛行物理 | 屬核心功能，不能以刪除授權文件解決 | 砍除後 SITL 無法運作 | 保留並附版本、來源、license、修改狀態及動態依賴說明 | **必須保留並履約** |
| E05 | 訓練精度與安全用途界線 | GIS、Unity 地圖、駕駛／無人機模擬 EXE、驗收文件 | 資料有時間差與推估，不能當唯一導航或安全決策來源 | 移除告知會提高誤用與責任風險 | UI、手冊、報價及合約一致標示模擬／訓練用途、資料日期、精度及不適用情境 | **Release blocker** |

## 三、是否從產品移除：快速決策表

| 項目 | 建議 | 原因 |
|---|---|---|
| GIS High／PBR Models | **維持移除** | 與 Web 視覺不一致，且引入模型／材質授權與維護成本 |
| 3DMR／GIS 3D 模型庫 | **維持移除** | 不是 Satellite 核心；移除後仍可用 OSM 白灰建築 |
| Esri 可下載預覽包 | **維持移除** | browser view 權利不能推定為離線 GLB 再散布權 |
| Web Esri 即時預覽 | **暫時條件保留或換自架 imagery** | 只用於後台瀏覽，不進 Unity package；若要完全零第三方依賴則替換 |
| OSM 公用 raster endpoint | **維持移除** | 官方禁止 bulk download／prefetch；改用 PBF 自架 renderer |
| OpenStreetMap 資料 | **保留** | 是 World Selector、建築、道路與體素模式核心；重點是履行 ODbL |
| Satellite imagery adapter | **保留介面，無授權來源時停用** | 直接刪除會失去產品主要寫實模式 |
| 站台共享地圖 | **保留並補權限／生命週期** | 可降低重複生成與流量，但必須有 opt-in、ACL、取消分享與稽核 |
| Redis 7.4 | **替換為 Valkey** | 避免 RSALv2／SSPLv1 商業交付不確定性 |
| MinIO Community image | **單機移除，多節點替換** | AGPL／維護狀態與產品交付不匹配；storage abstraction 可保留 |
| Arnis Runtime Fork | **保留** | 是獨立的 OpenStreetMap 體素模式；附 Apache-2.0 及 ODbL 文件即可 |
| DJI 來源不完整模型 | **交付版移除** | 不影響核心飛行功能，卻有直接再散布風險 |
| manifest、notices、SBOM、用途告知 | **不能移除** | 這些是交付證據與責任邊界，不是多餘功能 |

## 四、目前可以做什麼、還不能承諾什麼

| 問題 | 現在的準確答案 |
|---|---|
| GIS High／3DMR 是否仍會被使用？ | 不會；正式執行路徑、UI、API、Docker 及資產庫均已移除。 |
| Unity 能否下載其他使用者的地圖？ | 能；只列出 ready、使用者選擇分享、且 provider 允許再散布的地圖。 |
| 現在能否合法把 NLSC／Esri 畫面存成站台包給所有人？ | 不能直接推定。沒有對該資料集及使用方式的書面權利證據，就不會進共享 catalog。 |
| World Selector 是否已經有全球離線底圖？ | 程式與匯入流程已完成，但目前機器未匯入 Planet；正式 Server 仍需 Docker、足夠儲存與實際 import／render 驗證。 |
| 目前能否直接作為多租戶 Internet SaaS 交付？ | 還不能；至少先完成 G01、G03、U03、U05、S01–S04。 |
| 能否先做單一客戶內網版本？ | 可以，但仍需合法 imagery／DEM、OSM attribution、API key、移除未授權飛行器資產及完整 notices。 |

## 五、正式交付 Gate

- [x] GIS High、3DMR、GIS 模型庫與相關 UI／API 已移除。
- [x] 舊 Esri／不明 imagery 的可下載 preview package 已移出正式路徑。
- [x] Unity 不直連或預抓 OSM 官方 raster tiles。
- [x] 站台地圖 catalog 只列出允許再散布的完成地圖。
- [ ] 正式 imagery provider 的四項權利與書面證據已確認。
- [ ] 服務區 PBF 已匯入 Generator PostGIS，正式環境已關閉 Overpass fallback。
- [ ] 自架 basemap renderer 已在目標 Server 完成 import、render、更新與備份演練。
- [ ] 本機 DEM 已匯入，離線方案已關閉 network fallback。
- [ ] Private file endpoint 已完成 tenant ACL；共享 map 有取消分享與生命週期管理。
- [ ] Redis 7.4 與 MinIO 已替換或完成法務核准的交付方式。
- [ ] Internet 部署已強制 API key、TLS、restricted CORS、強密碼與 secret rotation。
- [ ] 每次 release 已產生 SBOM、第三方 notices、資料授權快照與 container digest。
- [ ] Unity／模擬 EXE 持續顯示必顯 attribution，並排除授權不完整的飛行器模型。
- [ ] UI、手冊、報價與合約已一致標示訓練用途、資料日期與精度界線。

## 六、官方依據

- [OpenStreetMap Tile Usage Policy](https://operations.osmfoundation.org/policies/tiles/)
- [OpenStreetMap Planet files](https://planet.openstreetmap.org/)
- [OpenStreetMap Copyright and ODbL](https://www.openstreetmap.org/copyright)
- [Overv OpenStreetMap Tile Server](https://github.com/Overv/openstreetmap-tile-server)
- [國土測繪圖資服務雲使用規定](https://maps.nlsc.gov.tw/pro/use_clause.jsp)
- [Redis licenses](https://redis.io/legal/licenses/)
- [MinIO license](https://github.com/minio/minio/blob/master/LICENSE)

## 七、結論

這次修改已把最容易混淆且不是核心的 GIS High、3DMR、GIS 模型庫與可下載 Esri 測試包移除；Server 與 Unity 的主線現在一致為 Satellite Real Terrain，加上受授權控制的站台共享地圖。

剩餘風險主要不是「畫面功能做不出來」，而是正式資料授權、自架全球資料的部署容量、多租戶檔案 ACL，以及 Compose 內 Redis／MinIO 的交付選型。這些完成前可以持續開發與單機驗證，但不應對外宣稱為已完成全球離線、多租戶、可任意再散布的正式服務。
