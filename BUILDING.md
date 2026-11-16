# 構建與驗證

一般使用者直接下載完整 Windows Release，不需執行以下步驟。

## 啟動器

使用 Windows x64、Python 3.12（含 Tcl/Tk）：

```powershell
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements-dev.txt
.\.venv\Scripts\python.exe tools/build_refined.py
```

輸出為 `ArknightsJPZH.exe`。建置程序會測試來源程式、打包程式與帶有錯誤 Python/Tcl 環境變數的打包程式，並確認安全連線仍啟用憑證與主機名稱驗證，成功後才視為可發布。一般啟動器不需要資源提取依賴；維護者重建資料時另裝 `requirements.txt`。

## 翻譯外掛

使用 .NET 6 SDK；取得完整 Release 並將其 `runtime` 置於專案根目錄。先讓自有日服遊戲啟動一次產生 BepInEx interop，再指定其目錄：

```powershell
dotnet build plugins/ArknightsLocalization.RichTextFix/ArknightsLocalization.RichTextFix.csproj -c Release -p:GameInteropPath="C:/YostarGames/Arknights_JP/BepInEx/interop"
```

外掛來源與實際嵌入的 Resources 都包含在專案中。不要混用舊版 Resources 重新編譯後宣稱與本 Release 相同。SDK 建置輸出可能因編譯器版本而有不同二進位雜湊，應以測試與資源內容確認。

本機 interop、遊戲記錄、帳號設定與遊戲本體不應提交。更新原始資料時請使用維護工具與公開上游資料，並對照同一個日服客戶端版本。

## 驗證原則

檢查語法與檔案校驗只能證明構建／安裝完整性。對於新顯示元件，應從實際遊戲畫面及有限的公開文字查詢記錄確認，不能用大量資料案例通過代替實機確認。

本版修改不替換遊戲資源 bundle，僅安裝外掛、設定與翻譯字典。發布包應排除個人設定、記錄、快取、備份、interop 及開發環境。

## 2.1.10 補充

簡易首頁位於 `arklocalizer/launcher_simple_ui.py`，安裝及備份仍使用既有後端。GitHub Release 提示位於 `launcher_release_notice.py`，遊戲狀態自動刷新位於 `launcher_status.py`。啟動器不再連接舊版獨立詞庫更新介面。可執行 `python -m unittest discover -s tests -p "test_*.py"` 檢查啟動器操作狀態與更新來源隔離；Tk 測試需有可用桌面。

本版保留 Resources 中的衝突核對與首頁語音資料，以及對應產生器；重新產生資料時須一併使用，不能只更新通用字典。發布包使用 `python tools/package_refined.py <輸出目錄>` 製作，會校驗 runtime 與啟動器雜湊。

若 Python 發行版的 `ssl.py` 依賴額外的憑證政策檔，建置腳本會隨 EXE 打包該檔案；不略過 TLS 憑證驗證。

怪獵聯動資料保留於 Resources/monster_hunter_2110.json 與 monster_hunter_2110_global.json；前者由上下文產生器載入，後者使用 arklocalizer.xunity.write_translation_file 產生 61_monster_hunter_20261116.txt，並同步更新 runtime 檔案清單。
