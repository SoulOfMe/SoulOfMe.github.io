---
layout: default
title: 如何开始使用本项目
---

## 欢迎！

感谢您对本项目感兴趣。这是一个用于快速搭建现代化、响应式网站的模板，基于 Jekyll 和 Tailwind CSS 构建。

### 快速上手

1.  **克隆仓库**：
    ```bash
    git clone https://github.com/username/repo.git
    cd repo
    ```

2.  **安装依赖**：
    确保您已安装 Ruby 和 Jekyll。然后运行：
    ```bash
    bundle install
    ```

3.  **本地运行**：
    ```bash
    bundle exec jekyll serve
    ```
    现在，您可以在 `http://localhost:4000` 访问您的网站。

### 项目结构

-   `_config.yml`：网站配置文件。
-   `_layouts`：页面布局模板。
-   `_posts`：您的博客文章或文档。
-   `index.html`：网站主页。

### 自定义

-   **修改标题和描述**：编辑 `_config.yml` 文件。
-   **更改样式**：由于我们使用 Tailwind CSS CDN，您可以直接在 HTML 中使用 Tailwind 的类名。
-   **添加新页面**：在根目录创建新的 `.html` 或 `.md` 文件，并添加 `layout: default` 的 front matter。