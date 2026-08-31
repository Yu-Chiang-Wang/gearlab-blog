# GearLab Blog — Claude 專案說明

## 專案概覽
- **網站**：https://yu-chiang.me
- **定位**：3C 評測選物部落格，主題為 iPad 配件、GaN 充電器
- **框架**：Hugo 靜態網站 + PaperMod 主題（git submodule）
- **部署**：GitHub Actions → GitHub Pages（push main 自動部署）

## 新增文章工作流

每次新增文章請依 [WORKFLOW.md](WORKFLOW.md) 執行，涵蓋 SEO 前置研究、本地驗證、push 部署檢查、Google Search Console 提交與發布後追蹤。

## Debug 積極度

**每次寫完 content 或改 layout，必須用 preview 工具驗證實際畫面，不可只靠 build 成功就回報完成。**

驗證清單：
- 截圖確認文章頁面頂部（標題、麵包屑）
- 截圖確認表格、清單等複雜元素的 mobile（375px）和 desktop（1280px）
- 檢查 HTML 結構確認 class 名稱正確後再寫 CSS
- 新增 partial / layout 後用 `curl` 確認已注入 HTML

## 目錄結構

```
├── content/
│   ├── posts/          # 評測文章
│   ├── about.md        # 關於頁
│   ├── links.md        # Linktree 風格導覽頁
│   ├── privacy-policy.md
│   ├── affiliate-disclosure.md
│   └── selection-criteria.md
├── assets/
│   ├── css/extended/brand.css  # 品牌樣式（會併進主 stylesheet）
│   └── images/                 # 所有文章圖片放這（**不是 static/**，見下方「圖片處理」）
├── layouts/
│   ├── _default/
│   │   └── links.html          # /links 頁自訂版型
│   ├── _markup/
│   │   └── render-image.html   # Markdown ![]() 的 render hook
│   ├── _shortcodes/
│   │   └── figure.html         # 覆寫內建 figure，走 responsive-image
│   └── _partials/
│       ├── responsive-image.html   # 全站唯一的 <img> 產生器（WebP + srcset）
│       ├── cover.html              # 覆寫 PaperMod 封面（LCP 圖，eager + fetchpriority）
│       ├── google_analytics.html   # 覆寫內建 GA：gtag.js 延到 load 後才載
│       ├── schema.html             # JSON-LD
│       ├── extend_footer.html      # PaperMod footer 延伸（訂閱表單）
│       └── extend_head.html        # 自訂 CSS/JS 注入點、自架字體
├── static/
│   ├── CNAME               # yu-chiang.me
│   └── fonts/              # 自架 Sora woff2（不再打 fonts.googleapis.com）
├── themes/PaperMod/        # git submodule
├── hugo.toml               # 主設定
└── .github/workflows/hugo.yml  # CI/CD
```

## hugo.toml 重點設定

- `theme = "PaperMod"`
- `languageCode = "zh-tw"`
- `googleAnalytics = "G-23LVXQLZNR"`
- `[outputs] home = ["HTML", "RSS", "JSON"]`（支援 RSS feed）
- Supabase 訂閱參數放在 `[params]` 區塊：
  - `supabaseUrl` — Supabase 專案 URL
  - `supabaseAnonKey` — Supabase anon/public key

## 文章 front matter 格式

```yaml
---
title: ""
date: 2026-05-10T12:00:00+08:00
draft: false
description: ""
tags: ["GaN充電器", "充電器", "3C"]
categories: ["充電器"]
slug: "url-slug"
cover:
  image: ""
  alt: ""
---
```

## 麵包屑 section 標題

各 section 須有 `_index.md` 設定 `title`，否則顯示英文資料夾名。

## Email 訂閱系統（Supabase）

架構：純前端 JavaScript 呼叫 Supabase REST API，不需後端。

### Supabase 初次設定（只需做一次）

在 Supabase SQL Editor 執行：

```sql
CREATE TABLE subscribers (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  email TEXT NOT NULL UNIQUE,
  created_at TIMESTAMPTZ DEFAULT now()
);

ALTER TABLE subscribers ENABLE ROW LEVEL SECURITY;

CREATE POLICY "allow_public_subscribe" ON subscribers
  FOR INSERT TO anon WITH CHECK (true);
```

### 設定金鑰

在 hugo.toml `[params]` 填入：

```toml
supabaseUrl = "https://faznibhmbrciqwctygzd.supabase.co"
supabaseAnonKey = "sb_publishable_h5WNBoFK8Y2MfpaghnBJUg_5QbZp-QC"
```

## 圖片處理（2026-08-31 PageSpeed 調校後）

**新文章的圖片一律放 `assets/images/posts/<slug>/`，不要放 `static/images/`。**

- `assets/` 底下的圖才會被 Hugo 處理：自動產生 360/480/640/720/960/1280（＋原寬）
  的 WebP、srcset、`sizes`、`width`/`height`
- 原圖直接丟原始尺寸，不用自己先壓；縮圖與轉檔交給
  `layouts/_partials/responsive-image.html`
- front matter 的 `cover.image` 路徑寫法不變（`/images/posts/<slug>/xxx.jpg`），
  `extend_head.html` 會另外把原圖發佈回 `/images/…` 原網址，讓 `og:image` 不會破
- 放進 `static/images/` 的圖會原尺寸直出、沒有 width/height → PageSpeed 的
  「提升圖片傳送效能」和「圖片元素沒有明確的 width 和 height」會同時扣分

## PaperMod 客製化規則

- **不要修改 themes/PaperMod/**，改用 `layouts/` 覆蓋
- Footer 延伸：`layouts/_partials/extend_footer.html`
- Head 延伸：`layouts/_partials/extend_head.html`
- 自訂頁面版型：`layouts/_default/<type>.html`
- 注意：Hugo v0.146+ 將 `partials/` 改為 `_partials/`，PaperMod 已跟進

## 建置指令

```bash
hugo server          # 本地預覽（http://localhost:1313）
hugo --minify --cleanDestinationDir  # 正式建置
```

## 聯絡信箱
jonathanwang1103@gmail.com

## 附屬連結
蝦皮分潤：`af_id=16336070025`
