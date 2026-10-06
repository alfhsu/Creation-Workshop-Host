# Creation Workshop Host (CWH)

Creation Workshop Host 是一套以 Java 撰寫的 **DLP / SLA 光固化（樹脂）3D 印表機列印主機**。
它通常安裝在 Raspberry Pi（也支援 Windows / 一般 Linux）上，透過 **序列埠（G-code）** 控制印表機的馬達，
並透過 **投影機 / 顯示器** 逐層投影切片影像。使用者只要用瀏覽器連到主機，就能上傳檔案、管理印表機與監看列印進度。

- 舊版介面截圖：[host/cwh.png](host/cwh.png)
- 新版介面截圖：[host/cwhNew.png](host/cwhNew.png)

---

## 功能特色

1. 直接列印 STL 檔（內建切片器），也可從 Thingiverse 或任意網址載入 STL。
2. 列印由 Creation Workshop 匯出的 Zip / CWS 檔案。
3. 相容 Creation Workshop 的 XML 印表機設定檔（`.machine`、`.slicing`）。
4. 單一主機可管理多台印表機。
5. 自訂印表機遮罩（mask overlay）。
6. TLS 加密與 Basic 認證。
7. 設定檔中可使用 [FreeMarker](http://freemarker.org/) 樣板語法。
8. RESTful API，方便開發者整合。
9. 錄製與回放列印過程影片（Raspberry Pi 相機）。
10. 可從 GUI 直接執行自訂 G-code。
11. 外掛式通知框架（WebSocket、Email）。
12. JavaScript 計算器：計算漸層、曝光時間、抬升速度與距離。
13. 透過 WebSocket 推送印表機與列印工作事件。
14. 自動更新。
15. 透過 UPnP 在區域網路中自動探索印表機（[browseprinter.sh](host/bin/browseprinter.sh)），免設定網路。
16. 可使用模擬序列埠與模擬顯示器建立印表機設定，方便無硬體開發測試。

---

## 檔案結構

```
Creation-Workshop-Host/
├── README.md
├── LICENSE.md                  # CC BY-SA 4.0 授權
└── host/                       # 主程式所有內容
    ├── BuildCWH.xml            # Ant 打包腳本（產生 cwh / cwhClient / cwhTestKit zip）
    ├── build.number            # Ant 自動遞增的版本號
    ├── cwh-0.xxx.zip           # 預先打包好的伺服器發行版
    ├── cwhClient-0.xxx.zip     # 用來自動尋找區網內印表機的用戶端
    ├── cwhTestKit-0.xxx.zip    # 硬體相容性測試套件
    ├── installnotes.txt        # Raspberry Pi 安裝筆記
    ├── networkinstall.sh       # 網路安裝輔助腳本
    ├── web.keystore            # 啟用 SSL 時使用的範例 keystore
    ├── bin/                    # 啟動 / 停止 / 工具腳本
    │   ├── start.sh            # Linux / Pi：下載安裝最新版並啟動（也負責安裝 Java）
    │   ├── start.bat           # Windows 啟動
    │   ├── stop.sh             # 停止服務
    │   ├── startdev.sh / debug.sh   # 開發 / 除錯模式啟動
    │   ├── downgrade.sh        # 降版
    │   ├── cwhservice          # Linux init.d 服務腳本
    │   ├── browseprinter.sh/.bat    # 在區網中找出 CWH 印表機並開啟瀏覽器
    │   ├── slicebrowser.bat    # 切片結果檢視工具
    │   └── testKit.sh / testKitDev.sh  # 執行硬體相容性測試
    ├── install/
    │   └── raspi-config-cwhost.sh   # Raspberry Pi 系統設定腳本
    ├── os/                     # 各平台原生函式庫（RXTX 序列埠等）
    │   ├── Linux/{armv61,i686,ia64,x86_64}
    │   ├── win32/
    │   └── win64/
    ├── libs/                   # 第三方 JAR（Jetty、RESTEasy、Jackson、jSSC、RXTX、
    │                           #   Cling(UPnP)、FreeMarker、Guava、JavaMail、JUnit/Mockito…）
    ├── resources/              # 目前預設的 Web GUI（Bootstrap + jQuery）
    ├── resourcesnew/           # 開發中的新版 Web GUI（AngularJS + Bootcards + OpenJsCad 3D 預覽）
    ├── src/
    │   ├── config.properties   # 主設定檔（預設值）
    │   └── org/area515/
    │       ├── util/           # 共用工具：樣板引擎、郵件、IO、JSON 編解碼
    │       └── resinprinter/
    │           ├── server/       # 進入點 Main.java、HostProperties（讀設定）、REST ApplicationConfig
    │           ├── services/     # REST API：machine、printers、printJobs、files、settings、media
    │           ├── printer/      # 印表機、機器設定、切片設定檔模型與 PrinterManager
    │           ├── job/          # 列印工作管理、處理執行緒、各種列印檔處理器（CWS/Zip、STL）
    │           ├── slice/        # STL 解析與 Z 軸切片（ZSlicer、SliceBrowser）
    │           ├── stl/          # 3D 幾何資料結構（點、線、三角面）
    │           ├── gcode/        # G-code 控制（通用、Sedgwick 等）
    │           ├── serial/       # 序列埠實作（jSSC、RXTX、Console 模擬）
    │           ├── display/      # 投影 / 顯示裝置管理
    │           ├── projector/    # 以 Hex 指令控制投影機開關（Acer、ViewSonic…）
    │           ├── notification/ # 通知框架：WebSocket、Email
    │           ├── discover/     # UPnP 廣播，讓用戶端能自動找到主機
    │           ├── network/      # Linux 無線網路管理
    │           ├── inkdetection/ # 以影像偵測樹脂量
    │           ├── printphoto/   # 將圖片直接當成列印檔處理
    │           ├── minercube/    # 產生迷宮方塊（.cubemaze）列印檔
    │           ├── stream/       # 影片漸進式下載 Servlet（/video）
    │           ├── security/     # Jetty SSL / Basic Auth 設定
    │           └── client/       # 用戶端探索程式 Main
    ├── tests/                  # JUnit 測試原始碼
    └── testbin/                # 測試用編譯產物與資源
```

---

## 使用方式

### 1. 安裝最新穩定版（Raspberry Pi / Linux）

```bash
sudo wget https://github.com/area515/Creation-Workshop-Host/raw/master/host/bin/start.sh
sudo chmod 777 start.sh
sudo ./start.sh
```

`start.sh` 會檢查並安裝合適的 Java 版本、從 GitHub 下載最新的 `cwh-0.xxx.zip`、解壓後以背景方式啟動伺服器
（輸出寫入 `log.out` / `log.err`）。

若要安裝每日開發版（不穩定），在參數帶入 repo 擁有者：

```bash
sudo wget https://github.com/WesGilster/Creation-Workshop-Host/raw/master/host/bin/start.sh
sudo chmod 777 start.sh
sudo ./start.sh WesGilster
```

Raspberry Pi 完整安裝教學：
- [Wiki：Raspberry Pi 手動安裝說明](https://github.com/area515/Creation-Workshop-Host/wiki/Raspberry-Pi-Manual-Setup-Instructions)
- [影片：從頭在 Raspberry Pi 上安裝 CWH](https://www.youtube.com/watch?v=ng1Sj2ktWhU)

### 2. 在 Windows 上安裝

1. 下載 [`host/cwh-X.XX.zip`](host/)（或 WesGilster fork 中的開發版）。
2. 解壓到任意資料夾。
3. 雙擊 `start.bat`（實際執行：`java -Djava.library.path=os/win64 -cp lib/*;. org.area515.resinprinter.server.Main`）。

### 3. 開啟網頁介面

伺服器啟動後，用瀏覽器連到：

```
http://<主機 IP>:9091/
```

（啟用 SSL 時預設埠為 443。）

不知道主機 IP？下載並解壓 `host/cwhClient-X.XX.zip`，然後執行：

```bash
sudo browseprinter.sh      # Linux
browseprinter.bat          # Windows（雙擊）
```

它會透過 UPnP 在區網中找到 CWH 主機並自動開啟瀏覽器。

### 4. 列印流程

1. 在 GUI 中建立印表機：選擇顯示器（投影機）與序列埠，或使用模擬裝置測試。
2. 啟動印表機。
3. 上傳 STL、Zip/CWS 或圖片檔（也可貼上網址直接下載）。
4. 選擇檔案開始列印，可即時查看目前切片影像、暫停、停止，或調整曝光時間、Z 軸抬升距離與速度。

示範影片：[搭配 Creation Workshop 與 Zip 檔使用 CWH](https://www.youtube.com/watch?v=J3HTCkxlKcw)

---

## 設定

主設定檔為 `config.properties`。安裝後的覆寫設定會存放在 `~/3dPrinters/config.properties`；
印表機、機器與切片設定則分別存放於 `~/3dPrinters/*.printer`、`~/Machines/*.machine`、`~/Profiles/*.slicing`。

常用設定項目：

| 設定 | 說明 | 預設 |
| --- | --- | --- |
| `printerHostPort` | Web 服務埠 | `9091`（SSL 時 `443`） |
| `hostGUI` | 使用的 GUI 目錄：`resources`（現行）或 `resourcesnew`（新版） | `resources` |
| `fakeserial` / `fakedisplay` | 是否提供模擬序列埠 / 顯示器 | `false` |
| `SerialCommunicationsImplementation` | 序列埠實作（`JSSCCommPort` 或 RXTX 系列） | — |
| `printFileProcessor.<類別>=true` | 啟用的列印檔處理器（CWS、STL、MinerCube、圖片） | — |
| `notify.<類別>=true` | 啟用的通知外掛（WebSocket、Email） | — |
| `useSSL`、`keystoreFilename`、`*.clientUsername/Password` | TLS 與 Basic 認證設定 | `useSSL=false` |
| `imagingCommand` / `streamingCommand` | 拍照 / 錄影指令（預設為 `raspistill` / `raspivid`） | — |
| `toEmailAddresses`、`smtpServer`… | 列印完成 Email 通知 | — |

**切換到新版 GUI**：把

```
hostGUI=resources
```

改成

```
hostGUI=resourcesnew
```

等新版 GUI 功能完整後，將會成為預設介面。

---

## 開發者 API

伺服器以內嵌 Jetty 執行，主要端點如下：

| 路徑 | 說明 |
| --- | --- |
| `/` | 靜態 Web GUI（依 `hostGUI` 設定） |
| `/services/printers/...` | 印表機管理：`list`、`get/{name}`、`save`、`start/{name}`、`stop/{name}`、`executeGCode/{name}/{gcode}`、`moveZ/{name}/{distance}`、`homeZ/{name}`… |
| `/services/printJobs/...` | 列印工作：`get/{jobId}`、`stopJob/{jobId}`、`togglePause/{jobId}`、`currentSliceImage/{jobId}`、`overrideExposuretime/{jobId}/{time}`… |
| `/services/files/...` | 列印檔：`list`、`uploadPrintableFile`、`uploadviaurl/{filename}/{uri}`、`delete/{filename}` |
| `/services/machine/...` | 主機層級：序列埠 / 顯示器列表、網路介面、無線連線、重開機、診斷，以及舊版相容 API |
| `/services/settings`、`/services/media` | 主機設定、相機影像 / 錄影 |
| `/video/*` | 錄影檔漸進式下載 |
| `ws://<host>/printerNotification/{printerName}` | 印表機事件 WebSocket |
| `ws://<host>/printJobNotification/{printJobName}` | 列印工作事件 WebSocket |

範例：

```bash
curl http://<主機 IP>:9091/services/printers/list
```

---

## 從原始碼建置

本專案原本以 Eclipse 開發，沒有 Maven / Gradle 設定：

1. 將 `host/src` 加入原始碼路徑、`host/libs/**/*.jar` 加入 classpath，編譯輸出到 `host/srcbin`；
   測試（`host/tests`，需 `libs/testing` 下的 JUnit / Mockito / PowerMock）輸出到 `host/testbin`。
2. 在 `host/` 下執行 Ant：

   ```bash
   cd host
   ant -f BuildCWH.xml
   ```

   會產生 `cwh-0.<build>.zip`、`cwhClient-0.<build>.zip` 與 `cwhTestKit-0.<build>.zip`。
   注意 `BuildCWH.xml` 只負責打包，不會編譯 Java 原始碼。
3. 本機開發時可直接執行：

   ```bash
   CP="srcbin:$(find libs -name '*.jar' ! -path 'libs/testing/*' | tr '\n' ':')"
   java -Djava.library.path=os/Linux/x86_64 -cp "$CP" org.area515.resinprinter.server.Main
   ```

   搭配 `fakeserial=true`、`fakedisplay=true` 即可在沒有印表機硬體的環境下測試。

---

## 授權

[Creative Commons 姓名標示-相同方式分享 4.0 國際授權（CC BY-SA 4.0）](http://creativecommons.org/licenses/by-sa/4.0/)，
作者 Wes Gilster 與 Sean O'Bryan。詳見 [LICENSE.md](LICENSE.md)。
