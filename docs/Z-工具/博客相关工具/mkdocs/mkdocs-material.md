Links

- [mkdocs-material](https://github.com/squidfunk/mkdocs-material)
- [Markdown样式](https://squidfunk.github.io/mkdocs-material/reference/admonitions/)

应用案例

- https://kitware.github.io/vtk-examples/site/

## 提示框
配置
```yaml
markdown_extensions:
  - admonition
  - pymdownx.details
  - pymdownx.superfences
theme:
  icon:
    admonition:
      note: octicons/tag-16
      abstract: octicons/checklist-16
      info: octicons/info-16
      tip: octicons/squirrel-16
      success: octicons/check-16
      question: octicons/question-16
      warning: octicons/alert-16
      failure: octicons/x-circle-16
      danger: octicons/zap-16
      bug: octicons/bug-16
      example: octicons/beaker-16
      quote: octicons/quote-16
```

示例

!!! note "这是 note 类型的提示框"
提示：更多精彩内容记得关注我啊

```markdown
!!! note "这是 note 类型的提示框"
提示：更多精彩内容记得关注我啊

!!! success "这是 success 类型的提示框"
成功！

!!! failure "这是 failure 类型的提示框"
失败！

!!! bug "这是 bug 类型的提示框"
发现一个 bug，请尽快修复！
```

提示框折叠：`!!!`改成`???`即可

## 数学公式
参考文档：[MathJax - Material for MkDocs (squidfunk.github.io)](https://squidfunk.github.io/mkdocs-material/reference/mathjax/#mkdocsyml)

添加代码

1. `./docs/javascripts/mathjax.js`
```javascript
window.MathJax = {
  tex: {
    inlineMath: [["\\(", "\\)"]],
    displayMath: [["\\[", "\\]"]],
    processEscapes: true,
    processEnvironments: true
  },
  options: {
    ignoreHtmlClass: ".*|",
    processHtmlClass: "arithmatex"
  }
};

document$.subscribe(() => { 
  MathJax.typesetPromise()
})
```

2. `./mkdocs.yml`
```yaml
markdown_extensions:
  - pymdownx.arithmatex:
      generic: true

extra_javascript:
  - javascripts/mathjax.js
  - https://polyfill.io/v3/polyfill.min.js?features=es6
  - https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js
```

使用

1. 块：`$$...$$`、`\[...\]`
2. 行：`$...$`、`\(...\)`


## 示例
### 普通文档
`mkdocs.yml`

```yaml
site_name: 学习Cherno的OpenGL课程
site_url: https://geodoer.gitee.com/learn-cherno-open-gl

theme:
  name: material
  palette:
    scheme: slate
  language: "zh"
  features:
    - navigation.instant  #即时加载（无需完全加载页面，即可打开网站）
    - navigation.tracking #锚点跟踪
    - navigation.footer   #上一页、下一页
    # - navigation.tabs     #将一级目录生成选项卡
    - navigation.tabs.sticky  #黏性
    - navigation.indexes      #子目录下的index.md会在分区项中显示（使用title修改标题，不然会标题会显示成index）
    - toc.follow              #目录锚点跟随
    - navigation.top          #向上滚时，提供"回到顶部"的按钮


  #调色板（切换主题）
  palette:
    - scheme: default
      toggle:
        icon: material/brightness-7 
        name: Switch to dark mode
    - scheme: slate
      toggle:
        icon: material/brightness-4
        name: Switch to light mode

#插件
plugins:
  - tags
  - search

#md文档的扩展
markdown_extensions:
  - abbr
  - admonition
  - attr_list
  - def_list
  - footnotes
  - md_in_html
  - toc:
      permalink: true
  - pymdownx.arithmatex:
      generic: true
  - pymdownx.betterem:
      smart_enable: all
  - pymdownx.caret
  - pymdownx.details
  - pymdownx.emoji:
      emoji_generator: !!python/name:materialx.emoji.to_svg
      emoji_index: !!python/name:materialx.emoji.twemoji
  - pymdownx.highlight:
      anchor_linenums: true
  - pymdownx.inlinehilite
  - pymdownx.keys
  - pymdownx.magiclink:
      repo_url_shorthand: true
      user: squidfunk
      repo: mkdocs-material
  - pymdownx.mark
  - pymdownx.smartsymbols
  - pymdownx.superfences:
      custom_fences:
        - name: mermaid
          class: mermaid
          format: !!python/name:pymdownx.superfences.fence_code_format
  - pymdownx.tabbed:
      alternate_style: true
  - pymdownx.tasklist:
      custom_checkbox: true
  - pymdownx.tilde
```