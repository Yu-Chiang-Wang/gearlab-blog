# 新增文章工作流

從草稿到發布後驗證的完整流程。每次新增文章都依此順序執行。

## 1. 規劃與 SEO 前置研究

- **關鍵字研究**：確認主關鍵字 + 1–2 個長尾關鍵字（例：「GaN 充電器 推薦 2026」）
- **競品檢查**：Google 搜尋目標關鍵字，看前 5 名文章結構、字數、缺漏點
- **slug 規劃**：英文 kebab-case,反映關鍵字（例：`best-gan-charger-2026`）
- **內部連結地圖**：列出本文要連到/被連到的既有文章

## 2. 建立文章檔案

```bash
hugo new posts/your-slug.md
```

填寫 front matter(CLAUDE.md 已定義格式),重點:

- `title`:50–60 字內,含主關鍵字
- `description`:120–160 字,作為 meta description
- `tags` / `categories`:沿用既有分類,不要創新
- `slug`:對齊檔名
- `cover.image` + `cover.alt`:social share 必備,alt 寫實際內容(SEO + a11y)

## 3. 內容撰寫 SEO 檢查點

- **H1 只有 title**,內文用 H2 / H3(避免跳級)
- **首段 150 字內**自然帶入主關鍵字
- **表格、清單**:複雜元素要在 mobile 375px 測試(CLAUDE.md 指示)
- **圖片**:放 `assets/images/posts/<slug>/`(**不是 static/**),全部加 alt
  - 放 `assets/` 才會被 Hugo 處理成 WebP + srcset + width/height;放 static/ 會原尺寸直出,PageSpeed 直接扣分
  - 原圖丟原始尺寸即可,**不用先壓縮**,縮圖交給 `layouts/_partials/responsive-image.html`
  - front matter 的 `cover.image` 一樣寫 `/images/posts/<slug>/xxx.jpg`(路徑不變)
- **內部連結**:至少 2–3 條連到既有文章
- **外部連結**:權威來源開新分頁(`{target="_blank" rel="noopener"}`)
- **聯盟連結**:蝦皮 `af_id=16336070025`,並在文中或文末聲明(站內已有 `/affiliate-disclosure`)

## 4. 本地預覽驗證(強制)

```bash
hugo server
```

按 CLAUDE.md 指示用 preview 工具截圖驗證:

- [ ] 文章頁頂部:標題、麵包屑、cover image
- [ ] Mobile 375px:表格不溢出、字級可讀
- [ ] Desktop 1280px:版面比例正常
- [ ] TOC 自動產生、層級正確
- [ ] 內部連結點得到、無 404
- [ ] Footer 訂閱表單仍正常

## 5. 建置與本地最終測試

```bash
hugo --minify --cleanDestinationDir
```

- 檢查 `public/` 內 `<slug>/index.html` 存在
- `public/sitemap.xml` 已包含新 URL
- `public/index.xml`(RSS)首篇為新文章
- 用瀏覽器開 `public/posts/<slug>/index.html` 確認 meta tags:
  - `<title>`、`<meta name="description">`
  - `og:title` / `og:description` / `og:image`
  - `twitter:card`

## 6. Push 與部署驗證

```bash
git add content/posts/<slug>.md assets/images/posts/<slug>/
git commit -m "feat: add <topic> article"
git push origin main
```

部署後檢查:

- [ ] GitHub Actions(`.github/workflows/hugo.yml`)綠燈
- [ ] `https://yu-chiang.me/posts/<slug>/` 200 OK
- [ ] `view-source:` 確認 GA `G-23LVXQLZNR` 已注入
- [ ] 用手機實機開一次(CDN 快取、行動體驗)
- [ ] PageSpeed Insights 跑一次主要指標(LCP / CLS)

## 7. Google Search Console 提交

1. 進入 [Search Console](https://search.google.com/search-console),選 `yu-chiang.me` 資源
2. 上方搜尋列貼新文章完整 URL → **網址審查**
3. 看到「網址不在 Google 服務索引中」→ 點 **要求建立索引**
4. 等待 1–2 分鐘的 live test,確認沒有「索引問題」
5. 確認 `https://yu-chiang.me/sitemap.xml` 在「Sitemap」頁面狀態為「成功」(一次性設定,之後 Hugo 自動更新)
6. 1–3 天後回 Search Console 確認該 URL 已被索引

## 8. 發布後追蹤(1–2 週內)

- **Search Console → 成效**:篩選該 URL,看曝光、CTR、平均排名
- **GA4**:確認有真實流量、停留時間
- **若 7 天仍未索引**:檢查 robots、canonical、是否被誤標 `draft: true`
- **依排名調整**:若主關鍵字在 11–20 名,補強內容/內鏈再次 request indexing
