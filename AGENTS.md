# zh-thai-translation

本檔是所有 agent 的專案指令正本；`CLAUDE.md` 只匯入本檔。

## 這是什麼

- 專案 id：`zh-thai-translation`；機密等級：內部。
- 工廠中泰翻譯 PWA，單畫面雙按鈕對話、翻譯與朗讀，常用句可離線使用。
- GitHub Pages 與 GAS 是正式部署用途；本機開發不等於測試後端，翻譯／回饋可能寫入正式 Sheets。

## 資訊的唯一來源

| 資訊 | 正本 |
| --- | --- |
| 現況、下一步、效益數字 | siyulio-workspace 的 `projects/zh-thai-translation/state.md`（依 id 找，不假設相鄰路徑） |
| 安裝、啟動、驗證 | 本檔 |
| 變更歷史 | `git log` |
| 部署說明 | `README.md`、`.github/workflows/deploy.yml`、`google-apps-script.js` |

## 目錄與工具鏈

- Vanilla JS + Vite PWA（不是 Vue）+ Google Apps Script 後端。
- `src/`：前端；`public/`：靜態資源；`test/check.mjs`：本機自檢；根目錄 GAS 檔為後端來源。
- Node 24 + npm 可執行本機流程；現有 CI 使用 Node 20，兩者分開驗證，不以本機通過推定 CI 通過。
- 套件版本以 `package-lock.json` 為準，不在契約任務升級相依。

## 安裝

在 repo 根目錄執行：

```sh
npm ci
```

## 啟動

以下只啟動本機 Vite 並讀 HTML，不讀 env、不開瀏覽器、不呼叫翻譯後端；會建立忽略的 Vite 快取，結束自動關閉：

```sh
node --input-type=module -e "import { createServer } from 'vite'; const s=await createServer({envFile:false,server:{host:'127.0.0.1',port:4320,strictPort:true,open:false}}); try { await s.listen(); const r=await fetch('http://127.0.0.1:4320/tw-thai-translator/'); if(!r.ok) throw Error(String(r.status)); console.log('HTTP',r.status); } finally { await s.close(); }"
```

互動開發為 `npm run dev -- --host 127.0.0.1`；先確認測試用後端與瀏覽器 localStorage 設定，再執行翻譯／回饋。沒有測試設定時，只做上方 HTTP 檢查。

## 驗證

```sh
node test/check.mjs
node --check src/gemini.js
node --check google-apps-script.js
```

- 自檢只載入 `src/data.js` 與 `src/cache.js`；一般 Node 執行使用記憶體 Map，不指定 localStorage 儲存檔，不呼叫 API、不改檔。
- 改 `src/data.js` 後必跑自檢；語意正確性仍需泰語母語者核對，不能用自檢替代。
- 不要當驗證跑：`npm run build`（寫 dist）、部署 workflow、GAS 部署、正式翻譯／回饋 E2E，以及產生報告的腳本。

## 核心設計

- 術語與預譯常用句資料在 `src/data.js`（離線使用、不呼叫 API）。
  泰文存男性語尾，`toFemaleThai()` 轉女性；prompt 只注入輸入命中的術語。
- 翻譯先查本地 LRU，再選 GAS proxy；`src/gemini.js` 仍保留直連與 fallback，勿宣稱已移除。
- `VITE_*` 會進前端 bundle；不把 key 放前端。正式 GAS key 應留在 Script Properties。
- 回饋閉環：翻譯泡泡 👍👎 → Sheet `feedback-*` 列 → 修 data.js。
- `node test/check.mjs` = 資料完整性自檢，改 data.js 後必跑。

## 開工與收工

- 開工看 workspace state.md、`git status` 與 `git log --oneline -10`；遇未提交變更先停。
- 使用 `feature-*`／`fix-*` 分支，不直接 commit 到 `main`；收工記錄驗證與 commit，下一步只更新 workspace 對應條目。
- prompt 在前端與 GAS 各有一份；變更時核對兩端一致，部署另取授權。

## 不可碰

- `.env*`（`.env.example` 除外）、憑證、真實使用紀錄與個資不讀、不修改、不提交。
- `main` push 會觸發 Pages 部署；push、部署、GitHub secrets／variables、GAS 屬性與 key 輪替需 Jason 當次授權。
- 不用正式 GAS／Gemini／Sheets 做自動驗證，也不把現況或效益數字另抄進 repo。
