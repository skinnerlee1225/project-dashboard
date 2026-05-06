# 多專案追蹤報表 — 部署與分享指南

> 這份文件教你如何把 `project-tracker.html` 分享給團隊成員。
> 提供三種方式，從最推薦到最簡單，選一種適合你的就好。

---

## 目錄

1. [檔案說明](#1-檔案說明)
2. [方式一：GitHub Pages（推薦）](#2-方式一github-pages推薦)
3. [方式二：Netlify 拖拉部署](#3-方式二netlify-拖拉部署)
4. [方式三：直接傳檔案](#4-方式三直接傳檔案)
5. [客製化：修改專案資料](#5-客製化修改專案資料)
6. [常見問題 FAQ](#6-常見問題-faq)

---

## 1. 檔案說明

這是一個 **單一 HTML 檔案**，打開就能用，不需要安裝任何軟體。

| 項目 | 內容 |
|------|------|
| 檔案名稱 | `project-tracker.html` |
| 檔案大小 | 約 55 KB |
| 外部依賴 | 僅 Google Fonts（字型），無網路也能用，只是字型會退回系統預設 |
| 瀏覽器支援 | Chrome、Safari、Firefox、Edge 皆可 |
| 深色模式 | 自動跟隨系統設定 |

打開方式：直接雙擊 `.html` 檔案，或拖進瀏覽器視窗即可。

---

## 2. 方式一：GitHub Pages（推薦）

> **適合誰**：有 GitHub 帳號的人，或需要長期維護、隨時更新的情境。
>
> **結果**：你會得到一個公開網址，例如 `https://你的帳號.github.io/project-tracker/`，
> 任何人點連結就能看到互動報表。

### 步驟 1：安裝 Git（如果還沒裝的話）

打開「終端機」（Mac 按 `Command + 空白鍵`，輸入 `Terminal`）。

```bash
# 檢查有沒有安裝 Git
git --version
```

如果有顯示版本號（例如 `git version 2.39.0`），就可以跳到步驟 2。

如果沒有，Mac 會自動提示你安裝 Xcode Command Line Tools，按「安裝」就好。

### 步驟 2：在 GitHub 建立新 Repository

1. 用瀏覽器打開 [github.com/new](https://github.com/new)
2. **Repository name** 填入：`project-tracker`（或你喜歡的名字）
3. **Description** 填入：`多專案進度追蹤報表`（選填）
4. 選擇 **Public**（GitHub Pages 免費版需要公開）
5. **不要**勾選 "Add a README file"
6. 點「Create repository」

### 步驟 3：把檔案推上去

在終端機裡，依序執行以下指令。請把 `你的帳號` 換成你的 GitHub 帳號名稱：

```bash
# 1. 建立資料夾並進入
mkdir project-tracker
cd project-tracker

# 2. 把 HTML 檔案複製進來，並改名為 index.html
#    ⚠️ 改名為 index.html 是關鍵，GitHub Pages 會自動找這個檔名
cp /你的HTML檔案路徑/project-tracker.html ./index.html

# 3. 初始化 Git
git init

# 4. 加入檔案
git add index.html

# 5. 建立第一個 commit
git commit -m "初始化：多專案追蹤報表"

# 6. 連結到 GitHub（把「你的帳號」換成你自己的）
git remote add origin https://github.com/你的帳號/project-tracker.git

# 7. 推上去
git branch -M main
git push -u origin main
```

> **白話說明**：上面這段指令做的事情就是——
> 建一個資料夾 → 把 HTML 丟進去 → 告訴 Git「我要追蹤這個檔案」→ 傳到 GitHub。

### 步驟 4：開啟 GitHub Pages

1. 在瀏覽器打開你的 repo 頁面：`https://github.com/你的帳號/project-tracker`
2. 點上方的 **Settings**（齒輪圖示）
3. 左側選單找到 **Pages**
4. 在 **Source** 區塊，選擇 **Deploy from a branch**
5. Branch 選 `main`，資料夾選 `/ (root)`
6. 點 **Save**
7. 等 1～2 分鐘，頁面上方會出現你的網址：

```
https://你的帳號.github.io/project-tracker/
```

把這個網址傳給任何人，他們就能直接看到報表了！

### 之後更新怎麼辦？

每次修改完 `index.html`，只需要：

```bash
cd project-tracker
git add index.html
git commit -m "更新進度資料"
git push
```

等約 1 分鐘，網頁就會自動更新。

---

## 3. 方式二：Netlify 拖拉部署

> **適合誰**：不想碰終端機、不想用 Git 的人。
>
> **結果**：同樣得到一個公開網址，例如 `https://random-name.netlify.app`。

### 步驟

1. 用瀏覽器打開 [app.netlify.com/drop](https://app.netlify.com/drop)
2. 如果沒有帳號，用 GitHub 或 Email 免費註冊一個
3. 準備一個資料夾，裡面放一個檔案：把 `project-tracker.html` **改名為 `index.html`**
4. 把這個**整個資料夾**拖進 Netlify 頁面上的虛線框
5. 等幾秒鐘，部署完成！
6. Netlify 會給你一個網址，點進去確認沒問題
7. （選用）點 **Site configuration → Change site name** 改成好記的名字

> **白話說明**：就是把資料夾拖進網頁裡，Netlify 就幫你上線了。
> 就像把檔案拖進 Google Drive 一樣簡單。

### 更新方式

在 Netlify 後台點 **Deploys**，再把更新後的資料夾拖進去就好。

---

## 4. 方式三：直接傳檔案

> **適合誰**：只是臨時分享給幾個人看，不需要長期維護。

### 選項 A：通訊軟體直傳

直接用 LINE、Telegram、Slack、Email 把 `project-tracker.html` 傳給對方。

對方收到後：
- **電腦**：雙擊檔案，會自動用瀏覽器開啟
- **手機**：點檔案 → 選擇「用瀏覽器開啟」

> 注意：每次更新都需要重新傳一次。

### 選項 B：雲端硬碟分享

1. 把 `project-tracker.html` 上傳到 Google Drive / Dropbox / iCloud
2. 產生分享連結，傳給對方
3. 對方下載後用瀏覽器開啟

> 注意：雲端硬碟**無法直接在線上預覽 HTML**，對方需要下載後才能互動操作。

---

## 5. 客製化：修改專案資料

如果你想把範例資料換成自己的專案，用任何文字編輯器（VS Code、Sublime Text、甚至記事本）打開 HTML 檔案。

### 找到資料區塊

在檔案中搜尋 `const projects`，你會看到類似這樣的結構：

```javascript
const projects = {
  usdt: {
    name: 'USDT 理財平台 H5 活動頁',  // ← 改這裡：專案名稱
    sub: 'H5 Campaign Page',           // ← 改這裡：副標題
    color: 'var(--blue)',               // ← 改這裡：主題色
    dates: [                            // ← 改這裡：日期列表
      { date: '5/02（六）', key: '0502' },
      { date: '5/03（日）', key: '0503' },
      // ...
    ],
    daily: {
      '0502': {                         // ← 每天的資料
        ov: 58,                         // 整體進度百分比
        st: '20/32',                    // User Story 完成數
        qaS: '62/142',                  // QA 測試通過數
        bugs: 11,                       // Bug 數量
        // ... 更多欄位
      }
    }
  },
  // 新增專案：複製整個 usdt 區塊，改 key 名稱和內容
};
```

### 新增一個專案

複製任一個現有專案的整個區塊，貼在 `projects` 物件裡面，然後修改：

1. 物件的 key（例如改成 `newProject`）
2. `name`（專案名稱）
3. `sub`（副標題）
4. `color`（可用：`var(--blue)`, `var(--teal)`, `var(--purple)`, `var(--coral)`, `var(--pink)`）
5. `dates` 和 `daily` 裡的實際資料

### 可用的主題色

| 色碼變數 | 顏色 | 適合用於 |
|----------|------|----------|
| `var(--blue)` | 藍色 | 主要專案 |
| `var(--teal)` | 青色 | 次要專案 |
| `var(--purple)` | 紫色 | 新專案 |
| `var(--coral)` | 珊瑚橘 | 緊急專案 |
| `var(--pink)` | 粉紅 | 活動類專案 |

---

## 6. 常見問題 FAQ

**Q：打開 HTML 檔案是空白的？**
確認你是用「瀏覽器」開啟，不是用文字編輯器。在檔案上按右鍵 → 用 Chrome / Safari 打開。

**Q：GitHub Pages 顯示 404？**
確認檔案名稱是 `index.html`（不是 `project-tracker.html`），而且 Pages 設定的 branch 是 `main`。部署需要約 1~2 分鐘，稍等再重新整理。

**Q：深色模式怎麼切換？**
報表會自動跟隨你的系統設定。Mac 到「系統設定 → 外觀」切換；Windows 到「設定 → 個人化 → 色彩」切換。

**Q：手機上可以看嗎？**
可以，報表有做響應式設計，手機瀏覽器也能正常操作，包含切換專案、日期和各部門 Tab。

**Q：可以讓報表設密碼嗎？**
GitHub Pages 免費版不支援密碼保護。如果需要存取控制，建議改用 Netlify（有 Password Protection 功能，需付費方案）或直接傳檔案。

**Q：沒有網路的環境能用嗎？**
可以。直接用瀏覽器開啟 HTML 檔案即可，所有功能都能正常運作，只是 Google Fonts 字型會退回系統預設字型，不影響使用。

---

> 有問題歡迎在群組討論，或直接聯絡報表維護人。
