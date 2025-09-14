---
title: "学习笔记：C++基础"

date: 2025-09-14T05:00:00Z
categories: ["C++"]
author: "Qingfeng Zhang"

---

# 1.环境配置

macOS环境下安装VS Code插件C/C++和CodeLLDB

添加VS Code的运行配置文件launch.json:

``` json
{
    "version": "0.2.0",
    "configurations": [
        {
            "type": "lldb",
            "request": "launch",
            "name": "C++ debug",
            "program": "${fileDirname}/${fileBasenameNoExtension}",
            "args": [],
            "cwd": "${workspaceFolder}",
            "preLaunchTask": "C/C++: g++ 生成活动文件"
        }
    ]
}
```

添加VS Code的任务配置文件tasks.json:

``` json
{
    "tasks": [
        {
            "type": "cppbuild",
            "label": "C/C++: gcc 生成活动文件",
            "command": "/usr/bin/gcc",
            "args": [
                "-fdiagnostics-color=always",
                "-g",
                "${file}",
                "-o",
                "${fileDirname}/${fileBasenameNoExtension}"
            ],
            "options": {
                "cwd": "${fileDirname}"
            },
            "problemMatcher": [
                "$gcc"
            ],
            "group": {
                "kind": "build",
                "isDefault": true
            },
            "detail": "调试器生成的任务。"
        }
    ],
    "version": "2.0.0"
}
```
