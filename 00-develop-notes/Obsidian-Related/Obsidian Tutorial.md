---
title: Obsidian Tutorial
created: 2024-05-19
tags:
    - Tutorial
    - Obsidian
---

# Obsidian Tutorial

## Video Tutorial

- [Tutorial From Bilibili](https://www.bilibili.com/video/BV1Qi421D7uv/?p=14&spm_id_from=pageDriver)

## Shortcuts

**Quick Switcher**: `CMD + O`
**Command Palette**: `CMD + P`
**Insert Markdown Link**: `CMD + K`
**Insert Template**: `CMD + OPT + T`

## Query

Find in current note: `CMD + F`
Search in all note: `CMD + SHIFT + F`

### Search Options 

- `keyword1 keyword2`: 搜索同时存在两个关键词的 note
- `keyword1 OR keyword2`: 搜索存在任意一个关键词的 note
- `tag:#TAG`: 搜索 TAG
- `line:(keyword1 keyword2)`: Find matches on the same line.
- `task:""`: Find all task

### Embed search results in a note

To embed search results in a note, add a `query` code block:

```query
embed OR search
```

### Reference

[Obsidian Search Plugin Help Document](https://help.obsidian.md/Plugins/Search)

## Theme

- Minimal
- Primary

## Plugins

**Installed Community Plugins**

- Dataview
- Excalidraw
- File Explorer Note Count
- Kanban
- Recent Files
- Vimrc Support

### Dataview

#### Example

```dataview
table file.size/1024 + " KB" as file-size, file.ctime as "created time"
from "Obsidian-Related"
sort file.ctime desc
```

#### Reference

[Dataview Reference Document](https://blacksmithgu.github.io/obsidian-dataview)

## Callout

Use callouts to include additional content without breaking the flow of your notes.

Below is sample.

> [!NOTE] Title
> References
> References2

### Reference

[Callouts Guide](https://help.obsidian.md/Editing+and+formatting/Callouts)
