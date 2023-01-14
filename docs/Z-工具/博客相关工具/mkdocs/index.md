---
title: Mkdocs
---

相关链接：

1. [mkdocs中文文档](https://markdown-docs-zh.readthedocs.io/zh_CN/latest/)
2. [MkDocs Themes](https://github.com/mkdocs/mkdocs/wiki/MkDocs-Themes)

## 编写文档
1. 文档写在`.\docs`文件夹下
2. 编写首页`.\docs\index.md`
3. 在项目根目录下，创建mkdocs配置文件`mkdocs.yml`
4. `mkdocs serve`本地调试

## 部署
### Github
1. 运行`mkdocs gh-deploy`命令，工具会将相应内容推送到`gh-pages`分支上
2. 在Github项目设置中，在GitPage中选择`gh-pages`分支
3. 通过`https://<user-name>.github.io/<project-name>`访问

### Gitee
1. `mkdocs build`生成静态文件，位于`.\site`
2. 推送`.\mkdocs.yml`、`.\docs\*`、`.\site\*`到仓库当中
3. 开启GiteePages服务，选择对应分支
4. 通过`https://<user-name>.gitee.io/<project-name>`访问

### Gitee
参考链接：

1. [使用Gitlab Pages发布Obsidian笔记](https://about.gitlab.com/blog/2022/03/15/publishing-obsidian-notes-with-gitlab-pages/)