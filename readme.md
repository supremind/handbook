# AVA Handbook

这个文件 (readme.md) 其实应该放整个文档的简介, 用户打开文档首页看到的就是这里.

## 编辑

- `SUMMARY.md` 是文档目录, 添加新章节需要更新该文档
- 图片放 `images` 目录, 按章节分子目录, 如 `images/ch-01/ava-concepts.jpg`, 在正文中即可这样引用 (使用相对项目根目录的路径)
    ```markdown
    ![AVA](/images/ch-01/ava-concepts.jpg)
    ```

## 工具

1. 使用 Markdown 编辑器编辑文档
    - [Typora](https://typora.io): 所见即所得, 新手友好

2. 使用 [Github Desktop](https://desktop.github.com/) 进行版本管理: 拉取别人的更新, 推送自己的更新, 提 pr 等

## build

使用 gitbook-cli 本地 build 文档

```bash
# 安装 gitbook-cli
❯ npm install gitbook-cli -g
❯ gitbook init
# 本地预览文档
❯ gitbook serve
# build 文档
❯ gitbook build
```

生成的内容 commit 至 gh-pages 分支, 并 push
```bash
❯ git checkout gh-pages
❯ git rm --cached -r .
❯ cp -r _book/* .
❯ git add .
❯ git commit ...
❯ ...
```