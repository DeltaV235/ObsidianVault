---

title: Rime 自定义快捷键
created: 2025-11-29
language: Yaml
tags:
    - Rime
    - Shortcut
    - Keybind

---

```yaml
patch: 
# 其他配置... 
  key_binder: 
    bindings: 
      - { when: has_menu, accept: "Control+k", send: Page_Up } 
      - { when: has_menu, accept: "Control+j", send: Page_Down } 
      - { when: has_menu, accept: "Control+h", send: Left }
      - { when: has_menu, accept: "Control+l", send: Right }
```

## Reference

[Rime 输入法中的快捷键](https://einverne.github.io/post/2021/10/rime-shortcut.html)