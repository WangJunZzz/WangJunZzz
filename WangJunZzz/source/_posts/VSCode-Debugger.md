---
title: VSCode-Debugger
date: 2019-08-03 11:36:50
tags:
- VSCode
categories: 
- DotNet
---

### VSCode 调试
1. 新建一个dotnet mvc项目
2. 项目中添加 Microsoft.DotNet.Watcher.Tools 引用
3. VSCode添加c#插件
4. 添加附件到进程调试配置文件
``` json
{
    // 使用 IntelliSense 了解相关属性。 
    // 悬停以查看现有属性的描述。
    // 欲了解更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [   
        {
            "name": "附加到进程",
            "type": "coreclr",
            "request": "attach",
            "processId": "${command:pickProcess}"
        }
    ]
}
```
5. 在控制台运行dotnet watch run