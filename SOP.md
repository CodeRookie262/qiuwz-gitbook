# GitBook (HonKit) 搭建与部署 SOP

本文档详细记录了如何从零开始搭建一个基于 HonKit (GitBook 替代方案) 的文档/博客系统，并将其自动部署到 GitHub Pages。

## 1. 环境准备

在开始之前，请确保你的电脑上安装了以下环境：

*   **Git**: 用于代码版本控制。
*   **Node.js**: 建议版本 v18 或更高 (推荐 v20+)。
    *   验证命令: `node -v`

## 2. 项目初始化

### 2.1 创建并进入目录
```bash
mkdir my-gitbook
cd my-gitbook
```

### 2.2 初始化 Node.js 项目
生成 `package.json` 文件：
```bash
npm init -y
```

### 2.3 安装 HonKit
由于官方 `gitbook-cli` 已不再维护，我们使用兼容性更好的 `honkit`：
```bash
npm install honkit --save-dev
```

### 2.4 修改 `package.json` 脚本
打开 `package.json`，在 `"scripts"` 部分添加启动和构建命令：

```json
"scripts": {
  "start": "honkit serve",
  "build": "honkit build"
}
```

## 3. 基础内容配置

### 3.1 初始化文件结构
在根目录创建以下文件：

**1. README.md** (封面/首页)
```markdown
# 文档标题

这里是简介...
```

**2. SUMMARY.md** (目录结构)
*这是 GitBook 的核心文件，决定了左侧导航栏的结构。*
```markdown
# Summary

* [简介](README.md)
* [第一章](chapter1.md)
    * [第一节](chapter1/section1.md)
```

**3. .gitignore** (忽略文件)
防止垃圾文件上传到仓库：
```text
node_modules
_book
```

### 3.2 本地预览
运行以下命令启动本地服务器：
```bash
npm run start
```
然后在浏览器访问 `http://localhost:4000` 查看效果。

## 4. GitHub 仓库与自动部署

### 4.1 创建 GitHub 仓库
在 GitHub 上创建一个新仓库（例如 `my-gitbook`），**不需要**初始化 README 或 .gitignore。

### 4.2 配置 GitHub Actions (自动部署)
为了实现“推送到 GitHub 自动更新网页”，我们需要配置 Actions。

1.  在项目根目录创建目录：`.github/workflows/`
2.  新建文件 `.github/workflows/deploy.yml`，填入以下内容：

```yaml
name: Deploy GitBook to GitHub Pages

on:
  push:
    branches:
      - main  # 监听 main 分支的变动

jobs:
  deploy:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Install dependencies
        run: npm install

      - name: Build GitBook
        run: npm run build

      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./_book
```

### 4.3 推送代码
```bash
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin <你的GitHub仓库地址>
git push -u origin main
```

## 5. 开启 GitHub Pages

1.  推送成功后，等待 Actions 运行完成（仓库页面 -> Actions 标签 -> 变绿）。
2.  进入仓库 **Settings** -> **Pages**。
3.  **Source**: 选择 `Deploy from a branch`。
4.  **Branch**: 选择 **`gh-pages`** 分支 (该分支由脚本自动生成)。
5.  点击 **Save**。

稍等片刻，顶部会出现你的访问地址：`https://<用户名>.github.io/<仓库名>/`

## 6. 日常维护

以后写文章只需要三步：
1.  在 `SUMMARY.md` 添加新章节链接。
2.  创建对应的 `.md` 文件并写入内容。
3.  提交代码：
    ```bash
    git add .
    git commit -m "更新内容"
    git push
    ```
网站会自动更新。

