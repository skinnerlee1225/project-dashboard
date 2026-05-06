# 多專案進度追蹤 Dashboard — 產品實作規格書

> **文件版本**：v1.0  
> **最後更新**：2025-05-06  
> **適用對象**：工程師、PM、技術主管  
> **目標**：說明如何將這個靜態 HTML Dashboard 改造為自動串接真實資料來源的系統

---

## 目錄

1. [系統總覽](#1-系統總覽)
2. [架構設計](#2-架構設計)
3. [資料來源與串接方式](#3-資料來源與串接方式)
4. [資料收集排程](#4-資料收集排程)
5. [Dashboard 前端改造](#5-dashboard-前端改造)
6. [部署方案](#6-部署方案)
7. [擴充建議](#7-擴充建議)

---

## 1. 系統總覽

### 1.1 現狀

目前的 Dashboard 是一個**純靜態 HTML 檔案**，所有專案資料直接寫在 JavaScript 裡面（`const projects = {...}`）。優點是零依賴、開即用；缺點是資料要手動更新。

### 1.2 目標

改造為**自動化收集 + 靜態渲染**的架構，每天自動從以下來源拉取資料：

| 資料來源 | 收集什麼 | 對應 Dashboard 區塊 |
|----------|----------|---------------------|
| Slack | 各部門的每日進度訊息 | 產品進度、設計、前端、後端 |
| GitHub / GitLab | Commit 記錄、PR 狀態 | Commit 記錄 Tab |
| QA 測試工具 | 測試結果、Bug 統計 | QA 測試 Tab |
| 專案管理工具 | User Story 狀態、里程碑 | 產品進度 Tab |

### 1.3 設計原則

這套系統刻意選擇**輕量架構**：用排程腳本收集資料、輸出成 JSON、再由 HTML 讀取 JSON 渲染。不需要後端伺服器、不需要資料庫。這樣做的好處是部署簡單（放 GitHub Pages 就能跑）、維護成本低、任何工程師都能接手。

---

## 2. 架構設計

### 2.1 整體流程

```
定時排程（GitHub Actions / Cron）
    │
    ├── 1. 從 Slack API 拉取頻道訊息
    ├── 2. 從 GitHub API 拉取 Commit / PR
    ├── 3. 從 QA 工具 API 拉取測試結果
    ├── 4. 從 Jira / Linear 拉取 User Story
    │
    ▼
收集腳本（Node.js / Python）
    │
    ├── 解析 + 格式化
    │
    ▼
輸出 data.json
    │
    ▼
HTML Dashboard 讀取 data.json 渲染
    │
    ▼
GitHub Pages 自動部署
```

白話說明：每天跑一次自動收集腳本，把各來源的資料整理成一個 JSON 檔案，HTML 頁面打開時讀這個 JSON 就能顯示最新資料。

### 2.2 資料夾結構

```
project-tracker/
├── index.html              ← Dashboard 頁面（改造後從 data.json 讀取）
├── data.json               ← 自動生成的資料檔
├── scripts/
│   ├── collect-slack.js     ← Slack 訊息收集
│   ├── collect-github.js   ← GitHub commit 收集
│   ├── collect-qa.js       ← QA 結果收集
│   ├── collect-pm.js       ← User Story / 里程碑收集
│   └── build-data.js       ← 整合所有來源，輸出 data.json
├── .github/
│   └── workflows/
│       └── daily-collect.yml  ← GitHub Actions 排程
├── .env.example            ← 環境變數範本（API Token 等）
└── README.md               ← 說明文件
```

### 2.3 data.json 格式

這個 JSON 的結構和目前 HTML 裡面 `const projects` 完全一致，只是從寫死改成外部讀取：

```json
{
  "lastUpdated": "2026-05-06T23:00:00+08:00",
  "projects": {
    "usdt": {
      "name": "USDT 理財平台 H5 活動頁",
      "sub": "全民衝刺季 — v1.0 ｜ 預計 5/20 上線",
      "color": "var(--blue)",
      "dates": [
        { "date": "5/06（三）", "key": "0506" }
      ],
      "daily": {
        "0506": {
          "ov": 68,
          "st": "24/32",
          "qaS": "87/142",
          "bugs": 7,
          "bugD": "P0:1 · P1:3 · P2:3",
          "dO": "+2%",
          "dS": "+1",
          "dQ": "+7",
          "dB": "-1",
          "product": [ ... ],
          "design": [ ... ],
          "fe": [ ... ],
          "be": [ ... ],
          "qaT": [ ... ],
          "commits": [ ... ]
        }
      }
    }
  }
}
```

### 2.4 「更新數據」按鈕

Dashboard 頁面內建了一個「更新數據」按鈕，點擊後會從同目錄下的 `data.json` 拉取最新資料並即時刷新畫面。

**運作機制**：

1. 使用者點擊「更新數據」按鈕
2. 前端透過 `fetch('data.json')` 發出 HTTP 請求
3. 如果成功取得 JSON，覆蓋目前的專案資料，重新渲染所有區塊
4. 頁面上方的狀態列會顯示「資料更新於 2026/5/6 下午 11:00」
5. 如果 `data.json` 不存在或格式錯誤，會顯示錯誤提示，畫面保持內建的範例資料不受影響

**狀態指示燈**：

| 燈號 | 含義 |
|------|------|
| 綠色（閃爍） | 已成功載入外部資料 |
| 黃色 | 正在拉取中 |
| 紅色 | 拉取失敗 |

**注意事項**：

由於瀏覽器的安全限制（CORS），`fetch` 在本機直接開 HTML 檔案（`file://` 協定）時會被阻擋。要讓按鈕正常運作，需要透過 HTTP 伺服器提供頁面，例如部署到 GitHub Pages，或在本機用以下指令啟動開發伺服器：

```bash
# 在 project-tracker 資料夾下執行
npx serve .
# 或
python -m http.server 8080
```

然後用瀏覽器開啟 `http://localhost:8080` 即可。

---

## 3. 資料來源與串接方式

### 3.1 Slack — 收集每日進度

**用途**：從各部門頻道抓取成員回報的進度訊息，轉換成 Dashboard 的進度數據。

**前置設定**：

1. 到 [api.slack.com/apps](https://api.slack.com/apps) 建立一個 Slack App
2. 在 OAuth & Permissions 加入以下 Scope：`channels:history`, `channels:read`, `users:read`
3. 安裝到 Workspace，取得 Bot Token（`xoxb-` 開頭）

**收集邏輯**（`scripts/collect-slack.js`）：

```javascript
// 核心概念：從指定頻道讀取今天的訊息
const { WebClient } = require('@slack/web-api');
const slack = new WebClient(process.env.SLACK_BOT_TOKEN);

async function collectSlack() {
  // 設定要監控的頻道
  const channels = {
    design: 'C0123DESIGN',     // #proj-design 頻道 ID
    frontend: 'C0123FRONTEND', // #proj-frontend
    backend: 'C0123BACKEND',   // #proj-backend
    product: 'C0123PRODUCT',   // #proj-product
  };

  const today = new Date();
  today.setHours(0, 0, 0, 0);
  const oldest = Math.floor(today.getTime() / 1000); // Unix 時間戳

  const result = {};

  for (const [dept, channelId] of Object.entries(channels)) {
    // 拉取今天的所有訊息
    const history = await slack.conversations.history({
      channel: channelId,
      oldest: String(oldest),
      limit: 200,
    });

    result[dept] = history.messages.map(msg => ({
      user: msg.user,
      text: msg.text,
      timestamp: msg.ts,
    }));
  }

  return result;
}
```

**建議的頻道使用方式**：

讓團隊成員在專屬頻道用固定格式回報進度，例如：

```
[進度] Landing Page - 85% - 進行中
[進度] 社群報銷頁面 - 40% - 進行中
[完成] DB Schema 建立
```

收集腳本解析這些格式化訊息，自動轉換成 Dashboard 的進度項目。如果團隊不習慣固定格式，也可以改用 Slack 的「工作流程建立工具」（Workflow Builder）做一個每日進度回報表單，這樣收到的資料更結構化。

**如何取得頻道 ID**：

在 Slack 裡對頻道名稱按右鍵 → 查看頻道詳情 → 最下方會顯示頻道 ID（C 開頭的字串）。

---

### 3.2 GitHub / GitLab — 收集 Commit 與 PR

**用途**：自動抓取每日的 commit 記錄，包含作者、訊息、變更檔案、行數統計。

**前置設定**：

1. 到 GitHub Settings → Developer settings → Personal access tokens → Fine-grained tokens
2. 建立 Token，給予目標 repo 的 `Contents: Read` 權限
3. 存到環境變數 `GITHUB_TOKEN`

**收集邏輯**（`scripts/collect-github.js`）：

```javascript
// 核心概念：用 GitHub API 拉取某天的所有 commit
async function collectGitHub() {
  const repos = [
    { owner: 'your-org', repo: 'usdt-h5-frontend', project: 'usdt' },
    { owner: 'your-org', repo: 'usdt-h5-backend', project: 'usdt' },
    { owner: 'your-org', repo: 'cex-wallet', project: 'cex' },
  ];

  const since = new Date();
  since.setHours(0, 0, 0, 0);

  const results = {};

  for (const { owner, repo, project } of repos) {
    // Step 1: 拉取 commit 列表
    const commitsRes = await fetch(
      `https://api.github.com/repos/${owner}/${repo}/commits?since=${since.toISOString()}`,
      { headers: { Authorization: `Bearer ${process.env.GITHUB_TOKEN}` } }
    );
    const commits = await commitsRes.json();

    // Step 2: 逐一取得每個 commit 的檔案變更細節
    const detailed = [];
    for (const c of commits) {
      const detailRes = await fetch(
        `https://api.github.com/repos/${owner}/${repo}/commits/${c.sha}`,
        { headers: { Authorization: `Bearer ${process.env.GITHUB_TOKEN}` } }
      );
      const detail = await detailRes.json();

      detailed.push({
        t: new Date(c.commit.author.date).toTimeString().slice(0, 5), // "21:34"
        h: c.sha.slice(0, 7),                                         // "a3f8e21"
        a: c.commit.author.name,                                       // "Eric Lin"
        av: getInitials(c.commit.author.name),                         // "EL"
        c: getColorByAuthor(c.commit.author.name),                     // "var(--blue)"
        tp: detectCommitType(c.commit.message),                        // "feat"
        m: c.commit.message.split('\n')[0],                            // 第一行
        f: detail.files.map(f => ({
          n: f.filename,
          a: f.additions,
          d: f.deletions,
        })),
      });
    }

    if (!results[project]) results[project] = [];
    results[project].push(...detailed);
  }

  return results;
}

// 從 commit message 偵測類型（feat / fix / refactor 等）
function detectCommitType(message) {
  const m = message.toLowerCase();
  if (m.startsWith('feat')) return 'feat';
  if (m.startsWith('fix')) return 'fix';
  if (m.startsWith('hotfix')) return 'hotfix';
  if (m.startsWith('refactor')) return 'refactor';
  if (m.startsWith('perf')) return 'perf';
  if (m.startsWith('docs')) return 'docs';
  return 'feat';
}
```

**GitLab 的差異**：

如果你用的是 GitLab 而不是 GitHub，API 端點不同但邏輯一樣：

```
# GitLab 的 commit API
GET https://gitlab.com/api/v4/projects/:id/repository/commits?since=2026-05-06
Header: PRIVATE-TOKEN: your-token
```

---

### 3.3 QA 測試工具 — 收集測試結果

**用途**：從 QA 工具取得測試案例的 Pass / Fail / Pending 狀態和 Bug 統計。

根據你用的 QA 工具，串接方式不同。以下列出常見的三種：

#### 方案 A：TestRail（推薦）

TestRail 有完整的 REST API，可以直接拉測試計畫的結果。

```javascript
async function collectTestRail() {
  const base = 'https://your-org.testrail.io/index.php?/api/v2';
  const auth = Buffer.from(`${email}:${apiKey}`).toString('base64');
  const headers = { Authorization: `Basic ${auth}`, 'Content-Type': 'application/json' };

  // 取得測試計畫下的所有測試結果
  const runs = await fetch(`${base}/get_runs/${projectId}`, { headers });
  const tests = await fetch(`${base}/get_tests/${runId}`, { headers });

  // status_id: 1=Passed, 2=Blocked, 3=Untested, 4=Retest, 5=Failed
  return tests.map(t => ({
    id: t.custom_id || `TC-${t.id}`,
    n: t.title,
    d: t.custom_description || '',
    r: t.status_id === 1 ? 'pass' : t.status_id === 5 ? 'fail' : 'pend',
  }));
}
```

#### 方案 B：Google Sheets（最簡單）

如果團隊的 QA 是用試算表記錄，可以用 Google Sheets API 讀取：

1. 建立一個 Google Sheet，表頭為：`編號 | 測試項目 | 測試內容 | 分類 | 結果`
2. 結果欄填 `Pass` / `Fail` / `Pending`
3. 啟用 Google Sheets API，建立服務帳號金鑰

```javascript
const { google } = require('googleapis');

async function collectFromSheets() {
  const auth = new google.auth.GoogleAuth({
    keyFile: 'service-account.json',
    scopes: ['https://www.googleapis.com/auth/spreadsheets.readonly'],
  });
  const sheets = google.sheets({ version: 'v4', auth });

  const res = await sheets.spreadsheets.values.get({
    spreadsheetId: 'your-sheet-id',
    range: 'QA測試!A2:E200', // 跳過表頭
  });

  const rows = res.data.values;
  // 按分類分組
  const grouped = {};
  for (const [id, name, desc, section, result] of rows) {
    if (!grouped[section]) grouped[section] = [];
    grouped[section].push({
      id, n: name, d: desc,
      r: result === 'Pass' ? 'pass' : result === 'Fail' ? 'fail' : 'pend',
    });
  }

  return Object.entries(grouped).map(([sec, items]) => ({ sec, items }));
}
```

#### 方案 C：Jira Bug 統計

如果 Bug 是開在 Jira 上，可以用 JQL 查詢統計：

```javascript
async function collectJiraBugs() {
  const jql = `project = USDT AND type = Bug AND status != Done`;
  const res = await fetch(
    `https://your-org.atlassian.net/rest/api/3/search?jql=${encodeURIComponent(jql)}`,
    { headers: {
      Authorization: `Basic ${Buffer.from(`email:api-token`).toString('base64')}`,
      'Content-Type': 'application/json',
    }}
  );
  const data = await res.json();

  // 按優先級統計
  const bugs = data.issues;
  const p0 = bugs.filter(b => b.fields.priority.name === 'Highest').length;
  const p1 = bugs.filter(b => b.fields.priority.name === 'High').length;
  const p2 = bugs.filter(b => b.fields.priority.name === 'Medium').length;

  return {
    total: bugs.length,
    bugD: `P0:${p0} · P1:${p1} · P2:${p2}`,
  };
}
```

---

### 3.4 專案管理工具 — 收集 User Story 與里程碑

**用途**：從 Jira / Linear / Notion 取得 User Story 的狀態和完成進度。

#### Jira 範例

```javascript
async function collectUserStories() {
  const jql = `project = USDT AND type = Story ORDER BY priority DESC`;
  const res = await fetch(
    `https://your-org.atlassian.net/rest/api/3/search?jql=${encodeURIComponent(jql)}&fields=summary,status,priority,customfield_10001`,
    { headers: { /* 同上 */ } }
  );
  const data = await res.json();

  return data.issues.map(issue => ({
    id: issue.key,                                    // "USDT-001"
    t: issue.fields.summary,                          // "活動首頁 Landing Page"
    d: issue.fields.description || '',
    ac: issue.fields.customfield_10001 || 0,           // 驗收標準數（自訂欄位）
    pri: mapPriority(issue.fields.priority.name),      // "P0"
    s: mapStatus(issue.fields.status.name),            // "done" / "prog" / "todo"
  }));
}

function mapStatus(jiraStatus) {
  const map = { 'Done': 'done', 'In Progress': 'prog', 'In Review': 'rev', 'To Do': 'todo' };
  return map[jiraStatus] || 'todo';
}
```

#### Linear 範例

```javascript
async function collectFromLinear() {
  const query = `{
    issues(filter: { project: { id: { eq: "project-id" } } }) {
      nodes { identifier title description state { name } priority }
    }
  }`;

  const res = await fetch('https://api.linear.app/graphql', {
    method: 'POST',
    headers: {
      Authorization: process.env.LINEAR_API_KEY,
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({ query }),
  });

  const { data } = await res.json();
  return data.issues.nodes.map(i => ({
    id: i.identifier,
    t: i.title,
    d: i.description || '',
    pri: i.priority <= 1 ? 'P0' : 'P1',
    s: mapLinearStatus(i.state.name),
  }));
}
```

---

## 4. 資料收集排程

### 4.1 使用 GitHub Actions（推薦）

建立 `.github/workflows/daily-collect.yml`：

```yaml
name: Daily Data Collection
on:
  schedule:
    - cron: '0 15 * * *'   # UTC 15:00 = 台灣時間 23:00，每天收集一次
  workflow_dispatch:         # 也可以手動觸發

jobs:
  collect:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: '20'

      - run: npm install

      - name: Run data collection
        env:
          SLACK_BOT_TOKEN: ${{ secrets.SLACK_BOT_TOKEN }}
          GITHUB_TOKEN: ${{ secrets.GH_PAT }}
          QA_API_KEY: ${{ secrets.QA_API_KEY }}
          JIRA_EMAIL: ${{ secrets.JIRA_EMAIL }}
          JIRA_TOKEN: ${{ secrets.JIRA_TOKEN }}
        run: node scripts/build-data.js

      - name: Commit and push data.json
        run: |
          git config user.name "Dashboard Bot"
          git config user.email "bot@example.com"
          git add data.json
          git diff --cached --quiet || git commit -m "更新 $(date +%Y-%m-%d) 進度資料"
          git push
```

白話說明：GitHub Actions 每天晚上 11 點自動跑收集腳本，把結果存成 data.json，然後自動 commit + push。因為有開 GitHub Pages，push 之後網頁就會自動更新。

### 4.2 環境變數設定

到 GitHub Repo → Settings → Secrets and variables → Actions → New repository secret，依序加入：

| Secret 名稱 | 說明 | 取得方式 |
|-------------|------|----------|
| `SLACK_BOT_TOKEN` | Slack Bot Token（xoxb- 開頭） | Slack App 管理頁面 → OAuth & Permissions |
| `GH_PAT` | GitHub Personal Access Token | Settings → Developer settings → Tokens |
| `QA_API_KEY` | QA 工具的 API Key | 各工具的 API 設定頁面 |
| `JIRA_EMAIL` | Jira 帳號 Email | 你的 Atlassian 帳號 |
| `JIRA_TOKEN` | Jira API Token | [id.atlassian.com/manage/api-tokens](https://id.atlassian.com/manage/api-tokens) |

---

## 5. Dashboard 前端改造

### 5.1 改為從 JSON 讀取資料

把 HTML 中的 `const projects = {...}` 替換為 fetch 讀取 JSON：

```javascript
// 修改前（靜態寫死）
const projects = { usdt: { ... }, cex: { ... } };

// 修改後（從 JSON 載入）
let projects = {};

async function loadData() {
  try {
    const res = await fetch('data.json');
    const data = await res.json();
    projects = data.projects;

    // 顯示最後更新時間
    document.getElementById('lastUpdate').textContent =
      `最後更新：${new Date(data.lastUpdated).toLocaleString('zh-TW')}`;

    renderAll();
  } catch (e) {
    document.getElementById('tabBody').innerHTML =
      '<div style="text-align:center;padding:40px;color:var(--t2)">' +
      '無法載入資料，請確認 data.json 檔案存在</div>';
  }
}

loadData();
```

### 5.2 加入最後更新時間顯示

在 HTML 的 footer 區域加入更新時間：

```html
<div class="footer">
  <span id="lastUpdate">載入中...</span>
  <span> · Sample Dashboard</span>
</div>
```

### 5.3 離線 Fallback

如果希望在沒有 data.json 時仍然能顯示範例資料：

```javascript
async function loadData() {
  try {
    const res = await fetch('data.json');
    if (!res.ok) throw new Error('No data.json');
    const data = await res.json();
    projects = data.projects;
  } catch {
    // Fallback：使用內建的範例資料
    projects = fallbackProjects;
  }
  renderAll();
}

// 保留目前的假資料作為 fallback
const fallbackProjects = { usdt: { ... } };
```

---

## 6. 部署方案

### 6.1 GitHub Pages（推薦）

整個流程形成閉環：收集腳本推送 data.json → GitHub Pages 自動重新部署 → 使用者開網址就看到最新資料。

設定步驟（只需做一次）：

1. 建 GitHub repo，把 `index.html`、`data.json`、`scripts/` 推上去
2. Settings → Pages → Deploy from branch → `main` → `/ (root)` → Save
3. Settings → Secrets → 加入各 API Token
4. 推送 `.github/workflows/daily-collect.yml`
5. 完成。之後每天自動更新

### 6.2 如果需要即時更新（進階）

如果每天一次不夠，想要更即時的更新，可以考慮：

「Webhook 觸發」：在 Slack 設定一個 Slash Command（例如 `/update-dashboard`），任何人輸入這個指令就觸發一次收集。或者在 GitHub repo 設定 push webhook，每次有人 push code 就自動更新 commit 記錄。

「短輪詢」：把排程改為每 30 分鐘跑一次（`cron: '*/30 * * * *'`），注意各 API 的 Rate Limit。

---

## 7. 擴充建議

### 7.1 串接更多資料來源

| 來源 | 用途 | API 文件 |
|------|------|----------|
| Figma | 設計稿完成狀態 | [figma.com/developers](https://www.figma.com/developers) |
| Notion | 文件/規格完成進度 | [developers.notion.com](https://developers.notion.com) |
| Sentry | 線上錯誤監控 | [docs.sentry.io/api](https://docs.sentry.io/api/) |
| Confluence | 文件完成狀態 | Atlassian REST API |

### 7.2 通知機制

可以加一個 Slack 通知，每天收集完成後自動發送摘要到指定頻道：

```javascript
await slack.chat.postMessage({
  channel: '#proj-dashboard',
  text: `📊 今日進度更新\n` +
        `USDT H5：${usdtProgress}% (+${usdtDiff}%)\n` +
        `CEX 錢包：${cexProgress}%\n` +
        `查看完整報表：https://your-org.github.io/project-tracker/`,
});
```

### 7.3 權限控管

如果有敏感資料，可以用 Netlify 的 Password Protection 功能（付費方案），或把 GitHub repo 設為 Private 並用 Cloudflare Access 保護 Pages 網址。

---

## 附錄：API Token 安全提醒

所有 API Token 都必須存放在環境變數或 GitHub Secrets 中，絕對不能直接寫在程式碼裡。如果不小心 commit 了 Token，要立即到對應平台撤銷並重新產生。GitHub 有內建的 Secret Scanning 功能，開啟後會自動偵測意外提交的 Token。
