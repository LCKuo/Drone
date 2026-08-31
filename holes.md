# GIS Server 與模擬產品：第三方元件及商業交付風險評估

> 盤點日期：2026-08-31
> 適用範圍：Interact Vision GIS Server、Unity 模擬 Client、DroneBridge 與標準部署設定
> 文件目的：說明各項第三方依賴實際用於何處、移除後的影響，以及建議的產品處理方式。本文是產品與技術決策依據，不取代個案法律意見。

## 一、管理決策摘要

目前風險並不代表整套系統不能販售。多數項目可透過「移除示範資產」、「更換資料來源」、「預設停用」或「補齊授權紀錄」處理。

### 1. 可直接從標準交付版移除

| 項目 | 移除範圍 | 對主要功能的影響 | 建議 |
|---|---|---|---|
| Esri 預覽地圖包 | 兩組既有 Taipei 101 預覽／測試地圖 | 只會移除示範存檔；不影響 GIS 產生器、World Selector 或一般地圖下載 | 從正式安裝包及預設地圖清單移除 |
| DJI Inspire 2 第三方模型 | 一個選配飛行器外觀與相關 QA 設定 | 不影響目前選用的飛行器、飛行物理或 SITL | 在來源與再散布權證明補齊前移除 |
| Cloudflare Tunnel | 公開網域的外部連線方式 | LAN、內網及本機部署仍可使用；只是不再由該 Tunnel 提供外網入口 | 不列為產品必要元件，改為客戶選配的網路架構 |
| MinIO（單機標準方案） | Docker Compose 中的物件儲存服務 | 單一 GIS Server 可改用本機儲存；多節點方案才需要共享物件儲存 | 從單機標準部署移除，但保留 S3 相容儲存介面 |

### 2. 不宜直接刪除，應更換或預設停用

| 項目 | 原因 | 建議處理 |
|---|---|---|
| NLSC 正射影像下載／烘焙 | 用於衛星真實地形；直接刪除會失去台灣正射影像，但目前權限宣告過度寬鬆 | 預設禁止下載、永久快取、衍生及再散布；取得個案授權後才開啟，或改用客戶自有／商業影像 |
| OSM 官方公共圖磚 | 只供 World Selector 底圖，但現行預抓與持久快取不符合官方公共服務政策 | 保留 World Selector，改用自建圖磚或允許快取的商業服務 |
| Redis 7.4 | 用於工作佇列、分散式鎖及限流；直接刪除會降低多工作程序能力 | 單機可使用 inline 模式；正式擴充方案改用 Valkey |
| AWS Terrarium 高程 | 用於地形高程，不是單純視覺裝飾 | 優先改用本機授權 DEM；若保留，補齊底層資料來源署名 |
| Mapterhorn 地形服務 | 只影響 Web 3D 地圖的地形起伏 | 驗證條款與完整署名；不符合交付條件時改接自有 DEM 地形服務 |
| 3DMR 外部模型 | 增加少量地標模型細節，並非核心建築來源 | 標準版預設停用；需要時依單一資產授權快照啟用 |

### 3. 不建議移除

| 項目 | 不建議移除的原因 |
|---|---|
| OpenStreetMap 資料（ODbL） | 是道路、建築輪廓、標籤及真實世界地圖生成的核心資料；移除後 GIS High、衛星建築及多數地圖產生能力會大幅缺失 |
| PostGIS | 是 GIS 空間查詢與資料庫核心；替換成本高，且獨立部署及正確履行 GPL 告知義務即可使用 |
| pymavlink | 是 PX4 SITL／MAVLink 通訊核心；移除後模擬器無法正常控制與接收遙測 |
| 授權 manifest、第三方清單及 SBOM | 這些是交付合規機制，不是多餘功能；應修正並補齊，而不是刪除 |
| 地圖精度及模擬用途告知 | 用於界定產品用途與安全邊界；移除會增加誤用及責任風險 |

## 二、各項風險的實際使用位置與移除影響

| ID | 項目與目前狀態 | 實際用在何處 | 主要程式／資料位置 | 完全移除後的結果 | 建議決策 |
|---|---|---|---|---|---|
| R01 | **NLSC 正射影像**；GIS 預設 provider，下載權限目前被程式宣告為可用 | GIS「衛星真實地形」產生、影像貼入 GLB、Unity 可下載地圖包 | `app/config.py`、`app/services/imagery.py`、`app/services/generator.py` | 台灣正射影像地表無法產生；OSM 建築、道路、地形及既有地圖仍可使用 | **保留介面、預設停用下載**；改接客戶授權影像或取得書面授權 |
| R02 | **Esri 預覽地圖包**；現有測試資料 | Unity 內兩組 Taipei 101 儲存地圖／預覽 | `DroneBridge/config/map_assets/gis-28c5a624ae37/`、`gis-59b90a7d79af/`、`map_profiles.json` | 只少兩組示範地圖，不影響地圖生成與下載 | **直接從正式版移除**；preview-only 資產不得標示可再散布 |
| R03 | **OSM 官方公共 raster tile**；World Selector 預設 endpoint，含預抓與 7 日快取 | Unity World Selector 的範圍選擇背景圖；不是最終地形或建築來源 | `Assets/DroneTraining/Runtime/DroneWorldMapSelector.cs` | World Selector 失去背景底圖；GIS Server 生成管線仍存在 | **更換服務，不刪除 Selector**；使用自建圖磚或合約允許快取的 provider |
| R04 | **授權 manifest 權限硬編碼**；目前會把部分影像標成可修改、可再散布 | 每一個產生地圖包的 `licenses.json` | `app/services/generator.py`、`app/services/imagery.py` | 若刪除 manifest，客戶更無法辨識來源與權限 | **修正，不能移除**；所有權限由 provider 設定產生，未知時一律為 `false` |
| R05 | **OSM 向量資料／ODbL**；核心資料來源 | 建築輪廓、道路、標籤、3D 擠出、衛星模式建築與 Arnis 地圖資料 | `app/services/osm.py`、`app/services/generator.py` | 多數真實世界 GIS 地圖功能失去資料來源 | **保留**；完成署名、來源版本／日期與衍生資料庫流程 |
| R06 | **AWS Terrarium／Tilezen 高程**；hybrid DEM 的網路 fallback | GIS 地形高程、Unity 地形網格 | `app/config.py`、`app/services/elevation.py` | 未配置本機 HGT 時會退回平坦地形 | **優先改用本機授權 DEM**；保留時補齊 Tilezen 及底層資料集署名 |
| R07 | **Mapterhorn**；Web 3D 地圖的外部 terrain source | 後台互動式 3D 地圖的地形起伏 | `app/static/js/map-explorer.js` | Web 地圖仍可顯示影像與建築，但地形可能變平；Unity 下載地圖不受影響 | **選配或替換**；確認條款並顯示完整 attribution |
| R08 | **3DMR 模型**；外部模型功能預設可用，但衛星模式不使用 | OSM 標記存在 3DMR ID 時，補充高細節地標／物件 | `app/config.py`、`app/services/assets.py`、`app/services/generator.py` | 少數模型改用程序化建築或一般 fallback，核心功能仍可用 | **標準版預設停用**；只接受可商用及可再散布的資產，保存授權快照 |
| R09 | **公司資產、Kenney、Poly Haven、CC BY 模型**；已納入本地資產庫 | GIS／Unity 的建築、植栽、道路物件、材質與飛行器外觀 | 資產 manifests、`THIRD_PARTY_NOTICES.md`、Unity StreamingAssets attribution | 視覺品質與模型種類下降 | **保留**；CC0 留來源紀錄，CC BY 顯示作者、作品、網址、授權及修改狀態 |
| R10 | **DJI Inspire 2 + Zenmuse X5S 模型**；來源及授權欄位缺失，且不是目前選用飛行器 | Unity 選配飛行器外觀與動畫 QA | `DroneBridge/config/imported_models/custom-dji-inspire2-x5s-animated-v1/`、`airframe_profiles.json`、`TrainerAnimatedModelQa.cs` | 少一個選配外觀；飛行物理、SITL 及目前 Parrot profile 不受影響 | **直接從正式版移除**，取得完整來源與再散布權後再評估恢復 |
| R11 | **Parrot CC BY 外觀與品牌名稱**；部分模型有明確 CC BY 4.0 來源 | Unity 飛行器外觀及 profile 清單 | `Assets/StreamingAssets/DroneTraining/Aircraft/Parrot/ATTRIBUTION.md`、`airframe_profiles.json` | 飛行器選項減少；物理可改用通用外觀繼續運作 | **模型可保留**並完成署名；若不需要品牌辨識，改用通用命名與公司自製外觀 |
| R12 | **Arnis Runtime Fork（Apache-2.0）**；體素化地圖模式 | Unity World Selector 的開放地理資料擷取與體素化地圖產生 | Arnis Runtime Fork、本機地圖建立流程、相關 profile | 體素化地圖模式消失；GIS High／衛星模式仍可運作 | **保留**；安裝包附 Apache-2.0 License、版權與 NOTICE，並履行 OSM ODbL |
| R13 | **MinIO Community Server（AGPLv3）**；標準 Docker Compose 目前會部署 | 產生 GLB、metadata、manifest 的物件儲存及多工作節點共用 | `docker-compose.yml`、`app/services/storage.py` | 單機改用 local storage 幾乎不影響；多節點失去共享物件儲存 | **單機標準版移除 MinIO**；保留 storage abstraction，多節點改用客戶既有合規 S3 服務 |
| R14 | **Redis 7.4（RSALv2／SSPLv1）**；標準 Docker Compose 目前啟用 | 工作佇列、分散式 tile lock、限流與多 worker 協調 | `docker-compose.yml`、`app/services/jobs.py`、`app/services/rate_limit.py` | 單程序可改 inline；多 worker、分散式鎖及擴充能力下降 | **替換為 Valkey**；小型單機可用 inline，保留佇列架構 |
| R15 | **PostGIS（GPL）**；GIS 核心服務 | 空間資料儲存、區域查詢、OSM 幾何與工作資料 | PostgreSQL/PostGIS 容器及 GIS repository | 真實世界 GIS 查詢與生成管線需大幅重寫 | **保留**；作為獨立資料庫服務，附授權與 source link，不將其程式碼併入專有程式 |
| R16 | **pymavlink（LGPL）**；模擬核心依賴 | DroneBridge 與 PX4 SITL 的 MAVLink 控制、狀態與遙測 | `DroneBridge/src/drone_bridge/sitl.py`、`tools/diagnose.py` | SITL 控制與遙測中斷 | **保留**；維持獨立／動態依賴，附 LGPL、來源與修改內容 |
| R17 | **Cloudflare Tunnel**；目前是外部公開網址的部署選項，不是程式核心 | 將內網 GIS Server 暴露為公開 HTTPS 網域 | 客戶／部署端 Cloudflare 設定；GIS 程式碼未直接依賴 | 公網網址入口停止；LAN、VPN、反向代理及本機存取不受影響 | **不納入標準產品依賴**；需要外網時由部署方案另選企業網路服務 |
| R18 | **客戶匯入模型及自備圖資**；選配功能 | 匯入 FBX／OBJ 飛行器、客戶指定影像、DEM 或其他地理資料 | `DroneModelFileDialog.cs`、`DroneExternalModelImporter.cs`、local/template imagery 設定 | 無法建立客製飛行器或使用客戶自有資料 | **保留但加管控**；匯入時記錄來源、權利人、授權範圍與可否交付 |
| R19 | **第三方清單與 SBOM 不完整**；交付流程缺口 | 客戶法務審查、安裝包 notices、版本追溯與弱點處理 | `THIRD_PARTY_NOTICES.md`、Unity notices、容器與 Python／Unity 套件清單 | 移除文件只會使交付風險增加，不會減少依賴 | **補齊，不能移除**；每次 release 自動生成並保留版本快照 |
| R20 | **地圖精度、訓練用途與安全邊界**；產品及合約要求 | GIS 頁面、Unity 模擬器、使用手冊、報價／合約與驗收文件 | UI 告知、使用手冊、EULA／合約、驗收條件 | 使用者可能把訓練資料當成即時導航或安全資料 | **保留並一致化**；明示為模擬／訓練用途，不作為唯一導航或飛航安全依據 |

## 三、功能層級的移除判斷

這張表可用於評估「從系統移除」是否合理。

| 決策類型 | 項目 | 移除／更換後仍保留的能力 | 會失去的能力 |
|---|---|---|---|
| 可立即移除 | Esri 預覽地圖包 | GIS 生成、Unity 下載、World Selector、既有合法地圖 | 兩組測試預覽存檔 |
| 可立即移除 | DJI Inspire 2 模型 | SITL、飛行物理、其他飛行器 profile | 一個選配外觀與動畫 |
| 可從標準版移除 | MinIO | 單機地圖生成、下載與本機儲存 | 多節點共享物件儲存；需要其他 S3 服務替代 |
| 可從標準版移除 | Cloudflare Tunnel | 內網、本機、VPN 或其他反向代理存取 | Cloudflare 提供的公開入口 |
| 可預設停用 | 3DMR | OSM 建築、程序化模型、衛星及地形 | 少數高細節外部模型 |
| 可替換 | Mapterhorn | Web 影像、建築與 Unity 地圖 | Web 地形起伏，直到替代 terrain source 上線 |
| 必須替換來源 | OSM 公共圖磚 | World Selector 流程及 GIS 生成架構 | 更換期間的 Selector 背景圖 |
| 必須替換或授權 | NLSC 下載／烘焙 | OSM 建築、道路、DEM、非衛星模式 | 台灣正射影像地表，直到合法來源配置完成 |
| 不建議移除 | OSM 向量資料、PostGIS | — | 真實世界 GIS 的主要生成與查詢能力 |
| 不建議移除 | pymavlink | — | PX4 SITL 控制與遙測 |
| 不能以刪除處理 | manifest、notices、SBOM、安全告知 | — | 合規證據、來源追溯與用途邊界 |

## 四、正式交付前的必要修正

### Release blocker

1. 將 NLSC provider 的下載、永久快取、衍生及再散布權限改為契約設定，預設全部為 `false`。
2. 從交付版刪除兩組 Esri preview-only 地圖包及其預設 profile。
3. 從交付版刪除 DJI Inspire 2 模型、profile 與僅為該模型使用的 QA 資料，除非能提出可驗證的來源與再散布授權。
4. 停止對 `tile.openstreetmap.org` 進行預抓與持久離線快取，改接自建或正式授權圖磚服務。
5. 讓 `licenses.json` 完全依資料來源的實際政策產生；未知權利不得自動推定為可再散布或可修改。

### 產品化修正

1. 單機 GIS Server 的標準 Compose 改用 local storage；多節點方案再配置客戶合規的 S3 相容儲存。
2. Redis 7.4 改為 Valkey；單機輕量部署可採 inline queue。
3. 交付地圖優先使用本機授權 DEM；若使用 Terrarium／Tilezen，補齊實際底層資料集 attribution。
4. 對 Mapterhorn 與 3DMR 設定明確的啟用條件、失效 fallback 及 attribution。
5. 生成完整 `THIRD_PARTY_NOTICES` 與 SBOM，涵蓋容器、Python、Unity、資料、模型、材質及字型。
6. 客戶匯入資產時保存來源、權利人、授權版本、允許用途、再散布權及檔案雜湊。

## 五、交付檢查表

- [ ] 交付包不含 Esri preview-only 地圖資產。
- [ ] 交付包不含授權來源不明的 DJI 模型。
- [ ] NLSC 或其他影像 provider 未經明確授權時，不允許離線下載、永久快取或衍生交付。
- [ ] World Selector 不以 OSM 官方公共圖磚提供預抓／離線功能。
- [ ] 每個地圖包的 `licenses.json` 與實際 provider 權限一致。
- [ ] OSM、DEM、影像、模型及材質 attribution 可在 Web、Unity 與交付文件中查閱。
- [ ] 單機版未包含不需要的 MinIO；需要多節點時已指定合規的共享儲存方案。
- [ ] Redis 已改用 Valkey，或該部署採用不需要分散式佇列的 inline 模式。
- [ ] 已產生當次版本的第三方清單、SBOM、容器清單與授權快照。
- [ ] 安裝包不含 `.env`、私鑰、預設密碼、Cloudflare token、供應商 API key 或原始高解析受限資料。
- [ ] 使用手冊與 UI 明確標示模擬／訓練用途及資料精度限制。

## 六、官方參考資料

- [國土測繪圖資服務雲使用規定](https://maps.nlsc.gov.tw/pro/use_clause.jsp)
- [OpenStreetMap Tile Usage Policy](https://operations.osmfoundation.org/policies/tiles/)
- [OpenStreetMap Copyright and License](https://www.openstreetmap.org/copyright)
- [Esri Master Agreement and Product-Specific Terms](https://www.esri.com/en-us/legal/terms/master-agreement)
- [Tilezen／Joerd Attribution](https://github.com/tilezen/joerd/blob/master/docs/attribution.md)
- [Mapterhorn Attribution](https://mapterhorn.com/attribution/)
- [MinIO License](https://github.com/minio/minio/blob/master/LICENSE)
- [Redis Licenses](https://redis.io/legal/licenses/)
- [Arnis License](https://github.com/louis-e/arnis/blob/main/LICENSE)
- [GNU LGPL](https://www.gnu.org/licenses/lgpl-3.0.html)

## 七、整體判定

| 問題 | 判定 |
|---|---|
| GIS Server 與模擬 Client 是否可以作為商業產品？ | 可以；需先完成 Release blocker |
| 是否需要因第三方風險刪除整套真實世界 GIS？ | 不需要；核心 OSM／PostGIS 管線可保留 |
| 哪些內容最適合立即刪除？ | Esri 預覽地圖包、來源不明的 DJI Inspire 2 模型 |
| 哪些內容適合從標準版移除？ | 單機部署中的 MinIO、作為產品必要條件的 Cloudflare Tunnel |
| 哪些內容應更換而不是刪除？ | NLSC 下載來源、OSM 公共圖磚 endpoint、Redis 7.4、未完成 attribution 的 DEM／Web terrain source |
| 哪些內容刪除後會破壞核心功能？ | OSM 向量資料、PostGIS、pymavlink，以及授權／安全告知機制 |
