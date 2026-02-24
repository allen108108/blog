# Allen's Tech Blog

使用 Jekyll + OpenClaw 自動化發布的技術部落格。

## 結構

- **main branch**: Jekyll 原始碼 + 新文章 (2026+)
- **gh-pages branch**: 編譯後的網站（包含新舊文章）
- **backup-before-automation-20260224**: 備份

## 使用 OpenClaw 發布文章

```bash
openclaw agent --message "寫一篇關於 XXX 的技術文章並發布到部落格"
```

或使用 skill：

```bash
cd ~/.openclaw/custom-skills/chirpy-blog-publisher
node index.js publish "標題" "內容" "分類1,分類2" "標籤1,標籤2"
```

## 本地開發

```bash
bundle install
bundle exec jekyll serve
```
