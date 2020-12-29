---
title: platform_device_id
tags: 
  - go-flutter-plugin
  - Flutter
comments: true
categories: 
  - products
toc: true
donate: true
keywords: platform_device_id,go-flutter-plugin,flutter
date: 2020-04-22
---

Get device id from android、ios、windows、linux、mac

## Description

- Windows 

`BIOS UUID` e.g. `99A4D301-53F5-11CB-8CA0-9CA39A9E1F01`
- Linux 

`BIOS UUID` e.g. `32a70060-2a39-437e-88e2-d68e6154de9f`
- Mac 

`IOPlatformUUID` e.g. `02662E79-E342-521C-98EA-D4C18B61FEF3`

- Android 

`androidid` 

- IOS 

`identifierForVendor`


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
import 'package:platform_device_id/platform_device_id.dart';

String deviceId = await PlatformDeviceId.getDeviceId;
```

## Use go-flutter

### Usage

Import as:

```go
import platform_device_id "github.com/BestBurning/platform_device_id/go"
```

Then add the following option to your go-flutter [application options](https://github.com/go-flutter-desktop/go-flutter/wiki/Plugin-info):

```go
flutter.AddPlugin(&platform_device_id.PlatformDeviceIdPlugin{}),
```

## Pub Dev

it's published to [pub.dev](https://pub.dev/packages/platform_device_id#-readme-tab-)

## Source Code

[platform_device_id](https://github.com/BestBurning/platform_device_id) open source with [Github](https://github.com/BestBurning)
and the licenses is [BSD-3-Clause](https://opensource.org/licenses/BSD-3-Clause)

