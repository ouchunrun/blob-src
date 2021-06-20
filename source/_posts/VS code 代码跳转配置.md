---
title: VS code 代码跳转配置
date: 2021-6-20
tags: [vscode]
---

- 1.文件跟目录下添加`.vscode\setting.json` 文件，添加内容如下
```
{
    "files.defaultLanguage": "cpp",
    "editor.formatOnType": true,
    "editor.snippetSuggestions": "top",
    "C_Cpp.clang_format_sortIncludes": true,
    "C_Cpp.intelliSenseEngine": "Tag Parser",
    "C_Cpp.errorSquiggles": "Disabled",
    "C_Cpp.autocomplete": "Disabled",
    "files.associations": {
        "iostream": "cpp"
    }
}
```

- 2.设置里面搜索`intelliSenseEngine`，设置`C_Cpp: Intelli Sense Engine`为`Tag Parser`

- 3.在引用商店中搜索扩展，安装以下扩展程序：
    - C/C++
    - C++ Intellisense
    - Code Runner

- 4.重启 vs code
