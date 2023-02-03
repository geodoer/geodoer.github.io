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