# 迁移笔记 - 原 Jekyll 站点配置

## 站点信息
- 标题: 行思笔记
- 作者: 行思笔记
- 邮箱: hi@hackthoughts.com
- 描述: 分享技术洞见、个人思考和生活方式。
- 域名: https://hackthoughts.com

## 原文章 (_posts/)
1. 2024-10-18-welcome-to-jekyll.markdown
2. 2024-10-22-partialeq-and-partialord-in-rust-traits.markdown
3. 2024-11-27-module-reloading-in-ipython.markdown
4. 2024-11-27-shortcut-actions-macos-commands.markdown

## Hugo 配置参考 (hugo-bearblog 主题)
```toml
baseURL = "https://hackthoughts.com"
theme = 'hugo-bearblog'
title = "行思笔记"
author = "行思笔记"
copyright = "Copyright © 2024, 行思笔记."
languageCode = "zh-CN"
enableRobotsTXT = true

disableKinds = ["taxonomy"]
ignoreErrors = ["error-disable-taxonomy"]

[permalinks]
  blog = "/:slug/"

[params]
  description = "分享技术洞见、个人思考和生活方式。"

[markup]
  [markup.highlight]
    style = 'friendly'
    lineNos = true
    lineNumbersInTable = false
    codeFences = true
```
