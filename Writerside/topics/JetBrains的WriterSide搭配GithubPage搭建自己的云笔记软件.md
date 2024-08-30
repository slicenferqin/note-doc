# JetBrains的 WriterSide 搭配 GithubPage搭建自己的云笔记软件
## 什么是 WriterSide {id="writerside"}
JetBrains 的 WriterSide 是一个专为编写技术文档和帮助文件而设计的强大文档编辑工具。它支持多种格式，并拥有丰富的语法高亮和自动补全功能，适合开发者和技术写作爱好者使用。

主要功能和特点：
- **Markdown 支持**：WriterSide 完全支持 Markdown 语法，使得编写结构化内容更加简单高效。
- **与 JetBrains IDE 集成**：WriterSide 无缝嵌入 JetBrains 系列 IDE，允许你在开发过程中随时记录和编辑文档。
- **版本控制和协作**：通过集成 Git 版本控制系统，WriterSide 提供了强大的版本管理和协作功能，使得多人编辑和跟踪文档变得更加容易。

选择 WriterSide 作为云笔记软件的编辑器，主要是因为它的灵活性和强大功能，特别适合需要编写复杂内容或技术文档的用户，而且作为 JetBrains 系列的软件，统一的快捷键、UI界面会带来更好的体验。
## 什么是 GithubPage
Github Pages 是 Github 提供的一项免费静态网站托管服务，它允许你将 Git 仓库中的内容发布为一个静态网站，非常适合托管个人博客、项目文档等内容。

功能和特点：
- **免费托管**：Github Pages 为你提供免费的静态网站托管服务，并支持自定义域名，方便你打造属于自己的品牌形象。
- **静态网页生成支持**：完全支持静态网页生成工具（如 Jekyll），可以轻松地将 Markdown 文件转化为网页。
- **自动部署**：与 Git 仓库紧密集成，每当你更新仓库时，Github Pages 会自动重新部署网站，让你的内容始终保持最新。

通过 Github Pages 托管你的云笔记软件，你可以在任何时间、任何地点访问你的笔记，同时享受 Github 的版本控制和协作功能，确保你的笔记内容安全可靠。
## 如何搭建自己的云笔记/个人博客站
### 在Github上创建一个笔记仓库 {id="github"}
![新建仓库](repo.png)
### 新建一个WriterSide项目 {id="writerside_1"}
略
### 新增Github工作流 {id = "github_1"}
![workflow.png](workflow.png)

在项目下依次新建.github/workflow/build-docs.yml
在 build-docs.yml中填入以下内容
```yaml
name: Build documentation

on:
  push:
    branches: ["main"]
  workflow_dispatch:

permissions:
  id-token: write
  pages: write

env:
  INSTANCE: 'Writerside/in'
  ARTIFACT: 'webHelpIN2-all.zip'
  DOCKER_VERSION: '241.18775'

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Build docs using Writerside Docker builder
        uses: JetBrains/writerside-github-action@v4
        with:
          instance: ${{ env.INSTANCE }}
          artifact: ${{ env.ARTIFACT }}
          docker-version: ${{ env.DOCKER_VERSION }}

      - name: Save artifact with build results
        uses: actions/upload-artifact@v4
        with:
          name: docs
          path: |
            artifacts/${{ env.ARTIFACT }}
            artifacts/report.json
          retention-days: 7
  test:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Download artifacts
        uses: actions/download-artifact@v4
        with:
          name: docs
          path: artifacts

      - name: Test documentation
        uses: JetBrains/writerside-checker-action@v1
        with:
          instance: ${{ env.INSTANCE }}
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    needs: [build, test]
    runs-on: ubuntu-latest
    steps:
      - name: Download artifacts
        uses: actions/download-artifact@v4
        with:
          name: docs

      - name: Unzip artifact
        run: unzip -O UTF-8 -qq '${{ env.ARTIFACT }}' -d dir

      - name: Setup Pages
        uses: actions/configure-pages@v4

      - name: Package and upload Pages artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: dir

      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```
### WriterSide项目绑定远程仓库 {id="writerside_2"}
```Bash
git init
git add .
git commit -m "first commit"
git branch -M main
git remote add origin git@github.com:xxx/notes.git
git push -u origin main
```
### 新增Markdown文档
在 topics文件夹下新建一个 markdown 文档
![topics.png](topics.png)

并将其添加到 in.tree文件中
![tree.png](tree.png)

这样就 ok 啦
### 推送到仓库
使用 IDE自带的Git 工具将文件推送到仓库
或者使用命令
```Bash
git push -u origin main
```
### 查看workflow执行情况
![actions.png](actions.png)

在Actions中可以查看 workflow 的执行情况，全部执行通过就代表文章部署到站点成功，接下来就可以通过链接访问到页面了。
### 查看博客站点
访问地址规则：https://用户名.github.io/仓库名,就可以看到站点页面了。

