---
layout: wiki
title: cheat sheet
cate1:
cate2:
description: 常用命令集锦
keywords: cheat sheet
type:
link:
---

## windows

- windows 11 右键菜单设置

管理员运行命令：

`reg.exe add "HKCU\Software\Classes\CLSID\{86ca1aa0-34aa-4e8b-a509-50c905bae2a2}\InprocServer32" /f /ve` 重启就恢复win10右键了

`reg.exe delete "HKCU\Software\Classes\CLSID\{86ca1aa0-34aa-4e8b-a509-50c905bae2a2}\InprocServer32" /va /f` 这个是恢复win11右键

参考：[有没有什么办法可以让win11右键默认显示更多选项？](https://www.zhihu.com/question/480356710/answer/2279799895)

## swagger

```bash
docker pull swaggerapi/swagger-editor
docker run -d -p 80:8080 swaggerapi/swagger-editor
```

This will run Swagger Editor (in detached mode) on port 80 on your machine, so you can open it by navigating to http://localhost in your browser.

参考：[swagger-editor](https://github.com/swagger-api/swagger-editor)

