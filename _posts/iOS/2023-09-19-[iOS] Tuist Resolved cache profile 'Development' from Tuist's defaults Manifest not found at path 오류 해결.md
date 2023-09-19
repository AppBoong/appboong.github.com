---
title: "[iOS] Tuist Resolved cache profile 'Development' from Tuist's defaults Manifest not found at path 오류 해결"
last_modified_at: 2023-09-19T19:06:00-05:00
toc: true
categories:
  - iOS
tags:
  - Tuist
  - Xcode
---

## Tuist generate 이슈

```shell
tuist generate
```
> 터미널에 tuist 로 프로젝트를 생성하려는데 이런 오류가 발생했다

```shell
Resolved cache profile 'Development' from Tuist's defaults Manifest not found at path /Users/username Consider creating an issue using the following link:
https://github.com/tuist/tuist/issues/new/choose
```

## 해결 방법

```shell
sudo xcode-select -s /Applications/Xcode.app/Contents/Developer
```