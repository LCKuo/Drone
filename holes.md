# GIS Server 與模擬產品商業銷售風險避坑指南

> 盤點日期：2026-08-31  
> 對象：英特艾互動科技有限公司作為軟體開發商、授權人、系統整合商及售後維護商  
> 範圍：GIS Server、Web 後台、地圖生成、Unity／模擬 EXE 下載地圖、3D 模型、部署映像與客戶合約  
> 本文件為工程與商業風險盤點，不取代律師、會計師或資料供應機關的正式意見。

## 一、先講結論

你可以收費販售自己的 GIS Server、管理後台、地圖生成器、模擬介接、安裝服務與維護服務。FastAPI、PostgreSQL、PostGIS、MapLibre、PX4、Gazebo 等元件不會因為你收費就禁止銷售。

真正的坑是：**你不是只賣自己的程式，現行輸出還可能把第三方影像、圖磚、資料與模型複製進客戶可下載的 GLB／地圖包。**

目前版本不應原樣商用出貨。下列四項屬於 Stop-Ship：

1. NLSC 公開 WMTS 被程式標示成可永久快取、可製作衍生資產及可再散布，但公開條款沒有明確授予這些權利。
2. 兩個既有 Esri 預覽地圖包明明是 `preview-only-web-display`，`licenses.json` 卻標成 `allowRedistribution: true`、`allowModification: true`。
3. DJI Inspire 2 模型沒有來源、作者、購買證明與可轉散布授權。
4. Unity World Selector 直接使用 `tile.openstreetmap.org`，而功能包含預抓；OSM 公共圖磚政策明確禁止 prefetch 和 offline map。

只要把這些問題隔離、建立授權閘門並補合約，主要軟體授權成本仍可維持 NT$0，而且你可以正常向客戶收軟體授權與服務費。

## 二、風險總表

| 等級 | 項目 | 現況 | 可能後果 | 出貨判定 |
|---|---|---|---|---|
| P0 | NLSC 影像離線烘焙／交付 | 程式直接宣告可下載、快取、衍生及再散布 | 停止服務、補授權費、下架、客戶求償 | **禁止原樣出貨** |
| P0 | Esri 預覽地圖包 | preview-only 卻標示可再散布 | 侵權通知、補商業授權、客戶驗收失敗 | **禁止出貨** |
| P0 | DJI Inspire 2 模型 | 無來源及授權證明 | 原作者／商城追償、模型下架、客戶索賠 | **禁止出貨** |
| P0 | OSM 公共圖磚預抓 | World Selector 使用官方公共圖磚 | IP 被封鎖、客戶功能中斷、沒有 SLA | **正式版必修** |
| P1 | 授權 manifest 判定錯誤 | `allowRedistribution` 由程式硬編碼 | 自動把不可交付資料放進客戶包 | **必修** |
| P1 | MinIO AGPLv3 | 正式 Docker Compose 會部署 MinIO | 客戶法務拒絕、需履行 AGPL 義務 | 建議換 local storage |
| P1 | Redis 7.4 RSAL／SSPL | 非傳統 OSI 開源授權 | 採購審查、使用範圍爭議 | 建議換 Valkey |
| P1 | OSM ODbL | 有署名，但衍生資料庫流程未文件化 | attribution／衍生資料庫義務不完整 | 必修文件與流程 |
| P1 | 高程來源 attribution | 只標示 AWS Terrarium | 底層資料集署名不完整 | 必修 |
| P1 | `THIRD_PARTY_NOTICES.md` 不完整 | 未涵蓋全部套件、容器、資料與模型 | 客戶法務拒收、違反告知義務 | 必修 |
| P1 | 地圖精度及飛安責任 | 若合約宣稱可作導航／真實飛行 | 事故、重大賠償與保險拒賠 | 合約必修 |
| P2 | Cloudflare 免費 Tunnel | 沒有企業 SLA | 中斷時你仍對客戶負 SLA | 合約與架構處理 |
| P2 | 第三方品牌／外觀 | DJI、Parrot 等名稱及產品外觀 | 商標、背書或混淆爭議 | 加免責並避免冒稱 |
| P2 | 客戶上傳資料 | 未明確要求客戶保證權利 | 客戶侵權資料由你代為處理／散布 | 合約必修 |

## 三、NLSC：目前最大的商業坑

### 3.1 為什麼公開服務不等於可包進商品

NLSC 公開條款允許合法公開顯示並要求附上來源，但明確禁止大量下載；條款也指出，只開放給付費且需登入才能看到地圖的網站不在一般免費使用範圍。你的商業產品可能同時符合：

- 客戶付費取得系統。
- 管理後台需要登入。
- 使用者框選區域後批次下載圖磚。
- 影像被烘焙進 GLB。
- GLB 被模擬 EXE 下載並離線使用。
- 地圖包可能被備份、複製到多台 Client 或交給第三方。

因此，不能只憑公開 WMTS 可連線就宣稱影像可永久商用轉散布。[NLSC 使用條款](https://maps.nlsc.gov.tw/pro/use_clause.jsp)

### 3.2 現行程式的實際風險

目前 `app/services/imagery.py` 對 NLSC provider 設定：

```text
downloadable = true
persistent_cache = true
derivative_assets = true
```

輸出 metadata 還寫入：

```text
offlinePolicy = persistent_allowed
```

`app/services/generator.py` 對所有 imagery 又固定輸出：

```text
allowRedistribution = true
allowModification = true
```

這些是程式自己的宣告，不是 NLSC 正式授權證明。商用前必須改成由每個 provider 的實際契約決定，預設值應為 `false`。

### 3.3 安全的商業方式

建議採用以下其中一種：

1. 公司按專案向 NLSC 購買加值使用資料，費用獨立列在報價單並保存購買證明。
2. 客戶自行購買或提供圖資，公司只處理客戶已合法取得的檔案。
3. 與 NLSC 取得明確書面回覆，確認可下載、快取、製成 3D 地圖資產、交付客戶及允許的裝置數。
4. 預設產品只提供 OSM 建築／道路與公司自有地面材質；衛星影像是另購功能。

報價單不要只寫「GIS 地圖」。應拆成：

```text
GIS 軟體授權費
地圖生成服務費
第三方圖資採購／授權費（依區域另計）
圖資更新費
```

## 四、Esri：預覽可以，不能偷偷變成交付資產

Esri 網站服務免費可存取，不代表資料進入公有領域；一般免費條款不授予商業重製、衍生與再散布權。商業使用應使用適用的付費服務與客戶合約保護條款。[Esri Terms](https://www.esri.com/en-us/legal/terms/web-site-service)

目前已存在：

```text
D:\DroneTrainingSystem\DroneBridge\config\map_assets\gis-28c5a624ae37\licenses.json
D:\DroneTrainingSystem\DroneBridge\config\map_assets\gis-59b90a7d79af\licenses.json
```

其中 Esri 項目同時出現：

```json
{
  "license": "preview-only-web-display",
  "allowRedistribution": true,
  "allowModification": true
}
```

這是矛盾且危險的授權資料。必須：

- 把兩個預覽包標示為 development-only，排除於安裝包、備份種子資料與客戶 Demo。
- 把 preview-only 資料的 `allowRedistribution`、`allowModification` 改成 `false`。
- 生產環境若沒有正式商業契約，禁止建立任何 Esri 離線包。
- CI／發布程序掃描 `preview-only`、`esri` 等關鍵字，命中即阻止 Release。

## 五、OpenStreetMap：可以商用，但不要濫用公共伺服器

OSM 資料依 ODbL 可免費商用。你可以把 OSM 道路、建築高度與 footprint 轉成可視化 3D Produced Work，但需正確署名；如果建立並公開使用經修改的衍生資料庫，還需處理 ODbL 對該資料庫的義務。這不會自動要求你公開整個 GIS Server 或 Unity EXE。

真正會馬上出問題的是公共圖磚伺服器。OSM 官方政策明確禁止：

- Bulk download。
- Prefetch。
- 預先建立城市／區域圖磚包。
- 離線使用及散布 tile archive。

而 `D:\DroneTrainingSystem\Sitl_Drone\Assets\DroneTraining\Runtime\DroneWorldMapSelector.cs` 目前預設：

```text
https://tile.openstreetmap.org
```

正式產品應改成：

- 下載合法的 OSM PBF 資料至自家 PostGIS。
- 由 GIS Server 自建 raster／vector tile。
- World Selector 只連自家 GIS API。
- UI 永遠顯示 `© OpenStreetMap contributors`，不可藏在深層設定。
- 保留使用資料版本、下載日期與衍生處理紀錄。

否則一旦客戶批次框選，服務可能在沒有通知的情況下封鎖。[OSM Tile Usage Policy](https://operations.osmfoundation.org/policies/tiles/)

## 六、3D 模型、材質與品牌

### 6.1 可以賣的資產

| 資產 | 商用狀態 | 要求 |
|---|---|---|
| 公司自行建立的模型、材質 | 可 | 保存原始檔、作者／外包契約與權利讓與證明 |
| Poly Haven／Kenney CC0 | 可 | 建議仍在第三方清單列出來源 |
| CC BY 模型 | 可 | 顯示作者、作品名、來源、授權及是否修改 |
| 3DMR CC0／CC BY | 可 | 下載時固定保存 license snapshot，不只保存網址 |

### 6.2 DJI Inspire 2 模型

目前只有處理後模型與專案說明，找不到原始授權證明。模型出現在專案目錄，不等於公司擁有再散布權。

出貨前採三選一：

1. 找出原商城訂單與授權全文，確認允許納入遊戲／應用程式並交付客戶。
2. 取得作者書面商用轉散布授權。
3. 從 Release 移除並以公司原創模型取代。

不要只把檔名改掉或把 FBX 包進 AssetBundle；這不會改變著作權狀態。

### 6.3 商標與產品外觀

即使模型著作權授權合法，DJI、Parrot 與機型名稱仍可能涉及商標或外觀識別。產品頁與合約應：

- 只作必要的相容性及訓練用途描述。
- 不把第三方 Logo 當作自己的品牌素材。
- 不宣稱經原廠認證、合作或背書，除非真的有書面證明。
- 加註所有商標屬於各自權利人。

## 七、開源軟體：可以收費，但要履行義務

### 7.1 一般可安全商用

FastAPI、Nginx、PostgreSQL、MapLibre、PX4、Gazebo、QGroundControl、Arnis、ufbx 及常見 MIT／BSD／Apache 套件，可以被包含在收費產品中。

你需要：

- 安裝包包含所有必要 License／NOTICE。
- `THIRD_PARTY_NOTICES.md` 列出實際版本、來源、授權和用途。
- 不刪除原作者著作權聲明。
- Apache-2.0 元件如有 NOTICE，連同 NOTICE 一起交付。
- 不使用開源專案名稱暗示官方背書。

### 7.2 PostGIS GPL

用 PostGIS 當資料庫，不會使你的 GIS 應用程式自動變成 GPL。若公司修改 PostGIS 本體並對外散布，才要處理對應 GPL 原始碼義務。[PostGIS GPL FAQ](https://postgis.net/documentation/faq/gpl-license/)

### 7.3 pymavlink LGPL

pymavlink 是 LGPL。維持為獨立 Python 程序或未修改的動態依賴最乾淨。交付時應包含：

- LGPL 授權全文。
- 使用版本與來源網址。
- 對應原始碼或合規的取得方式。
- 如果修改 pymavlink，本公司修改內容的對應原始碼。

這不代表整個 Unity EXE 必須開源。

### 7.4 MinIO AGPL

MinIO Community Server 採 AGPLv3。[MinIO Repository](https://github.com/minio/minio)

AGPL 不會因為 MinIO 是獨立服務就自動感染所有專有程式，但你仍需處理 MinIO 本身的散布、修改與網路使用義務。部分客戶法務會直接拒絕 AGPL。

目前產品已支援 local storage。單機正式版建議：

```text
STORAGE_PROVIDER=local
```

並從交付 Compose 移除 MinIO。需要 S3 叢集時，再選客戶已有的合規物件儲存或正式商業方案。

### 7.5 Redis 7.4

Redis 7.4 採 RSALv2／SSPLv1；你的產品不是 Redis 競爭性資料庫服務時通常不需付費，但授權不是傳統 OSI 開源，會增加企業法務審查成本。[Redis Licenses](https://redis.io/legal/licenses/)

建議改用 Valkey，或在小型單機版使用 inline queue。

## 八、你應該怎麼賣

### 8.1 商品拆分

建議報價拆成：

| 項目 | 公司實際販售內容 |
|---|---|
| GIS Server 軟體授權 | 公司程式、API、管理後台、權限、工作佇列與更新 |
| 模擬器介接授權 | World Selector、下載、解包、載入、LOD、碰撞與 attribution UI |
| 地圖建置服務 | 依客戶指定範圍生成、驗證及最佳化地圖 |
| 第三方圖資費 | NLSC、商業影像或客戶指定資料；獨立報價、實報實銷 |
| 模型／材質包 | 僅公司原創或已取得再散布權的資產 |
| 安裝部署 | Server、Client、網路、備份與資安設定 |
| 年度維護 | Bug 修正、相容性更新、備份檢查與授權資料更新 |
| SLA | 服務時間、回應時間、修復目標與排除事項 |

不要用一句「包含全球衛星 3D 地圖」包辦全部。全球影像通常由不同供應商、不同區域、不同離線條件組成，無法永久保證免費。

### 8.2 推薦授權模式

可以採用：

- Server：每一正式部署站點／每台 Server 授權。
- Client：每台模擬 Client 或同時連線數授權。
- 維護：每年軟體授權費的固定百分比，或固定年度費。
- 第三方圖資：依專案及實際範圍另計，不包含在永久軟體授權中。
- 客戶自備資料：公司收處理服務費，但客戶保證其擁有合法權利。

你的 EULA 只能控制自己的程式，不能把 CC BY、ODbL 或其他第三方資料重新宣稱成公司專有且禁止一切使用。第三方元件必須保留其原始授權。

## 九、客戶合約一定要寫的條款

### 9.1 授權標的與範圍

- 授權的是 Server、Client、站點、裝置或同時連線數，要寫清楚。
- 是否允許備援機、測試機、離線機與災難復原副本。
- 客戶不得拆出第三方影像、圖磚或模型另行轉售。
- 公司程式的反編譯、重製與轉授權限制，必須排除依法不得限制的情況及第三方開源授權。

### 9.2 第三方資料

- 第三方圖資不屬於公司智慧財產。
- 使用範圍受各資料來源條款與採購契約限制。
- 第三方調價、停止服務、改版、撤回 API 或修改授權時，可以替換資料源或另行報價。
- 未經授權不得快取、抽取、轉散布或交給關係企業。

### 9.3 客戶提供資料

要求客戶保證：

- 有權提供影像、CAD、BIM、航測、模型與個人資料。
- 公司依指示處理、轉檔與部署不侵害第三人權利。
- 若客戶資料侵權，由客戶負責處理並補償公司因此產生的損失。

### 9.4 地圖精度與飛安

必須明確寫明：

- 系統主要用途是模擬、訓練、展示與規劃。
- OSM 建築高度、道路、影像與 DEM 可能過時、缺漏或有誤差。
- 不得作為唯一的真實導航、避障、飛行許可、測量、救災或生命安全依據。
- 真實操作必須使用經核准的航圖、感測器、飛控限制及人工判斷。

否則一次事故的損害可能遠高於整份軟體合約金額。

### 9.5 保固、SLA 與責任上限

- 免費 OSM、NLSC、Cloudflare 或其他第三方服務中斷，不可無條件算成公司違反 SLA。
- 約定支援時間、回應時間和修復目標，不要只寫「永久維護」或「隨時修好」。
- 區分軟體錯誤、第三方服務、客戶網路、硬體、錯誤操作與不可抗力。
- 設定責任上限，例如以該專案已付費用或一定期間維護費為上限；重大故意、法律不得排除事項另依法律。
- 排除所失利益、間接損害及客戶未備份造成的資料損失，仍需由律師依實際交易調整。

### 9.6 資安與個資

GIS 後台已有帳號、密碼、API Key、操作紀錄及對外 Tunnel，正式產品還應約定：

- 客戶與公司各自的帳號管理責任。
- 資安更新期限。
- 弱點通報及事故通知流程。
- Log 保存時間與個資用途。
- 遠端維護需取得客戶授權並留存紀錄。
- 專案終止後資料返還、刪除及備份清除方式。

## 十、發布流程要加入的技術閘門

只靠人工閱讀授權檔容易出錯，建議建立自動 Release Gate：

### 10.1 資產匯入閘門

每個 imagery、DEM、模型與材質都必須有：

```json
{
  "source": "來源網址或採購文件編號",
  "license": "授權名稱／契約編號",
  "commercialUse": true,
  "allowModification": true,
  "allowRedistribution": true,
  "allowOffline": true,
  "attribution": "需顯示的完整文字",
  "proofFile": "內部授權證明位置",
  "expiresAt": null,
  "customerScope": "允許的客戶／站點／裝置範圍"
}
```

任何欄位不明時預設 `false`，不能由程式猜測。

### 10.2 地圖包輸出閘門

- 只有 `commercialUse && allowRedistribution && allowOffline` 全部為 `true` 才能生成可下載 GLB。
- Preview provider 只能在開發環境顯示，API 不提供下載。
- 每個地圖包附不可修改的 `licenses.json`、資料版本、生成時間及 attribution。
- 客戶下載頁先顯示資料來源及使用限制。
- 地圖包加入唯一 ID、客戶 ID 及授權 scope，方便稽核但不得侵犯個資。

### 10.3 CI／安裝包掃描

至少阻擋下列關鍵字或狀況：

```text
preview-only
evaluation-only
non-commercial
NC
unknown-license
allowRedistribution = false
allowOffline = false
無 proofFile
```

另外掃描安裝包是否意外含有 `.env`、私鑰、管理密碼、Cloudflare token、供應商 API key、原始高解析影像或未授權 FBX。

## 十一、必備交付文件

每一版正式產品至少包含：

1. 公司 EULA／軟體授權合約。
2. `THIRD_PARTY_NOTICES.md`。
3. Open Source Software Bill of Materials（SBOM），包含版本與授權。
4. 地圖資料來源與 attribution 清單。
5. 每個離線地圖包的 `licenses.json`。
6. 模型及材質來源清單。
7. 客戶採購的圖資授權或書面同意影本。
8. 安裝、備份、還原、升級與移除手冊。
9. 支援範圍、EOL 與資安更新政策。
10. 地圖精度、模擬用途及禁止真實導航的聲明。

SBOM 建議採 SPDX 或 CycloneDX，讓客戶資安／法務可以直接掃描，而不是只交一段模糊文字。

## 十二、簽約前與出貨前 Checklist

### 簽約前

- [ ] 報價已把軟體費、圖資費、服務費與維護費分開。
- [ ] 客戶選定區域及資料來源。
- [ ] 已確認圖資是公司採購或客戶合法提供。
- [ ] 合約未承諾永久免費第三方影像。
- [ ] 合約未把免費第三方服務當作無條件 SLA。
- [ ] 已限制用途，不作真實導航、測量及生命安全唯一依據。
- [ ] 已約定客戶資料的權利保證。
- [ ] 已設定責任上限、維護期限與 EOL。

### 出貨前

- [ ] NLSC 圖資有對應書面授權／採購證明。
- [ ] 安裝包完全不含 Esri preview-only 資產。
- [ ] DJI Inspire 2 模型已移除或授權證明完整。
- [ ] World Selector 不再直接預抓 OSM 公共圖磚。
- [ ] MinIO 已移除或完成 AGPL／商業授權審查。
- [ ] Redis 已換 Valkey，或已完成 RSAL／SSPL 審查。
- [ ] 所有可下載地圖包均通過 redistribution／offline gate。
- [ ] `THIRD_PARTY_NOTICES.md`、SBOM、模型清單與地圖 attribution 完整。
- [ ] 沒有預設密碼、測試 API Key、Cloudflare token 或私鑰。
- [ ] 已完成備份還原與升級回復測試。
- [ ] Web 與 EXE 內 attribution 可見且不藏在深層介面。

## 十三、建議立即執行順序

### 第一批：出貨封鎖

1. 將 NLSC provider 的 `downloadable`、`persistent_cache`、`derivative_assets` 與 redistribution 改成由契約設定，預設 `false`。
2. 修正兩個 Esri preview 地圖包的錯誤授權旗標，並從所有 Release 排除。
3. 移除未證明授權的 DJI Inspire 2 模型。
4. World Selector 改接自家 GIS 圖磚，不再使用公共 OSM prefetch。

### 第二批：產品化

5. 建立 provider／asset 授權資料庫及自動 Release Gate。
6. 補完整 `THIRD_PARTY_NOTICES.md` 與 SBOM。
7. 正式版改用 local storage＋Valkey。
8. Web、EXE 與下載頁統一顯示 attribution 和資料限制。

### 第三批：商務文件

9. 請台灣律師依本清單製作 EULA、維護條款、資料條款與責任限制。
10. 建立圖資採購、授權證明、客戶 scope 與到期日的內部台帳。
11. 報價模板拆出第三方圖資費與年度維護費。
12. 建立每一版產品的授權封存包，以應付客戶稽核或第三方通知。

## 十四、最終判定

| 問題 | 判定 |
|---|---|
| 可以販售自己的 GIS Server 嗎？ | 可以 |
| 可以販售模擬 EXE 介接與地圖載入功能嗎？ | 可以 |
| 開源元件會讓整套產品不能收費嗎？ | 不會 |
| 現在可以把 NLSC 公開 WMTS 影像直接烘焙後賣給客戶嗎？ | **不應；需書面授權或採購加值資料** |
| 現在可以交付 Esri 預覽地圖包嗎？ | **不可以** |
| 現在可以交付 DJI Inspire 2 模型嗎？ | **授權證明補齊前不可以** |
| 可以讓客戶直接大量下載 OSM 官方圖磚嗎？ | **不可以；應自建圖磚** |
| 修正上述問題後，軟體授權成本能維持 NT$0 嗎？ | 大致可以；圖資、Windows、SLA 與商業服務另計 |

最安全的商業定位是：

> 公司販售 GIS 軟體、地圖處理能力、模擬整合、部署與維護；第三方影像與資料依客戶、範圍、用途及交付形式另行取得授權與報價。任何未明確允許商用、離線及再散布的資料，預設不進入客戶安裝包。
