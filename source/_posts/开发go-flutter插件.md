---
title: 开发go-flutter插件
tags: Flutter
comments: true
categories: technology
originContent: 
toc: true
keyworlds: flutter,插件,go-flutter
date: 2020-04-19 10:03:34
---


### 创建

```
flutter create --org com.example --template=plugin test_hover
```

### 初始化

```
cd test_hover
# test_hover/

hover init-plugin github.com/my-organization/test_hover
```

### 插件开发

```
cd go
# test_hover/go
```

### 运行示例

```
cd ../example
# test_hover/example

flutter build bundle
hover init
hover plugins get
yes | hover run
```

### git本地、远端初始化

```
# test_hover/example
```

### go-flutter发布

```
cd ..
# test_hover

hover publish-plugin
```

### flutter发布

```
# test_hover

flutter packages pub publish --dry-run
flutter packages pub publish
```