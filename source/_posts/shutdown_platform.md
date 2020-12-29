---
title: shutdown_platform
tags: 
  - go-flutter-plugin
  - Flutter
comments: true
categories: 
  - products
toc: true
donate: true
keywords: shutdown_platform,go-flutter-plugin,flutter
date: 2020-04-23
---

The plugin for `go-flutter` to shutdown platform `windows`、`linux`、`mac` only

## Getting Started

### Install

Add this to your package's pubspec.yaml file:

```
dependencies:
  platform_device_id: ^0.1.0
```

### Usage

a simple usage example:

```
import 'package:shutdown_platform/shutdown_platform.dart';

String out = await ShutdownPlatform.shutdownNow;
```

## Use go-flutter

### Usage

Import as:

```go
import shutdown_platform "github.com/BestBurning/shutdown_platform/go"
```

Then add the following option to your go-flutter [application options](https://github.com/go-flutter-desktop/go-flutter/wiki/Plugin-info):

```go
flutter.AddPlugin(&shutdown_platform.ShutdownPlatformPlugin{}),
```


## Pub Dev

it's published to [pub.dev](https://pub.dev/packages/shutdown_platform#-readme-tab-)

## Source Code

[shutdown_platform](https://github.com/BestBurning/shutdown_platform) open source with [Github](https://github.com/BestBurning)
and the licenses is [BSD-3-Clause](https://opensource.org/licenses/BSD-3-Clause)

